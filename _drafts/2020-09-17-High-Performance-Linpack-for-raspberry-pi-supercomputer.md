---
layout: post
title: High Performance Linpack for my Raspberry Pi supercomputer
date: 2020-09-17
---

- [Introduction](#introduction)
- [The build](#the-build)
- [Installing MPI (Message Passing Interface)](#installing-mpi-message-passing-interface)
- [Installing ATLAS linear algebra library](#installing-atlas-linear-algebra-library)
- [Installing High Performance Linpack on a Raspberry Pi](#installing-high-performance-linpack-on-a-raspberry-pi)
- [My test methodology with Linpack](#my-test-methodology-with-linpack)
- [Test results](#test-results)
- [Useful resources](#useful-resources)

## Introduction

I've always wanted to build a cluster computer. There is something exotic about it. Ever since I heard of [Beowulf clusters](https://beowulf.org/overview/faq.html) I've felt some geek need to build a cluster or a supercomputer.
And the new Raspberry Pi 4s with faster 1.5GHz CPU and up to 8GB RAM made me think I was in with a chance of building something fast. As it turns out the 3 node cluster I built would have been in the [Top500](https://www.top500.org) in [2003](http://hpl-calculator.sourceforge.net/hpl-calculations.php) but that's ok. I had fun.
For my simple cluster all I needed was more than 1 Pi, a fast network switch and some software to link the Pis over the network and to test performance.

## The build

2x Raspberry Pi 4 8GB RAM
1x Raspberry Pi 4 4GB RAM, this lower RAM model limited performance but it's what I had
1x Netgear GS305P switch
3x Cat6 network cable (plus another to connect it to the router for SSH access to the Pis)
3x GeeekPi Raspberry Pi ICE Tower Cooler, superior cooling particularly with Arctic MX-4 thermal paste, so pretty!

Various Pi storage options (microSD and PXE boot) for the OS, see results below.

![RaspberryPi_4_with_GeeekPi_ICE_Tower_Cooler]({{ site.url }}/assets/RaspberryPi_4_with_GeeekPi_ICE_Tower_Cooler.jpeg)

The Pi were connected to the switch in a hub and spoke pattern.

## Installing MPI (Message Passing Interface)

blah

## Installing ATLAS linear algebra library

blah

## Installing High Performance Linpack on a Raspberry Pi

blah

## My test methodology with Linpack

blah

## Test results

blah

## Useful resources

blah