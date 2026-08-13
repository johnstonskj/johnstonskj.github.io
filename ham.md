---
title: Ham Radio
layout: page
---

## Shack

```text
  left ground bus ┈┈┈┈┈┈┈┈┈┈┈┈┊┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ right ground bus
                             ┌────────────────┐
       ┌─────────────────────┤   Main Power   ├───────────────────────┐
       │              ┌──────┤    Filtered    ├────────┐              │
       │              │      └────────┬───────┘        │              │
       │              │       ┊       │                │              │
  ┌────┴─────┐     ┌──┴──┐    ┊    ┌──┴──┐          ┌──┴──┐    ┌──────┴──────┐
  │ Computer │     │ PSU │    ┊    │ PSU │          │ PSU │    │ HT charging │
  └──────────┘     └──┬──┘    ┊    └──┬──┘          └──┬──┘    └─────────────┘
                      ├──USB  ┊       ├──USB           ├──USB
                ┌─────┴────┐  ┊  ┌────┴─────┐     ┌────┴─────┐
                │  GMRS &  │  ┊  │ Elecraft │     │  IC-705  │
                │ Scanner  │  ┊  │          │     │   sBitz  │
                └──────────┘  ┊  └──────────┘     └──────────┘
```

The main power input is handled by a
[WAudio W-5900](https://waudiohifi.com/products/waudio-ac-noise-power-conditioner-mains-purifier-audio-video-noise-filter-surge-protector-with-us-standard-sockets-black)
providing 8 filtered and 4 unfiltered outlets. Three of the filtered outlets are
used for [BTech RPS-30PRO](https://baofengtech.com/product/rps-30pro/) 30 Amp
power supplies. The computer, monitors, and the chargers for various HTs and
accessories are not using filtered outlets.

I run everything off Anderson power pole connectors, so every RPS-30PRO feeds
into a [Chunzehui F-1005 9 Port 40A Power Distributor](https://www.czh-labs.com/products/chunzehui-f-1005-9-port-40a-anderson-powerpole-connector-power-splitter-distributor-source-strip-1-input-and-8-output).
Because so many small devices and tools use USB power these days I also have a
[StarTech 10-Port Industrial USB 5Gbps Hub](https://www.startech.com/en-us/usb-hubs/st1030usbm)
at each power position with a short Anderson power cable.

There are two main ground busses, large copper heavy duty, on either side of
bench with the sides slightly assymmetrical and shown with the dashed line in
the figure above. Each of the left bus is bonded to the right which is in turn
bonded to the case of the WAudio W-5900.

### IC-705

My first *grown up* radio.

* Icom [IC-705](https://www.icomamerica.com/lineup/products/IC-705/) 160m-70cm
  all-mode transceiver.
  * Icom [IC-SP3](https://www.universal-radio.com/catalog/hamhf/sp3b.html)
    speaker from a ham fleamarket.
  * VE2DX Electronics [IM1-HDMI V2](https://www.dxengineering.com/parts/vex-im1-hdmiv2)
    bluetooth CIV to HDMI meter display with [5.5" camera field monitor](https://www.amazon.com/dp/B0FN4G3NX6).
    The monitor has fantastic resolution, but more importantly has HDMI inputs
    and tripod mounts, **and** a detachable recharchable battery!
  * I Also have the VE2DX
    [IM1-4BTPLUS V2](https://www.dxengineering.com/parts/vex-im14btplusv2)
    Icom stand-alone digital meter and TrueCIV interface for mobile use,
    and a [CT17B-7DM V2](https://www.dxengineering.com/parts/VEX-CT17B-7DMV2)
    bluetooth and USB Icom interface 5-port TrueCIV data hub.
* Icom [AH-705](https://www.icomjapan.com/lineup/options/AH-705/) antenna
  tuner.
* The Peovi® [carry cage](https://peovi.com/products/ic705-tatical-cage) with
  optional *back wrap*, *front cover*, and *nato rail*. I specifically use the
  nato rail and a small tripod adapter to mount the camera monitor directly on
  top of the radio.
* Bencher [BY-1](https://www.vibroplex.com/contents/en-us/p199.html) Iambic
  Paddle, although my CW is not yet readcy for for the airwaves.

While I rarely use this radio mobile, I do have the Windcamp Touring Series
[Field Pack](https://www.windcamp.cn/productinfo/1458116.html) for it and
use that to store a number of less-frequently used accessories (the AH-705 lives
in it) and some mobile parts so that I could just grab the radio if I wanted.
Another really useful add-on from Windcamp is the
[RC-1](https://www.windcamp.cn/productinfo/23663.html) quick remove antenna
support, or the [RC-2](https://www.windcamp.cn/productinfo/732728.html)
antenna support. I also like their power cables, and have the
[power pole](https://www.windcamp.cn/productinfo/1460910.html) (with ferrites)
and [vehicle power](https://www.windcamp.cn/productinfo/23668.html?templateId=349170).

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
  * Vintage Heathkit [HS-24](https://www.radiomuseum.org/r/heath_hs_24_hs2.html) speaker.
* PX3 Panadapter
* [KXPA100](https://elecraft.com/products/kxpa100-100w-amplifier-with-optional-internal-atu)
  100w power amplifier with the optional
  [KXAT100](https://elecraft.com/products/kxat100-internal-100w-atu-kit)
  internal antenna tuner installed.
* [W2](https://elecraft.com/products/w2) Remote sensing, auto-ranging RF power
  and SWR meter with DCV/U-200 sensor for 2m/70cm at 200W.

The KX3 is a wonderful radio for mobile use, scaling from the very simple with
just the radio and AX1 or small EFHW antenna, to a the full shack experience
with PX3, KXPA100 and a good power supply.

In the shack I keep the radio and panadapter mounted a little higher off the
desk which I find more comfortable. To accomplish this I used a 5x5x2" block of
polished granite andmounted a 1" ball connector as a solid base. Using the
fantastic [KX3 mount](https://gemsproducts.com/product/kx3-mount-plate/) and
[PX3 mount](https://gemsproducts.com/product/px3-mount/) from Side KX and some
RAM arms I have a really nice, stable setup.

> The Side KX gear is so well made that after finding the mounts I also got the
> end panels for the [KX3](https://gemsproducts.com/product/kx3-end-panels-and-cover/)
> and [PX3](https://gemsproducts.com/product/px3-end-panels/), as well as well
> as the covers for the [KX3](https://gemsproducts.com/product/kx3-cover/)
> and [PX3](https://gemsproducts.com/product/px3-cover/).

When I'm mobile I have a core setup in a
[ES80](https://elecraft.com/products/es80_es80-kx3-carrying-case) with the KX3
KXPD2, mic, and the following:

* The very small
  [AX1](https://elecraft.com/collections/antennas/products/ax1-antenna) 20, 17,
  and 15 meter antenna with the
  [AXE1](https://elecraft.com/collections/ax-line-antennas-1/products/axe1_40-meter-antenna-extender)
  loading coil for the AX1 for 40m use.
* The [AXB1](https://elecraft.com/products/axb1_axb1-whip-bipod) small bipod
  mount when using the AX1 directly attached to the KX3, and the
  [AXT1](https://elecraft.com/products/axt1_axt1-tripod-adapter) adapter
  to mount AX1 on any camera trpod.
* LifePO4 battery
* Anderson power meter
* EFHW
* 75' throw line

If I am going to have the space, time, *and* power, I may also take another
[ES80](https://elecraft.com/products/es80_es80-kx3-carrying-case) for the
KXPA100 and cables as well as a (smaller)
[ES60](https://elecraft.com/collections/compact-cases/products/es60_es60-compact-padded-carrying-case-copy)
case for the PX3 and it's cables.

### sBitz

TBD.

### GMRS & Scanner

As we use GMRS for family activities and for emergency preparedness the shack
has a permanent GMRS base consisting of a Radioddity DB20-G 20W radio and
antenna.

It's nice to have a completely separate scanner while looking for activity on
the local 2m/70cm bands, and so a dedicated Radioddity DB50 with a simple but
effective discone works very well.

### Fixed Antenna

* GMRS ground base
* Discone scanner
* 2M/70cm J-Pole
* Arrow Antenna 2m/70cm J-Pole, mounted horizontally for digital modes.
* Home-made 6M wire dipole
* Radiowavz 10M Bazooka

Each antenna is mounted in the garage attic and it's cable is fed to a fixed
SO-239 mount attached to a horizontal beam. These consist of two 8" 90° angle
brackets in 3/8" stainless steel with four SO-239 pass-thru connectors. These
allow working in the attic space without having to have cables dangling to the
floor. I use Messi and Paoloni
[UltraFlex 10](https://messi.it/en/catalogue/50-ohm-coaxial-cables/standard-cables-list/ultraflex-10-400.htm)
for antenna to connector,r for some of the lighter antenna the Messi and Paoloni
[Airborne 5](https://messi.it/en/catalogue/50-ohm-cables-ham-radio/airborne-5.htm).
I use the UltraFlex 10 exclusively for connector to patch rack and a mix of the
UltraFlex 10 and Airborne 5 for rack to rig and rack to meter runs.

```text
   antennas ─┐ ┌─ antennas
             ┴ ┴
         connectors
             ┬ ┬
             ╱ ╱
   meters    │ │
    ┬ ┬   ┌──┴─┴──┐
    │ └───┤ Patch ├─ left ground bus
    └─────┤  Rack │
          └──┬─┬──┘
             │ └───┤ Transceivers
             └─────┤   Scanners
```

A 6U 10" wall mounted rack with (currently) two 1u inserts each with seven
D-series connector blanks. I add good quality 50Ω D-series BNC connectors to
these so there is one row for the fixed antenna above, and one row for the
shack radios. Radios and antennas are matched with 30cm RG58 BNC cables. For
example, the IC-705 is often switched between the two 2m/70cm antenna as one
is used for voice (the vertical) and one for data (the horizontal).

| # | Radio   | # | Antenna     | # | Meter          |
|---|---------|---|-------------|---|----------------|
| 1 | GMRS    | 1 | GMRS        | 1 |                |
| 2 | Scanner | 2 | Scanner     | 2 |                |
| 3 | IC-705  | 3 | 20/70 (v)   | 3 | *Blank*        |
|   | └──     | 4 | 20/70 (h)   | 4 | DG-503 HF in   |
| 4 | *Bench* | 5 | 6m          | 5 | DG-503 HF out  |
| 5 | KXPA #1 | 6 | 10m         | 6 | DG-503 VHF in  |
| 6 | KXPA #2 | 7 | *50W Dummy* | 7 | DG-503 VHF out |
| 7 | sBitz   |   | --          |   | --             |

Port 4 on the radio row connects to a BNC connector fixed to the shack bench so  
that any mobile or HT can be connected on the bench to any of the antenna for
testing.

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

Really useful things:

* Windcamp [AP-4](https://www.windcamp.cn/productinfo/372471.html) 4-port
  power-pole distribution in a neat flat format, I think I have one in every
  radio bag.

### Car

* Icom IC-2730B with Diamond Antenna NR770HB
* Yaesu VX-6
* GMRS

### Other Handhelds

* Baofeng F8
* Explorer QRZ-1
* Baofeng UV5R mini

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

## Field Antenna

* Buddipole
* Arrow Antenna

-----

## MeshCore

* Everyday carry node, SenseCAP TR1000-E.
* Travel carry node, Heltec MeshPocket.
* Home repeater, SenseCAP Solar Node Pro.
* Camping/mobile repeater, Heltec V4.
