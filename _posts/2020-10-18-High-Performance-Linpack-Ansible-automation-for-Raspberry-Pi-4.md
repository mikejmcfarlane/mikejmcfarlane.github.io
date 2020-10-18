---
layout: post
title: High Performance Linpack Ansible automation for the Raspberry Pi 4
date: 2020-10-18
---

- [Introduction](#introduction)
- [Ansible automation to build HPL](#ansible-automation-to-build-HPL)
  - [Automation overview](#automation-overview)
  - [How to run the automation](#how-to-run-the-automation)
  - [Some points for thought found with the automation](#some-points-for-thought-found-with-the-automation)
- [Testing](#testing)
  - [Testing BLAS versions](#testing-BLAS-versions)
  - [Testing problem size in relation to memory size and clock speed](#testing-problem-size-in-relation-to-memory-size-and-clock-speed)
- [Power usage](#power-usage)
- [Conclusions](#conclusions)

# Introduction

I recently added a fourth node to my High Performance Linpack (HPL) Raspberry Pi 4 cluster. Building 3 nodes by hand was getting a little boring, and likely to cause mistakes and inconsistencies between nodes. So time for a little automation.
In this post I will give an overview of the automation, so problems that I encountered particularly in regards to cluster performance, and report on some test results with the new 4 node cluster.

# Ansible automation to build HPL

## Automation overview

[Ansible](https://www.ansible.com) is one of my favourite automation tools. It's easy to learn, easy to hack something together quickly or teach to others, and it scales well to mid size environments. Large enterprise environments with hundreds to thousands of nodes to manage may have better tools (up for debate) but for small environments like a typical Pi cluster with tens of nodes it's ideal.

If you are new to Ansible here are a couple of good guides to Ansible to help you understand the automation here, otherwise I've assumed you know some Ansible.

* [Get started with Ansible](https://www.ansible.com/resources/get-started)
* [How to Use Ansible: A Reference Guide](https://www.digitalocean.com/community/cheatsheets/how-to-use-ansible-cheat-sheet-guide)

## How to run the automation

The automation is still a work in progress. Whilst it works reliably on my cluster and Ansible node, I've not put much effort in to making it generic. Also, as there are quite a lot of different configurations, I haven't made this a single line with arguments approach, so there are a few lines required to do most tasks e.g. setup a new node, configure and build HPL, or wipe an existing one. Also there is some reliance on an NFS mount (from a Synology Diskstation).

Control of the various playbooks is mostly using:

- [Ansible tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html)
- [Ansible group vars file](https://github.com/mikejmcfarlane/ansible_pi/blob/master/group_vars/all.yml)

Here is my [ansible_pi repo](https://github.com/mikejmcfarlane/ansible_pi). The [readme](https://github.com/mikejmcfarlane/ansible_pi/blob/master/README.md) covers most options and how to run them.

Broadly the instructions and Ansible playbooks cover:

- Setting up a Pi as an Ansible master node for running the automation (I mostly work from an iPad Pro so have a Pi I use for linux things.)
- How to configure a new Pi with some basics to allow Ansible automation.
- Setting up PXE boot. Check out my [previous blog post on setting up PXE boot on a Synology Diskstation](https://mikejmcfarlane.github.io/blog/2020/09/12/PXE-boot-raspberry-pi-4-from-synology-diskstation).
- Setting up, building and running High Performance Linpack on a Pi cluster. This includes discrete tasks for various BLAS libraries including building OpenBLAS and ATLAS. The ATLAS build is about 8 hours in a Pi, so I built it once and stuck the libraries on an NFS mount. More about [building ATLAS on another blog post](https://mikejmcfarlane.github.io/blog/2020/09/17/High-Performance-Linpack-for-raspberry-pi-supercomputer).

There is also a [Jupyter notebook](https://github.com/mikejmcfarlane/pi_notebooks) that apart from all the graphs and data has some useful tools for setting up HPL runs e.g. suggesting suitable values for N and NB based on the cluster size and amount of memory per node.

## Some points for thought found with the automation

- nodes file - I found out that the order of the IP addresses in the `nodes-Xpi` file matters to performance. In an early version of the automation a loop in the code put the IP addresses as groups like `1.1.1.1, 2.2.2.2, 3.3.3.3,1.1.1.1, 2.2.2.2, 3.3.3.3` but for optimum performance they need to be grouped like `1.1.1.1, 1.1.1.1, 2.2.2.2, 2.2.2.2, 3.3.3.3, 3.3.3.3`.
- PXE boot - In the previous post I found that a 3 node cluster was reasonably performant relative to microSD, but for this 4 node cluster I started to find a larger load average for each node and a resulting significant drop in performance. I didn't fully bottom this out, and discovered some other performance issues (more below), so need to follow this up as don't think the PXE boot should have been that much slower.
- SD card failure - during testing one of my Sandisk Extreme microSD cards failed. It wasn't that old, and I haven't done a huge amount of testing with it, and once the HPL tests are running there is little disk activity, so should have been no problems. Maybe I just got a bad card.
- MPI channels - Message Passing Interface has a new faster communication protocol called channel 4. I noticed some issues with this protocol so dropped back to channel 3 with the Nemesis protocol. This needs further investigation. You can re-configure the automation to build with channel 4 in the Ansible group vars file.
- build not performance repeatable - in addition to the other performance issues I also experienced a significant performance drop from some other factor. At one point I was hitting 4.62Gflops, but then inexplicably after a rebuild the performance dropped to around 4.22Gflops. I've not been able to resolve this despite completely rebuilding the cluster. I suspect perhaps an OS or library upgrade in `apt get upgrade` step, or how that interacts with built ATLAS which seems the most affected, in the automation may have changed something I wasn't aware of. Spent quite a bit of time on this, and it's just time to get a blog post out, so one for another day or a better brain!

# Testing

I had planned to test quite a number of factors including the effect of PXE boot with link aggregation on the Synology Diskstation, and the effects of the different MPI channel protocols, but have spent too long debugging the automation, and I'm keen to be moving on to explore docker clusters, so am just presenting the results of the code tests that I wanted to run.

## Testing BLAS versions

Reading up on Pi clusters indicated a variety of BLAS libraries in use, so this seemed like a good place to start. The options explored were ATLAS from the Raspbian repo, build ATLAS from source and build OpenBLAS from source.


## Testing problem size in relation to memory size and clock speed

Increasing the memory size allows a larger problem size to be loaded, but what is the actual effect of that. In particular, is it worth buying the 8GB memory Pi compared to the 4GB memory model? Lastly, what is the effect of overclocking?

![Is more memory worth it in a Pi HPL cluster?]({{ site.url }}/assets/ansible_hpl/Is_more_memory_worth_it.png)

# Power usage

I used the power meter on my APC UPS to measure the approximate power drawn by the Pi cluster under various conditions. Base power covers the networking and file server with no Pis booted. Subsequent values show as before with the cluster booted (but not overclocked) with no load and HPL load.

![Approximate power usage in a Pi HPL cluster?]({{ site.url }}/assets/ansible_hpl/pi_cluster_power_usage.png)

# Conclusions




