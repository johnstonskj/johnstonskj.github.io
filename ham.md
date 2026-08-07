---
title: Ham Radio
layout: page
---

## Shack

```text
                           ┌────────────┐
     ┌─────────────────────┤ Main Power ├────────────────────────┐
     │             ┌───────┤  Filtered  ├─────────┐              │
     │             │       └──────┬─────┘         │              │
     │             │              │               │              │
┌────┴─────┐    ┌──┴──┐        ┌──┴──┐         ┌──┴──┐    ┌──────┴──────┐
│ Computer │    │ PSU │        │ PSU │         │ PSU │    │ HT charging │
└──────────┘    └──┬──┘        └──┬──┘         └──┬──┘    └─────────────┘
                   ├──USB         ├──USB          ├──USB
              ┌────┴────┐    ┌────┴─────┐    ┌────┴─────┐
              │  GMRS & │    │ Elecraft │    │  IC=705  │
              │ Scanner │    │          │    │   sBitz  │
              └─────────┘    └──────────┘    └──────────┘
```

As much as possible all coax is Messi and Paoloni.

### GMRS & Scanner

### Elecraft

* Elecraft [KX3](https://elecraft.com/products/kx3-all-mode-160-6-m-transceiver)
  with the following installed options:
  [KXAT3](https://elecraft.com/products/kxat3-20-watt-internal-automatic-antenna-tuner)
  20w Internal antenna tuner,
  [KXBC3](https://elecraft.com/products/kxbc3-internal-nimh-charger-real-time-clock)
  NiMH battery-charger and real-time clock, and
  [KXFL3](https://elecraft.com/products/kxfl3-dual-roofing-filter-for-the-kx3)
  roofing filter.
  * [KXPD2](https://elecraft.com/collections/keyer-paddle/products/kxpd2-attached-precision-keyer-paddle)
    precision keyer paddle.
  * Mini microphone by G1JKS, from [Etsy](https://www.etsy.com/listing/4504479593/electraft-kx2-and-kx3-mini-microphone-by).
  * Old Heathkit speaker.
* PX3 Panadapter
* [KXPA100](https://elecraft.com/products/kxpa100-100w-amplifier-with-optional-internal-atu)
  100w power amplifier with the optional
  [KXAT100](https://elecraft.com/products/kxat100-internal-100w-atu-kit)
  internal antenna tuner installed.

For mobile usage:

* [AX1](https://elecraft.com/collections/antennas/products/ax1-antenna) 20, 17,
  and 15 meter antenna.
* [AXE1](https://elecraft.com/collections/ax-line-antennas-1/products/axe1_40-meter-antenna-extender)
  loading coil for the AX1 for 40m use.
* [AXB1](https://elecraft.com/products/axb1_axb1-whip-bipod) small bipod mount
  when using the AX1 directly attached to the KX3.
* [AXT1](https://elecraft.com/products/axt1_axt1-tripod-adapter) small adapter
  to mount AX1 on any camera trpod.
* [ES80](https://elecraft.com/products/es80_es80-kx3-carrying-case) for the KX3,
  KXPD2, mic, AX1 (and accessories) as well as the following:
  * ...
* [ES80](https://elecraft.com/products/es80_es80-kx3-carrying-case) for the
  KXPA100.
* [ES60](https://elecraft.com/collections/compact-cases/products/es60_es60-compact-padded-carrying-case-copy)
  for the PX3.

### IC-705

* Icom IC-705
  * Icom speaker
  * Bluetooth CIV and screen
* Bencher CW Iambic Paddle

### sBitz

### Fixed Antenna

* GMRS ground base
* Discone scanner
* 2M/70cm J-Pole
* Arrow Antenna 2m/70cm J-Pole, mounted horizontally for digital modes.
* Home-made 6M wire dipole
* 10M Bazooka

Antenna patch panel:

A 6U 10" wall mounted rack with (currently) two 1u inserts each with seven
D-series connector blanks. I add good quality 50Ω D-series BNC connectors to
these so there is one row for the fixed antenna above, and one row for the
shack radios. Radios and antennas are matched with 30cm RG58 BNC cables. For
example, the IC-705 is often switched between the two 2m/70cm antenna as one
is used for voice (the vertical) and one for data (the horizontal).

Common Usage:

| # | Radio   | # | Antenna   |
|---|---------|---|-----------|
| 1 | GMRS    | 1 | GMRS      |
| 2 | Scanner | 2 | Scanner   |
| 3 | IC-705  | 3 | 20/70 (v) |
|   | └──     | 4 | 20/70 (h) |
| 4 | KX3     | 5 | 6m        |
| 5 | sBitz   | 6 | 10m       |
|   |         | 7 | *Front*   |
| 7 | *Desk*  |   |           |

Port 7 in both inserts are special, for the antenna row it is connected to an
external SO-239 connector on the front of the garage to hook up temporary
large antenna, or the experimental gutter-tenna. On the radio front, the port
connects to a BNC connector fixed to the shack desk so that any mobile or HT can
be connected on the bench to any of the antenna for testing.

Additionally, another row provides 3 pairs of in/out connectors for meters
allowing the meters to be patched in to any radio/antenna pair as needed.

### Computer

Software

* Logging
  * RUMlogNG (macos and ios)
  * HAMRS Pro (macos and ios)
  * QSO Upload Utility
  * tsql
* Operating
  * SDR-Control -- IC-705 remote control
  * KX3UI -- KX3 remote control
  * WSPR Watch
  * MacDoppler
  * DX Toolbox
  * Ham Pulse
  * wfview
  * xCluster
  * CW Keyer
  * iDigi
  * QTH
  * QRV
  * Rotor
  * Multimode Cocoa
* Radio Tools
  * Elecraft KX3 Utility
  * K3MemoryManager
  * SignalScope X
  * Serial
* Antenna Tools
  * VNAmate
  * RF MatchMaker
  * MININEC Pro
  * SimNEC
  * RF Toolbox
  * Ham Toolbox

-----

## Mobile

### Handhelds

* Yaesu VX-6
* Baofeng F8

### Elecraft KX3 (Again)

### Lab599 TX-500

* Transceiver with battery box

### Radio Box

* zBitx HF/6m
* GMRS
* 2m/70cm

### Boafeng Repeater Box

* 2 Boafeng UV5R
* Repeater
* USB Power Bank

-----

## Antenna

* Buddipole
* Arrow Antenna

-----

## MeshCore

* Everyday carry node, SenseCAP TR1000-E.
* Travel carry node, Heltec MeshPocket.
* Home repeater, SenseCAP Solar Node Pro.
* Camping/mobile repeater, Heltec V4.
