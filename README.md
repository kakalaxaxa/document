# Automation UI Builder — Nền Tảng Tự Động Hóa Kiểm Thử Toàn Diện
*(FlowBuilder Automation Platform — Enterprise Edition)*

> **Automation UI Builder** là nền tảng kiểm thử tự động thế hệ mới (All-in-One Enterprise Test Automation Platform), hỗ trợ thiết kế, trực quan hóa và thực thi kịch bản kiểm thử đa nền tảng (**Web UI Playwright, REST API, Database SQL SUT, Mobile Appium, Control Flow Logic**) với khả năng đồng bộ 2 chiều không mất mát (Lossless 3-Way Sync) giữa **Sơ đồ đồ thị (Diagram Canvas)**, **Bảng phân cấp (Table Steps)** và **Mã nguồn TypeScript (Script DSL)**.

---

## 🌟 Tính Năng Nổi Bật

### 1. Đồng Bộ 3 Chiều Không Mất Mát (Lossless 3-Way Synchronization)
- **Diagram View**: Kéo thả các khối kiểm thử, vòng lặp (`WhileGroup`, `ForGroup`) và rẽ nhánh (`IfElseGroup` 2 làn True/Else) trực quan. Tự động căn chỉnh sơ đồ theo giải thuật Dagre Auto-Layout.
- **Table Step View**: Thao tác nhanh dạng bảng danh sách phân cấp (Hierarchy Indent/Outdent), đổi thứ tự bằng kéo thả CDK Drag-Drop.
- **Script Editor**: Trình soạn thảo mã nguồn DSL chuyên nghiệp (CodeMirror 6 / Monaco Editor) với IntelliSense tự động gợi ý và nút **✨ Format Code** chuẩn hóa cú pháp.

### 2. Hệ Thống Khối Lệnh Cấu Trúc (Control Flow Containers)
- **IfElseGroup**: Container 2 ngăn độc lập (Làn True màu Emerald & Làn Else màu Rose), tự động bắt dính các bước con khi thả vào container.
- **WhileGroup & ForGroup**: Khối vòng lặp hiển thị biểu thức điều kiện và giới hạn số vòng lặp an toàn (`maxIterations`, `delayMs`).

### 3. Execution Console & Logging Thời Gian Thực
- **Gom nhóm log theo Step**: Log thực thi được nhóm theo từng bước rõ ràng.
- **Sub-Test Case Tree**: Danh sách bước con của Sub-Test Case được gom nhóm có thể thu gọn/mở rộng.
- **Ảnh chụp màn hình (Screenshot Lightbox)**: Tự động lưu ảnh chụp màn hình khi chạy test hoặc khi gặp lỗi, hỗ trợ xem trực tiếp thumbnail và phóng to ảnh gốc.
- **Truyền phát kép (SSE & WebSocket)**: Cập nhật tiến độ chạy từng node theo thời gian thực qua Server-Sent Events và WebSocket (`/ws/live-log`).

### 4. Tiện Ích Ghi Thao Tác Trình Duyệt (Chrome Web Recorder Extension)
- Tiện ích mở rộng Manifest V3 (TestCase Studio Edition) tự động ghi nhận các thao tác `click`, `input`, `hover`, trích xuất Selector thông minh đa lớp và đồng bộ trực tiếp vào Canvas.

### 5. Bảo Mật & Bản Quyền Doanh Nghiệp (Enterprise Licensing & RBAC)
- **Mật mã bất đối xứng RSA-2048**: Ký số và xác thực file bản quyền `.lic` kết hợp khóa phần cứng máy chủ (HWID SHA-256 Fingerprint).
- **Làm rối mã nguồn (ProGuard Obfuscation)**: Bảo vệ toàn bộ bytecode nghiệp vụ cốt lõi `com.bank.automation.core.security.**`.
- **Phân quyền 6 vai trò**: `ADMIN`, `IT_SUPPORT`, `APPROVER`, `DEVELOPER`, `TESTER`, `VIEWER`.
- **Router Guards**: `authGuard`, `guestGuard`, `roleGuard`, `authInterceptor` (tự động gắn JWT Token và bắt lỗi `401`).

---

## 🏗️ Kiến Trúc Hệ Thống (Architecture)

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                   KIẾN TRÚC TỔNG THỂ                                   │
└────────────────────────────────────────────────────────────────────────────────────────┘

    [ CLIENT: ANGULAR 21 SPA + TAILWIND CSS v4 ]
    ├─ Diagram Canvas (ng-diagram 1.3, Dagre Auto-Layout, Custom Group Nodes)
    ├─ Table Step View (CDK Drag-Drop, Hierarchical Tree)
    ├─ Script Editor (CodeMirror 6 / Monaco, TypeScript DSL)
    ├─ AST Flow Engine (2-Way Lossless AST Synchronizer)
    ├─ Chrome Web Recorder Extension (Manifest V3 Companion)
    └─ Router Security (authGuard, guestGuard, roleGuard, authInterceptor)
                 │
                 ▼ (HTTPS / RESTful JSON / SSE / WebSocket)
    [ BACKEND GATEWAY: SPRING BOOT 2.7.18 LTS / JAVA 11 LTS (Port: 9000) ]
    ├─ Security & JWT Auth Service & Multi-Tenant RBAC
    ├─ Project & Test Case Management (Metadata vs Flow DAG JSON Separation)
    ├─ Enterprise Licensing & RSA-2048 / HWID Verifier (ProGuard Obfuscated)
    ├─ AI Flow Generator (OpenAI / Claude / Custom API)
    └─ Coordinator & Task Dispatcher
                 │
                 ├──────────────────────────────┬──────────────────────────────┐
                 ▼                              ▼                              ▼
          [ REDIS 7.2 ]                   [ MYSQL 8.0 ]               [ WORKER POOL ]
       Pub/Sub Event Bus &           Persistent DB Storage       Playwright & REST Runners
       Task Distributed Queue        (Projects, Tests, Logs)     (Worker 1, Worker 2, ...)
```

---

## 🚀 Hướng Dẫn Cài Đặt & Khởi Chạy (Quick Start)

### 1. Yêu cầu môi trường
- **Docker** & **Docker Compose** đã cài đặt.
- **Node.js 20+** (nếu chạy Frontend độc lập).
- **Java 11 LTS / Maven 3.8+** (nếu chạy Backend độc lập).

### 2. Khởi chạy toàn bộ hệ thống bằng Docker Compose
Tại thư mục gốc của dự án, chạy lệnh:

```bash
docker compose up -d --build
```

### 3. Danh sách các dịch vụ & Cổng truy cập
| Dịch Vụ | Cổng Truy Cập | Mô Tả |
| :--- | :--- | :--- |
| **Frontend Web App** | `http://localhost:4200` hoặc `http://localhost` | Giao diện làm việc chính của người dùng |
| **Backend Gateway API** | `http://localhost:9000` | REST API Gateway và Swagger Docs |
| **MySQL Database** | `localhost:3306` | Cơ sở dữ liệu chính (`automation_db`) |
| **Redis Server** | `localhost:6379` | Hàng đợi tác vụ và Pub/Sub logs / License sync |
| **Chrome Extension** | `./chrome-extension` | Cài đặt dạng Unpacked qua `chrome://extensions` |

---

## 📚 Tài Liệu Kỹ Thuật Chi Tiết
Để xem tài liệu kiến trúc chuyên sâu, bộ danh mục Action Catalog và đặc tả thuật toán AST Flow Engine, vui lòng tham khảo:
- [`SYSTEM_ARCHITECTURE_DOCUMENTATION.md`](./SYSTEM_ARCHITECTURE_DOCUMENTATION.md) — Đặc tả kiến trúc hệ thống toàn diện chuẩn Enterprise.
- [`SYSTEM_DOCUMENTATION.md`](./SYSTEM_DOCUMENTATION.md) — Đặc tả chi tiết toàn bộ tính năng và danh mục thao tác.
- [`PRODUCT_BUSINESS_DOCUMENTATION.md`](./PRODUCT_BUSINESS_DOCUMENTATION.md) — Tài liệu nghiệp vụ & yêu cầu sản phẩm (PRD/BRD).
- [`Backend/BACKEND_SYSTEM_ARCHITECTURE.md`](./Backend/BACKEND_SYSTEM_ARCHITECTURE.md) — Kiến trúc chi tiết Backend, Database Schema & Orchestrator.
- [`Backend/docs/DEPLOYMENT_AND_LICENSE_GUIDE.md`](./Backend/docs/DEPLOYMENT_AND_LICENSE_GUIDE.md) — Hướng dẫn đóng gói bàn giao & quản trị bản quyền.
