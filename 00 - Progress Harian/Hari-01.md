# SOC Hari 01 — [19-04-2026]

## Yang dilakukan hari ini
- Buat struktur folder SOC di Obsidian
- Daftar LetsDefend.io
- Memahami struktur Tier 1/2/3

## Kesan pertama tentang dunia SOC
Lumayan agak menarik, sepertinya harus teliti dan juga harus bisa mengambil keputusan yang tepat.

## Quiz

1. Apa yang dilakukan SOC Tier 1 setiap harinya?

SOC Tier 1 bertugas melakukan pemantauan terus-menerus terhadap layar monitoring yang menampilkan notifikasi keamanan dari berbagai perangkat seperti SIEM atau antivirus. Tugas utamanya adalah melihat setiap alert yang muncul dan melakukan pengecekan awal yaitu menentukan apakah alert tersebut serius atau sekadar alarm palsu. Jika ditemukan indikasi ancaman yang perlu analisis lebih dalam, alert tersebut akan diteruskan ke Tier 2. 

2. Bedanya True Positive dan False Positive itu apa?
   Berikan contoh masing-masing.

- **True Positive** adalah kondisi ketika alarm keamanan menyala dan memang benar terjadi insiden atau aktivitas berbahaya.  
    _Contoh:_ Sistem mendeteksi adanya proses enkripsi file secara tiba-tiba di komputer karyawan, dan setelah dicek ternyata benar terjadi serangan ransomware.
    
- **False Positive** adalah kondisi ketika alarm menyala padahal sebenarnya tidak terjadi ancaman apa pun.  
    _Contoh:_ Seorang pegawai salah mengetik kata sandi sebanyak lima kali berturut-turut, lalu sistem mengeluarkan notifikasi "Percobaan Serangan Brute Force", meskipun itu hanya kesalahan manusia biasa.

2. Apa itu IOC? Sebutkan 3 contoh bentuk IOC.

IOC merupakan singkatan dari Indicator of Compromise, yaitu jejak atau bukti digital yang menunjukkan kemungkinan suatu sistem telah dibobol atau disusupi. IOC bisa diibaratkan sebagai sidik jari pelaku kejahatan di dunia digital.

Tiga contoh bentuk IOC antara lain:

- Nilai hash (MD5, SHA256) dari file berbahaya yang sudah dikenal.
- Alamat IP server yang digunakan penyerang untuk mengendalikan komputer korban (Command & Control).
- Nama domain mencurigakan yang sengaja dibuat mirip dengan domain resmi untuk mengelabui korban (domain phishing).

4. Kenapa False Positive itu masalah buat SOC analyst?

False Positive menjadi masalah karena jumlahnya yang banyak dapat memicu kejenuhan atau yang sering disebut _alert fatigue_. Ketika setiap hari layar monitor dipenuhi notifikasi yang ternyata bukan ancaman, kewaspadaan analis dapat menurun. Akibatnya, ketika muncul alert yang benar-benar berbahaya (True Positive), respons yang diberikan bisa menjadi lambat atau bahkan terlewat karena sudah terbiasa menganggap alarm sebagai gangguan biasa.

5. Kalau kamu dapat alert "login gagal 50x dari IP asing",
   langkah pertama apa yang kamu lakukan?

Langkah pertama bukan langsung memblokir IP, melainkan mengumpulkan konteks tambahan. Pertama, periksa reputasi IP di layanan publik seperti VirusTotal untuk melihat apakah pernah dilaporkan jahat. Kedua, cek akun target percobaan login: apakah memiliki hak akses penting (misal administrator) atau hanya akun tamu biasa. Ketiga, lihat log keamanan lain: apakah IP tersebut juga mencoba mengakses layanan lain di jaringan. Jika hasil pengecekan menunjukkan IP benar-benar mencurigakan, lakukan pemblokiran sementara disertai pembuatan tiket untuk investigasi lanjutan. Jika belum ada riwayat buruk, cukup dicatat dan dipantau dalam waktu dekat.

## Yang belum jelas
...

## Target besok
Linux log analysis — baca /var/log/auth.log