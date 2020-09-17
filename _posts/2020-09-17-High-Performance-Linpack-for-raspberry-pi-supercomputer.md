---
layout: post
title: High Performance Linpack for my Raspberry Pi supercomputer
date: 2020-09-17
---

- [Introduction](#introduction)
- [The build](#the-build)
- [Initial setup](#initial-setup)
- [Installing MPI (Message Passing Interface)](#installing-mpi-message-passing-interface)
- [Disable CPU throttling](#disable-cpu-throttling)
- [Installing ATLAS linear algebra library](#installing-atlas-linear-algebra-library)
- [Overclocking = fail! Huh!](#overclocking--fail-huh)
- [Installing High Performance Linpack on a Raspberry Pi](#installing-high-performance-linpack-on-a-raspberry-pi)
- [Adding nodes](#adding-nodes)
- [SSH access between nodes](#ssh-access-between-nodes)
- [Benchmarking a single node](#benchmarking-a-single-node)
- [Benchmarking multiple nodes](#benchmarking-multiple-nodes)
- [My test methodology with Linpack](#my-test-methodology-with-linpack)
  - [Test 1:  Vary P and Q, for both PXE boot and microSD card storage.](#test-1-vary-p-and-q-for-both-pxe-boot-and-microsd-card-storage)
  - [Test 2: Vary block size, using microSD storage as it was slightly faster in Test 1.](#test-2-vary-block-size-using-microsd-storage-as-it-was-slightly-faster-in-test-1)
- [Test results](#test-results)
- [Next steps](#next-steps)
- [Useful resources](#useful-resources)
- [Some useful commands](#some-useful-commands)

## Introduction

I've always wanted to build a cluster computer. There is something exotic about it. Ever since I heard of [Beowulf clusters](https://beowulf.org/overview/faq.html) I've felt some geek need to build a cluster or a supercomputer.
And the new Raspberry Pi 4s with faster 1.5GHz CPU and up to 8GB RAM made me think I was in with a chance of building something fast. As it turns out the 3 node cluster I built would have been in the [Top500](https://www.top500.org) in [2003](http://hpl-calculator.sourceforge.net/hpl-calculations.php) but that's ok. I had fun.
For my simple cluster all I needed was more than one Pi, a fast network switch and some software to link the Pis over the network and to test performance.

## The build

* 2x Raspberry Pi 4 8GB RAM
* 1x Raspberry Pi 4 4GB RAM, this lower RAM model limited performance but it's what I had
* 1x Netgear GS305P switch
* 3x Cat6 network cable (plus another to connect the switch to the router for SSH access to the Pis)
* 3x GeeekPi Raspberry Pi ICE Tower Cooler, superior cooling particularly with Arctic MX-4 thermal paste, so pretty!

Various Pi storage options (microSD and PXE boot) for the OS, see results below.

![RaspberryPi_4_with_GeeekPi_ICE_Tower_Cooler]({{ site.url }}/assets/RaspberryPi_4_with_GeeekPi_ICE_Tower_Cooler.jpeg)

The Pi were connected to the switch in a hub and spoke pattern.

On the thermal paste, it really makes a difference compared to thermal stickers that are supplied with many coolers and heatsinks. Good application of the paste also makes a difference. From thermal stickers to thermal paste is worth approx 5-10ºC, where the poor application is worth approx 5ºC and good application is worth 10ºC. Poor application is basically just not enough! Typically during the benchmarking the CPUs never went over 40ºC with the ICE Tower Cooler and thermal paste.

## Initial setup

You need these packages to make [MPI](http://www.mpich.org/), [ATLAS](https://sourceforge.net/projects/math-atlas/) and [HPL High Performance Linpack](http://www.netlib.org/benchmark/hpl/). Whilst in theory you can install a linear algrebra (BLAS) library and MPI from the Raspbian repos I found that making ATLAS is much faster (on a single node, repo ATLAS gives about 5Gflops, making ATLAS from source gives about 8Gflops). I also tried OpenBLAS which builds a lot quicker than ATLAS, but then I couldn't build HPL. So we are going to make MPI, ATLAS and HPL.

Either way, you need to make HPL so:

```
sudo apt install gfortran automake
```

## Installing MPI (Message Passing Interface)

The Message Passing Interface handles the task of communicating over the network between the nodes.

```
mkdir ~/tmp
cd ~/tmp
wget https://www.mpich.org/static/downloads/3.4a3/mpich-3.4a3.tar.gz
tar xzvf mpich-3.4a3.tar.gz
cd mpich-3.4a3
./configure
make -j 4
sudo make install
```

## Disable CPU throttling

The Pi doesn't run full time at 1.5GHz, it usually only bursts to this on demand, so it's worth setting the CPU to the maximum full time as it's worth about 1% improvement in the benchmarks. You also need it set to build and tune ATLAS, so:

```
echo performance | sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

This is reset to `ondemand` on reboot, so ensure you set it each time you boot before running benchmarking tests.

## Installing ATLAS linear algebra library

It's a maths library, I'm not smart enough to know what it does, let alone explain it, so let's get it built:
```
cd ~/tmp
wget https://sourceforge.net/projects/math-atlas/files/Stable/3.10.3/atlas3.10.3.tar.bz2
tar xjvf atlas3.10.3.tar.bz2
mkdir -p ~/tmp/atlas-build
cd ~/tmp/atlas-build
../ATLAS/configure
make
```

This one takes a while to build, like 8 hours, so set _make_ going and come back tomorrow! Don't parallelise with the _j_ option as it messes with the ATLAS tuning.

## Overclocking = fail! Huh!

The ATLAS build was so slow I thought I would stop it, [overclock the Pi](https://www.tomshardware.com/uk/reviews/raspberry-pi-4-b-overclocking,6188.html) and restart _make_. I tried adding the following to `/boot/config.txt`:

```
over_voltage=6
arm_freq=2000
```
and when ATLAS build crashed and rebooted the Pi I tried
```
over_voltage=2
arm_freq=1750
```
which still caused the Pi to crash. I then built ATLAS at the normal clock frequency and tried running HPL overclocked and that just crashed. I'm using good quality Pi power supplies so doubt it is that. So I admitted defeat and ran with the standard CPU frequency and voltage.

## Installing High Performance Linpack on a Raspberry Pi

This is the actual benchmarking software, build it with:

```
cd ~/tmp
wget http://www.netlib.org/benchmark/hpl/hpl-2.3.tar.gz
tar xzvf hpl-2.3.tar.gz
cd hpl-2.3
cd setup
sh make_generic
cp Make.UNKNOWN ../Make.rpi
cd ..
vi Make.rpi
```

Most of _Make.rpi_ is ok, you just need to edit a few lines to point to the tools we have just built:

```
# ----------------------------------------------------------------------
# - Platform identifier ------------------------------------------------
# ----------------------------------------------------------------------
#
ARCH         = rpi
#
# ----------------------------------------------------------------------
# - HPL Directory Structure / HPL library ------------------------------
# ----------------------------------------------------------------------
#
TOPdir       = $(HOME)/tmp/hpl-2.3
INCdir       = $(TOPdir)/include
BINdir       = $(TOPdir)/bin/$(ARCH)
LIBdir       = $(TOPdir)/lib/$(ARCH)
#
HPLlib       = $(LIBdir)/libhpl.a 
#
# ----------------------------------------------------------------------
# - Message Passing library (MPI) --------------------------------------
# ----------------------------------------------------------------------
# MPinc tells the  C  compiler where to find the Message Passing library
# header files,  MPlib  is defined  to be the name of  the library to be 
# used. The variable MPdir is only used for defining MPinc and MPlib.
#
MPdir        = /usr/local 
MPinc        = -I /usr/local/include
MPlib        = /usr/local/lib/libmpich.so
#
# ----------------------------------------------------------------------
# - Linear Algebra library (BLAS or VSIPL) -----------------------------
# ----------------------------------------------------------------------
# LAinc tells the  C  compiler where to find the Linear Algebra  library
# header files,  LAlib  is defined  to be the name of  the library to be 
# used. The variable LAdir is only used for defining LAinc and LAlib.
#
LAdir        = /home/pi/tmp/atlas-build
LAinc        = 
LAlib        = $(LAdir)/lib/libf77blas.a $(LAdir)/lib/libatlas.a
```

Finally:

```
make arch=rpi
```

## Adding nodes

When you add more nodes to the cluster, assuming they are the same hardware and OS versions, you don't need to re-run the _make_ commands. Just repeat the initial _apt install_ and _rsync_ copy the _tmp_ dir over to the new nodes. For _MPI_ re-run _sudo make install_, for _HPL_ you might need to re-run _make arch=rpi_. Other than that it should be good.

It's definitely worth having identical hardware configurations if possible. Trying to tune the problem size to fit different memory sizes is awkward and non-optimal, but it's what I had so I was happy just to explore and learn what I could and accept that I wasn't going to get maximum absolute performance. I still spent a long time tweaking though:)

## SSH access between nodes

Before you can start benchmarking, MPI needs to be able to SSH from node to node. Each node needs a key, and the key needs copied to the other nodes. Check out [How to Set Up SSH Keys on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804) but broadly on each node you need to:

```
ssh-keygen
ssh-copy-id pi@<IP or hostname for every node>
ssh pi@<IP or hostname for every node>
```

## Benchmarking a single node

To benchmark a single node:

```
cd ~/tmp/hpl-2.3/bin/rpi
vi nodes-1pi
```

and for each CPU core you want to run enter either _localhost_ or that nodes IP address. This example will use all 4 cores on a single Raspberry Pi 4 node whose IP address is _192.168.1.147_:

```
192.168.1.147
192.168.1.147
192.168.1.147
192.168.1.147
```

then setup _HPL.dat_ in the same directory, here is an example:

```
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
5120         Ns
1            # of NBs
128          NBs
0            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
2            Ps
2            Qs
16.0         threshold
1            # of panel fact
2            PFACTs (0=left, 1=Crout, 2=Right)
1            # of recursive stopping criterium
4            NBMINs (>= 1)
1            # of panels in recursion
2            NDIVs
1            # of recursive panel fact.
1            RFACTs (0=left, 1=Crout, 2=Right)
1            # of broadcast
1            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
1            # of lookahead depth
1            DEPTHs (>=0)
2            SWAP (0=bin-exch,1=long,2=mix)
64           swapping threshold
0            L1 in (0=transposed,1=no-transposed) form
0            U  in (0=transposed,1=no-transposed) form
1            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)
```

and run it with:

```
mpiexec -f nodes-1pi ./xhpl
```
Sit back and marvel at the sheer raw power of your single node supercomputer!

## Benchmarking multiple nodes

Need more speed? Add more nodes.

```
cd ~/tmp/hpl-2.3/bin/rpi
vi nodes-1pi
```

and add the IP or hostname for each core of each node you want to run, so in this example I only use two cores of the 3 nodes in my cluster (but really you want to run all 4 cores mostly so enter each IP 4 times):

```
192.168.1.147
192.168.1.147
192.168.1.121
192.168.1.121
192.168.1.184
192.168.1.184
```

With more nodes you can enter a bigger problem size and the values of _NB_, _P_ and _Q_ really start to matter. Here is a starter _HPL.dat_ for my 3 node cluster with a problem size that is tuned for 4GB on the 4GB Pi:

```
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
32768         Ns
1            # of NBs
224          NBs
0            PMAP process mapping (0=Row-,1=Column-major)
1           # of process grids (P x Q)
3            Ps
4            Qs
16.0         threshold
1            # of panel fact
2            PFACTs (0=left, 1=Crout, 2=Right)
1            # of recursive stopping criterium
4            NBMINs (>= 1)
1            # of panels in recursion
2            NDIVs
1            # of recursive panel fact.
1            RFACTs (0=left, 1=Crout, 2=Right)
1            # of broadcast
1            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
1            # of lookahead depth
1            DEPTHs (>=0)
2            SWAP (0=bin-exch,1=long,2=mix)
64           swapping threshold
0            L1 in (0=transposed,1=no-transposed) form
0            U  in (0=transposed,1=no-transposed) form
1            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)

```

If all three Pis had 8GB memory then I could have increased the problem size. Use _htop_ or _top_ to keep an eye on memory size during a run. You want to aim for memory usage between 75% and 85% which leaves some memory for the OS. If you run out of memory the run will fail.

then run with:

```
mpiexec -f nodes-3pi ./xhpl
```

## My test methodology with Linpack

I found the following useful to help me optimise the tests:

* [Top500 HPL Calculator](http://hpl-calculator.sourceforge.net)
* [HPL Frequently Asked Questions](http://www.netlib.org/benchmark/hpl/faqs.html)
* [HPL Tuning](http://www.netlib.org/benchmark/hpl/tuning.html)

I spent some time exploring how the values of _N_, _NB_, _P_ and _Q_ affected performance then built some simple test matrices. Luckily _HPL.dat_ allows test matrices to be run, not just single values, so it is easy to enter a full test matrix then leave it all running. For example in _HPL.dat_

```
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
32768         Ns
12            # of NBs
8 16 32 64 96 128 160 192 224 256 288 320          NBs
0            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
3          Ps
4           Qs
```
will run 12 tests, one for each NB value.

I empirically established the maximum problem size I could run given the 4GB memory Pi was _32768_ which used about 80% memory on the 4GB node, and much less on the two 8GB nodes.

The final test matrices were:

### Test 1:  Vary P and Q, for both PXE boot and microSD card storage.

```
P      :       3        4        2        6        1       12        3 
Q      :       4        3        6        2       12        1        3 
```

### Test 2: Vary block size, using microSD storage as it was slightly faster in Test 1.

```
NB     :       8       16       32       64       96      128      160      192     224      256      288      320
```

## Test results

blah

## Next steps

I'm pretty happy with what I have done here. I might automate the build so as I add more nodes it will be easier to setup. And it would be good to figure out what is going on with overclocking and OpenBLAS, to see if I can squeeze some more performance out the cluster. The overclocking issue is particularly bugging me, so will def come back to that!

## Useful resources

Thanks to these sites for giving me the info I needed.

* [HPL (High Performance Linpack): Benchmarking Raspberry PIs](https://www.howtoforge.com/tutorial/hpl-high-performance-linpack-benchmark-raspberry-pi/)
* [Building HPL and ATLAS for the Raspberry Pi](https://computenodes.net/2018/06/28/building-hpl-an-atlas-for-the-raspberry-pi/)
* [Linpack and BLAS on Wee Archie](https://www.epcc.ed.ac.uk/blog/2017/06/07/linpack-and-blas-wee-archie)
* [Performance analysis of single board computer clusters](https://www.sciencedirect.com/science/article/pii/S0167739X1833142X)
* [MPICH Installer’s Guide](https://www.mpich.org/static/downloads/3.3.2/mpich-3.3.2-installguide.pdf)

## Some useful commands

`vcgencmd measure_clock arm` - see the current CPU frequency.

`vcgencmd measure_temp` - see the current CPU temperature.

`htop` - the prettiest way to see current CPU and memory usage, and also the load average. Needs `sudo apt install htop`.