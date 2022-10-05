---
layout: post
title: Hunting Secret Key in JavaScript File
categories: [Web Security, Bug bounty write-up]
author: Rafshanzani Suhada
---

Banyak dari kalian yang mungkin belum tahu mengenai hal ini dan artikel ini akan menjelaskan cara untuk mendapatkan _Secret Key_ di _File .js_.

Berikut Tool yang akan digunakan:

- https://github.com/lc/gau
- https://github.com/projectdiscovery/nuclei

## Hunting JavaScript File
Dengan bantuan _Tool_ **gau** (getallurls) kita dapat mengekstraksi URL melalui AlienVault's Open Threat Exchange, the Wayback Machine, dan Common Crawl kemudian melakukan _Filter_ endpoint untuk JS file saja.

```
echo example.com | gau --threads 10 | cut -d"?" -f1 | grep -E "\.js(?:onp?)?$" | tee jslinks.txt
```

Setelah URL _File .js_ terkumpul, selanjutnya mencari _Secret Key_ pada _File JavaScript (.js)_ kita menggunakan _Tool_ nuclei dengan template yang sudah disediakan.

```
nuclei -t exposures/tokens/ -l jslinks.txt
```

![Nuclei Scan](https://infosec.zerobyte.id/images/nuclei-token-exposure-scan.png)

Sebagai contoh di sini saya menemukan Secret Key yang dapat mengakses salah satu Provider Mailer.
![Sendgrid Secret Key](https://infosec.zerobyte.id/images/sendgrid-secret-key-leaked.png)
