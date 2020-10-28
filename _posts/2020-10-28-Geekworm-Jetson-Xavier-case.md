---
layout: post
title: Geekworm N100 case for Jetson Xavier mini-review
date: 2020-10-28
---

- [Introduction](#introduction)
- [Assembly](#assembly)
- [Performance](#performance)

# Introduction

I recently bought a [Jetson Xavier NX Developer Kit](https://www.nvidia.com/en-gb/autonomous-machines/embedded-systems/jetson-xavier-nx/), and it's a super cool piece of kit. Lots of projects in mind involving my Mindstorms robots, but first I'm going to add it in to the Raspberry Pi based Kubernetes cluster I'm building.
Whilst the Jetson looks very cool sitting on my desk, it's an expensive bit of kit to have sat unprotected on a studio/work desk, so a case was required. There isn't a lot of choice out there, so I quickly came to the [Geekworm N100 Metal Case](https://geekworm.com/products/geekworm-nvidia-jetson-nano-metal-case-with-power-reset-control-switch). I got the [case](https://www.amazon.co.uk/gp/product/B07RSVBHW1/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1) and the required [Geekworm IPEX MHF4 Antenna](https://www.amazon.co.uk/gp/product/B08B1G6VZP/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1).

# Assembly

![geekworm n100 case parts]({{ site.url }}/assets/geekworm/geekworm_n100_case_parts.jpeg)

Assembly was pretty straight forward, and all the parts plus some spares are in the box. There are even some tweezers, which I didn't use! I just followed the [Geekworm video guide](https://youtu.be/7Cqr9R04htc). The only tricky part was pushing the tiny antenna connectors on to the equally tiny NVMe wifi module connectors. It's the same struggle with the wifi connectors in my Mac Mini. After struggling with it for 10 minutes I took the NVMe module out so I could place it firmly on the desk to attach the connectors and they popped straight on then.

The antenna connectors are also prone to spinning when tightening the antenna, so I might go back and try a different combination of spring washers to stop them rotating.

# Performance

![geekworm n100 on my desk]({{ site.url }}/assets/geekworm/geekworm_n100_case_on_desk.jpeg)

It looks very smart:) The quality of the case and components is really good, and the assembled unit feels solid which is what I wanted for my expensive device. The front power switch also has a useful LED.

The wifi antenna also seems to do a very good job. Speedtest.net is a crude but useful relative comparison, and the Jeston Xavier in the Geekworm case actually performed a few Mbps faster than my iPad Pro, and about as fast as a wired ethernet connection for internet access. (Setup is a Synology RT2600ac with a 4G connection from a MikroTik SXT LTE6).

![geekworm n100 speedtest]({{ site.url }}/assets/geekworm/geekworm_n100_speedtest.jpeg)

Recommended!