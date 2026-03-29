# Security & Transparency

This document describes SqlPulse's security model, what data leaves your machine, and how the license system works. The full source code of all security-critical paths is included below so you can audit it directly.

---

## What data leaves your machine

**During normal use: nothing.**

Query history, filter state, settings, and all other data is stored locally in `%LOCALAPPDATA%\SqlPulseTool\`. Nothing is uploaded anywhere.

---

## Credential storage policy

**SqlPulse does not store SQL Server passwords.**

This is a deliberate design decision, not a limitation:

- SQL Server passwords are not written to `settings.json` or any other file
- The **Quick Connect** feature stores server name, database, and username (for display only) — never the password
- Authentication is handled by SSMS itself when you open a new query window — SqlPulse does not intercept or cache credentials
- **Windows Authentication (Integrated Security)** is recommended and fully supported with no credentials to manage

If you use SQL Server Authentication, your login name is shown in the Quick Connect list as a reminder, but no password is stored or transmitted.

> **Why not store passwords with DPAPI?** DPAPI (Windows Data Protection API) encrypts data so only the current Windows user can decrypt it on the same machine. While this is a common pattern, it still results in credential material on disk — accessible if the Windows account is compromised, accidentally committed to version control, or included in a backup. SSMS itself uses Windows Credential Manager rather than file-based credential storage. SqlPulse follows the same principle: **no passwords on disk**.

---

**During Pro license activation: exactly three values are sent:**

| Field | Value | Notes |
|-------|-------|-------|
| `licenseKey` | The key you purchased | Used to look up your license record |
| `fingerprint` | `SHA-256(UUID + CpuId + DiskSerial)`, first 32 hex chars | One-way hash — raw hardware IDs are never sent |
| `isVm` | `true` / `false` | Used to apply VM licensing rules |

No hostname, no username, no IP address is stored server-side. No query content, no connection strings, nothing else.

After activation, the signed token is stored locally and verified **offline** on every startup. No further internet calls are made unless you explicitly trigger a renewal.

> **Note:** The activation server is not yet deployed. Pro license activation is not live in the current release. This document describes the intended architecture.

---

## How the license system works

The license system uses **RSA-SHA256 signed tokens** (JWT-style). The private key lives exclusively in the Cloudflare Worker environment (never in the extension binary). The extension only contains the public key — used to verify the token locally.

This means:
- There is no "phone home" on startup
- The token cannot be forged without the private key
- Opening this code does **not** help anyone bypass the license — the security is entirely in the RSA key pair, not in code obscurity

---

## Activation server source (Cloudflare Worker)

The server is ~200 lines of TypeScript deployed as a Cloudflare Worker. It has no database — state is stored in Cloudflare KV.

### `src/index.ts` — HTTP endpoints

```typescript
/**
 * SqlPulse License Activation – Cloudflare Worker
 *
 * Endpoints:
 *   POST /activate   – register machine, return signed token
 *   POST /renew      – extend expiry, return new signed token
 *   POST /deactivate – free machine slot
 */

// POST /activate
async function handleActivate(body, env) {
  const licenseKey  = body.licenseKey;   // your purchased key
  const fingerprint = body.fingerprint;  // SHA-256 hash of hardware IDs
  const isVm        = body.isVm;         // boolean

  // Look up the license in KV store
  const record = await getLicense(env.LICENSES, licenseKey);
  if (!record) return error('License key not found.', 404);
  if (!record.active) return error('License is inactive.', 403);

  // Register this machine (checks maxComputers limit)
  const result = await registerFingerprint(env.LICENSES, licenseKey, fingerprint, isVm);
  if ('error' in result) return error(result.error, 403);

  // Build and sign a JWT-style token valid for 365 days
  const token = await buildToken(licenseKey, fingerprint, isVm, result.record, env.PRIVATE_KEY);
  return json({ token, tier: result.record.tier, expiresAt: '...' });
}
```

### `src/tokenSigner.ts` — RSA-SHA256 signing

```typescript
export async function signToken(payload, privateKeyPem) {
  const privateKey = await importPrivateKey(privateKeyPem);

  // Token format: Base64Url(header).Base64Url(payload).Base64Url(RSA-SHA256-signature)
  const header     = toBase64Url(JSON.stringify({ alg: 'RS256', typ: 'JWT' }));
  const payloadB64 = toBase64Url(JSON.stringify(payload));

  const signature = await crypto.subtle.sign(
    'RSASSA-PKCS1-v1_5',
    privateKey,
    new TextEncoder().encode(`${header}.${payloadB64}`)
  );

  return `${header}.${payloadB64}.${arrayBufferToBase64Url(signature)}`;
}
```

**Payload fields:**

| Field | Meaning |
|-------|---------|
| `k` | License key |
| `f` | Machine fingerprint (32-char hex hash) |
| `v` | Is virtual machine |
| `t` | License tier |
| `mc` | Max computers (0 = unlimited) |
| `ia` | Issued at (unix timestamp) |
| `ex` | Expires at (unix timestamp, `ia + 365 days`) |

---

## Client-side source (C#, .NET Framework 4.8)

### `MachineFingerprint.cs` — what the fingerprint contains

```csharp
private static MachineFingerprintResult Compute()
{
    // Read three WMI values
    var uuid       = GetWmiValue("Win32_ComputerSystemProduct", "UUID");
    var cpuId      = GetWmiValue("Win32_Processor", "ProcessorId");
    var diskSerial = GetWmiValue("Win32_DiskDrive", "SerialNumber");

    // Hash them — raw values are never sent anywhere
    var raw = uuid + cpuId + diskSerial;
    using var sha = SHA256.Create();
    var hashBytes = sha.ComputeHash(Encoding.UTF8.GetBytes(raw));
    var fp = BitConverter.ToString(hashBytes).Replace("-", "").ToLowerInvariant();

    // Only the first 32 hex characters are used
    fp = fp.Length >= 32 ? fp.Substring(0, 32) : fp;

    return new MachineFingerprintResult { Fingerprint = fp, IsVirtualMachine = isVm };
}
```

### `ActivationToken.cs` — local RSA signature verification

```csharp
public static ActivationToken? Parse(string? raw)
{
    // Token is three Base64Url parts: header.payload.signature
    var parts = raw.Trim().Split('.');
    if (parts.Length != 3) return null;

    // Verify RSA-SHA256 signature using the embedded public key
    // If this fails, the token is rejected — no network call needed
    var signedBytes = Encoding.UTF8.GetBytes(parts[0] + "." + parts[1]);
    var sigBytes    = FromBase64Url(parts[2]);
    if (!VerifySignature(signedBytes, sigBytes)) return null;

    // Decode payload
    var payloadJson = Encoding.UTF8.GetString(FromBase64Url(parts[1]));
    using var doc   = JsonDocument.Parse(payloadJson);
    // ... map fields to ActivationToken object
}

private static bool VerifySignature(byte[] data, byte[] signature)
{
    using var rsa = RSA.Create();
    rsa.ImportParameters(new RSAParameters
    {
        Modulus  = PublicKeyModulus,   // 2048-bit RSA public key, embedded in binary
        Exponent = PublicKeyExponent,  // 65537
    });
    return rsa.VerifyData(data, signature, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
}
```

The public key (modulus) is embedded as raw bytes in the binary — no network call is needed to verify a token.

---

## Verification steps

If you want to independently verify the above:

1. **ILSpy / dnSpy** — decompile `SqlPulseTool.SsmsExtension.dll`, look at `MachineFingerprint` and `ActivationToken` classes
2. **Wireshark / Fiddler** — monitor outbound HTTPS during activation; you will see exactly one POST to the activation endpoint and nothing else during normal use
3. **Process Monitor** — filter on the SSMS process to verify that no unexpected file or network activity occurs

---

## Reporting security issues

Please report security vulnerabilities via [GitHub Issues](https://github.com/IstvanSafar/SqlPulse/issues) or by emailing directly. Do not post exploit details publicly before a fix is available.
