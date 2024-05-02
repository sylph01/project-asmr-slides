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

## https://speakerdeck.com/sylph01/adding-security-to-microcontroller-ruby

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
  - **264kB SRAM**, 2MB Flash
- Wireless LAN (802.11n) with CYW43439
- 1353 yen (as of 4/18/2024)
  - 技適 certified!

![bg right:35%](images/pi_pico_w.jpg)

<!--
  https://www.switch-science.com/products/8171

  it's 技適 certified so we can legally use wireless functionality in Japan!

  btw the size of SRAM becomes important later
-->

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

<!--
  I came up with this name while writing the CfP
-->

----

# More like
# Adding
# **SSL/TLS** to
# Microcontroller
# Ruby

<!--
  _class: stacking-headers
-->

----

# or
# Adding
# **Networking** to
# Microcontroller
# Ruby

<!--
  _class: stacking-headers
-->

<!--
  but meme value, right?
-->

----

# [Demonstration](https://youtu.be/oYiBG7P2yyE)

If you want to see it in person, come find me any time during the Kaigi!

----

# Caution: Here be dragons

- Cryptographic API is prone to misuse
- Most implementation here are experimental
  - Gems that entered PicoRuby/R2P2 are pretty much "prod-ready"
  - Everything else (esp. stuff that touches networking hardware) should be considered experimental

<!--
  here's the obligatory warning
-->

----

# Caution: Embedded bugs are hard

I'm treating Pico W as "a normal computer" whenever I can.

**This is not trivial at all.** There could be many random stuff I am missing.

This is possible thanks to:

- Pico SDK (including lwIP, Mbed TLS)
- R2P2 (shell system)

----

----

<!-- _class: titlepage -->

# Part 1:
# Cryptography in PicoRuby

----

# Quick Recap: SHA256 in Ruby

```ruby
require 'openssl'

digest = OpenSSL::Digest.new('sha256')
digest.update('The magic words are ')
digest.update('squeamish ossifrage.')

OpenSSL::Digest::SHA256.hexdigest(
  'The magic words are squeamish ossifrage.'
)
```

----

# Quick Recap: AES in Ruby (encryption)

```ruby
require 'openssl'

cipher = OpenSSL::Cipher::AES128.new('CBC')
cipher.encrypt
key = cipher.random_key
iv  = cipher.random_iv

enc =  cipher.update('The magic words are ')
enc += cipher.update('squeamish ossifrage.')
enc += cipher.final
```

----

# Quick Recap: AES in Ruby (decryption)

```ruby
# cont'd
decipher = OpenSSL::Cipher::AES128.new('CBC')
decipher.decrypt
decipher.key = key
decipher.iv  = iv

plain =  decipher.update(enc)
plain += decipher.final
```

----

# 


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
- Debugging Networking
  - Use RasPi 5 as a WiFi router
  - Wireshark
- misc
  - mruby/c's String's actual C representation is guaranteed to be null-terminated https://github.com/mrubyc/mrubyc/blob/master/src/c_string.c#L71-L103

----

- Connect to WiFi AP with CYW43
- Introduction to lwIP
- DNS
  - actually this was easy
- TCP Client
  - Application Layered TCP
- If you have TCP Client then Basic HTTP is easy

----

- TLS
  - with lwIP and ALTCP TLS is pretty trivial
  - Implementation quirks
    - you cannot call `malloc()` and `free()`
      - They are libc functions. If you do it hangs up
      - `mrbc_raw_alloc`, also lwIP has its own memory management
    - OOM
      - to enable TLS I had to reduce R2P2's heap memory significantly

----

----

<!-- _class: titlepage -->

# Part 3:

# Fleshing Out Networking

----

(overview)

- Look & Feel of the API
- Blocking vs Non-Blocking
- Porting of MicroPython's Networking
  - details
  - Sockets?

----

----

<!-- _class: titlepage -->

# Conclusion

----

<!-- _class: titlepage_white -->

# We **can haz** TLS in PicoRuby!

![bg brightness:0.5](images/IMG_0468.JPG)

----

# Stuff delivered

- In PicoRuby/R2P2 now!
  - Base16, Base64
  - SHA256
  - AES Encryption (CBC, GCM)
  - Random Number Generator

----

# Stuff delivered (cont)

- Experimental
  - CYW43
    - Connect to WiFi
  - Net
    - DNS
    - TCPClient
    - HTTP(S)Client
      - GET

----

# But do we **really** need TLS?

- Symmetric crypto is okay, but asymmetric crypto is **very slow**
- Performance numbers from similar environments:
  - [STM32L562E Cortex-M33 at 110 MHz, wolfSSL](https://www.wolfssl.com/docs/stm32/)
    - RSA 2048 Signing: 9.208 ops/sec
    - RSA 2048 Verfiication: **0.155 ops/sec**
    - ECDHE 256 Key agreement: **0.661 ops/sec**

----

# But do we **really** need TLS?

Also, without a trust store, we will get **encryption** through TLS, but we will not get the **authentication** part of TLS.

For these reasons, depending on your security needs, it would be enough to **just use symmetric crypto** between the gateway and use TLS from there.

<!--
  In this case, the gateway has a trust store (trusted certificate list) and thus will be capable of authentication.

  Note: In this case you have to provision your devices with a fixed symmetric key. Physical compromise of the device is possible. But do you even care about that in most cases?
-->

----

## You might want to use a **Raspberry Pi Zero 2 W** if you want a "more traditional" computer experience

Runs a Linux, can SSH into it, can run a GUI, has enough power to run asymmetric crypto

![bg right:35%](images/PXL_20240424_142957171.jpg)

----

# Why use a Pico?

**It's closer to hardware.** It's easier to embed into other hardware.

![bg right:35%](images/pi_pico_w.jpg)


----

<!--
  _class: titlepage_white
-->

# In an IoT environment,

# these devices **work in concert**

![bg brightness:0.3](images/PXL_20240424_142957171.jpg)

![bg brightness:0.3](images/pi_pico_w.jpg)

----

<!--
  _class: titlepage_white
-->

# With Pico's WiFi connectivity,

# we can say **Actual IoT with Ruby** is closer to a reality!

![bg brightness:0.2](images/PXL_20240424_142957171.jpg)

![bg brightness:0.75](images/pi_pico_w.jpg)

----

# Possible Future Work in IoT area

- CBOR/COSE/CoAP support
  - Think of it as "JSON in binary"
- There are working groups in the IETF geared towards "constrained environments"
  - Authentication and Authorization for Constrained Environments
  - Lightweight Authenticated Key Exchange
  - Software Updates for Internet of Things

----

<!--
  _class: titlepage_white center
-->

# It's a **Blue Ocean**

![bg brightness:0.9](images/DSCF0156.jpeg)

<!--
  The community is waiting for your contribution!
-->

----

# Shoutouts

(@ mentions are in GitHub ID)

- Major shoutouts to @hasumikin for the extensive work in PicoRuby and helping me develop stuff on it
- Past RubyKaigi speakers, esp. @unasuke and @shioimm (Team Protocol Implementers!)

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
