# ATS System (Applicant Tracking System)

一個基於微服務架構的求職者追蹤系統，用於管理招聘流程中的各個環節。

## 專案結構

本專案採用微服務架構，包含以下服務模組：

### 核心服務

#### **[applicant-service/](applicant-service/)** - 求職者管理服務
處理求職者資料的 CRUD 操作，包含：
- **Controller**: [`ApplicantController`](applicant-service/src/main/java/org/ats/applicantservice/controller/ApplicantController.java) - 提供 `/api/applicants` REST API
- **Service**: 求職者業務邏輯處理
- **Entity**: 求職者實體模型
- **Repository**: 基於 JPA 的資料存取層
- **Application**: [`ApplicantServiceApplication`](applicant-service/src/main/java/org/ats/applicantservice/ApplicantServiceApplication.java) - 服務啟動類
- **Configuration**: 資料庫連線配置 (PostgreSQL: `applicant_db`)

#### **[job-service/](job-service/)** - 職位管理服務  
負責職位發布、編輯和查詢功能，包含：
- **Controller**: [`JobController`](job-service/src/main/java/org/ats/jobservice/controller/JobController.java) - 提供 `/api/jobs` REST API
- **Service**: [`JobService`](job-service/src/main/java/org/ats/jobservice/service/JobService.java) - 職位業務邏輯處理
- **Entity**: [`Job`](job-service/src/main/java/org/ats/jobservice/entity/Job.java) - 職位實體 (包含職位標題、部門、狀態、申請人數統計等)
- **Repository**: [`JobRepository`](job-service/src/main/java/org/ats/jobservice/repository/JobRepository.java) - 職位資料存取
- **Config**: [`WebConfig`](job-service/src/main/java/org/ats/jobservice/config/WebConfig.java) - CORS 跨域設定
- **Application**: [`JobServiceApplication`](job-service/src/main/java/org/ats/jobservice/JobServiceApplication.java) - 服務啟動類
- **Configuration**: 資料庫連線配置 (PostgreSQL: `job_db`)

#### **[interview-service/](interview-service/)** - 面試管理服務
管理面試安排和記錄，包含：
- **Controller**: 
  - [`InterviewSessionController`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewSessionController.java) - 面試場次管理 `/api/interviewSessions`
  - [`InterviewScoreController`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewScoreController.java) - 面試回饋管理 `/api/feedbacks`
- **Service**: [`InterviewSessionService`](interview-service/src/main/java/org/ats/interviewservice/service/InterviewSessionService.java) - 面試排程和管理邏輯
- **Entity**: 
  - [`InterviewSession`](interview-service/src/main/java/org/ats/interviewservice/entity/InterviewSession.java) - 面試場次實體
  - `Feedback` - 面試回饋實體
  - `Interviewer` - 面試官實體
  - `Location` - 面試地點嵌入式物件
- **DTO**: [`ScheduleRequest`](interview-service/src/main/java/org/ats/interviewservice/dto/ScheduleRequest.java) - 面試排程請求物件
- **Configuration**: 資料庫連線配置 (PostgreSQL: `interview_db`)

#### **[process-service/](process-service/)** - 流程管理服務
控制招聘流程的狀態轉換，包含：
- **Configuration**: 資料庫連線配置 (PostgreSQL: `process_db`)
- **Business Logic**: 處理招聘流程各階段的狀態管理
- **Integration**: 與其他服務協調流程進度

#### **[recruitment-service/](recruitment-service/)** - 招聘管理服務
統籌整個招聘活動，包含：
- **Configuration**: 資料庫連線配置 (PostgreSQL: `recruitment_db`)
- **Orchestration**: 協調各服務間的招聘流程
- **Dashboard**: 提供招聘活動總覽功能

#### **[tracked-applicant-service/](tracked-applicant-service/)** - 求職者追蹤服務
記錄求職者在招聘流程中的狀態變化，包含：
- **Controller**: [`TrackedApplicantController`](tracked-applicant-service/src/main/java/org/ats/trackedapplicantservice/controller/TrackedApplicantController.java) - 提供 `/api/trackedApplicants` REST API
- **Features**:
  - 求職者推薦功能 (`POST /api/trackedApplicants?applicantId=1&jobId=10`)
  - 依職位查詢追蹤記錄 (`GET /api/trackedApplicants?jobId=1`)
  - 追蹤記錄管理 (新增、刪除、查詢)
- **Integration**: 透過 Feign Client 整合 applicant-service
- **DTO**: `TrackedApplicantDTO` - 追蹤資料傳輸物件
- **Configuration**: 資料庫連線配置 (PostgreSQL: `tracked_applicant_db`)

### 共用模組
- **[common-dto/](common-dto/)** - 共用資料傳輸物件
  - `InterviewScoreDTO` - 面試評分資料傳輸物件
  - 其他跨服務通訊的共用資料格式

### 基礎設施
- **[dockerfiles/](dockerfiles/)** - Docker 映像檔建構配置
- **[postgres-data/](postgres-data/)** - PostgreSQL 資料庫資料存放目錄
- **[src/](src/)** - 共用原始碼目錄
- **容器化配置**:
  - 每個服務都包含獨立的 [`Dockerfile`](job-service/Dockerfile)
  - 使用 [`start.sh`](job-service/start.sh) 腳本啟動 Cloud SQL Proxy
  - 統一使用 Java 17 Alpine 映像

### 配置檔案
- **[docker-compose.yml](docker-compose.yml)** - Docker Compose 服務編排配置
- **[build-and-push.sh](build-and-push.sh)** - 自動化建構和推送腳本 (支援 GCP Artifact Registry)
- **[pom.xml](pom.xml)** - Maven 主要建構配置檔案

## 技術棧

- **後端框架**: Spring Boot 3.x
- **建構工具**: Maven
- **容器化**: Docker & Docker Compose
- **資料庫**: PostgreSQL (每個服務使用獨立資料庫)
- **雲端服務**: Google Cloud Platform (Cloud Run + Cloud SQL)
- **資料存取**: Spring Data JPA + Hibernate
- **服務間通訊**: OpenFeign
- **開發環境**: IntelliJ IDEA

## 服務架構特點

### 微服務設計模式
- **資料庫分離**: 每個服務使用獨立的 PostgreSQL 資料庫
- **服務間通訊**: 透過 REST API 和 Feign Client 進行服務間調用
- **獨立部署**: 每個服務可獨立建構、測試和部署

### 雲原生架構
- **容器化部署**: 所有服務都支援 Docker 容器化
- **雲端資料庫**: 使用 Cloud SQL Proxy 連接 Google Cloud SQL
- **自動化 CI/CD**: 透過 [`build-and-push.sh`](build-and-push.sh) 實現自動化部署

## API 端點總覽

| 服務 | 端點 | 功能描述 |
|------|------|----------|
| applicant-service | `/api/applicants` | 求職者 CRUD 操作 |
| job-service | `/api/jobs` | 職位 CRUD 操作 |
| interview-service | `/api/interviewSessions` | 面試場次管理 |
| interview-service | `/api/feedbacks` | 面試回饋管理 |
| tracked-applicant-service | `/api/trackedApplicants` | 求職者追蹤記錄 |

## 快速開始

### 前置需求
- Java 17+
- Docker & Docker Compose
- Maven 3.6+
- Google Cloud SDK (用於雲端部署)

### 本地開發環境設置

1. 複製專案到本地
```bash
git clone <repository-url>
cd ats-system
```

2. 使用 Docker Compose 啟動所有服務
```bash
docker-compose up -d
```

3. 或者使用提供的建構腳本部署到 GCP
```bash
chmod +x build-and-push.sh
./build-and-push.sh
```

### 建構與測試

使用 Maven 建構專案：
```bash
./mvnw clean install
```

運行測試：
```bash
./mvnw test
```

### 個別服務啟動

進入特定服務目錄並啟動：
```bash
cd applicant-service
./mvnw spring-boot:run
```

## 資料庫架構

每個服務使用獨立的 PostgreSQL 資料庫：
- `applicant_db` - 求職者資料
- `job_db` - 職位資料  
- `interview_db` - 面試資料
- `process_db` - 流程資料
- `recruitment_db` - 招聘資料
- `tracked_applicant_db` - 追蹤資料

## 貢獻指南

歡迎提交 Issue 和 Pull Request 來改善這個專案。在貢獻代碼前，請確保：

1. 遵循現有的代碼風格
2. 添加適當的測試用例
3. 更新相關文檔
4. 確保所有服務都能正常啟動

## 授權

[請在此處添加授權資訊]
