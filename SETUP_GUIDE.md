# 🤖 오코리믹 - AI 자동화 블로그 시스템

## 📊 프로젝트 개요

**오코리믹(okonomis.com)**은 자동차 경제 전문매체로, AI 기반 자동화 시스템을 통해 운영됩니다.

### 🏗️ 시스템 아키텍처

```
n8n 스케줄러 → GitHub Actions 웹훅 → AI 스크래퍼 실행 → 
뉴스 수집 → AI 재작성 → Cloudflare 이미지 업로드 → 
마크다운 생성 → Git 커밋/푸시 → Cloudflare Pages 자동 배포
```

### 🎯 주요 기능
- **자동 뉴스 수집**: 리포테라 사이트맵에서 최신 기사 자동 수집
- **AI 콘텐츠 재작성**: GPT-4o-mini로 독창적인 콘텐츠 생성
- **이미지 최적화**: Cloudflare Images로 자동 압축 및 CDN 최적화
- **SEO 최적화**: 자동 태그 생성, 사이트맵, 구조화된 데이터
- **Hugo 정적 사이트**: 빠른 로딩 속도와 높은 성능
- **이메일 알림**: 처리 결과 자동 보고

## 🔧 필수 설정 및 API 키

### 1. OpenAI API 키 (필수) 🤖
- **용도**: AI 기사 재작성, SEO 태그 자동 생성
- **모델**: GPT-4o-mini (빠르고 경제적)
- **획득**: [OpenAI Platform](https://platform.openai.com/api-keys)
- **비용**: 기사당 약 0.2원 (매우 저렴!)

### 2. Cloudflare API 설정 (선택사항) ☁️
- **Cloudflare API Token**: 이미지 자동 업로드 및 최적화
- **Cloudflare Account ID**: 계정 식별용
- **획득 방법**:
  1. [Cloudflare Dashboard](https://dash.cloudflare.com) → My Profile → API Tokens
  2. "Create Token" → "Cloudflare Images" 템플릿
  3. Account 및 Zone 권한 부여

### 3. GitHub 인증 🔐
- **PAT Token**: n8n 웹훅 및 자동 커밋용
- **권한**: `repo`, `workflow`, `write:packages`
- **획득**: GitHub Settings → Developer settings → Personal access tokens

### 4. 이메일 알림 설정 (선택사항) 📧
- **SENDER_EMAIL**: Gmail 발신자 계정
- **SENDER_PASSWORD**: Gmail 앱 비밀번호
- **RECIPIENT_EMAIL**: 알림 수신 이메일

## 🔐 GitHub Repository Secrets 설정

Repository → Settings → Secrets and variables → Actions → New repository secret

| Secret Name | 설명 | 필수 여부 | 비고 |
|------------|------|----------|------|
| `OPENAI_API_KEY` | OpenAI API 키 | ✅ **필수** | GPT-4o-mini 사용 |
| `CLOUDFLARE_API_TOKEN` | Cloudflare Images API 토큰 | 🔶 선택 | 이미지 최적화용 |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare 계정 ID | 🔶 선택 | 이미지 업로드용 |
| `PAT_TOKEN` | GitHub Personal Access Token | ✅ **필수** | 자동 커밋용 |
| `SENDER_EMAIL` | Gmail 발신자 이메일 | 🔶 선택 | 알림 이메일용 |
| `SENDER_PASSWORD` | Gmail 앱 비밀번호 | 🔶 선택 | 알림 이메일용 |
| `RECIPIENT_EMAIL` | 알림 수신 이메일 | 🔶 선택 | 처리 결과 알림 |

## 📁 프로젝트 파일 구조

```
blogai2/
├── 📄 ai_scraper.py           # 메인 AI 스크래퍼 (뉴스 수집 + AI 재작성)
├── 📄 send_email.py           # 이메일 알림 시스템
├── 📄 config.yaml             # Hugo 사이트 설정
├── 📄 requirements.txt        # Python 의존성
├── 📁 .github/workflows/
│   └── auto-scraper.yml       # GitHub Actions 워크플로우
├── 📁 content/
│   ├── automotive/            # 자동차 카테고리 기사들
│   └── authors/               # 작성자 정보
├── 📁 layouts/                # Hugo 템플릿
├── 📁 static/                 # 정적 파일 (CSS, JS, 이미지)
├── 📄 processed_articles.db   # SQLite DB (중복 방지용)
└── 📄 SETUP_GUIDE.md         # 이 파일
```

## 🤖 n8n 자동화 워크플로우

### 📊 워크플로우 JSON (복사해서 n8n에 Import)

```json
{
  "meta": {
    "instanceId": "blogai-automation"
  },
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [{
            "field": "hours",
            "hoursInterval": 8
          }]
        }
      },
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.2,
      "position": [420, 300],
      "id": "schedule-trigger",
      "name": "⏰ 8시간마다 자동 실행"
    },
    {
      "parameters": {
        "url": "https://api.github.com/repos/YOUR_USERNAME/blogai2/dispatches",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "event_type",
              "value": "scrape-content"
            },
            {
              "name": "client_payload",
              "value": "={{ { \"sitemap_url\": \"https://www.reportera.co.kr/news-sitemap.xml\", \"triggered_by\": \"n8n-schedule\", \"timestamp\": $now.toISO() } }}"
            }
          ]
        }
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [640, 300],
      "id": "github-trigger",
      "name": "🚀 GitHub Actions 웹훅 트리거",
      "credentials": {
        "httpHeaderAuth": {
          "id": "github-auth",
          "name": "GitHub Authorization"
        }
      }
    }
  ],
  "connections": {
    "⏰ 8시간마다 자동 실행": {
      "main": [[
        {
          "node": "🚀 GitHub Actions 웹훅 트리거",
          "type": "main",
          "index": 0
        }
      ]]
    }
  }
}
```

### 🔐 n8n 인증 설정
1. **Credentials → Add credential → HTTP Header Auth**
2. **Name**: `Authorization`
3. **Value**: `Bearer YOUR_GITHUB_PAT_TOKEN`
4. **저장 후 워크플로우에서 선택**

## 🚀 시스템 테스트 및 모니터링

### 1. 수동 테스트 방법
- **GitHub Actions**: Repository → Actions → "Auto Content Scraper" → "Run workflow"
- **n8n**: 워크플로우에서 "Test workflow" 클릭
- **로컬 테스트**: `python ai_scraper.py` (환경변수 설정 필요)

### 2. 실시간 모니터링
- **📊 GitHub Actions**: 실행 로그, 성공/실패 상태
- **📈 n8n Dashboard**: 실행 기록, 성공률 통계
- **☁️ Cloudflare Pages**: 자동 배포 상태, 빌드 로그
- **📧 이메일 알림**: 처리 완료 및 에러 보고

### 3. 성능 지표
- **처리 속도**: 기사당 평균 30초
- **성공률**: 일반적으로 95% 이상
- **비용 효율성**: 일일 기사 20개 기준 약 5원

## ⚙️ 시스템 커스터마이징

### 📅 스케줄 조정
n8n Schedule Trigger에서 설정 변경:
```json
// 2시간마다
{"field": "hours", "hoursInterval": 2}

// 매일 오전 9시 (크론 표현식)
{"field": "cronExpression", "cronExpression": "0 9 * * *"}

// 30분마다 (고빈도)
{"field": "minutes", "minutesInterval": 30}
```

### 📊 처리 기사 수 조정
`ai_scraper.py` 파일 수정:
```python
# 라인 144 근처
urls = urls[:20]  # 20개 → 원하는 숫자로 변경
# 권장: 10-30개 (너무 많으면 GitHub Actions 타임아웃)
```

### 🤖 AI 모델 변경
GitHub Secrets에서 `OPENAI_MODEL` 설정:
- **gpt-4o-mini**: 현재 설정 (빠르고 경제적) ✅
- **gpt-3.5-turbo**: Legacy 모델 (더 저렴)
- **gpt-4**: 고품질 (더 비쌈)

### 🎯 타겟 사이트 변경
GitHub Actions 워크플로우에서 `SITEMAP_URL` 수정:
```yaml
env:
  SITEMAP_URL: 'https://다른사이트.com/sitemap.xml'
```

## 💰 운영 비용 분석

### 🤖 OpenAI API (GPT-4o-mini)
| 항목 | 토큰 사용량 | 단가 | 기사당 비용 |
|------|------------|------|------------|
| **기사 재작성** | ~1,000 토큰 | $0.00015/1K | 약 0.15원 |
| **SEO 태그 생성** | ~100 토큰 | $0.00015/1K | 약 0.015원 |
| **이메일 요약** | ~200 토큰 | $0.00015/1K | 약 0.03원 |
| **합계** | - | - | **기사당 약 0.2원** |

### ☁️ Cloudflare Services
- **Images**: 월 100,000건 무료 → 초과시 $5/월
- **Pages**: 무료 (무제한 빌드)
- **CDN**: 무료 (전 세계 배포)

### 📊 월간 예상 비용 (일 20기사 기준)
- **최소 구성** (OpenAI만): **월 120원** 💸
- **풀 구성** (AI + Cloudflare): **월 5,120원**
- **GitHub Actions**: 무료 (월 2,000분 제한)
- **n8n**: 무료 (자체 호스팅) 또는 €20/월 (클라우드)

### 💡 비용 최적화 팁
1. **배치 크기 조정**: 기사 수를 줄여 비용 절약
2. **스케줄 조정**: 실행 빈도를 줄여 API 호출 최소화
3. **무료 서비스 활용**: GitHub Actions, Cloudflare Pages 무료 한도 활용

## 🔍 트러블슈팅 가이드

### 🚨 일반적인 문제 및 해결방법

#### 1. AI 재작성 실패
- **증상**: "OpenAI API Error" 메시지
- **원인**: API 키 오류, 크레딧 부족, 모델 접근 권한
- **해결**:
  ```bash
  # API 키 확인
  echo $OPENAI_API_KEY
  
  # 크레딧 잔액 확인 (OpenAI Dashboard)
  # 새 API 키 재발급 (필요시)
  ```

#### 2. 이미지 업로드 실패
- **증상**: "Cloudflare API Error" 또는 이미지 누락
- **원인**: API 토큰 권한, Account ID 오류
- **해결**:
  ```bash
  # Cloudflare 설정 확인
  curl -X GET "https://api.cloudflare.com/client/v4/accounts" \
    -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"
  ```

#### 3. GitHub Actions 실패
- **증상**: 워크플로우 빨간색 상태
- **원인**: PAT 토큰 권한, 의존성 설치 실패
- **해결**:
  - Repository Settings → Secrets 확인
  - PAT 토큰 권한: `repo`, `workflow` 체크
  - Actions 탭에서 상세 로그 확인

#### 4. n8n 웹훅 실패
- **증상**: "HTTP 401 Unauthorized"
- **원인**: GitHub PAT 토큰 만료
- **해결**:
  - GitHub에서 새 PAT 토큰 생성
  - n8n Credentials에서 토큰 업데이트

### 📊 로그 확인 체크리스트
- [ ] **n8n 실행 기록**: 웹훅 전송 성공 여부
- [ ] **GitHub Actions**: 각 단계별 성공/실패 상태  
- [ ] **AI 품질 검토**: 재작성된 기사 내용 확인
- [ ] **이미지 최적화**: Cloudflare Images 업로드 상태
- [ ] **사이트 배포**: Cloudflare Pages 빌드 로그
- [ ] **이메일 알림**: 처리 완료 보고서 수신

## 🎉 시스템 완료 및 자동화 흐름

### ✅ 설정 완료 체크리스트
- [ ] **GitHub Secrets** 모든 API 키 설정 완료
- [ ] **n8n 워크플로우** 임포트 및 활성화
- [ ] **첫 번째 수동 테스트** 성공적으로 실행
- [ ] **이메일 알림** 정상 수신 확인
- [ ] **사이트 배포** Cloudflare Pages 연동 완료

### 🔄 완전 자동화된 프로세스

시스템이 **8시간마다 자동으로**:

1. 🕐 **n8n 스케줄러 실행**
2. 🔗 **GitHub Actions 웹훅 트리거**
3. 📰 **리포테라에서 최신 뉴스 수집**
4. 🤖 **GPT-4o-mini로 독창적 콘텐츠 재작성**
5. 🖼️ **Cloudflare Images로 이미지 최적화**
6. 🏷️ **SEO 태그 및 메타데이터 자동 생성**
7. 📝 **Hugo 마크다운 파일 생성**
8. 💾 **Git 커밋 및 푸시**
9. 🚀 **Cloudflare Pages 자동 배포**
10. 📧 **이메일로 처리 결과 보고**

### 🌟 최종 결과

**완전 무인 자동화 블로그 시스템 완성!** 

- ⚡ **고성능**: Hugo 정적 사이트 + Cloudflare CDN
- 💰 **초저비용**: 월 120원부터 (AI 비용)
- 🎯 **SEO 최적화**: 자동 태그, 사이트맵, 구조화 데이터
- 📱 **반응형**: 모바일 최적화 완료
- 🔒 **안전함**: HTTPS, 보안 헤더 적용
- 📊 **모니터링**: 실시간 상태 확인 및 알림

---

### 🆘 지원 및 문의

- **이메일**: hangil9910@gmail.com
- **GitHub**: 이슈 등록을 통한 기술 지원
- **업데이트**: 지속적인 시스템 개선 및 기능 추가

**오코리믹 AI 자동화 시스템을 이용해 주셔서 감사합니다!** 🚗⚡ 