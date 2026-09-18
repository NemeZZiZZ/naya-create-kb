# Exhibit inventory (read as text)

FCC internal/external photos and report text, re-read on the second pass
for **silkscreen, chip markings, test points and labels** — not for
hero shots. Source files: `research/fcc/` in the companion repo.

## Boards (per-PCB silkscreen revisions)

| Board | Silkscreen | Where seen |
|---|---|---|
| Half mainboard | `Create_L_KB 20250220_V11` (bottom + top) | fullres p6-Im1/p6-Im2 |
| Wing board | `Create_L_Wing 20250206_V11` | fullres p9-Im1 |
| Dongle | `BOK17_Dongle 20241106 V01` | DG p0/p1 |
| Tune module ring | `Tune_Touch_20240926_V00` | CRL-view-4 |
| Track module ring | `Y08_06025_20250227_V00` | CRL-view-5 |
| Module MCU board | `Touch_MB 20250227 V10` (carries STM32F411CEU6) | hardware.md |

## SoCs (confirmed on die)

| Part | Marking | Role |
|---|---|---|
| nRF52811 | `N52811 / CKAAD0 / 2301ME` | halves |
| nRF52840 | `N52840 / CKAAD0 / 2301ME` | dongle |
| STM32F411CEU6 | readable on ring board | module MCU (Tune/Track) |

## Batteries (label text)

| Pack | Label | Where |
|---|---|---|
| Half buffer cell | `FH301217 3.7 V 50 mAh 0.185 Wh` | CRR/CRL teardowns |
| Tune module | `FH 202030 3.7 V 1000 mAh 3.7 Wh` (20250603) | CRL-view-4 |
| Track module | `FH364046 3.7 V 700 mAh 2.59 Wh` (20250611) | CRL-view-5 |
| Smallest puck | `ICR … 300 mAh 3.7 V 1.11 Wh` (cylindrical) | CRL-view-6 |

## Test points (labeled)

- **Half mainboard USB-C tail:** `SWDIO / SWDCLK / TP1 / TP7` + `D+/D-` +
  ANT keepout — the halves break out SWD at the tail (fullres p4-Im2).
- **Dongle back:** `SWDIO / TP1 / TP2` (DG p1-Im2).

## Antennas

| Part | Type | Gain |
|---|---|---|
| RF0400A (halves) | PCB | 0.8 dBi (CRL BLE test report) |
| RF0401A (dongle) | PCB | 0.91 / 2.29 / 3.70 dBi @ 2400/2450/2500 MHz, eff. 17.8→31.6 % (DG antenna spec, EMQuest EMQ-100, BTL anechoic, tested 2025-07-14) |

## Report metadata

- BLE test report no. `BTL-FCCP-1-2506C291`, GFSK, PHY 1M/2M/125 k coded.
- Test software: `nrfconnect-setup-5.1.0`.
- Grant issued 2025-08-29, lab BTL Inc (Dongguan).

## Corrections this pass

- Half mainboard photo shows **V11** silkscreen (20250220); the earlier
  KB table said `V13`. The live unit's rev may differ from the filing
  rev — recorded as the filing's rev here, with the table updated to
  note the V11 exhibit.
- Module MCU (**STM32F411CEU6**) was already correct in the hardware
  table — the second pass confirms it on die and adds the per-module
  ring board names (Tune/Track).
