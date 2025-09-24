<p align="center"><img src="docs/assets/logo-tresno.png" alt="TRESNO Logo" width="160"/></p>

# TRESNO Product Docs (tresno-product-docs)

Dokumentasi produk dan operasi **TRESNO V2** (Merapi). Repositori ini menyajikan versi **Markdown** untuk GitHub dan versi **DOCX** untuk dokumen formal.

## Struktur
- `docs/product/` — Deskripsi produk/teknis (versi terukur & siap presentasi).
- `docs/specs/` — Finalisasi spesifikasi & payload (spec freeze).
- `docs/roadmap/` — Roadmap mingguan Jul–Okt 2025 (MD + Excel + DOCX).
- `docs/templates/` — Template pengujian, rilis, operasi, dan data lapangan.
- `docs/sticker/` — Sticker card lapangan (ringkas).
- `.github/profile/` — README untuk profil organisasi GitHub.

> Tanggal build: 2025-09-24

# TRESNO V2 — Dokumen Produk & Rancangan Teknis
Tracking Based on Smart GPS Technology untuk kawasan rawan bencana Merapi
## Ringkasan Eksekutif
TRESNO adalah sistem navigasi dan pelacakan dua arah berbasis LoRa + GNSS untuk membantu pengawasan dan penyelamatan di kaki Gunung Merapi. Versi 2 (TRESNO V2) mengatasi keterbatasan V1 melalui arsitektur LoRa mesh berbasis peran (role‑based)—Navigator (pengunjung), Relay (penguat jangkauan), dan Gateway (basecamp). Sistem ini memungkinkan pengunjung mendapat arah kembali ke basecamp, relawan memantau posisi pengunjung, serta pelaksanaan pencarian tanpa bergantung pada internet atau infrastruktur Wi‑Fi yang mahal.

## 1) Latar Belakang & Masalah
- Situasi: Kawasan pengawasan meliputi area wisata hutan pinus dan jalur pencarian pakan ternak, dengan medan berbukit, pepohonan rapat, dan jalur tempuh tidak rata.
- Kendala Lapangan:
- Pengunjung kerap tidak turun sebelum pukul 18.00 dan berisiko tersesat/tertahan gelap.
- Relawan terbatas dan rawan kelelahan saat melakukan pencarian manual.
- Komunikasi konvensional (HT) terbatas jangkauan dan hanya antar‑relawan; tidak menyediakan lokasi pengunjung secara langsung.
- Dampak:
- Pencarian sulit, memakan waktu, dan berbahaya bagi pengunjung maupun relawan.
- Pernah terjadi insiden kehilangan nyawa.
Kebutuhan inti: Sistem yang tangguh, minim infrastruktur, dan andal untuk navigasi dua arah serta pemantauan lokasi real‑time tanpa internet.

## 2) Rekap TRESNO V1 (Sejarah & Pelajaran)
Arsitektur V1:
- Mobile tracker (±5×5×4 cm): GNSS u‑blox Neo‑6M, LoRa RA‑02 (SX1278), OLED 0,96″, sensor suhu/kelembaban, baterai Li‑ion 123 (kapasitas tak diketahui), charger TP4056.
- Basecamp station: Antena yagi sudut beam 90°, Raspberry Pi + layar sentuh untuk webapp, UPS portable.
Temuan/Kendala:
- Jangkauan hanya 200–400 m (non‑LOS, banyak obstruksi).
- Kelembaban & debu mengganggu elektronik.
- Daya: baterai cepat habis; pengisian lambat, proteksi minim.
- Minim dokumentasi: membuat relawan/operator kebingungan.
- Toubleshooting: sulit dilakukan oleh orang awam.
- Outcome: Tidak memenuhi kebutuhan operasional; tidak dipakai relawan.
Pelajaran: Diperlukan arsitektur multi‑hop (mesh), ketahanan lingkungan, manajemen daya yang jauh lebih baik, dokumentasi dan pengujian yang lengkap, serta alat analisis kinerja untuk mempercepat troubleshooting.

## 3) Visi & Sasaran TRESNO V2
Visi: Menyediakan jalur pulang yang jelas bagi pengunjung dan visibilitas posisi bagi relawan dalam kondisi minim infrastruktur.
Sasaran Terukur (KPI):
- Cakupan efektif: ≥ 3 km (non‑LOS bertingkat) melalui kombinasi relay.
- Keandalan pesan (PDR): ≥ 90% untuk beacon lokasi periodik di jaringan mesh.
- Waktu dari SOS ke alarm gateway: ≤ 30 detik pada kondisi mesh nominal.
- Daya tahan Navigator: ≥ 24 jam operasional (profil campuran: tracking, navigasi layar, sleep) dengan baterai 3600 mAh.
- Ketahanan lingkungan: perlindungan setara IP‑rating fungsional melalui desain casing minim lubang + conformal coating pada PCB.

## 3a) KPI Terukur (Definisi, Perhitungan, Ambang)
Di bawah ini adalah himpunan KPI yang praktis dan bisa langsung diturunkan dari data yang sudah tersedia (throughput, link margin, PER/Loss %, latensi per hop, RSSI/SNR per hop). Untuk tiap KPI dicantumkan definisi singkat, cara hitung, serta tiga tingkat target: Good / Target / Stretch.
### KPI Inti (Network‑level)
- PDR End‑to‑End (PDR_e2e)
Definisi: Persentase paket Navigator yang tiba di Gateway (apa pun jumlah hop).
Hitung: PDR_e2e = 1 − PER_e2e (PER_e2e dari Loss%).
Ambang: ≥85% / ≥90% / ≥95%.
- Latensi End‑to‑End (p50/p95/p99)
Definisi: t_rx_gateway − t_tx_nav.
Ambang (beacon periodik): ≤10 s / ≤5 s / ≤3 s.
Ambang (SOS): ≤45 s / ≤30 s / ≤15 s.
- Cakupan Efektif Multi‑hop
Definisi: Jarak max Nav→GW dengan PDR_e2e ≥ 90% dan p95 latensi SOS ≤ 30 s.
Ambang: ≥2 km / ≥3 km / ≥4 km (dengan penempatan relay tepat).
- Keandalan SOS
Definisi: Persentase event SOS yang memicu alarm Gateway + latensinya.
Hitung: (SOS_terdeteksi / SOS_dikirim), dan p95 latensi SOS.
Ambang: ≥99%, p95 ≤ 30 s (Target).
- Link‑Margin Minimum per Rute (p5)
Definisi: Persentil ke‑5 link margin terendah di semua hop rute e2e.
Hitung: LM = RxPower(dBm) − Sensitivitas(dBm) sesuai profil SF/BW/CR LLCC68.
Ambang: ≥ +3 dB / ≥ +6 dB / ≥ +10 dB (p5).
Jika tabel sensitivitas belum dipetakan, gunakan pendekatan: SNR p5 ≥ SNR_min(SF)+6 dB.
- Efisiensi Flooding (Redundancy Factor)
Definisi: Rata‑rata jumlah transmisi radio (termasuk relay & duplikat) per satu paket e2e sukses.
Hitung: total_tx_radio / delivered_e2e (butuh log TX dari relay).
Ambang: ≤6 / ≤4 / ≤2.5 (lebih kecil lebih efisien).
- Stabilitas Rute
Definisi: Frekuensi perubahan rute utama per jam (route‑change rate).
Hitung: Deteksi perubahan next‑hop dominan sepanjang waktu.
Ambang: ≤6/jam / ≤3/jam / ≤1/jam.
- Throughput Efektif End‑to‑End (Goodput)
Definisi: Laju payload bersih yang diterima Gateway.
Hitung: bit payload bersih per detik; opsional dinormalisasi terhadap kapasitas PHY profil yang dipakai.
Ambang relatif: ≥15% / ≥25% / ≥35% dari kapasitas bersih profil (gunakan sebagai indikator kapasitas, bukan patokan regulasi duty‑cycle).
### KPI Per‑Hop (Diagnostik & Tuning)
- Latensi per Hop (p95)
Ambang: ≤2 s / ≤1 s / ≤0.5 s per hop.
- RSSI & SNR per Hop (p5)
Ambang umum: RSSI p5 ≥ −112 dBm (konservatif Sub‑GHz), SNR p5 ≥ SNR_min(SF)+6 dB.
Catatan: Lebih representatif menggunakan link‑margin p5 (lihat KPI #5).
- PER per Hop (PER_hop)
Ambang: ≤10% / ≤5% / ≤2%.
### KPI Operasional & Energi
- Kepatuhan Interval Beacon (On‑Time Rate)
Definisi: Persentase beacon yang tiba ≤ 1.5× periode yang diatur.
Ambang: ≥85% / ≥92% / ≥98%.
- Daya Tahan Navigator (Profil Campuran)
Ambang: ≥18 jam / ≥24 jam / ≥36 jam (baterai 3600 mAh).
- Energi per Paket Sukses (mWh/packet)
Hitung: (ΔmAh × V) / packets_delivered_e2e.
Ambang: ≤2.0 / ≤1.0 / ≤0.5 mWh (indikatif; kalibrasi setelah uji awal).
- Uptime Relay & Gateway
Ambang bulanan: ≥99.0% / ≥99.5% / ≥99.9%.
- Waktu Pemulihan (Recovery Time)
Definisi: Dari gangguan lokal (node restart/link drop) hingga jaringan kembali PDR_e2e ≥ 90%.
Ambang: ≤60 s / ≤30 s / ≤15 s.
### KPI Kapasitas & Skala
- Jumlah Navigator Serentak (PDR_e2e ≥ 90%)
Ambang awal: ≥10 / ≥20 / ≥30 node (disesuaikan duty‑cycle/regulasi).
- Waktu Penemuan Rute (Route Discovery Time)
Ambang: ≤30 s / ≤15 s / ≤7 s.
- Burst‑Loss (Mean Burst Length)
Ambang: ≤3 / ≤2 / ≤1 paket hilang beruntun.

### Mengaitkan KPI dengan Dataset yang Ada
- PER/Loss % → langsung untuk PDR_e2e, PER_hop, dan burst‑loss.
- Latency per hop → agregasi ke latensi e2e dan p95 per hop.
- RSSI/SNR per hop → turunkan link‑margin p5 (butuh lookup sensitivitas LLCC68 per SF/BW/CR).
- Throughput → tampilkan goodput e2e sebagai indikator kapasitas.
- Timestamps → ukur on‑time rate, route discovery time, recovery time, dan SOS latency.
- Counter TX relay (jika ada) → hitung redundancy factor.
Rekomendasi dashboard: tampilkan p50 & p95, bukan hanya rata‑rata; gunakan kode warna hijau/kuning/merah dengan ambang sesuai daftar di atas. Untuk tren, gunakan window 1–5 menit agar adaptif terhadap perubahan kondisi.
### Paket KPI Minimum vs Lanjutan
- Minimum (Go/No‑Go lapangan): PDR_e2e, p95 latensi SOS, link‑margin p5, cakupan efektif, daya tahan Navigator.
- Lanjutan (optimasi mesh): redundancy factor, route‑discovery time, stabilitas rute, burst‑loss, energi per paket.

### 3b) Tabel KPI (Siap Slide)
Legenda: ↑ lebih besar lebih baik · ↓ lebih kecil lebih baik.
#### Tabel A — KPI Inti (Network‑Level)
Catatan: “cap” = estimasi kapasitas bersih profil PHY (SF/BW/CR) yang digunakan.
#### Tabel B — KPI Per‑Hop, Operasional & Skala
Untuk implementasi dashboard: tampilkan p50/p95 & gunakan warna hijau/kuning/merah sesuai ambang, serta window 1–5 menit untuk smoothing.

## 4) Solusi Teknis V2 (Ringkasan)
- LoRa Mesh Berbasis Peran:
- Navigator (dibawa pengunjung) – mengirim lokasi berkala, navigasi balik (trace‑back), tombol SOS, dan dapat bertindak sebagai forwarder oportunistik.
- Relay (titik strategis) – meneruskan pesan menuju gateway atau antar‑relay.
- Gateway (basecamp) – konsolidasi data, alarm SOS, analitik link & replay di webapp.
- Routing: AODV sederhana yang disesuaikan untuk LoRa, memanfaatkan controlled flooding terarah. Rencana kombinasi Sequence Number Controlled Flooding (SNCF), Reverse Path Forwarding (RPF), dan Level‑Based Flooding (LBF); subset paling aplikatif akan dipilih setelah uji.
- Antena & Penempatan:
- Navigator: omni PCB pigtail 5 dBi di sisi casing.
- Relay: flower‑pot ~6,1 dBi pada tiang 3 m dengan tiga kawat penahan.
- Gateway: yagi beam 90° diarahkan ke relay terdekat (target hop awal ~1 km).
- Konektivitas Gateway: Unit penerima LoRa → UART → RS485 → RJ45 → Ethernet Cat5e ke ruang basecamp; power berbagi pada kabel yang sama dengan penurun (buck) di ujung.
- Aplikasi Web (Raspberry Pi host): Peta live, alarm SOS, jejak rute, analitik mesh & link, serta penyimpanan pelacakan kasus.

## 5) Arsitektur Sistem (Gambaran Umum)
- Layer Perangkat: Navigator ↔ Relay ↔ Gateway (multi‑hop).
- Layer Komunikasi Radio: LoRa Sub‑GHz, parameter adaptif (SF/BW/CR) sesuai profil.
- Layer Routing: AODV adaptasi + controlled flooding (SNCF/RPF/LBF – selektif).
- Layer Transport Lokal: UART/RS485 untuk jarak dari tiang antena ke ruang base.
- Layer Aplikasi: Webapp di Raspberry Pi, akses via smartphone lokal (hotspot/SSID khusus).
Alur Operasi Utama:
- Navigator mengirim beacon lokasi berkala + status (baterai, SOS, dsb.).
- Relay meneruskan paket (aturan role‑based/heuristik kualitas link).
- Gateway mengagregasi, menampilkan peta & metrik, dan membunyikan buzzer saat SOS.
- Untuk navigasi balik, Navigator menampilkan arah ke basecamp/rute balik (trace‑back).

## 6) Desain Perangkat
### 6.1 Navigator (dibawa pengunjung)
Target dimensi: 10 × 8 × 2 cm.
Komponen kunci:
- MCU: ESP32‑S3.
- Radio: LoRa Ebyte E220‑400T22D (LLCC68).
- GNSS: u‑blox M10.
- Display: TFT 3,5″ touchscreen.
- Waktu: RTC DS3231.
- Penyimpanan: microSD card.
- Daya: IP5306/FM5324GA (pengisian & pengosongan) + soft‑latch power button.
- Sensor: MPU9250 (IMU), BMP280 (baro), SHT30‑D (suhu/kelembaban).
- I/O: Buzzer aktif, USB‑C.
- Upload Firmware: via jig khusus (pad terpadu di PCB).
- Antena: omni PCB pigtail 5 dBi (eksternal, sisi casing).
Fitur perangkat lunak:
- Beacon lokasi periodik ke gateway.
- Trace‑back rute pulang; tampilan kompas/arah dinamis ke basecamp.
- SOS (tombol/aksi layar) → prioritas jaringan + buzzer di gateway.
- Log perjalanan ke microSD.
- Manajemen daya: mode sleep terkontrol (tombol pendek = sleep/wake; tekan lama = dialog shutdown). Saat sleep, fungsi inti (perekaman minimal/periodik & telemetri) tetap aktif sesuai profil hemat energi.
- Target daya tahan: ≥ 24 jam (baterai 3600 mAh, profil campuran).
Ketahanan:
- PCB conformal coating, desain casing minim lubang, holster untuk sabuk/tas.
### 6.2 Relay (titik penguat jangkauan)
Fungsi: Meneruskan paket dari Navigator → Gateway (langsung atau via relay lain).
Basis HW: Mirip Navigator tanpa layar/sensor/charger user‑grade.
- Energi: Panel surya 6 V 2 A (±90×350×15 mm) → mini MPPT keluaran 5 V → LDO di PCB.
- Baterai: ~4× kapasitas Navigator.
- Antena: flower‑pot high‑gain (~6,1 dBi), tiang 3 m + 3 titik sling baja penahan.
- Sinkron waktu: u‑blox Neo‑6M untuk RTC; dapat mengirim paket validasi link berkala (untuk analitik di gateway).
### 6.3 Gateway (basecamp)
- Antena: yagi beam 90° (pada tiang 7–8 m), diarahkan ke relay terdekat.
- Konversi & Transport: LoRa UART → RS485 → RJ45 → Cat5e ke ruang base.
- Daya via kabel: Tegangan dinaikkan di tiang, diturunkan (buck) ke 5 V/3 V di ujung.
- Unit Dalam Ruang: Raspberry Pi, tombol power, buck converter, buzzer (tanpa layar).
- Webapp lokal: Akses via smartphone pada SSID/URL lokal.

## 7) Protokol & Data
- Routing: AODV adaptasi untuk LoRa, dengan aturan controlled flooding:
- SNCF (Selective Node Coded Flooding) – seleksi node forwarder.
- RPF (Reverse Path Forwarding) – hindari loop, prefer rute balik.
- LBF (Link‑Based Flooding) – berbasis kualitas link (RSSI/SNR/PER per paket).
- Catatan: Implementasi bertahap; subset teknik dipilih sesuai hasil uji.
- Prioritas SOS: Paket bertanda prioritas; retry & TTL khusus.
- Adaptasi radio: Profil SF/BW/CR per peran/hop; frame periodik vs on‑demand.
Contoh payload (ringkas):
{
  "type": "BEACON|SOS|ACK|LINKTEST",
  "role": "NAV|RELAY|GW",
  "node_id": "NAVxx",
  "hop": 2,
  "lat": -7.6,
  "lon": 110.4,
  "alt": 910,
  "speed": 0.8,
  "heading": 132,
  "batt": 3.85,
  "rssi": -103,
  "snr": 8.2,
  "ts_gps": 1737690000,
  "flags": {"sos": false}
}

### 7a) Spesifikasi Radio/PHY Tetap (Profil V2)
- Band: 433 MHz (Sub‑GHz)
- Spreading Factor (SF): 12 untuk semua perangkat (Navigator, Relay, Gateway)
- Bandwidth (BW): 125 kHz
- Coding Rate (CR): 4/5
- Daya Panc ar (TX): 22 dBm (pastikan sesuai regulasi lokal)
- CRC PHY: aktif (LoRa layer)
- Catatan operasional: SF12 + BW125 menghasilkan Time‑on‑Air (ToA) relatif besar, sehingga payload harus ramping dan interval kirim dikendalikan untuk menjaga kapasitas jaringan & kepatuhan duty‑cycle.
### 7b) Format Payload (String‑Based, dengan Prioritas SOS)
Prinsip: payload berupa string ASCII ringan, mudah di‑debug, namun tetap ringkas. Sediakan dua mode:
- Verbose (kunci=nilai, mudah dikembangkan)
- Compact (urutan tetap, hemat byte)
Bidang inti (disarankan):
- T (type): BEA (beacon), SOS, ACK, LKT (link‑test)
- P (priority): 1 untuk SOS, selain itu 0
- SEQ (sequence): counter paket untuk de‑duplikasi
- TTL: batas hop (0–15)
- SRC (node id) & ROLE: N/R/G
- H (hop): hop saat ini (diisi/ditambah oleh relay)
- TS (timestamp GPS/RTC)
- LAT,LON,ALT (opsional disingkat: LA,LO,AL)
- SPD,HDG (opsional)
- B (tegangan baterai) – khusus Navigator/Relay
- SIG (CRC aplikasi 16‑bit hex, opsional namun dianjurkan)
Contoh – Verbose (mudah dibaca):
V2|T=BEA|P=0|SEQ=12AB|TTL=6|SRC=N12|ROLE=N|H=0|TS=1737690000|LAT=-7.60123|LON=110.41568|ALT=910|SPD=0.8|HDG=132|B=3.85|SIG=8F3A
Contoh – SOS Verbose:
V2|T=SOS|P=1|SEQ=12AC|TTL=8|SRC=N12|H=0|TS=1737690123|LAT=-7.60150|LON=110.41610|B=3.50|MSG=HELP|SIG=92BD
Contoh – Compact (hemat byte, urutan tetap):
2|B|0|12AB|6|N12|N|0|1737690000|-7.60123|110.41568|910|0.8|132|3.85|8F3A
Aturan Ringkas:
- Delimiter utama |, key=value untuk mode Verbose; hindari karakter | dalam nilai.
- Seluruh string ASCII uppercase untuk kunci; nilai numerik desimal (lat/lon 5–6 digit pecahan) atau heksadesimal untuk SEQ/SIG.
- SIG: CRC16 (mis. CRC‑CCITT) 4 heks digit; LoRa PHY sudah punya CRC, tetapi CRC aplikasi membantu verifikasi & de‑duplikasi.
- Dedup: Gateway/Relay menyimpan recent SEQ per SRC (cache LRU) untuk membuang duplikat.
- Fail‑safe: Jika payload > batas, kirim Compact; untuk SOS, kirim berulang dengan jitter pendek hingga ACK.

## 8) Aplikasi Web (Gateway)
Fitur Utama:
- Peta live posisi Navigator; replay dan jejak rute.
- Alarm SOS (bunyi buzzer, notifikasi visual).
- Analitik jaringan mesh & link: PDR per hop, latensi, RSSI/SNR, link margin, snapshot tabel rute, heatmap jangkauan (dari paket link‑test Relay).
- Pelacakan kasus: Simpan perjalanan dan peristiwa SOS (untuk investigasi & pelatihan).
- Ringkasan TRESNO V2: Halaman naratif singkat untuk pengantar cepat.
Operasional: Di‑host pada Raspberry Pi; akses lokal via smartphone.

## 9) Daya, Energi & Keandalan
- Navigator: Profil hemat – duty cycle GNSS, mode layar adaptif, interval kirim dinamis; target ≥ 24 jam.
- Relay: Tenaga surya + baterai besar; pemantauan SOC sederhana; watchdog untuk auto‑recovery.
- Perlindungan: Conformal coating, gasket casing, ventilasi terkontrol (anti‑kondensasi), konektor tertutup.

## 10) Keamanan & Keselamatan
- Keselamatan pengguna: Navigasi balik jelas; instruksi sederhana di layar; fallback kompas.
- Protokol SOS: Jalur prioritas, alarm gateway, logging.
- Integritas data: CRC, nomor urut (sequence), TTL/anti‑loop.
- Privasi: ID anonim; data hanya lokal; purge otomatis untuk data kasus lama.

## 10a) SOP Pemasangan (Lapangan)
Tujuan: Menjamin pemasangan aman, mendapatkan link yang baik, dan suplai energi memadai.
Pemilihan lokasi Relay:
- Aman & stabil: Tanah tidak terlalu lunak; hindari area rawan longsor/banjir.
- Cahaya cukup: Lokasi cukup terbuka agar panel surya mendapat sinar matahari memadai sepanjang hari.
- Link bagus: Verifikasi RSSI/SNR/link‑margin ke tetangga dengan uji link‑test sebelum pemasangan final.
- Akses perawatan: Dapat dijangkau untuk inspeksi berkala.
Pemasangan mekanik & antena (Relay):
- Tiang ±3 m dengan tiga kawat penahan (120°), tegangkan secukupnya; lakukan pengeboran tanah secara hati‑hati.
- Antena flower‑pot vertikal; gunakan drip loop pada kabel RF, seal konektor (self‑amalgamating tape + heat‑shrink).
- Panel surya menghadap khatulistiwa, kemiringan ±10–15°; kabel diberi strain‑relief.
- Label arah, ID node, dan tanggal pemasangan pada enclosure.
Pemasangan antena Gateway:
- Lokasi dekat basecamp, fokus ke kualitas link ke relay pertama; paparan matahari tidak kritis.
- Walau material non‑konduktif dan di sekitar ada pohon/bambu lebih tinggi, risiko induksi petir tetap ada. Disarankan proteksi lonjakan minimal: TVS/gas‑discharge di jalur RF/data, grounding titik tunggal bila memungkinkan, dan loop tetesan pada kabel masuk.
- Jalur RS485/RJ45: gunakan pasangan terpilin, rute kabel aman dari gesekan & genangan; beri strain‑relief.
## 10b) SOP Operasi dan Maintenance
Pra‑operasi (sebelum turun lapangan):
- Periksa versi firmware & konfigurasi (SF12/BW125/CR4/5/TX 22 dBm).
- Sinkronisasi waktu (RTC DS3231) di Gateway/Relay; Navigator akan discipline dari GNSS.
- Uji beacon dan SOS end‑to‑end (cek buzzer gateway & notifikasi webapp).
- Cek baterai: Navigator ≥ 90%; Relay panel & baterai berfungsi; uptime gateway normal.
Operasi harian:
- Pantau dashboard KPI (PDR_e2e, SOS latency p95, link‑margin p5, on‑time rate).
- Jika PDR_e2e < 85% atau link‑margin p5 < +3 dB pada rute utama, rencanakan reposisi/penambahan relay.
- Validasi beacon on‑time rate ≥ 92%; bila turun, cek suplai energi dan posisi antena.
- Catat anomali (burst‑loss > 2 paket, route‑change > 6/jam) untuk tindakan korektif.
Uji berkala:
- Mingguan: Uji SOS terjadwal; inspeksi visual konektor & guy‑wire.
- Bulanan: Review log energi, bersihkan debu/kotoran, tes redundancy factor dan route discovery time.
Penanganan insiden:
- Bila terjadi SOS, prioritas ke lokalisasi cepat di webapp; dorong Navigator terdekat sebagai relay mobile bila perlu.
- Setelah insiden, lakukan post‑mortem singkat berbasis KPI (latensi, PDR, burst‑loss) dan susun perbaikan.
Trigger perawatan (berbasis KPI):
- PDR_e2e < 90% berulang → tambah/geser relay, cek antena/konektor.
- On‑time < 85% → evaluasi interval beacon & suplai energi.
- Link‑margin p5 < +3 dB → naikkan ketinggian antena/ubah orientasi/kurangi halangan.
- Uptime < 99% → cek catu daya & watchdog.

## 11) Rencana Deploy & Site Planning
- Survei lokasi: LOS, kepadatan pohon, titik ketinggian, sudut elevasi.
- Penempatan relay: Jarak antar‑relay dengan margin RSSI/SNR; tiang 3 m + 3 kawat penahan.
- Arah antena yagi: Ke relay pertama/hub.
- Jalur kabel RS485/RJ45: Proteksi petir dasar, strain‑relief.
- Komisioning: Uji link‑test, kalibrasi waktu, uji SOS end‑to‑end.

## 12) Uji Lapangan & Validasi
- Skenario: LOS vs non‑LOS, cuaca lembap/berdebu, siang vs malam.
- Metrik: PDR, latensi, jarak/hop, RSSI/SNR, konsumsi daya, waktu tanggap SOS.
- Alat bantu: Logger di gateway; replay peta; analisis CSV (kompatibel LoRa Analyzer).
- Kriteria lulus: KPI pada Bagian 3 tercapai dan terukur.

## 13) Risiko & Mitigasi
- Jangkauan kurang: Tambah relay; optimasi SF/BW/CR; evaluasi antena & posisi.
- Lingkungan lembap/berdebu: Conformal coating; casing minim bukaan; desikan opsional.
- Daya tidak cukup: Profil hemat agresif; panel surya lebih besar; baterai cadangan.
- Kompleksitas routing: Mulai dari AODV minimal; tambahkan fitur flooding selektif bertahap.

## 14) Roadmap (Jul–Oct 2025) — Weekly Blocks

## 15) Tim & Peran (Saran)) Tim & Peran (Saran)
- System Lead / PM
- RF & Antenna Engineer
- Embedded/Firmware Engineer (ESP32, LoRa)
- Hardware & Power Engineer (skematik, layout, PD, MPPT)
- Mechanical & Enclosure (casing, holster, sealing)
- Backend/Web Engineer (Raspberry Pi, webapp, visualisasi)
- Field Ops (survei, pemasangan, prosedur uji)
- QA/Safety (protokol SOS, SOP lapangan)

## 16) Anggaran (Kategori Utama)
- Perangkat keras: PCB, komponen, antena, baterai, panel surya, tiang & kawat penahan, konektor RS485/RJ45.
- Perangkat lunak: Pengembangan firmware/web, alat uji/analitik.
- Mekanikal: Casing, holster, gasket, pelapis.
- Operasional: Transport, pemasangan, perizinan lokasi (bila perlu).
- Cadangan risiko: 10–20%.

## 17) Visual yang Direkomendasikan (untuk Presentasi)
- Diagram arsitektur sistem (mesh berbasis peran: Navigator–Relay–Gateway, RS485 ke base).
- Peta topologi dengan posisi relay dan rute cakupan.
- Blok diagram perangkat (Navigator/Relay/Gateway) + daftar komponen utama.
- Urutan SOS (sequence diagram) dari tombol hingga alarm gateway.
- State machine daya Navigator (normal → sleep → wake → dialog shutdown).
- Mockup UI webapp (peta, kartu KPI, notifikasi SOS, replay rute).
- Skema tiang relay (ketinggian, antena flower‑pot, 3 kawat penahan).
- Grafik metrik uji (PDR vs jarak/hop, latensi, RSSI/SNR).

## 18) Lampiran
### 18.1 Parameter Radio (Profil Tetap V2)
- Band: 433 MHz (LLCC68/E220‑400T22D)
- TX Power: 22 dBm (patuh regulasi setempat)
- SF/BW/CR: SF12 / 125 kHz / 4/5 untuk seluruh peran
- CRC PHY: aktif; disarankan CRC aplikasi (SIG) untuk dedup & validasi
- Catatan: ToA besar di SF12—jaga interval beacon & ukuran payload agar kapasitas mesh tetap sehat.
### 18.2 Glosarium Singkat
- AODV: Ad‑hoc On‑Demand Distance Vector (routing reaktif).
- SNCF/RPF/LBF: Teknik kontrol flooding & forwarder selektif.
- PDR: Packet Delivery Ratio.
- TTL: Time To Live paket.

## 19) Narasi Singkat (Elevator Pitch)
“TRESNO adalah solusi navigasi dua arah untuk kawasan Merapi yang minim infrastruktur. Dengan LoRa mesh berbasis peran, pengunjung selalu tahu arah pulang, sementara relawan melihat posisi mereka secara real‑time. Tanpa internet, hemat daya, dan tahan lingkungan—TRESNO membantu mempercepat pencarian dan menyelamatkan nyawa.”

### Catatan Penutup
Dokumen ini adalah draft terstruktur untuk presentasi dan pegangan teknis awal. Bagian routing (SNCF/RPF/LBF) akan difinalkan setelah uji terbatas; KPI dapat disesuaikan mengikuti hasil lapangan dan batas regulasi radio setempat.

## Lampiran – Sticker Card Lapangan (1 Halaman)
Tujuan: Lembar ringkas yang bisa ditempel pada enclosure atau dibawa tim.
### 1) Radio Profile (Tetap V2)
- 433 MHz, SF12 / BW 125 kHz / CR 4/5, TX 22 dBm (patuh regulasi).
- ToA besar di SF12 → payload ringkas & interval kirim terkendali.
- PHY CRC aktif; SIG (CRC app) di payload untuk dedup & verifikasi.
### 2) Payload Cepat (String ASCII)
- Mode Verbose: key=value dipisah |.
- Mode Compact: urutan tetap, dipisah | (hemat byte).
- Prioritas SOS: P=1 (retry+jitter), lainnya P=0.
- Bidang umum: T(jenis), P, SEQ, TTL, SRC, ROLE, H, TS, LAT/LO/AL, B, SIG.
- Link‑test (opsional): tambahkan per‑hop: PH=hop:rssi:snr;… dan delay per hop: DT=hop:detik;… (diisi relay saat forward).
### 3) SOP Singkat
Pemasangan Relay: tanah tidak terlalu lunak, lokasi cukup terbuka (matahari), link bagus (uji cepat RSSI/SNR), akses perawatan. Tiang 3 m + 3 kawat penahan; bor tanah hati‑hati; seal konektor; panel surya miring 10–15°.
Antena Gateway: dekat basecamp, fokus ke link ke relay pertama. Pertimbangkan proteksi lonjakan minimal (TVS/GDT), strain‑relief, drip loop kabel. RS485/RJ45 pakai pasangan terpilin.
Operasi: pra‑operasi (cek firmware/RTC, uji BEACON & SOS), pantau dashboard KPI; jika PDR_e2e<85% atau LM p5<+3 dB → evaluasi posisi/antenna/relay. Uji SOS mingguan; bersih‑bersih bulanan.
### 4) KPI Cepat (Ambang Target)
- PDR_e2e ≥ 90% · SOS p95 ≤ 30 s · Link‑margin p5 ≥ +6 dB
- On‑time ≥ 92% · Latensi/hop p95 ≤ 1 s · Uptime ≥ 99.5%

## Contoh Payload (Siap Pakai)
### A) Minimum (BEA – Compact, untuk operasi harian)
Mendukung KPI tingkat jaringan: PDR_e2e, latensi E2E, on‑time, cakupan, energi/baterai. (KPI per‑hop diperoleh dari frame link‑test berkala.)
2|B|0|01AF|6|NAV12|N|0|1737690000|-7.60123|110.41568|910|0.8|132|3.85|8F3A
Urutan bidang (Compact): VER|TYPE|P|SEQ|TTL|SRC|ROLE|H|TS|LAT|LON|ALT|SPD|HDG|B|SIG
- VER = 2 (versi payload)
- TYPE = B (Beacon), P = 0 (normal)
- SEQ = 01AF (hex), TTL = 6
- SRC = NAV12, ROLE = N, H = 0 (diisi relay saat forward)
- TS = 1737690000 (epoch s), koordinat & status sesuai contoh
- SIG = CRC16 app (heks 4 digit)
### B) Full (LKT – Verbose + Per‑Hop, untuk analitik KPI lengkap)
Mendukung KPI per‑hop (RSSI/SNR/latensi hop), stabilitas rute, dan efisiensi flooding.
V2|T=LKT|P=0|SEQ=01B0|TTL=8|SRC=N12|ROLE=N|H=0|TS=1737690456|LAT=-7.60180|LON=110.41650|ALT=915|B=3.82|PH=1:-104:7.8;2:-109:6.1;3:-111:5.0|DT=1:1.2;2:0.9;3:1.0|SIG=AB12
Keterangan:
- T=LKT (link‑test); PH berisi pasangan hop:rssi:snr yang ditambahkan setiap relay saat meneruskan.
- DT berisi hop:delay_s (estimasi waktu simpan‑terus per hop); opsional jika tersedia.
- Gateway dapat menurunkan link‑margin per hop dari RSSI + tabel sensitivitas (SF12/BW125/CR4/5) dan menghitung latensi per hop dari DT.
Disarankan mengirim LKT secara berkala (mis. setiap 2–5 menit per node) agar dashboard KPI per‑hop tetap mutakhir tanpa membebani jaringan.
