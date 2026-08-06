# Whole-House Water Monitoring Station — Revised Specification

**Status:** Design v2 (cost-reduced, pre-treatment)
**Platform:** ESP32-S3 + ESPHome → Home Assistant
**Network:** IoT VLAN (VLAN 10 on this network), static IP
**Revision date:** 2026-08-06

---

## 1. Scope and design premise

This is a **change-detection and infrastructure-health system**, not a
drinking-water safety instrument. Conductivity/TDS reveals changes in
dissolved solids; it cannot identify lead, PFAS, bacteria, or pesticides.
Nothing in this build should be represented as certifying potability.

The house currently has **no whole-home filtration or softener**. That
single fact drives the entire cost reduction: every dual-channel item in
the original proposal exists to measure treatment efficacy, and there is
nothing yet to measure across.

The design therefore splits into:

- **Phase 1 (build now):** pressure, flow, temperature, leak detection.
- **Phase 2 (on filter install):** conductivity, differential pressure.
- **Phase 3 (optional):** motorized shutoff, pH.

Phase 1 physically pre-plumbs every Phase 2 port so the later work is a
screwdriver job, not a re-pipe.

### What Phase 1 actually buys you

With no treatment to monitor, the system's job is:

| Capability | Value |
|---|---|
| Overnight/no-occupancy flow detection | Catches slab leaks, running toilets, irrigation faults |
| Static pressure trending | Detects PRV drift/failure before it damages fixtures |
| Thermal expansion spike detection | Closed system + water heater can spike >100 psi; kills expansion tank and appliances silently |
| Total/daily consumption | Baseline for anomaly alerts and utility bill reconciliation |
| Point leak detection at equipment | Immediate local alarm, HA-independent |

These findings are worth more today than any TDS number on untreated
municipal supply, where conductivity is essentially a flat line.

---

## 2. Changes from the original proposal

| Original | Revised | Saving | Rationale |
|---|---|---|---|
| 2 × Atlas EZO K 1.0 conductivity | Deferred to Phase 2; sample ports pre-plumbed | −$460 | No pre/post pair exists to compare; absolute TDS on municipal supply is not actionable |
| 2 × probe flow cells / manifold | Deferred | −$80–150 | Follows the above |
| 2 × 4–20 mA industrial transmitters | 2 × 0–100 psi ratiometric transducer (304SS, ¼" NPT) | −$120–250 | Proven pattern already running on the pool filter monitor |
| Isolated 4–20 mA input modules | ADS1115 16-bit I²C ADC | −$35–75 | Removes ESP32 ADC nonlinearity without loop-powered hardware |
| Stainless thermowell | DS18B20 strapped to pipe under insulation | −$35 | Tracks water temp within ~1 °C at steady flow; one fewer wetted part |
| RS-485 interface | UART broken out to terminal block; MAX485 added if needed | −$25 | $6 part, zero reason to buy it in advance |
| pH channel | Removed from roadmap until a well/neutralizer/RO exists | −$100+ | Requires calibration, must stay wet, finite probe life, negligible value on municipal supply |
| ESP32 Ethernet board | ESP32-S3 N16R8 on Wi-Fi, IoT VLAN | −$20 | Matches existing fleet; Ethernet remains an option |
| IP65 industrial enclosure | IP54 indoor enclosure w/ DIN rail | −$40 | Indoor mechanical-room install |

**Result: ~$1,125–1,750 → ~$350–450 in Phase 1 parts.**

### Deliberately *not* cut

- **Flow meter.** A 1" full-port positive-displacement meter with pulse
  output ($110–160). The $20 Hall turbines are undersized for a 1" main,
  create measurable pressure loss, and lack documented potable-water
  wetted materials. This is the sensor that pays for itself.
- **Buying both pressure transducers now** even though only one is wired.
  They are $18 each and matched-batch units drift together, which is what
  makes a future ΔP number honest.
- **Isolation valves and unions** at every sensor location.

---

## 3. System layout

```
Water main (from meter / PRV)
   │
   ├── ISO valve ─── 1" tee w/ ¼" FNPT boss ──► P1  (pressure, wired Phase 1)
   │
   ├── 1" tee w/ ¼" ball valve, CAPPED ──────► raw conductivity sample (Phase 2)
   │
   ├── DS18B20 strapped to pipe under insulation
   │
   ├── 1" pulse-output PD meter, unions both sides
   │
   ▼
[ FUTURE FILTER / SOFTENER FOOTPRINT — unions pre-installed both ends ]
   │
   ├── 1" tee w/ ¼" FNPT boss, CAPPED ───────► P2  (pressure, Phase 2)
   │
   ├── 1" tee w/ ¼" ball valve, CAPPED ──────► treated conductivity sample (Phase 2)
   │
   ▼
House distribution
```

Rope leak sensors run along the floor beneath the equipment area and the
water heater pan.

### Pre-plumb checklist (this is what makes Phase 2 cheap)

- [ ] Two capped ¼" sample tees with ball valves — one upstream, one at the
      future filter outlet position
- [ ] Spare ¼" FNPT boss downstream for P2
- [ ] Unions bracketing the reserved filter footprint (min. 24" of straight
      run, verify against candidate filter body dimensions before soldering)
- [ ] Conduit or smurf tube from equipment area to enclosure with a pull
      string **and** one spare 4-conductor already run
- [ ] Isolation valves upstream and downstream of the whole assembly

---

## 4. Phase 1 bill of materials

| Item | Qty | Est. cost |
|---|---|---|
| ESP32-S3-N16R8 | 1 | $15 |
| ADS1115 16-bit I²C ADC breakout | 1 | $6 |
| 0–100 psi ratiometric transducer, 304SS, ¼" NPT, 0.5–4.5 V | 2 | $36 |
| 1" full-port pulse-output PD water meter (reed or Hall) | 1 | $110–160 |
| DS18B20 waterproof probe + pipe insulation | 1 | $8 |
| Rope-style leak sensor | 2 | $20 |
| Piezo buzzer + driver transistor | 1 | $5 |
| 12 V 2 A PSU + buck converter to 5 V | 1 | $18 |
| IP54 enclosure, DIN rail, terminal blocks, glands | 1 | $45 |
| Plumbing: 1" tees w/ ¼" bosses, ball valves, unions, ISO valves, adapters | — | $90–140 |
| Wire, fuse, ferrules, misc | — | $15 |
| **Phase 1 total** | | **~$368–468** |

### Phase 2 (at filter install)

| Item | Est. cost |
|---|---|
| Atlas Scientific EZO-EC circuit + K 1.0 probe (×1, see below) | $230 |
| Probe flow cell + needle valve + 3-way sample valve | $70–110 |
| Calibration solutions | $30 |
| Wire second pressure transducer (already purchased) | $0 |
| MAX485 breakout, if industrial probes chosen instead | $6 |
| **Phase 2 total** | **~$330–370** |

**Single-probe alternating design.** Rather than two conductivity channels,
run **one** EZO K 1.0 behind a 3-way sample valve (manual or a small
motorized valve) that alternates raw/treated on a 15-minute cycle.
Conductivity baselines move over days, so simultaneity buys nothing.
Saves $230 and leaves one probe to calibrate instead of two.

### Phase 3 (optional)

| Item | Est. cost |
|---|---|
| 1" motorized ball valve, 12 V, w/ manual override | $60–90 |
| Independent leak-latch relay (HA-independent trip path) | $15 |

---

## 5. Electrical design

- ADS1115 powered from the **5 V rail** so 0.5–4.5 V transducer output can
  be read directly at gain `6.144` with no divider. SDA/SCL pulled up to
  3.3 V (open-drain, safe with 3.3 V ESP32 logic).
- Pressure transducers are ratiometric to their 5 V supply. Use a
  well-regulated buck; supply ripple appears directly as pressure noise.
- Pulse input: internal pull-up, hardware `pulse_counter` peripheral,
  100 ms internal filter for reed-switch bounce.
- Leak inputs: dry contact to GND, internal pull-up, `inverted: true`.
- Buzzer is driven by an ESPHome `on_press` automation on the leak sensor,
  so it fires with Home Assistant offline or unreachable.
- Reserved on the terminal block: UART TX/RX/3V3/GND for Phase 2 Atlas or
  MAX485; ADS1115 channels A1–A3 (A1 = P2).
- Fuse the 12 V input. Ground the enclosure. Do not bond the water piping
  to the enclosure ground.

---

## 6. Home Assistant entities

Phase 1:

```
sensor.water_pressure_inlet
sensor.water_pressure_static_24h_max
sensor.water_temperature
sensor.water_flow_rate
sensor.water_total_gallons
sensor.water_daily_gallons
binary_sensor.water_leak
binary_sensor.water_flowing
binary_sensor.water_overnight_flow_alert
binary_sensor.water_pressure_abnormal
```

Phase 2 adds:

```
sensor.water_pressure_outlet
sensor.water_filter_pressure_drop
sensor.water_raw_conductivity
sensor.water_treated_conductivity
sensor.water_treatment_efficiency
sensor.water_filter_gallons
binary_sensor.water_filter_replacement_due
```

---

## 7. Alert logic

Baseline-derived, not absolute limits.

**Continuous-flow leak.** Flow > 0.1 GPM sustained for 30 minutes with no
fixture-scale variation → warning. Any flow at all between 02:00 and 05:00
for more than 15 continuous minutes → alert. Suppress during irrigation
windows (see `smart-irrigation` interlock note below).

**Catastrophic flow.** Flow > 12 GPM sustained 5 minutes → high-priority
alert; this is the trigger that later drives the Phase 3 shutoff valve.

**Static pressure.** Measure with zero flow.
- Below 40 psi → supply problem or partially closed valve
- Above 80 psi → PRV out of adjustment; above 100 psi → PRV failed or
  expansion tank waterlogged, act immediately
- Rising trend of the 24 h static maximum over weeks → PRV drifting

**Thermal expansion.** Pressure rise > 20 psi with zero flow within 60
minutes of water-heater recovery → expansion tank has lost its air charge.

**Temperature.** Below 40 °F at the main → freeze risk. Above 80 °F on the
cold line → unusual, worth investigating.

**Leak sensors.** Immediate; local buzzer plus HA notification, no delay
and no debounce beyond 1 second.

**Phase 2, differential pressure.** Alert when P1 − P2 > 10 psi during
sustained flow above ~3 GPM, with the final threshold taken from the
filter manufacturer's rated ΔP. Track gallons since replacement, days
since replacement, and maximum observed ΔP; replace on whichever comes
first.

**Phase 2, conductivity.** Alert when treated conductivity exceeds the
30-day baseline by more than 25% for 15 minutes while flowing, or when raw
conductivity moves more than 30% from baseline. For RO, track
rejection % = (1 − product EC ÷ feed EC) × 100.

---

## 8. Integration notes

- Device belongs on the **IoT VLAN** (VLAN 10 on this network) with the
  pool filter monitor. Same ESPHome pattern, same naming convention.
- Firewall must permit the ESPHome API from the IoT VLAN to the Home
  Assistant host only.
- Irrigation interlock: if the Meshtastic/ET irrigation project drives
  zones, publish an `input_boolean` that suppresses overnight-flow alerts
  during scheduled watering rather than hard-coding a time window.
- Add the device to NetBox with role `iot` and tag `water`.

---

## 9. Safety and materials notes

- Anything in contact with potable water should have documented wetted
  materials, and ideally NSF/ANSI 61 or 372 certification. The inexpensive
  transducers specified here are typically 304 stainless diaphragm on a
  ¼" NPT fitting; **verify the wetted-material datasheet before installing**
  and prefer a stainless or lead-free brass adapter, not standard brass.
- Every electronic sensor sits behind an isolation valve so it can be
  serviced without shutting down the house.
- The Phase 3 motorized valve should be an independent safety subsystem
  with a manual override lever — not a single ESP32 GPIO as the only path
  to closing the main.
- Pressure-test the assembly before energizing anything electrical, and
  leave it under observation for 24 hours before closing up the wall or
  insulating.
