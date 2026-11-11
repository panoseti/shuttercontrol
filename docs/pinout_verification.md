# Pinout Verification

Date: 2026-11-11

All polarized and multi-pin parts in `shuttercontrol.kicad_sch` were checked against manufacturer datasheets. **No pin-number errors were found.** No design files were changed as a result of this check.

## Method

1. Exported the schematic netlist (`kicad-cli sch export netlist`) and listed every part as pin number → pin name → net. For dual-gate parts, also checked which inputs belong to which output in the symbol units.
2. Compared each pin table with the "Pin Configuration and Functions" section of the datasheet for the exact package in the BOM.
3. Cross-checked the PCB against the schematic: every pad's net in `shuttercontrol.kicad_pcb` matches the schematic netlist (0 mismatches), and every schematic part is on the PCB. Checked pad numbering of the footprints that matter (TO-252 tab, SOT-89 tab, SOT-23-3, TL3342, 3314G, polarized capacitors).

Resistors and non-polarized capacitors were not checked. Connector pin assignments (J1–J6) are design choices, not datasheet facts, and are out of scope.

## Verified against the manufacturer datasheet

| Ref | Part / package | Datasheet pinout | Result |
|---|---|---|---|
| U8 | TI DRV8251ADDAR, DDA (HSOP-8) | 1 IPROPI, 2 IN2, 3 IN1, 4 VREF, 5 VM, 6 OUT1, 7 GND, 8 OUT2, PAD = GND | Match |
| U12 | TI TLVH432AIDBZR, DBZ (SOT-23-3) | 1 CATHODE, 2 REF, 3 ANODE | Match |
| U1, U2, U9 | TI TLV9031DBVR, DBV (SOT-23-5) | 1 OUT, 2 V−, 3 IN+, 4 IN−, 5 V+; push-pull output | Match |
| U3 | TI SN74LVC1G17DBVR, DBV | 1 NC, 2 A, 3 GND, 4 Y, 5 VCC | Match |
| U5 | TI SN74LVC1G32DBVR, DBV | 1 A, 2 B, 3 GND, 4 Y, 5 VCC | Match |
| U6 | TI SN74LVC2G04DBVR, DBV (SOT-23-6) | 1 1A, 2 GND, 3 2A, 4 2Y, 5 VCC, 6 1Y | Match, including input/output pairing (1→6, 3→4) |
| U10 | onsemi LM317MABDTRKG, DPAK | 1 Adjust, 2 Vout, 3 Vin; tab connected to pin 2 | Match; footprint tab pad is pad 2 |
| RV1, R27 | Bourns 3314G-1-105E / 3314G-1-201E | Terminal 2 = wiper; 1 and 3 = ends (1 = CCW, 3 = CW) | Match; footprint pad 2 is the single pad on its side |
| S1, S2 | E-Switch TL3342F450QG | Terminals 1–2 shorted, 3–4 shorted; pressing connects the two pairs | Match; footprint numbers each pair as one pad number; pad size and spacing (6.3 × 3.8 mm centers) match the recommended layout |

## Verified against secondary sources

The manufacturer PDF could not be downloaded for these parts (Nexperia returned HTTP 403, ST timed out). The pinouts were checked against the sources listed. To fully close these out, compare with the original Nexperia and ST datasheets.

| Ref | Part / package | Source used | Pinout | Result |
|---|---|---|---|---|
| U4, U7 | Nexperia 74LVC2G08DC,125 (VSSOP-8) | TI SN74LVC2G08, DCU package (same industry pinout) | 1 1A, 2 1B, 3 2Y, 4 GND, 5 2A, 6 2B, 7 1Y, 8 VCC | Match, including gate pairing (1,2→7 and 5,6→3) |
| U13, U14 | Nexperia 74LVC1G157GW,125 (SOT-363) | Mirrored copies of the Nexperia datasheet; Nexperia 74LVC1G157-Q100 datasheet | 1 I1, 2 GND, 3 I0, 4 Y, 5 VCC, 6 S; S low → Y = I0, S high → Y = I1 | Match. With S2 released, MAN_EN is pulled high and selects I1 (AUTO), consistent with the silkscreen "hold = MANUAL" |
| U11 | ST L78L33ABUTR (SOT-89) | Excerpt of the ST datasheet (elcodis); two third-party 78L33 SOT-89 datasheets on LCSC | 1 OUTPUT, 2 GND (tab), 3 INPUT | Match; custom footprint `SOT-89-3_RectPads` has pad 1 and pad 3 on the outer leads and pad 2 on the center lead and tab |

## Diodes and polarized capacitors

| Ref | Part | Check | Result |
|---|---|---|---|
| D1 | onsemi MBRS240LT3G (SMB) | Anode on V12, cathode on V12_D; footprint pad 1 = cathode (band) | Correct |
| D2, D3 | 1N4148W (SOD-123) | Anodes on V3_D2/V3_D3, cathodes on V3IN | Correct |
| D4, D5 | 1N4004W (SOD-123F) | Anodes on VIN_MOBO/VIN_AUX, cathodes on V24 | Correct |
| D6 | 1N4004W (SOD-123F) | Anode on V12 (LM317 output), cathode on V24 (LM317 input); reverse-protection diode | Correct |
| C7 | 47 µF 63 V electrolytic | Pin 1 (+, square pad) on V12CAP | Correct |
| C11 | CDA CPM-12R0L804R-TW (DNP) | Pin 1 (+, square pad) on C11_P | Correct in the design; the positive lead could not be confirmed because the CDA datasheet was not available |

## Notes

- **Do not substitute TLVH431 for U12.** In the SOT-23-3 (DBZ) package, TLVH431 is 1 REF, 2 CATHODE, the reverse of TLVH432 on pins 1 and 2.
- **U8 VREF tied to 3.3 V is within range.** The DRV8251A datasheet allows 0–5 V on VREF.
- **LM317MABDTRKG is active.** Only NCV317MBDTG and NCV317MBTG are marked discontinued in the onsemi datasheet.
- **J6 pin 1 location** on the Molex 87438-0443 body was not confirmed against the Molex drawing (the Molex site timed out).

## Sources

- TI: [DRV8251A](https://www.ti.com/lit/ds/symlink/drv8251a.pdf) (SLVSFU6), [TLVH431/TLVH432](https://www.ti.com/lit/ds/symlink/tlvh432.pdf) (SLVS555N), [TLV902x/TLV903x](https://www.ti.com/lit/ds/symlink/tlv9031.pdf) (SNOSDA3H), [SN74LVC1G17](https://www.ti.com/lit/ds/symlink/sn74lvc1g17.pdf) (SCES351Y), [SN74LVC1G32](https://www.ti.com/lit/ds/symlink/sn74lvc1g32.pdf) (SCES219W), [SN74LVC2G04](https://www.ti.com/lit/ds/symlink/sn74lvc2g04.pdf), [SN74LVC2G08](https://www.ti.com/lit/ds/symlink/sn74lvc2g08.pdf) (SCES198N)
- onsemi: [LM317M](https://www.onsemi.com/pdf/datasheet/lm317m-d.pdf), [MBRS240LT3](https://www.onsemi.com/pdf/datasheet/mbrs240lt3-d.pdf)
- Bourns: [3314](https://www.bourns.com/docs/Product-Datasheets/3314.pdf)
- E-Switch: [TL3342](https://dev.e-switch.com/wp-content/uploads/2022/06/TL3342.pdf)
- 74LVC1G157: [Farnell copy of the Nexperia datasheet](https://www.farnell.com/datasheets/2787496.pdf), [Nexperia 74LVC1G157-Q100](https://assets.nexperia.com/documents/data-sheet/74LVC1G157_Q100.pdf)
- L78L33: [elcodis excerpt of the ST datasheet](https://elcodis.com/parts/1379332/L78L33ABUTR_p4.html), [third-party 78L33 datasheet 1](https://atta.szlcsc.com/upload/public/pdf/source/20240930/B0D637D170813FBEC3C5DF489B34052F.pdf), [third-party 78L33 datasheet 2](https://atta.szlcsc.com/upload/public/pdf/source/20250918/73619059103F16A1CB202C36A2F2E3FA.pdf)
- Still to obtain: Nexperia [74LVC2G08](https://www.nexperia.com/product/74LVC2G08DC) and [74LVC1G157](https://www.nexperia.com/group/74lvc1g157), ST [L78L](https://www.st.com/resource/en/datasheet/l78l.pdf)
