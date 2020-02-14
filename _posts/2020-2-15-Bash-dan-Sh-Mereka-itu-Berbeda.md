---
layout: post
title: Bash != Sh (Mereka itu berbeda)
categories: [Scripting Bash]
author: Edo Maland
---

# BASH != SH

## Something Different

![enter image description here](https://images5.alphacoders.com/520/thumb-1920-520207.jpg)

Sebenarnya **bash dengan sh itu tidaklah sama** , kita tidak bisa bilang **sh** itu adalah **bash**.  Kalau kita ngomongnya dari **standarisasi** sh itu adalah **POSIX** ( Portable Operating System Interface ) untuk sistem shell. 

Dimana fungsi **/bin/sh** itu adalah untuk **menetapkan default** dari system, tanpa kita harus melihat dulu **shell** apa yang kita gunakan. Oiya fyi, Shell dan terminal itu berbeda yah. Untuk penjelasan lebih detail perbedaan bash dan terminal bisa lihat video dibawah ini.

[![IMAGE ALT TEXT HERE](https://user-images.githubusercontent.com/17976841/74555102-94950a80-4f8d-11ea-9cfd-91c0183fe114.png)](https://youtu.be/Yt57-gg9jVg)

Kalo kita analogikan , ceritanya hampir sama seperti ini . 
Ada hewan reptil yang ber **Ordo Crocodilia** yang dimana memiliki **standarisasi** . Agar bisa masuk ke dalam ordo tersebut, reptil tersebut harus  memiliki ciri ciri sebagai berikut : 

 - Kulit tebal mengandung kepingan tulang yang tersusun berderet dan
   berlunas
 - Kepala berbentuk pyramid, keras dan kuat, 
- dilengkapi dengan gigi runcing bertipe gigi poliodont. 
- Mata kecil terletak dibagian kepala yang menonjol di dorsal-latera.

Oke, kalau sudah cukup paham coba lihat dan perhatikan seksama gambar dibawah ini :
Menurut teman teman, gambar diatas adalah hewan apa ? 

![enter image description here](https://miro.medium.com/max/1415/1*W3qTqtjsegoamoy68Qul2Q.png)


Saya yakin **70%** dari kalian bakal ngejawab kalau dua gambar itu  adalah **BUAYA**  (bukan buaya darat ). 

Sebenarnya dari gambar diatas , itu adalah dua gambar yang berbeda dimana gambar sebelah kiri adalah **Buaya** dan gambar disebelah kanananya adalah **Alligator** . Kalau dilihat sekilas emang mereka kelihatan mirip , tapi coba deh kita lihat lebih detail lagi kearah moncongnya . 

Disana **Buaya** memiliki moncong panjang dan menyempit, **cenderung membentuk huruf “V”**. Sementara **Aligator memiliki moncong yg pendek, lebar, dan berbentuk huruf “U”**. 

> Walupun mereka memiliki **feature** yang berbeda tapi mereka telah terstandarisasi sehingga termasuk dari **Ordo Crocodilia .**

 Jadi peran si sh itu hampir sama seperti Ordo Crocodilia  ini, yang  dimana `bash,dash,ksh dan csh` adalah **jenis shell** yang telah **terstandariasi** sehingga bisa disebut **shell**. 
 
 ![enter image description here](https://cdn.psychologytoday.com/sites/default/files/field_blog_entry_images/2018-10/171026-f-rn211-001.jpg)

## Lalu kenapa kebanyakan orang masih beranggapan bahwa sh itu adalah Bash 

###  1. Salah Kaprah 
Karena kebanyakan distro saat ini, mengarahkan sh untuk menggunakan jenis shell bash atau bisa kita sebut symbolic **( sh symbolic point to bash )**.

![enter image description here](https://www.pngkey.com/png/detail/315-3152007_png-animuthinku-thinking-meme-face-anime.png)


Tidak **semua** distro contohnya debian ( Linux Kali 4.9.0)  symbolic ke **bash** melainkan ke **dash**. Untuk melihat symbolic kemanakah sh kalian . Kalian bisa gunakan perintah dibawah ini

    root@emaland:~# file -h /bin/sh

Inilah penyebabnya script yang kalian download atau yang telah kalian buat Error dan Fail di distro yang  sebenarnya **tidak symbolic ke bash** . Ini adalah **sebagain** contoh bagian dari kode bash yang tidak bisa jalan di shell lain seperti **/bin/dash** atau kita menyebutnya ( **Bashism )**.

    root@emaland:~# echo "echo $'Jarak\tKita'" > program  
    root@emaland:~# dash program
    $Jarak   Kita
    
    root@emaland:~# bash program
    Jarak   Kita
    
Kode diatas tidak akan berjalan dengan baik di shell dengan jenis dash, sedangkan di saat kita menjalankan program tersebut menggunakan shell bash, outputnya akan baik baik saja tanpa adanya symbol **’$ ‘**.       Munculnya symbol **$’ ‘** ini **disebabkan** karena tidak didefinisikan oleh **POSIX hingga 2008.**  

![image](https://user-images.githubusercontent.com/17976841/74555594-a4611e80-4f8e-11ea-8e73-899fd07af823.png)
Kalau masih penasaran tentang **Bashism** ini bisa mengunjunginya [disini](https://mywiki.wooledge.org/Bashism)

### 2. Shebang ( Sharp & Bang )
![enter image description here](https://linuxize.com/post/bash-shebang/featured.jpg)

Sebuah line yang diawali dengan prefix #! yang menuju ke path interpreter yang akan digunakan, dimana program itu akan di eksekusi melalui GNU/Linux.

    #!/bin/bash <- ini adalah shebang 

![enter image description here](https://miro.medium.com/max/1059/0*KjWdichzMtZNI1W1.jpg)

Jadi shebang apa yang digunakan di file scripting kita ? 

**Shebang** yang kalian gunakan **tergantung** dari symbolic system yang kita gunakan. Apabila **/bin/sh symbolic ke /bin/bash** itu tidak akan menjadi masalah, yang menjadi **masalah** adalah di saat **/bin/sh** kita tidak **symbolic ke **/bin/bash .**** 

       #!/bin/bash
Jadi alangkah bagusnya di saat membuat shebang, langsung  di arahkan saja interpreternya ke bash misalnya ataupun dash jika ingin menggunakan dash.

### 3. File Ekstensi

![enter image description here](https://miro.medium.com/max/1521/1*J4GPASLHTAMEwums35krng.png)

Penggunaan ektensi **.sh** ini bukanlah untuk menujukan kalau dia adalah ektensi yang dimiliki untuk **bash.** Sedikit cerita, di jaman dulu ektensi ini bertujuan untuk membedakan mana yang **binary dan mana yang scripting** ,  serta untuk pengingat juga *kalau lupa.

Apabila sudah menggunakan **shebang** di script kita , penggunaan ektensi yang membosankan ini tidak dibutuhkan sama sekali . Kita cukup memberikan file permsion executable di program kita dan kemudian dijalankan seperti perintah dibawah ini 

    chmod +x scripting  
    root@emaland:~# ./scripting

Apabila kita memberikan fie execute **tanpa adanya shebang** di awal , maka script akan **memanggil shell default** apa yang telah ditetapkan oleh terminal. Untuk mengecek shell default apa yang digunakan oleh system kalian ketikan perintah :

    root@emaland:~# echo $SHELL


Mungkin pembahasan tentang Shell and Bash ini sampai disini dulu, jikalau ada kurangnya mohon maaf dan apabila ada masukan dan tambahan sangat diterima, Okelah kalau begitu. Terimakasih, kalo di rasa tulisan ini bermanfaat, silahkan **Share**.  


Semoga kebermanfaatan ini terus berlanjut!
