> Versi: v1.0.0 | Status: disetujui | Menggantikan: -
> Sumber: Lumbung-TownHall `platform/versions/v1.0.0/` pada commit `673a467` ([platform/versions/v1.0.0](https://github.com/Coding-Skuy/Lumbung-TownHall/blob/673a467126e195ffd67c9d3ff21d108193a6c3ec/platform/versions/v1.0.0))

# 40 — Tech Stack lumbung-data-pipeline

Matriks pipeline v1.0.0.

## Pipeline

| Pipeline | Sumber | Tujuan | Jadwal/Batas | Tiket |
|---|---|---|---|---|
| Timbangan + trip | Backend (data gudang) | DB `lumbung` via API (service key) | idempoten per kunci alami | LMB-DATA-002 |
| Harga referensi | 3 referensi (Pasar Induk, pasar kota, eceran daring) | Papan harga | Jumat 15.00; publikasi maks 17.00 | LMB-DATA-003 |
| Event keluar | Domain Lumbung | Titeny `POST /api/v1/event/masuk` | retry 5/30/300 dtk lalu DLQ | LMB-DATA-004 |

## Kontrak event (registry Titeny)

`source: lumbung`; event v1.0.0: `panen.dicatat`, `harga.diperbarui`, `stok.berubah`, `stok.snapshot_harian`. Amplop: `event_id` (`evt_YYYYMMDD_NNNNNN`), `schema_version`, `source`, `occurred_at`/`received_at`, `batch_id`. Header `X-Idempotency-Key`.
