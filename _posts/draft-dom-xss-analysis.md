---
layout: post
title: DOM-Based Cross-Site Scripting Analysis
categories: [Web Security]
author: Novran Faathir
---

Artikel ini akan membahas tentang bagaimana cara melakukan Static Analysis untuk menemukan kerentanan DOM XSS.

# Apa itu DOM XSS?
DOM-Based Cross-Site Scripting adalah salah satu celah XSS, celah ini juga bisa dibilang sebagai JavaScript XSS yang dimana Attacker akan mengeksploitasi XSS tersebut di level JavaScript.

| Reflected XSS                                  | DOM XSS                                              |
| ---------------------------------------------- |:----------------------------------------------------:|
| Dapat dilihat menggunakan CTRL+U (view-source) | Tidak dapat dilihat menggunakan CTRL+U (view-source) |
| Mudah ditemukan menggunakan Web Scanner        | Agak sulit untuk ditemukan menggunakan Web Scanner   |

![JS XSS](https://infosec.zerobyte.id/images/javascript-xss.png)

## Source
Di dalam JavaScript, Source berguna untuk mengambil sebuah Value (dinamis) yang dapat diubah oleh User, bisa dibilang hal tersebut mirip-mirip seperti Parameter yang disimpan kedalam Variable.

Contoh beberapa Source:
- location.search
- location.hash

## Sink
Sink adalah sebuah Function yang bisa dibilang kurang aman yang biasanya digunakan untuk memproses isi dari `Source`.

Contoh beberapa Sink:
- document.write()
- document.writeln()
- element.innerHTML
- element.outerHTML
