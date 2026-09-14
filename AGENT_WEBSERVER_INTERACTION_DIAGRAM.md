# Sơ Đồ Tương Tác Agent — Web Server — Frontend

> Tài liệu mô tả toàn diện cơ chế giao tiếp giữa **FlowBuilder Agent** (Golang), **Backend Gateway** (Spring Boot), **Frontend Web UI** (Angular 21), **Worker Pool** và các hệ thống phụ trợ.

---

## 1. Kiến Trúc Tổng Thể (High-Level Architecture)

```mermaid
graph TB
    subgraph TESTER_MACHINE["🖥️ Máy Trạm Tester"]
        UI["🌐 Web UI<br/>(Angular 21 SPA)<br/>Port 80/443"]
        AGENT["⚙️ FlowBuilder Agent<br/>(Golang Daemon)<br/>Port 8765"]
        CHROME["🔍 Chrome Browser<br/>(CDP Debug Port)"]
        PHONE["📱 Mobile Device<br/>(ADB)"]
    end

    subgraph SERVER_CLOUD["☁️ Server / Docker Compose"]
        NGINX["🔀 Nginx Reverse Proxy<br/>Port 80/443"]
        GW["🏢 Backend Gateway<br/>(Spring Boot)<br/>Port 9000"]
        WORKER["👷 Worker Pool<br/>(Spring Boot × N)<br/>Port 8081"]
        MYSQL[("🗄️ MySQL 8.0<br/>Port 3306")]
        REDIS[("⚡ Redis 7.2<br/>Port 6379")]
    end

    subgraph EXTERNAL["🌍 Bên Ngoài"]
        SUT["🎯 Hệ Thống Đích<br/>(SUT)"]
        AI["🤖 AI Providers<br/>(Anthropic/OpenAI/<br/>Gemini)"]
    end

    UI <-->|"WebSocket + HTTP REST<br/>ws://127.0.0.1:8765/ws"| AGENT
    UI -->|"HTTPS REST API<br/>(JWT Auth)"| NGINX
    NGINX -->|"Reverse Proxy<br/>/api/*"| GW
    GW <-->|"Pub/Sub + Task Queue"| REDIS
    GW <-->|"JPA/Hibernate"| MYSQL
    WORKER <-->|"Pub/Sub + Task Queue"| REDIS
    WORKER <-->|"JPA/Hibernate"| MYSQL
    AGENT -->|"HTTP Load Test<br/>(Zero Backend)"| SUT
    AGENT -->|"CDP Protocol"| CHROME
    AGENT -->|"ADB Commands"| PHONE
    AGENT -->|"HTTPS REST"| AI
    AGENT -.->|"Version Check +<br/>Auto-Update .EXE"| GW
    WORKER -->|"Playwright/REST"| SUT
    GW -->|"SSE + WebSocket<br/>/ws/live-log"| UI
```

---

## 2. Mô Hình Tam Giác — Triangular Topology

> **Nguyên tắc thiết kế cốt lõi**: Backend Gateway **KHÔNG** kết nối trực tiếp tới Agent. Web UI đóng vai trò **nhạc trưởng (Orchestrator)** kết nối đồng thời tới cả Backend và Agent.

```mermaid
graph LR
    subgraph TRIANGLE["Mô Hình Tam Giác Điều Phối"]
        direction TB
        WEB["🌐 Web UI<br/>(Orchestrator)"]
        BE["🏢 Backend Gateway<br/>(Data Store)"]
        AG["⚙️ Agent<br/>(Local Executor)"]
    end

    WEB -->|"① Lấy kịch bản<br/>GET /api/test-cases/{id}"| BE
    WEB -->|"② Nạp kịch bản<br/>PRELOAD_SCENARIO (WS)"| AG
    AG -->|"③ Bắn tải SUT<br/>(Zero Backend Call)"| SUT["🎯 SUT"]
    AG -->|"④ Stream Metrics<br/>METRICS_TICK (WS 1s)"| WEB
    WEB -->|"⑤ Lưu kết quả<br/>POST /api/load-test/save"| BE
```

### Giải thích chiều dữ liệu:

| Bước | Chiều | Giao thức | Mô tả |
|:---:|:---|:---|:---|
| ① | Web UI → Backend | HTTP REST (JWT) | Lấy kịch bản test, metadata, dataset từ CSDL |
| ② | Web UI → Agent | WebSocket / HTTP | Nạp trọn bộ kịch bản + CSV vào RAM Agent |
| ③ | Agent → SUT | HTTP/1.1 & HTTP/2 | Sinh tải cực hạn, **không gọi về Backend** |
| ④ | Agent → Web UI | WebSocket | Stream telemetry RPS, Latency p50/90/95/99 mỗi 1s |
| ⑤ | Web UI → Backend | HTTP REST (JWT) | Chuyển tiếp FINAL_REPORT để lưu trữ vào MySQL |

---

## 3. Giao Thức & Cổng Kết Nối Chi Tiết

```mermaid
graph LR
    subgraph PROTOCOLS["Bảng Giao Thức"]
        direction TB
        P1["HTTP/1.1 REST"]
        P2["WebSocket RFC 6455"]
        P3["SSE (Server-Sent Events)"]
        P4["Redis Pub/Sub"]
        P5["Chrome DevTools Protocol"]
        P6["ADB (Android Debug Bridge)"]
    end
```

| Kết nối | Cổng | Giao thức | Xác thực | Mô tả |
|:---|:---:|:---|:---|:---|
| Web UI ↔ Agent | `8765` | HTTP REST + WebSocket | Token `flowbuilder-agent-secret` | Điều khiển local, chỉ bind `127.0.0.1` |
| Web UI → Nginx | `80/443` | HTTPS REST | — | Static SPA + Reverse Proxy |
| Nginx → Gateway | `9000` | HTTP REST | Forward headers | Proxy `/api/*` requests |
| Web UI ↔ Gateway | `9000` | REST + SSE + WebSocket | JWT Bearer Token | API chính, live-log streaming |
| Gateway ↔ Redis | `6379` | Redis Protocol | Password auth | Pub/Sub event bus + Task Queue |
| Gateway ↔ MySQL | `3306` | JDBC/MySQL | Username/Password | Persistent storage |
| Worker ↔ Redis | `6379` | Redis Protocol | Password auth | Nhận task, publish kết quả |
| Worker ↔ MySQL | `3306` | JDBC/MySQL | Username/Password | Đọc/ghi kết quả test |
| Agent → Chrome | Dynamic | CDP (HTTP + WS) | — | Ghi/phát kịch bản web |
| Agent → Mobile | USB/TCP | ADB Protocol | — | Automation thiết bị Android |
| Agent → AI | `443` | HTTPS REST | API Key | Gọi LLM providers |
| Agent → Gateway | `9000` | HTTP REST | Public (permitAll) | Chỉ kiểm tra version & tải .EXE |

---

## 4. Luồng Tương Tác WebSocket Agent (Chi tiết)

```mermaid
sequenceDiagram
    participant UI as 🌐 Web UI
    participant AG as ⚙️ Agent (ws://127.0.0.1:8765/ws)
    participant SUT as 🎯 SUT

    Note over UI, AG: Kết nối WebSocket Song Công (Full Duplex)

    rect rgb(230, 245, 255)
        Note over UI, AG: 📡 Pha 1: Khám Phá & Đồng Bộ
        UI->>AG: PING
        AG-->>UI: PONG {version, cpu, ram, engineState, license}
        UI->>AG: SET_LICENSE {payload, signature}
        AG-->>UI: LICENSE_STATUS {valid, maxVUs, expiry}
        UI->>AG: SYNC_SERVER {gatewayUrl, version}
    end

    rect rgb(230, 255, 230)
        Note over UI, AG: 📦 Pha 2: Nạp Kịch Bản (Preload)
        UI->>AG: PRELOAD_SCENARIO {vus, duration, scenarios, steps, csvData}
        AG-->>UI: PRELOAD_COMPLETED {scenarioCount, dataRowCount}
    end

    rect rgb(255, 245, 230)
        Note over UI, SUT: 🚀 Pha 3: Sinh Tải & Giám Sát
        UI->>AG: START_LOAD_TEST
        AG-->>UI: LOAD_TEST_STARTED
        loop Mỗi 1 giây
            AG->>SUT: HTTP Requests (GET/POST/PUT/DELETE)
            AG-->>UI: METRICS_TICK {rps, latency_p50/p90/p95/p99, activeVUs, errors, throughput}
        end
    end

    rect rgb(255, 230, 230)
        Note over UI, AG: 🛑 Pha 4: Dừng & Báo Cáo
        UI->>AG: STOP_LOAD_TEST
        AG-->>UI: LOAD_TEST_STOPPED
        AG-->>UI: FINAL_REPORT {totalRequests, avgLatency, errorRate, percentiles...}
    end
```

---

## 5. Luồng Tương Tác Web Recording (CDP)

```mermaid
sequenceDiagram
    participant UI as 🌐 Web UI
    participant AG as ⚙️ Agent
    participant CR as 🔍 Chrome (CDP)

    rect rgb(240, 235, 255)
        Note over UI, CR: 🎬 Luồng Ghi Kịch Bản Web
        UI->>AG: START_RECORDING {url}
        AG->>CR: Chrome Launch (--remote-debugging-port)
        AG->>CR: CDP: Page.navigate(url)
        AG-->>UI: RECORDING_STARTED

        Note over AG, CR: Tester thao tác trên trình duyệt...
        CR-->>AG: CDP Events (click, input, navigate)
        AG-->>UI: RECORDED_STEP {action, selector, value, screenshot}

        loop Screencast 60 FPS
            CR-->>AG: CDP: Page.screencastFrame
            AG-->>UI: SCREENCAST_FRAME {base64_jpeg}
        end

        UI->>AG: INSPECT_ELEMENT {x, y}
        AG->>CR: CDP: DOM.getNodeForLocation(x, y)
        AG-->>UI: ELEMENT_INSPECTED {tagName, selectors, attributes}

        UI->>AG: PLAY_STEP {action, selector, value}
        AG->>CR: CDP: Runtime.evaluate / Input.dispatchEvent
        AG-->>UI: STEP_TEST_RESULT {success, screenshot}

        UI->>AG: STOP_RECORDING
        AG->>CR: CDP: Disconnect
        AG-->>UI: RECORDING_STOPPED {totalSteps}
    end
```

---

## 6. Luồng Gateway — Worker Pool (Redis Pub/Sub)

```mermaid
sequenceDiagram
    participant UI as 🌐 Web UI
    participant GW as 🏢 Gateway (Port 9000)
    participant RD as ⚡ Redis
    participant W1 as 👷 Worker 1
    participant W2 as 👷 Worker 2
    participant DB as 🗄️ MySQL
    participant SUT as 🎯 SUT

    rect rgb(245, 240, 255)
        Note over UI, SUT: ▶️ Luồng Thực Thi Test Case trên Server
        UI->>GW: POST /api/automation/run/{id}
        GW->>GW: runEventService.nextRunId() → createEmitter(runId)
        GW->>RD: RPUSH automation:job_queue {JobTaskMessage}
        GW-->>UI: SSE emitter created, 200 OK {runId}

        Note over RD, W2: Competing Consumer Pattern (Atomic Pop)
        W1->>RD: BLPOP automation:job_queue (2s timeout)
        RD-->>W1: JobTaskMessage {runId, testCase, flowDAG}

        loop Từng Step trong Flow DAG
            W1->>SUT: Playwright action / REST call / DB Query
            SUT-->>W1: Response
            W1->>RD: PUBLISH automation:events:{runId} {step, log, screenshot}
            RD-->>GW: PatternTopic automation:events:* → onMessage()
            GW-->>UI: SSE event: step/log (qua SseEmitter)
            W1->>RD: SET automation:run_status:{runId} "RUNNING" (TTL 30m)
        end

        W1->>DB: Cập nhật status=PASSED/FAILED + kết quả
        W1->>RD: PUBLISH automation:events:{runId} {event: "done"}
        RD-->>GW: Relay done event
        GW-->>UI: SSE event: done {status, durationSec}
    end

    rect rgb(255, 235, 235)
        Note over UI, W2: 🛑 Luồng Hủy Test Đang Chạy
        UI->>GW: POST /api/automation/runs/{runId}/stop
        GW->>RD: PUBLISH automation:control_channel {STOP_RUN, runId}
        RD-->>W1: WorkerJobConsumer.onMessage()
        W1->>W1: AutomationRunner.cancelRun(runId)
        W1-->>UI: SSE event: done {status: "CANCELLED"}
    end
```

### 6.1. Worker Grid Heartbeat & Discovery

```mermaid
sequenceDiagram
    participant W1 as 👷 Worker 1
    participant W2 as 👷 Worker 2
    participant RD as ⚡ Redis
    participant GW as 🏢 Gateway

    loop Mỗi 5 giây (@Scheduled)
        W1->>RD: SET automation:worker_heartbeat:{w1Id} {cpu, ram, slots, activeTasks} TTL=15s
        W2->>RD: SET automation:worker_heartbeat:{w2Id} {cpu, ram, slots, activeTasks} TTL=15s
    end

    GW->>RD: SCAN automation:worker_heartbeat:*
    RD-->>GW: Worker snapshots (loại bỏ stale > 20s)
    GW->>GW: WorkerGridService tổng hợp:<br/>onlineWorkers, totalSlots,<br/>availableSlots, healthSummary

    Note over GW: SuiteSchedulerService sử dụng<br/>availableSlots để phân batch<br/>Parallel vs Sequential execution
```

### 6.2. Bảng Redis Keys & Channels

| Redis Key / Channel | Loại | TTL | Mục đích |
|:---|:---|:---:|:---|
| `automation:job_queue` | List (RPUSH/BLPOP) | — | Hàng đợi task Competing Consumer |
| `automation:events:{runId}` | Pub/Sub Channel | — | Stream sự kiện step/log/done từ Worker → Gateway |
| `automation:control_channel` | Pub/Sub Channel | — | Broadcast lệnh STOP_RUN tới tất cả Workers |
| `automation:run_status:{runId}` | String (SET) | 30m | Cache trạng thái run (RUNNING/PASSED/FAILED) |
| `automation:worker_heartbeat:{workerId}` | String (SET) | 15s | Heartbeat CPU, RAM, slots từ Worker |
| `automation:license_channel` | Pub/Sub Channel | — | Đồng bộ license mới tới toàn cluster |
| `system:events:browser_engine_reload` | Pub/Sub Channel | — | Hot-reload browser engine (Chromium ↔ Obscura) |

### 6.3. SSE Stream — Frontend Nhận Kết Quả Thời Gian Thực

```mermaid
sequenceDiagram
    participant UI as 🌐 Web UI (EventSource)
    participant GW as 🏢 Gateway

    UI->>GW: GET /api/automation/runs/{runId}/stream?token={jwt}
    Note over GW: TokenAuthenticationFilter<br/>xác thực JWT từ query param<br/>(EventSource không hỗ trợ header)
    GW-->>UI: event: connected {runId: 1001}

    loop Nhận từ Redis automation:events:{runId}
        GW-->>UI: event: step {nodeId, stepName, status:"RUNNING", durationMs}
        GW-->>UI: event: log {logLine: "[BROWSER] Navigating..."}
        GW-->>UI: event: step {nodeId, stepName, status:"PASSED", durationMs:250}
    end

    GW-->>UI: event: done {status:"PASSED", durationSec:12}
    Note over UI: SseEmitter complete → đóng kết nối
```

---

## 7. Luồng Auto-Update Agent

```mermaid
sequenceDiagram
    participant AG as ⚙️ Agent v2.8.0
    participant GW as 🏢 Gateway
    participant NG as 🔀 Nginx

    rect rgb(255, 250, 230)
        Note over AG, NG: 🔄 Tự Động Cập Nhật Binary
        AG->>GW: GET /api/v1/platform/agent/version (timeout 1500ms)
        alt Có phiên bản mới (v2.8.1)
            GW-->>AG: {version: "2.8.1", downloadUrl: "..."}
            AG->>NG: GET /downloads/flowbuilder-agent-v2.8.1.exe
            NG-->>AG: Binary file (.exe)
            Note over AG: 1. Lưu → agent.exe.new<br/>2. Kiểm tra size > 1MB<br/>3. Rename cũ → agent.exe.old<br/>4. Rename mới → agent.exe<br/>5. Spawn process mới<br/>6. os.Exit(0)
        else Đã mới nhất
            GW-->>AG: {version: "2.8.0"}
            Note over AG: Bỏ qua, không cập nhật
        end
    end
```

---

## 8. Luồng Xác Thực License (Offline RSA-2048)

```mermaid
sequenceDiagram
    participant UI as 🌐 Web UI
    participant GW as 🏢 Gateway
    participant AG as ⚙️ Agent
    participant DB as 🗄️ MySQL

    rect rgb(240, 255, 245)
        Note over UI, AG: 🔐 Xác Thực Bản Quyền Offline
        UI->>GW: GET /api/license/current (JWT Auth)
        GW->>DB: Query license record
        DB-->>GW: {payload, signature}
        GW-->>UI: License data

        UI->>AG: SET_LICENSE {payload, signature} (WebSocket)
        Note over AG: Xác thực chữ ký RSA-2048<br/>crypto/rsa.VerifyPKCS1v15<br/>với Public Key nhúng sẵn<br/>(Hoàn toàn OFFLINE)
        AG-->>UI: LICENSE_STATUS {valid:true, maxVUs:100, expiry:"2027-01-01"}
    end
```

---

## 9. Luồng Mobile Automation

```mermaid
sequenceDiagram
    participant UI as 🌐 Web UI
    participant AG as ⚙️ Agent
    participant ADB as 📱 ADB
    participant DEVICE as 📲 Android Device

    rect rgb(255, 240, 245)
        Note over UI, DEVICE: 📱 Luồng Tự Động Hóa Mobile
        UI->>AG: MOBILE_GET_DEVICES
        AG->>ADB: adb devices
        ADB-->>AG: List of devices
        AG-->>UI: MOBILE_DEVICES [{serial, model, status}]

        UI->>AG: MOBILE_CAPTURE_SCREENSHOT {serial}
        AG->>ADB: adb -s {serial} exec-out screencap -p
        ADB->>DEVICE: Capture
        DEVICE-->>ADB: PNG data
        ADB-->>AG: PNG data
        AG-->>UI: MOBILE_SCREENSHOT {base64}

        UI->>AG: MOBILE_GET_HIERARCHY {serial}
        AG->>ADB: uiautomator dump
        DEVICE-->>AG: XML hierarchy
        AG-->>UI: MOBILE_HIERARCHY {xml/json}

        UI->>AG: MOBILE_EXECUTE_ACTION {serial, action:"tap", x:100, y:200}
        AG->>ADB: adb shell input tap 100 200
        ADB->>DEVICE: Touch event
        AG-->>UI: MOBILE_ACTION_RESULT {success:true}
    end
```

---

## 10. Bảng Tổng Hợp Tất Cả API Endpoints Của Agent

### 10.1. Hệ Thống & Bản Quyền

| Method | Endpoint | Mô tả |
|:---|:---|:---|
| `GET` | `/health` | Health check + Agent discovery (version, CPU, RAM, license) |
| `GET` | `/api/license` | Lấy trạng thái license hiện tại |
| `POST` | `/api/license` | Nạp và xác thực license RSA-2048 |
| `POST` | `/api/sync-server` | Đồng bộ thông tin Backend Gateway (auto-update) |

### 10.2. Load Test

| Method | Endpoint | Mô tả |
|:---|:---|:---|
| `POST` | `/api/preload` | Nạp kịch bản + dataset CSV vào RAM |
| `POST` | `/api/start` | Khởi động sinh tải |
| `POST` | `/api/stop` | Dừng khẩn cấp |

### 10.3. Smart API Testing

| Method | Endpoint | Mô tả |
|:---|:---|:---|
| `POST` | `/api/policy/ping` | Kiểm tra thông tuyến mạng/firewall |
| `POST` | `/api/testcases/batch-run` | Chạy batch test cases API |
| `POST` | `/api/http/execute` | Thực thi 1 HTTP request (bypass CORS) |

### 10.4. Chrome CDP Web Recording

| Method | Endpoint | Mô tả |
|:---|:---|:---|
| `GET` | `/api/chrome/launch` | Mở Chrome với CDP debug port |
| `GET` | `/api/chrome/shortcut` | Tạo desktop shortcut |
| `GET` | `/api/chrome/close` | Đóng Chrome |
| `GET` | `/api/record/start` | Bắt đầu ghi kịch bản |
| `GET` | `/api/record/stop` | Dừng ghi |
| `GET` | `/api/record/status` | Kiểm tra trạng thái ghi |
| `POST` | `/api/record/inspect-mode` | Bật/tắt inspect element |
| `POST` | `/api/record/inspect` | Lấy thông tin DOM element tại (x, y) |
| `POST` | `/api/record/highlight` | Highlight element trên trang |
| `POST` | `/api/record/clear-highlight` | Xóa highlight |
| `POST` | `/api/record/navigate` | Điều hướng URL |
| `GET` | `/api/record/back` | Quay lại trang trước |
| `GET` | `/api/record/forward` | Trang tiếp theo |
| `GET` | `/api/record/reload` | Tải lại trang |
| `POST` | `/api/record/click` | Click tại tọa độ (x, y) |
| `POST` | `/api/record/text` | Gõ văn bản |
| `POST` | `/api/record/key` | Gửi phím bấm (Enter, Tab...) |
| `GET` | `/api/record/screenshot` | Chụp ảnh màn hình trang |
| `POST` | `/api/record/play-step` | Chạy thử 1 bước test |

### 10.5. AI & LLM

| Method | Endpoint | Mô tả |
|:---|:---|:---|
| `GET` | `/api/ai/claude-settings` | Quét cấu hình Claude local |
| `POST` | `/api/ai/generate-flow` | Sinh test flow bằng AI (proxy tới LLM) |
| `POST` | `/api/ai/spec-heal` | Chữa lành kịch bản từ đặc tả BA |

### 10.6. Proxy Config

| Method | Endpoint | Mô tả |
|:---|:---|:---|
| `GET` | `/api/config/proxy` | Lấy cấu hình proxy hiện tại |
| `POST` | `/api/config/proxy` | Lưu cấu hình proxy |
| `POST` | `/api/config/proxy/test` | Test kết nối qua proxy |

### 10.7. Mobile Automation

| Method | Endpoint | Mô tả |
|:---|:---|:---|
| `GET` | `/api/mobile/status` | Trạng thái ADB |
| `GET` | `/api/mobile/devices` | Danh sách thiết bị kết nối |
| `GET` | `/api/mobile/current-app` | App đang chạy foreground |
| `GET` | `/api/mobile/packages` | Danh sách ứng dụng đã cài |
| `GET` | `/api/mobile/hierarchy` | Cây giao diện UI XML/JSON |
| `GET` | `/api/mobile/screenshot` | Chụp ảnh màn hình điện thoại |
| `POST` | `/api/mobile/action` | Tap, swipe, keyevent, text |
| `POST` | `/api/mobile/run-step` | Thực thi 1 bước test mobile |
| `POST` | `/api/mobile/maestro/run` | Chạy Maestro YAML flow |

---

## 11. WebSocket Message Types (JSON Protocol)

### Định dạng tin nhắn WebSocket:
```json
{
  "type": "<COMMAND hoặc EVENT>",
  "token": "flowbuilder-agent-secret",
  "payload": { ... },
  "timestamp": 1726310000000
}
```

### Commands (Web UI → Agent):

| Type | Nhóm | Mô tả |
|:---|:---|:---|
| `PING` | System | Kiểm tra trạng thái Agent |
| `GET_STATUS` | System | Lấy trạng thái engine |
| `SET_LICENSE` | License | Nạp license |
| `GET_LICENSE` | License | Lấy license hiện tại |
| `SYNC_SERVER` | System | Đồng bộ thông tin Gateway |
| `PRELOAD_SCENARIO` | Load Test | Nạp kịch bản |
| `START_LOAD_TEST` | Load Test | Bắt đầu sinh tải |
| `STOP_LOAD_TEST` | Load Test | Dừng sinh tải |
| `PING_POLICY` | API Test | Kiểm tra thông tuyến |
| `EXECUTE_API_BATCH` | API Test | Chạy batch API |
| `EXECUTE_HTTP_STEP` | API Test | Chạy 1 HTTP request |
| `START_RECORDING` | CDP | Bắt đầu ghi web |
| `STOP_RECORDING` | CDP | Dừng ghi |
| `PAUSE_RECORDING` | CDP | Tạm dừng |
| `CLOSE_BROWSER` | CDP | Đóng trình duyệt |
| `PLAY_STEP` | CDP | Chạy lại 1 bước |
| `INSPECT_ELEMENT` | CDP | Soi phần tử DOM |
| `HIGHLIGHT_ELEMENT` | CDP | Highlight element |
| `CLEAR_HIGHLIGHT` | CDP | Xóa highlight |
| `SET_INSPECT_MODE` | CDP | Bật/tắt inspect mode |
| `NAVIGATE` | CDP | Điều hướng URL |
| `NAVIGATE_BACK` | CDP | Quay lại |
| `NAVIGATE_FORWARD` | CDP | Trang tiếp |
| `RELOAD` | CDP | Tải lại |
| `DISPATCH_CLICK` | CDP | Click chuột |
| `DISPATCH_TEXT` | CDP | Gõ text |
| `DISPATCH_KEY` | CDP | Gửi phím |
| `CAPTURE_SCREENSHOT` | CDP | Chụp màn hình |
| `MOBILE_GET_DEVICES` | Mobile | Lấy danh sách thiết bị |
| `MOBILE_GET_HIERARCHY` | Mobile | Lấy cây UI |
| `MOBILE_CAPTURE_SCREENSHOT` | Mobile | Chụp màn hình mobile |
| `MOBILE_EXECUTE_ACTION` | Mobile | Thực thi hành động |
| `MOBILE_RUN_STEP` | Mobile | Chạy bước test mobile |

### Events (Agent → Web UI):

| Type | Nhóm | Mô tả |
|:---|:---|:---|
| `PONG` | System | Phản hồi trạng thái (version, CPU, RAM) |
| `LICENSE_STATUS` | License | Kết quả xác thực RSA |
| `PRELOAD_COMPLETED` | Load Test | Kịch bản đã nạp xong |
| `LOAD_TEST_STARTED` | Load Test | Đã kích hoạt Goroutines |
| `METRICS_TICK` | Load Test | Telemetry mỗi 1 giây (RPS, Latency, VUs) |
| `LOAD_TEST_STOPPED` | Load Test | Đã dừng test |
| `FINAL_REPORT` | Load Test | Báo cáo tổng kết |
| `BATCH_PROGRESS` | API Test | Tiến độ batch |
| `BATCH_TESTCASE_FINISHED` | API Test | 1 testcase hoàn tất |
| `BATCH_COMPLETED` | API Test | Batch hoàn tất |
| `HTTP_STEP_RESULT` | API Test | Kết quả HTTP request |
| `POLICY_PING_RESULT` | API Test | Kết quả kiểm tra tuyến |
| `RECORDED_STEP` | CDP | Bước vừa ghi được |
| `RECORDING_STOPPED` | CDP | Đã dừng ghi |
| `STEP_TEST_RESULT` | CDP | Kết quả chạy thử bước |
| `SCREENCAST_FRAME` | CDP | JPEG Base64 frame (60 FPS) |
| `ELEMENT_INSPECTED` | CDP | Thông tin DOM element |
| `MOBILE_DEVICES` | Mobile | Danh sách thiết bị |
| `MOBILE_HIERARCHY` | Mobile | Cây giao diện |
| `MOBILE_SCREENSHOT` | Mobile | Ảnh màn hình Base64 |
| `MOBILE_ACTION_RESULT` | Mobile | Kết quả hành động |
| `MOBILE_STEP_RESULT` | Mobile | Kết quả bước test |
| `ERROR` | System | Lỗi runtime |

---

## 12. Sơ Đồ Deployment (Docker Compose)

```mermaid
graph TB
    subgraph DOCKER["🐳 Docker Compose Network: automation-net"]
        direction TB

        subgraph DATA_LAYER["💾 Data Layer"]
            MYSQL[("🗄️ MySQL 8.0<br/>automation-mysql<br/>expose: 3306<br/>mem: 2GB, cpu: 1.0")]
            REDIS[("⚡ Redis 7.2 Alpine<br/>automation-redis<br/>expose: 6379<br/>mem: 512MB, cpu: 0.5")]
        end

        subgraph APP_LAYER["🔧 Application Layer"]
            GW["🏢 Gateway<br/>automation-gateway<br/>APP_ROLE=GATEWAY<br/>port: 9000<br/>mem: 2GB, cpu: 2.0"]
            W1["👷 Worker 1<br/>APP_ROLE=WORKER<br/>port: 8081<br/>mem: 4GB, cpu: 2.0"]
            W2["👷 Worker 2<br/>APP_ROLE=WORKER<br/>port: 8081<br/>mem: 4GB, cpu: 2.0"]
        end

        subgraph WEB_LAYER["🌐 Web Layer"]
            FE["🖥️ Frontend<br/>automation-frontend<br/>Nginx + Angular SPA<br/>ports: 80, 443<br/>mem: 256MB, cpu: 0.5"]
        end

        FE -->|"depends_on<br/>(healthy)"| GW
        GW -->|"depends_on<br/>(healthy)"| MYSQL
        GW -->|"depends_on<br/>(healthy)"| REDIS
        W1 -->|"depends_on<br/>(healthy)"| GW
        W2 -->|"depends_on<br/>(healthy)"| GW
    end

    subgraph LOCAL["🖥️ Tester Machine (Ngoài Docker)"]
        AGENT["⚙️ FlowBuilder Agent<br/>127.0.0.1:8765<br/>Golang native .EXE"]
    end

    AGENT -.->|"Auto-update<br/>GET /api/v1/platform/agent/version"| GW
```

---

## 13. Bảo Mật & Xác Thực (Security Overview)

```mermaid
graph LR
    subgraph AUTH["🔐 Các Cơ Chế Xác Thực"]
        direction TB
        JWT["JWT Bearer Token<br/>(Web UI ↔ Gateway)"]
        RSA["RSA-2048 Offline<br/>(License Verification)"]
        TOKEN["Agent Secret Token<br/>(Web UI ↔ Agent)"]
        REDIS_PW["Redis Password<br/>(Internal Network)"]
        MYSQL_PW["MySQL Credentials<br/>(Internal Network)"]
    end

    JWT --- NOTE1["Spring Security<br/>authInterceptor tự gắn JWT<br/>roleGuard kiểm tra 6 vai trò:<br/>ADMIN, IT_SUPPORT, APPROVER,<br/>DEVELOPER, TESTER, VIEWER"]

    RSA --- NOTE2["Public Key nhúng trong Agent<br/>Xác thực OFFLINE 100%<br/>Kiểm tra: maxVUs, expiry, HWID"]

    TOKEN --- NOTE3["Mặc định: flowbuilder-agent-secret<br/>Cấu hình: -token hoặc AGENT_TOKEN<br/>Chỉ bind 127.0.0.1 (loopback)"]
```

---

> **Cập nhật lần cuối**: 2026-09-14
