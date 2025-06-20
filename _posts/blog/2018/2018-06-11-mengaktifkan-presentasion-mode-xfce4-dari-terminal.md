---
layout: 'post'
title: 'Mengaktifkan Presentation Mode XFCE4 dari Terminal'
date: '2018-06-11 17:58'
permalink: '/blog/:title'
author: 'BanditHijo'
license: true
comments: true
toc: true
category: 'blog'
tags: ['XFCE']
pin:
hot:
contributors: []
description: "Presentation Mode adalah mode dimana laptop kita tidak akan jatuh pada kondisi blank screen, standby, atau sleep saat sistem dalam keadaan idle atau tidak digunakan. Contoh penerapannya seperti saat presentasi atau saat menonton film dengan durasi yang panjang."
---

![Banner](https://s20.postimg.cc/5q63tl41p/banner_post_16.png)


# Pendahuluan

**Presentation Mode** pada **xfce4-power-manager**. Apa sih itu?

Jadi, yang dimaksud dengan *presentation mode* adalah mode dimana laptop kita tidak akan jatuh pada kondisi *blank screen*, *standby*, atau *sleep* saat sistem dalam keadaan *idle* atau tidak digunakan. Contoh penerapannya seperti saat presentasi atau saat menonton film dengan durasi yang panjang.

Kenapa harus mengaktifkan **Presentation Mode** pada aplikasi **xfce4-power-manager** melalui Terminal? Bukankah sudah terdapat GUI?

![Gambar 1](https://s20.postimg.cc/awi675yml/gambar_01.png)

Gambar 1. xfce4-power-manager graphical user interface

Sederhana saja jawabannya. Kita pasti selalu mencari cara untuk melakukan hal-hal yang rumit agar menjadi lebih praktis. Yaa, praktis.


# Solusi

Coba jalankan perintah `xfconf-query` di Terminal.

```
$ xfconf-query
```

```
Channels:
  xfce4-session
  ristretto
  pointers
  keyboards
  xfwm4
  displays
  xfce4-mime-settings
  xfce4-settings-editor
  xfce4-power-manager 👈️
  xsettings
  xfce4-settings-manager
  xfce4-desktop
  parole
  xfdashboard
  xfce4-appfinder
  xfce4-keyboard-shortcuts
  xfce4-notifyd
  thunar
  xfce4-mixer
  xfce4-volumed-pulse
  xfce4-panel
  xfcethemer
```

Kalo saya tidak salah menebak, `xfconf` ini diambil dari kata *xf configuration*. Dan kita dapat melihat *channels* yang terdiri dari aplikasi-aplikasi bawaan XFCE yang dapat kita konfigurasi menggunakan *xfconf*. Dalam hal ini, kita akan menggunakan *channel* `xfce4-power-manager`.

Lalu, coba jalankan `xfconf-query` dengan menambah *option* `-h`.

```
$ xfconf-query -h
```

```
Usage:
  xfconf-query [OPTION…] - Xfconf commandline utility

Help Options:
  -h, --help            Show help options

Application Options:
  -V, --version         Version information
  -c, --channel         The channel to query/modify
  -p, --property        The property to query/modify
  -s, --set             The new value to set for the property
  -l, --list            List properties (or channels if -c is not specified)
  -v, --verbose         Verbose output
  -n, --create          Create a new property if it does not already exist
  -t, --type            Specify the property value type
  -r, --reset           Reset property
  -R, --recursive       Recursive (use with -r)
  -a, --force-array     Force array even if only one element
  -T, --toggle          Invert an existing boolean property
  -m, --monitor         Monitor a channel for property changes
```

*Output* di atas menampilkan *options* yang dapat kita gunakan untuk mengkonfigurasi *channel*.

Sekarang, coba lihat file `.xml` yang digunakan untuk mengkonfigurasi *xfconf* pada *channel* `xfce4-power-manager`.

```
$ vim ~/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-power-manager.xml
```

```xml
!filename: $HOME/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-power-manager.xml
<?xml version="1.0" encoding="UTF-8"?>

<channel name="xfce4-power-manager" version="1.0">
    <property name="xfce4-power-manager" type="empty">
    <property name="brightness-switch-restore-on-exit" type="int" value="0"/>
    <property name="brightness-switch" type="int" value="0"/>
    <property name="show-tray-icon" type="bool" value="false"/>
    <property name="general-notification" type="bool" value="true"/>
    <property name="critical-power-action" type="uint" value="1"/>
    <property name="inactivity-on-battery" type="uint" value="15"/>
    <property name="inactivity-sleep-mode-on-battery" type="uint" value="1"/>
    <property name="lid-action-on-battery" type="uint" value="1"/>
    <property name="logind-handle-lid-switch" type="bool" value="true"/>
    <property name="lid-action-on-ac" type="uint" value="0"/>
    <property name="presentation-mode" type="bool" value="false"/> 👈️
    <property name="dpms-on-ac-sleep" type="uint" value="31"/>
    <property name="blank-on-ac" type="int" value="30"/>
    <property name="dpms-on-ac-off" type="uint" value="32"/>
    <property name="lock-screen-suspend-hibernate" type="bool" value="true"/>
    <property name="power-button-action" type="uint" value="3"/>
    <property name="handle-brightness-keys" type="bool" value="true"/>
    <property name="brightness-on-battery" type="uint" value="9"/>
    <property name="blank-on-battery" type="int" value="10"/>
    <property name="dpms-on-battery-sleep" type="uint" value="11"/>
    <property name="dpms-on-battery-off" type="uint" value="12"/>
    <property name="show-panel-label" type="int" value="1"/>
  </property>
</channel>
```

Kita dapat lihat di atas, terdapat *property name* **presentation-mode** pada bari ke 15 dengan *type boolean* bernilai *false*. Artinya, saat ini *presentation-mode* dalam keadaan tidak aktif.

Nah, kita dapat merubah nilai yang ada di dalam isi file `.xml` ini dengan menggunakan `xfconf-query`.

Buka Terminal dan jalankan perintah di bawah, untuk membuat *presentation-mode* bernilai *true*.

```
$ xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -s true
```

Untuk merubah kembali menjadi bernilai *false* tinggal ganti nilai `-s true` menjadi `-s false`.

```
$ xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -s false
```

Kalo kita tela'ah perintah di atas, kita menggunakan beberapa *options* diantaranya:

1. `-c` untuk *channel*
1. `-p` untuk *property*, dan
1. `-s` untuk *value*

Selanjutnya kita akan membuat perintah yang panjang di atas menjadi lebih praktis. Karena masih belum praktis kalo kita menuliskan perintah sepanjang di atas.

Cara praktisnya dengan menggunakan **alias**.

Buka file `~/.bashrc` atau `~/.zshrc`. Tergantung tipe **shell** apa yang kalian gunakan. `$ which $SHELL`

Karena saya menggunakan **zsh shell**, maka saya akan menambahkan alias pada perintah di atas pada file `.zshrc` yang ada di direktori `$HOME`.

```
$ vim ~/.zshrc
```

```bash
!filename: $HOME/.zshrc
# Presentation Mode XFCE4-POWER-MANAGER
alias presentationmode-on="xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -s true"
alias presentationmode-off="xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -s false"
```

Dapat dilihat pada baris *alias* di atas, saya menggunakan nama `presentationmode-on` untuk mengaktifkan dan `presentationmode-off` untuk menonaktifkan *presentation-mode*.

Untuk mencobanya, kalian bisa membuka Terminal baru, atau merefresh Terminal yang saat ini sedang terbuka. `$ exec $SHELL`

![Gambar 2](https://s20.postimg.cc/wze9g4mr1/gambar_02.gif)

Kalian juga dapat menambahkan *output* berupa `echo` agar menampilkan tulisan seperti yang saya lakukan pada contoh di atas.

```bash
!filename: $HOME/.zshrc
# Presentation Mode XFCE4-POWER-MANAGER
alias presentationmode-on="xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -s true; echo 'Presentation Mode ON'"
alias presentationmode-off="xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -s false; echo 'Presentation Mode OFF'"
```

Bisa juga ditambahkan notification.

```bash
!filename: $HOME/.zshrc
# Presentation Mode XFCE4-POWER-MANAGER
alias presentationmode-on="xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -s true; echo 'Presentation Mode ON'; notify-send '[ON] Presentation Mode'"
alias presentationmode-off="xfconf-query -c xfce4-power-manager -p /xfce4-power-manager/presentation-mode -s false; echo 'Presentation Mode OFF'; notify-send '[OFF] Presentation Mode'"
```

Saya rasa cukup seperti ini saja.


# Referensi

1. [askubuntu.com/questions/407287/change-xfce4-power-manager-option-from-terminal/407298#407298](https://askubuntu.com/questions/407287/change-xfce4-power-manager-option-from-terminal/407298#407298)
<br>Diakses tanggal: 18/06/11
