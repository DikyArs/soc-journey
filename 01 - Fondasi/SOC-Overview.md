
### Tiga Tier SOC 

- Tier 1 – Alert Analyst  
  - Monitor dashboard SIEM, pilah alert, bedakan *true positive* (TP) vs *false positive* (FP). Teruskan ke Tier 2 jika perlu.  
  - Skill: dasar SIEM, baca log.

- Tier 2 – Incident Responder  
  - Investigasi mendalam, analisis malware, isolasi/hentikan ancaman.  
  - Skill: forensik, *threat hunting*.

- Tier 3 – Threat Hunter  
  - Cari ancaman tak terdeteksi, buat aturan deteksi baru.  
  - Skill: forensik lanjut, *reverse engineering*.

### Posisi Sebagai Intern (Tier 1)

- Tugas utama: penyaring pertama, bukan penyelesai insiden besar.  
- Ujung tombak penentu TP/FP:  
  - Salah klasifikasi TP → ancaman lolos.  
  - Terlalu banyak FP diteruskan → Tier 2 kewalahan.

### Alur Kerja Tier 1 Setiap Hari 

```
Alert masuk di SIEM  
       ↓  
Lihat konteks (user, IP sumber, waktu kejadian)  
       ↓  
Cek apakah IOC dikenal berbahaya  
       ↓  
Tentukan: True Positive atau False Positive?  
       ↓  
False Positive → dokumentasi & close  
True Positive  → eskalasi ke Tier 2 + buat tiket
```

Penjelasan langkah:

1. Alert masuk di SIEM – Notifikasi muncul karena aktivitas cocok dengan *rule* yang ada.  
2. Lihat konteks – Periksa user/sumber, IP asal (internal/eksternal), waktu kejadian (jam tidak biasa = mencurigakan). Konteks membantu menilai kewajaran.  
3. Cek IOC dikenal berbahaya – IOC (IP jahat, domain mencurigakan, hash file virus). Jika ada di *threat intelligence* → sinyal ancaman kuat.  
4. Tentukan TP atau FP – Gabungkan konteks + hasil cek IOC untuk putuskan.  
5. Jika False Positive – Dokumentasikan alasan FP (misal: IP cabang rutin scanning), lalu *close*. Dokumentasi penting agar tidak terulang.  
6. Jika True Positive – Eskalasi ke Tier 2 segera. Buat tiket lengkap berisi: waktu, sumber, target, IOC ditemukan, alasan TP. Jangan lakukan *containment* sendiri – itu tugas Tier 2.

### Istilah Wajib 

- SIEM – Sistem pengumpul log keamanan terpusat (dari server, firewall, antivirus) untuk deteksi aktivitas mencurigakan dan membunyikan alarm.  
- Alert – Pemberitahuan dari SIEM saat aktivitas cocok *rule*.  
- True Positive (TP) – Alert benar; aktivitas ancaman nyata (misal: serangan atau malware aktif).  
- False Positive (FP) – Alert keliru; aktivitas normal yang memicu alarm (misal: admin *update* di luar jam kerja).  
- IOC (Indicator of Compromise) – Petunjuk kompromi sistem (IP jahat, domain C2, hash file virus, string unik di log). Digunakan untuk verifikasi ancaman.  
- Triage – Proses sortir & prioritas alert: pilih yang paling mendesak (misal: *brute force* RDP dari IP luar) vs tunda (misal: 1× gagal login).  
- Escalation – Meneruskan kasus ke tier lebih tinggi karena perlu keahlian/wewenang/akses lebih, setelah yakin TP.  
- Playbook – Panduan langkah demi langkah penanganan serangan spesifik (contoh: *Playbook Ransomware* berisi matikan jaringan, ambil sampel, cek hash di sandbox). Tier 1 harus tahu *playbook* relevan saat *triage*.
