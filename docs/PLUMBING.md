# Plumbing and install order

## Pre-plumb checklist

- [ ] Isolation valve upstream and downstream of the entire assembly
- [ ] 1" tee with ¼" FNPT boss for **P1** (wired in Phase 1)
- [ ] 1" tee with ¼" ball valve, **capped** — raw conductivity sample (Phase 2)
- [ ] Unions bracketing the reserved filter footprint; verify the straight
      run against the dimensions of the filter body you expect to buy
- [ ] 1" tee with ¼" FNPT boss, **capped** — P2 (Phase 2)
- [ ] 1" tee with ¼" ball valve, **capped** — treated sample (Phase 2)
- [ ] Unions on both sides of the flow meter so it can be pulled for service
- [ ] Conduit from the equipment area to the enclosure with a pull string
      **and** one spare 4-conductor already run

## Order of work

1. Shut off at the street, open the lowest fixture, drain down.
2. Dry-fit the whole assembly on the bench first. Measure the total length
   against the available run before cutting anything.
3. Solder or press-fit with all sensors removed — heat kills transducers
   and DS18B20 probes.
4. Cap every Phase 2 port. Do not leave a threaded boss open behind a wall.
5. Pressure test. Observe for 24 hours before energizing electronics,
   insulating, or closing anything up.
6. Install sensors, then insulate over the DS18B20.

## Materials caution

Anything touching potable water should have documented wetted materials
and ideally NSF/ANSI 61 or 372 certification. The inexpensive transducers
in the BOM are typically 304 stainless diaphragm on a ¼" NPT fitting —
**check the datasheet before installing** and use a stainless or lead-free
brass adapter rather than standard brass.

## Service

Every electronic sensor sits behind an isolation valve. Nothing in this
build should require shutting down the house to replace.
