# Parts list

The meter has two units joined by a 4-wire cable:

- **RF head**: a die-cast box that sits in the coax between the amplifier and the antenna. It
  holds the directional coupler, two attenuators and two log detectors. Only DC leaves it.
- **Display unit**: an ESP32 with a screen and a 16-bit ADC. It works out power and SWR, shows
  them, and sends them over Wi-Fi.

Designed for 1.8–54 MHz at up to 150 W (Xiegu XPA125B output), so 100 W has headroom.
Reference designators match [schematic.svg](schematic.svg).

## RF head

| Ref | Qty | Part | Spec / notes | Approx. cost |
|---|---|---|---|---|
| T1, T2 | 2 | Ferrite toroid **FT82-43** | 21 mm OD, mix 43. Same core for both so the two couplings match | $3 |
| — | 1 | Enamelled copper (magnet) wire, 26 AWG | About 2 m. 24 turns on each core, plus leads | $5 (spool) |
| — | 2 | Coax, **RG-142** (or RG-400), 5 cm each | Single-turn primaries through T1 and T2. PTFE dielectric doesn't melt when you solder it. RG-58 works if that's what you have | $5 |
| J1, J2 | 2 | **SO-239** chassis socket, 4-hole flange | PTFE insulator, silver or gold centre pin. J1 from amp, J2 to antenna | $8 |
| ATT1, ATT2 | 2 | **20 dB fixed attenuator, SMA, 50 Ω, ≥ 2 W**, DC–3 GHz or better | Protects the detectors and puts 100 W at about +2 dBm on the AD8307. See the DIY alternative below | $10 |
| — | 2 | RG-316 pigtail, SMA male to open end, about 10 cm | From the coupler ports (X, Y) to the attenuators | $6 |
| — | 2 | SMA male–male adapter | Attenuator to detector module, if the module has an SMA female input | $4 |
| U1, U2 | 2 | **AD8307 RF power detector module** (1–500 MHz, about −75 to +16 dBm, SMA input) | Log detector, 25 mV/dB. Buy one whose listing says the input is 50 Ω terminated. Runs on 5 V, about 10 mA each | $20 |
| J3 | 1 | **GX12 4-pin** aviation connector, panel socket | DC only: +5 V, GND, REF, FWD | $3 (pair) |
| FB1–FB4 | 4 | Ferrite bead, mix 43, slip-on (e.g. FB-43-101) or SMD 600 Ω | One in series with each DC line at J3 | $2 |
| C3–C10 | 8 | Ceramic capacitor, 1 nF + 100 nF, 50 V (4 of each) | Bypass each J3 line to chassis | $1 |
| — | 1 | Die-cast aluminium box, **Hammond 1590BB** (119 × 94 × 34 mm) | Keep the RF inside metal | $15 |
| — | 1 | Single-sided copper-clad board, about 100 × 60 mm | Ground plane for the coupler, solder-on "dead-bug" style | $3 |
| — | — | Solder lugs, M3 hardware, standoffs, heat-shrink, nylon cable ties | Ground lugs at J1, J2 and J3; ties hold the toroids | — |

**RF head subtotal: about $85**

### DIY attenuator (instead of ATT1 and ATT2)

Each attenuator is a 20 dB π pad, 50 Ω in and out, built on the copper-clad board:

| Position | Value | Rating | Why |
|---|---|---|---|
| Input shunt | 61.9 Ω, 1 % | **1 W**, metal film | Takes about 0.2 W at 150 W through the coupler |
| Series | 249 Ω, 1 % | ¼ W | |
| Output shunt | 61.9 Ω, 1 % | ¼ W | |

Keep leads very short. The SMA attenuators are worth the extra few dollars because they're
already calibrated to 20 dB.

## Display unit

| Ref | Qty | Part | Spec / notes | Approx. cost |
|---|---|---|---|---|
| U3 | 1 | **LilyGO T-Display** (ESP32, 1.14" ST7789 135 × 240) | You already have one. Shows FWD W, REF W, SWR and sends data over Wi-Fi | — |
| U4 | 1 | **ADS1115** 16-bit I²C ADC module | Address 0x48 (ADDR to GND). Runs on 3.3 V from the T-Display. The ESP32's own ADC is too nonlinear for this, and its ADC2 pins don't work while Wi-Fi is on | $4 |
| R1, R2 | 2 | 1 kΩ, ¼ W | RC filter in front of each ADC input | — |
| C1, C2 | 2 | 100 nF ceramic | RC filter (1.6 kHz corner) and RF bypass | — |
| J4 | 1 | GX12 4-pin cable plug (mate of J3) | | (in J3 pair) |
| — | 1 | 4-core shielded cable, 1–2 m | Ground the shield **at the RF head only** | $5 |
| — | 1 | Small plastic or 3D-printed case for the T-Display | Screen window, USB-C opening | — |
| — | 1 | Clip-on ferrite (mix 31) for the USB-C cable | Keeps RF off the shack's 5 V supply | $3 |
| — | 1 | Perfboard, about 50 × 70 mm | Mounts ADS1115 and RC filters | $2 |

**Display unit subtotal: about $15** (plus the T-Display you already have)

## Test and calibration (you probably have these)

- 50 Ω dummy load rated for at least 100 W (for directivity check and calibration)
- The HL2 by itself (about 5 W) as a known low-power source with the amp bypassed
- Multimeter

## Design numbers

How the part values were chosen, at 24 turns on each core:

| Quantity | Value |
|---|---|
| Coupling factor | 20·log₁₀(24) = **−27.6 dB** |
| 100 W (+50.0 dBm) at a coupler port | +22.4 dBm (0.17 W) |
| …after the 20 dB attenuator, at the AD8307 | **+2.4 dBm** (detector tops out about +16 dBm) |
| 150 W at the AD8307 | +4.2 dBm |
| 1 W at the AD8307 | −17.6 dBm |
| Reflected at 100 W, SWR 1.05:1 (return loss 32 dB) | −29.9 dBm, well above the detector's −75 dBm floor |
| AD8307 output at +2.4 dBm | about 2.2 V, inside ADS1115's ±4.096 V range on a 3.3 V supply |
| ADS1115 resolution at that range | 125 µV = 0.005 dB per step |
| T2 magnetising reactance at 1.8 MHz (A_L ≈ 557 nH/t², 24 t → 0.32 mH) | about 3.6 kΩ, so it barely loads the 50 Ω line |
| T2 peak flux at 1.8 MHz, 100 W | about 150 gauss (falls with frequency: about 77 G on 80 m) |

If T2 gets warm after long 160 m or 80 m transmissions at full power, rewind it on an
**FT114-43** with the same 24 turns. Both cores must keep the same turn count.

Directivity comes from layout and matching, not parts. Expect 25–30 dB on HF and less on 6 m.
The build notes cover how to check it with the dummy load.
