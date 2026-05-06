# RICE Scoring — Synapsy MVP
> Framework: Score = (Reach × Impact × Confidence) / Effort
> Context: PMF signal = 50 users / 50+ thẻ / 48h sau lần upload đầu

---

## 5 Tính Năng Cốt Lõi

| # | Feature | PRD | Reach | Impact | Confidence | Effort (person-mo) | Score |
|---|---------|-----|-------|--------|------------|--------------------|-------|
| F1 | PDF Upload & Parse | PRD-01 | 50 | 3 | 100% | 0.5 | **300** |
| F3 | Auto Concept Cards | PRD-03 | 50 | 3 | 80% | 1.0 | **120** |
| F2 | Diagnostic Quiz + Study Roadmap | PRD-02 | 50 | 3 | 80% | 2.0 | **60** |
| F4 | Source Traceability (Xem Nguồn) | doc.md | 40 | 2 | 80% | 2.0 | **32** |
| F5 | Spaced Repetition System (SRS) | PRD-05 | 50 | 2 | 80% | 3.0 | **27** |


## 2×2 Value-Effort Matrix

Value = Reach × Impact × Confidence

| | Low Effort (≤1 mo) | High Effort (>1.5 mo) |
|---|---|---|
| **High Value (≥100)** | ✅ **F1** (300), **F3** (120) | 🎯 **F2** (60) |
| **Med/Low Value (<100)** | — | ⚠️ **F4** (32), ❌ **F5** (27) |

---

## Feature Descriptions

### F1 — PDF Upload & Parse
User kéo thả PDF → Firebase Storage → MinerU Precision API → markdown/JSON.
Gate feature: không có F1, không có gì khác hoạt động.
- Tech: MinerUParserAdapter, Firebase Admin SDK
- Success metric: >95% extract rate, <30s/20-page doc

### F2 — Diagnostic Quiz + Study Roadmap
GPT-4o đọc toàn bộ text một lần → tạo quiz chẩn đoán lỗ hổng → outline roadmap "học gì tối nay".
Core differentiator: giải quyết "ôn thi không biết bắt đầu từ đâu".
- Tech: gpt-4o, PLANNER_PROMPT, streaming output
- Success metric: Toàn bộ nội dung cốt lõi PDF xuất hiện trong outline

### F3 — Auto Concept Cards (Flashcard Generation)
GPT-4o-mini expand từng lesson song song (Promise.allSettled) → Concept Cards (Q → Reveal → A + difficulty rating).
Direct link to PMF: user phải có 50+ thẻ để swipe.
- Tech: gpt-4o-mini, parallel generation, fallback to outline nếu AI fail
- Success metric: >80% cards được interact (Reveal), lỗi <2%

### F4 — Source Traceability (Xem Nguồn)
Mỗi flashcard gắn tag page + line từ PDF gốc. Bấm "Xem Nguồn" → màn hình chia đôi, highlight đúng dòng.
Trust layer: phá vỡ rào cản tâm lý "AI bịa kiến thức".
- Tech: ContentList JSON từ MinerU, page/line metadata trên mỗi card
- Aha Moment: user nghi ngờ → bấm → xác nhận → swipe 20 thẻ liên tiếp

### F5 — Spaced Repetition System (SRS / SM-2)
CardProgress per user: interval, easeFactor, repetitions, nextReviewDate.
Càng dùng → SRS data tích lũy → lộ trình càng cá nhân → switching cost tăng.
Data flywheel = moat chính vs ChatGPT (stateless).
- Tech: SM-2 algorithm, IProgressRepository, ReviewCardUseCase, GetDueCardsUseCase
- Success metric: tỷ lệ "Good/Easy" tăng dần theo thời gian

---

## Reasoning (tại sao chấm vậy)

| Feature | Reach | Impact | Confidence | Effort | Ghi chú |
|---------|-------|--------|------------|--------|---------|
| F1 PDF Parse | 50 (gate — 100% users) | 3 | 100% | 0.5 | Pure infra, MinerU API docs đã có, Firebase setup chuẩn |
| F3 Concept Cards | 50 | 3 | 80% | 1.0 | gpt-4o-mini rẻ + parallel gen; direct link PMF 50 thẻ |
| F2 Diagnostic Quiz | 50 | 3 | 80% | 2.0 | Core differentiator nhưng GPT-4o prompt cho med content VN cần iteration |
| F4 Traceability | 40 (80% users nghi ngờ AI) | 2 | 80% | 2.0 | Split-screen + highlight UI phức tạp hơn logic |
| F5 SRS / SM-2 | 50 | 2 | 80% | 3.0 | Moat dài hạn nhưng 48h PMF window không cần scheduling — "review all" đủ |

**F4 Impact = 2 thay vì 3**: Traceability là Aha Moment, nhưng nó không *tạo ra* 50 thẻ — nó chỉ unlock sự tin tưởng để user tiếp tục swipe. Impact gián tiếp.

**F5 Impact = 2**: Trong cửa sổ 48h của PMF signal, SRS scheduling chưa kịp activate. User review toàn bộ thẻ trong session đầu tiên — không cần SM-2. SRS trở nên quan trọng từ tuần 2 trở đi (retention + moat).

---

## 2×2 Value-Effort Matrix

Value = Reach × Impact × Confidence

| | Low Effort (≤1 mo) | High Effort (>1.5 mo) |
|---|---|---|
| **High Value (≥100)** | ✅ **F1** (300), **F3** (120) | 🎯 **F2** (60) |
| **Med/Low Value (<100)** | — | ⚠️ **F4** (32), ❌ **F5** (27) |

---

## Quyết Định

**Quick Win (làm trước):** F1 → F3
- F1 là gate, không có lý do gì để không build trước. Effort thấp nhất, score cao nhất.
- F3 là đường thẳng nhất tới PMF signal (50 thẻ). gpt-4o-mini nhanh + rẻ.

**Strategic Bet (moat dài hạn):** F2 + F5
- F2 là core differentiator ("ôn thi không biết bắt đầu từ đâu") — phải build nhưng cần thời gian prompt engineering đúng.
- F5 là data flywheel — build sau PMF confirm, đây là lý do user không thể chuyển sang ChatGPT.

**Non-starter cho MVP v0:** F5 full SRS
- Thay bằng "review all cards" mode (0.3 person-month) — đủ để test PMF trong 48h.
- Full SM-2 scheduling build sau khi 50 user confirm xong.

**F4 — Quan sát đặc biệt:**
- RICE score thấp (32) vì effort cao + impact gián tiếp.
- NHƯNG đây là Aha Moment và Riskiest Assumption của doc.md.
- Giải pháp: build **MVP version của F4** — thay vì split-screen full, chỉ hiện "Trang 12 · Dòng 3 · [Mở nguồn]" text link → effort giảm từ 2.0 xuống 0.5 month → score tăng lên 128.
- **Recommendation: Build F4-lite trước, F4-full sau PMF.**

---

## Build Order Đề Xuất

```
Week 1–2:  F1 (gate)         → 0.5 mo
Week 2–4:  F3 (50 thẻ PMF)   → 1.0 mo
Week 4–6:  F4-lite (trust)   → 0.5 mo  ← MVP version only
Week 6–10: F2 (diagnostic)   → 2.0 mo  ← core differentiator
Post-PMF:  F5 full SRS       → 3.0 mo  ← moat
```

Total to PMF test: ~4 months (F1 + F3 + F4-lite + F2)
