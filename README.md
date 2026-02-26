# timedeal-platform-docs
# 🛒 동네콕 (DongneKok) - 하이퍼로컬 타임딜 커머스 플랫폼

> **💡 안내 사항 (Notice)**
> 본 프로젝트는 단순한 토이 프로젝트를 넘어, 실제 로컬 상권 파트너십(MOU) 및 상용화 수준의 비즈니스 로직을 포함하여 개발되었기에 핵심 비즈니스 로직이 담긴 **소스 코드는 비공개(Private)** 처리하였습니다.
> 대신 본 문서를 통해 프로젝트의 **아키텍처 설계, 보안/인프라 구축 과정, 트러블슈팅 경험 및 협업 시스템(이슈 템플릿 등)**을 상세히 공개합니다. 상세한 기술 블로그 기록도 함께 참고 부탁드립니다.

<br>

## 🎯 1. Project Overview
오프라인 베이커리의 재고 관리 비효율을 해결하기 위해 기획된 지역 기반 타임딜 플랫폼입니다. 
초기 팀 프로젝트로 시작하여 기획과 백엔드를 전담하였으며, 현재는 **1인 리팩토링**을 통해 기존 레거시 인프라를 전면 폐기하고 **Cloudflare Zero Trust와 AWS Serverless 기반의 클라우드 네이티브 아키텍처**로 고도화하였습니다.

* **개발 기간:** 2025.03 - 2025.11 (초기 모델) / 2026.01 - 현재 (인프라 및 아키텍처 고도화)
* **담당 역할:** Backend & Cloud Infrastructure

<br>

## 🏗️ 2. System Architecture
<img width="925" height="507" alt="동네콕 아키텍처 drawio" src="https://github.com/user-attachments/assets/199231ee-5f43-42c6-a7eb-fa94f126818e" />




### 🌐 Domain & Routing Strategy
* **Frontend (`dongnekok.shop`):** AWS S3와 CloudFront를 연동하여 정적 자산의 글로벌 캐싱 성능을 확보하고 비용을 최적화했습니다. (Cloudflare DNS Only 적용)
* **Backend (`api.dongnekok.shop`):** Cloudflare Tunnel을 활용해 백엔드 서버의 공인 IP를 은닉하고, 단일 보안 터널을 통해서만 API 통신이 가능하도록 방어 표면을 구축했습니다. (Cloudflare Proxy 적용)

### 🗄️ Database Architecture (ERD)
<img width="1144" height="1168" alt="image" src="https://github.com/user-attachments/assets/1f166872-20bc-48e8-8c81-d275c92824dc" />

* **데이터 모델링 포인트:** 타임딜 커머스의 특성상 특정 시간대에 빈번하게 발생하는 '재고 조회 및 주문' 트랜잭션을 안정적으로 처리하기 위해 정규화를 진행하고, 로컬 상점과 사용자 간의 주문/결제 데이터 정합성을 보장하도록 관계형 데이터베이스(MySQL) 구조를 설계했습니다.

  
<br>

## 🛠️ 3. Tech Stack
| 분류 | 기술 스택 |
| --- | --- |
| **Backend** | Java 21, Spring Boot, Spring Security, Spring Data JPA |
| **Database** | MySQL 8.0, Redis |
| **Infra & DevOps:** | AWS (EC2, S3, CloudFront, EventBridge, Lambda), Cloudflare (DNS, Tunnel), Docker, GitHub Actions |
| **Frontend** | React, Axios |

<br>

## 🔥 4. Key Experience & Troubleshooting

### 1️⃣ 린 스타트업 기반 MVP 검증 및 비즈니스 문제 해결 [[🔗 MVP 검증 기록](https://velog.io/@mgo0415/%EB%8F%99%EB%84%A4%EC%BD%95-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%BD%94%EB%94%A9%EB%B3%B4%EB%8B%A4-%EB%A8%BC%EC%A0%80-%EB%8F%99%EB%84%A4-%EB%B9%B5%EC%A7%91-%EC%82%AC%EC%9E%A5%EB%8B%98%EB%93%A4%EA%B3%BC-MOU%EB%A5%BC-%EB%A7%BA%EB%8B%A4-MVP-%EA%B3%A0%EA%B0%9D-%EA%B2%80%EC%A6%9D)]
* **문제:** 타임딜 커머스 특성상 특정 시간대 트래픽 집중이 예상되나, 실제 수요 검증 없이 오버엔지니어링될 위험 인지
* **해결:** 로컬 베이커리의 '당일 폐기 원가 손실'을 타겟으로 한 상생 MOU 의향서를 직접 작성해 현장 영업 및 파트너사 선제 확보
* **성과:** 실제 사용자의 Pain Point를 반영한 직관적 UI 설계 및 마감 시간 트래픽 방어를 위한 백엔드 아키텍처의 비즈니스적 당위성 확립

### 2️⃣ S3 투트랙(Two-Track) 아키텍처 및 Event-Driven 무결성 검증 [[🔗 아키텍처 설계 문서](https://velog.io/@mgo0415/Refactoring-%EC%A1%B8%EC%97%85%EC%9E%91%ED%92%88%EC%9D%98-%ED%95%9C%EA%B3%84%EB%A5%BC-%EB%84%98%EC%96%B4-S3-%EB%B9%84%EC%9A%A9-%ED%9A%A8%EC%9C%A8%ED%99%94%EC%99%80-%EB%B3%B4%EC%95%88-%EB%91%90-%EB%A7%88%EB%A6%AC-%ED%86%A0%EB%81%BC-%EC%9E%A1%EA%B8%B0-feat.-Tunnel-CDN)]
* **문제:** 단일 S3 Presigned URL 사용 시 썸네일 캐싱 불가로 인한 비용 낭비와, 클라이언트 직업로드 시 보안 위협 식별
* **해결:** 공개 이미지(CDN)와 민감 문서(Presigned URL)의 접근 방식을 분리하고, AWS Lambda와 EventBridge를 연동한 무결성(매직 넘버, 크기) 검사 로직 구현
* **성과:** 백엔드 I/O 부하 0% 유지 및 불필요한 스토리지 비용/런타임 보안 위협 방어

### 3️⃣ Zero Trust 인프라 및 백오피스 논리적 망 분리 [[🔗 인프라 구축기](https://velog.io/@mgo0415/Refactoring-%EC%8B%A0%EC%9E%85%EC%9D%98-%EC%8B%9C%EC%84%A0-%EB%B0%B1%EC%98%A4%ED%94%BC%EC%8A%A4Admin-%EA%B5%AC%EC%B6%95%EA%B3%BC-Zero-Trust-%EB%B3%B4%EC%95%88-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%84%A4%EA%B3%84%EC%9D%98-%EC%97%AC%EC%A0%95)]
* **문제:** Admin 페이지 구축 시, 서버 IP 노출로 인한 DDoS 및 포트 스캐닝 위협 차단 필요
* **해결:** AWS EC2 인바운드 포트를 전면 차단하고 Cloudflare Tunnel을 도입해 공격 표면(Attack Surface) 원천 제거, OTP 인증 정책 적용
* **성과:** 사내 VPN 구축 비용 없이 논리적 망 분리 효과 달성 및 고정 IP 유지 비용 절감

### 4️⃣ 우아한 기능 저하(Graceful Degradation)를 통한 무중단 UX 제공 [[🔗 트러블슈팅 기록](https://velog.io/@mgo0415/Troubleshooting-AWS-%EB%B9%84%EC%9A%A9-0%EC%9B%90-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%84%9C%EB%B2%84%EA%B0%80-%EA%BA%BC%EC%A0%B8%EB%8F%84-%EA%B3%A0%EC%9E%A5-%EB%82%9C-%EC%82%AC%EC%9D%B4%ED%8A%B8%EC%B2%98%EB%9F%BC-%EB%B3%B4%EC%9D%B4%EC%A7%80-%EC%95%8A%EA%B2%8C-%ED%95%98%EA%B8%B0)]
* **문제:** 클라우드 비용 절감을 위한 서버 중단 시, 백엔드 다운으로 인해 프론트엔드 UI가 깨지고 502/CORS 에러 노출
* **해결:** 앱 초기화 시 경량 `HEAD` 요청으로 서버 헬스체크를 수행하는 HOC(MaintenanceGuard) 패턴 구현 및 Spring Security 403/CORS 에러 오버라이딩 해결
* **성과:** 추가 인프라 비용 없이 서버 OFF 상태에서도 안정적인 '점검 중' 화면 제공

### 5️⃣ 리버스 프록시(Nginx) 라우팅 충돌 해결 및 OAuth2 커스터마이징 [[🔗 트러블슈팅 기록](https://velog.io/@mgo0415/%ED%8A%B8%EB%9F%AC%EB%B8%94%EC%8A%88%ED%8C%85-Cloudflare-Tunnel-Docker-Nginx-Spring-Boot-OAuth2-404-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0%EA%B8%B0)]
* **문제:** Nginx 리버스 프록시 환경에서 소셜 로그인 리다이렉트 시 404 에러 연쇄 발생
* **해결:** Spring Security의 기본 엔드포인트를 Nginx Prefix(`/api`) 라우팅 규칙에 맞게 동적으로 오버라이딩 처리
* **성과:** 프론트엔드-프록시-백엔드 통신 흐름을 단일화하여 안정적인 API 네트워크 통신망 구축

<br>

## 🤝 5. Collaboration & Team Workspace

단순한 코드 공유를 넘어, 실제 비즈니스 환경과 동일한 협업 파이프라인을 구축하여 프로젝트를 운영했습니다.

* **Issue Template & PR Convention:** 기능 추가, 버그 수정 등 목적에 맞는 템플릿을 고도화하여 리뷰어의 맥락 파악 시간을 단축했습니다.
* ### 📌 체계적인 이슈 추적 및 문서화 (Issue Tracker & Convention)

<p align="center">
  <img src="https://github.com/user-attachments/assets/0d613762-cbc6-408d-b723-97eda6512a1b" width="48%" alt="이슈 목록">
  <img src="https://github.com/user-attachments/assets/fbb4eb3f-8c76-41c2-bc63-9cc843b39984" width="48%" alt="상세 이슈 템플릿">
</p>

* **커밋/이슈 컨벤션:** `[FEAT]`, `[Refactor]`, `[Infra]` 등 직관적인 말머리와 라벨링을 통해 작업의 성격을 명확히 분리하고 추적성을 높였습니다.
* **상세 템플릿:** 단순한 버그 수정이나 기능 추가를 넘어, '배경/목적 - 기술 스펙 - 체크리스트'로 이어지는 상세 템플릿을 사용하여 작업 전 설계의 타당성을 검증했습니다.


### 🤝 6. Collaboration & Workspace (문서화 및 에셋 관리)
단순한 코드 공유를 넘어, 실제 스타트업 환경과 동일한 수준의 체계적인 문서화 및 자산 관리 파이프라인을 구축했습니다.

<p align="center">
  <img src="https://github.com/user-attachments/assets/6c9a7b35-441c-4f54-a01b-08ccdf688eb7" width="48%" alt="노션 워크스페이스">
  <img src="https://github.com/user-attachments/assets/07c8a1ba-6646-4b87-acd0-ccaa2b70d1c3" width="48%" alt="구글 드라이브 에셋 관리">
</p>

* **Notion 기반 사내 위키:** 기술(Architecture, ERD, API) 문서뿐만 아니라 기획, 영업(상생 협력 의향서) 등 목적별로 문서를 카테고리화하여 중앙 집중식으로 관리합니다.
* **Google Drive 자산 동기화:** 아키텍처 원본 파일(.drawio), 데이터베이스 산출물 등을 기술 개발 전용 폴더 체계(01_Architecture ~ 04_Infra)로 세분화하여 안전하게 보관 및 링크 연동하고 있습니다.
* **추후 고도화 계획 (To-be):** Jira 티켓 기반의 스프린트 관리 및 Slack 연동을 통한 CI/CD 배포 자동화 알림 파이프라인 구축 (진행 예정)
