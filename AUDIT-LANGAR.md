# AUDIT-LANGAR — DEAD-CODE / DUPLICATES / TEST-GAPS / BUGS

> Branch: `audit/zcode-20260828` · Date: 2026-08-28 · Scope: repo `langar` (main @ 90171c8..4ff00f3)
> نقشهٔ کل ارگانیسم و رودمپ در شاخهٔ audit مخزن ofn-node است (SYSTEM-MAP/ROADMAP/SUMMARY-FA).

## A. کد مرده (verified by call-graph search)

| Item | Location | Evidence |
|---|---|---|
| BrainRouter | `langar/brain/brain_router.py` (89 خط) | صفر فراخوان در تولید؛ فقط test_upgrades.py |
| ACELoop | `langar/core/ace.py` (133 خط) | human_core.reflect_and_improve مستقیم SelfImprover را صدا می‌زند، نه ACE را |
| CoachAgent | `langar/agents/coach_agent.py` | در main.py:49 ساخته می‌شود؛ domains=[] ⇒ رباط هیچ‌وقتش انتخاب نمی‌کند؛ should_intervene/propose مرده |
| MemorySystem.search_semantic | `langar/core/memory.py:87` | pipeline روزانه فقط load_long_term(7) می‌خواند |
| Contract.allows/must_ask | `langar/core/contract.py` | /contract فقط نمایش می‌دهد؛ منطق مجوز اجرا نمی‌شود |
| tracer decorator | `langar/observability/tracer.py` | sink وصل (main.py:100) ولی @trace روی هیچ تابعی نیست |
| ai.py fallback path | `langar/ai.py` | فقط وقتی _CORE=None؛ در main.py هرگز |
| world_model._weather | `core/world_model.py:74-77` | همیشه None |
| research_wiring_example.py | ریشه | self-declared «not imported» |

## B. `_verify/` — کپی‌های سایهٔ خطرناک
- `_verify/db.py` (491 خط): schema v3 گیرکرده (اصلی v8)، تمام توابع v4-v8 غایب؛ **رفتار واگرا** در experiment_report (گارد `rv is not None` در `_verify/db.py:439` که `db.py:626` ندارد — نسخهٔ اصلی روی لیست خالی StatisticsError می‌دهد).
- `_verify/migrations.py`: SCHEMA_VERSION="3".
- `_verify/const_test.py`: 13 اصل در برابر 14 اصلی.
- `_verify/config_fresh.py`: فیلدهای search_provider/brave/serpapi غایب.
- ریسک: `_verify/test_arch.py` روی این کپی‌ها «سبز» می‌دهد ⇒ اعتماد کاذب. پیشنهاد: حذف یا sync مکانیکی از این شاخه پس از تأیید مالک.

## C. تکرارها (duplication اثبات‌شده — تنها مبنای مجاز بازنویسی)
1. `researcher/search_providers.py` ≡ `langar-pro/app/research/search_providers.py` — بایت‌به‌بایت یکسان.
2. `researcher/source_quality.py` ≡ نسخهٔ pro — یکسان.
3. `researcher/synthesizer.py` ≈ pro — **ناسازگار**: pro با `(SYS, question, ranked, target)` چهار آرگومان موقعیتی؛ brain اصلی امضای `(system, ctx_dict)` دارد ⇒ فراخوان pro با providers اصلی TypeError.
4. `core/constitution.py` (14 اصل) در برابر `app/research/constitution.py` (13 اصل — اصل «تناسب ارتباطات» غایب).
5. بانک سؤال دوتایی: `ai.py DOMAINS` (body/mind/…) در برابر `brain/question_bank.py BANK` (body_hrv/mind_emotion/…) — همان متن، کلید متفاوت.
6. نرمال‌ساز رقم فارسی دوتایی: `bot.py:72 _en()` و `hrv.py:14 _DIGITS`.
7. مدل هاردکد سه‌جایی: `claude-haiku-4-5-20251001` در providers.py:58، ai.py:202، pro/brain.py:48.

## D. شکاف تست
- صفر تست برای bot.py (1916 خط)، صفر تست async، صفر برای pro_client/ailab/budget/world_model/contract/safety/observability.
- `tests/test_upgrades.py:124` پرانتز بسته نشده ⇒ یک check هرگز اجرا نمی‌شود (خروجی رشتهٔ بی‌اثر).
- تست پرو (FastAPI/Postgres) وجود ندارد (خود README اعتراف دارد sandbox تست نشده).

## E. باگ‌ها (اولویت‌بندی مالک)
- **P1**: ناسازگاری فراخوان synthesizer در pro (بخش C-3) — مسیر تحقیق pro عملاً nonfunctional است.
- **P2**: `db.py:626` experiment_report روی روزهای بدون RMSSD می‌تواند `StatisticsError` بدهد (گارد نسخهٔ _verify درست‌تر است — برگرداندن همان گارد به نسخهٔ اصلی کوچک‌ترین پچ است).
- **P2**: `bot.py:1252` target="armin" و `researcher.py:22 TARGET_ARMIN` هاردکد — به config منتقل شود.
- **P3**: موارد B و D و حذف کد مرده پس از رأی.
- Rollback همه: revert تک‌کامیت؛ هر پچ additive و تست‌دار.

## F. وضعیت استقرار
- [FACT] این مخزن از 2026-06-28 روی گیت‌هاب تغییر نکرده؛ روی VPS اصلی (طبق langar_bot.service و deploy.sh) deploy شده بوده. وضعیت الانِ سرویس زندهٔ آن [UNKNOWN] — در رجیستری بات‌های والت، LangarBot به‌عنوان DORMANT (بدون env token) ثبت است؛ بات HRV فعال فعلاً در هیچ گره‌ای مشاهده نشد.
