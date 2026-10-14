> Versi: v1.0.0 | Status: disetujui | Menggantikan: -
> Sumber: Lumbung-TownHall `platform/versions/v1.0.0/` pada commit `673a467`; [townhall-roadmap.md §2](https://github.com/Coding-Skuy/ChefGenie-TownHall/blob/main/townhall-roadmap.md); [KONTEKS §8/§9](https://github.com/Coding-Skuy/ChefGenie-TownHall/blob/main/KONTEKS-LINTAS-DIVISI.md)

# 60 — Blast Radius lumbung-data-pipeline

Batas context v1.0.0 Lumbung (townhall-roadmap §2): 3 komoditas pilot, fee Rp1.500/kg dekat dan Rp2.500/kg jauh, radius 25 km; point of concern titik impas 18.000 kg/bulan dan susut <5%.

## Boleh disentuh

- Menarik timbangan, trip, dan harga referensi lalu menyerahkannya ke backend lewat API.
- Membangun amplop dan mengirim event registry dengan `source: lumbung`.
- Jadwal dan DLQ pipeline.

## Dilarang

- Menyentuh DB logis divisi lain (`pasaree`, `titipo`, `pedaree`, `pawonee`, `titeny`).
- Menambah `event_type` di luar registry KONTEKS §3 tanpa bump versi + 30 hari dual-read.
- Menambah komoditas keempat atau melayani di luar radius 25 km (non-goal PRD/30).
- Menulis langsung ke DB (termasuk `lumbung`) di luar API backend.
- Membaca atau menulis data pengguna (service key tidak punya akses auth pengguna).
- Mengubah nama atau skema event tanpa bump versi.

## Kontrak diproduksi

- Event ke Titeny (amplop idempoten, `source: lumbung`).
- Dataset bersih untuk `lumbung-ai-models` dan harga referensi untuk papan.

## Kontrak dikonsumsi

- API backend Lumbung (`X-Service-Key`).
- Tiga harga pasar referensi yang dimasukkan Admin Gudang tiap Jumat.

## Aturan perubahan

Perubahan di luar batas ini = PR requirement di Lumbung-TownHall lebih dulu, lalu ADR baru di repo ini. Perubahan angka lintas-divisi wajib memperbarui KONTEKS §terkait + dokumen divisi + CHANGELOG versi dalam PR yang sama (KONTEKS §9).
