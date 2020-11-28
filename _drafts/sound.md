# Pi sound

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