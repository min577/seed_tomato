# 🍅 토마토 스마트팜 API 문서

**Version:** 1.0.0  
**Last Updated:** 2024-11-28  
**Author:** Seed Farm Development Team

---

## 📋 목차

1. [서버 구성](#서버-구성)
2. [실시간 데이터 API](#실시간-데이터-api)
3. [시장 가격 API](#시장-가격-api)
4. [카메라 제어 API](#카메라-제어-api)
5. [AI 분석 API](#ai-분석-api)
6. [수확량 예측 API](#수확량-예측-api)
7. [챗봇 API](#챗봇-api)
8. [에러 처리](#에러-처리)

---

## 🖥️ 서버 구성

| 서버 | URL | 설명 |
|------|-----|------|
| **n8n 워크플로우** | `http://seedfarm.co.kr:5678/webhook` | 메인 API 서버 (외부 접근) |
| **라즈베리파이** | `http://192.168.49.219:8080` | 카메라 스트리밍 (내부) |
| **수확량 예측** | `http://192.168.49.101:8002` | ML 서버 (내부) |
| **YOLO 분석** | `http://192.168.49.101:8001` | 객체 감지 (내부) |

---

## 📊 실시간 데이터 API

### GET /data-realtime
**실시간 토마토 데이터 조회**

InfluxDB에서 최근 1시간 내 마지막 YOLO 분석 결과를 반환합니다.

**Response:**
```json
{
  "Ready": 45,
  "Not_Ready": 23,
  "Disease_Bad": 2,
  "Truss": 8
}
```

| 필드 | 설명 |
|------|------|
| `Ready` | 수확 가능한 완숙 토마토 |
| `Not_Ready` | 미성숙 토마토 |
| `Disease_Bad` | 병해충 감염 토마토 |
| `Truss` | 화방 (꽃봉우리) |

---

### GET /data-history
**과거 데이터 조회**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `hours` | number | 24 | 조회 시간 범위 |

**Example:** `/data-history?hours=24`

**Response:**
```json
{
  "data": [
    {
      "time": "2024-11-21T06:00:00Z",
      "Ready": 40,
      "Not_Ready": 20,
      "Disease_Bad": 1,
      "Truss": 7
    }
  ]
}
```

---

### GET /data-summary
**일일 요약 데이터**

오늘 00:00부터 현재까지의 요약 데이터를 반환합니다.

**Response:**
```json
{
  "total_ready": 450,
  "total_not_ready": 230,
  "total_disease": 15,
  "avg_truss": 8
}
```

---

## 💰 시장 가격 API

### GET /price-compare
**도매가 vs 소매가 비교**

KAMIS 도매 시장가와 네이버 쇼핑 소매가를 비교합니다.

**Response:**
```json
{
  "success": true,
  "date": "2024-11-28",
  "wholesale_summary": {
    "high": 3500,
    "mid": 2800,
    "cherry": 8500
  },
  "online_summary": {
    "lowest_price": 5900,
    "lowest_mall": "쿠팡",
    "lowest_title": "완숙 토마토 1kg",
    "lowest_link": "https://...",
    "median_price": 7500,
    "average_price": 7200
  },
  "comparison": [
    {
      "grade": "상품",
      "wholesale_price": 3500,
      "online_lowest": 5900,
      "margin_rate": 68
    }
  ]
}
```

---

### GET /price-history
**가격 추이 조회**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start` | date | ✅ | 시작 날짜 (YYYY-MM-DD) |
| `end` | date | ✅ | 종료 날짜 (YYYY-MM-DD) |

**Example:** `/price-history?start=2024-11-21&end=2024-11-28`

**Response:**
```json
{
  "success": true,
  "period": {
    "start": "2024-11-21",
    "end": "2024-11-28"
  },
  "count": 7,
  "stats": {
    "high": {
      "min": 3200,
      "max": 3800,
      "avg": 3500,
      "change": 200
    }
  },
  "data": [
    {"date": "2024-11-21", "high": 3400, "mid": 2700},
    {"date": "2024-11-22", "high": 3500, "mid": 2800}
  ]
}
```

---

## 📷 카메라 제어 API

### GET /camera-status
**카메라 상태 조회**

```json
{
  "monitoring": true,
  "interval": 600,
  "last_capture": "2024-11-21T07:20:00Z",
  "white_balance": "auto"
}
```

---

### POST /camera-capture
**즉시 촬영**

라즈베리파이 카메라로 촬영 후 YOLO 분석을 수행합니다.

```json
{
  "success": true,
  "data": {
    "Ready": 45,
    "Not_Ready": 23,
    "Disease_Bad": 2,
    "Truss": 8,
    "ripeness_rate": 66,
    "disease_rate": 4
  },
  "timestamp": "2024-11-21T07:30:00Z"
}
```

---

### POST /capture-test
**테스트 촬영 (랜덤 데이터)**

실제 카메라 없이 랜덤 데이터를 생성하여 테스트합니다.

---

### POST /camera-start
**모니터링 시작**

설정된 간격으로 자동 촬영을 시작합니다.

---

### POST /camera-stop
**모니터링 중지**

---

### POST /camera-interval
**촬영 간격 설정**

**Request Body:**
```json
{
  "interval": 600
}
```

| 값 | 설명 |
|----|------|
| 60 | 1분 |
| 300 | 5분 |
| 600 | 10분 |
| 1800 | 30분 |
| 3600 | 1시간 |

---

### POST /camera-white-balance
**화이트 밸런스 설정**

**Request Body:**
```json
{
  "mode": "auto"
}
```

| 모드 | 설명 |
|------|------|
| `auto` | 자동 |
| `fluorescent` | 형광등 |
| `tungsten` | 백열등 |
| `daylight` | 주광 |

---

## 🤖 AI 분석 API

### POST /capture-analyze
**토마토 이미지 분석 (YOLO)**

**Request Body:**
```json
{
  "image": "base64_encoded_image...",
  "mimeType": "image/jpeg",
  "fileName": "capture.jpg"
}
```

**Response:**
```json
{
  "success": true,
  "message": "YOLO 분석 완료",
  "data": {
    "Ready": 12,
    "Not_Ready": 8,
    "Disease_Bad": 1,
    "Truss": 5,
    "ripeness_rate": 60,
    "disease_rate": 5
  }
}
```

---

### POST /disease-diagnosis
**병해충 AI 진단**

Gemini AI를 사용하여 병해충을 진단합니다.

**Request Body:**
```json
{
  "image": "base64_encoded_image...",
  "mimeType": "image/jpeg"
}
```

**Response:**
```json
{
  "success": true,
  "diagnosis": "🔬 진단 결과\n\n### 1. 🚦 건강 상태: 주의\n...",
  "healthStatus": "주의",
  "timestamp": "2024-11-21T07:30:00Z"
}
```

**진단 항목:**
- 건강 상태 평가 (건강/주의/위험)
- 발견된 증상
- 원인 분석
- 긴급 조치 사항
- 치료 방법
- 예방 관리

---

## 📈 수확량 예측 API

### POST /predict
**AI 수확량 예측**

> **Base URL:** `http://192.168.49.101:8002`

**모델 정보:**
- 알고리즘: Random Forest
- 정확도: R² = 0.9084
- 학습 데이터: 농진청 스마트팜 DB (103,000+ 건)

**Request Body:**
```json
{
  "month": 11,
  "temperature": 25.0,
  "humidity": 65.0,
  "co2": 700,
  "solar_radiation": 1200,
  "growth_stage": "생육중기",
  "facility_type": "비닐하우스"
}
```

| 필드 | Type | Required | Description |
|------|------|----------|-------------|
| `month` | int | ✅ | 예측 월 (1-12) |
| `temperature` | float | ✅ | 평균 온도 (°C) |
| `humidity` | float | ✅ | 평균 습도 (%) |
| `co2` | float | | CO₂ 농도 (ppm) |
| `solar_radiation` | float | | 일사량 (J/㎠) |
| `growth_stage` | string | | 생육초기/생육중기/생육후기/수확기 |
| `facility_type` | string | | 비닐하우스/유리온실 |

**Response:**
```json
{
  "predicted_yield": 18.5,
  "confidence_interval": {
    "lower": 16.2,
    "upper": 20.8,
    "std_dev": 1.2
  },
  "input_summary": {
    "month": 11,
    "temperature": 25.0,
    "humidity": 65.0
  },
  "recommendations": [
    "현재 온도가 적정 범위입니다.",
    "습도를 70% 수준으로 높이면 수확량 증가 가능"
  ]
}
```

---

## 💬 챗봇 API

### POST /chat-message
**AI 농업 상담 챗봇**

Gemini AI 기반 농업 상담 서비스입니다.

**Request (multipart/form-data):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `message` | string | ✅ | 사용자 메시지 |
| `image` | file | | 첨부 이미지 (선택) |

**Response:**
```json
{
  "response": "토마토 잎이 노랗게 변하는 원인은 여러 가지가 있습니다...",
  "timestamp": "2024-11-21T07:30:00Z",
  "success": true
}
```

---

## 🎥 라즈베리파이 직접 API

> **Base URL:** `http://192.168.49.219:8080`

### GET /stream
**실시간 MJPEG 스트리밍**

```html
<Image source={{ uri: 'http://192.168.49.219:8080/stream' }} />
```

### GET /snapshot
**스냅샷 이미지**

현재 프레임의 JPEG 이미지를 반환합니다.

---

## ❌ 에러 처리

### 에러 응답 형식
```json
{
  "success": false,
  "error": "에러 메시지",
  "code": "ERROR_CODE"
}
```

### HTTP 상태 코드

| 코드 | 설명 |
|------|------|
| 200 | 성공 |
| 400 | 잘못된 요청 |
| 404 | 리소스 없음 |
| 500 | 서버 오류 |
| 502 | 게이트웨이 오류 |
| 504 | 타임아웃 |

---

## 📱 클라이언트 사용 예시 (React Native)

```javascript
import { 
  getRealtimeData, 
  getPriceCompare,
  captureNow,
  predictYield 
} from './services/api';

// 실시간 데이터 조회
const data = await getRealtimeData();
console.log(`수확 가능: ${data.Ready}개`);

// 가격 비교
const prices = await getPriceCompare();
console.log(`도매가: ${prices.wholesale_summary.high}원/kg`);

// 즉시 촬영
const result = await captureNow();
console.log(`분석 완료: ${result.data.Ready}개 감지`);

// 수확량 예측
const prediction = await predictYield({
  month: 11,
  temperature: 25,
  humidity: 65,
});
console.log(`예상 수확량: ${prediction.predicted_yield}kg`);
```

---

## 📚 관련 문서

- [시스템 아키텍처](./system_documentation.md)
- [n8n 워크플로우](./n8n_현재노드)
- [Swagger UI](./swagger-ui.html)
- [OpenAPI Spec](./swagger-api.yaml)

---

**© 2024 Seed Farm. All rights reserved.**