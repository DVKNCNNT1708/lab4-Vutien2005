# FIT4110_lab04_docker_packaging

**Học phần:** FIT4110 – Dịch vụ kết nối và Công nghệ nền tảng  
**Buổi 4:** Đóng gói service với Docker & tư duy công nghệ nền tảng  
**Case study:** Smart Campus Operations Platform  
**Repo nền:** `FIT4110_lab03_postman_mock_testing`

> Lab 03 đã có OpenAPI contract, Postman Collection, Mock Server và Newman report.  
> Lab 04 dùng lại logic đó để kiểm tra một điều mới: **service có chạy ổn khi được đóng gói thành Docker container không?**

---

## 1. Ý tưởng nối tiếp từ Lab 03 sang Lab 04

Ở Lab 03, luồng làm việc là:

```text
OpenAPI Contract → Mock Server → Postman Test → Newman Report → CI Evidence
```

Ở Lab 04, luồng đó được mở rộng thành:

```text
OpenAPI Contract
→ Service thật
→ Dockerfile
→ Docker Image
→ Docker Container
→ Postman/Newman chạy lại trên container
→ Evidence
```

Lab 04 hiện đã đồng bộ lại với contract IoT của Lab 03 theo payload:

```json
{
  "device_id": "ESP32-LAB-A01",
  "metric": "temperature",
  "value": 31.5,
  "unit": "celsius",
  "timestamp": "2026-05-13T08:30:00+07:00"
}
```

Boundary dùng trong bài:

```text
temperature: -40 đến 80
```

Thông điệp chính của buổi học:

> Một API pass Postman trên máy cá nhân chưa đủ.  
> Service cần được đóng gói thành container để người khác có thể chạy lại nhất quán.

---

## 2. Mục tiêu sau buổi lab

Sau khi hoàn thành Lab 04, mỗi nhóm cần làm được:

- Viết được `Dockerfile` cho service của nhóm.
- Dùng `.dockerignore` để giảm context build.
- Tách cấu hình runtime qua `.env.example`.
- Không commit secret thật vào repo.
- Chạy app bằng user non-root trong container.
- Có `HEALTHCHECK` gọi `GET /health`.
- Build được Docker image.
- Run được container từ image.
- Chạy lại Postman Collection của Lab 03 trên container.
- Kiểm tra được functional, auth, negative, boundary và schema lỗi `ProblemDetails`.
- Xuất Newman report làm bằng chứng.
- Viết được `RUN_LOCAL.md` hướng dẫn người khác chạy lại trong 3–5 bước.

---

## 3. Cấu trúc repo

```text
FIT4110_lab04_docker_packaging/
├── README.md
├── RUN_LOCAL.md
├── Dockerfile
├── .dockerignore
├── .env.example
├── .gitignore
├── Makefile
├── package.json
├── requirements.txt
├── src/
│   └── iot_app/
│       ├── __init__.py
│       └── main.py
├── contracts/
│   └── iot-ingestion.openapi.yaml
├── postman/
│   ├── collections/
│   │   └── FIT4110_lab04_iot_docker.postman_collection.json
│   └── environments/
│       ├── FIT4110_lab04_mock.postman_environment.json
│       └── FIT4110_lab04_local.postman_environment.json
├── mock-data/
├── scripts/
├── docs/
├── checklists/
├── templates/
├── reports/
└── .github/
    └── workflows/
        └── docker-newman.yml
```

---

## 4. Chuẩn bị môi trường

Cần cài trước:

- Git
- Docker Desktop hoặc Docker Engine
- Node.js 20.x LTS
- npm
- Postman Desktop hoặc Postman Web

Cài dependencies phục vụ Prism, Spectral, Newman:

```bash
npm install
```

Kiểm tra:

```bash
docker --version
docker info
node --version
npx newman --version
npx prism --version
```

---

## 5. Chạy service local không dùng Docker

Cài Python dependencies:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Chạy API:

```bash
uvicorn iot_app.main:app --app-dir src --host 0.0.0.0 --port 8000
```

Kiểm tra:

```bash
curl http://localhost:8000/health
```

---

## 6. Build và chạy bằng Docker

Build image:

```bash
docker build -t fit4110/iot-ingestion:lab04 .
```

Run container:

```bash
docker run --rm \
  --name fit4110-iot-lab04 \
  -p 8000:8000 \
  --env-file .env.example \
  fit4110/iot-ingestion:lab04
```

Kiểm tra health:

```bash
curl http://localhost:8000/health
```

---

## 7. Chạy lại Postman Collection trên container

Chạy Newman với local environment:

```bash
npm run test:local
```

Hoặc dùng script:

```bash
bash scripts/run-newman.sh local
```

Report được sinh trong:

```text
reports/
```

---

## 7.1 Postman Environment (không hardcode)

Postman Collection không được hardcode `baseUrl`, `authToken` hoặc URL mock của service khác. Tất cả các giá trị cần đặt trong Postman Environment.

Biến môi trường bắt buộc (ví dụ `team-iot`):

- `env`: mock | local
- `baseUrl`: http://localhost:4010 (mock) or http://localhost:8000 (local)
- `authToken`: lab-token (mock) or local-dev-token (local)
- `teamName`: team-iot
- `aiVisionMockUrl`: http://localhost:4011

Ví dụ chạy Newman (mock):

```bash
npm run mock:iot
npm run test:mock
```

Ví dụ chạy Newman (local):

```bash
npm run test:local
```


## 8. Các lệnh nhanh bằng Makefile

```bash
make install
make lint
make mock
make test-mock
make build
make run
make test-docker
make stop
```

---

## 9. Bài làm của từng nhóm

Mỗi nhóm dùng repo này làm mẫu, sau đó thay phần IoT bằng service của mình.

| Nhóm | Cần thay đổi |
|---|---|
| `team-iot` | Có thể dùng mẫu này trực tiếp, mở rộng thêm endpoint từ Lab 03 |
| `team-camera` | Thay `src/` bằng Camera Stream service, thêm OpenCV headless |
| `team-gate` | Thay bằng Access Gate service, lưu ý biến môi trường DB |
| `team-vision` | Thay bằng AI Vision service, chuẩn bị model YOLOv8n hoặc mock model |
| `team-analytics` | Thay bằng Analytics service, chưa bắt buộc TimescaleDB trong Lab 04 |
| `team-core` | Thay bằng Core Business policy engine |
| `team-notify` | Thay bằng Notification service, không commit token thật |

---

## 10. Điều kiện hoàn thành Lab 04

Một nhóm được xem là hoàn thành khi:

- `Dockerfile` build được image.
- Image chạy được container.
- Container có `GET /health` trả `200`.
- Service chạy bằng non-root user.
- Có `.dockerignore`.
- Có `.env.example`.
- Có `RUN_LOCAL.md`.
- Chạy lại Postman/Newman pass trên container.
- Có test cho functional, auth, negative, boundary.
- Error response trả đúng dạng `ProblemDetails`.
- Có report trong `reports/`.
- Có bằng chứng image tag đúng quy ước.

Tag gợi ý:

```text
v0.1.0-<team>
```

Ví dụ:

```bash
docker tag fit4110/iot-ingestion:lab04 ghcr.io/<owner>/team-iot:v0.1.0-team-iot
```

---

## 11. Artefact cần nộp

```text
Dockerfile
.dockerignore
.env.example
RUN_LOCAL.md
contracts/<team>.openapi.yaml
postman/collections/<team>.postman_collection.json
postman/environments/<team>_local.postman_environment.json
reports/newman-lab04-local.xml
reports/newman-lab04-local.html
ảnh chụp /health hoặc log container
tag image đã push lên registry
```

---

## 12. Rubric gợi ý

| Tiêu chí | Điểm |
|---|---:|
| Dockerfile đúng, build được | 2.0 |
| Container chạy được và `/health` pass | 2.0 |
| Non-root, `.dockerignore`, `.env.example` tốt | 2.0 |
| Newman/Postman test pass trên container | 2.0 |
| RUN_LOCAL.md rõ ràng, người khác chạy lại được | 1.0 |
| Evidence đầy đủ: log/report/image tag | 1.0 |
| **Tổng** | **10.0** |

---

## 13. Tinh thần của buổi học

Sau Buổi 3, nhóm đã chứng minh:

```text
API đúng contract khi kiểm thử bằng Postman/Newman.
```

Sau Buổi 4, nhóm cần chứng minh thêm:

```text
API đó có thể được đóng gói, chạy lại và kiểm thử trong container.
```

Đây là bước đệm trực tiếp cho Buổi 5:

```text
Docker container đơn lẻ → Docker Compose nhiều service → Plug-a-thon.

---

## Appendix: Postman / Prism / Newman & Lab 03 checklist

5. Quy định về Postman Environment

Collection không được hardcode:

- `baseUrl`
- `authToken`
- URL mock của service khác

Tất cả các giá trị này phải đặt trong Postman Environment.

Biến môi trường bắt buộc

| Biến | Mock environment | Local environment | Ý nghĩa |
|---|---|---|---|
| env | mock | local | Môi trường đang chạy test |
| baseUrl | http://localhost:4010 | http://localhost:8000 | URL service chính |
| authToken | lab-token | local-dev-token | Token hoặc API key |
| teamName | team-iot | team-iot | Tên nhóm hoặc tên service |
| aiVisionMockUrl | http://localhost:4011 | http://localhost:4011 | URL mock của service phụ thuộc |

Ví dụ URL trong Postman:

```
{{baseUrl}}/readings
```

Ví dụ Authorization header:

```
Authorization: Bearer {{authToken}}
```

6. Chạy Mock Server

Contract mẫu của IoT Ingestion nằm tại:

[contracts/iot-ingestion.openapi.yaml](contracts/iot-ingestion.openapi.yaml)

Chạy mock IoT:

```bash
npm run mock:iot
```

Mock server mặc định chạy tại: `http://localhost:4010`

Kiểm tra mock server:

```bash
curl http://localhost:4010/health
```

Nếu cần chạy mock AI Vision cho consumer-side smoke test:

```bash
npm run mock:vision
```

Mock AI Vision có thể chạy tại: `http://localhost:4011`

7. Lưu ý về Prism Mock Server

Prism Mock Server giúp kiểm thử sớm khi service thật chưa hoàn thiện. Tuy nhiên, cần phân biệt rõ giữa Mock Server và Service thật:

| Mock Server | Service thật |
|---|---|
| Trả response theo OpenAPI example | Chạy logic thật |
| Có thể mô phỏng status code | Tự xử lý nghiệp vụ |
| Không kiểm chứng database thật | Có database thật |
| Không chứng minh auth thật | Có auth thật |
| Không dùng để đo latency thật | Có thể kiểm thử hiệu năng cơ bản |

Header `Prefer: code=401` là tính năng hỗ trợ mock response của Prism — không dùng để chứng minh auth thật. Test auth đúng phải gửi request thiếu token hoặc token sai.

Ví dụ test trong Postman (auth):

```javascript
pm.test("Unauthorized request returns 401 or 403", function () {
  pm.expect([401, 403]).to.include(pm.response.code);
});
```

8. Chạy Postman Collection bằng Newman

Chạy với mock environment:

```bash
npm run test:mock
```

Chạy với local environment:

```bash
npm run test:local
```

Report sẽ được xuất vào thư mục `reports/`.

Ví dụ chạy Newman trực tiếp:

```bash
npx newman run postman/collections/FIT4110_lab04_iot_docker.postman_collection.json \
  -e postman/environments/teams/team-iot_mock.postman_environment.json \
  -r cli,junit,htmlextra \
  --reporter-junit-export reports/newman-report.xml \
  --reporter-htmlextra-export reports/newman-report.html
```

9. Cấu trúc Postman Collection

Mỗi collection cần có các folder sau:

- `01_Functional`
- `02_Auth`
- `03_Negative`
- `04_Boundary_Reliability`
- `05_Consumer_side_Smoke`
- `06_Local_only_NonFunctional`

9.1 Functional

Kiểm thử các luồng hợp lệ.

Ví dụ assertion:

```javascript
pm.test("Status code is 201", function () {
  pm.response.to.have.status(201);
});

pm.test("Response has readingId", function () {
  const json = pm.response.json();
  pm.expect(json).to.have.property("readingId");
});
```

9.2 Auth

Kiểm thử request có token hợp lệ, thiếu token và token sai. Không ép mock trả 401 bằng `Prefer: code=401` để chứng minh auth.

9.3 Negative

Kiểm thử dữ liệu sai (thiếu field, sai kiểu, payload không hợp lệ).

Ví dụ:

```javascript
pm.test("Invalid payload returns client error", function () {
  pm.expect([400, 422]).to.include(pm.response.code);
});
```

9.4 Boundary / Reliability

Không viết test chỉ kiểm tra lại request body đã gửi. Test boundary cần kiểm tra response.

Ví dụ:

```javascript
pm.test("High temperature is accepted with warning or rejected as invalid", function () {
  pm.expect([201, 400, 422]).to.include(pm.response.code);

  if (pm.response.code === 201) {
    pm.expect(pm.response.headers.has("X-Warning")).to.equal(true);
  }

  if ([400, 422].includes(pm.response.code)) {
    const json = pm.response.json();
    pm.expect(json).to.have.property("detail");
  }
});
```

9.5 Consumer-side Smoke

Consumer-side smoke test dùng để kiểm tra service có thể gọi contract tối thiểu của service phụ thuộc. Ví dụ IoT cần gọi mock của AI Vision:

```
POST {{aiVisionMockUrl}}/detect
```

9.6 Local-only NonFunctional

Test latency/SLA chỉ chạy với service thật.

Ví dụ:

```javascript
if (pm.environment.get("env") === "local") {
  pm.test("Response time is below 1000ms on local service", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
  });
}
```

10. Alert Notification API (Lab 03 focus)

Contract chính (ví dụ) nằm trong `contracts/openapi.yaml`. Endpoints cần test:

- `GET /health`
- `GET /api/alerts`
- `GET /api/alerts/{alertId}`
- `PATCH /api/alerts/{alertId}/acknowledge`
- `PATCH /api/alerts/{alertId}/resolve`
- `POST /events`
- `POST /api/notifications`
- `GET /api/notifications`

Chạy mock server cho Alerts (nếu có):

```bash
npm run mock:alerts
```

11. Chạy Newman cho Alerts

```bash
npm run test:alerts
```

Report sẽ được tạo trong `reports/`.

12. Quy trình làm bài (tóm tắt)
- Lint OpenAPI bằng Spectral
- Import OpenAPI vào Postman và tạo collection theo folder bắt buộc
- Tạo environment mock và local
- Chạy Prism mock server
- Chạy Newman trên mock
- Chạy Newman trên local (service thật)
- Viết consumer-side smoke test gọi mock của ít nhất một service phụ thuộc
- Xuất report

13. Data-driven Testing

Sử dụng file trong `mock-data/` làm `--iteration-data` cho Newman. Ví dụ:

```bash
npx newman run postman/collections/FIT4110_lab04_iot_docker.postman_collection.json \
  -e postman/environments/teams/team-iot_mock.postman_environment.json \
  --iteration-data mock-data/sensor-reading-valid.json
```

14. GitHub Actions (CI) - gợi ý

CI nên chạy: install deps → lint contract → start Prism → wait-on health → run Newman → upload reports. (Xem file workflow mẫu trong `.github/workflows/`)

15. Sản phẩm cần nộp & Điều kiện hoàn thành

Giữ nguyên các yêu cầu nộp & điều kiện hoàn thành như phần **Artefact** và **Điều kiện hoàn thành Lab 04** ở trên.

16. Lỗi thường gặp

- `ECONNREFUSED`: Mock server chưa chạy → `npm run mock:iot` và kiểm tra `/health`
- Newman chạy vào mock dù muốn local: kiểm tra env file và collection variables
- `401 Unauthorized` ở happy path: kiểm tra `authToken` và header Authorization
- Test pass trên mock nhưng fail trên local: service thật chưa validate auth hoặc chưa đúng contract

---

If you want, I can now:

- Verify and replace any hardcoded `http://localhost:4010` or tokens inside the collection JSON with `{{baseUrl}}` / `{{authToken}}`.
- Run `npx newman` locally here to generate a sample report (you'll need to run the mocks or the service).

Tell me which of those you'd like me to do next.
```
