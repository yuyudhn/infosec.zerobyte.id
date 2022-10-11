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

# DOM: Source and Sink

```
Note:
Source dan Sink adalah kunci yang perlu diperhatikan untuk melakukan analisa terhadap celah DOM XSS.
```

### Source
Di dalam JavaScript, Source adalah sebuah sintaks yang berguna untuk mengambil sebuah Value (isi), Value tersebut dapat diubah-ubah oleh User itu sendiri (bersifat Dynamic), bisa dibilang hal tersebut mirip-mirip seperti Parameter Input yang disimpan kedalam Variable.

Source:
- location.search
- location.hash
- location.href
- document.referrer

### Sink
Sink adalah sebuah Function yang biasanya digunakan untuk memproses _Value_ dari `Source`.

Sink:
- document.write()
- document.writeln()
- element.innerHTML
- element.outerHTML

Sebenarnya masih banyak lagi list `Source` dan `Sink` yang perlu diperhatikan, kalian dapat **Googling** untuk menemukan list yang lebih lengkap.

# Example: Sink to Source Analysis

Ini adalah salah satu contoh untuk melakukan analisa terhadap DOM Manipulation.

![source-to-sink-dom-xss](https://infosec.zerobyte.id/images/source-to-sink-dom-xss.png)

Pada _JavaScript_ di atas terdapat sebuah **Sink** yaitu `document.write()` yang memanggil _Variable_ dari _productid_. _Variable productid_ sendiri menggunakan salah satu sintaks **Source** yaitu `location.href` yang di-parsing menggunakan _Function_ `searchParams` untuk mengambil URL Parameter `get('pid')`.

| to "Sink"        | from "Source" |
| ---------------- |:-------------:|
| document.write() | location.href |


### Exploitation
```
https://[redacted].com/[redacted]/cart?pid="%20onerror="alert(1)"%20x="
```
![DOM Triggered](https://infosec.zerobyte.id/images/dom-xss-location-href.png)

# EOF
Untuk menemukan kerentanan DOM XSS sendiri sebenarnya kalian hanya perlu memperhatikan `Source` dan `Sink` yang terdapat pada JavaScript di Website yang sedang kalian lakukan Assessment.

Terima kasih.
