# lcid-select

브라우저 언어를 자동 감지하여 LCID 값을 반환하는 언어 선택 드롭다운 컴포넌트입니다.

- PC: Select2 기반 검색 가능한 드롭다운
- 모바일: 네이티브 OS 피커
- 브라우저 언어(`navigator.language`) 자동 감지 및 초기 선택
- 120개 이상의 언어/로케일 지원
- 의존성: jQuery + Select2 (PC) / 없음 (모바일)

---

## 파일 구성

```
lcid-select.js       # 소스
lcid-select.min.js   # 압축본 (배포용)
```

---

## 예제

[예제 페이지](https://northkoreanuser.github.io/lcid-select/)

```html
<!-- 1. CDN -->
<link href="https://cdn.jsdelivr.net/npm/select2@4.1.0-rc.0/dist/css/select2.min.css" rel="stylesheet"/>
<script src="https://cdn.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/select2@4.1.0-rc.0/dist/js/select2.min.js"></script>

<!-- 2. 콜백 먼저 정의 -->
<script>
  window.onLcidChange = function(code, lcid, hex) {
    console.log(code, lcid, hex); // 예: ko-KR, 1042, 0x0412
  };
</script>

<!-- 3. select 선언 -->
<select id="lang-select" data-codes="ko-KR,en-US,ja-JP"></select>

<!-- 4. 스크립트 로드 -->
<script src="lcid-select.min.js"></script>
```

> **주의:** `window.onLcidChange`는 반드시 `lcid-select.js` **로드 전에** 정의해야 초기 자동선택 이벤트를 받을 수 있습니다.

---

## `data-*` 속성

| 속성 | 필수 | 설명 | 예시 |
|---|---|---|---|
| `data-codes` | ✅ | 표시할 언어 코드 목록 (콤마 구분, 순서 유지) | `"ko-KR,en-US,ja-JP"` |
| `data-filter` | ❌ | 텍스트 방향 필터 (`LTR` 또는 `RTL`만 표시) | `"LTR"` |
| `data-placeholder` | ❌ | 드롭다운 플레이스홀더 텍스트 | `"Select language..."` |
| `data-width` | ❌ | Select2 드롭다운 너비 (PC 전용) | `"300px"`, `"100%"` |

### 예시

```html
<!-- 기본 -->
<select id="lang-select" data-codes="ko-KR,en-US,ja-JP"></select>

<!-- LTR 언어만 표시 -->
<select id="lang-select" data-codes="ko-KR,en-US,ja-JP,ar-SA" data-filter="LTR"></select>

<!-- RTL 언어만 표시 -->
<select id="lang-select" data-codes="ar-SA,he-IL,fa-IR" data-filter="RTL"></select>

<!-- 플레이스홀더 + 너비 지정 -->
<select id="lang-select"
  data-codes="ko-KR,en-US,en-GB,ja-JP,zh-CN,fr-FR,de-DE"
  data-placeholder="Select language..."
  data-width="300px">
</select>
```

---

## 이벤트 수신 방법

### 방법 1: `window.onLcidChange` 콜백

```html
<script>
  window.onLcidChange = function(code, lcid, hex) {
    console.log(code);  // "ko-KR"
    console.log(lcid);  // 1042
    console.log(hex);   // "0x0412"
  };
</script>
<select id="lang-select" data-codes="ko-KR,en-US,ja-JP"></select>
<script src="lcid-select.min.js"></script>
```

### 방법 2: `lcid-change` 커스텀 이벤트

```html
<select id="lang-select" data-codes="ko-KR,en-US,ja-JP"></select>
<script src="lcid-select.min.js"></script>
<script>
  document.getElementById('lang-select').addEventListener('lcid-change', function(e) {
    const { code, lcid, hex } = e.detail;
    console.log(code, lcid, hex);
  });
</script>
```

> 커스텀 이벤트는 `bubbles: true`로 발생하므로 상위 요소에서도 수신 가능합니다.

```js
document.addEventListener('lcid-change', function(e) {
  console.log(e.detail); // { code, lcid, hex }
});
```

---

## 자동 언어 감지

스크립트 로드 시 `navigator.language`를 읽어 `data-codes` 후보 중 가장 적합한 언어를 자동 선택합니다.

| 브라우저 언어 | data-codes | 자동 선택 결과 |
|---|---|---|
| `ko` | `ko-KR,en-US,ja-JP` | `ko-KR` (기본형 매칭) |
| `ko-KR` | `ko-KR,en-US,ja-JP` | `ko-KR` (정확 매칭) |
| `zh-TW` | `ko-KR,en-US,zh-CN` | `zh-CN` (기본형 `zh` 매칭) |
| `fr` | `ko-KR,en-US,ja-JP` | `ko-KR` (매칭 없음 → 첫 번째) |

**매칭 우선순위:**
1. `navigator.language`가 `data-codes`에 정확히 있는 경우
2. `navigator.language`의 기본 언어코드(`ko-KR` → `ko`)가 후보 기본형과 일치하는 경우
3. 매칭 없으면 `data-codes`의 첫 번째 항목

---

## PC / 모바일 동작 차이

| 환경 | 감지 기준 | 동작 |
|---|---|---|
| PC | `window.matchMedia('(pointer: coarse)')` = false | Select2 검색 가능한 드롭다운 |
| 모바일 | `window.matchMedia('(pointer: coarse)')` = true | 네이티브 OS 피커 (jQuery/Select2 불필요) |

모바일에서는 jQuery와 Select2가 없어도 정상 동작합니다.

---

## 지원 언어 목록 (일부)

| 코드 | LCID | 언어 | 방향 |
|---|---|---|---|
| `ko-KR` | 1042 (0x0412) | Korean (South Korea) | LTR |
| `en-US` | 1033 (0x0409) | English (United States) | LTR |
| `en-GB` | 2057 (0x0809) | English (United Kingdom) | LTR |
| `ja-JP` | 1041 (0x0411) | Japanese (Japan) | LTR |
| `zh-CN` | 2052 (0x0804) | Chinese Simplified (China) | LTR |
| `zh-TW` | 1028 (0x0404) | Chinese Traditional (Taiwan) | LTR |
| `ar-SA` | 1025 (0x0401) | Arabic (Saudi Arabia) | RTL |
| `he-IL` | 1037 (0x040D) | Hebrew (Israel) | RTL |
| `fr-FR` | 1036 (0x040C) | French (France) | LTR |
| `de-DE` | 1031 (0x0407) | German (Germany) | LTR |
| `es-ES` | 3082 (0x0C0A) | Spanish (Spain) | LTR |
| `ru-RU` | 1049 (0x0419) | Russian (Russia) | LTR |

전체 목록은 소스 코드의 `LOCALES` 배열을 참고하세요.

---

## 브라우저 지원

| 브라우저 | 지원 여부 |
|---|---|
| Chrome 60+ | ✅ |
| Firefox 55+ | ✅ |
| Safari 12+ | ✅ |
| Edge 79+ | ✅ |
| IE 11 | ❌ (CustomEvent 미지원) |

---

## 라이선스

[License](https://github.com/northkoreanuser/lcid-select/blob/main/License)
