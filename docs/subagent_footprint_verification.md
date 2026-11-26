# Footprint Verification

Date: 2026-11-25

Every footprint in `shuttercontrol.kicad_pcb` (33 footprint types, 78 parts) was compared with the manufacturer's datasheet or drawing: package code, body size, pitch, lead diameter against drill size, recommended land pattern, and pin-1 / polarity location. Pin-name mapping was already covered in [`pinout_verification.md`](pinout_verification.md) and is only revisited where this check found a problem.

No design files were changed.

## Summary

- **One footprint is wrong: C7.** The part is φ10 × 12.5 mm with 5.0 mm lead pitch. The footprint is for φ8 mm with 3.5 mm pitch.
- **C10 is end-of-life.** TDK's last order date has passed, and the recommended replacement does not fit the footprint.
- **R27 / RV1 terminals 1 and 3 are mirrored** compared with the Bourns top view. This only reverses the adjustment direction. It also contradicts an entry in `pinout_verification.md`.
- **C11 needs a physical check.** The lead pitch was confirmed. The lead-row offset and the polarity were not.
- **Minor issues:** several through-holes are larger than the manufacturer recommends (J1, J2–J5), and the D4–D6 pads are narrower than recommended.
- All ICs, the remaining discretes, the chip passives, S1/S2, J6, JP1 and TP1–TP3 match their datasheets.

## Method

1. Extracted pad number, position, size and drill for every footprint instance from `shuttercontrol.kicad_pcb`.
2. Checked that the schematic `Footprint` field matches the PCB footprint for every reference. All match; H1–H3 (mounting holes) exist only on the PCB, which is expected.
3. Found and read the manufacturer datasheets. This was split across three research subagents (ICs, passives and discretes, connectors). J6 was checked directly against the drawing in `datasheet/874380443_sd.pdf`.
4. Re-read the claims that drive action items (C7, C10, C11, R27/RV1, J1 hole size) in the original PDFs, and checked board clearance where it mattered.

**Bottom-side parts.** About 50 SMD parts are on B.Cu, including U1–U4, U6, U7, U9–U14, D1, D2, D4–D6 and most 0805 passives; the JLCPCB CPL marks them `Bottom`. KiCad stores their pad coordinates already mirrored, so pin 1 appears on the opposite Y side in the raw PCB data. These are standard library footprints flipped to the back, not footprint errors. Clearance checks below account for the board side.

**Provenance.** Each finding is tagged:

| Tag | Meaning |
|---|---|
| **V** | Verified directly in the manufacturer document during the cross-check |
| **S** | Read from the manufacturer document by a subagent; not independently re-checked |
| **I** | Inferred, or taken from a distributor page rather than a datasheet |

## Action items

### 1. C7: wrong footprint (must fix) [V]

| | |
|---|---|
| Part | Nichicon UBT1J470MPD8TD, 47 µF 63 V, DNP |
| Datasheet | Nichicon UBT catalog CAT.8100M: UBT1J470MPD8 is **φ10 × 12.5 mm**. Lead table: φ10 → **P = 5.0 mm**, φd = 0.6 mm. "TD" is ammo-pack taping with straight leads and does not change the pitch [S]. |
| Footprint | `Capacitor_THT:CP_Radial_D8.0mm_P3.50mm` |
| Fix | Use `Capacitor_THT:CP_Radial_D10.0mm_P5.00mm`. The 0.8 mm drill is fine for 0.6 mm leads. |
| Layout impact | No top-side part or pad lies within 6 mm of the new body center (the nearby C3, R1 and R2 are on B.Cu, under the board). Pad 1 can stay where it is; pad 2 moves 1.5 mm further along the same axis, to (128.90, 120.16). A 0.5 mm F.Cu track runs with its centerline 1.27 mm from that hole, about 0.22 mm edge-to-edge from a 1.6 mm pad. Run DRC after the change and nudge the track if needed. |

### 2. C10: end-of-life, and the outline is slightly short [V]

| | |
|---|---|
| Part | TDK/EPCOS B32529D5475M289, 4.7 µF film, DNP |
| Datasheet | TDK B3252* datasheet: B32529D5475M has max dimensions w × h × l = **7.8 × 13.0 × 7.8 mm** and 5.0 mm lead spacing. "289" = ammo pack [S]. |
| Footprint | `C_Rect_L7.2mm_W11.0mm_P5.00mm_FKS2_FKP2_MKS2_MKP2` (L 7.2 × W 11.0, drill 0.8) |
| Assessment | Pitch and drill are correct. The body is 7.8 mm long against a 7.2 mm outline, so the silkscreen and courtyard are about 0.3 mm short at each end. The thickness (7.8 mm against 11.0 mm) is conservative. |
| Lifecycle | TDK PCN: discontinued 2026-02-27, **last order 2026-06-12**, last shipment 2026-09-11. Suggested replacement B32522C0475M289 has **15 mm** lead spacing and does not fit this footprint. |
| Action | Choose a new part or drop the position, then update the footprint to match. |

### 3. R27 / RV1: terminals 1 and 3 are mirrored [V]

| | |
|---|---|
| Part | Bourns 3314G-1-201E (R27) and 3314G-1-105E (RV1) |
| Datasheet | Bourns 3314 datasheet: the land pattern matches the footprint exactly (two 1.3 × 1.3 pads at 2.3 mm centers; one 2.0 × 1.3 pad; 5.5 mm between rows). The top view has wiper 2 at the top, **3 bottom-left, 1 bottom-right**. Rotated so the wiper is at the bottom, that is 1 top-left and 3 top-right. |
| Footprint | `Potentiometer_Bourns_3314G_Vertical`: pad 2 at (0, +2.75), **pad 1 at (+1.15, −2.75), pad 3 at (−1.15, −2.75)**. Pads 1 and 3 are swapped left/right compared with the datasheet. |
| Impact | The adjustment direction is reversed (clockwise lowers instead of raises the 1–2 resistance). No functional failure. |
| Caveat | The top view read was the 3314J-1 drawing. The 3314G shares the terminal layout on the same page [S]. |
| Action | Swap pads 1 and 3 only if the adjustment direction matters. Correct the R27/RV1 row in `pinout_verification.md`, which says "1 = CCW, 3 = CW … Match". |

### 4. C11: confirm lead offset and polarity on a sample [V, partial]

| | |
|---|---|
| Part | CDA CPM-12R0L804R-TW (12 V 0.8 F, 17 × 17 × 22 mm); footprint also intended for CPM-12R0L255R-TW (12 V 2.5 F, 21 × 21 × 32 mm). DNP. |
| Datasheet | CDA CPM series spec, 2025-08-11 (dimensions page). DA type: P = 8.5 ± 0.5 mm (D17), 10.5 mm (D21), lead Φ0.6. The end view shows **both leads centered left-to-right, with the lead row close to one body edge**. That distance is not dimensioned. The side view marks + and −, but the end view does not say which lead is which. |
| Footprint | Custom `CDA_CPM_DA_D17.0-21.0mm_P8.50-10.50mm`: + at (0, 0), − at 8.5 and 10.5 mm, drill 1.0, lead row assumed 2.6 mm from the body edge. |
| Assessment | Pitch, both − hole positions and drill are correct. A subagent measured the lead-row offset from the (not-to-scale) drawing at about 2.45 mm, which agrees with the 2.6 mm assumption, but this is not a dimensioned value. |
| Board check | If the leads were centered on the body instead, the D17 body would cover J5, and the D21 body would also cover D4, D5 and R9. The datasheet drawing suggests that does not happen. |
| Action | On a physical part, measure the lead-to-edge distance and identify the + lead. Make sure the + mark on the silkscreen is clear. |

### 5. Through-holes larger than recommended (minor)

| Ref | Part | Recommended hole | Footprint drill | Tag |
|---|---|---|---|---|
| J1 | Molex 90147-1108 (C-Grid III single-row vertical receptacle) | Ø0.80 +0.10/−0.05 | 1.0 | V |
| J1 alt | TE 534237-6 (AMPMODU, 8 pos; "-6" = 8 positions confirmed) | Ø0.88 ± 0.08 | 1.0 | S |
| J2–J5 | Molex 22-11-2032 / -2022 / -2042 (KK 254) | Ø1.02 ± 0.05 | 1.19 | S |
| R20 | Uniroyal MFR02SF1000A10 (lead Ø0.65 ± 0.05) | — | 1.2 | S |

All of these can be soldered. J1 may sit tilted unless it is held during soldering. Optional changes: drill J1 at about 0.9 mm and J2–J5 at about 1.05 mm.

The connector subagent identified 22-11-20x2 as Molex **6373** series (KK 254 vertical friction-lock header) from drawing SD-6373-001. This was not independently confirmed [S]. The pin positions match the KiCad 6410 footprint either way.

### 6. D4–D6 pads narrower than recommended (minor) [S]

| | |
|---|---|
| Part | 1N4004W, LCSC C18199087 (hongjiacheng / ichjc.com) |
| Datasheet | Case SOD-123FL [V]. Body 2.55–2.85 × 1.6–1.9 mm, overall length 3.6–3.9 mm. Recommended land: pad width ≥ 1.5 mm, inner gap ≤ 1.9 mm. |
| Footprint | `D_SOD-123F`: 1.1 × 1.1 pads at ±1.4 mm (inner gap 1.7, outer extent 3.9) |
| Assessment | The package type and pitch are right. The pads are narrower than recommended, and the outer extent equals the maximum part length, leaving no toe fillet. |
| Optional | Pads of about 1.2 × 1.5 mm at ±1.45 mm. |

## Footprints that match the datasheet

### ICs

| Ref | Part / package | Datasheet | Result | Tag |
|---|---|---|---|---|
| U8 | TI DRV8251ADDAR, DDA (HSOP-8 PowerPAD) | SLVSFU6 (package drawing 4214849/B) | Exposed pad 2.41 × 3.1 mm nominal (2.41 across the rows) matches the footprint exactly. Pitch 1.27; rows 5.3 mm apart against 5.4 recommended. TI's thermal land is larger (2.71 × 3.4 mask-defined opening on 2.95 × 4.9 copper) with 8 optional Ø0.2 mm vias; enlarging the EP copper is optional. The board has 6 vias (0.3 mm drill) under the EP [V]. | S |
| U1, U2, U9 | TI TLV9031DBVR, SOT-23-5 (DBV) | SNOSDA3H | Matches the DBV land pattern (KiCad toe is about 0.05 mm shorter) | S |
| U3 / U5 | TI SN74LVC1G17DBVR / SN74LVC1G32DBVR, DBV | SCES351Y / SCES219W | Same DBV land pattern | S |
| U6 | TI SN74LVC2G04DBVR, SOT-23-6 (DBV) | SCES195 | Matches; pin 1 top-left, numbered counter-clockwise | S |
| U12 | TI TLVH432AIDBZR, SOT-23-3 (DBZ) | SLVS555N | Pin order matches; KiCad pads about 0.15 mm longer than TI | S |
| U4, U7 | Nexperia 74LVC2G08DC,125, VSSOP8 (SOT765-1) | Nexperia 74LVC2G08 Rev. 16 (2019-07-29); NXP SOT765-1 package information (2016). | Geometry compared with the NXP SOT765-1 reflow footprint: pads 0.30 × 0.75 mm, outer span 3.5 mm, centers at ±1.375. KiCad pads are 0.35 × 1.25 at ±1.4: longer, slightly wider, 0.15 mm gap between pads. Compatible. The datasheet ordering table lists 74LVC2G08DC as VSSOP8, SOT765-1 [V]. | S |
| U13, U14 | Nexperia 74LVC1G157GW,125, SOT363 | Nexperia 74LVC1G157 Rev. 10 (2021-06-10); NXP SOT363 package information (2018). | 74LVC1G157GW = SC-88 / SOT363 per the datasheet ordering table [V]. Pitch 0.65, pin count and pin 1 match. NXP reflow footprint: rows ±0.9, lands about 0.6 × 0.4–0.5 (read from the figure, ambiguous). KiCad: rows ±0.8375, 1.025 × 0.35. **Check pad size visually.** | S |
| U10 | onsemi LM317MABDTRKG, DPAK (case 369C) | LM317M/D rev 18 | Pitch 4.56–4.57 and tab = Vout (pin 2) match; KiCad lead pads are smaller than onsemi's but standard | S |
| U11 | ST L78L33ABUTR, SOT-89 | **ST datasheet not retrieved** (st.com timed out). Geometry compared with JEDEC TO-243 / NXP SOT89 package information (2016). | Overall land length 4.6 mm matches. Tab pad is 1.733 mm wide, about the package tab width (1.8 max) and narrower than the NXP reference land (about 2.0). Workable; 2.0 mm would follow the reference. Pin order (1 OUT, 2 GND/tab, 3 IN) still rests on the secondary sources in `pinout_verification.md`. | S |

### Discretes and passives

| Ref | Part | Result | Tag |
|---|---|---|---|
| D1 | onsemi MBRS240LT3G, SMB | Pad 1 = cathode; pads 2.3 mm tall against 2.74 recommended (still wider than the 2.03 mm lead) | S |
| D2, D3 | Vishay 1N4148W-HE3-08, SOD-123 (not 123F) | Matches. DigiKey lists this MPN as obsolete. | S |
| C1–C6, C8, C9 | 0805 MLCCs (YAGEO, Samsung) | All 0805; CL21B474 and CL21A225 are 1.25 mm thick, which does not affect the land pattern | S |
| All 0805 resistors | Uniroyal / YAGEO / FOJAN | 0805 by MPN | I |
| R22 | FOJAN FRC1210J102TS | 1210, 0.5 W | S |
| R28 | Uniroyal 1206W4F100KT5E | 1206, 1 Ω, 0.25 W (LCSC page only) | I |
| R20 | Uniroyal MFR02SF1000A10, 2 W | Body Ø4.0 × 11.5 mm. Works with formed leads on DIN0617 (P20.32). A closer fit is DIN0414 (optional). | S |
| S1, S2 | E-Switch TL3342F450QG | Land pattern (6.3 × 3.8 mm pad centers, 1.7 × 1.0 pads) matches. "450" is 450 gf operating force, not height. | S |
| TP1–TP3 | Keystone 5000 / 5001 / 5002 | Recommended hole 0.040" (1.0 mm) matches | S |

### Connectors

| Ref | Part | Result | Tag |
|---|---|---|---|
| J1 | Molex 90147-1108 / TE 534237-6 | Single-row 8-pos 2.54 mm vertical receptacles; pitch matches. DigiKey WM14834-ND = 90147-1108 and A26420-ND = 534237-6 confirmed. Hole size: see item 5. | V / S |
| J2–J5 | Molex 22-11-2032 / -2022 / -2042 | Pitch and pin count match. Hole size: see item 5. | S |
| J6 | Molex 87438-0443 (Pico-SPOX, 4 ckt, beige) | Checked against drawing 874380231-SD rev D1, sheet 2. Pad split 3.55 / 0.8 keep-out / 2.65, width 0.85, pitch 1.5 all match. Circuit 1 location matches the recommended (top-view) layout. Mating face points at the board edge, with the body about 0.5 mm inside the edge. | V |
| JP1 | XFCN PZ254V-11-02P | 0.64 mm square post, recommended hole Ø1.02; footprint 1.0 | S |

## Open points for manual review

1. C7: change to `CP_Radial_D10.0mm_P5.00mm` and run DRC (one F.Cu track is close to the new pad 2).
2. C11: lead-row offset and + lead on a physical part.
3. C10: replacement part (EOL), then footprint.
4. R27 / RV1: decide whether to swap pads 1 and 3; correct `pinout_verification.md`.
5. U13 / U14: visual check of SOT-363 pad size against the Nexperia SOT363 reflow footprint.
6. U11: obtain the ST L78L datasheet (st.com and all mirrors tried timed out or returned HTML) and confirm the SOT-89 pin order and recommended footprint (tab land width).
7. (closed 2026-11-26: Nexperia 74LVC2G08 Rev. 16 confirms "DC" = SOT765-1)
8. J2–J5: confirm the Molex series (6373 according to the subagent) and the recommended hole.
9. `pinout_verification.md` cites <https://www.farnell.com/datasheets/2787496.pdf> as a copy of the 74LVC1G157 datasheet; that file is actually the **74LVC1G3157** (analog switch). Replace the reference with `datasheet/74LVC1G157_Nexperia_Rev10.pdf`.
10. Optional: J1 drill about 0.9 mm, J2–J5 drill about 1.05 mm, D4–D6 pads about 1.2 × 1.5 mm.

## Sources

Local copies of all documents below (except ST L78L) are in [`datasheet/`](../datasheet/).


- Molex 87438 drawing 874380231-SD rev D1: `datasheet/874380443_sd.pdf` (in repo)
- Nichicon UBT catalog: <https://www.nichicon.co.jp/english/series_items/catalog_pdf/e-ubt.pdf>; Nichicon radial taping: <https://www.nichicon.com/getmedia/db907a2c-1e93-45c3-ac35-3127b06dc2cd/lytic-radial-taping.pdf>
- TDK B32520–B32529: <https://www.tdk-electronics.tdk.com/inf/20/20/db/fc_2009/B32520_529.pdf>
- CDA CPM series: <https://en.cda-cap.com/wp-content/uploads/2025/08/EN_Bus_Elx_DS_4089_CPM_Series.pdf>
- Bourns 3314: <https://www.bourns.com/docs/Product-Datasheets/3314.pdf>
- Molex 90147 drawing SDA-90147 (mirror): <https://cdn.eicom.ru/media/PDF/872901.pdf>
- Molex KK 6373 drawing SD-6373-001 (mirror): <https://www.farnell.com/cad/2792066.pdf>
- TE 534237 customer drawing: via LCSC datasheet mirror
- TI: [DRV8251A](https://www.ti.com/lit/ds/symlink/drv8251a.pdf), [TLVH432](https://www.ti.com/lit/ds/symlink/tlvh432.pdf), [TLV9031](https://www.ti.com/lit/ds/symlink/tlv9031.pdf), [SN74LVC1G17](https://www.ti.com/lit/ds/symlink/sn74lvc1g17.pdf), [SN74LVC1G32](https://www.ti.com/lit/ds/symlink/sn74lvc1g32.pdf), [SN74LVC2G04](https://www.ti.com/lit/ds/symlink/sn74lvc2g04.pdf)
- onsemi: [LM317M](https://www.onsemi.com/pdf/datasheet/lm317m-d.pdf), [MBRS240LT3](https://www.onsemi.com/download/data-sheet/pdf/mbrs240lt3-d.pdf)
- ST: [L78L](https://www.st.com/resource/en/datasheet/l78l.pdf) (not retrieved: st.com timed out)
- Nexperia 74LVC2G08 Rev. 16 (mirror: <https://static.chipdip.ru/lib/441/DOC011441482.pdf>) and 74LVC1G157 Rev. 10 (mirror: <https://cdn.components.ru/media/PDF/1726248.pdf>); assets.nexperia.com refused direct downloads. Package geometry from NXP package information: [SOT765-1](https://www.nxp.com/docs/en/package-information/SOT765-1.pdf), [SOT363](https://www.nxp.com/docs/en/package-information/SOT363.pdf), [SOT89](https://www.nxp.com/docs/en/package-information/SOT89.pdf)
- Vishay 1N4148W (doc 85748): <https://www.farnell.com/datasheets/2046233.pdf>
- E-Switch TL3342: <https://sten-eswitch-13110800-production.s3.amazonaws.com/system/asset/product_line/data_sheet/165/TL3342.pdf>
- Keystone 5000–5004 (M65p56): <https://www.keyelco.com/userAssets/file/M65p56.pdf>
- LCSC datasheets: C18199087 (1N4004W), C58717 (MFR02S), C377773 (Samsung CL21), C2930439 (FRC1210), C492401 (PZ254V)
