---
layout: post
title: Mencatat login SSH pada Linux Server
categories: [Auditing]
author: Rafshanzani Suhada
---

Bisakah kita dapat mengetahui siapa saja yang masuk kedalam Server menggunakan SSH Service? tentu saja bisa, kita hanya perlu mencari tahu lokasi file log pada server tersebut. Umumnya lokasi file log pada **Ubuntu** berada di `/var/log/auth.log` dan **Centos** berada di `/var/log/secure`

**Ubuntu**
```
grep 'sshd' /var/log/auth.log
```

**Centos**
```
grep 'sshd'/var/log/secure
```

**Output**
```
[...]
Feb 17 08:26:01 Suhada sshd[16059]: Accepted password for root from xx.xxx.x.xxx port xxx ssh2
Feb 17 08:26:01 Suhada sshd[16059]: pam_unix(sshd:session): session opened for user root by (uid=0)
```

Namun disini kita akan setting supaya setiap user yang login SSH akan tercatat pada Server.

**Oke let's go!**

![Topologi Sederhana](https://i.ibb.co/zxPm6sB/Whats-App-Image-2020-02-21-at-13-45-44.jpg "Topologi Sederhana")

Di atas adalah contoh topologi sederhana yang akan kita bahas.

```
SERVER : 172.16.1.2/25
TESTING USER : 172.16.1.121/25
USER 1 : 172.16.1.120/25

== Informasi SSH Users ==

 # TESTING USER # 

USER : usertesting
PASS : passwordnyatesting
```

**Note:** Pastikan SSH (Server) sudah terinstall pada Server

- Ubuntu
```
$ sudo apt-get install openssh-server
```

- Centos
```
$ yum install openssh-server
```

# Sisi Client

Lakukan ssh menggunakan **TESTING USER**

```
$ ssh usertesting@172.16.1.2
usertesting@172.16.1.2's password: **********
```

Untuk melakukan pengecekan SSH Connection, kalian dapat menggunakan variable `$SSH_CONNECTION`.

- **Contoh**
```
$ echo $SSH_CONNECTION
172.16.1.121 54088 172.16.1.2 22
```

- **Penjelasan**
 * `172.16.1.121` = IP Address User (TESTING USER)
 * `54088` = Port SSH yang dibuka pada sisi Client untuk melakukan remote ke Server (akan terbuka dengan sendirinya secara random)
 * `202.89.117.69` = IP Address Server
 * `22` = Default Port SSH pada Server

Untuk kebutuhan Logging, kita harus parsing terlebih dahulu, supaya dapat mengambil output dari `IP Address User`, `Port SSH Server`, dan `IP Address Server` saja.

```
Bagimana caranya ? (¬‿¬)
```

![Berfikir dulu](https://i.kym-cdn.com/entries/icons/mobile/000/032/100/cover4.jpg "Berfikir dulu")

## Perintah untuk Parsing `$SSH_CONNECTION`

- **Mengambil IP Address User**

```
$ echo $SSH_CONNECTION | cut -d ' ' -f 1   # CARA PERTAMA DENGAN CUT
172.16.1.121
$ echo $SSH_CONNECTION | awk '{ print $1}'   # CARA KEDUA DENGAN AWK
172.16.1.121
```

- **Mengambil Default Port SSH Server**

```
$ echo $SSH_CONNECTION | cut -d ' ' -f 4
22
$ echo $SSH_CONNECTION | awk '{ print $4}'
22
```

- **Mengambil IP Address Server**

```
$ echo $SSH_CONNECTION | cut -d ' ' -f 3
172.16.1.2
$ echo $SSH_CONNECTION | awk '{ print $3}'
172.16.1.2
```

Jika sudah mendapatkan informasi tersebut lalu bagaimana cara agar setiap users yang login, Server akan mencatat aktifitas tersebut pada kita?

Oke akan saya jelaskan terlebih dahulu.

# Sisi Server

Pertama yang dilakukan tentu saja masuk kedalam server 😂.
Buatlah file `ssh-login.sh` ( filename terserah diisi dengan apa saja. ) dan edit file tersebut seperti script dibawah ini & pindahkan file ke `/etc/profile.d/`.

```
root@zerobyteid:~# cat ssh-login.sh
#!/bin/bash
# SCRIPT UNTUK MENCATAT USER YANG LOGIN

IP_ADDRESS_USER=$(echo $SSH_CONNECTION | cut -d ' ' -f 1)
PORT_SSH=$(echo $SSH_CONNECTION | awk '{ print $4}')
IP_ADDRESS_SERVER=$(echo $SSH_CONNECTION | cut -d ' ' -f 3)

# VARIABLE UNTUK TIMESTAMP

TANGGAL=$(date "+%Y-%m-%d %H:%M:%S")

# PESAN

echo "User ${USER} telah masuk kedalam server (${IP_ADDRESS_SERVER}) menggunakan IP ADDRESS ${IP_ADDRESS_USER} PORT ${PORT_SSH} Pada ${TANGGAL}" >> /tmp/user-login-ssh.txt
echo "------------------------------" >> /tmp/user-login-ssh.txt

root@zerobyteid:~# mv ssh-login.sh /etc/profile.d/
root@zerobyteid:~# chmod +x /etc/profile.d/ssh-login.sh
```

**Note:** /tmp/user-login-ssh.txt => Lokasi Log File yang akan digunakan untuk mencatat setiap user masuk kedalam Server menggunakan SSH


# Testing

Untuk melakukan uji coba lakukan SSH dari `usertesting` ke `Server`.

- **User Testing**
```
$ ssh usertesting@172.16.1.2
root@172.16.1.2's password: **********
```

Bilamana user sudah melakukan SSH ke Server, maka setelah itu lakukan pengecekan pada Server dan pastikan apakah Script tersebut sudah mencatat User yang Login.

- **Server**
```
root@zerobyteid:~# cat /tmp/user-login-ssh.txt
User usertesting telah masuk kedalam server (172.16.1.2) menggunakan IP ADDRESS 172.16.1.121 PORT 22 Pada 09 Mar 2020 16:40
------------------------------
```

Maka ketika `user1` melakukan login ke server melalui SSH maka Script akan tetap mencatat nya

```
root@zerobyteid:~# cat /tmp/user-login-ssh.txt
User usertesting telah masuk kedalam server (172.16.1.2) menggunakan IP ADDRESS 172.16.1.121 PORT 22 Pada 09 Mar 2020 16:40
------------------------------
User usersatu telah masuk kedalam server (172.16.1.2) menggunakan IP ADDRESS 172.16.1.120 PORT 22 Pada 10 Mar 2020 10:04
------------------------------
```

