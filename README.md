# water-monitor

Whole-house water monitoring station for Empire12 — ESP32-S3 + ESPHome →
Home Assistant.

Measures inlet pressure, flow, temperature and point leaks on an untreated
municipal supply, and is pre-plumbed to add differential pressure and
conductivity when a whole-home filter or softener is installed.

**Phase 1 parts cost: ~$370–470.** See [`docs/SPEC.md`](docs/SPEC.md) for
the full design, bill of materials, alert logic and the reasoning behind
each cost reduction from the original $1,125–1,750 proposal.

> This is a change-detection and infrastructure-health system. It does not
> certify drinking-water safety. Conductivity cannot detect lead, PFAS,
> bacteria or pesticides.

## Layout

```
docs/SPEC.md                        design, BOM, alert thresholds
docs/PLUMBING.md                    install order and pre-plumb checklist
esphome/water-monitor.yaml          device config (pin map lives here)
esphome/packages/pressure.yaml      P1 via ADS1115 A0; P2 stubbed
esphome/packages/flow.yaml          pulse counter + totalizer
esphome/packages/temperature.yaml   DS18B20 strap-on
esphome/packages/leak.yaml          rope sensors + local buzzer
esphome/packages/conductivity.yaml  Phase 2, fully commented out
homeassistant/packages/             utility meters, statistics, automations
homeassistant/lovelace/             dashboard
```

## Quick start

```bash
cp esphome/secrets.yaml.example esphome/secrets.yaml
# fill in wifi + generate an API key:
openssl rand -base64 32

esphome config esphome/water-monitor.yaml     # validate
esphome run esphome/water-monitor.yaml        # flash over USB the first time
```

Then copy `homeassistant/packages/water_monitor.yaml` into the HA
`config/packages/` directory and restart Home Assistant.

## Commissioning order

1. Flash and boot on the bench with no sensors. Confirm Wi-Fi, API, OTA.
2. Attach the DS18B20, read the ROM address from the log, pin it in
   `packages/temperature.yaml`.
3. Attach the ADS1115 and one transducer to a **hose bib** before touching
   the main — this validates the whole pressure chain with zero plumbing
   risk and characterizes your static supply pressure for a few days.
4. Pressure-test the plumbing assembly. Observe 24 hours before insulating.
5. Wire the flow meter last; confirm `pulses_per_gallon` against the meter's
   mechanical register over a known 10-gallon draw.
6. Let it run two weeks before enabling alerts, so the baselines mean
   something.

## Network

Static IP on the IoT VLAN (VLAN 10 on this network), set in
`esphome/secrets.yaml`. The firewall should allow the ESPHome API from this
host to the Home Assistant host only. Add it to your inventory/IPAM with
role `iot`, tag `water`.

## Phase 2

When a filter or softener is installed:

- Wire the second transducer (already purchased) to ADS1115 A1 and
  uncomment the Phase 2 block in `packages/pressure.yaml`.
- Add one Atlas EZO-EC K 1.0 behind a 3-way sample valve and uncomment
  `packages/conductivity.yaml` in the main config's `packages:` list.

## License

MIT
