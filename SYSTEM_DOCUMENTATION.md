# 📘 TÀI LIỆU ĐẶC TẢ CHI TIẾT TOÀN BỘ HỆ THỐNG AUTOMATION UI BUILDER
# (SYSTEM REQUIREMENTS, FEATURES & SPECIFICATION MANUAL)

> **Phiên bản tài liệu**: `3.0 Enterprise Edition`  
> **Ngày cập nhật**: `09/09/2026`  
> **Trạng thái**: Đã chuẩn hóa chính xác 100% theo mã nguồn thực tế

---

## 1. 🌐 TỔNG QUAN HỆ THỐNG (SYSTEM OVERVIEW)

**Automation UI Builder** *(FlowBuilder Automation Platform)* là nền tảng kiểm thử tự động toàn diện (All-in-One Enterprise Test Automation Platform) hỗ trợ đa nền tảng (**Web UI Playwright, REST API, Database SQL SUT, Mobile Appium, JavaScript Logic Flow**). Nền tảng cho phép người dùng xây dựng, trực quan hóa, chỉnh sửa mã nguồn và thực thi kịch bản kiểm thử thông qua **3 giao diện đồng bộ 2 chiều không mất mát (Lossless 3-Way Sync)**:

1. **Diagram Canvas (Sơ đồ luồng trực quan)**: Kéo thả các khối chức năng, tự động bố cục đồ thị qua Dagre Layout, trực quan hóa luồng rẽ nhánh và vòng lặp với các Container Node chuyên biệt (`IfElseGroup` 2 làn True/Else, `WhileGroup`, `ForGroup`).
2. **Table Step View (Dạng bảng phân cấp)**: Thao tác nhanh dạng danh sách cây phân cấp (Hierarchy Indent/Outdent), sắp xếp lại thứ tự bằng kéo thả (CDK Drag-Drop).
3. **Script Code Editor (Trình soạn thảo mã nguồn DSL & Playwright)**: Viết mã trực tiếp dạng TypeScript DSL (CodeMirror 6 / Monaco Editor) với tính năng tự động hoàn thành (IntelliSense), hỗ trợ nút **✨ Format Code** chuẩn hóa cú pháp.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                   KIẾN TRÚC TỔNG THỂ                                   │
└────────────────────────────────────────────────────────────────────────────────────────┘

    [ ANGULAR 21 CLIENT (SPA) + TAILWIND CSS v4 ]
    ├─ Diagram View (ng-diagram 1.3, Dagre Auto-Layout, Container Groups)
    ├─ Table Step View (CDK Drag-Drop, Hierarchy Indent/Outdent)
    ├─ Script Editor View (CodeMirror 6 / Monaco, TypeScript DSL, Format Code)
    ├─ AST Flow Engine (2-Way Code <-> Graph Synchronizer)
    ├─ Execution Console (Real-time Step Logs & Screenshot Lightbox)
    ├─ Allure Diagnostics Dashboard (ApexCharts 5.3, Fast Pre-Aggregated Summary)
    └─ Router Security (authGuard, guestGuard, roleGuard, authInterceptor)
                 │
                 ▼ (RESTful JSON / SSE / WebSocket /ws/live-log)
    [ BACKEND GATEWAY (Spring Boot 2.7.18 LTS / Java 11 LTS) ]
    ├─ Auth & User Management (JWT / Multi-Tenant RBAC / Created-By Ownership)
    ├─ Project & Environment Manager (Dynamic Variables Interpolation)
    ├─ Test Case & Test Suite Engine (Metadata vs Flow DAG JSON Separation)
    ├─ Enterprise Licensing & RSA-2048 / HWID Verifier (ProGuard Obfuscated)
    ├─ AI Flow Generator (OpenAI / Claude / Custom API Integration)
    └─ Coordinator & Task Dispatcher
                 │
                 ├──────────────────────────────┬──────────────────────────────┐
                 ▼                              ▼                              ▼
          [ REDIS 7.2 ]                   [ MYSQL 8.0 ]               [ WORKER POOL ]
       Pub/Sub Event Bus &           Persistent DB Storage       Playwright & REST Runners
       Task Distributed Queue        (Projects, Tests, Logs)     (Worker 1, Worker 2, ...)
```

---

## 2. 🛡️ BẢO MẬT, ĐỊNH TUYẾN & PHÂN QUYỀN (SECURITY & RBAC)

### 2.1. Router Guards
- **`authGuard` (`core/guards/auth.guard.ts`)**:
  - Bảo vệ layout chính (`AppLayoutComponent`) và toàn bộ các tuyến đường quản trị (`/projects`, `/test-cases`, `/test-suite`).
  - Nếu phiên đăng nhập bị mất hoặc chưa xác thực $\rightarrow$ Tự động chuyển hướng về `/signin?returnUrl=<current_url>`.
- **`guestGuard` (`core/guards/guest.guard.ts`)**:
  - Chặn người dùng đã đăng nhập truy cập lại các trang xác thực (`/signin`, `/signup`).
  - Đã có phiên đăng nhập $\rightarrow$ Tự động chuyển hướng về trang làm việc chính `/projects`.
- **`roleGuard` (`core/guards/role.guard.ts`)**:
  - Kiểm tra vai trò của người dùng khi truy cập các trang quản trị đặc quyền (`/system/licenses`, `/system/users`).
- **`authInterceptor` (`core/interceptors/auth.interceptor.ts`)**:
  - Tự động gắn token JWT vào Header `Authorization: Bearer <token>` cho mọi request nội bộ.
  - Bắt lỗi HTTP `401 Unauthorized` để dọn dẹp session và điều hướng về trang đăng nhập.

### 2.2. Phân Quyền & Quyền Sở Hữu Kịch Bản (Data Ownership)
- **Thông tin khởi tạo (`createdBy`, `createdDate`)**: Mọi Test Case và Test Suite đều lưu vết người tạo và ngày tạo trong cơ sở dữ liệu.
- **Quy tắc phân quyền 6 vai trò**:
  - **`ADMIN`**: Toàn quyền cấu hình dự án, quản lý thành viên, nạp giấy phép bản quyền và thực thi.
  - **`IT_SUPPORT`**: Cổng quản trị chuyên biệt (`/system/licenses`, `/system/users`), đổi mật khẩu, đồng bộ người dùng, quản lý kích hoạt/vô hiệu hóa giấy phép. Không bị popup chặn khi hết hạn license.
  - **`APPROVER`**: Thẩm định kịch bản, phê duyệt (`APPROVE`) hoặc từ chối (`REJECT`), trao đổi bình luận.
  - **`DEVELOPER`**: Thiết kế lưu đồ, quản lý Test Suites, viết hàm JavaScript toàn cục và cấu hình biến môi trường.
  - **`TESTER`**: Thiết kế kịch bản, thực thi kiểm thử và chỉ được chỉnh sửa/xóa kịch bản do chính mình tạo ra (`createdBy === currentUser.username`).
  - **`VIEWER`**: Quyền chỉ xem (Read-only) kịch bản và theo dõi báo cáo kết quả.

---

## 3. 🏗️ KIẾN TRÚC INFRASTRUCTURE & CONTAINER (DOCKER)

Hệ thống được đóng gói và triển khai qua **Docker Compose**:

| Container Service | Image / Base | Cổng Expose | Vai trò & Trách nhiệm |
| :--- | :--- | :--- | :--- |
| **`automation-frontend`** | `nginx:alpine` + Angular 21 (Production Bundle) | `4200:80`, `80:80` | Phục vụ Web App SPA, định tuyến client-side, proxy API. |
| **`automation-gateway`** | Spring Boot 2.7.18 (`eclipse-temurin:11-jre`) | `9000:9000` | Tiếp nhận API, xác thực JWT, quản lý CRUD kịch bản, điều phối Task qua Redis. |
| **`automation-worker`** | Spring Boot 2.7.18 + Playwright (`x2 instances`) | `8081` (Nội bộ) | Lắng nghe task từ Redis qua `BRPOP`, khởi chạy Browser Context Pool và chạy test. |
| **`automation-mysql`** | `mysql:8.0` | `3306:3306` | Lưu trữ cơ sở dữ liệu quan hệ chính (`automation_db`), bảng tính sẵn `project_stats`. |
| **`automation-redis`** | `redis:7.2-alpine` | `6379:6379` | Hàng đợi tác vụ phân tán, Pub/Sub log thời gian thực, kênh điều khiển khẩn cấp & sync license. |

---

## 4. 🧠 BỘ ĐỒNG BỘ 2 CHIỀU THỜI GIAN THỰC (AST FLOW ENGINE)

Trung tâm của hệ thống là **`AstFlowEngineService`**, chịu trách nhiệm chuyển đổi không mất dữ liệu (lossless) giữa mã nguồn TypeScript DSL và mô hình Đồ thị (Graph Nodes & Edges).

```
          ┌─────────────────────────────────────────────────────────┐
          │                   TypeScript DSL Code                   │
          │ flow.web.openUrl('https://...');                        │
          │ if (response.status === 200) { flow.web.click('btn'); } │
          └────────────────────────────┬────────────────────────────┘
                                       │
                                       │ (1) Lexer & AST Parser (Tự động bọc function)
                                       ▼
          ┌─────────────────────────────────────────────────────────┐
          │                    AST Abstract Tree                    │
          │ [FlowCall, IfStatement(consequent, alternate), ...]     │
          └────────────────────────────┬────────────────────────────┘
                                       │
                                       │ (2) astToGraph() + Catalog Hydration
                                       ▼
          ┌─────────────────────────────────────────────────────────┐
          │                  Diagram Graph Model                    │
          │ Nodes: [IfElseGroup, WhileGroup, ForGroup, StepNode]   │
          │ Edges: [Input -> Output Orthogonal Connections]         │
          └────────────────────────────┬────────────────────────────┘
                                       │
                                       │ (3) graphToCode()
                                       ▼
          ┌─────────────────────────────────────────────────────────┐
          │                   Generated Clean DSL                   │
          └─────────────────────────────────────────────────────────┘
```

### 4.1. Các Khối Lệnh Cấu Trúc (Control Flow Containers)
Hệ thống sử dụng mô hình **Group Container** trực quan, đồng bộ style giữa các khối:

#### A. Khối Điều Kiện: `IfElseGroupComponent`
- **Kích thước chuẩn**: `680 x 280 px`.
- **Cấu trúc**: Container 2 ngăn song song:
  - **Làn trái (Emerald)**: Nhánh `True (Nếu đúng)` $\rightarrow$ Chứa các bước có `branch: 'true'`.
  - **Làn phải (Rose)**: Nhánh `Else (Nếu sai)` $\rightarrow$ Chứa các bước có `branch: 'false'`.
- **Dữ liệu**: Các node con liên kết qua `parentNode: ifGroupId` và `groupId: ifGroupId`.
- **Cổng kết nối**: 1 Cổng Input bên trái và 1 Cổng Output bên phải.
- **Tương tác**: Click vào Header thanh công cụ mở Drawer cấu hình Condition Expression.

#### B. Khối Vòng Lặp: `WhileGroupComponent` & `ForGroupComponent`
- **Kích thước chuẩn**: `550 x 280 px`.
- **Cấu trúc**: Container bo góc chuyên biệt hiển thị biểu thức lặp `while (condition)` hoặc vòng lặp đếm `for (count)`.
- **Dữ liệu**: Hỗ trợ kéo thả node con vào trong vùng chứa với cơ chế tính toán va chạm $\ge 20\%$ diện tích tiếp xúc.

---

## 5. 💻 EXECUTION CONSOLE & LOGGING SYSTEM

- **Step-Based Log Grouping**: Các bản ghi log được tự động gom nhóm theo từng Step.
- **Sub-Test Case Collapsible**: Với các bước gọi Sub-Test Case, danh sách bước con được hiển thị dạng cây thu gọn/mở rộng trực quan.
- **Ảnh chụp màn hình (Screenshot Lightbox)**:
  - Tích hợp nút xem nhanh `📸 Ảnh` và thumbnail trực tiếp trong log.
  - Hỗ trợ popup xem ảnh kích thước gốc và tải ảnh về máy dạng `.png`.
- **Console Filter & Search**: Lọc nhanh log theo trạng thái (`ALL`, `ERROR`, `SUCCESS`) và tìm kiếm từ khóa thời gian thực.
- **Kênh truyền phát**: Hỗ trợ đồng thời Server-Sent Events (`GET /api/automation/stream/{runId}`) và WebSocket STOMP (`/ws/live-log`).

---

## 6. 📚 ACTION CATALOG (BỘ DANH MỤC THAO TÁC)

Hệ thống hỗ trợ hơn 50+ từ khóa kiểm thử với đầy đủ schema tham số:

### 6.1. Nhóm Web UI Automation (Playwright Engine)
| Keyword | Tên Thao Tác | Tham Số Chính |
| :--- | :--- | :--- |
| `OPEN_URL` | Mở trang web | `url` |
| `CLICK` | Nhấp chuột | `locatorValue`, `selector`, `locatorType` |
| `DOUBLE_CLICK` | Nhấp đúp chuột | `locatorValue`, `selector`, `locatorType` |
| `RIGHT_CLICK` | Nhấp chuột phải | `locatorValue`, `selector`, `locatorType` |
| `HOVER` | Rê chuột | `locatorValue`, `selector`, `locatorType` |
| `INPUT_TEXT` | Nhập văn bản | `locatorValue`, `selector`, `testData`, `text`, `value` |
| `CLEAR_TEXT` | Xóa văn bản | `locatorValue`, `selector`, `locatorType` |
| `PRESS_KEY` | Nhấn phím bàn phím | `key` (Enter, Tab, Escape...) |
| `SCROLL_TO` | Cuộn tới phần tử | `locatorValue`, `selector`, `locatorType` |
| `DRAG_AND_DROP`| Kéo và thả phần tử | `sourceLocator`, `targetLocator` |
| `ASSERT_VISIBLE` | Kiểm tra hiển thị | `locatorValue`, `selector`, `timeout` |
| `ASSERT_TEXT` | Kiểm tra nội dung text | `locatorValue`, `selector`, `expectedText` |
| `ASSERT_VALUE` | Kiểm tra giá trị input | `locatorValue`, `selector`, `expectedValue` |
| `WAIT_FOR_ELEMENT`| Chờ phần tử xuất hiện | `locatorValue`, `selector`, `timeout` |
| `TAKE_SCREENSHOT` | Chụp ảnh màn hình | `screenshotName`, `value` |
| `WAIT_TIME` | Tạm dừng luồng | `timeout` (ms) |

### 6.2. Nhóm API Testing (REST Client)
| Keyword | Tên Thao Tác | Tham Số Chính |
| :--- | :--- | :--- |
| `HTTP_GET` | Gửi yêu cầu GET | `endpoint`, `url`, `headers` |
| `SEND_REQUEST` | Gửi HTTP Request (POST/PUT/DELETE) | `method`, `endpoint`, `url`, `body`, `headers` |
| `ASSERT_STATUS` | Kiểm tra mã phản hồi HTTP | `expectedStatus` (200, 201, 400...) |
| `EXTRACT_RESPONSE`| Trích xuất biến từ JSON Response | `jsonPath`, `variableName` |

### 6.3. Nhóm Database (SQL Engine)
| Keyword | Tên Thao Tác | Tham Số Chính |
| :--- | :--- | :--- |
| `EXECUTE_QUERY` | Thực thi câu lệnh SQL | `sqlQuery`, `query`, `targetDb` |
| `ASSERT_ROW_COUNT`| Kiểm tra số lượng bản ghi trả về | `expectedRowCount` |

### 6.4. Nhóm Logic & Control Flow
| Keyword | Tên Thao Tác | Tham Số Chính |
| :--- | :--- | :--- |
| `IF_CONDITION` | Nhánh rẽ điều kiện (If-Else) | `conditionExpression` |
| `WHILE_LOOP` | Vòng lặp điều kiện | `conditionExpression`, `maxIterations`, `delayMs` |
| `FOR_LOOP` | Vòng lặp đếm | `loopCount`, `iteratorVariable` |
| `CALL_SUB_TESTCASE`| Gọi kịch bản kiểm thử con | `subTestCaseId` |
| `EXECUTE_CODE` | Chạy đoạn mã JavaScript tùy chỉnh | `script`, `codeLanguage` |

---

## 7. 🚀 QUY TRÌNH THỰC THI & TÍNH NĂNG NÂNG CAO

1. **AI Assistant & Flow Generator**: Nhận prompt ngôn ngữ tự nhiên từ người dùng $\rightarrow$ sinh ra kịch bản hoàn chỉnh 100% dạng Diagram & Code, quản lý AI Key và lịch sử học tập.
2. **Chrome Web Recorder Companion**: Extension Manifest V3 tự động ghi thao tác người dùng trên web, bắt locator đa lớp và đẩy thẳng vào Canvas.
3. **Hooks System (Thiết lập trước/sau test)**:
   - `BEFORE_ALL` / `AFTER_ALL`: Khởi tạo môi trường, dọn dẹp dữ liệu.
   - `BEFORE_EACH` / `AFTER_EACH`: Đăng nhập, chụp màn hình sau mỗi bước.
   - `ON_FAILURE`: Tự động chụp ảnh màn hình và lưu log khi có bước bị lỗi (Failed).
4. **Review & Collaboration**: Đính kèm nhận xét (Review Comments `💬`) trực tiếp trên từng Node bước kiểm thử.
5. **Auto-Format Code**: Nút `✨ Format Code` tích hợp trong Script Editor tự động căn lề và định dạng mã nguồn chuẩn mực.
