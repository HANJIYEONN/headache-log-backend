# 두통 기록 차트 — Backend (FastAPI)

두통이 올 때마다 투약 사항과 상태를 기록하는 개인 기록 앱의 백엔드 API.
(프론트엔드: [headache-log-frontend](https://github.com/HANJIYEONN/headache-log-frontend) / 원본 모노레포: [headache-log](https://github.com/HANJIYEONN/headache-log))

## 스택

- **FastAPI** (Python) + **SQLAlchemy**
- **DB**: MySQL — 로컬 개발은 내 컴퓨터 MySQL, 배포는 **TiDB Cloud** (MySQL 호환)
- **인증**: Google 로그인 (ID 토큰 검증) + 자체 JWT 발급 (유효기간 7일)
- **배포**: GitHub Actions → SSH → 오라클 클라우드 VM에서 Docker 실행 ✅ 가동 중

## 지금까지 만든 기능 (2026-07-14 ~ 07-18)

| 날짜 | 내용 |
|---|---|
| 07-14 | 프로젝트 시작: FastAPI + SQLAlchemy + 기록(entry) CRUD API |
| 07-15 | 약 종류(medication) 컬럼 추가, 기록 수정(PUT) |
| 07-16 | Google 로그인: ID 토큰 검증 → 자체 JWT 발급, 기록 API 로그인 필수 + 사용자별 분리 |
| 07-17 | 자주 복용하는 약(즐겨찾기, 최대 3개): 폼 내용 전체 저장, 수정/삭제 API |
| 07-17 | DB를 PostgreSQL → MySQL로 전환 (로컬 MySQL + 배포 TiDB Cloud) |
| 07-18 | 모노레포에서 백엔드 분리, GitHub Actions 자동배포 성공 (오라클 VM + Docker) |

## API 요약

| 메서드 | 경로 | 설명 |
|---|---|---|
| POST | `/api/v1/auth/google` | 구글 ID 토큰 검증 → 자체 JWT 발급 |
| GET/POST | `/entries` | 내 기록 목록 / 새 기록 (로그인 필수) |
| GET/PUT/DELETE | `/entries/{id}` | 기록 조회/수정/삭제 |
| GET/POST | `/favorites` | 자주 복용하는 약 목록 / 추가 (최대 3개) |
| PUT/DELETE | `/favorites/{id}` | 즐겨찾기 수정 / 삭제 |
| GET | `/health` | 서버 상태 확인 |

기록 항목: 날짜 · 약 종류 · 복용횟수 · 효과여부 · 촉발요인 · 생리기간 · 혈압(수축기/이완기/맥박)

## 로컬 실행

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
cp .env.example .env   # 값 채우기
.venv/bin/uvicorn app.main:app --reload   # http://localhost:8000
```

## 배포 (GitHub Actions 자동배포)

`main`에 푸시하면 `.github/workflows/deploy.yaml`이 실행돼요:

```
main 푸시 → SSH로 오라클 VM 접속 → 이 저장소 pull → Secret으로 .env 생성
→ Docker build & run → 헬스체크(최대 5분) → 완료
```

**필요한 Secrets** (Settings → Secrets → Actions):
`OCI_HOST` · `OCI_USERNAME` · `SSH_PRIVATE_KEY` · `ENV_FILE_CONTENT`

비밀값(DB 접속 정보, JWT 키 등)은 코드에 없고 전부 `.env`/Secrets로만 관리해요.
