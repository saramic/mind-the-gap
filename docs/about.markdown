---
layout: page
title: About
permalink: /about/
---

**Mind the Gap** reviews the [DFRobot FireBeetle 2 ESP32-C6][firebeetle] alongside three [Fermion mmWave Human Presence Sensors (SEN0691)][sensor] — the same class of radar used to detect a person standing in the gap between a train and the platform edge.

The build puts the sensors through their paces:

- Fusing three mmWave sensors on one ESP32-C6 to cover a wider "platform edge" zone than any single sensor can see
- Characterising presence-detection latency, range, and false-positive rate against real human movement
- Power and timing budget for a battery- or rail-powered deployment

Source and logs:

- [Project repository on GitHub][repo]

[firebeetle]: https://www.dfrobot.com/product-2771.html
[sensor]: https://www.dfrobot.com/product-2793.html
[repo]: https://github.com/saramic/mind-the-gap
