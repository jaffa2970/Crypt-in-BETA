# Crypt-in — developer beta

Hardware-backed file encryption on an ESP32-S3 dongle. This repository is where
the beta package is downloaded from, and where you report what you find.

**Beta opens 30 September 2026.**

---

## Read this first: you are getting a `-dev` unit

This beta does not go to end users. It goes to developers who will open the
dongle. The units run the **`-dev`** firmware with eFuses **not burned**, and
that has consequences you would find in an afternoon anyway — so we would rather
say them first:

- **`CMD_GET_SECRET` (`0x04`) answers.** `file_secret` leaves the chip on
  request. On `-prod` that opcode is closed.
- **The flash is in the clear.** No flash encryption: an SPI dump gives up the
  seed.
- **No Secure Boot v2, JTAG not disabled, units are re-flashable** — which is
  also why you can turn the board back into an ordinary ESP32-S3 if you stop.

**A finding on `-dev` is not automatically a finding on the product.** Here is
the honest cut:

| | `-dev` (beta) | `-prod` (sold) |
|---|---|---|
| `0x04` / `file_secret` | answers | closed |
| flash | in the clear | encrypted |
| Secure Boot v2, JTAG | absent | active |
| **HID protocol, PIN gate, encryption, licence enforced on-chip** | **identical** | **identical** |

**That last row is where the beta is worth your time.** Everything in it is the
same code, so a defect found there is a defect in the product — not in the test
unit. Telling you where to look is more useful than letting you spend a weekend
finding what we already know.

---

## One download today — and a second one, later

| download | talks to | use it for |
|---|---|---|
| `cryptin-beta-*.zip` | **production** — `license.lake8.dev` | using the dongle: setup, encryption, backup, licence transfer. This is the product |

A second package is coming: a red-team build that points at a **clone** of the
licence server, so that the server can be attacked without anyone touching the
one that serves real licences. It is not in this Release, and **we are not
putting a date on it** — it will appear as an extra asset here when it is ready.

### The licence server is not a target yet, and here is the honest reason

The clone exists and runs **the same image as production** — same Dockerfile,
same entrypoint, same code, only the `.env` differs. What it does not have yet
is a network of its own: today it sits on the same LAN as the production
machine, which also serves other products and holds signing keys that cannot be
rotated. Inviting attacks at it right now would be inviting them onto that
machine. So we are not inviting them.

**Out of scope for this beta:** `license.lake8.dev`, the machines behind it, and
the other products that share them. If you find something that only reproduces
against the server, **report it and stop pulling on it** — the VDP, its safe
harbour and the 7 / 21 / 90 SLAs cover you exactly the same for that report.

**In scope, and it is the larger half:** the `-dev` firmware, the HID protocol,
the PIN gate, the on-chip encryption, the Windows application, and this package
itself. The table above says which of those carry over to the shipped product
unchanged — all of them.

⚠️ **One thing that is *not* a finding:** forging a licence by dumping your own
flash. On a `-dev` unit the flash is in the clear and the licence check is not a
security boundary — that is the definition of `-dev`, and it is in the table
above.

When the clone opens, this is the objective we most want someone to reach, and
it is stated here so you can start thinking about it:

> **Make the licence server issue a token it should not have issued — and show a
> real dongle accepting it.**

---

## Hardware you need

We do not ship hardware. You buy the board.

| Requirement | Why |
|---|---|
| ESP32-S3 with **native USB** | without it there is no HID, and the dongle does not exist. A board with only a serial bridge cannot be one |
| flash **≥ 8 MB** | the partition table ends at `0x800000` |
| flash that sustains **QIO at 80 MHz** | size alone is not enough — the flash chip has to hold that mode |
| **two USB-C ports** (or a separate UART header) | flashing goes over UART, where RTS drives EN and auto-reset works. On the native socket RTS does not reach EN — measured three times |

The reference board is an **N16R8**. ⚠️ **Note what is *not* required:** the
firmware needs neither the 16 MB nor the PSRAM — it declares 8 MB and leaves
PSRAM off. Declaring 16 MB of flash *or* Octal PSRAM hangs the unit in the
second-stage bootloader (cause still unknown). An N16R8 boots precisely because
those two are never touched. If you read "N16R8" and conclude you need 16 MB,
you are reading it wrong: it is blessed because it is the only configuration
that has ever booted, not because of what it has extra.

## System requirements

**Windows 11 or later, 64-bit.** No runtime to install: the application is
self-contained.

*(Windows 10 left Microsoft support on 14 October 2025. The app is not blocked
there — "unsupported" is a support statement, not a lock — but we will not claim
it.)*

---

## Download and verify

Get the package from [Releases](../../releases). **Verify it before running it:**
it contains unsigned executables that flash hardware, which is exactly what a
careful person checks first.

```powershell
Select-String -Path .\SHA256SUMS.txt -Pattern '([0-9a-f]{64})\s+(.+)' | ForEach-Object {
  $h = $_.Matches[0].Groups[1].Value
  $f = $_.Matches[0].Groups[2].Value.Trim()
  $a = (Get-FileHash $f -Algorithm SHA256).Hash
  "{0}  {1}" -f $(if ($a -eq $h) { 'OK     ' } else { 'DIFFERS' }), $f
}
```

**You should see five `OK` lines.** If you see four, that is not a pass.

*(For a single file from `cmd`: `certutil -hashfile CryptinPersonal.exe SHA256`.)*

### Two things Windows will do, and one you should not do

- **"Windows protected your PC" will appear.** The executables are **not
  signed**. The path through is *More info → Run anyway*.
- **Do not run it as administrator.** SmartScreen triggers on the Mark of the
  Web and ignores privileges, so elevating bypasses nothing — and an unsigned
  binary asking for elevation shows the yellow "Unknown publisher" UAC prompt,
  which is worse. The app does not need it: opening a COM port is not privileged.

⚠️ **The beta does not have reproducible builds.** .NET single-file executables
are not byte-for-byte reproducible — two builds twenty minutes apart from
identical sources produce different hashes. The published hash identifies **the
artefact you received**, not the source.

---

## Updating the firmware during the beta

> **During the beta, updating the firmware means reprogramming from scratch:**
> the dongle comes back blank, with a new seed and a new PIN, and the licence has
> to be transferred again. **The 24 words of the previous dongle do not bring it
> back.**

The merged image spans `0x0`–`0x94E50`, and the bytes landing on the `nvs`
partition are padding. Writing it erases seed, `file_secret`, PIN counter and
**K_attestation** — and K_attestation is not derived from the seed, so it cannot
be recovered from the words. An update that *preserves* identity would write only
the application at `0x10000`, and that path does not exist yet.

---

## Reporting what you find

**Open an issue here.** That is the channel — a new feature with no way back
produces frustration, not data.

For anything security-relevant, the reference perimeter is the vulnerability
disclosure policy, with its published SLAs: **7 days to acknowledge, 21 to
assess, 90 to fix.**

## Licensing — the repository is not the package

The `LICENSE` file in this repository covers **this repository's own files**. It
does not cover what you download from the Release, which is three different
things:

- **`CryptinPersonal.exe`, `CryptinUpdater.exe` and the dongle firmware** are
  **proprietary**. They are not Apache-2.0, and the `LICENSE` above does not
  apply to them.
- **`esptool.exe` v5.3.1** is **GPL-2.0**, unmodified, taken from Espressif's
  official release. It is invoked as a **separate process**, and its licence text
  travels inside the archive at `licenses/esptool-v5.3.1-GPL-2.0.txt`.
- **CryptinSDK** is Apache-2.0 and lives in its own repository — none of it is in
  this download.

Full terms are on the site: <https://cryptin.lake8.dev/docs/license/>.

## What this beta does not promise

- ⛔ **No OTA and no firmware update path.** See above.
- ⛔ **Licence expiry is not enforced offline.** Nothing sets the dongle's clock,
  so offline a licence does not expire. Said plainly rather than discovered.
- ⛔ **The session is not bound to the process.** Auto-lock is done, the binding
  is not. It is already published among the open findings on the security model
  page, and it stays there.
- ⛔ **Beta units are not production-class**, and a beta dongle does not become a
  production dongle: the eFuses and the signing key both change at that boundary.
