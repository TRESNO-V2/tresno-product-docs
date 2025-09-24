# TRESNO V2 — Dokumen Produk & Rancangan Teknis (Versi Terukur)

*Tracking Based on Smart GPS Technology untuk kawasan rawan bencana Merapi*

## Ringkasan Eksekutif
TRESNO adalah sistem navigasi & pelacakan dua arah berbasis **LoRa + GNSS** untuk membantu pengawasan dan penyelamatan di kaki Gunung Merapi. V2 mengatasi keterbatasan V1 lewat arsitektur **LoRa mesh berbasis peran**—**Navigator** (pengunjung), **Relay** (penguat jangkauan), dan **Gateway** (basecamp). Sistem ini memungkinkan **arah pulang ke basecamp** bagi pengunjung dan **pemantauan posisi** oleh relawan—**tanpa internet**.

## Sasaran Terukur (KPI Ringkas)
- **PDR_e2e ≥ 90%**, **SOS p95 ≤ 30 s**, **cakupan efektif ≥ 3 km** (multi-hop).
- **Daya tahan Navigator ≥ 24 jam**, **link-margin p5 ≥ +6 dB**, **on-time ≥ 92%**.

## Arsitektur (Gambaran)
- **Navigator** → kirim lokasi periodik, **SOS**, trace-back ke basecamp.
- **Relay** → meneruskan paket (multi-hop) pada titik strategis.
- **Gateway** → konsolidasi data, buzzer SOS, peta live & analitik link.

## Radio/PHY
433 MHz, **SF12 / BW 125 kHz / CR 4/5**, TX 22 dBm, CRC PHY aktif. ToA besar → payload ringkas, interval kirim terkendali.

## Protokol & Payload (ringkas)
Jenis: `BEA` (beacon), `SOS`, `ACK`, `LKT` (link-test). Format string ASCII **Verbose** (key=value) atau **Compact** (urutan tetap) dengan **SIG = CRC16 CCITT-FALSE**. Lihat dokumen **Spec Freeze** untuk detail & test vector.

## Aplikasi Web (Gateway)
Peta live, alarm SOS, replay rute, analitik KPI (PDR/latensi/link-margin), case tracking.

## Uji & Validasi (ringkas)
**P2P (1 hop)** untuk jarak efektif; **Mesh (2–3 hop)** untuk cakupan ≥ 3 km dan efisiensi flooding.

## Roadmap (Jul–Okt 2025)
Finalisasi spesifikasi & payload → prototipe Navigator/Relay/Gateway → uji lapangan #1 → controlled flooding subset → uji #2 dan freeze SOP.
