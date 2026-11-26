# Inspur SA5212H5 — BIOS & BMC Firmware Archive

**Canonical repo:** [alphingj/SA5212H5](https://github.com/alphingj/SA5212H5) · [Issues](https://github.com/alphingj/SA5212H5/issues) · [Releases](https://github.com/alphingj/SA5212H5/releases)

Firmware preservation archive for the **Inspur SA5212H5** 2U rack server.
No public download for these exact builds was found (searched 2026-11-26) — this repo exists so owners can still recover / reflash the machine.

## Download

```bash
git clone https://github.com/alphingj/SA5212H5.git
cd SA5212H5
sha256sum -c SHA256SUMS.txt
```

Or download the `.bin` files / `SA5212H5.zip` from [Releases](https://github.com/alphingj/SA5212H5/releases).

> **⚠️ Flash at your own risk.** Wrong board revision = brick. Verify checksum first, keep a known-good SPI programmer + backup of your current ROM before flashing. See [docs/FLASHING.md](docs/FLASHING.md).

## Hardware

| Item | Value |
|---|---|
| System | Inspur SA5212H5, 2U rack, dual-socket |
| Board OEM / platform | Inventec `NeonCity` (Purley) — strings: `NeonCityEPECB`, `NeonCityEPRP`, `PurleyLBGEPDVP` |
| Chipset | Intel Lewisburg (C620 series), Purley platform |
| CPU | Intel Xeon Scalable 1st/2nd Gen (Skylake / Cascade Lake) |
| BIOS vendor | AMI (American Megatrends Inc.), UEFI, `RELEASE_VS2015` build tree |
| BMC | ASPEED AST2500, AMI MegaRAC (`ast2500evb_ami`, `Linux-3.14.17-ami`) |
| Variant tag | `Ali` — Alibaba-customised build (filename suffix) |

## Files

| File | Type | Version / Build | Date (in name) | Size | SHA-256 |
|---|---|---|---|---|---|
| `SA5212H5/SA5212H5_BIOS_PLIPP03100_Ali_20220513.bin` | BIOS — full 32 MiB SPI image (IFD + ME + GbE + BIOS) | `PLIPP03100`, AMI build 05/23/2022 | 2022-05-13 | 33,554,432 B | `3f35b8897dafcdc6b72f7622ac5afb30a4ef437af66c06a5be91bf7100ca0506` |
| `SA5212H5/SA5212H5_BMC_Ali_5.52.11_20220328.bin` | BMC — full 32 MiB flash image | `5.52.11` (`UFW_VERSION=5.52.11`, `FW_DESC=RR11-5 AST2500 EMMC KERN3 UPDATE BUILD 1`) | 2022-03-28 | 33,554,432 B | `92aa1e99582da627c8c3d68e274e53be2de5531093583ca6ef276724e70f5728` |
| `SA5212H5.zip` | Zip of the two files above (duplicate — prefer GitHub Release) | — | — | 37,133,226 B | `b135ac20f517540345330ee8b06f9a7acd0b797c6c82fa8eb5fe7ea7a7966818` |

MD5 (legacy, for tools that ask for it):

```
75e6ee262d139ed3a0dfdede2bbd2570  SA5212H5_BIOS_PLIPP03100_Ali_20220513.bin
11b0b6cbe42ed4ea203f16d142b76fba  SA5212H5_BMC_Ali_5.52.11_20220328.bin
```

Full analysis: [docs/ANALYSIS.md](docs/ANALYSIS.md) · Flashing: [docs/FLASHING.md](docs/FLASHING.md) · SPI programmers (CH341A/B, T48, RT809F, Dediprog): [docs/PROGRAMMERS.md](docs/PROGRAMMERS.md) · Checksums: [SHA256SUMS.txt](SHA256SUMS.txt)

## Verify

```bash
sha256sum -c SHA256SUMS.txt
# or, from repo root after cloning:
cd SA5212H5 && sha256sum -c ../SHA256SUMS.txt
```

## Compatibility — read before flashing

- Confirmed only for **SA5212H5** with the `Ali` build. Other SA5212 variants (e.g. SA5212M5), other regions, or non-Ali boards may need different images — do not assume cross-compat.
- BIOS image is a **full SPI dump layout** (Intel Flash Descriptor at `0x10`, signature `5AA5F00F`), not a capsule-only update. Flash method must match: BMC web/IPMI update vs external SPI programmer are different paths — see flashing guide.
- BMC image targets **AST2500** only.

## Provenance & licence

- Source: user-owned hardware dump / vendor-supplied files, archived as-is for preservation and right-to-repair.
- BIOS/BMC firmware remains © Inspur / Inventec / AMI / Intel and their licensors. No licence is granted by this archive beyond personal repair use on hardware you own. No warranty.
- If you are the rights holder and want this taken down or corrected, open an [Issue](https://github.com/alphingj/SA5212H5/issues).

## Help wanted

Owners: please open an [Issue](https://github.com/alphingj/SA5212H5/issues) / PR with board photos, `dmidecode` output, current BIOS/BMC versions, and whether flashing worked (method used). That turns this dump into a verified recovery resource.

## Repo notes

Canonical repo: <https://github.com/alphingj/SA5212H5>. GitHub git limit is 100 MB/file — both `.bin` files (32 MiB) fit in git, but prefer **GitHub Releases** for binaries and keep the repo light. `SA5212H5.zip` duplicates the `.bin` files; attach it to a Release instead of committing it twice.
