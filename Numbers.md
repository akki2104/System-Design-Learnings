# The Numbers Reference Card

The go-to card for back-of-the-envelope estimation.
Populated fully during Topic 003. Referenced in every case study.

---

## Powers of Ten — Data Sizes

| Unit | Bytes | Real-world feel |
|------|-------|----------------|
| 1 KB | 10³ | A short text paragraph |
| 1 MB | 10⁶ | A JPEG photo (compressed) |
| 1 GB | 10⁹ | A ~10-minute HD video clip |
| 1 TB | 10¹² | ~1000 HD movies |
| 1 PB | 10¹⁵ | Facebook stores ~100s of PB |

---

## Latency Numbers Every Engineer Should Know

| Operation | Latency (approx) | Notes |
|-----------|-----------------|-------|
| L1 cache reference | ~0.5 ns | |
| L2 cache reference | ~7 ns | 14× L1 |
| RAM reference | ~100 ns | |
| Read 1 MB from RAM | ~250 µs | |
| SSD random read | ~100 µs | |
| Read 1 MB from SSD | ~1 ms | |
| Disk seek (HDD) | ~10 ms | |
| Datacenter round trip (same region) | ~0.5 ms | |
| Cross-region round trip (US→EU) | ~150 ms | |
| Packet US→Australia | ~150 ms | |

**Rule of thumb:** RAM is 1000× faster than disk. Network within datacenter ≈ 0.5 ms. Cross-continental ≈ 100–150 ms.

---

## QPS Estimation Formula

```
QPS = (DAU × average_requests_per_user_per_day) / 86,400
Peak QPS ≈ 2× to 10× average QPS
```

---

## Storage Estimation Formula

```
Storage/day = writes_per_day × avg_object_size × replication_factor
Storage/5yr = Storage/day × 365 × 5
```

---

## Bandwidth Estimation Formula

```
Bandwidth = QPS × avg_payload_size_bytes
```

---

## Cache Sizing Rule of Thumb

```
80/20 rule: 20% of data generates 80% of reads
Cache size target ≈ 20% of total working dataset
```

---

## Availability Table

| Nines | Downtime per year | Downtime per month |
|-------|------------------|-------------------|
| 99% (2 nines) | ~3.65 days | ~7.2 hours |
| 99.9% (3 nines) | ~8.76 hours | ~43.2 min |
| 99.99% (4 nines) | ~52.6 min | ~4.3 min |
| 99.999% (5 nines) | ~5.26 min | ~26 sec |

---

## Server Capacity Rules of Thumb (directional only)

| Workload | Rough QPS per commodity server |
|----------|-------------------------------|
| Static content (nginx) | ~10,000–50,000 |
| Simple API (DB-backed) | ~1,000–5,000 |
| CPU-heavy computation | ~100–500 |

> These are order-of-magnitude guides for interviews — always state your assumptions.

---

## Common Object Sizes (approximate)

| Object | Size |
|--------|------|
| Tweet / short text | ~280 bytes |
| User metadata row | ~1 KB |
| Profile photo (thumb) | ~200 KB |
| Photo (full, compressed) | ~1–3 MB |
| 1-minute audio | ~1 MB |
| 1-minute HD video | ~50–100 MB |
| Chat message | ~100 bytes |
