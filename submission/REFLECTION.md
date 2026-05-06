# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** Ninh Quang Trí
**Cohort:** A20-K1
**Ngày submit:** 2026-05-06

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

> Paste output của `python 00-setup/detect-hardware.py` vào đây, hoặc điền thủ công:

- **OS:** Windows 10 (MINGW64)
- **CPU:** Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
- **Cores:** 4 physical / 8 logical
- **CPU extensions:** AVX, AVX2, FMA, F16C
- **RAM:** 15.9 GB
- **Accelerator:** NVIDIA GeForce GTX 1050 (2048 MiB)
- **llama.cpp backend đã chọn:** CPU (AVX/NEON tuning)
- **Recommended model tier:** TinyLlama-1.1B (Q4_K_M)

**Setup story** (≤ 80 chữ): Để chạy lab trên MINGW, tôi đã sửa Makefile để hỗ trợ Windows, cài thêm `prometheus-client` và patch `app.py` của `llama_cpp.server` để có endpoint `/metrics`. Tôi cũng xử lý lỗi "Long Path" khi install bằng cách dùng thư mục `tmp_pip` ở root.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

> Paste bảng từ `benchmarks/01-quickstart-results.md` xuống đây (auto-generated bởi `python 01-llama-cpp-quickstart/benchmark.py`).

| Model    | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
| -------- | --------: | ----------------: | ----------------: | -------------------: | ------------------: |
| (Q4_K_M) |       992 |         191 / 247 |       40.3 / 70.3 |   2712 / 2791 / 2793 |                24.8 |
| (Q2_K)   |       204 |         324 / 414 |       37.6 / 52.8 |   2585 / 2929 / 3024 |                26.6 |

**Một quan sát** (≤ 50 chữ): Q2_K tải nhanh hơn hẳn (204ms vs 992ms) và có throughput nhỉnh hơn một chút (26.6 tok/s). Tuy nhiên, Q4_K_M vẫn đạt mức >20 tok/s, đủ mượt cho chat, nên đây là lựa chọn cân bằng tốt nhất cho máy tôi.

_Answer here._

---

## 3. Track 02 — llama-server load test

> Chạy 2 lần locust ở concurrency 10 và 50, paste tóm tắt bên dưới.

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
| ----------: | --------: | ------------: | -----------: | -----------: | -------: |
|          10 |      0.23 |         21000 |        45000 |        45000 |       0% |
|          50 |      0.22 |         19000 |        45000 |        45000 |       0% |

**KV-cache observation** (từ `record-metrics.py`): peak `llamacpp:kv_cache_usage_ratio` ở concurrency 10 = **0.0**, nghĩa là bộ nhớ cache vẫn còn rất dư dả cho model nhỏ này.

**Reflection** (≤ 60 chữ): Với TinyLlama, KV-cache usage gần như bằng 0 trên model 1.1B. Tuy nhiên, latency P95 lên đến 45s cho thấy CPU 4 nhân đời cũ là nút thắt cổ chai chính khi xử lý nhiều user cùng lúc.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: localhost only
- **N17 (Data pipeline):** stub: in-memory dict
- **N18 (Lakehouse):** stub: SQLite
- **N19 (Vector + Feature Store):** stub: TOY_DOCS

- embed: 0.0 ms
- retrieve: 0.1 ms
- llama-server: 12116 ms

**Reflection** (≤ 60 chữ): Bottleneck nằm hoàn toàn ở llama-server (LLM inference). Thời gian xử lý lên tới 12s mỗi câu hỏi là do CPU phải thực hiện prefill/decode trên model 1.1B mà không có hỗ trợ từ GPU. Các bước retrieve/embed gần như không đáng kể (0.1ms).

_Answer here._

---

## 5. Bonus — The single change that mattered most

> **Most important section.** Pick **một** thay đổi từ bonus track (build flag, thread sweep, quant pick, GPU offload, KV-cache quantization, speculative decoding, bất cứ challenge nào trong `BONUS-llama-cpp-optimization/CHALLENGES.md`) đã tạo ra speedup lớn nhất trên máy bạn.

**Change:** _<vd: rebuild llama.cpp với `-DGGML_NATIVE=ON -DGGML_BLAS=ON`; vd: hạ `-t` từ 12 xuống 6; vd: bật Metal trên M2>_

**Before vs after** (paste 2-3 dòng từ sweep output):

```
before: <số liệu>
after:  <số liệu>
speedup: ~<X.Y>×
```

**Tại sao nó work** (1–2 đoạn ngắn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với một bạn cùng lớp đang ngồi cạnh. Tránh "vibes-based" reasoning — bám vào mô hình mental của hardware (memory bandwidth? compute? cache?). Nếu kết quả khác kỳ vọng từ deck, nói rõ — đó là phần grader thưởng điểm._

---

## 6. (Optional) Điều ngạc nhiên nhất

_(1–2 câu — không bắt buộc, nhưng người grader đọc tất cả)_

_Answer here._

---

## 7. Self-graded checklist

- [ ] `hardware.json` đã commit
- [ ] `models/active.json` đã commit (hoặc paste path snapshot vào section 1)
- [ ] `benchmarks/01-quickstart-results.md` đã commit
- [ ] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [ ] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [ ] Ít nhất 6 screenshots trong `submission/screenshots/` (xem `submission/screenshots/README.md`)
- [ ] `make verify` exit 0 (chạy ngay trước khi push)
- [ ] Repo trên GitHub ở chế độ **public**
- [ ] Đã paste public repo URL vào VinUni LMS

---

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Nếu private, grader không xem được → 0 điểm.
