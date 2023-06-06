---
layout: post
title: Bypass Root Detection Aplikasi Android Menggunakan Frida
categories: [Pentesting, Mobile Application Pentesting]
author: yuyudhn
---

## Bypass Root Detection

Dalam dunia pengembangan aplikasi mobile, keamanan merupakan hal yang sangat penting. Salah satu aspek keamanan yang sering diterapkan adalah deteksi root pada perangkat Android. Root detection digunakan untuk mengidentifikasi apakah perangkat yang menjalankan aplikasi tersebut telah di-root atau tidak. 

Namun, bagi seorang peneliti keamanan atau pengembang aplikasi yang ingin melakukan pengujian lebih lanjut, root detection dapat menjadi hambatan. Ada beberapa modul untuk melakukan bypass root detection pada aplikasi Android, seperti Magisk, Shamiko, Xposed, dll. 

Tapi bagaimana jika didalam aplikasi Android yang kita pakai sudah terdapat function untuk memeriksa dan mengenali modul diatas? Di sinilah peran Frida sebagai alat dynamic instrumentation sangat penting. Dalam artikel ini, kami akan menjelaskan bagaimana menggunakan Frida untuk melakukan bypass root detection pada perangkat Android, membuka peluang untuk menguji aplikasi dengan lebih bebas dan mendalam.

## Bypass Root Detection with Frida

Disini saya tidak akan menjelaskan bagaimana cara menginstall ataupun mengkonfigurasi frida maupun frida-server, kalian bisa googling sendiri untuk cara instalasinya. Namun lebih ke bagaimana kita melakukan inspeksi function Root Detection yang ada pada aplikasi, lalu melakukan hooking menggunakan frida untuk membypass function tersebut.

Yang dibutuhkan:
- Jadx-GUI
- Frida
- Android Emulator

Bagi pengguna Kali Linux, `jadx` dapat diinstall menggunakan perintah
```bash
sudo apt update && sudo apt install jadx -y
```

Untuk pemanasan, kita akan coba untuk melakukan bypass root detection pada aplikasi [AndroGoat](https://github.com/satishpatnayak/MyTest/blob/master/AndroGoat.apk).

Sebelum memulai tutorial ini, buat satu file .js dengan nama terserah (sebagai contoh disini saya buat dengan nama inori.js), lalu isikan didalamnya:
```javascript
Java.perform(function () {
   // Bypass Root Detection
});
```
Pertama, buka file .apk menggunakan Jadx-GUI, lalu cari function yang digunakan untuk menjalankan root detection. Berikut beberapa keyword yang mungkin bisa dipakai: "/bin", "which su", "rooted".

![Search root detection](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjpLqONSFdhX-VFA3Gko4GYvdCqfkaOKcPL7UC2BfH8MCntKDgB0M0qySNa7Z-TBr_kL7k3DzW779EVGuokBZ6ukpIUsmkJVTjOG-GgPeVgICVMx39aKKecy2Hmpf3QCy95roAgFLLOTBeNOnc5iYpVjIgI6n2rlJaGTMSQ1nMZu-69cmds0qQnzSrlWA/s746)

Disini terlihat ada dua fungsi yang digunakan untuk mendeteksi root pada device, yakni fungsi `isRooted` dan fungsi `isRooted1`.

![Root Detection](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhNRBd2Ga0AMIaxZJxNkTNrCXq74t9eR7Bv9bk7c2bMxuwDQHYIknVhgSrT1cSmRBVpeas2ICfi91TKkBPgiQ_aTY13awzJx--jA1rqrE8XgxcRZE2O252d3dp8I1om54RjWy-ZZKdbUkw8YRBrz021jkAkjKHRh-P4Ywvrchx_S2aqUUv7vNUiHTpsbw/s833)

Selanjutnya, klik kanan pada nama fungsi `isRooted` lalu klik **Copy as frida snippet**. 

![Copy to Frida](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEivj-0etv6I7j_tL4KasoZfDiCWdEaLDLG6PCh1auYSFB4irhdUcXsvrdDb4tjO5hHjoiOxa85b4RtxLQAkDPnihZroBcmU4SiDGiA1H9lxvZN2i7UeNcxlSBb_TWSTYsfiiGEyeJSpVdBUcvbjI7e6-leHjHCEhyAtfl3V1QGa8-ijulDU9pwI6s1pJg/s777)

Paste kedalam file .js yang sudah kita siapkan sebelumnya. Ubah nilai return menjadi false. Sekarang kode yang kita miliki menjadi seperti berikut:
```javascript
Java.perform(function () {
   // Bypass Root Detection
   let RootDetectionActivity = Java.use("owasp.sat.agoat.RootDetectionActivity");
    RootDetectionActivity["isRooted"].implementation = function () {
        console.log(`RootDetectionActivity.isRooted is called`);
        let result = this["isRooted"]();
        console.log(`RootDetectionActivity.isRooted result=${result}`);
        return false;
    };
});
```

Ulangi langkah sebelumnya ke function `isRooted1`. Namun karena `RootDetectionActivity` sudah dideklarasikan sebelumnya, hapus untuk yang kedua. Sehingga final script untuk bypass root detection kita menjadi:

```javascript
Java.perform(function () {
   // Bypass Root Detection
    let RootDetectionActivity = Java.use("owasp.sat.agoat.RootDetectionActivity");
    RootDetectionActivity["isRooted"].implementation = function () {
        console.log(`RootDetectionActivity.isRooted is called`);
        let result = this["isRooted"]();
        console.log(`RootDetectionActivity.isRooted result=${result}`);
        return false;
    };
    RootDetectionActivity["isRooted1"].implementation = function () {
        console.log(`RootDetectionActivity.isRooted1 is called`);
        let result = this["isRooted1"]();
        console.log(`RootDetectionActivity.isRooted1 result=${result}`);
        return false;
    };    
});

```

Selanjutnya kita bisa spawn aplikasi AndroGoat menggunakan perintah:

```bash
frida -l inori.js -U -f owasp.sat.agoat
```

"owasp.sat.agoat" adalah nama proses yang bisa dicek menggunakan perintah `frida-ps -Uai`.

Ketika kita cek di AndroGoat, maka notifikasi yang muncul ketika kita cek adalah "Device is not rooted" yang menunjukkan bypass root detection kita berhasil.

![Root Bypassed](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhrhhsN2MBWMFTfzxuIoINe0teyTyv4I-ozDL5-cLvsE_PAz_R3Mx0qwMQ31WUIYortoiARqDjIWpRrQ_4s83sNKzSRDJFYfJwvtGmJAlnSvDC86fDHao9CJEX7L784ekNDypzU7RTDm48lpgp5rDGm-2NwQW-0_UITPzHaDcP6Fk_NxwTRsYKK5WdqCg/s839)

Jadi teori sederhananya dalam melakukan bypass root detection adalah:

Inspeksi file .apk menggunakan Jadx > Cari fungsi root detection > Copy ke frida snippet > ubah nilai return menjadi false.

### Bypass iRoot Plugin

Oke, sekarang ke real case dimana aktifitas bypass root detection ini harus dilakukan ketika saya melakukan pentesting mobile apps salah satu client. Namun karena alasan privasi, saya tidak akan me-reveal nama aplikasi maupun tampilan aplikasinya.

Root detection pada aplikasi ini tidak bisa dibypass menggunakan script publik dari Frida CodeShare. Berikut hasil yang didapat ketika mencoba melakukan bypass dengan script [antiroot](https://codeshare.frida.re/@dzonerzy/fridantiroot/) dari Frida CodeShare.

![Apps Killed](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhBLpnRH1XVJNZzb5UQHnQI_Kj4R2XscaMf51qZKet5c0Ds3JUe994uWlEAh05vpnS-jSa6sIGdtw3yiQfZDFCs8j4r3uKwaiW3IVD_JAsTlPrMEeyw6VXG5CCdDwYGQuUKTkTNiKPCiLXG7Fz4vKTK1E4Q3jUDRaWuXb1xrWW797zEssh0rR0fgaSkZw/s850)

Intinya, ketika aplikasi di spawn menggunakan Frida, root masih terdeteksi dan aplikasi langsung force closed.

Selanjutnya kita cari function Root Detection menggunakan Jadx seperti yang sudah kita bahas sebelumnya.

![search /bin](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi9QL_4sr-EqtZCa0IHl4Ewtaf7hpBAQIg-t7tRlMTA0WD5x7dnQEU9OJM4mK5Ew1l9g0i7SZ0rFbPT5XCUZMHzovVsN8dSc0pnGk1HDZFG3YrIPZIvc6p6yn2bnAmxWD_LL_wGDBjugxOHfpCTA7NPltvfAMJrJwFP24dQGFfQUtclWq5M7XUsJ2VbVQ/s1114)

![Root Check](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgbBYiUbftkpqYrkt-SfX_bmYfvodqiqPxehO-U1bSTxx8OY4JceA1a5ebAbsfOv4UaAADr1KDxd5HYMZnaD8JhyPdYso5QiJUumAbg_6JI-5nAvLStEcW3H1-BW9zNHHjKA7o4Mvu74gsbSKCLbKwYRhWYLERCH24RvIFVfBjTMG-qVHi_jSw1ZAXhFw/s923)

Disini ketika tau bahwa aplikasi menggunakan plugin iRoot, saya mencoba public script dari Frida CodeShare namun ternyyata masih gagal.

![iRoot](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiJQdJPVWswQ1UizvE3mQtv9QzNDg1LY3wn6T3ngqSgPzzjPFmo3JVszMtrPmxo_ad4Ip-vpaJ7AsvNC0YrmG4cHuYNnNJD9HJ77APrL8jIzgJVqbXIkArvOKe3VW2DbL1paXuSb-SKuy7eNYwr0-UCvSBrmn9vwxrABVbmHvAkP4A-AZuN5WrZonq-bw/s931)

- https://codeshare.frida.re/@JJK96/iroot-root-detection-bypass/

Singkat cerita, ketemulah function untuk mendeteksi root tersebut.

![isRooted](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh4d_0fIOiovcONE7fvOV4bAoXxOrjV5doTbrld4burmhpX4s0LnjXrZpOIU9iTpFzvSFCGJ2F8tXMX5pcQUcluVmTGFJ53OWj5ZUFfynMblrxgFW5T5NrAK9UwvoiY3BjwLp2YytJ-eLp9DU0qJ2P1g3DXdOx3c_GbTbhK5N6eiUCAOxGg1Y-8C_jGRg/s991)

Selanjutnya kita cari lagi dimana class u1.b dipanggil.

![Search u1.b](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgD4dWfU5wCurg7XMFXf3-IXbkFELa9UmIZ4mUaH6UKs5KZynF29IteNhur8Vdh3K3OyJhJBfVUwcdBc6_v3mVgtMPP60lavH8R1lbvR5hQqB3Z3gX7IXzijxyu1_xLbBAzvdiaX3J5vhS2aqcBCHV90mjxm3aklpVtqMYou_kRTKXi8txmA7HcBQx-Sg/s872)

Setelah ketemu, kita langsung copy ke frida snippet.

![Copy to Frida](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgs6dkWOg_xJPClrBcZnN0iAAZQSljTDZdedgQJ_DGxdiVzT9hOl3GJpVBONR74rWPspai0ufNO-ZMI7o5DwFOgswWy0p1UtSbjTWsqvmGSL3m8npjm6dv3a240eOh2-QSdlVhmLCeGrlWh-EDaKhA9Zfdi8X6C8HUIDJ1fQ7UZMuoYY0d2s-u5ODEZ0g/s893)

Final script:
```javascript
Java.perform(function () {
   // iRoot Bypass
    let IRoot = Java.use("de.cyberkatze.iroot.IRoot");
      IRoot["execute"].implementation = function (str, jSONArray, callbackContext) {
      console.log(`IRoot.execute is called: str=${str}, jSONArray=${jSONArray}, callbackContext=${callbackContext}`);
      let result = this["execute"](str, jSONArray, callbackContext);
      console.log(`IRoot.execute result=${result}`);
      return false;
   };
});


```
Root bypassed.

![Bypassed](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgjfMRV1Ui-g5nYvmW8Ik3zN-ft3xPc5tqHoPNgnc3xqshZVjUjuLN6xf6rDiis0h6S-EWml2W-XGog3vvOQv3x2Zxdb23cSKEME99a7keaGPLRnVWAuzPa-_2UjbVl4uM4ccON-5vdWX5DMOXMFuVLdCOfHrc24tB6PxTxvu-mZCqC3uS2MvRTwrhEdg/s953)

Oke mungkin itu saja mengenai bypass root detection menggunakan Frida. Di artikel selanjutnya (jika saya sempat) kita akan membahas mengenai bypass SSL Pinning pada aplikasi.