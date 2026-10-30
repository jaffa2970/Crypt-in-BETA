# Crypt-in — developer beta

Hardware-backed file encryption on an ESP32-S3 dongle. This repository is where
the beta package is downloaded from, and where you report what you find.

**The beta is live since 29 September 2026.** Current package: **`beta-2026-10-30`**,
published 30 October 2026. It adds **New identity** to the app, the guided cure for the
G2 advisory; the firmware is still the one of `beta-2026-10-13` — see
[What changed in beta-2026-10-30](#what-changed-in-beta-2026-10-30).

> ⚠️ **If you flashed a dongle from the `beta-2026-09-24` package, update it.**
> Seven red-team rounds on the firmware found defects in that version, two of them
> rated High — the list is under [What changed in beta-2026-10-13](#what-changed-in-beta-2026-10-13).
> Updating keeps your 24 words, your PIN, your files and your licence: see
> [Updating from beta-2026-09-24](#updating-from-beta-2026-09-24). **Do not update
> with the command-line updater.**
>
> **Already on `beta-2026-10-13`?** Replace the app folder with the new package.
> Nothing to flash, no transfer. Then consider **New identity** — see
> [What changed in beta-2026-10-30](#what-changed-in-beta-2026-10-30).

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

There is a second build, for the red team: it points at a **clone** of the
licence server, so that the server can be attacked without anyone touching the
one that serves real licences. It is not an asset of this Release: you build it
yourself from the published sources,
[`jaffa2970/cryptin-sdk-beta`](https://github.com/jaffa2970/cryptin-sdk-beta),
with `dotnet build -c RedTeam`, and every window of that build carries
«⚠ RED TEAM — redteam-license.lake8.dev» in its title. **The red-team track is
open since 29 October 2026** (target date was 30 October 2026, after a slip
from 15 October 2026; the cloud provider's written permission for a public test
arrived on 14 October 2026). Its only target is the clone (`redteam-license.lake8.dev`) — never
`license.lake8.dev`. The [red-team rules](https://cryptin.lake8.dev/security/red-team-rules/)
carry the live status and the perimeter. **Production is never a
target.**

**By downloading this package you accept these rules and the perimeter they
define:** you connect to production only to run the product and you never attack
it; the only server you may ever attack is the clone. The
rules are ours and we keep them.

### Why the target is a clone and not the licence server — the honest reason

The clone runs **the same image as production** — same Dockerfile, same
entrypoint, same code, only the `.env` differs. Until 18 October 2026 it sat on
the same LAN as the production machine, which also serves other products and
holds signing keys that cannot be rotated: inviting attacks at it would have
meant inviting them onto that machine, so we did not. Since **18 October 2026**
it runs on a cloud server of its own (Hetzner, Falkenstein), with its own name
and TLS certificate, off that machine. We ran our acceptance tests on it, and
the red-team track opened against it on 29 October 2026 — and only against it.

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

Against the clone, this is the objective we most want someone to reach:

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

**You should see six `OK` lines.** If you see five, that is not a pass.

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

## What changed in beta-2026-10-30

The app only. The firmware, `firmwareirmware.factory.bin`, is still byte-identical to
`beta-2026-10-13` (sha256 `f66a17cd…94df9d`, version `90d0cd9`).

**New identity**, in the bottom row of the main window, gives your dongle a new seed —
new 24 words, new PIN — and moves your licence onto it. It is the cure for **G2**: the
seeds of the first beta were drawn before the hardware entropy source was on, and
recovering with your 24 words brings the same seed back. Done by hand, a new seed cost
you the licence; this keeps it.

1. With the dongle unlocked, the app asks the licence server to hold the licence for the
   move. Nothing changes yet; you have **7 days** for the rest.
2. Decrypt the files you still need. **The new identity cannot open anything the old one
   encrypted.**
3. The app reprograms the dongle over the UART cable. On its first start the dongle makes
   a new seed and shows you the new 24 words and the new PIN.
4. After a restart of the app, the new identity claims the licence.

- **Once per licence every 365 days**, separate from the three transfers. The expiry
  date does not change.
- **The old 24 words are retired:** they can no longer activate a licence. Keep them only
  while something you need is still encrypted with them — they rebuild the old identity,
  and decrypting needs no licence.
- **A registered backup dongle carries the old seed, so it stops counting.** Registering
  a new backup is free again this year.

The release notes say which parts we saw running and which only the server's tests cover.

---

## What changed in beta-2026-10-14

The app and the updater only. The firmware, `firmware\firmware.factory.bin`, is
byte-identical to `beta-2026-10-13` (sha256 `f66a17cd…94df9d`, version `90d0cd9`). These are the defects we met running the update from
`beta-2026-09-24` ourselves, on a beta unit, against production.

| where | before | now |
|---|---|---|
| Setup → *Program*, on a chip that is already a dongle | Asked before overwriting, without saying that an update is done elsewhere; yes gave **24 new words and no licence** | Says it: updating → answer **No**, use **Transfer license** |
| Transfer license → *Program* | Same question | Says that for an update **Yes** is right |
| Command-line updater, on a dongle | Refused, without saying what to do | Refuses and points to Transfer license |
| Setup, programming failed | Always "try another USB port" | The updater's own reason |
| Updater error messages | Seven still in Italian | English |
| Licence step, right after activation | A token refused by the dongle went unsaid if the server check lagged | Shown, in red |
| End of Transfer license | Expiry in local time | UTC, like the rest of the app |
| Recovery, step 1 | "Photo of the UART socket goes here" | A drawing: cable in the **UART** socket |

The release notes say which of these we saw running and which we only compiled.

**If the dongle seems to vanish after a reset and you run VMware Workstation:** its
*USB Arbitration Service* holds the device until you choose host or VM. Set
*Edit → Preferences → USB* to connect new devices to the host. Not a Crypt-in defect.

---

## What changed in beta-2026-10-13

Between 8 and 11 October 2026 the firmware went through seven red-team rounds. They
found the defects below in the firmware of `beta-2026-09-24`. **This release fixes
all of them.**

| id | severity | what it was |
|---|---|---|
| F1 | **High** | `CMD_CHALLENGE` (`0x03`): a second request arriving while a signature was being computed could replace the message half-way, so two signatures shared one Ed25519 nonce — and from that pair the identity signing key can be computed. Needs an unlocked session and a process on the host. The 24-word seed does not come out, but the key is enough to sign as the dongle. Fixed: the command is copied before it is processed, the signature nonce is hedged, and every signature is verified on the chip before it is sent. |
| H1 | **High** | A malformed USB control request made the dongle send back the last report any process had written — a `VERIFY_PIN` included, PIN in clear — and execute it again. On a beta unit it re-unlocked the session after auto-lock, without the PIN. Fixed in the USB stack. |
| G6 | Medium | Cutting power between the PIN comparison and the write of the attempt counter gave a free guess. Fixed: the attempt is counted and written before the comparison. |
| J1 | Medium | Some USB control requests sent with the wrong direction wrote host bytes past a one-byte buffer in the USB stack and crashed the dongle. Fixed. |
| G1 | Low | An over-long report length was truncated instead of rejected, and could make the dongle run the previous command again — possibly another process's. Fixed: rejected. |
| G2 | Medium | The identity seed was drawn at first boot before the hardware entropy source was enabled. We measured first draws across reset and real power cuts and found no repetition, but the source was not what the design required. Fixed: the seed now comes from the enabled entropy source, compressed with SHA-256. **A recovered dongle keeps its old seed** — see the note below. |
| K2 | Low | `VERIFY_PIN` had no rate limit beyond the three attempts. Now at most one attempt per second; an early one answers `ERR_PIN_TOO_SOON` (`0xE5`) and is not counted. |
| J4/K4 | Medium | Version 1 `.crin` files written by the chip are not standard AES-GCM past 27 bytes, and their header is not authenticated. Fixed by format version 2. Version 1 files still open. |
| L3 | Medium | On an unlocked session, a second process could produce a reply that a client would take for the verified end of someone else's decryption. Fixed: the decrypt close now carries a nonce from the host. |

**About G2 and recovery.** Recovering with your 24 words gives back the seed
generated by the old firmware. That is what lets you open your old files. If the
dongle will protect anything that matters, give it a new identity with **New
identity** in the app (since `beta-2026-10-30`): it keeps your licence. Do not make
one by hand — reprogramming the dongle and setting it up again leaves the licence on
the old words, and the new dongle cannot encrypt.

**Also new:**

- **`.crin` format version 2.** The dongle now writes only version 2: standard
  AES-GCM, with the 66-byte header authenticated. Version 1 files still open; nothing
  writes them any more. Spec: <https://cryptin.lake8.dev/docs/crin-spec/>.
- **Protocol:** `ENCRYPT_BLOCK_V2` (`0x0F`) and `DECRYPT_BLOCK_V2` (`0x10`) write
  and read format version 2. `ENCRYPT_BLOCK` (`0x0D`) now answers `ERR_DEPRECATED`;
  `DECRYPT_BLOCK` (`0x0E`) stays, for version 1 files. New status codes
  `ERR_PIN_TOO_SOON` (`0xE5`) and `ERR_SIGN_FAULT` (`0xF9`). Reference: <https://cryptin.lake8.dev/docs/protocol/>.
- **The app and the firmware go together.** The new app does not encrypt with the
  old firmware, and the old app does not encrypt with the new firmware.
- **Without the dongle, your files can still be recovered from the 24 words.** The
  tool that does it is not in this package yet.

**Still open, as before:** the session is not bound to the process (F-001-REV),
F-ISO, and the PIN travels in clear on the USB bus (F-003/B1). They are on the
security model page: <https://cryptin.lake8.dev/docs/security-model/>.

---

## Updating from beta-2026-09-24

Writing the new firmware reprograms the dongle from scratch — the image overwrites
the `nvs` partition, and with it the seed and `K_attestation`, the key the licence
server knows the chip by. So the update is done from the new app's **Transfer
license**: in one pass it writes the new firmware, puts your 24 words back on the
chip, and moves your beta licence onto the chip's new `K_attestation`.

1. **Have your 24 words in front of you.** They are the only way back to the files
   you encrypted with the old firmware. Your PIN does not change: it comes from the
   words.
2. Plug in both cables, the native USB and the UART. Open the **new** app and click
   **Transfer license**. For *What happened to the old dongle*, choose *It stopped
   working* — for an update it is the closest answer. Enter the 24 words, then
   **Program** on the UART port. The app asks before it overwrites a chip that is
   already a dongle: answer **Yes**.
3. When it says *Chip ready — one restart left*, close the app and open it again. It
   offers to finish the transfer: say yes and unlock with your PIN. If no PIN box
   appears, unplug the native USB cable for ten seconds and plug it back in.
4. The transfer uses one of the licence's three transfers a year. The expiry date
   does not change.

Files encrypted before the update stay in format version 1 and still open.
Everything you encrypt from now on is version 2.

> **Do not update with the command-line updater.** It refuses a chip that is
> already a dongle, and if you force it the chip starts over with a new identity:
> the app then shows you **24 new words**. They are not the ones that matter — your
> files and your licence belong to the old ones. If it has already happened, do
> step 2 anyway: the transfer puts your old words back.

We ran this path end to end on a beta unit, against production, before publishing:
three version 1 files (a binary, a text file, a folder) opened byte-identical after
the update, and a new file came out as version 2 and opened again.

There is still no update that keeps the chip's own `K_attestation`: that would mean
writing only the application, and that path does not exist yet.

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

- **`CryptinPersonal.exe` and `CryptinUpdater.exe`** are **GPL-3.0**. Their
  source is published, as a beta, in
  [`jaffa2970/cryptin-sdk-beta`](https://github.com/jaffa2970/cryptin-sdk-beta)
  (`src/CryptinPersonal/`, `src/CryptinUpdater/`). The updater is the **test
  build** described there: it verifies no signature. Our builds are not
  reproducible, so the source is the same but a rebuild will not match these
  binaries byte for byte. The `LICENSE` above does not apply to them.
- **The dongle firmware** (`firmware/firmware.factory.bin`) is **proprietary**
  and closed source. In this beta it is the `-dev` build and it is **not
  signed**: verify it by its hash in `SHA256SUMS.txt`, not by a signature.
- **`esptool.exe` v5.3.1** is **GPL-2.0**, unmodified, taken from Espressif's
  official release. It is invoked as a **separate process**, and its licence text
  travels inside the archive at `licenses/esptool-v5.3.1-GPL-2.0.txt`.
- **CryptinSDK** and the `cryptin` command-line client are **Apache-2.0**. They
  are **not in this download**; their source is published in the same
  repository (`src/CryptinSDK/`, `src/CryptinCli/`), each folder with its own
  `LICENSE` and `NOTICE`. No binary package of the SDK or the CLI has been
  released, and no date has been announced for one.

Full terms are on the site: <https://cryptin.lake8.dev/docs/license/>.

## What this beta does not promise

- ⛔ **No OTA.** Updating the firmware is a reprogram plus a licence transfer —
  see *Updating from beta-2026-09-24* above.
- ⛔ **Licence expiry is not enforced offline.** Nothing sets the dongle's clock,
  so offline a licence does not expire. Said plainly rather than discovered.
- ⛔ **The session is not bound to the process.** Auto-lock is done, the binding
  is not. It is already published among the open findings on the security model
  page, and it stays there.
- ⛔ **Beta units are not production-class**, and a beta dongle does not become a
  production dongle: the eFuses and the signing key both change at that boundary.
