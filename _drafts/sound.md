# Pi sound

- [Pi sound](#pi-sound)
  - [espeak text to speech](#espeak-text-to-speech)
  - [spectrum analyser](#spectrum-analyser)
  - [sonic-pi](#sonic-pi)
    - [Install](#install)
    - [sonic pi cli](#sonic-pi-cli)
      - [sonic pi cli testing](#sonic-pi-cli-testing)
  - [Making sounds and music with Pi](#making-sounds-and-music-with-pi)
    - [Background](#background)

## espeak text to speech

Ansible install

```
ansible -i hosts all -m ansible.builtin.apt -a "name=espeak state=latest" --become
```

This is a little broken as of 11/2020 [Espeak alsa issue](https://www.raspberrypi.org/forums/viewtopic.php?t=242842)

So need `aplay` e.g.

```
espeak 'Hello' --stdout | aplay
```
Or with ansible

```
ansible -i hosts all -m shell -a "espeak 'Hello' --stdout | aplay"
```

There appear to be some fixes at [After Raspbian Update, espeak issue](https://forum.dexterindustries.com/t/after-raspbian-update-espeak-issue/6811/5)

Can also install `espeak-ng`

```
sudo apt install espeak-ng
```

and run with

```
espeak-ng "hello"
```

A deep dive on some espeak options, and a python lib [How To Make Your Raspberry Pi Speak
](https://www.dexterindustries.com/howto/make-your-raspberry-pi-speak/)

## spectrum analyser

[Build a spectrum analyser with Scroll pHAT and pHAT DAC](https://learn.pimoroni.com/tutorial/sandyj/scroll-phat-spectrum-analyser)

## sonic-pi

### Install

Do NOT install with `apt` as sound does not work. Install the deb from [Get Sonic Pi for Raspberry Pi OS](https://sonic-pi.net/#rp)

If do install from `apt`, the sound will not work due to `jackd` so `killall jackd`. 

### sonic pi cli

- [Widdershin/sonic-pi-cli](https://github.com/Widdershin/sonic-pi-cli)
- [lpil/sonic-pi-tool](https://github.com/lpil/sonic-pi-tool)
- [Sonic Pi for standalone installations - use xvfb with no GUI](https://in-thread.sonic-pi.net/t/sonic-pi-for-standalone-installations/225/4)
- [midi notes to real notes](https://www.dummies.com/computers/raspberry-pi/use-note-chord-names-make-music-raspberry-pi/)
- [How Go and Raspberry Pi power piano-playing AI](https://opensource.com/article/17/10/making-raspberry-pi-powered-ai-play-piano)
- [pygame](https://www.pygame.org/wiki/about)

#### sonic pi cli testing

Tested with sonic-pi-cli and xvdb on a headless Pi, with dec 2020 image. It appears that sonic pi itself requires a GUI. First attempt with cli and using October 2020 image with a GUI, but running headless, resulted in highly distorted sound. Other sounds and espeak-ng all worked. Using dec 2020 iamge could not start sonic pi as wanted an x server, and this is the case even when running xvdb.

Going to try other python libs for creating music and sounds.


## Making sounds and music with Pi

### Background

+ https://towardsdatascience.com/mathematics-of-music-in-python-b7d838c84f72
+ https://wiki.python.org/moin/PythonInMusic
+ https://pytorch.org/tutorials/beginner/audio_preprocessing_tutorial.html



