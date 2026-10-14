# AGENTS.md — lumbung-data-pipeline

Pipeline data Lumbung: ingest timbangan, trip, dan 3 harga referensi secara idempoten ke Lumbung, serta penerbit event ke Titeny.

## Sumber kebenaran

Requirement hidup di [Lumbung-TownHall](https://github.com/Coding-Skuy/Lumbung-TownHall), bukan di repo ini. Dilarang menduplikasi requirement di repo kode; requirement berubah = PR di Lumbung-TownHall dulu (versi beku `v1.0.0` tidak diedit; perubahan masuk v1.1.0/v2.0.0).

| Dokumen | Isi yang berlaku untuk repo ini |
|---|---|
| [`platform/versions/v1.0.0/FSD/30-kontrak.md`](https://github.com/Coding-Skuy/Lumbung-TownHall/blob/main/platform/versions/v1.0.0/FSD/30-kontrak.md) | Event domain Lumbung dan idempotensi UUID |
| [`platform/versions/v1.0.0/FSD/20-model-data.md`](https://github.com/Coding-Skuy/Lumbung-TownHall/blob/main/platform/versions/v1.0.0/FSD/20-model-data.md) | Entitas timbangan/trip/fee |
| [`produk/50-autentikasi-perangkat.md`](https://github.com/Coding-Skuy/Lumbung-TownHall/blob/main/produk/50-autentikasi-perangkat.md) | Kunci server-ke-server |
| [`keuangan/20-papan-harga-mingguan.md`](https://github.com/Coding-Skuy/Lumbung-TownHall/blob/main/keuangan/20-papan-harga-mingguan.md) | 3 harga referensi tiap Jumat 15.00 |
| [`platform/versions/v1.0.0/BRD/30-keuangan-fee.md`](https://github.com/Coding-Skuy/Lumbung-TownHall/blob/main/platform/versions/v1.0.0/BRD/30-keuangan-fee.md) | BR-305 lembar kerja 3 referensi disimpan 6 bulan |
| [`platform/versions/v1.0.0/SRS/30-non-functional.md`](https://github.com/Coding-Skuy/Lumbung-TownHall/blob/main/platform/versions/v1.0.0/SRS/30-non-functional.md) | Keamanan dan DB per service |
| [`docs/schedule.md`](./docs/schedule.md) | Jadwal pipeline di repo ini |
| [`KONTEKS-LINTAS-DIVISI.md`](https://github.com/Coding-Skuy/ChefGenie-TownHall/blob/main/KONTEKS-LINTAS-DIVISI.md) (ChefGenie-TownHall) | Event registry §3, komoditas pilot §4, grade §5, satuan/ID/uang §6 |
| [`townhall-roadmap.md`](https://github.com/Coding-Skuy/ChefGenie-TownHall/blob/main/townhall-roadmap.md) (ChefGenie-TownHall) | Batas context §2, Definisi Selesai §6 |

Angka terkunci v1.0.0 yang berlaku di seluruh repo Lumbung:

- 3 komoditas pilot: cabai, bawang, kangkung (KONTEKS §4; dilarang mencampur angka 3 dan 12 komoditas tanpa label fase).
- Fee logistik murni: Dekat (<15 km) Rp1.500/kg, Jauh (15-25 km) Rp2.500/kg; radius layanan 25 km (BRD/30 BR-301).
- Uang integer rupiah, berat integer gram; pembayaran petani H+1 dari berat bersih per grade x harga papan.
- Papan harga dihitung Jumat 15.00 (median 3 referensi - 10%, bulatkan ke bawah kelipatan Rp500), publikasi maksimal 17.00.
- Titik impas 18.000 kg/bulan (600 kg/hari); target 2 ton/hari; susut total <5%.

## Tech stack terkunci

| Lapisan | Pilihan terkunci |
|---|---|
| Prinsip | Idempoten (kunci alami), auditabel (sumber + waktu ingest), terpisah (tidak menulis ke DB divisi lain) |
| Jalur tulis | Lewat API `lumbung-backend-service` dengan `X-Service-Key` (hak minimal: tulis pipeline + baca agregat) |
| Event keluar | Registry Titeny (11 `event_type` terkunci), amplop idempoten, `POST /api/v1/event/masuk` + `X-Idempotency-Key`, retry 3x (5/30/300 dtk) lalu DLQ |
| Bahasa implementasi | Diputuskan lewat ADR-0002 sebelum kode pertama LMB-DATA-002 (v1.0.0 belum mengunci bahasa pipeline; selama itu repo hanya berisi dokumen dan kontrak) |

Larangan: menulis ke DB divisi lain, `event_type` di luar registry, menyimpan service key di repo.

## Aturan kerja AI

1. **Baca dulu.** Baca `AGENTS.md` ini dan seluruh ADR di `docs/decisions/` sebelum menulis kode.
2. **TDD merah-hijau.** Tulis tes gagal dari acceptance criteria tiket, lalu implementasi. Rilis diblokir bila QA gate gagal (SRS `40-acceptance-criteria.md`: QA-001..QA-004; tidak ada tes yang di-skip).
3. **Kontrak berversi.** Kontrak API/event hanya diubah lewat bump versi + ADR baru. Perubahan requirement dimulai dari PR di Lumbung-TownHall, bukan dari repo ini.
4. **Commit.** Awalan `docs:` / `feat:` / `fix:`; setiap PR wajib memperbarui `CHANGELOG.md` repo ini.
5. **ADR immutable.** Keputusan arsitektur baru = ADR bernomor berikutnya di `docs/decisions/`; ADR lama tidak diedit (diganti dengan ADR baru berstatus "Menggantikan ADR-NNNN"), sama seperti aturan versi beku TownHall.
6. **DB per service.** Dilarang menyentuh basis data divisi lain. Basis data logis hanya `lumbung`; Titeny hanya membaca (read-only) ke DB sumber lewat kontrak event, bukan lewat akses langsung dari repo ini.
7. **Angka dari dokumen.** Angka bisnis (tarif, ambang, batas) hanya dari dokumen TownHall; jangan mengarang atau menghitung ulang di tempat yang dilarang.
8. **Idempoten.** Ingest ulang dengan kunci alami sama tidak menggandakan baris; event memakai `event_id` `evt_YYYYMMDD_NNNNNN` (duplikat diabaikan konsumen).
9. **Konsumen tercatat.** Setiap event keluar punya konsumen terdokumentasi di dokumen kontrak event repo ini.

## Definisi selesai

Dari [townhall-roadmap.md §6](https://github.com/Coding-Skuy/ChefGenie-TownHall/blob/main/townhall-roadmap.md), berlaku per milestone:

- [ ] Dokumen versi lengkap (BRD/FRD/FSD/PRD/SRS) + `SNAPSHOT-ROADMAP.md` di Lumbung-TownHall.
- [ ] QA gate berbasis SRS lulus (TDD merah-hijau); rilis diblokir bila gagal.
- [ ] Kontrak API terbit; audit keamanan bersih (JWT audiens `lumbung` + DB per service).
- [ ] Bukti metrik lintas-divisi (KONTEKS §7) tercatat.
- [ ] CHANGELOG versi diperbarui; dokumen beku tidak diedit.
