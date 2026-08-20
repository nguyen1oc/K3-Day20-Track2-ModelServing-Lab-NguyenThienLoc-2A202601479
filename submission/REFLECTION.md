# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Nguyen Thien Loc - 2A202601479
**Cohort:** A20-K3
**Ngày submit:** 2026-08-20

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 11
- **CPU:** 11th Gen Intel Core i5-1135G7 @ 2.40 GHz
- **Cores:** 4 physical / 8 logical
- **CPU extensions:** AVX2
- **RAM:** 15.7 GB
- **Accelerator:** NVIDIA GeForce MX350 (2 GB CUDA)
- **llama.cpp asset đã tải:** `llama-b10488-bin-win-cuda-12.4-x64.zip`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** Q4_K_M + UD-Q2_K_XL

**Chạy ở đâu:** Laptop cá nhân
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Tôi dùng Qwen bản nhỏ để setup nhẹ hơn. Windows PowerShell ban đầu gặp lỗi encoding
với runner, nên tôi dùng phiên bản script tương thích. Sau đó CUDA runtime và cả hai
model chạy được trên máy, không cần dùng cloud fallback.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 52604 | 1031 / 1524 | 184.8 / 193.9 | 10610 / 13737 / 13737 | 5.4 |
| UD-Q2_K_XL | 0.39 | 3570 | 763 / 1002 | 182.5 / 205.0 | 12328 / 13859 / 13859 | 5.5 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 tiết kiệm 0.11 GB (khoảng 22%) và có TTFT P50 thấp hơn, nhưng TPOT cùng tốc độ
decode gần như không đổi. E2E P50 cao hơn vì độ dài câu trả lời thay đổi. Tôi chỉ dùng
Q2 khi cần tiết kiệm RAM hoặc ổ đĩa, không phải để tăng tốc rõ rệt.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.81 | 9300 | 16000 | 20000 | 8.1 | 0.0% |
| 50 | 0.47 | 22000 | 54000 | 55000 | 11.6 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 0.58×
- **P95 tăng:** 3.38×
- **Effective concurrency ở 50 users:** 11.6 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.92 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hoà ở 50 users. RPS giảm từ 0.81 xuống 0.47 trong khi P95 tăng từ 16 s
lên 54 s. Busy slots đạt 3.92/4 và deferred requests lên 46, nên phần latency tăng
thêm chủ yếu là thời gian chờ hàng đợi. Tôi sẽ tune `--parallel` trước vì nó điều
khiển trực tiếp số decode chạy đồng thời và có thể tăng goodput trước khi thêm phần cứng.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | not included | stub |
| N17 Data pipeline | not included | stub |
| N18 Lakehouse | not included | stub |
| N19 Vector + features | keyword-overlap fallback | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.1 ms
- llm: 5723.1 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck, đúng với kỳ vọng của tôi cho pipeline local nhỏ này. Để giảm một
nửa latency, tôi sẽ giảm decode trước bằng cách giới hạn số output tokens hoặc dùng
cấu hình serving nhanh hơn.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** Đổi số CPU threads từ 4 xuống 1 cho model Q4.

```
before:  13.0 tok/s ở 4 threads
after:   40.3 tok/s ở 1 thread
speedup: 3.10×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

Kết quả này khá bất ngờ: một thread nhanh hơn cấu hình mặc định bốn core. Với
`ngl=99`, phần lớn công việc của model được offload sang GPU nên decode không còn bị
giới hạn thuần bởi CPU. Nhiều CPU threads hơn làm tăng chi phí scheduling và đồng bộ
quanh GPU, nhưng không tạo thêm song song hữu ích.

Đường cong giảm đến tám threads rồi phục hồi một phần ở mười sáu threads. Hình dạng
không đơn điệu này cho thấy contention và biến động thời gian, không phải CPU scaling
đơn giản. Vì vậy tôi dùng một thread cho serving và load test.

---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** B5 / C8 semantic cache (offline demo)

**Numbers:**

```
before:  8 LLM calls, 2000 ms simulated decode
after:   5 LLM calls, 1250 ms simulated decode
speedup: 1.60× simulated
```

**Điều này nói lên gì mà deck chưa nói:**

Ở threshold 0.80, semantic cache hit 3/8 requests (38%) và bỏ qua 3 lần gọi LLM,
tương đương khoảng 750 ms decode mô phỏng. Các cache hit trả về gần như ngay lập tức,
nhưng threshold sweep phẳng vì offline mode dùng bag-of-words, khiến similarity chủ yếu
là 0 hoặc 1. Kết quả này chỉ chứng minh logic cache; muốn tune threshold đáng tin cậy
cần embedding model thật. Threshold quá thấp cũng có nguy cơ trả về câu trả lời cũ hoặc sai.

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [ ] `hardware.json` committed
- [ ] `models/active.json` committed
- [ ] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [ ] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [ ] `benchmarks/02-server-results.md` committed (`make load-report`)
- [ ] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [ ] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [ ] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [ ] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [ ] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` → **exit 0**
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [ ] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
