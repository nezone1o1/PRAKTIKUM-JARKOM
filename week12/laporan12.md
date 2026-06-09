Berikut adalah hasil parafrase dari modul praktikum ICMP Anda tanpa menggunakan emotikon. Penyampaian kalimat telah diubah menjadi lebih bervariasi dan profesional dengan tetap menjaga struktur serta akurasi teknis laporan.

---

# Modul 12 ICMP (Internet Control Message Protocol)

### Analisis Perilaku Pesan Kendali Jaringan dan Mekanisme Diagnostik Utilitas Jaringan

Nama    :Samuel Nelson Wabiser
NIM     : 103072400111

Kelas   : IF-04-04

---

## Daftar Isi

* [Dasar Teori ICMP]
* [Eksperimen 1: Analisis Paket ICMP Ping]
* [Eksperimen 2: Analisis Paket ICMP Traceroute]

---

## Dasar Teori ICMP

ICMP digunakan oleh perangkat host maupun router untuk saling bertukar informasi pada lapisan network, terutama dalam mendeteksi gangguan jaringan (error reporting) atau melakukan uji diagnostik. Paket ICMP ini dibungkus langsung di dalam enkapsulasi datagram IP dengan nilai field Protocol sama dengan 1.

Struktur dasar dari header ICMP tersusun atas 3 komponen utama yang selalu ada pada tiap jenis paketnya:

* **Type (8-bit)** berfungsi untuk menentukan kategori makro atau jenis dari pesan ICMP.
* **Code (8-bit)** berfungsi untuk memberikan keterangan tambahan yang lebih spesifik berdasarkan kategori pada field Type.
* **Checksum (16-bit)** berfungsi untuk memvalidasi integritas data paket ICMP dari potensi kerusakan bit selama proses transmisi.

---

## Eksperimen 1: Analisis Paket ICMP Ping

Pengujian skenario ini dilakukan dengan menjalankan perintah `ping -n 10 www.ust.hk` melalui terminal Windows, yang memicu terjadinya pengiriman dan penerimaan paket kueri dua arah secara berurutan.
![ping -n 10 www.ust.hk](../Assets/Modul12.1.png)
### 1. Alur Transmisi Paket

Melalui rekaman aktivitas pada Wireshark, proses uji koneksi ke server `www.ust.hk` dengan alamat IP `143.89.209.9` dari perangkat lokal pengirim ber-IP `10.218.2.21` memperlihatkan proses pertukaran pesan kueri ICMP yang saling berpasangan dan berjalan sekuensial.

### 2. Bedah Struktur Data Header ICMP (Wireshark)

#### **A. Paket ICMP Echo Request (Outbound)**

Merupakan paket kueri yang dikirimkan oleh komputer asal menuju ke server tujuan:

* **Type:** `8` (Echo / Ping request)
* **Code:** `0`
* **Checksum:** `0x4c20` (Valid)
* **Identifier (BE):** `1` (`0x0001`)
* **Identifier (LE):** `256` (`0x0100`)
* **Sequence Number (BE):** `315` (`0x013b`)
* **Sequence Number (LE):** `15105` (`0x3b01`)
* **Payload Data:** Memuat beban data standar dengan ukuran sebesar 32 byte.
![ICMP Echo Request](../Assets/Modul12.2.png)
#### **B. Paket ICMP Echo Reply (Inbound)**

Merupakan paket balasan yang dikirimkan oleh server target `143.89.209.9` kembali ke perangkat asal:

* **Type:** `0` (Echo / Ping reply)
* **Code:** `0`
* **Checksum:** `0x5420` (Valid)
* **Identifier (BE):** `1` (`0x0001`) — *identik dengan paket request*
* **Identifier (LE):** `256` (`0x0100`) — *identik dengan paket request*
* **Sequence Number (BE):** `315` (`0x013b`) — *identik dengan paket request*
* **Sequence Number (LE):** `15105` (`0x3b01`) — *identik dengan paket request*
* **Payload Data:** Memuat beban data standar sebesar 32 byte — *identik dengan paket request*

> **Analisis:** Kesamaan parameter nilai *Identifier* serta *Sequence Number* pada paket balasan (Reply) menjadi parameter penting bagi sistem operasi pengirim untuk mencocokkan setiap jawaban dengan permintaan yang dikirim sebelumnya. Hal inilah yang mendasari kalkulasi nilai durasi *Round-Trip Time* (RTT) secara akurat.

---

## Eksperimen 2: Analisis Paket ICMP Traceroute

Skenario berikutnya dijalankan dengan mengeksekusi perintah `tracert www.inria.fr`. Utilitas ini bekerja dengan cara memanipulasi parameter nilai TTL (*Time to Live*) pada header IP secara bertahap mulai dari nilai TTL = 1.

### 1. Hasil Eksekusi Utilitas `tracert` pada Terminal
![tracert](../Assets/Modul12.7.png)
Setelah mengakses direktori `C:\Windows\System32` lewat Command Prompt atau PowerShell, baris log menunjukkan bahwa sistem berhasil memetakan rute lompatan awal beserta informasi latensinya:

* **Hop 1:** Mengarah ke gerbang utama (default gateway) jaringan lokal kampus pada IP **10.252.241.1** dengan waktu respons yang sangat minim yaitu `1 ms`, `1 ms`, `1 ms`. Catatan waktu ini mengindikasikan bahwa interkoneksi pada area lokal (LAN) berjalan sangat optimal.
* **Hop berikutnya:** Paket data mulai diteruskan keluar dari jaringan lokal menuju ke infrastruktur penyedia layanan internet (ISP) publik sebelum akhirnya diarahkan menuju rute antarnegara.

### 2. Analisis Paket pada Wireshark (Mekanisme TTL & Pesan Kesalahan)

Melalui pengamatan log paket yang terekam pada Wireshark, dapat diidentifikasi karakteristik dari mekanisme kerja fungsi `tracert` pada platform Windows:

#### **A. Paket Request dari Klien (`192.168.0.192`)**

* Berbeda dari sistem operasi berbasis Unix atau Linux yang umumnya mengandalkan probe UDP port tinggi, utilitas `tracert` pada Windows terbukti memanfaatkan jenis paket **ICMP Echo Request**.
* **Type:** `8` | **Code:** `0`
* **Manipulasi TTL:** Paket data pertama dilepas ke jaringan dengan karakteristik lapisan IP ber-TTL = 1. Pada paket-paket berikutnya, nilai parameter TTL ini dinaikkan secara bertahap menjadi TTL = 2, TTL = 3, dan seterusnya.
![alt text](../Assets/Modul12.8.png)
#### **B. Paket Respons dari Router Perantara**

* Saat paket ber-TTL = 1 tiba di perangkat router pada lompatan pertama, nilai TTL tersebut langsung dikurangi 1 oleh sistem router hingga menyentuh angka **0**.
* Akibat nilai TTL yang telah habis, router akan membuang paket tersebut lalu mengirim balik sinyal pemberitahuan kesalahan (Error Message) menuju ke host pengirim.
* **Type:** `11` (Time-to-live exceeded)
* **Code:** `0` (Time to live exceeded in transit)
* **Checksum:** `0xf4ff` (Status: Good/Valid)
* **Payload Penyerta:** Pada paket Type 11 ini, perangkat router menyisipkan kembali duplikat dari header IP asal beserta payload data asli yang sempat dibuangnya. Langkah ini ditujukan agar susunan protokol pada komputer klien dapat mengenali secara spesifik paket permintaan mana yang memicu terjadinya peristiwa *drop* di router tersebut.

---

## Kontribusi

Laporan praktikum ini disusun untuk memenuhi salah satu tugas akademik pada program studi S-1 Informatika Telkom University. Apabila Anda bermaksud memberikan masukan, koreksi, ataupun pengembangan lebih lanjut terhadap dokumentasi ini, silakan ikuti prosedur berikut:

1. Lakukan **Fork** pada repositori ini.
2. Buat branch kerja baru yang akan menampung penambahan fitur atau perbaikan dari Anda.
3. Kirimkan **Pull Request** yang dilengkapi dengan deskripsi ringkas mengenai poin-poin perubahan yang dilakukan.

---

**Informatics Lab - Telkom University**
*Copyright © 2026 - UKM Coder*