# AI 기반 문서 추출·요약·카테고리 분류 시스템

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-Backend-009688?logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-4169E1?logo=postgresql&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-ORM-D71F00?logo=sqlalchemy&logoColor=white)
![React](https://img.shields.io/badge/React-Frontend-61DAFB?logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-Build-646CFF?logo=vite&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-LLM-000000?logo=ollama&logoColor=white)
![PaddleOCR](https://img.shields.io/badge/PaddleOCR-OCR-FF6F00)
![PaddlePaddle](https://img.shields.io/badge/PaddlePaddle-Framework-0062B0)

비정형 문서(`.hwp`, `.hwpx`, `.pdf`, `.docx`, `.ppt`, `.pptx`)를 업로드하면 텍스트 추출, OCR 보완, 요약 및 카테고리 분류, PostgreSQL 저장까지 자동으로 처리하는 웹 애플리케이션입니다.

이 프로젝트는 FastAPI 백엔드와 React 프런트엔드로 구성되어 있으며, 문서 처리 결과를 비동기 작업 단위로 추적할 수 있습니다.

---

## 주요 기능

- 문서 업로드 및 비동기 처리
- 포맷별 텍스트 추출
- OCR 기반 텍스트 보완
- Ollama 기반 요약 및 카테고리 분류
- PostgreSQL 이력 저장 및 검색
- React UI를 통한 진행 상태 및 결과 확인

---

## 지원 문서 형식

- `.hwp`
- `.hwpx`
- `.pdf`
- `.docx`
- `.ppt`
- `.pptx`

---

## 처리 흐름

```text
파일 업로드
  -> 파일 저장
  -> 확장자별 추출기 실행
  -> 텍스트/JSON 정리
  -> Ollama 요약 및 카테고리 분류
  -> PostgreSQL 저장
  -> 프런트엔드에서 상태 조회 및 결과 확인
```

포맷별 처리 방식은 대략 다음과 같습니다.

- `PDF`: 내장 텍스트 추출 후 부족한 페이지는 OCR로 보완
- `DOCX`: 문단/표 직접 추출 후 포함 이미지 OCR 수행
- `HWPX`: XML 기반 직접 파싱
- `HWP`: 직접 텍스트 추출 시도 후 필요 시 COM 변환 및 OCR fallback
- `PPT/PPTX`: 슬라이드 텍스트 추출, 일부 이미지 OCR 보조

---

## 기술 스택

| 영역 | 사용 기술 |
|---|---|
| Backend | FastAPI, Uvicorn |
| DB | PostgreSQL, SQLAlchemy |
| LLM | Ollama, httpx |
| OCR | PaddleOCR, PaddlePaddle |
| 문서 처리 | pdfplumber, pdf2image, PyMuPDF, python-docx, python-pptx, olefile |
| Frontend | React, Vite, Axios |

---

## 프로젝트 구조

```text
rag-project-final/
├─ main.py                  # FastAPI 진입점 및 API 라우트
├─ database.py              # SQLAlchemy 엔진/세션 생성
├─ models.py                # Category, Document, Job 테이블 정의
├─ document_pipeline.py     # 확장자별 extractor 라우팅
├─ llm_chain.py             # Ollama 호출 및 요약/분류 결과 파싱
├─ config.py                # 추출기 공통 설정
├─ app/
│  ├─ config.py             # 애플리케이션 환경설정
│  └─ schemas.py            # API 응답 스키마
├─ extractors/
│  ├─ pdf_ext/              # PDF 추출기
│  ├─ docx_ext/             # DOCX 추출기
│  ├─ hwp_ext/              # HWP/HWPX 추출기
│  └─ ppt_ext/              # PPT/PPTX 추출기
├─ frontend/
│  ├─ src/App.jsx           # 업로드/상태/결과 UI
│  ├─ src/api.js            # 백엔드 API 호출
│  └─ package.json
├─ data/
│  ├─ uploads/              # 업로드 원본 저장 경로
│  ├─ work/                 # 작업용 임시 경로
│  └─ ocr_output/           # 추출 JSON 저장 경로
└─ output/                  # 일부 추출기의 중간 산출물 경로
```

---

## 실행 전 준비사항

다음 구성 요소가 필요합니다.

- Python 3.10 이상
- PostgreSQL
- Ollama
- Node.js 및 npm

문서 형식에 따라 추가 의존성이 필요할 수 있습니다.

- `PDF OCR`: Poppler 필요
- `HWP 처리`: Windows 환경에서 Hancom Office/COM 기반 동작이 필요할 수 있음

---

## 환경변수

`.env.example`를 복사해 `.env`를 만든 뒤 값을 설정합니다.

```bash
# Windows PowerShell
copy .env.example .env
```

```bash
# macOS/Linux
cp .env.example .env
```

주요 환경변수는 아래와 같습니다.

| 변수명 | 설명 | 예시 |
|---|---|---|
| `DATABASE_URL` | PostgreSQL 연결 문자열 | `postgresql+psycopg2://postgres:YOUR_PASSWORD@localhost:5432/document_db` |
| `APP_NAME` | FastAPI 앱 이름 | `Document Analyzer` |
| `UPLOAD_DIR` | 업로드 파일 저장 경로 | `./data/uploads` |
| `WORK_DIR` | 작업용 임시 경로 | `./data/work` |
| `OCR_OUTPUT_DIR` | 추출 JSON 저장 경로 | `./data/ocr_output` |
| `OLLAMA_URL` | Ollama 서버 주소 | `http://localhost:11434` |
| `OLLAMA_MODEL` | 사용할 Ollama 모델명 | `llama3.1:8b` |
| `OLLAMA_TIMEOUT_SEC` | Ollama 요청 타임아웃 | `300` |

주의:

- 현재 구조상 `DATABASE_URL`이 없으면 서버 시작 시점에 실패합니다.
- `.env.example`는 예시 파일이고, 실제 실행에는 `.env`가 필요합니다.

---

## 백엔드 실행

### 1. 가상환경 생성 및 패키지 설치

```bash
python -m venv .venv
```

```bash
# Windows PowerShell
.venv\Scripts\activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

```bash
# macOS/Linux
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### 2. Ollama 모델 준비

```bash
ollama pull llama3.1:8b
```

`OLLAMA_MODEL`을 다른 값으로 바꿨다면 해당 모델을 미리 받아둬야 합니다.

### 3. Poppler 설치 (Windows, PDF OCR 사용 시 필수)

이 프로젝트의 PDF OCR 경로는 `pdf2image`를 사용하므로, Windows에서는 Poppler가 설치되어 있어야 합니다.

설치 순서:

1. Windows용 Poppler 압축 파일을 다운로드
2. 원하는 위치에 압축 해제
3. 압축 해제한 폴더의 `Library/bin` 경로를 시스템 PATH에 추가
4. 새 터미널을 열고 아래 명령으로 설치 여부 확인

예시 경로:

```text
C:\Users\<사용자명>\Downloads\poppler-xx\Library\bin
```

확인 명령:

```bash
where.exe pdfinfo
where.exe pdftoppm
python -c "import shutil; print(shutil.which('pdfinfo'))"
```

위 명령에서 경로가 정상적으로 출력되면 준비가 된 상태입니다.

### 4. 서버 실행

```bash
uvicorn main:app --reload --port 8000
```

헬스체크:

- `GET http://localhost:8000/health`

---

## 프런트엔드 실행

```bash
cd frontend
npm install
npm run dev
```

기본 개발 서버 주소:

- [http://localhost:5173](http://localhost:5173)

백엔드 기본 주소:

- [http://localhost:8000](http://localhost:8000)

---

## API 요약

### 1. 처리 시작

- `POST /api/process/start`
- `multipart/form-data`
- 필드명: `file`

응답 예시:

```json
{
  "job_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

### 2. 작업 상태 조회

- `GET /api/process/{job_id}`

응답 예시:

```json
{
  "job_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "status": "completed",
  "progress": 100,
  "stage": "completed",
  "message": "처리 완료",
  "result": {
    "filename": "sample.pdf",
    "raw_text": "...",
    "ocr_text": "",
    "merged_text": "...",
    "summary": "...",
    "category": "기술",
    "main_category": "기술",
    "sub_category": "AI",
    "confidence": 0.0,
    "category_reason": ""
  }
}
```

### 3. 처리 이력 조회

- `GET /api/history`

쿼리 파라미터:

- `limit`: 기본 `50`, 최대 `200`

### 4. 처리 이력 검색

- `GET /api/history/search`

쿼리 파라미터:

- `q`: 검색어
- `limit`: 기본 `50`, 최대 `200`

---

## 트러블슈팅

### 1. `DATABASE_URL is required in environment (.env)`

원인:

- `.env` 파일이 없거나 `DATABASE_URL`이 비어 있음

해결:

1. `.env.example`를 `.env`로 복사
2. `DATABASE_URL` 값을 실제 PostgreSQL 환경에 맞게 수정
3. PostgreSQL 실행 여부 확인

### 2. Ollama 모델 관련 오류

예:

- 모델을 찾을 수 없음
- Ollama 엔드포인트 연결 실패

해결:

1. Ollama 서버 실행 여부 확인
2. `.env`의 `OLLAMA_URL` 확인
3. `.env`의 `OLLAMA_MODEL` 확인
4. `ollama pull <모델명>` 실행

### 3. PDF OCR 오류

예:

- `Unable to get page count`
- `pdfinfo not installed`

원인:

- Poppler가 설치되지 않았거나 PATH에 없음

해결:

1. Poppler 설치
2. `Library/bin` 경로를 PATH에 추가
3. 새 터미널에서 `where.exe pdfinfo`로 확인

### 4. HWP 처리 실패

원인:

- Hancom Office COM 자동화 환경 문제
- 변환 타임아웃
- 파일 형식/인코딩 이슈

해결:

1. Windows 환경에서 한글 프로그램 설치 상태 확인
2. HWP 파일이 정상적으로 열리는지 수동 확인
3. 필요 시 OCR fallback 경로 사용 여부 로그 확인

---

## 주의사항

- 작업 상태 추적은 메모리 기반이므로 서버 재시작 시 진행 중인 `job_id` 상태는 유지되지 않습니다.
- 이력 조회는 PostgreSQL 저장 결과를 기준으로 합니다.
- 일부 추출기는 중간 산출물을 `output/`에 저장하고, API 파이프라인용 JSON은 `data/ocr_output/`에 저장합니다.
- CORS는 현재 전체 허용으로 설정되어 있어 운영 환경에서는 별도 제한이 필요합니다.

---

## 개선 아이디어

- 비동기 작업 큐 도입
- 작업 상태를 DB에도 저장하도록 확장
- 프런트엔드 이력 UI 개선
- 추출기별 출력 경로 일관성 정리
- 예외 메시지 및 운영 로그 개선
- 테스트 코드 추가

---

## 라이선스 / 배포 메모

외부 OCR, PDF, HWP 처리 도구 및 모델 의존성이 있으므로 실제 배포 전에는 다음 항목을 별도로 점검하는 것을 권장합니다.

- 서버 OS별 의존성 설치 여부
- PostgreSQL 연결 정보 보안 관리
- `.env` 및 업로드 파일의 Git 제외 처리
- 운영 환경용 CORS 및 로그 정책
