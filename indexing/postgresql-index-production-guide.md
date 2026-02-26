# PostgreSQL Index Guide (Production Ready)

> Tested on PostgreSQL 14+

---

## Index Type Comparison

| Index Type | Cocok Untuk | Contoh Create | Kelebihan | Kekurangan | Use Case Ideal || --- | --- | --- | --- | --- | --- |
| B-Tree (default) | =, <, >, BETWEEN, ORDER BY, LIKE 'abc%' | CREATE INDEX idx_email ON users(email); | Stabil, umum, support sorting | Tidak optimal untuk JSON/array | OLTP umum |
| Hash | Equality (=) | CREATE INDEX idx_id_hash ON users USING HASH(id); | Simple equality lookup | Jarang lebih cepat dari B-Tree | Rare use case |
| GIN | JSONB, Array, Full Text | CREATE INDEX idx_data ON orders USING GIN(data); | Sangat cepat untuk JSON containment | Index besar, write lambat | JSON-heavy workload |
| GiST | Range, spatial, overlap | CREATE INDEX idx_period ON bookings USING GIST(period); | Cocok untuk range & geo | Lebih besar dari B-Tree | Booking, time range |
| BRIN | Tabel besar & data terurut | CREATE INDEX idx_created ON logs USING BRIN(created_at); | Sangat kecil & cepat dibuat | Kurang presisi | Log, time-series |
| Composite | Multi-column filter | CREATE INDEX idx_user_status ON orders(user_id, status); | Optimal untuk 2+ kolom | Order kolom penting | Query kombinasi |
| Partial | Subset data | CREATE INDEX idx_paid ON orders(user_id) WHERE status = 'PAID'; | Size kecil & cepat | Hanya cocok kondisi spesifik | Status dominan |
| Unique | Cegah duplikasi | CREATE UNIQUE INDEX idx_email_u ON users(email); | Enforce business rule | Insert conflict handling | Validasi data |
| Expression | Function di WHERE | CREATE INDEX idx_lower_email ON users(LOWER(email)); | Case-insensitive search | Query harus identik | Search normalization |
---

# 1️⃣ B-Tree (Default)

### Cocok Untuk
- Equality
- Range
- Sorting
- Prefix search

```sql
CREATE INDEX idx_email ON users(email);
Production Notes

Default & paling aman

Tambah disk usage

Memperlambat INSERT/UPDATE

2️⃣ GIN (JSON / Array)
CREATE INDEX idx_data ON orders USING GIN(data);
Cocok Untuk
SELECT * FROM orders
WHERE data @> '{"status":"paid"}';
Production Warning

Index size bisa sangat besar

High write workload → pertimbangkan ulang

3️⃣ BRIN (Very Large Tables)
CREATE INDEX idx_created ON logs USING BRIN(created_at);
Ideal Jika

Data append-only

Kolom terurut (timestamp, id incremental)

Kelebihan

Sangat kecil (MB untuk miliaran row)

Cepat dibuat

4️⃣ Composite Index
CREATE INDEX idx_user_status
ON orders(user_id, status);
Left-Most Rule

Index (user_id, status) efektif untuk:

WHERE user_id = ?

WHERE user_id = ? AND status = ?

Tidak optimal untuk:

WHERE status = ? saja

5️⃣ Partial Index
CREATE INDEX idx_paid_orders
ON orders(user_id)
WHERE status = 'PAID';
Cocok Jika

Sebagian besar data tidak perlu di-index

Ada kondisi dominan

6️⃣ Expression Index
CREATE INDEX idx_lower_email
ON users(LOWER(email));

Query harus identik:

SELECT * FROM users
WHERE LOWER(email) = 'a@mail.com';
🔥 Operational Considerations
Cek Index Usage
SELECT
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan
FROM pg_stat_user_indexes;
Cek Index Size
SELECT
    relname,
    pg_size_pretty(pg_relation_size(indexrelid))
FROM pg_stat_user_indexes;
Reindex
REINDEX INDEX idx_name;
⚠️ Write Penalty Reminder

Setiap index tambahan berarti:

INSERT lebih lambat

UPDATE lebih lambat

DELETE lebih lambat

Vacuum lebih berat

📌 Golden Rule

Temukan slow query

Jalankan EXPLAIN ANALYZE

Buat index

Benchmark sebelum & sesudah

Hitung trade-off disk & write cost

Index yang tidak dipakai adalah beban, bukan optimasi.
