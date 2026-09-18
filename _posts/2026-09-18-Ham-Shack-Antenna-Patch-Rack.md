---
title: "Ham Shack Antenna Patch Rack"
layout: postx
category: ham
---

As described in the top-level [ham](/ham) page I currently have five fixed
antenna in the garage over the "shack". When I only had the single IC-705, and
fewer antenna, I started with a **Comet** 2-way switch but
quickly outgrew it. At a [Mike and Key club](https://www.mikeandkey.org/index.php)
swap meet I found a pair of [MFJ-1701](https://www.dxengineering.com/parts/mfj-1701)
(6-port) antenna selectors, which are really nice but only one input and HF only.
So, I repurposed the Comet switch and had the IC-705 go into it, then it split
with the 2m/70cm antenna on one port and an MFJ-1701 on the other. Not ideal.

## Antennas

Each antenna is mounted in the garage attic and it's cable is fed to a fixed
SO-239 mount attached to a horizontal beam. These consist of two 8" 90° angle
brackets in 3/16" stainless steel with four SO-239 pass-thru connectors. These
allow me to work in the attic space without having to have cables dangling to
the floor. I use Messi and Paoloni
[UltraFlex 10](https://messi.it/en/catalogue/50-ohm-coaxial-cables/standard-cables-list/ultraflex-10-400.htm)
for antenna to connector and
[Airborne 5](https://messi.it/en/catalogue/50-ohm-cables-ham-radio/airborne-5.htm)
for some of the lighter antenna.

> The two connector brackets came from Etsy but are no longer available. So, I
> bought 3' of 1.75"×1/8" aluminum 90 degree angle stock and cut my own lengths
> and installed SO-239 bulkhead connectors after tapping the stock. The SO-239
> external thread is 5/8"-24, so I purchased a tap and  37/64" drill bit set
> [amazon.com](https://www.amazon.com/dp/B0FPX1ZQZY) which works perfectly.

I use the UltraFlex 10 exclusively for connector to patch rack and a mix of the
UltraFlex 10 and Airborne 5 for rack to equipment runs.

## The Rack

The following schematic shows the antennas and connectors described above and
the patch rack attached now to the desk in place of the Comet and MFJ switches,
The placement of the connecting lines to the rack is not simply illustrative,
I route all antenna through the existing port on the top, all transceivers
through an existing port on the bottom, and all accessories through a slot on
the left side I made by cutting some of the ventilation slots open.

```text
   antennas ─┐ ┌─ antennas      ╮
             ┴ ┴                ├─ attic
         connectors             │
             ┬ ┬                ╯
             ╱ ╱
   meters    │ │
    ┬ ┬   ┌──┴─┴──┐
    │ └───┤ Patch ├┈┈┈ left ground bus
    └─────┤  Rack │
          └──┬─┬──┘
             │ └───┤ Transceivers
             └─────┤   Scanners
```

The 6U 10" wall mounted rack currently has three 1U inserts, each with seven
D-series connector blanks. I add good quality 50Ω D-series BNC connectors to
these so there is one row for the fixed antenna in the middle, one row for the
shack radios, and a top row for accessories. Radios and antennas are matched
with 1' or 2' RG58 BNC cables. For example, the IC-705 is often switched between
the two 2m/70cm antenna as one is used for voice (the vertical) and one for data
(the horizontal).

Here's a picture of the finished rack.

![Antenna Patch Rack](/assets/img/posts/ham-antenna-patch-rack.jpeg)

This is a map of the current connections. Note that row 2 is used for labeling,
rows 4 and 5 are currrently empty. Also, the
[NISSEI DG-503](https://www.radioddity.com/products/nissei-swr-meter) is a dual
HF and VHF/UHF SWF meter, so this allows the connection of a radio to the input
of the 503 and the 503's output to an antenna.

| Row 1 | Radio   | Row 3 | Antenna      | Row 6 | Meter          |
|-------|---------|-------|--------------|-------|----------------|
| 1     | GMRS    | 1     | GMRS         | 1     |                |
| 2     | Scanner | 2     | Scanner      | 2     |                |
| 3     | IC-705  | 3     | 20/70 (v)    | 3     |                |
| 4     | *Bench* | 4     | 20/70 (h)    | 4     | DG-503 HF in   |
| 5     | KXPA #1 | 5     | 6m           | 5     | DG-503 HF out  |
| 6     | KXPA #2 | 6     | 10m          | 6     | DG-503 VHF in  |
| 7     | sBitz   | 7     | *200W Dummy* | 7     | DG-503 VHF out |

Port 4 on the radio row connects to a BNC connector fixed to the shack bench so  
that any mobile or HT can be connected on the bench to any of the antenna for
testing. I also like having a permanent dummy load available load so I can
easily connect any radio for testing. Additionally, if I am only using one of
the antenna ports on the KXPA100 the other *must* be connected to this load.

## Parts

* 10" 6U wall mount rack [amazon.com](https://www.amazon.com/dp/B0FN7F5K9S).
* 3×10" 1U 7-port D-series blank panel [amazon.com](https://www.amazon.com/dp/B0GXZBMJFK).
* 21×50 Ohm D-series BNC connectors [amazon.com](https://www.amazon.com/dp/B0FX9B3V5P).
* 7×D-series blanks [amazon.com](https://www.amazon.com/dp/B0D2W3D638).
* 7×12" RG58 BNC patch cable [amazon.com](https://www.amazon.com/dp/B0D1BJM85Z).
* 4×24" RG58 BNC patch cable [amazon.com](https://www.amazon.com/dp/B0D1BYWSF5).
* Mini magnetic flash light [amazon.com](https://www.amazon.com/dp/B0C54ZH8WW).
* 200W Dummy load [amazon.com](https://www.amazon.com/dp/B0F6HNB73L).

For labels I use Dymo 1/2" flexible nylon as these stick way better to imperfect
surfaces and hold up better over time. Also, for cables I use "write-on" wire
zip ties [amazon.com](https://www.amazon.com/dp/B017TVXB5I) as the tab is perfect
for a small strip of the Dymo 1/2" label.
