# RUN_LOCAL.md – Hướng dẫn chạy Lab 04 - Alert Notification Service

Tài liệu này giúp người khác clone repo sạch và chạy lại Alert Notification service trong Docker.

---

## 1. Clone repo

```bash
git clone <repo-url>
cd FIT4110_lab04_docker_packaging
```

---

## 2. Cài dependencies cho Newman/Prism/Spectral

```bash
npm install
```

---

## 3. Build Docker image

```bash
docker build -t alert-notification:lab04 .
```

---

## 4. Run container

```bash
docker run --rm \
  --name alert-notification-lab04 \
  -p 8000:8000 \
  --env-file .env.example \
  alert-notification:lab04
```

Mở terminal khác, kiểm tra:

```bash
curl http://localhost:8000/health
```

Kết quả mong đợi:

```json
{
  "status": "ok",
  "service": "alert-notification",
  "version": "1.0.0"
}
```

---

## 5. API Endpoints

### Health Check
```bash
GET http://localhost:8000/health
```

### Alerts Management
```bash
# List all alerts
GET http://localhost:8000/api/alerts?severity=WARNING&status=ACTIVE

# Get alert by ID
GET http://localhost:8000/api/alerts/{alertId}

# Acknowledge alert
PATCH http://localhost:8000/api/alerts/{alertId}/acknowledge

# Resolve alert
PATCH http://localhost:8000/api/alerts/{alertId}/resolve
```

### Event Ingestion (trigger alerts)
```bash
POST http://localhost:8000/events
Content-Type: application/json
Authorization: Bearer local-dev-token

{
  "eventType": "SENSOR_READING",
  "eventId": "uuid",
  "deviceId": "device-001",
  "metric": "temperature",
  "value": 85,
  "unit": "celsius",
  "timestamp": "2026-05-26T10:30:00Z"
}
```

### Notifications Management
```bash
# Send notification
POST http://localhost:8000/api/notifications
Content-Type: application/json
Authorization: Bearer local-dev-token

# List notifications
GET http://localhost:8000/api/notifications
```

---

## 6. Chạy Newman test trên container

```bash
npm run test:local
```

Report sinh tại:

```text
reports/newman-lab04-local.xml
reports/newman-lab04-local.html
```

---

## 7. Dừng container

Nếu không dùng `--rm` hoặc container còn chạy:

```bash
docker stop alert-notification-lab04
```

---

## 8. Lệnh nhanh

```bash
make build
make run
make test-docker
make stop
```

---

## 9. Troubleshooting

- **Port 8000 bận**: Thay `-p 8000:9000` và sửa .env.example `APP_PORT=9000`
- **Database error**: Kiểm tra `DATABASE_URL` trong .env.example
- **Auth fail**: Sử dụng `Authorization: Bearer local-dev-token` trong request headers
