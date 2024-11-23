---
title: Actual Security in Microcontroller Ruby!?
paginate: true
marp: true
theme: argent
---

<!-- _class: titlepage -->

# "**A**ctual" **S**ecurity in **M**icrocontroller **R**uby!?
## Ryo Kajiwara/梶原 龍 (sylph01)
### 2024/12/x @ KeebWorld Conference 2024

---

# だれ / なに

![](images/Screenshot_20240826_070525.png)

<!--
  やせいのプログラマをしていて、認証認可とか暗号とかができます。
  今年のRubyKaigiではRaspberry Pi Pico Wのネットワーク機能をPicoRubyから使えるようにするという話をしました。今日の話も似たような話です。タイトルも似てるし。
-->

---

![bg fit](images/GLMNNWKbIAAjuxk.jpeg)
![bg fit](images/GLMNNTjaIAA_Sog.jpeg)

<!--
  キーボード的な話をするならば、日常使いのキーボードはKeebioのIrisで、過去に合計7枚組んで4枚が現役、まだ組んでいないPCBが2枚あります。
  この子はポップンミュージックの蒼井硝子（あおい-しょうこ）ちゃんというキャラクターをモチーフにしたキーキャップであるGMK Shokoと、その色に合う軽量リニアスイッチであるYushakobo Fairy Silent Linear Switchを使っています。軽量リニアスイッチ、いいですよね
-->

---

![bg fit](images/PXL_20241123_083846365.jpg)

<!--
  Phyrexia: Keeb Will Be One
-->

---

# RubyKaigi followupでこんなこと言ってましたね？

![](images/adding-security.png)

---

# 今日の話

- RP2350のセキュアブートで遊んでみた
- キーボードへのインパクトは？
- PicoRubyへのインパクトは？

---

# 注意

- いつもの「暗号には気をつけよう」に加えて
- **チップに対して不可逆な操作を行います**
  - One-Time Programmable Memory (OTP)への書き込みは名前の通り不可逆です
  - セキュアブートはこの領域に鍵のフィンガープリントを書き込むことで行います
  - 鍵のバックアップをし損ねると**そのRP2350は文鎮です**

---

# RP2350

Raspberry Pi Pico 2世代の石。

- ARM Cortex-M33
  - Dual-core, 150MHz
  - 520KB on-chip SRAM
    - up from 260KB!
  - **ARM TrustZone搭載**
    - 「$5で買えるTEE」

![bg fit right:35% vertical](images/PXL_20241123_045751940.jpg)
![bg fit right:35%](images/PXL_20241123_084918864.jpg)

<!--
  今手頃に手に入るRP2350っていうとRaspberry Pi Pico 2とSparkFun Pro Micro RP2350。後者はUSB Type-CだしPro Microフォームファクタなのでキーボード組み込みには都合いいけど高い。コンスルーの高さ間違えて数千円捨てるところだった
-->

---

# セキュアブート

- Boot Signing
  - 秘密鍵を使って署名したファームウェアを書き込み、OTPに書き込んだ公開鍵を使って署名を検証
  - 通常セキュアブートといえばこっち
  - RP2350においては**RISC-Vコアを完全に無効化する**
- Encrypted Boot
  - 暗号化したバイナリを書き込み、OTPに書き込んだ鍵を使って復号、プログラムを実行する

---

# 実際にやってみる

---

---

# キーボードにとって何が嬉しいの？

ファームウェアの悪意のある書き換えから身を守ることができる、ということは:

- 不在の間にキーマップを勝手に書き換えられることから身を守ることができる
- ネットワークにつながる板だった場合に**キーストロークの悪意のあるモニタリングと送信から身を守ることができる**
  - キーボードがネットワークにつながる？→PicoRubyはWiFi対応したのでできてしまうんですね…

---

# ぶっちゃけプログラマブルな自作キーボードって高いコンプライアンス要件にとっては悪夢では

外部から制限付きとはいえ計算機を持ち込んでいるに等しい。WiFiに繋がるキーボードなんてもってのほか。

ファームウェアが信用できない場合は信用できない計算機を持ち込んでいることになる。**動かしているファームウェアに確証が持てる**ことはその点で嬉しい（本当に？）。

---

# OTPの存在がかなり嬉しい

- デバイスに恒久的な初期設定値を焼くことができる
  - 身近なところだとデバイスの識別子とか
- セキュアモードからしか読めないOTP領域を設定できるので
  - 鍵を安全に保管することができる

---

# PicoRubyで使える？

- Boot Signing/Encrypted Boot自体はビルド手順にちょっと手を加えることでできそう
- RP2350固有の機能をPicoRubyから使うにはPicoRuby側に手を入れる必要がある
  - OTPの値を読む
  - RP2350には乱数生成器(TRNG)がついているのでそれもほしい
  - SHA256のアクセラレータも使える？
