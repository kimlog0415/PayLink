# 프로젝트명: Pay-Link Project
**"내 계좌를 결제창처럼" - 서버 없이 동작하는 스마트 송금 배너 & 브릿지 도구**

## 1. 프로젝트 비전
- **본질**: 복잡한 결제 모듈 없이, 단순 딥링크와 브릿지 페이지를 결합해 '나만의 결제/후원 관문'을 제공.
- **차별점**: 
    - `toss.me`의 빈자리를 채우는 계좌 기반 즉석 링크 생성.
    - 금액(`amount`)을 지정하여 단순 후원뿐만 아니라 **'정산/판매'** 용도로 확장.
    - 개인정보(실명) 수집 없이 익명성을 보장하는 프라이버시 친화적 설계.

## 2. 핵심 기능 (Updated)
### A. 금액 지정 및 메시지 (Payment Context)
- **고정 금액**: 판매나 정산 시 사용자가 금액을 수정하지 못하도록 설정 가능.
- **자율 금액**: 후원 목적일 경우 송금자가 직접 금액을 입력하도록 비워둠.
- **송금 메시지**: "삼겹살 값", "커피 후원" 등 랜딩 페이지에 표시될 메시지 설정.

### B. 프라이버시 우선 설계 (Privacy First)
- **예금주 미수집**: 실명을 입력받지 않고 메시지로 대체. (실명 확인은 송금 앱 내 전산 조회에 맡김)
- **무저장(No-DB)**: 어떤 계좌 정보도 서버에 남기지 않고 오직 암호화된 URL로만 유통.

### C. 하이브리드 대응 (Device Optimized)
- **Mobile**: User Gesture(버튼 클릭) 기반의 자동 딥링크 실행 (`supertoss://` 등).
- **PC**: 스마트폰 촬영을 위한 실시간 QR 코드 생성 및 계좌번호 복사 기능 제공.

## 3. 기술 스택 및 구조
- **Hosting**: GitHub Pages (Public Repository)
- **Core**: HTML5, Vanilla JS, Tailwind CSS
- **Library**:
  - `CryptoJS`: URL 파라미터 암호화 (사용자 계좌번호 노출 방지)
  - `qr-code-styling`: 로고가 포함된 커스텀 QR 코드 생성
- **URL Schema**:
  - `https://[ID].github.io/[Repo]/pay?d=[AES_Encrypted_String]`
  - 암호화 데이터 포함 항목: `bank`, `account`, `amount`, `message`

## 4. 사용자 흐름 (User Flow)
1. **생성(Generator)**: `index.html`에서 계좌 정보와 금액, 메시지를 입력.
2. **포장(Encryption)**: 입력값이 암호화된 고유 URL과 배너 HTML 코드를 발급받음.
3. **공유(Share)**: 블로그, SNS, 카톡 등에 해당 링크/배너를 게시.
4. **결제(Bridge)**: 후원자가 링크 클릭 시 `pay.html` 접속 -> 기기 감지 -> 송금 실행.

## 5. 법적 및 보안 가이드라인
- **면책 조항**: "본 도구는 변환기일 뿐이며 금융 거래의 주체가 아님"을 브릿지 페이지 하단에 명시.
- **난독화**: 최소한 Base64 이상의 암호화를 통해 주소창 내 계좌번호 직관 노출 차단.

## 보안 검토 체크리스트

### ✅ 라이브러리 공식 여부 확인

| 라이브러리 | 공식 PyPI 이름 | 공식 문서 |
|---|---|---|
| Pillow | `Pillow` (대문자 P) | https://pillow.readthedocs.io |
| imagehash | `ImageHash` | https://github.com/JohannesBuchner/imagehash |
| send2trash | `Send2Trash` | https://github.com/arsenetar/send2trash |

- `requirements.txt`에 **버전 고정** 필수 (`==` 사용)
- 추가 라이브러리 도입 시 반드시 PyPI 공식 페이지 확인: https://pypi.org

### ✅ 파일 접근 보안

- 이 앱은 **읽기(Read)** 와 **삭제(Trash 이동)** 만 허용
- 파일 경로는 반드시 `pathlib.Path`로 처리
- 스캔 범위는 사용자가 선택한 폴더 내부로만 제한 (`is_safe_path` 함수)

### ✅ 네트워크 통신 금지

- 완전 로컬 동작 — 네트워크 요청 없음
- `requests`, `urllib`, `http`, `socket` 관련 import 금지

### ✅ 위험 코드 패턴 금지

```python
# ❌ 절대 사용 금지
eval(...)          # 임의 코드 실행
exec(...)          # 임의 코드 실행
os.system(...)     # 셸 명령 실행
__import__(...)    # 동적 import
```

- `subprocess.run`: OS 기본 뷰어/플레이어 열기 용도만 허용 (preview_card.py)

### ✅ 삭제 기능 안전장치

- 파일 삭제는 반드시 `send2trash` 사용 — `os.remove()`, `Path.unlink()` 직접 삭제 금지
- 삭제 전 확인 다이얼로그 필수
- 삭제 실행 전 대상 파일 목록을 콘솔에 로그 출력

### 🚨 코드 작성/수정 후 마무리 절차 — 절대 생략 금지

**코드를 한 줄이라도 작성하거나 수정했다면 반드시 아래 순서를 실행할 것.**
**사용자가 명시적으로 요청하지 않아도 자동으로 실행해야 한다.**

```
🔍 보안 검토 중...
   → grep으로 네트워크 요청 코드 없음 확인 (requests/urllib/socket 등)
   → grep으로 위험 코드 패턴 없음 확인 (eval/exec/os.system/__import__)
   → grep으로 직접 삭제 없음 확인 (os.remove/Path.unlink)
   → subprocess.run 있으면 사유 명시 (OS 뷰어 열기 등 허용된 용도인지 확인)
   → requirements.txt 라이브러리 공식 PyPI 여부 확인

📝 CLAUDE.md 업데이트 중...
   → 변경된 기능/파일 구조 반영
   → 추가된 라이브러리 있으면 목록 갱신
   → 마지막 업데이트 날짜 기록

📦 Git commit 중...
   → git add [변경된 파일들]
   → git commit -m "feat/fix/refactor/docs: [변경 내용 요약]"

🚀 Git push 중...
   → git push origin main

✅ 완료!
```