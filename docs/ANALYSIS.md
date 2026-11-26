# Firmware Analysis — SA5212H5

Method: `file`, `xxd`, `strings`, SHA-256/MD5, Intel IFD parse. No execution, no modification of binaries. Date: 2026-11-26.

## Inventory

```
SA5212H5/
  SA5212H5_BIOS_PLIPP03100_Ali_20220513.bin   33,554,432 B (32 MiB exactly)
  SA5212H5_BMC_Ali_5.52.11_20220328.bin       33,554,432 B (32 MiB exactly)
SA5212H5.zip                                  37,133,226 B — contains only the two files above
```

Both `.bin` = exactly 32 MiB → full flash-chip images, not partial updates.

## BIOS image

- `file`: `Intel serial flash for PCH ROM`
- IFD signature at `0x10`: `5aa5f00f` ✓
- IFD regions (FLREG, base/limit):

| Region | Range | Size |
|---|---|---|
| GbE | `0x001000–0x002FFF` | 8 KiB |
| ME | `0x003000–0xA25FFF` | ~10.1 MiB |
| BIOS | `0x1000000–0x1FFFFFF` | 16 MiB |
| (upper gap / reserved) | rest | — |

- UEFI: 8× `_FVH` firmware volumes, AMI Aptio (American Megatrends Inc.), build date string `05/23/2022`
- Platform strings: `NeonCity`, `NeonCityEPECB`, `NeonCityEPRP`, `NeonCityFPGA`, `PurleyLBGEPDVP`, `Inventec`, SKUs `1U`, `2U`, `2U Non_QAT`, `Taishan`, `Shieldon`
- Build tree leaked in strings: `z:\code\Build\NeonCity\RELEASE_VS2015\...\PurleySktPkg\...`, `alipurleycrbpkg\...\aliplatformpeiinit`
- PXE: Intel Boot Agent 40G v1.1.09, PXE 2.1 Build 092
- Conclusion: AMI UEFI for Inventec Purley/Lewisburg board, Alibaba (`Ali`) CRB variant. Filename version `PLIPP03100`, file date 2022-05-13, internal build 2022-05-23.

## BMC image

- `file`: `data` — header is ARM exception vectors (`14f09fe5` ×7), not x86
- Bootloader: `U-Boot 2013.07 (Mar 25 2022 - 17:47:38)`, prompt `AST2500EVB>`
- SoC: ASPEED AST2500 (`ast2500evb_ami`, `ast_spiflash`, `ast_eth`)
- Stack: AMI MegaRAC, `Linux-3.14.17-ami`, `UFW_VERSION=5.52.11`, `FW_DESC=RR11-5 AST2500 EMMC KERN3 UPDATE BUILD 1`
- Toolchain string: `arm-none-linux-gnueabi-gcc (Sourcery CodeBench Lite 2011.09-70) 4.6.1`
- Conclusion: AMI MegaRAC BMC firmware 5.52.11 for AST2500, eMMC/KERN3 layout, built ~2022-03-25/28.

## External research

- `SA5212H5` = Inspur 2U Intel Xeon Scalable rack server (public listings confirm model/form factor only).
- No public hit for `PLIPP03100`, BIOS 2022-05-13 `Ali` build, or BMC `5.52.11` — supports the "no public link" premise. Vendor support pages for this model do not surface these builds.
- Caution: near-name `SA5212M5` parts (PSUs etc.) are a different generation — not evidence of firmware compat.

## Risks / unknowns

- Board revision / SKU (1U vs 2U, QAT vs Non-QAT) not read from a physical board — image auto-detects SKU at runtime per strings, but flashing the wrong system still risks brick.
- No release notes, no changelog, no vendor signature check performed.
- MAC / serial / FRU: full dumps may contain device-unique data in GbE/NVRAM regions. Check before redistributing a dump taken from *your* board (these files appear to be vendor images, but verify).

---
Canonical repo: <https://github.com/alphingj/SA5212H5>
