# Jadwal Pipeline Lumbung

Jadwal awal (WIB). Sesuaikan dengan jam operasional gudang.

| Pipeline | Jadwal | Keterangan |
|---|---|---|
| `ingest-timbangan` | Setiap 15 menit, 06.00–20.00 | Mengejar antrean timbang gudang |
| `ingest-trip` | Setiap 1 jam | Rekap trip kebun ke gudang |
| `ingest-pasar-induk` | Harian 05.00 | Harga pembuka pasar induk |

## Catatan Operasional

- Kegagalan satu batch tidak boleh menghentikan batch berikutnya; tulis ke log dan lanjutkan.
- Backfill dilakukan per rentang tanggal eksplisit, bukan dengan mengulang seluruh tabel.
- Monitoring: setiap jalan mencatat jumlah baris masuk, duplikat, dan gagal ke log terstruktur.
