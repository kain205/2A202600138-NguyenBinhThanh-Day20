# Dependency Map — Synapsy · NOW Sprint
> Nguồn: Roadmap NOW column · Workshop 4
> Câu hỏi: Cái gì có thể giết dự án trong 30 ngày tới?

---

## 3 External Dependencies

### DEP-1 — MinerU Precision API
**Vai trò:** Gate của toàn bộ pipeline. Không có MinerU → không có markdown → không có thẻ.

**Worst-case:** MinerU trả về extraction quality <60% trên PDF y khoa VN có bảng biểu và sơ đồ giải phẫu — user nhận được thẻ flashcard sai/thiếu nội dung và mất tin ngay từ lần đầu.

**Plan B:** Self-host **Marker** (open-source PDF→Markdown, PyTorch-based) trên Railway/Render như một Python microservice. Kết quả tương đương MinerU Precision cho text-heavy PDF, deploy được trong 5–7 ngày.

**Cost Plan B:** $20–30/month Railway hosting + 4–5 ngày engineering. Trade-off: mất bảng biểu phức tạp, nhưng text + heading structure vẫn đủ để sinh thẻ.

---

### DEP-2 — OpenAI API (GPT-4o + GPT-4o-mini)
**Vai trò:** Stage 1 (planner) + Stage 2 (card generator). 50 beta users upload cùng lúc = 50 concurrent API calls.

**Worst-case:** 50 users hit rate limit (429) đồng thời vào giờ cao điểm trước kỳ thi — toàn bộ card generation fail, users nhìn thấy màn hình lỗi ngay lần đầu dùng thử.

**Plan B:** Implement **request queue + exponential backoff** (1–2 ngày). Nếu vẫn không đủ quota, chuyển Stage 2 sang **Claude Haiku** (JSON output mode tương đương, rẻ hơn 2x, rate limit cao hơn) — hai provider chạy song song, fallback tự động.

**Cost Plan B:** Claude Haiku ~$0.003/1K input tokens vs gpt-4o-mini ~$0.006/1K — tiết kiệm 50%. 1–2 ngày engineering để thêm fallback provider.

---

### DEP-3 — Firebase (Storage + Auth)
**Vai trò:** Firebase Storage lưu PDF tạm để MinerU fetch URL. Firebase Auth giữ session. Spark plan (free) có giới hạn storage operations và bandwidth.

**Worst-case:** 50 users × 20MB PDF = 1GB + storage operations vượt Spark plan free tier — tất cả upload đồng loạt fail với lỗi quota, không có cách nào upload PDF mới.

**Plan B:** Chuyển storage sang **Cloudflare R2** ($0.015/GB, không có egress fee, S3-compatible API). Firebase Auth giữ nguyên — chỉ swap storage layer. R2 presigned URL hoạt động tương tự Firebase Storage public URL cho MinerU fetch.

**Cost Plan B:** ~$0.15/month cho 10GB (50 users × 20MB, xóa sau khi MinerU xử lý xong). 2–3 ngày migration. Firebase Auth không đổi.

---

## Critical Path — NOW Sprint

**7 tasks chính để đạt PMF signal:**

```
[T1] Firebase Storage setup + MinerU API integration
      │
      ├──► [T2] PDF upload UI (drag & drop + progress bar)   ← PARALLEL, không blocking
      │
      └──► [T3] MinerU polling pipeline (poll → markdown/JSON)
                  │
                  └──► [T4] GPT-4o-mini card generation (parallel per lesson)
                              │
                              ├──► [T6] Gắn source metadata vào mỗi card   ← PARALLEL với T5
                              │         (page + line từ ContentList JSON)
                              │
                              └──► [T5] Card flip UI (Q → Reveal → A + difficulty buttons)
                                          │
                                          └──► [T7] "Xem Nguồn" link display
                                                    ("Trang 12 · Dòng 3 · [Mở nguồn]")
```

**Critical Path (chuỗi dài nhất — không thể song song):**
> **T1 → T3 → T4 → T5 → T7**

5 tasks nối tiếp nhau. Bất kỳ task nào trong chuỗi này bị block → toàn bộ sprint bị trễ.

**Tasks có thể song song (không trên Critical Path):**
- T2 (upload UI) — chạy song song với T3, merge vào sau
- T6 (source metadata) — chạy song song với T4→T5, merge vào trước T7

---

## Effort Estimate — Critical Path

| Task | Effort | Phụ thuộc |
|------|--------|-----------|
| T1 Firebase + MinerU setup | 3 ngày | — |
| T3 MinerU polling pipeline | 2 ngày | T1 |
| T4 Card generation (gpt-4o-mini) | 3 ngày | T3 |
| T5 Card flip UI | 3 ngày | T4 |
| T7 Xem Nguồn link | 2 ngày | T5 + T6 |
| **Total Critical Path** | **~13 ngày** | |

T2 + T6 song song: +3 ngày (không extend Critical Path nếu đủ bandwidth).

**Kết luận:** MVP NOW sprint = ~3 tuần với 1 developer full-time. Risk duy nhất trên Critical Path là DEP-1 (MinerU quality) — nếu cần chuyển sang Plan B thì T3 kéo dài thêm 5–7 ngày.

---

## Investor Summary

> Ba thứ có thể giết sprint này: **MinerU** (quality), **OpenAI** (rate limit), **Firebase** (quota).
> Cả ba đều có Plan B deploy được trong 7 ngày, không cần rebuild architecture.
> Critical Path là 13 ngày. Buffer thực tế: 3 tuần.
