# Venue Amp: hardware concept (mainboard Rev A)

Version 0.1.0 · 2026-11-20

This is the list of main parts a manufacturer would declare in a hardware bill of materials (HBOM).
It is not a schematic. The parts are real, common types, so that manufacturer plus part number is a
valid identity. The board itself is fictional.

## Verification

The parts were checked on 2026-11-20 against manufacturer pages, distributors and the NVD API. The column
"Verified by" says how reliable an entry is:

- **manufacturer**: read on the manufacturer's own page
- **distributor**: seen at distributors only
- **open**: not confirmed

Pages of STMicroelectronics and Microchip could not be retrieved, so first-hand confirmation is
missing there.

## Parts

| Function | Part | Order code | Role | Security-relevant characteristics | Verified by |
|---|---|---|---|---|---|
| Microcontroller | STMicroelectronics STM32H743 | STM32H743ZIT6 | runs the firmware | Arm Cortex-M7 with memory protection unit, built-in Ethernet MAC | distributor and page excerpts (active); read-out protection levels and debug interface not confirmed from an ST source |
| External memory | Winbond W25Q64JV | W25Q64JVSSIQ | second firmware image for updates | external serial flash that can be read out | distributor (active) |
| Network | Microchip LAN8742A | LAN8742AI-CZ-TR (industrial) | control and audio | one Ethernet interface | distributor (active) |
| Signal processor | Analog Devices ADAU1452 | ADAU1452WBCPZ | sound processing | programmable; loads its program from a serial EEPROM or from the microcontroller | manufacturer (in production) |
| Program memory of the signal processor | serial EEPROM | type open | program of the signal processor | second place holding executable content | open |
| Converter | digital-to-analogue converter | type open | between signal processor and power stage | | open |
| Power stage | Texas Instruments TPA3255 | TPA3255DDVR | amplification | analogue inputs, no software inside | manufacturer (active) |
| Serial interface | Texas Instruments THVD1450 | THVD1450DR | legacy systems, remote control | serial field interface (RS-485) | manufacturer (active) |
| Service port | SWD header | | development, production | debug access populated | no part number |

There is no secure element on this board. That is intended: it is a typical finding a device threat
analysis should bring out.

## Vulnerability records for hardware

The NVD has no CPE for the STM32H743 itself, only for the family (`cpe:2.3:h:st:stm32h7:-:*:*:*:*:*:*:*`).
CVE-2019-14236 (bypass of the code read-out protection PCROP on several STM32 families) is recorded
for it. The part number from an HBOM therefore does not hit the entry directly. No CPE was found for
the flash and the signal processor. The Ethernet PHY and the parts from Texas Instruments were not
searched.

## What an HBOM can and cannot say

An HBOM states which part is fitted. It does not state how the part is configured in the shipped
device. Ground truth for this board: the SWD header is populated and the debug port is **left open**
in production units; read-out protection is not activated. A threat analysis based on the HBOM alone
proposes the same "debug port present" as for the controller, where the port is locked. The
difference only shows when a person assesses it.

## Planned files

- `hbom/venue-amp-rev-a.cdx.json`: as received from the manufacturing partner (import case)
