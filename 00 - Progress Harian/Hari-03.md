# Hari 03 — [28-04-2026]

## Yang dipelajari
- Struktur Windows Event Log (Security, System, Application)
- Event ID penting: 4624, 4625, 4688, 4720, 7045, 1102
- Analisis skenario dari Event ID
- Sysmon dan kenapa lebih powerful dari Event Log biasa


## Event ID yang paling penting menurut saya
4625 karena menandakan bahwa ada yang mau masuk secara ilegal/tidak sah


## Quiz

1. Event ID berapa yang muncul kalau ada user baru dibuat
   di Windows?

4720

2. Kenapa Event ID 1102 sangat mencurigakan?

karena menandakan _audit log telah dibersihkan_, yang sering dilakukan penyerang untuk menghilangkan jejak aktivitas mereka.

3. Bedanya Windows Event Log biasa dengan Sysmon itu apa?

Windows Event Log standar mencatat kejadian sistem, aplikasi, dan keamanan dasar, namun terbatas pada kebijakan audit bawaan. 

Sysmon (System Monitor) adalah alat pihak ketiga yang mencatat lebih detail, seperti pembuatan proses, koneksi jaringan, perubahan registry, dan akses file, sehingga mampu mendeteksi aktivitas mencurigakan yang tidak terekam di event log biasa.

4. Kalau kamu lihat banyak Event ID 4625 dari satu IP
   dalam 3 menit, lalu tiba-tiba muncul 4624 dari IP sama
   — itu indikasi serangan apa?

Mengindikasikan serangan brute force yang berhasil.

5. Sysmon Event ID berapa yang dipakai untuk lihat
   proses mana yang melakukan koneksi jaringan?

Event ID 3,  digunakan untuk mencatat koneksi jaringan beserta proses yang melakukan koneksi tersebut.

## Yang masih bingung
...

## Target besok
MITRE ATT&CK Framework