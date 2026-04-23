# SOC Hari 02 — [tanggal]

## Yang dipelajari
- Struktur /var/log di Linux
- File log penting: auth.log, syslog, dpkg.log
- Cara deteksi brute force dari log
- Format parsing baris log
- journalctl vs /var/log

## Hasil praktik terminal
- 

## Quiz

1. File log mana yang pertama kamu buka kalau ada laporan
   "ada yang coba login ke server berkali-kali"?

`/var/log/auth.log`. File ini mencatat semua percobaan otentikasi, termasuk SSH, sudo, login lokal, dll.

2. Apa arti baris log ini:
   "Failed password for admin from 10.0.0.5 port 22 ssh2"?

Ada percobaan login gagal (password salah) ke user `admin` dari IP `10.0.0.5` melalui SSH ke port 22 (port default SSH). Ini indikasi serangan atau kesalahan password.

3. Command apa yang dipakai untuk hitung berapa kali
   satu IP gagal login?

sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

4. Bedanya /var/log/auth.log dan journalctl itu apa?

`/var/log/auth.log` adalah file teks khusus yang hanya mencatat aktivitas autentikasi (seperti login dan sudo), sedangkan `journalctl` adalah antarmuka untuk mengakses log sistem terstruktur dalam format biner milik systemd yang mencakup semua jenis log (termasuk autentikasi) dengan metadata lebih kaya seperti waktu presisi tinggi, unit service, dan prioritas.


5. Kalau kamu lihat 1000 baris "Failed password for root"
   dari satu IP dalam 2 menit — apa yang kamu laporkan
   ke Tier 2?

Melaporkan indikasi serangan brute force aktif terhadap user `root` dari IP tersebut. dengan menyertakan - P sumber (misal `10.0.0.5`), Jumlah percobaan (1000x dalam 2 menit = ~8 percobaan/detik, Waktu kejadian dan durasi, Rekomendasi: blokir IP sementara dengan firewall atau fail2ban


## Yang masih bingung
...

## Target besok
Windows Event Logs & Event ID penting