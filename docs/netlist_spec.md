# shuttercontrol — netlist specification (transcribed from ckt_rev2h p1/p2 + BOM rev 1.0)

Source: docs/ckt_rev2h_p1of2.jpg, docs/ckt_rev2h_p2of2.jpg (V.02h, 10/11/26), docs/BOM_shutter-controller.xlsx (rev 1.0, 2026-10-12).

## Reference-designator resolution (conflicts in the hand drawing)
- p1 draws the two LVC1G157 muxes as U13/U14, but p2 and the BOM use U13-U17 for TLVH432. → muxes renamed **U18 / U19**.
- p1 draws the 10k manual-switch pull-ups as R25/R26, but p2 and BOM use R25-R34 for 1.24k. → pull-ups renamed **R40 (was p1 R25, S2/enable)** and **R41 (was p1 R26, S1/direction)**.
- p1 U12 drawn as TLVH431A; BOM says U12-U17 = TLVH432AIPK → U12 uses TLVH432AIPK.
- C5, C8 drawn as 0.33 µF; BOM (newer) says 0.47 µF → 0.47 µF used.
- Generic cell drawn at bottom-right of p1 (RT/RB/RK/Uz/Cs) is just the template for p2; not placed separately.
- CdS photocell NSL-19M51 is off-board (plugs into J4) → not placed.
- U10 heatsink is mechanical → not placed.

## Power / global nets
GND, V24 (after D4/D5), V12 (7812 output), V12CAP, V3IN (78L33 input), V3R3.

## Pin maps used
- TLV9031DBV (SOT-23-5): 1 OUT, 2 V-, 3 IN+, 4 IN-, 5 V+
- 74LVC1G17DBV (SOT-23-5): 1 NC, 2 A, 3 GND, 4 Y, 5 VCC
- 74LVC1G32DBV (SOT-23-5): 1 A, 2 B, 3 GND, 4 Y, 5 VCC
- 74LVC2G08DCT (SM8/SSOP-8): 1 1A, 2 1B, 3 2Y, 4 GND, 5 2A, 6 2B, 7 1Y, 8 VCC
- 74LVC2G04DBV (SOT-23-6): 1 1A, 2 GND, 3 2A, 4 2Y, 5 VCC, 6 1Y
- 74LVC1G157DBV (SOT-23-6): 1 I1, 2 GND, 3 I0, 4 Y, 5 VCC, 6 S
- DRV8251ADDAR (HSOP-8 EP): 1 IPROPI, 2 IN2, 3 IN1, 4 VREF, 5 VM, 6 OUT1, 7 PGND, 8 OUT2, 9 EP=GND
- LM7812SX (TO-263): 1 IN, 2 GND (tab), 3 OUT
- L78L33ABUTR (SOT-89): 1 OUT, 2 GND, 3 IN
- TLVH432AIPK (SOT-89): 1 CATHODE, 2 ANODE, 3 REF  (as annotated on p1 cell drawing — verify against TI datasheet)

## Connections

### V12CAP monitor (U1)
- R1 107k 1%: V12CAP — V12SENSE
- R2 12.1k 1%: V12SENSE — GND
- R3 2.32k 1%: V3R3 — VREF1V
- R4 1.0k 1%: VREF1V — GND
- C1 0.1µ: VREF1V — GND
- U1 TLV9031: 3(IN+)=V12SENSE, 4(IN-)=VREF1V, 1=V12CAPGOOD, 5=V3R3, 2=GND
- R5 100k: V12SENSE — V12CAPGOOD

### Night sensor (U2)
- J4 (2-pin): 1=V3R3, 2=SENSOR
- RV1 1M trimmer (rheostat, wiper tied to GND end): SENSOR — GND
- C2 0.1µ: SENSOR — GND
- U2 TLV9031: 4(IN-)=SENSOR, 3(IN+)=NIGHT_HYS, 1=NIGHT, 5=V3R3, 2=GND
- R6 100k: V3R3 — NIGHT_HYS
- R7 5.1k (drawn 5k): NIGHT_HYS — GND
- R8 51k (drawn 50k): NIGHT_HYS — NIGHT
- R11 1k: NIGHT — NIGHT_OUT (J1-8)

### V24 detect (U3)
- R9 22.1k 1%: V24 — V24DIV
- R10 2.0k 1%: V24DIV — GND
- U3 74LVC1G17: 2=V24DIV, 4=V24GOOD, 5=V3R3, 3=GND, 1=NC

### Control logic
- U4 74LVC2G08 (VCC 8=V3R3, GND 4)
  - U4a: 1=V24GOOD, 2=NIGHT, 7=AUTO_NIGHT
  - U4b: 5=AUTO_OR, 6=OPEN, 3=CMD_OPEN
- U5 74LVC1G32: 1=AUTO_NIGHT, 2=OVERRIDE, 4=AUTO_OR, 5=V3R3, 3=GND
- R12 10k: OVERRIDE — GND
- R13 10k: OPEN — GND
- U6 74LVC2G04 (VCC 5=V3R3, GND 2)
  - U6a: 1=CMD_OPEN, 6=CMD_OPEN_N
  - U6b: 3=MAN_DIR, 4=MAN_DIR_N
- U7 74LVC2G08 (VCC 8=V3R3, GND 4)
  - U7a: 1=CMD_OPEN, 2=V12CAPGOOD, 7=AUTO_IN1
  - U7b: 5=CMD_OPEN_N, 6=V12CAPGOOD, 3=AUTO_IN2
- U18 74LVC1G157: 1(I1)=AUTO_IN1, 3(I0)=MAN_DIR_N, 6(S)=MAN_EN, 4(Y)=DRV_IN1, 5=V3R3, 2=GND
- U19 74LVC1G157: 1(I1)=AUTO_IN2, 3(I0)=MAN_DIR,   6(S)=MAN_EN, 4(Y)=DRV_IN2, 5=V3R3, 2=GND
- R40 10k: V3R3 — MAN_EN
- R41 10k: V3R3 — MAN_DIR
- S1 pushbutton (direction): MAN_DIR — GND
- S2 pushbutton (enable): MAN_EN — GND

### Motor driver (U8)
- U8 DRV8251A: 1=IPROPI, 2=DRV_IN2, 3=DRV_IN1, 4=V3R3, 5=V12CAP, 6=OUT1, 7=GND, 8=MOTOR_B, 9(EP)=GND
- C3 0.1µ: V12CAP — GND
- C7 47µ 63V electrolytic (DNP): V12CAP(+) — GND
- L1 4.7mH: OUT1 — MOTOR_A
- C10 4.7µ film (DNP): MOTOR_A — MOTOR_B
- J2 (3-pin): 1=MOTOR_A, 2=GND, 3=MOTOR_B
- J3 (3-pin): 1=MOTOR_A, 2=GND, 3=MOTOR_B

### Motor current sense (U9)
- R14 634 1%: IPROPI — GND
- R15 10k: IPROPI — ISENSE
- C4 0.1µ: ISENSE — GND
- U9 TLV9031: 4(IN-)=ISENSE, 3(IN+)=MOTOR_HYS, 1=MOTOR_N, 5=V3R3, 2=GND
- R16 3.3k: V3R3 — MOTOR_HYS
- R17 100: MOTOR_HYS — GND
- R18 10k: MOTOR_HYS — MOTOR_N
- R19 1k: MOTOR_N — MOTOR_OUT (J1-7)

### Connectors
- J1 Molex 9014-71108 (8-pin SIP female): 1=VIN_MOBO, 2=NC, 3=NC, 4=GND, 5=OPEN, 6=OVERRIDE, 7=MOTOR_OUT, 8=NIGHT_OUT
- J5 (4-pin aux power / manual): 1=VIN_AUX, 2=MAN_DIR, 3=MAN_EN, 4=GND

### Power
- D4 1N4004W: A=VIN_MOBO, K=V24
- D5 1N4004W: A=VIN_AUX,  K=V24
- C5 0.47µ: V24 — GND
- U10 LM7812: 1=V24, 2=GND, 3=V12
- C6 0.1µ: V12 — GND
- D1 1N5819W: A=V12, K=V12_D
- R20 100Ω 2W: V12_D — V12CAP
- R21 300 (E24): V12CAP — V3_D2
- D2 1N4148W: A=V3_D2, K=V3IN
- R22 1k 0.5W: V24 — V3_D3
- D3 1N4148W: A=V3_D3, K=V3IN
- C8 0.47µ: V3IN — GND
- U11 L78L33: 3=V3IN, 2=GND, 1=V3R3
- C9 2.2µ: V3R3 — GND
- R23 5.49k 1%: V3R3 — CLAMP_REF
- R24 2.49k 1%: CLAMP_REF — GND
- U12 TLVH432: K=V3R3, A=GND, REF=CLAMP_REF  (4V clamp)

### Backup capacitor stack (sheet 2, 5 cells in series between V12CAP and GND)
Nodes: N0=V12CAP, N1=CAP1, N2=CAP2, N3=CAP3, N4=CAP4, N5=GND.
For cell i = 1..5 (top node N(i-1), bottom node N(i)):

| cell | R_top 1.24k | R_bot 1.24k | R_K 22Ω | U TLVH432 | C 3F/3V (DNP) | ref node |
|---|---|---|---|---|---|---|
| 1 | R25 | R26 | R35 | U13 | C11 | REF1 |
| 2 | R27 | R28 | R36 | U14 | C12 | REF2 |
| 3 | R29 | R30 | R37 | U15 | C13 | REF3 |
| 4 | R31 | R32 | R38 | U16 | C14 | REF4 |
| 5 | R33 | R34 | R39 | U17 | C15 | REF5 |

- R_top: N(i-1) — REFi
- R_bot: REFi — N(i)
- R_K: N(i-1) — Ki (cathode of U)
- U: K=Ki, A=N(i), REF=REFi
- C: + = N(i-1), − = N(i)
