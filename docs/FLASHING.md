# Flashing Guide — SA5212H5

> **Warning:** Firmware flashing can permanently brick the server. Do every step. If unsure, stop and ask in an [Issue](https://github.com/alphingj/SA5212H5/issues) with your board version first.

## Before you flash

1. Confirm model: chassis label + `dmidecode -t system` / BIOS setup screen says **SA5212H5**.
2. Record current BIOS/BMC versions (BIOS setup + BMC web UI → Firmware Information).
3. Verify download:
   ```bash
   sha256sum -c SHA256SUMS.txt
   ```
   Mismatch = do not flash.
4. Backup current firmware:
   - BIOS: BMC/IPMI backup if offered, else SPI programmer dump (32 MiB).
   - BMC: BMC web UI backup / `dd` of MTD partitions if you have SSH.
5. Stable power (UPS), no other updates running, console access (BMC KVM) ready.

## BIOS — PLIPP03100

Image layout: full 32 MiB SPI (IFD + ME + GbE + BIOS). Two paths:

- **Preferred — vendor/BMC path:** BMC web UI or IPMI firmware update, *only if* the UI accepts this `.bin`. If it expects a capsule/zip, do not force the raw SPI image.
- **Recovery — external programmer:** CH341A / CH341B, XGecu T48, RT809F, Dediprog, etc. + SOIC clip on the BIOS SPI chip, flash the full `.bin`, verify read-back hash matches. Full tool comparison, voltage safety, and step-by-step: [docs/PROGRAMMERS.md](PROGRAMMERS.md). Erases board-unique data if present in GbE/NVRAM — restore from your backup.

Do not interrupt power. First boot after flash can take several minutes (memory training).

## BMC — 5.52.11 (AST2500)

- **Preferred:** BMC web UI → Maintenance / Firmware Update → select `SA5212H5_BMC_Ali_5.52.11_20220328.bin`, preserve config if offered, wait for auto-reboot, verify version shows `5.52.11`.
- **Recovery:** U-Boot / SPI programmer on the BMC flash (near the AST2500 chip — not the BIOS chip; see [PROGRAMMERS.md §4](PROGRAMMERS.md)), only if you know the AST2500 layout. `FW_DESC` = `RR11-5 AST2500 EMMC KERN3 UPDATE BUILD 1`.
- BMC will be unreachable during update (~5–10 min). Do not power-cycle mid-flash.

## After flashing

1. Check versions in BIOS setup + BMC UI match expected.
2. Load BIOS defaults, re-apply your settings, reset BMC network if config was not preserved.
3. Report result (success/fail + method) in an [Issue](https://github.com/alphingj/SA5212H5/issues) — helps the next owner.

## Bricked?

- BIOS: reflash your backup with an SPI programmer.
- BMC: SPI recovery or vendor recovery procedure for AST2500.
- No backup = try the images in this repo with a programmer, then seek vendor support.

---
Canonical repo: <https://github.com/alphingj/SA5212H5>
