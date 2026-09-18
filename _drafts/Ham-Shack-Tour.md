---
title: "Ham Shack Tour"
layout: postx
category: ham
---

I don't really have a dedicated *shack*, for my growing collection, but I have
made the space I have in my garage rather productive. I spent money mainly on an
old oak roll-top desk from Facebook marketplace and then used some old closet
shelves to make additional storage on top, and a place to screw/bolt thing
rather than do too much damage to the desk.

## The Desk "Station"

> Unfortunately, my desk is rarely clean, this photo was taken as I was
> installing some [Palomar Engineers](https://palomar-engineers.com) RFI kits to
> the three major transceivers.

![Desk](/assets/img/posts/ham-shack-desk.jpeg)

### Power

The following schematic shows the power layout, starting with the main power
filter at the top on down.

```text
  left ground bus ┈┈┈┈┈┈┈┈┈┈┈┈┊┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ right ground bus
                             ┌────────────────┐
       ┌─────────────────────┤   Main Power   ├───────────────────────┐
       │              ┌──────┤    Filtered    ├────────┐              │
       │              │      └────────┬───────┘        │              │
       │              │       ┊       │                │              │
 ┌─────┴─────┐    ┌───┴───┐   ┊   ┌───┴───┐        ┌───┴───┐   ┌──────┴──────┐
 │ Mac mini  │    │ PSU-B │   ┊   │ PSU-P │        │ PSU-B │   │ HT charging │
 ├───────────┤    └───┬───┘   ┊   └───┬───┘        └───┬───┘   └─────────────┘
 │ MBook Pro │        ├──USB  ┊       │                ├──USB
 └───────────┘  ┌─────┴────┐  ┊  ┌────┴─────┐     ┌────┴─────┐
                │  DB20-G  │  ┊  │   KX3    │     │  IC-705  │
                ├──────────┤  ┊  │   PX3    │     ├──────────┤
                │   DB50   │  ┊  │ KXPA100  │     │   sBitz  │
                └──────────┘  ┊  └──────────┘     └──────────┘
```

The main power input is handled by a
[WAudio W-5900](https://waudiohifi.com/products/waudio-ac-noise-power-conditioner-mains-purifier-audio-video-noise-filter-surge-protector-with-us-standard-sockets-black)
providing 8 filtered and 4 unfiltered outlets. Two of the filtered outlets are
used for [BTech RPS-30PRO](https://baofengtech.com/product/rps-30pro/) 30 Amp
power supplies (PSU-B). One of the filtered outlets is for a
[Powerwerx SS-30DV](https://powerwerx.com/ss30dv-desktop-dc-power-supply-powerpole)
30 Amp power supply (PSU-P). The computer, monitors, and the chargers for
various HTs and accessories are not using filtered outlets.

I run everything off Anderson power pole connectors, so the two RPS-30PRO feed
into a [Chunzehui F-1005 9 Port 40A Power Distributor](https://www.czh-labs.com/products/chunzehui-f-1005-9-port-40a-anderson-powerpole-connector-power-splitter-distributor-source-strip-1-input-and-8-output).
Because so many small devices and tools use USB power these days I also have a
[StarTech 10-Port Industrial USB 5Gbps Hub](https://www.startech.com/en-us/usb-hubs/st1030usbm)
at each power position with a short Anderson power cable.

The SS-30DV is exclusively for the Elecraft setup and has no intermediate power
distribution.

### Ground

There are two main ground busses, large copper & heavy duty, on either side of
bench with the sides slightly assymmetrical and shown with the dashed line in
the figure above. The left bus is bonded to the right which is in turn bonded to
the case of the WAudio W-5900.

Additionally, most of the spaces where I have equipment such as the shelves on
top of the desk and the stand for the IC-705 have sheets of 1mm aluminium sheet
attached.

## Antenna Storage

To store field and spare antennas I used more of my spare closet shelves and
some offcut plywood to build a box approximately 30"×12"×18", with a cover at
one end and bought some casters so I can move it around. The idea of the plywood
cover was to drill holes through it to hold individual antenna while the larger
open part held tripods and bagged antenna and accessories.

![Antenna Storage](/assets/img/posts/ham-shack-antenna-roller.jpeg)

This cover ended up more sophisticated than originally planned, I cut two pieces
of plybood, one for the cover and one smaller to fit in the floor of the box
under the cover. I then drilled a number of large diameter holes through both
so that I could pass some short lengths of plumbing tube, 1.5" and 3" as well as
some simple 1" diameter holes in both for smaller antenna. This means that
anything passed through the holes stays upright as it's captive in the bottom.
Finally, a bunch of countersunk smaller holes are useful for dropping in HT
antennas and other accessories.

In the picture the large black pipe at the back shows that the whole thing is
sturdy enough to hold my Intellitron mast, in a corner.

## Radio and Cable Storage

![Other Storage](/assets/img/posts/ham-shack-roller.jpeg)

## Attic Antennas

I have currently five fixed antenna installed in the attic over the shack, handy
but initially messy.

Attached to desk just above the antenna and equipment rollers is the patch rack
I built for connecting radios, antenna, and accessories, see the post
{% post_url 2026-09-18-Ham-Shack-Antenna-Patch-Rack %} for details.
