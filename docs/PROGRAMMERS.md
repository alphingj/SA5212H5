# SPI Programmers — CH341A / CH341B & Alternatives

For external (out-of-system) flashing of the SA5212H5 BIOS + BMC chips.
Both images = **32 MiB (256 Mbit)** → chip is a 25-series SPI NOR, likely Winbond `W25Q256` / Macronix `MX25L256` / GigaDevice `GD25Q256` class. Confirm silkscreen on your board before buying / clipping.

> **Warning:** In-circuit flashing can damage board + programmer if voltage or orientation is wrong. Server fully OFF, PSUs unplugged, batteries removed. Work ESD-safe. Read § Safety first.

## 1. Which programmer?

| Tool | Price tier | SA5212H5 fit | Software | Notes |
|---|---|---|---|---|
| **CH341A** (black / green / gold) | $ | OK if 32 MiB supported + voltage fixed | flashrom, NeoProgrammer, AsProgrammer, SNANDer | Cheapest. Many clones have 5 V bug — see §2 |
| **CH341B** | $ | OK, same conditions | Same as CH341A | Newer WCH chip revision, functionally identical (USB→SPI/I²C/UART, same datasheet). Some CH341B boards ship native 3.3 V logic — verify, don't assume |
| **CH347** | $$ | Good, faster USB-HS | flashrom (newer), vendor tool | Newer WCH bridge, higher SPI speed. Check flashrom version supports it |
| **EZP2019 / EZP2023+** | $ | OK | Vendor Windows tool | Simple, chip database smaller. Verify 256 Mbit part listed |
| **SNANDer hardware / CH341 + SNANDer SW** | $ | OK | SNANDer | Good for SPI NAND/NOR, handles 4-byte addressing well |
| **XGecu T48 (TL866-3G)** / TL866II Plus | $$ | **Best hobby/pro pick** | Xgpro (Windows) | Successor to TL866II Plus, 40-pin ZIF, 31k–34k+ devices, selectable VCC, ISP header. Fast, reliable verify |
| **XGecu T56** | $$$ | Best, adds NAND/eMMC | Xgpro | Overkill unless you also do NAND/eMMC |
| **RT809F / RT809H** | $$–$$$ | Good | RT809F/H tool (Windows) | Popular board-repair tool, SOIC clip ISP flashing, wide BIOS chip database. RT809H adds NAND/eMMC |
| **Dediprog SF100 / SF600 Plus II** | $$$$ | **Pro / lab pick** | Dediprog SW, flashrom (`dediprog`) | Industrial ISP programmer, correct voltages, excellent reliability. Expensive |
| **Bus Pirate v3/v4** | $$ | Slow, works | flashrom (`buspirate_spi`) | Very slow for 32 MiB. Backup only in a pinch |
| **Raspberry Pi / Linux SPI board** | $$ | Works | flashrom (`linux_spi`) | Needs level care (Pi = 3.3 V only), manual wiring |
| **serprog (Arduino / STM32 / rp2040)** | $ | Works | flashrom (`serprog`) | DIY. Speed + wiring quality vary |

**Recommendation for this server:** CH341A/B (voltage-verified) for one-off recovery; XGecu T48 or RT809F if you flash often; Dediprog SF600 if this is lab/bench work.

### CH341A vs CH341B — what actually differs

- Same function: WCH USB bridge → SPI + I²C + UART. Same supported 24xx (I²C EEPROM) + 25xx (SPI flash) families. Same software (flashrom `-p ch341a_spi` drives both).
- Difference is chip/board revision, not capability. A/B suffix alone tells you **nothing** about output voltage — board layout decides that. Measure yours (§2).
- Buying tip: prefer kits sold as "CH341A/B + SOIC8 clip + 1.8 V adapter + SOIC8→DIP adapter" from a seller who states 3.3 V support. Avoid bare boards with no regulator markings.

## 2. Safety — voltage kills chips

Server SPI NOR is almost always **3.3 V** (some newer boards 1.8 V — read the chip datasheet for the exact part on your board; do not guess).

Known CH341A trap:

- Classic black CH341A mini-programmer powers the CH341 at 5 V, so SPI data lines (MOSI/SCK/CS) idle near 5 V while the ZIF VCC is 3.3 V. Out-of-spec for 3.3 V flash; can damage flash + connected PCH/BMC logic over time.
- Fix (hardware mod): lift CH341 pin 28 (VCC), feed it 3.3 V from the board regulator. Many guides + pre-modded / newer boards exist — a modded or native-3.3 V board is strongly preferred over an unmodded black board.
- CH341B boards: *some* are 3.3 V from production, some are not. Same test applies.

Before clipping any chip:

```text
1. Programmer plugged into USB, no chip/clip attached.
2. Multimeter: VCC pin of ZIF/clip header → must read chip voltage (3.3 V, or 1.8 V with adapter).
3. If VCC = 5 V and chip = 3.3 V/1.8 V → STOP. Mod, switch jumper if fitted, or use another programmer.
```

- **1.8 V chips:** insert the 1.8 V adapter board between programmer and clip/chip. Never flash 1.8 V part at 3.3 V/5 V.
- **Never** connect programmer VCC to a board that is also powered. One power source only.
- Pin 1 = dot / dimple / bevel on chip, red stripe / marked leg on clip cable. Reversed clip = no detect, possible damage. Double-check against chip datasheet pinout (pin 1 = CS# on most 25-series SOIC).

## 3. What you need

```text
[ ] Programmer (CH341A/B, T48, RT809F, Dediprog, ...)
[ ] SOIC8 test clip + cable   (if chip is SOIC-8)
[ ] SOIC16 test clip          (if chip is SOIC-16 — check your board; 256 Mbit parts come in both)
[ ] 1.8 V adapter             (only if chip datasheet says 1.8 V)
[ ] SOIC→DIP adapter board     (if you desolder the chip)
[ ] PC (Linux preferred for flashrom) + USB cable
[ ] Multimeter, tweezers, ESD strap, good light / magnifier
[ ] Optional: hot-air / soldering station if in-circuit read fails and chip must come off
```

Package check: 32 MiB BIOS/BMC chips on Purley-era boards are commonly SOIC-8, SOIC-16, or WSON-8. **WSON-8 has no legs — a SOIC clip won't grip.** WSON needs a probe adapter, pogo jig, or desoldering.

## 4. Identify BIOS vs BMC chip — do not mix

- SA5212H5 has (at least) two 32 MiB flashes: host **BIOS** (near PCH/CPU) and **BMC** (near ASPEED AST2500 SoC — small square ARM chip, usually near board edge / VGA / dedicated BMC LAN).
- Label each chip (photo + marker) before touching. Flashing BIOS image to BMC chip (or reverse) = both dead until re-flashed correctly.
- Record each chip's exact part number + voltage from its datasheet.

## 5. Universal procedure (any programmer)

```text
1. Server OFF, both PSUs unplugged, hold power button 5 s to drain, wait 1 min.
2. Photo board. Locate BIOS chip, then BMC chip. Note pin-1 dot each.
3. Attach clip: pin 1 clip ↔ pin 1 chip, firm even pressure, all pins contacted.
   Connect clip cable to programmer (correct header / adapter, pin 1 aligned).
4. Plug programmer into PC. Open software. Select exact chip part (or Auto-Detect).
   - No detect / ID = 0xFFFFFF / 0x000000 → reseat clip, check pin 1, check voltage, shorten cables. Do NOT erase.
5. READ current chip → backup1.bin. READ again → backup2.bin. Compare:
       sha256sum backup1.bin backup2.bin   (must be identical, size = 33,554,432 B)
   Different hashes = bad contact. Reseat, retry. Never proceed on unstable read.
6. Keep backups OFF the server, named with board + chip + date. These are your unbrick path.
7. Verify target file:  sha256sum -c SHA256SUMS.txt
8. ERASE → WRITE (target .bin) → VERIFY (programmer verify pass required).
9. READ back → readback.bin.  sha256sum readback.bin must equal target SHA-256 in README.
10. Unclip, reassemble, power on. First boot may take minutes. Enter setup, load defaults, check version.
```

BIOS target SHA-256: `3f35b8897dafcdc6b72f7622ac5afb30a4ef437af66c06a5be91bf7100ca0506`
BMC target SHA-256: `92aa1e99582da627c8c3d68e274e53be2de5531093583ca6ef276724e70f5728`

## 6. CH341A/B step-by-step

### Linux — flashrom (preferred)

```bash
# Install: Debian/Ubuntu `sudo apt install flashrom` · Arch `sudo pacman -S flashrom` · Fedora `sudo dnf install flashrom`
# CH341B uses same driver name:

sudo flashrom -p ch341a_spi -L | grep -i ch341   # programmer present

# 1. Probe (read chip ID only)
sudo flashrom -p ch341a_spi

# 2. Backup — twice, compare
sudo flashrom -p ch341a_spi -r backup1.bin
sudo flashrom -p ch341a_spi -r backup2.bin
sha256sum backup1.bin backup2.bin
cmp backup1.bin backup2.bin && echo IDENTICAL

# 3. Flash BIOS (example). -w writes + auto-verifies; add -V for verbose.
sudo flashrom -p ch341a_spi -w SA5212H5/SA5212H5_BIOS_PLIPP03100_Ali_20220513.bin

# If chip not auto-matched, force exact part (find name first):
#   sudo flashrom -p ch341a_spi --list-supported | grep -i 256
#   sudo flashrom -p ch341a_spi -c "W25Q256.V" -w <file>

# 4. Read-back check
sudo flashrom -p ch341a_spi -r readback.bin
sha256sum readback.bin
```

Notes:

- 32 MiB needs **4-byte addressing**. Current flashrom handles this; ancient Windows CH341 tools (v1.18–1.34) may truncate at 16 MiB or fail on 256 Mbit parts. If file size written ≠ 33,554,432 B, tool is too old — switch to flashrom / NeoProgrammer.
- Slow reads/writes (minutes for 32 MiB over CH341) = normal. Don't unplug mid-write.
- `No EEPROM/flash device found` / garbled ID → clip contact, pin 1, or board loading the bus (see §8).

### Windows

| Software | Use |
|---|---|
| **NeoProgrammer** (2.2.x) | Recommended free tool. Auto-detect, supports CH341A/B + 256 Mbit parts, read/write/verify |
| **AsProgrammer** | Good alternative, CH341A support, chip database editable |
| **SNANDer** | Solid for SPI NOR/NAND via CH341 |
| Vendor "CH341A Programmer" 1.29/1.34 | Legacy. Avoid for 32 MiB parts unless chip explicitly listed and size verified |

Driver: CH341 USB driver (WCH `CH341SER` / parallel driver) must be installed before the app sees the device. Flow in all apps: **Detect → Read (×2, save) → Erase → Blank check → Open .bin → Write → Verify.**

## 7. Other tools — quick start

| Tool | Flow |
|---|---|
| **XGecu T48 / TL866II** | Xgpro → Select IC (exact 25-series part) → ISP or ZIF mode → Read ×2 → Load file → Program → Verify. Set VCC to chip voltage in software |
| **RT809F/H** | RT809 tool → Smart Identify → Read ×2 → Open .bin → Write → Verify. Use ISP cable + clip for in-circuit |
| **Dediprog SF100/SF600** | Dediprog SW or `flashrom -p dediprog` → same read×2 / write / verify flow. Set target voltage in SW |
| **Bus Pirate** | `flashrom -p buspirate_spi:dev=/dev/ttyUSB0,spispeed=1M` → same flow. Expect long 32 MiB times |
| **Raspberry Pi** | Enable SPI, wire MOSI/MISO/SCK/CS/GND/3.3V, `flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=1000`. Pi GPIO = 3.3 V only |
| **serprog (Arduino/STM32)** | Flash serprog firmware, `flashrom -p serprog:dev=/dev/ttyACM0:115200` |

## 8. In-circuit failure modes

| Symptom | Cause / fix |
|---|---|
| ID reads `FFFFFF` / `000000`, no detect | Clip reversed, poor contact, wrong voltage, or chip not powered. Reseat, check pin 1 + VCC with meter |
| Reads differ every time | Clip contact marginal, cable too long, board circuitry driving bus. Clean pins, firmer clip, shorter leads, retry slower SPI speed if SW allows |
| Read OK but small / truncated | Tool lacks 256 Mbit / 4-byte support. Use flashrom / NeoProgrammer / Xgpro current version |
| Programmer powers board LEDs / fans twitch | Board back-feeding from programmer VCC. Stop. Some boards cannot be flashed in-circuit — desolder chip or use board's SPI header if present |
| Write verify fails at fixed offset | Bad clip contact during write, or write-protected (WP# / SR). Reseat, check chip status register, retry. Desolder if persistent |
| Dead after flash, readback hash correct | Wrong chip (BIOS↔BMC swapped), wrong board revision, or board-unique data overwritten. Reflash your backup, confirm target/chip pairing |

Desolder path: hot-air, remove chip, solder to SOIC→DIP adapter, flash in ZIF socket (most reliable reads), solder back with pin 1 correct. Practice on scrap board first.

## 9. After flashing

1. Read-back hash = target SHA-256 (§5 step 9). No hash match = not done.
2. BIOS: first POST slow, load setup defaults, set BMC network / boot order.
3. BMC: wait 5–10 min for BMC boot, browse to BMC IP, confirm `5.52.11`.
4. Report programmer + software + clip (in-circuit vs desoldered) + result in an [Issue](https://github.com/alphingj/SA5212H5/issues).

## Sources

Tool capabilities cross-checked against flashrom programmer list (`ch341a_spi`, `dediprog`, `buspirate_spi`, `serprog`), XGecu T48 (TL866-3G) specs, Win-Raid CH341A guide, and CH341A 5 V-mod documentation (EEVBlog / Libreboot). Board-specific chip part, package, and voltage must still be confirmed on your physical SA5212H5 — no SA5212H5 service manual with flash location was found publicly.

---
Canonical repo: <https://github.com/alphingj/SA5212H5>
