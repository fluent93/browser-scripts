# SAP SuccessFactors LMS 강의 진행바 제한 해제 가이드

> **환경**: SAP SuccessFactors 기반 사내 LMS  
> **목적**: 진행바(progress bar) 앞뒤 이동 제한 해제 및 빠른 수강 완료 처리

---

## 사전 준비

1. 강의 페이지 열기
2. `F12` → **Console** 탭 클릭
3. 처음 붙여넣기 시 경고 뜨면 `allow pasting` 직접 타이핑 후 Enter

---

## 방법 1: 단계별 검증 버전 (권장)

### Step 1 — 16배속 설정

강의 **Play 누른 후** Console에 입력:

```javascript
document.querySelectorAll('iframe')[0].contentDocument.querySelector('video').playbackRate = 16;
```

영상이 끝날 때까지 대기 (30분짜리 → 약 2분)

---

### Step 2 — 수강 완료 SCORM 저장

영상이 **끝난 직후** Console에 입력:

```javascript
var api = window.API_1484_11 || window.API;
if (api) {
    api.SetValue('cmi.completion_status', 'completed');
    api.SetValue('cmi.success_status', 'passed');
    api.SetValue('cmi.progress_measure', '1');
    api.SetValue('cmi.session_time', 'PT30M59S'); // 영상 길이에 맞게 수정
    api.Commit('');
    console.log('저장 완료!');
}
```

> **PT 시간 형식**: `PT시간H분M초S`
> | 영상 길이 | session_time |
> |-----------|-------------|
> | 14분 15초 | `PT14M15S` |
> | 30분 59초 | `PT30M59S` |
> | 1시간 5분 | `PT1H5M0S` |

---

### Step 3 — 완료 확인

`F5`로 페이지 새로고침 후 수강 완료 여부 확인

---

## 방법 2: 한 줄 자동화 버전 (PT 시간 자동 계산)

강의 **Play 누른 후** Console에 아래 한 줄 입력:

```javascript
(function(){var v=document.querySelectorAll('iframe')[0].contentDocument.querySelector('video');v.playbackRate=16;v.addEventListener('ended',function(){var api=window.API_1484_11||window.API;if(api){var h=Math.floor(v.duration/3600),m=Math.floor((v.duration%3600)/60),s=Math.floor(v.duration%60);api.SetValue('cmi.completion_status','completed');api.SetValue('cmi.success_status','passed');api.SetValue('cmi.progress_measure','1');api.SetValue('cmi.session_time','PT'+(h?h+'H':'')+m+'M'+s+'S');api.Commit('');alert('수강 완료! 새로고침하세요.');}}); console.log('설정 완료!');})();
```

- 16배속 자동 설정
- 영상 종료 시 PT 시간 자동 계산 후 SCORM 저장
- 완료되면 `"수강 완료! 새로고침하세요."` 팝업 자동으로 뜸

---

## 참고: 플랫폼 구조

| 항목 | 내용 |
|------|------|
| LMS | SAP SuccessFactors |
| 영상 위치 | `iframe[0]` 내부 HTML5 `<video>` 태그 |
| 완료 추적 방식 | SCORM 2004 (`API_1484_11`) + `ScormRteServlet` |
| 진행 제한 방식 | 플레이어 내부 `timeupdate` 이벤트로 seek 차단 |

---

## 주의사항

- 퀴즈가 포함된 과정은 별도 처리 필요
- 페이지가 사라지면 정상 (SCORM 종료 처리된 것) → 홈에서 완료 확인
- SCORM API가 없는 모듈은 16배속만으로도 완료 처리됨
