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

### Applicant Service - 求職者管理
**Base URL**: `/api/applicants`

| HTTP Method | Endpoint | 功能描述 | Controller Method |
|------------|----------|----------|-------------------|
| GET | `/api/applicants` | 取得所有求職者列表 | [`getAllApplicants()`](applicant-service/src/main/java/org/ats/applicantservice/controller/ApplicantController.java) |
| GET | `/api/applicants/{applicantId}` | 根據 ID 取得單一求職者資料 | [`getApplicantById()`](applicant-service/src/main/java/org/ats/applicantservice/controller/ApplicantController.java) |
| GET | `/api/applicants?jobId={jobId}` | 根據職位 ID 取得該職位的所有求職者 | [`getApplicantsByJobId()`](applicant-service/src/main/java/org/ats/applicantservice/controller/ApplicantController.java) |
| POST | `/api/applicants` | 新增求職者資料 | [`create()`](applicant-service/src/main/java/org/ats/applicantservice/controller/ApplicantController.java) |
| DELETE | `/api/applicants/{applicantId}` | 刪除指定求職者 | [`delete()`](applicant-service/src/main/java/org/ats/applicantservice/controller/ApplicantController.java) |

### Job Service - 職位管理
**Base URL**: `/api/jobs`

| HTTP Method | Endpoint | 功能描述 | Controller Method |
|------------|----------|----------|-------------------|
| GET | `/api/jobs` | 取得所有職位列表 | [`getAllJobs()`](job-service/src/main/java/org/ats/jobservice/controller/JobController.java) |
| GET | `/api/jobs/{id}` | 根據 ID 取得單一職位資料 | [`getJobById()`](job-service/src/main/java/org/ats/jobservice/controller/JobController.java) |
| POST | `/api/jobs` | 新增職位 | [`createJob()`](job-service/src/main/java/org/ats/jobservice/controller/JobController.java) |
| DELETE | `/api/jobs/{id}` | 刪除指定職位 | [`deleteJob()`](job-service/src/main/java/org/ats/jobservice/controller/JobController.java) |

### Interview Service - 面試管理
#### 面試場次管理
**Base URL**: `/api/interviewSessions`

| HTTP Method | Endpoint | 功能描述 | Controller Method |
|------------|----------|----------|-------------------|
| GET | `/api/interviewSessions` | 取得所有面試場次 | [`getAllSessions()`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewSessionController.java) |
| GET | `/api/interviewSessions/{id}` | 根據 ID 取得指定面試場次 | [`getSessionById()`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewSessionController.java) |
| GET | `/api/interviewSessions/interviewer/{id}` | 取得指定面試官的面試場次 | [`getSessionsForInterviewer()`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewSessionController.java) |
| POST | `/api/interviewSessions` | 新增面試場次 | [`createSession()`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewSessionController.java) |
| POST | `/api/interviewSessions/schedule` | HR 排程面試 (三位面試官對一位應聘者) | [`scheduleSession()`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewSessionController.java) |
| DELETE | `/api/interviewSessions/{id}` | 刪除指定面試場次 | [`deleteSession()`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewSessionController.java) |

#### 面試回饋管理
**Base URL**: `/api/feedbacks`

| HTTP Method | Endpoint | 功能描述 | Controller Method |
|------------|----------|----------|-------------------|
| GET | `/api/feedbacks?candidateId={candidateId}&interviewerId={interviewerId}` | 取得面試官對應聘者的回饋 | [`getFeedbackByCandidateAndInterviewer()`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewScoreController.java) |
| POST | `/api/feedbacks` | 儲存面試回饋 | [`saveFeedback()`](interview-service/src/main/java/org/ats/interviewservice/controller/InterviewScoreController.java) |

### Tracked Applicant Service - 求職者追蹤
**Base URL**: `/api/trackedApplicants`

| HTTP Method | Endpoint | 功能描述 | Controller Method |
|------------|----------|----------|-------------------|
| GET | `/api/trackedApplicants` | 取得所有追蹤記錄 | [`getAll()`](tracked-applicant-service/src/main/java/org/ats/trackedapplicantservice/controller/TrackedApplicantController.java) |
| GET | `/api/trackedApplicants?jobId={jobId}` | 根據職位 ID 取得該職位的追蹤記錄 | [`getByJobId()`](tracked-applicant-service/src/main/java/org/ats/trackedapplicantservice/controller/TrackedApplicantController.java) |
| POST | `/api/trackedApplicants?applicantId={applicantId}&jobId={jobId}` | 建立求職者推薦 (追蹤記錄) | [`recommend()`](tracked-applicant-service/src/main/java/org/ats/trackedapplicantservice/controller/TrackedApplicantController.java) |
| DELETE | `/api/trackedApplicants/{id}` | 刪除指定追蹤記錄 | [`delete()`](tracked-applicant-service/src/main/java/org/ats/trackedapplicantservice/controller/TrackedApplicantController.java) |

#### 主管操作管理
**Base URL**: `/api/supervisor-actions`

| HTTP Method | Endpoint | 功能描述 | Controller Method |
|------------|----------|----------|-------------------|
| POST | `/api/supervisor-actions` | 儲存主管操作記錄 | [`saveAction()`](tracked-applicant-service/src/main/java/org/ats/trackedapplicantservice/controller/SupervisorActionController.java) |

### Process Service - 流程管理
**Base URL**: `/api/processes`

| HTTP Method | Endpoint | 功能描述 | Controller Method |
|------------|----------|----------|-------------------|
| GET | `/api/processes` | 取得全域預設流程步驟 | [`getDefaultSteps()`](process-service/src/main/java/org/ats/processservice/controller/ProcessStepController.java) |

**實體結構**：
- [`ProcessStep`](process-service/src/main/java/org/ats/processservice/entity/ProcessStep.java) - 包含步驟順序、標籤、可能結果
- 支援步驟如：「初審」、「一面」、「二面」、「最終面試」
- 支援結果如：「進行中」、「通過」

## API 使用範例

### 求職者推薦流程
```bash
# 1. 建立求職者推薦
POST /api/trackedApplicants?applicantId=1&jobId=10

# 2. 查詢該職位的所有追蹤記錄
GET /api/trackedApplicants?jobId=10

# 3. 安排面試
POST /api/interviewSessions/schedule
Content-Type: application/json
{
  "candidateId": 1,
  "jobId": 10,
  "interviewerIds": [1, 2, 3],
  "scheduledTime": "2024-01-15T10:00:00"
}
```

### 面試回饋流程
```bash
# 1. 面試官提交回饋
POST /api/feedbacks
Content-Type: application/json
{
  "candidateId": 1,
  "interviewerId": 2,
  "score": 85,
  "comments": "表現良好，技術能力符合需求"
}

# 2. 查詢特定面試官對特定候選人的回饋
GET /api/feedbacks?candidateId=1&interviewerId=2
```

### CORS 設定
部分服務已設定 CORS 支援前端應用程式 (`http://localhost:4200`)：
- [`tracked-applicant-service`](tracked-applicant-service/src/main/java/org/ats/trackedapplicantservice/controller/SupervisorActionController.java) 的 SupervisorActionController
- [`job-service`](job-service/src/main/java/org/ats/jobservice/config/WebConfig.java) 透過 WebConfig 全域設定


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

## 開發進度與 To-Do

### 🚧 Process Service 實作狀況
**目前完成項目**：
- ✅ 基礎架構設立 (Spring Boot + JPA)
- ✅ [`ProcessStep`](process-service/src/main/java/org/ats/processservice/entity/ProcessStep.java) 實體設計完成
  - 支援步驟順序 (`stepOrder`)
  - 支援步驟標籤 (`label`) - 如「初審」、「一面」
  - 支援可能結果 (`possibleOutcomes`) - 如「進行中」、「通過」、「拒絕」
- ✅ [`ProcessStepController`](process-service/src/main/java/org/ats/processservice/controller/ProcessStepController.java) 基本框架
  - 提供 `GET /api/processes` 取得預設流程步驟
- ✅ [`ProcessStepService`](process-service/src/main/java/org/ats/processservice/service/ProcessStepService.java) 基礎服務層
- ✅ [`ProcessStepRepository`](process-service/src/main/java/org/ats/processservice/repository/ProcessStepRepository.java) 資料存取層

**待完成項目**：
 ☐ 個別求職者流程狀態追蹤實體 (`ApplicantProcessState`)
 ☐ 流程自動化觸發機制
 ☐ 流程統計和報表功能

### 📋 待開發功能 (To-Do List)

#### 🔔 通知服務 (Notification Service)
**新增服務建議**：
- **Email 通知**：
  - 面試邀請通知
  - 流程狀態變更通知
  - 面試結果通知
- **系統內通知**：
  - 即時通知 (WebSocket)
  - 通知歷史記錄
- **通知模板管理**：
  - 可自訂的通知範本
  - 多語言支援
- **整合點**：
  - Interview Service → 面試安排通知
  - Process Service → 流程變更通知
  - Tracked Applicant Service → 狀態更新通知

#### 📅 面試時程總表功能
**Interview Service 擴充**：
- **時程總覽 API**：
  ```
  GET /api/interview-schedule/calendar?startDate=2024-01-01&endDate=2024-01-31
  GET /api/interview-schedule/interviewer/{interviewerId}/calendar
  GET /api/interview-schedule/job/{jobId}/calendar
  ```
- **資料整合**：
  - 整合面試官行事曆
  - 整合會議室預約狀況
  - 整合求職者可面試時段
- **前端功能**：
  - 月曆檢視模式
  - 週檢視模式
  - 衝突提醒機制
  - 批量排程功能

#### 🎯 其他優化項目
- **API Gateway**：統一服務入口和路由管理
- **服務監控**：健康檢查和效能監控
- **資料分析**：招聘效率分析和儀表板
- **權限管理**：RBAC 角色權限控制
- **檔案管理**：履歷和面試記錄檔案上傳

### 🏗️ 架構改進建議
- **消息佇列**：使用 RabbitMQ 或 Kafka 實現非同步通訊
- **快取層**：Redis 快取常用資料
- **配置中心**：Spring Cloud Config 集中配置管理
- **服務註冊與發現**：Eureka 或 Consul

## 授權

[請在此處添加授權資訊]
