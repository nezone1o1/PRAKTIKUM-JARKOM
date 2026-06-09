

# Modul 11 DHCP (Dynamic Host Configuration Protocol)

### Analisis Alokasi IP Dinamis via Wireshark

Nama    : Samuel Nelson Wabiser
NIM     : 103072400111
Kelas   : IF-04-04

---

## Daftar Isi

* [Konsep Dasar (Proses DORA)]
* [Prosedur Percobaan]
* [Bedah Paket DHCP]
* [Kesimpulan]

---

## Konsep Dasar

Dalam sistem DHCP, distribusi alamat IP dari server ke perangkat klien dilakukan melalui empat tahapan utama yang dikenal sebagai proses DORA:

1. **DHCP Discover**: Klien mengirimkan pesan siaran (broadcast) ke seluruh jaringan untuk mendeteksi keberadaan server DHCP yang aktif.
2. **DHCP Offer**: Server yang menerima pesan tersebut akan membalas dengan menawarkan sebuah alamat IP beserta konfigurasi jaringan lainnya.
3. **DHCP Request**: Klien merespons tawaran tersebut dengan mengirimkan permintaan resmi untuk menggunakan alamat IP yang telah dialokasikan.
4. **DHCP ACK**: Server memberikan konfirmasi final (acknowledgment) bahwa alamat IP tersebut telah resmi dipinjamkan dan siap digunakan oleh klien.

---

## Percobaan

1. Aktifkan aplikasi Wireshark, lalu tentukan interface jaringan yang sedang terkoneksi ke internet.
2. Filter lalu lintas data pada Wireshark dengan mengetik kata kunci `dhcp` atau `bootp`.
3. Jalankan Command Prompt (CMD) menggunakan hak akses Administrator.
4. Lepaskan konfigurasi IP yang saat ini menempel pada perangkat dengan mengetik perintah `ipconfig /release`.
![ipconfig /release](../assets/Modul11.1.png)
5. Pastikan Wireshark mulai merekam lalu lintas data yang masuk dan keluar.
6. Minta alokasi alamat IP baru dari server DHCP dengan mengeksekusi perintah `ipconfig /renew`.
![ipconfig /renew](../assets/Modul11.2.png)
7. Tunggu hingga proses pencarian IP selesai, kemudian hentikan perekaman (stop capture) di Wireshark.
![hasil capture](../assets/Modul11.3.png)

---

## Bedah Paket DHCP

Berdasarkan data paket yang berhasil direkam pada Wireshark, berikut adalah poin-poin analisis pentingnya:

### 1. Lapisan Transport (UDP)

Protokol DHCP memanfaatkan jalur komunikasi **UDP** (User Datagram Protocol).

* **Port Asal (Source Port)**: 68.
* **Port Tujuan (Destination Port)**: 67.

### 2. Penelusuran Pesan DORA dalam Jaringan

| Kategori Pesan | IP Asal (Source) | IP Tujuan (Destination) | Penjelasan Operasional |
| --- | --- | --- | --- |
| **Discover** | `0.0.0.0` | `255.255.255.255` | Klien menyiarkan pesan ke jaringan karena belum punya IP. |
| **Offer** | `10.218.0.253` | `10.218.2.21` | Server mengirimkan rekomendasi alamat IP untuk klien. |
| **Request** | `0.0.0.0` | `255.255.255.255` | Klien menyiarkan permintaan untuk menyetujui tawaran IP tersebut. |
| **ACK** | `10.218.0.253` | `10.218.2.21` | Server memvalidasi bahwa IP resmi disewakan ke klien. |

### 3. Komponen Utama pada Header DHCP

* **Transaction ID**: Sebuah kode identifikasi unik (misalnya: `0xbc42c45e`) yang berfungsi menyelaraskan setiap pesan request dari klien dengan jawaban dari server agar tidak tertukar.
* **Client IP Address**: Alamat network spesifik yang dialokasikan oleh DHCP server agar bisa dipakai oleh perangkat klien.

---

## Kesimpulan

Praktikum ini membuktikan bahwa DHCP mengandalkan sifat connectionless dari protokol UDP. Penggunaan alamat broadcast (`255.255.255.255`) mutlak diperlukan pada fase awal karena perangkat klien sama sekali belum memiliki identitas IP formal di jaringan tersebut. Selain itu, eksistensi **Transaction ID** memegang peranan vital untuk menjamin akurasi distribusi IP, sehingga tidak terjadi tumpang tindih konfigurasi meskipun lalu lintas data di dalam jaringan sedang padat.

---

## Kontribusi

Dokumen laporan ini ditulis guna memenuhi penugasan praktikum pada program studi S-1 Informatika di Telkom University. Bagi Anda yang ingin memberikan masukan, kritik, maupun perbaikan terhadap isi dokumentasi ini, silakan ikuti panduan berikut:

1. Lakukan **Fork** pada repositori ini.
2. Buatlah branch kerja baru untuk menampung fitur atau revisi Anda.
3. Ajukan **Pull Request** disertai deskripsi mendalam mengenai perubahan yang Anda ajukan.

---

**Informatics Lab - Telkom University**
*Copyright © 2026 - UKM Coder*