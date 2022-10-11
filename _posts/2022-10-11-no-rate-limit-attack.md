---
layout: post
title: No Rate Limit Attack
categories: [Web Security]
author: Novran Faathir
---

No Rate Limit adalah suatu celah yang dimanfaatkan untuk **melakukan request berulang kali tanpa ada batasan dari aplikasi** atau lebih tepatnya **Attacker dapat mengulangi request tersebut secara berulang tanpa ada batasan**. No Rate Limit sendiri adalah nama Universal-nya Brute Force.

No Rate Limit termasuk dalam "**CWE-770**" yaitu **Allocation of Resources Without Limits or Throttling**, atau kalian dapat lihat pada halaman berikut ini <https://cwe.mitre.org/data/definitions/770.html>.

Tentu saja bagi seorang Bug Hunter ini sering dimanfaatkan untuk:
 - Email Triggering (Spam)
 - SMS Triggering (Spam)
 - Email Enumeration
 - Brute Force Reset Token Code
 - Bypass 2FA - Second Factor Authentication (Brute Force)

# Apakah serangan ini berbahaya?
Berbahaya atau tidak tergantung impact (dampaknya).

### Low - No Rate Limit Attack
Menurut Vulnerability Rating Taxonomy pada Bugcrowd beberapa hal tentang No Rate Limit termasuk dalam P4 (Priority 4) atau LOW.

![no-rate-limit-vrt](https://infosec.zerobyte.id/images/no-rate-limit-vrt.png)

Beberapa diantaranya sudah kita sebutkan di atas (Email Triggering & SMS Triggering).

### Critical - No Rate Limit Attack
Pada 2016 silam, Security Researcher bernama Anand Prakash berhasil menemukan celah No Rate Limit (Brute Force) pada beta.facebook.com. Hal ini menyebabkan ia dapat meretas seluruh akun Facebook milik orang lain.

Ia dapat melakukan Brute Force pada endpoint reset password beta.facebook.com, lebih tepatnya pada parameter "n=" ia dapat melakukan request tanpa ada batasan (menggunakan 6 digit kode: 000000–999999) hingga mendapatkan reset token code yang valid.

```
POST /recover/as/code HTTP/1.1
Host: beta.facebook.com

lsd={TOKEN}&n=[000000–999999]
```

Referensi : <https://medium.com/appsecure/responsible-disclosure-how-i-could-have-hacked-all-facebook-accounts-f47c0252ae4d>

---

# Bagaimana cara menemukan celah No Rate Limit?

Pertama-tama kita cari salah satu endpoint yang sekiranya dapat Trigger Email, contohnya Reset Password.

Kemudian masukan Email (akun yang sudah terdaftar milik kalian), sebelum kalian Submit, kalian perlu mencatat request-nya, kalian dapat menggunakan Burp Suite / Developer Tool (tekan F12 pada Browser).

Setelah tekan "Submit" kalian coba lakukan berulang kali, pada Burp Suute terdapat menu "Intruder", namun disini kami akan menggunakan CURL.

Kembali pada Burp Suite / Developer Tool, pada request yang tercatat kalian copy request tersebut sebagai curl dengan cara klik kanan dan "Copy as CURL".

![no-rate-limit-1](https://infosec.zerobyte.id/images/no-rate-limit-1.png)

Kemudian jalankan pada shell kalian seperti di bawah ini.

![no-rate-limit-2](https://infosec.zerobyte.id/images/no-rate-limit-2.png)

Supaya lebih mudah kalian dapat membuat script simple seperti di bawah ini:

![no-rate-limit-3](https://infosec.zerobyte.id/images/no-rate-limit-2.png)

Lakukan setidaknya 100 kali dan memastikan bahwa tidak ada limitasi dari aplikasi tersebut.

Bilamana **tujuan kalian melakukan Email Triggering** pastikan bahwa Email tersebut membanjiri akun kalian (sesuai dengan request yang kalian kirim).

---

Tentu saja celah ini sangat mudah ditemui pada Platform mana pun, **namun INGAT!** sebelum kalian berburu pastikan hal ini tidak mengganggu jalannya Production (dengan cara melihat **Scope** baik-baik), karena aktivitas ini sangat "harm".

