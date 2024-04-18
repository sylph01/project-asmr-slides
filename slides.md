---
title: Adding Security to Microcontroller Ruby
paginate: true
marp: true
theme: argent
---

<!-- _class: titlepage -->

# **A**dding **S**ecurity to **M**icrocontroller **R**uby
## Ryo Kajiwara/梶原 龍 (sylph01)
### 2024/5/16 @ RubyKaigi 2024

---

# Slides are available at:

## TBD (also change the image)

![bg right](images/frame.png)

---

<!-- _class: titlepage -->

# Hi!

---

<!--
  _class: titlepage_white
-->

# I do stuff

- Play rhythm games (especially DanceDanceRevolution)
  - btw, I'm DJing rhythm game songs at RubyMusicMixin this year so come see me there too!
- Play the bassoon/contrabassoon
- Ride a lot of trains (Rails!) (travelled on 99% of JR)
- Build keyboards

if anything catches your interest let's talk!

![bg brightness:0.5](images/Screenshot_20231204_183910.png)

![bg brightness:0.5](images/cfg_long.png)

![bg brightness:0.5](images/PXL_20210729_041115019.jpg)

![bg brightness:0.5](images/keebs.png)

---

<!-- _class: titlepage -->

# And I do stuff

that is more relevant to this talk:

- Freelance web developer focused on Digital Identity and Security
- Worked/ing on writing/editing and implementing standards
  - HTTPS in Local Network CG / Web of Things WG @ W3C, OAuth / Messaging Layer Security WG @ IETF
- Worked as an Officer of Internet Society Japan Chapter (2020-23)

---

<!-- _class: titlepage -->

# I'm from the **Forgotten Realm** of Japan called **Shikoku**

![bg right:40% contain](images/DcwvXpdX4AA9x1Q.jpeg)

<!-- _footer: image https://twitter.com/Mitchan_599/status/994221711942971392 -->

----

----

# Previously in "Adventures in the Dungeons of OpenSSL" ...

(talk at RubyConf Taiwan 2023)

- Implemented HPKE (RFC 9180)
  - Using OpenSSL gem (GH: sylph01/hpke-rb)
  - Also by extending OpenSSL gem

----

# Why not implement cryptography into **microcontrollers**?
# Maybe we can get **TLS** too?

----

# Raspberry Pi Pico W

- Board with RP2040 microcontroller
  - Dual-core ARM Cortex-M0+ @ 133MHz
- Wireless LAN (802.11n) with CYW43439

![bg right:35%](images/pi_pico_w.jpg)

----

# Target environment

- (all of the following is done by @hasumikin -san)
- PicoRuby + R2P2
  - https://github.com/picoruby/picoruby : Ruby implementation
  - https://github.com/picoruby/R2P2 : Shell system for Pi Pico (W)
- Well-known usage: "PRK Firmware: Keyboard is Essentially Ruby" (RubyKaigi 2021 Takeout) https://www.youtube.com/watch?v=5unMW_BAd4A

----

# MicroPython has this already

```python
import network
from time import sleep

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('ssid', 'password')

while wlan.isconnected() == False:
  print('Connecting...')
  sleep(1)
print(wlan.ifconfig())
```

<!-- _footer: code from https://projects.raspberrypi.org/en/projects/get-started-pico-w/2 -->

----

# Project
# **A**dding
# **S**ecurity to
# **M**icrocontroller
# **R**uby

<!--
  _class: stacking-headers
-->

----

# Progress

----

# Caution: Here be dragons

- Cryptographic API is prone to misuse
- Embedded bugs are hard
- Most implementation here are experimental
  - Gems that entered PicoRuby/R2P2 are pretty much "prod-ready"
  - Everything else (esp. stuff that touches networking hardware) should be considered experimental

<!--
  I'm treating Pico W as "a normal computer" whenever I can. This is not trivial at all. This is made trivial thanks to R2P2 (shell system).
-->

----

----

<!-- _class: titlepage -->

# Part 1:
# Cryptography in PicoRuby

----

(overview)

- we have Mbed TLS instead of OpenSSL
- PicoRuby had an implementation of CMAC using Mbed TLS
- actual impl details
  - Generating mruby/c objects compared with CRuby objects
    - Reference counters...
  - SHA-256, AES-CBC/GCM
    - Does not perform operations all at once. We are in a restricted environment
- Did I say "don't use fixed nonces"
  - RNG? it's not a given, we need to implement that
    - RNG using Ring Oscillator, von Neumann whitening

----

----

<!-- _class: titlepage -->

# Part 2:

# Networking

----

(overview)

- General overview of what was missing
- What do we mean by Networking here
  - WiFi, TCP/IP
- Porting of MicroPython's Networking
  - details
- Debugging Networking
  - you have kali in your house right

----

----

<!-- _class: titlepage -->

# Part 3:

# Networking in the Upper Layer

----

(overview)

- Now that we have sockets
- Basic HTTP is easy
- DNS?
- TLS?
- Tying it all together: HTTPS

----

----

<!-- _class: titlepage -->

# Conclusion

----

# Conclusion

# We **can haz** TLS in PicoRuby!

----

# But do we **really** need TLS?

- Symmetric crypto is okay, but asymmetric crypto is **very slow**
- Performance numbers from similar environments:
  - [STM32L562E Cortex-M33 at 110 MHz, wolfSSL](https://www.wolfssl.com/docs/stm32/)
    - RSA 2048 Signing: 9.208 ops/sec
    - RSA 2048 Verfiication: **0.155 ops/sec**
    - ECDHE 256 Key agreement: **0.661 ops/sec**
- Depending on your security needs, it would be enough to **just use symmetric crypto** between the gateway and use TLS from there

<!--
  Note: In this case you have to provision your devices with a fixed symmetric key. Physical compromise of the device is possible. But do you even care about that in most cases?
-->

----

# Possible Future Work in IoT area

- (yes, finish everything first, but...)
- CBOR/COSE/CoAP support
  - Think of it as "JSON in binary"
- There are working groups in the IETF geared towards "constrained environments"
  - Authentication and Authorization for Constrained Environments
  - Lightweight Authenticated Key Exchange
  - Software Updates for Internet of Things

----

# Shoutouts

(@ mentions are in GitHub ID)

- Major shoutouts to @hasumikin for the extensive work in PicoRuby and helping me develop stuff on it
- Past RubyKaigi speakers, esp. @unasuke and @shioimm

And to the organizers of RubyKaigi 2024!

---

# More Shoutouts

Sponsors of RubyKaigi, esp:

- codeTakt Inc.
  - I am currently developing an ID platform for public schools with them
- Nexway Co., Ltd.
  - I have played multiple times in TIS INTEC Group's orchestra


![bg right:25%](images/sponsors.png)

----

# Questions? / Comments?

## Twitter: @s01 or Fediverse: @s01@ruby.social
