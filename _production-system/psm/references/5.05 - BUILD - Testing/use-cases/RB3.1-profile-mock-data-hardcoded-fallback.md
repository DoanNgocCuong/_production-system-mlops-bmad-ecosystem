# 🔧 [PROB-ID]: Profile fetch fail → trả MOCK DATA hardcoded (tên/tuổi cứng) cho mọi user

> **1 câu:** Khi Profile API fail (404, timeout, parse error), hệ thống trả mock data cứng (name="Đoàn Cường", age=6) thay vì graceful degradation, gây ra lesson sinh sai cho user khác.

| Field | Value |
|-------|-------|
| **ID** | PROB-2026-03-18-profile-mock-data-hardcoded |
| **Type** | 🐛 Bug / 🏗️ Design Flaw |
| **Severity** | 🔴 SEV-2 |
| **Status** | ✅ Resolved |
| **Owner** | @doancuong |
| **Started** | 2026-03-18 14:21 |
| **Resolved** | 2026-03-18 |
| **Duration** | ~ 1 session |
| **Related** | [Langfuse Trace 059702af4e9a4457b0abcb5c978c0868](https://langfuse.com) |

---

# ═══════════════════════════════════════════════
# PHASE 1: VẤN ĐỀ — Chuyện gì đang xảy ra?
# ═══════════════════════════════════════════════

> _"Định nghĩa đúng vấn đề = giải quyết 50% vấn đề."_
> _Giải quyết triệu chứng ≠ Giải quyết nguyên nhân._

## 1.1 Trigger — Phát hiện vấn đề

**Phát hiện qua:** [ ] Code review / [ ] Testing / [ ] Alert / [x] Langfuse trace / [x] Manual investigation

**Symptom — Người dùng / hệ thống thấy gì:**

```json
{
    "user_id": "[MOCK DATA] 019cad43-0f4e-7697-9e69-9ab9ea970ae2",
    "profile_id": "[MOCK DATA] 019cad43-0f4e-7697-9e69-9ab9ea970ae2",
    "child": {
        "child_id": "mock_child_001",
        "child_name": "[MOCK DATA] Đoàn Cường",
        "age": 6,
        "interests": ["animals", "colors", "numbers"],
        "learning_history": ["colors", "numbers 1-5", "basic shapes"]
    },
    "language_preference": "vi",
    "raw_data": null
}
```

**Expected vs Actual:**

| | Expected | Actual |
|--|----------|--------|
| Khi API fail | Graceful degradation: child=None, pipeline dùng parent_config fallback | Trả mock data cứng: name="Đoàn Cường", age=6 cho MỌI user |
| Lesson output | Dùng tên/tuổi thật của user hoặc generic "bạn" | Lesson chứa "Đoàn Cường, 6 tuổi" cho user không phải Cường |
| Alert/Log | Error level + alert Google Chat | Warning level, không alert, stack trace bị nuốt |

## 1.2 Problem Statement — Phát biểu vấn đề

> ⚠️ **Luật:** Viết SYMPTOM trước. Chưa viết ROOT CAUSE ở bước này.

**Problem Statement (1-3 câu):**

Khi gọi Profile API fail (bất kỳ lý do nào), `ProfileService.fetch_profile()` trả về mock data hardcoded với tên="Đoàn Cường" và tuổi=6. Pipeline nhận mock data này, coi như thật, rồi sinh lesson cho "Đoàn Cường 6 tuổi" — hoàn toàn sai cho user khác. Ngoài ra, field `age` từ API có thể là date string ("27/10/2017") gây Pydantic ValidationError → cũng rơi vào mock data.

## 1.3 Context & Constraints — Bối cảnh và ràng buộc

- **Hệ thống liên quan:** ProfileService → ProfileClient → External API (`robot-api.hacknao.edu.vn`)
- **Pipeline flow:** Profile fetch chạy song song với Memory + Vision → kết quả đưa vào prompt builder
- **Ràng buộc:** Pipeline phải tiếp tục khi profile fail (không được crash cả request)
- **Design intent ban đầu:** Mock data là placeholder trong giai đoạn dev, nhưng không được thay thế trước production

---

# ═══════════════════════════════════════════════
# PHASE 2: NGUYÊN NHÂN — Tại sao xảy ra?
# ═══════════════════════════════════════════════

> _"Nghi ngờ suy nghĩ của chính mình."_
> _— First Principles: Bóc tách từng lớp, không chấp nhận bề mặt._

## 2.1 Hypothesis Generation — Đặt giả thuyết (MECE)

> Phân tích MECE: Tất cả cases dẫn đến MOCK DATA

```
MOCK DATA được trả khi nào?
│
├── 1. KHÔNG LẤY ĐƯỢC DATA TỪ API (ProfileClientError)
│   ├── 1.1  HTTP 4xx (401/403/404/429)
│   ├── 1.2  HTTP 5xx (500/502/503/504)
│   ├── 1.3  Timeout > 15s
│   └── 1.4  Network error (DNS/connection/SSL)
│
└── 2. LẤY ĐƯỢC DATA NHƯNG PARSE LỖI (generic Exception)
    ├── 2.1  Response body không phải JSON (JSONDecodeError)
    ├── 2.2  JSON không phải dict (AttributeError)
    ├── 2.3  language_mode = null → .lower() crash (AttributeError)
    └── 2.4  age = "27/10/2017" → Pydantic int parse fail (ValidationError)
```

| # | Giả thuyết | Khả năng (%) | Cách kiểm chứng | Status |
|---|-----------|--------------|------------------|--------|
| H1 | API trả 404 vì profile không tồn tại | High | curl API trực tiếp | ✅ Confirmed |
| H2 | age field là date string gây parse error | High | Xem API response thật | ✅ Confirmed |
| H3 | language_mode = null gây crash | Med | Code review | ✅ Confirmed (bug tiềm ẩn) |

## 2.2 Investigation Log — Kiểm chứng từng giả thuyết

```
[14:21] ── Phát hiện trên Langfuse trace ──
         Observation: profile_fetch latency=0.07s, output chứa "[MOCK DATA]"
         → Latency 70ms → loại trừ timeout (cần >15s)

[14:25] ── Test H1: curl API trực tiếp ──
         Action: curl "https://robot-api.hacknao.edu.vn/robot/api/v1/llm/user_profile?user_id=019cad43...&token=b1812cb7..."
         Result: HTTP 404 - {"status":404,"message":"Content not found. Type: Profile - id: 019cad43..."}
         Response time: 0.079s (khớp với Langfuse 0.07s)
         → ✅ XÁC NHẬN - Profile ID không tồn tại trên API

[14:30] ── Test H2: API trả age là date string ──
         Action: Kiểm tra Pydantic error log
         Result: "1 validation error for ChildProfile age - unable to parse string as integer, input_value='27/10/2017'"
         → ✅ XÁC NHẬN - age field từ API có 2 format: "4" (number string) và "27/10/2017" (date)

[14:35] ── Code review: language_mode null ──
         Action: Đọc code line 101-102
         Result: data.get("language_mode", "vi") trả None khi key tồn tại với value null
                 → language_mode.lower() → AttributeError
         → ✅ XÁC NHẬN - Bug tiềm ẩn, chưa trigger trong production
```

**Debug commands đã dùng:**

```bash
# Test API trực tiếp
curl -s -w "\nHTTP_CODE: %{http_code}\nTIME_TOTAL: %{time_total}s\n" \
  "https://robot-api.hacknao.edu.vn/robot/api/v1/llm/user_profile?user_id=019cad43-0f4e-7697-9e69-9ab9ea970ae2&token=b1812cb7-2513-408b-bb22-d9f91b099fbd"
```

## 2.3 Root Cause — 5 Whys

```
Symptom: Lesson chứa tên "Đoàn Cường" cho user không phải Cường

Why 1: Tại sao lesson chứa tên sai?
  → Vì pipeline nhận mock data có name="Đoàn Cường", age=6

Why 2: Tại sao trả mock data?
  → Vì ProfileService catch exception và return UserProfile hardcoded

Why 3: Tại sao exception xảy ra?
  → Case 1: API trả 404 (profile không tồn tại)
  → Case 2: age="27/10/2017" → Pydantic parse fail

Why 4: Tại sao mock data có tên cứng thay vì graceful degradation?
  → Design ban đầu dùng mock data cho dev, chưa thay thế cho production

Why 5: Tại sao pipeline không biết data là fake?
  → Check mock dựa trên string prefix "[MOCK DATA]" (fragile), không có flag rõ ràng

✅ ROOT CAUSE (multi):
  1. Mock data hardcoded chặn pipeline fallback logic (đã có sẵn cho child=None)
  2. age field không xử lý date format từ API
  3. language_mode null không được handle

Why 6: Tại sao bug này không được fix sớm hơn?
  → Lúc tester và dev cùng test cũng đã gặp 1 lần tương tự (mock data xuất hiện)
  → DevOps check config → không thay đổi gì → lần test sau "tự hết"
  → Chủ quan bỏ qua, không điều tra root cause
  → Thực tế: bug KHÔNG TỰ HẾT — chỉ là data khác nhau giữa các profile:
    - Profile A có age="4" (int string) → parse OK → không crash
    - Profile B có age="27/10/2017" (date string) → crash → fallback mock data
    - Lần test sau nếu đúng Profile A → "tự hết" → nhưng bug vẫn nằm đó

⚠️ BÀI HỌC: Bug intermittent trên production mà "tự hết" thì KHÔNG PHẢI HẾT.
   Phải điều tra root cause ngay lần đầu gặp, không chờ "confirm config không đổi".
   Đặc biệt khi external API trả data không nhất quán giữa các user/profile.
```

**Contributing Factors:**

| Factor | Ảnh hưởng | Preventable? |
|--------|-----------|--------------|
| `logger.warning` thay vì `logger.error` | Không trigger alert, khó phát hiện | ✅ |
| Không có `exc_info=True` | Mất stack trace | ✅ |
| Không có alert khi profile fetch fail | Không ai biết để fix | ✅ |
| Mock data block pipeline fallback | Pipeline fallback cho child=None đã có nhưng không bao giờ chạy | ✅ |
| **Chủ quan khi bug "tự hết"** | Bug intermittent không được điều tra → tồn tại đến production | ✅ |
| **API trả data không nhất quán** | age="4" vs age="27/10/2017" tùy profile → bug chỉ xuất hiện với 1 số user | ⚠️ Cần defensive parsing |

---

# ═══════════════════════════════════════════════
# PHASE 3: GIẢI PHÁP — Các hướng xử lý (MECE)
# ═══════════════════════════════════════════════

> _Liệt kê TẤT CẢ options → Đánh giá → Chọn → Giải thích tại sao loại các cái khác._

## 3.1 Solution Space — Các phương án

| # | Option | Mô tả (1-2 câu) | Effort | Risk | Leverage |
|---|--------|------------------|--------|------|----------|
| A | **Degraded profile (child=None)** | Khi fail, trả UserProfile(child=None, is_degraded=True). Pipeline tự fallback parent_config. | 🟢 Low | 🟢 Low | 🟢 High |
| B | **Raise exception lên pipeline** | ProfileService raise, pipeline catch và quyết định tiếp hay dừng | 🟡 Med | 🟡 Med | 🟡 Med |
| C | **Retry + circuit breaker** | Retry 2-3 lần, circuit breaker khi fail liên tục | 🔴 High | 🟡 Med | 🟢 High |

## 3.2 Decision — Chọn phương án

> **Chọn: Option A** (+ fix age parsing + fix language_mode null + alert)

| Item | Detail |
|------|--------|
| **Chọn** | Option A: Degraded profile |
| **Lý do** | Pipeline ĐÃ CÓ fallback cho child=None (lines 216-218). Chỉ cần bỏ mock data là fallback tự chạy. Minimal change, maximum impact. |
| **Rejected B** | Cần refactor pipeline, risk cao hơn, cùng kết quả |
| **Rejected C** | Over-engineering cho giai đoạn hiện tại, có thể làm sau |
| **Rollback plan** | Revert commit nếu có vấn đề |

---

# ═══════════════════════════════════════════════
# PHASE 4: THỬ NGHIỆM — Validate trước khi ship
# ═══════════════════════════════════════════════

## Experiment 1: Fix age parsing — handle cả 2 format

| Item | Detail |
|------|--------|
| **Giả thuyết** | `_parse_age()` có thể handle cả "4" (string) và "27/10/2017" (date) |
| **Setup** | Viết static method với fallback logic |
| **Expected** | "4"→4, "27/10/2017"→8, None→None, "abc"→None |
| **Actual** | ✅ 9/9 test cases pass |

```python
# Test results:
# "4" (string number)        → 4   ✅
# 4 (int)                    → 4   ✅
# "27/10/2017" (child date)  → 8   ✅
# "01/06/1993" (adult date)  → 32  ✅
# "2020-05-15" (ISO date)    → 5   ✅
# "33" (adult age string)    → 33  ✅
# None                       → None ✅
# "" (empty)                 → None ✅
# "abc" (garbage)            → None ✅ (+ log warning)
```

## Experiment 2: Degraded profile — pipeline fallback hoạt động

| Item | Detail |
|------|--------|
| **Giả thuyết** | Khi child=None, pipeline fallback về parent_config |
| **Setup** | Trace code path trong lesson_pipeline.py lines 213-218 |
| **Expected** | `child_age = parent_config.child_age`, `child_name = parent_config.child_name` |
| **Actual** | ✅ Code path confirmed — else branch handles child=None correctly |

```python
# lesson_pipeline.py lines 213-218 (đã có sẵn!)
if user_profile.child:
    child_age = user_profile.child.age or (parent_config.child_age ...)
    child_name = user_profile.child.child_name or (parent_config.child_name ...)
else:  # ← Khi child=None, branch này chạy
    child_age = parent_config.child_age if parent_config else None
    child_name = parent_config.child_name if parent_config else None
```

---

# ═══════════════════════════════════════════════
# PHASE 5: TRIỂN KHAI — Implementation
# ═══════════════════════════════════════════════

## 5.1 Changes — Các thay đổi

**Files changed:**

| File | Change | Why |
|------|--------|-----|
| `app/domains/profile/domain/entities.py` | Thêm `is_degraded: bool = False`, `degraded_reason: str \| None = None` vào UserProfile | Flag rõ ràng thay vì string check, kèm lý do cụ thể |
| `app/domains/profile/application/services/profile_service.py` | 1. Thêm `_parse_age()` xử lý 2 format<br>2. Fix `language_mode` null<br>3. Xóa `MOCK_USER_PROFILE` dead code<br>4. Thay mock data → degraded profile (child=None)<br>5. Tách parse từng field riêng biệt (age, learning_history, language_mode)<br>6. `logger.error` + `alert=` keyword<br>7. `send_alert_safe()` HIGH cho API fail, MEDIUM cho parse degrade | Root cause fixes + field isolation + observability |
| `app/pipeline/lesson_pipeline.py` | Thay `startswith("[MOCK DATA]")` → `user_profile.is_degraded` | Clean flag check |

**Key code diff:**

```python
# ❌ BEFORE (profile_service.py — except block):
except ProfileClientError as exc:
    logger.warning("profile.fetch.error", ...)  # Warning level, không stack trace
    return UserProfile(
        user_id=f"[MOCK DATA] {profile_id}",    # Hardcoded prefix
        child=ChildProfile(
            child_name="[MOCK DATA] Đoàn Cường",  # ❌ Tên cứng cho MỌI user
            age=6,                                  # ❌ Tuổi cứng
            interests=["animals", "colors", "numbers"],
        ),
        language_preference="vi",
    )

# ✅ AFTER — API error (ProfileClientError + generic Exception):
except ProfileClientError as exc:
    logger.error("profile.fetch.api_error", ..., alert="profile_api_unreachable")
    send_alert_safe(
        alert_type=AlertType.EXTERNAL_API_ERROR,
        level=AlertLevel.HIGH,
        message=f"Profile API failed → child=None for profile_id={profile_id}",
        ...
    )
    return UserProfile(
        user_id=profile_id,
        profile_id=profile_id,
        child=None,                      # ← Pipeline fallback tự chạy
        is_degraded=True,                # ← Flag rõ ràng
        degraded_reason=f"API error: {type(exc).__name__}: {exc}",
    )
```

```python
# ❌ BEFORE (parse logic — 1 try/catch lớn bọc toàn bộ):
child = ChildProfile(
    child_id=profile_id,
    child_name=data.get("name"),
    age=data.get("age"),  # "27/10/2017" → Pydantic crash → except → MOCK DATA
    learning_history=data.get("last_vocabulary", "").split(","),
)

# ✅ AFTER — Từng field parse riêng biệt:
# 2a. Parse age — track degradation by return value
raw_age = data.get("age")
parsed_age = self._parse_age(raw_age)
if raw_age is not None and parsed_age is None:
    degraded_fields.append("age")

# 2b. Parse learning history — isolated try/except
learning_history: list[str] = []
try:
    last_vocab = data.get("last_vocabulary")
    if last_vocab and isinstance(last_vocab, str):
        learning_history = [v.strip() for v in last_vocab.split(",") if v.strip()]
except Exception as exc:
    degraded_fields.append("learning_history")

child = ChildProfile(child_id=profile_id, child_name=str(child_name),
                     age=parsed_age, learning_history=learning_history)
```

```python
# ❌ BEFORE (language_mode):
language_mode = data.get("language_mode", "vi")  # None khi key tồn tại, value null
language_preference = lang_map.get(language_mode, language_mode.lower())  # AttributeError!

# ✅ AFTER — Defensive with str() wrap:
raw_lang = data.get("language_mode")
if raw_lang is not None:
    lang_str = str(raw_lang).strip()
    language_preference = lang_map.get(lang_str, lang_str.lower() if lang_str else "vi")
```

```python
# ❌ BEFORE (pipeline mock detection):
is_mock_data = user_profile.user_id.startswith("[MOCK DATA]")  # Fragile string check

# ✅ AFTER:
is_mock_data = user_profile.is_degraded  # Clean flag
```

```python
# ✅ NEW — Alert khi parse degradation (API OK nhưng data lạ):
if is_degraded:
    send_alert_safe(
        alert_type=AlertType.INVALID_JSON_FORMAT,  # API trả data, nhưng format sai
        level=AlertLevel.MEDIUM,
        message=f"Profile parse degraded for profile_id={profile_id}: {degraded_reason}",
        context={"profile_id": profile_id, "degraded_fields": degraded_fields},
    )
```

---

# ═══════════════════════════════════════════════
# PHASE 6: KIỂM TRA — Verify fix đã work
# ═══════════════════════════════════════════════

## 6.1 MECE: Tất cả cases sau khi fix

```
Profile fetch → UserProfile trả về
│
├── 1. API CALL THẤT BẠI
│   ├── 1.1  ProfileClientError (HTTP 4xx/5xx)
│   ├── 1.2  Timeout > 15s
│   ├── 1.3  Network error (DNS/connection/SSL)
│   └── 1.4  Unexpected Exception
│   │
│   └─→ KẾT QUẢ: UserProfile(child=None, is_degraded=True, degraded_reason="API error: ...")
│        ✅ Log: logger.error("profile.fetch.api_error", alert="profile_api_unreachable")
│        ⚠️ Alert: send_alert_safe(EXTERNAL_API_ERROR, HIGH) — no-op nếu aiohttp chưa cài
│        ✅ Luồng tiếp: pipeline fallback → parent_config hoặc prompt default 6-12 tuổi
│        ❌ KHÔNG trả MOCK DATA "Đoàn Cường"
│
└── 2. API CALL THÀNH CÔNG → PARSE RESPONSE
    │
    ├── 2.0  Parse thành công hoàn toàn
    │   └─→ KẾT QUẢ: UserProfile(child=ChildProfile(...), is_degraded=False)
    │        ✅ Log: logger.info("profile.fetch.success")
    │
    ├── 2.1  Response body không phải dict / data không phải dict
    │   └─→ KẾT QUẢ: UserProfile(child=None, is_degraded=True, degraded_reason="data_envelope")
    │        ✅ Log: logger.error(alert="profile_parse_unexpected_type")
    │        ✅ Alert: send_alert_safe(INVALID_JSON_FORMAT, MEDIUM)
    │
    ├── 2.2  age = "27/10/2017" (date string thay vì int) ← ★ CASE ĐANG GẶP
    │   └─→ KẾT QUẢ: _parse_age() convert → tuổi thật (8 tuổi)
    │        UserProfile(child=ChildProfile(age=8), is_degraded=False)
    │        ✅ KHÔNG crash, KHÔNG fallback mock data
    │
    ├── 2.3  age = "abc" hoặc format lạ không parse được
    │   └─→ KẾT QUẢ: _parse_age() → None + warning log
    │        UserProfile(child=ChildProfile(age=None), is_degraded=True, degraded_reason="age")
    │        ✅ Log: logger.warning("profile.age_parse_failed")
    │        ✅ Alert: send_alert_safe(INVALID_JSON_FORMAT, MEDIUM)
    │        ✅ Luồng tiếp: prompt dùng default 6-12 tuổi
    │
    ├── 2.4  language_mode = null / unexpected type
    │   └─→ KẾT QUẢ: language_preference="vi" (default)
    │        ✅ str() wrap đảm bảo không crash với non-string
    │        ✅ Log: logger.error(alert="profile_parse_language_mode") nếu crash
    │
    ├── 2.5  language_mode = "XX" (unknown value)
    │   └─→ KẾT QUẢ: language_preference="xx" (lowercase, pass through)
    │        ✅ Không crash, downstream handle unknown gracefully
    │
    ├── 2.6  name = null/empty → child = None
    │   └─→ KẾT QUẢ: UserProfile(child=None, is_degraded=False)
    │        ✅ Luồng tiếp: pipeline fallback parent_config
    │
    └── 2.7  last_vocabulary parse fail
        └─→ KẾT QUẢ: learning_history=[], degraded_fields=["learning_history"]
             ✅ Log: logger.error(alert="profile_parse_learning_history")
             ✅ Alert: send_alert_safe(INVALID_JSON_FORMAT, MEDIUM)
```

**Alert matrix:**

| Scenario | AlertType | AlertLevel | Khi nào fire |
|----------|-----------|------------|-------------|
| API call fail (1.x) | `EXTERNAL_API_ERROR` | `HIGH` | API unreachable/error |
| Parse degraded (2.1, 2.3, 2.4, 2.7) | `INVALID_JSON_FORMAT` | `MEDIUM` | API OK nhưng data format sai |
| Parse OK (2.0, 2.2, 2.5, 2.6) | — | — | Không alert |

> ⚠️ Alert hiện tại là **no-op** nếu `aiohttp` chưa cài trong Docker. Code đã sẵn sàng — chỉ cần cài `aiohttp` vào Docker image là alert hoạt động.

## 6.2 Verification checklist

- [x] Symptom ban đầu đã hết: Không còn "[MOCK DATA] Đoàn Cường" cho user khác
- [x] `MOCK_USER_PROFILE` dead code đã xóa khỏi codebase
- [x] age="4" parse thành công → 4
- [x] age="27/10/2017" parse thành công → 8
- [x] age=None/garbage → None (không crash, is_degraded=True)
- [x] language_mode=null → fallback "vi" (không crash)
- [x] Khi API fail → child=None → pipeline dùng parent_config
- [x] Khi API fail → `send_alert_safe(EXTERNAL_API_ERROR, HIGH)` (no-op nếu aiohttp missing)
- [x] Khi parse degrade → `send_alert_safe(INVALID_JSON_FORMAT, MEDIUM)` (no-op nếu aiohttp missing)
- [x] Alert import wrapped try/except → app không crash khi aiohttp missing
- [x] Pipeline dùng `user_profile.is_degraded` thay vì `startswith("[MOCK DATA]")` (cả 2 chỗ)
- [x] Từng field parse riêng biệt: age fail không ảnh hưởng learning_history

---

# ═══════════════════════════════════════════════
# PHASE 7: ĐÚC KẾT — Kaizen & Knowledge Capture
# ═══════════════════════════════════════════════

## 7.1 Common Mistakes — Các lỗi đã mắc

### M-01: Dùng mock data hardcoded làm fallback trong production

**Severity:** 🔴

❌ **Sai — Đừng làm thế này:**
```python
except Exception:
    return UserProfile(
        child=ChildProfile(child_name="Đoàn Cường", age=6),  # Tên/tuổi cứng
    )
```
> **Tại sao sai:** Mọi user khi API fail đều nhận data của "Đoàn Cường", prompt sinh lesson sai. Mock data cho dev không nên có trong production code.

✅ **Đúng — Làm thế này:**
```python
except Exception:
    return UserProfile(child=None, is_degraded=True)
    # Pipeline tự fallback: parent_config → hoặc generic "bạn"
```
> **Tại sao đúng:** Không có data giả, pipeline tự xử lý fallback có sẵn

🔍 **Detect trong codebase:**
```bash
rg "MOCK DATA|mock_child|mock_user" app/ --include="*.py"
```

---

### M-02: Nuốt exception bằng logger.warning thay vì logger.error

**Severity:** 🟠

❌ **Sai — Đừng làm thế này:**
```python
except ProfileClientError as exc:
    logger.warning("profile.fetch.error", error_message=str(exc))
    # Không có exc_info → mất stack trace
    # Warning level → không trigger alert
```
> **Tại sao sai:** Warning dễ bị bỏ qua. Không có stack trace → debug rất khó.

✅ **Đúng — Làm thế này:**
```python
except ProfileClientError as exc:
    logger.error("profile.fetch.error", error_message=str(exc), exc_info=True)
    send_alert_safe(AlertType.EXTERNAL_API_ERROR, AlertLevel.MEDIUM, ...)
```
> **Tại sao đúng:** Error level trigger alert. exc_info=True cho full stack trace.

🔍 **Detect trong codebase:**
```bash
rg "logger\.warning.*error" app/ --include="*.py"
```

---

### M-03: Không xử lý đa format từ external API

**Severity:** 🟠

❌ **Sai — Đừng làm thế này:**
```python
age=data.get("age")  # External API trả "4" hoặc "27/10/2017"
# → Pydantic expect int, crash khi nhận date string
```
> **Tại sao sai:** External API không kiểm soát được format. Phải phòng thủ.

✅ **Đúng — Làm thế này:**
```python
age=self._parse_age(data.get("age"))
# _parse_age: "4"→4, "27/10/2017"→8, "abc"→None (không crash)
```
> **Tại sao đúng:** Defensive parsing — xử lý mọi format, không crash.

🔍 **Detect trong codebase:**
```bash
rg "data\.get\(" app/domains/profile/ --include="*.py" | grep -v "_parse"
```

---

### M-04: `dict.get(key, default)` không fallback khi value là None

**Severity:** 🟡

❌ **Sai — Đừng làm thế này:**
```python
language_mode = data.get("language_mode", "vi")
# Key tồn tại với value null → trả None, KHÔNG trả "vi"
language_mode.lower()  # → AttributeError: 'NoneType' has no attribute 'lower'
```
> **Tại sao sai:** `dict.get()` chỉ dùng default khi KEY không tồn tại, không phải khi VALUE là None.

✅ **Đúng — Làm thế này:**
```python
language_mode = data.get("language_mode") or "vi"
# or "vi" fallback cả khi value là None hoặc empty string
```
> **Tại sao đúng:** `or` operator fallback cho cả None, "", 0, False — đúng semantic cần dùng.

🔍 **Detect trong codebase:**
```bash
rg 'data\.get\("[^"]+",\s*"[^"]+"\)' app/ --include="*.py"
# Tìm pattern: data.get("key", "default") → review xem có cần đổi sang `or` không
```

---

## 7.2 Best Practices Checklist

> ✅ Must-have | ⚠️ Should-have | 💡 Nice-to-have

### External API Integration

- [ ] ✅ KHÔNG dùng mock data hardcoded làm fallback trong production
- [ ] ✅ Dùng graceful degradation (trả data rỗng) thay vì data giả
- [ ] ✅ Defensive parsing cho mọi field từ external API (type có thể thay đổi)
- [ ] ✅ `logger.error` + `exc_info=True` khi external API fail
- [ ] ✅ Gửi alert khi external API fail (dùng `send_alert_safe`)
- [ ] ⚠️ Dùng `or` thay vì `dict.get(key, default)` khi cần fallback cho null value
- [ ] 💡 Retry + circuit breaker cho external APIs quan trọng

### Pipeline Resilience

- [ ] ✅ Pipeline phải tiếp tục khi 1 dependency fail (profile, memory, vision)
- [ ] ✅ Dùng flag (`is_degraded`) thay vì string prefix để detect fallback
- [ ] ⚠️ Log `is_degraded` trong metadata để track tỷ lệ fallback

---

## 7.3 Action Items — Ngăn tái phát

| # | Action | Priority | Owner | Due | Status |
|---|--------|----------|-------|-----|--------|
| 1 | Fix mock data → degraded profile (child=None) | P0 | @doancuong | 2026-03-18 | ✅ Done |
| 2 | Fix age parsing (2 format: int + date string) | P0 | @doancuong | 2026-03-18 | ✅ Done |
| 3 | Fix language_mode null (str() wrap + `or "vi"`) | P0 | @doancuong | 2026-03-18 | ✅ Done |
| 4 | Thêm `send_alert_safe()` — HIGH cho API fail, MEDIUM cho parse degrade | P1 | @doancuong | 2026-03-18 | ✅ Done (⚠️ import wrapped try/except — xem note bên dưới) |
| 5 | Thêm `is_degraded` + `degraded_reason` flag + pipeline update | P1 | @doancuong | 2026-03-18 | ✅ Done |
| 6 | Tách parse từng field riêng biệt (age, learning_history, language_mode) | P1 | @doancuong | 2026-03-18 | ✅ Done |
| 7 | Xóa `MOCK_USER_PROFILE` dead code + PII | P1 | @doancuong | 2026-03-18 | ✅ Done |
| 8 | Fix alert infrastructure — `aiohttp` missing trong Docker → app crash khi import | P1 | @doancuong | Backlog | ⏳ |
| 9 | Audit toàn bộ `data.get("key", default)` pattern | P2 | @doancuong | 2026-03-25 | ⏳ |
| 10 | Retry + circuit breaker cho Profile API | P3 | @doancuong | Backlog | ⏳ |

> **⚠️ Note về Alert (item 4 & 8):** `send_alert_safe` import trong `profile_service.py` được wrap bằng `try/except ImportError` vì alert infrastructure (`google_chat_transport.py`) phụ thuộc `aiohttp` — chưa cài trong Docker image. Khi `aiohttp` missing → `send_alert_safe` trở thành no-op → **alert không gửi nhưng app không crash**. Cần fix riêng alert infrastructure (item 8) để alert hoạt động đúng trên production.

---

## 7.4 Lessons Learned

**Làm tốt ✅ (lặp lại):**
- Phân tích MECE tất cả cases dẫn đến mock data trước khi fix
- Dùng curl test API trực tiếp → xác nhận root cause trong 5 phút
- Kiểm tra pipeline fallback logic đã có sẵn (không cần viết mới)
- Tách parse từng field riêng biệt → 1 field fail không kéo theo mất toàn bộ data
- Phân biệt AlertType theo semantic: `EXTERNAL_API_ERROR` (API fail) vs `INVALID_JSON_FORMAT` (data lạ)

**Cần cải thiện ⚠️ (fix process):**
- Không nên dùng mock data hardcoded trong production code
- Cần alert cho external API failures (hiện không biết khi nào fail) → ✅ Đã fix
- Cần defensive parsing cho mọi field từ external API → ✅ Đã fix
- Dead code với PII (tên thật) không nên tồn tại trong codebase → ✅ Đã xóa

**Câu hỏi mở ❓ (research thêm):**
- Profile API trả age là date hay number — có contract/schema không?
- Tỷ lệ profile fetch fail trên production là bao nhiêu? (Giờ đã có alert, có thể track)
- Có cần cache profile để giảm API calls?

---

## 7.5 Quick Reference Card

```
╔══════════════════════════════════════════════════════════════════╗
║  PROFILE FETCH FALLBACK — Quick Ref                            ║
╠══════════════════════════════════════════════════════════════════╣
║  🔴 M-01: mock data hardcoded → degraded profile (None)       ║
║  🟠 M-02: logger.warning → logger.error + alert= keyword      ║
║  🟠 M-03: data.get("age") raw → _parse_age() defensive       ║
║  🟡 M-04: get("key", "default") → get("key") or "default"    ║
╠══════════════════════════════════════════════════════════════════╣
║  ✅ BP-01: child=None → pipeline fallback parent_config        ║
║  ✅ BP-02: is_degraded + degraded_reason thay vì string prefix ║
║  ✅ BP-03: send_alert_safe(HIGH) khi API fail                  ║
║  ✅ BP-04: send_alert_safe(MEDIUM) khi parse degraded          ║
║  ✅ BP-05: parse từng field riêng biệt (field isolation)       ║
╠══════════════════════════════════════════════════════════════════╣
║  Alert mapping:                                                 ║
║    API fail     → EXTERNAL_API_ERROR / HIGH                     ║
║    Parse degrade → INVALID_JSON_FORMAT / MEDIUM                 ║
╠══════════════════════════════════════════════════════════════════╣
║  Test API:                                                      ║
║  curl -s "https://robot-api.hacknao.edu.vn/robot/api/v1/      ║
║    llm/user_profile?user_id=XXX&token=YYY"                     ║
║  Doc: docs/.../RB3.1-profile-mock-data-hardcoded-fallback      ║
╚══════════════════════════════════════════════════════════════════╝
```

---

# APPENDIX

## A. Relevant logs

```
# Profile API trả 404
{"event": "profile_http_error", "user_id": "019cad43...", "status": 404}

# Pydantic parse error (age là date string)
1 validation error for ChildProfile
age
  Input should be a valid integer, unable to parse string as an integer
  [type=int_parsing, input_value='27/10/2017', input_type=str]

# language_mode null (tiềm ẩn)
AttributeError: 'NoneType' object has no attribute 'lower'
```

## B. API Response Examples

```json
// Case 1: Profile tồn tại
{
    "status": 200,
    "message": "Lấy user profile thành công",
    "data": {
        "name": "Truck-kun",
        "age": "4",
        "language_mode": "VI-EN",
        "birthday": "01/06/1993",
        "favorite_movie": "Hà Nộ",
        "gender": "Nam",
        "learn_level": "Pre A1"
    }
}

// Case 2: Profile không tồn tại
{
    "status": 404,
    "message": "Content not found. Type: Profile - id: 019cad43...",
    "data": {}
}
```

## C. References

- Profile API: `https://robot-api.hacknao.edu.vn/robot/api/v1/llm/user_profile`
- Langfuse Trace: `059702af4e9a4457b0abcb5c978c0868`
- Pipeline fallback logic: `app/pipeline/lesson_pipeline.py` lines 213-218

## D. Incident phát sinh khi fix: `AlertType = None` crash production

### 1. Vấn đề

Deploy fix lên production → app crash tại line 284:
```
AttributeError: 'NoneType' object has no attribute 'INVALID_JSON_FORMAT'
  File "profile_service.py", line 284, in fetch_profile
    alert_type=AlertType.INVALID_JSON_FORMAT,
```

Luồng gãy hoàn toàn: `fetch_profile()` raise exception → `asyncio.gather()` fail → pipeline crash → user không nhận được lesson.

### 2. Nguyên nhân

Import chain trên Docker: `send_alert_safe` → `alerts/__init__` → `alert_manager` → `google_chat_transport` → `import aiohttp` → **ImportError** (aiohttp chưa cài trong Docker image).

Fallback `except ImportError` set `AlertType = None`. Nhưng code gọi `AlertType.INVALID_JSON_FORMAT` **không check None** → `AttributeError` → crash.

Vấn đề nằm ở chỗ `send_alert_safe` call (line 284) nằm **ngoài** cả 2 except block (ProfileClientError, Exception) → không ai catch → exception propagate lên pipeline.

```
fetch_profile()
├── try: API call
├── except ProfileClientError → return (OK, có guard)
├── except Exception → return (OK, có guard)
└── parse logic (KHÔNG CÓ try/except bọc)
    └── send_alert_safe(AlertType.INVALID_JSON_FORMAT)  ← CRASH TẠI ĐÂY
        → AlertType = None → AttributeError → propagate lên pipeline
```

### 3. Fix

Thêm guard `if AlertType is not None:` trước tất cả 3 chỗ gọi `send_alert_safe`:

```python
# Line 115 (except ProfileClientError):
if AlertType is not None:
    send_alert_safe(alert_type=AlertType.EXTERNAL_API_ERROR, ...)

# Line 150 (except Exception):
if AlertType is not None:
    send_alert_safe(alert_type=AlertType.EXTERNAL_API_ERROR, ...)

# Line 284 (parse degraded):
if is_degraded and AlertType is not None:
    send_alert_safe(alert_type=AlertType.INVALID_JSON_FORMAT, ...)
```

**Bài học:** Khi import fallback set biến = `None`, mọi chỗ dùng biến đó phải check `is not None`. Lambda no-op cho function thì OK (`send_alert_safe = lambda **kwargs: None`), nhưng enum/class = `None` thì attribute access sẽ crash.

---

## E. FAQ — 3 câu hỏi quan trọng sau khi fix

### Q1: Trường hợp nào gãy luồng không? API gãy 4xx/5xx thì sao?

**Không bao giờ gãy luồng.** `fetch_profile()` luôn return `UserProfile`, không bao giờ raise exception ra ngoài.

```
fetch_profile() — 3 lớp bảo vệ, KHÔNG CÓ ĐƯỜNG NÀO RAISE:
│
├── try: API call (line 94-99)
│   │
│   ├── except ProfileClientError (line 100-132)
│   │   → HTTP 4xx/5xx, timeout, network error
│   │   → return UserProfile(child=None, is_degraded=True)
│   │   → ❌ KHÔNG raise, ❌ KHÔNG crash
│   │
│   └── except Exception (line 134-166)
│       → Bất kỳ lỗi unexpected nào (DNS, SSL, bug trong client)
│       → return UserProfile(child=None, is_degraded=True)
│       → ❌ KHÔNG raise, ❌ KHÔNG crash
│
└── Parse response (line 168-310) — chạy SAU khi API call OK
    → Mỗi field parse riêng, có try/except riêng
    → Field lỗi = None, field OK = giữ nguyên
    → ❌ KHÔNG CÓ ĐƯỜNG NÀO RAISE
```

**Khi API 4xx/5xx cụ thể:**

| HTTP Status | Xảy ra gì | Kết quả |
|-------------|-----------|---------|
| 401/403 | `ProfileClientError` caught (line 100) | `child=None`, `is_degraded=True`, luồng tiếp tục |
| 404 | `ProfileClientError` caught (line 100) | `child=None`, `is_degraded=True`, luồng tiếp tục |
| 429 | `ProfileClientError` caught (line 100) | `child=None`, `is_degraded=True`, luồng tiếp tục |
| 500/502/503 | `ProfileClientError` caught (line 100) | `child=None`, `is_degraded=True`, luồng tiếp tục |
| Timeout | `ProfileClientError` caught (line 100) | `child=None`, `is_degraded=True`, luồng tiếp tục |
| DNS fail | `Exception` caught (line 134) | `child=None`, `is_degraded=True`, luồng tiếp tục |

**Luồng đi tiếp thế nào khi child=None?**

```python
# lesson_pipeline.py — code ĐÃ CÓ SẴN:
if user_profile.child:
    child_age = user_profile.child.age or (parent_config.child_age ...)
    child_name = user_profile.child.child_name or (parent_config.child_name ...)
else:  # ← child=None chạy vào đây
    child_age = parent_config.child_age if parent_config else None
    child_name = parent_config.child_name if parent_config else None
# → Nếu parent_config cũng None → child_age=None → prompt dùng default "6-12 tuổi"
```

---

### Q2: Trường hợp nào fallback mock data?

**KHÔNG CÒN MOCK DATA.** Constant `MOCK_USER_PROFILE` (chứa "Đoàn Cường", age=6) đã bị xóa hoàn toàn.

```
TRƯỚC (cũ):
  API fail → return MOCK_USER_PROFILE  →  child_name="Đoàn Cường", age=6  ❌

SAU (hiện tại):
  API fail → return UserProfile(child=None, is_degraded=True)  ✅
  Parse fail 1 field → return UserProfile(child=ChildProfile(field_lỗi=None), is_degraded=True)  ✅
```

Không còn đường code nào trả tên cứng, tuổi cứng, hay bất kỳ data giả nào.

Verify bằng lệnh:
```bash
rg "MOCK_USER_PROFILE|MOCK DATA|Đoàn Cường|mock_child" app/ --include="*.py"
# → Không tìm thấy kết quả nào
```

---

### Q3: Khi "degraded" thì data như nào?

Có 2 dạng degraded:

**Dạng 1: API fail → child=None hoàn toàn**

```json
{
    "user_id": "019cad43-0f4e-7697-9e69-9ab9ea970ae2",
    "profile_id": "019cad43-0f4e-7697-9e69-9ab9ea970ae2",
    "child": null,
    "language_preference": "vi",
    "raw_data": null,
    "is_degraded": true,
    "degraded_reason": "API error: ProfileClientError: HTTP 404"
}
```
→ Pipeline fallback: dùng `parent_config` hoặc prompt default "6-12 tuổi", không có tên trẻ.

**Dạng 2: API OK, parse 1+ field fail → giữ data thật, field lỗi = None**

Ví dụ: API trả `age="abc"` (không parse được), nhưng `name` và `language_mode` OK:

```json
{
    "user_id": "019cad43-0f4e-7697-9e69-9ab9ea970ae2",
    "profile_id": "019cad43-0f4e-7697-9e69-9ab9ea970ae2",
    "child": {
        "child_id": "019cad43-0f4e-7697-9e69-9ab9ea970ae2",
        "child_name": "Truck-kun",
        "age": null,
        "interests": [],
        "learning_history": ["apple", "banana"]
    },
    "language_preference": "bilingual",
    "raw_data": {"...": "original API response"},
    "is_degraded": true,
    "degraded_reason": "Parse failed for: age"
}
```
→ Pipeline dùng: tên thật "Truck-kun", age=None → prompt default "6-12 tuổi", language=bilingual.
→ **Chỉ mất field lỗi, giữ được tất cả field OK.**

**Dạng 3: Không degraded — tất cả OK**

```json
{
    "user_id": "019cad43-0f4e-7697-9e69-9ab9ea970ae2",
    "child": {
        "child_name": "Truck-kun",
        "age": 8,
        "learning_history": ["apple", "banana"]
    },
    "language_preference": "bilingual",
    "is_degraded": false,
    "degraded_reason": null
}
```

---

*Created by `@doancuong` on `2026-03-18` · Last updated: `2026-03-18` (implementation complete)*
*Naming: `PROB-YYYY-MM-DD-[slug].md` — `PROB-2026-03-18-profile-mock-data-hardcoded`*
