# GeoStream API Reference

**Version:** 1.0.0  
**Base URL:** `http://localhost:8080` (development) | `https://api.geostream.io` (production)  
**Last Updated:** December 2025

---

## 📋 Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Phase 1: Core Routing](#phase-1-core-routing-api)
- [Phase 2: Health Monitoring](#phase-2-health-monitoring-api)
- [Phase 3: Caching](#phase-3-caching-api)
- [Phase 4: Multi-Region](#phase-4-multi-region-api)

---

## Overview

The GeoStream API provides intelligent geo-routing, health monitoring, and multi-region traffic management.

### API Design Principles

- **RESTful** - Standard HTTP methods (GET, POST, PUT, DELETE)
- **JSON-Based** - All requests and responses use `application/json`
- **Idempotent** - Safe to retry GET, PUT, DELETE requests
- **Versioned** - `/api/v1/` prefix for stability

### Base Response Format

All API responses follow this structure:

```json
{
  "success": true,
  "data": {
    /* response payload */
  },
  "error": null,
  "metadata": {
    "timestamp": "2025-12-27T10:00:00Z",
    "request_id": "req_abc123",
    "version": "1.0.0"
  }
}
```

---

## Authentication

### Current (Phase 1-2): No Authentication

Development mode - no authentication required.

### Future (Phase 3+): API Key Authentication

```http
GET /api/v1/route
Authorization: Bearer YOUR_API_KEY
```

**Headers:**

- `Authorization: Bearer <api_key>` - Required for authenticated endpoints

**Error Response (401 Unauthorized):**

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "unauthorized",
    "message": "Invalid or missing API key"
  }
}
```

---

## Error Handling

### Standard Error Response

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "invalid_request",
    "message": "Client IP address is required",
    "details": {
      "field": "client_ip",
      "reason": "missing_field"
    }
  },
  "metadata": {
    "timestamp": "2025-12-27T10:00:00Z",
    "request_id": "req_xyz789"
  }
}
```

### HTTP Status Codes

| Code | Meaning               | When Used                               |
| ---- | --------------------- | --------------------------------------- |
| 200  | OK                    | Successful GET, PUT, DELETE             |
| 201  | Created               | Successful POST (resource created)      |
| 400  | Bad Request           | Invalid request parameters              |
| 401  | Unauthorized          | Missing or invalid API key              |
| 404  | Not Found             | Resource doesn't exist                  |
| 429  | Too Many Requests     | Rate limit exceeded                     |
| 500  | Internal Server Error | Unexpected server error                 |
| 503  | Service Unavailable   | All regions unhealthy (circuit breaker) |

### Error Codes

| Code                | HTTP Status | Description                        |
| ------------------- | ----------- | ---------------------------------- |
| `invalid_request`   | 400         | Malformed request or missing field |
| `unauthorized`      | 401         | Authentication failed              |
| `not_found`         | 404         | Resource not found                 |
| `rate_limited`      | 429         | Too many requests                  |
| `no_healthy_region` | 503         | All regions unavailable            |
| `internal_error`    | 500         | Unexpected server error            |

---

## Rate Limiting

### Current Limits (Phase 1-2)

- **Development:** Unlimited
- **Production:** 1000 requests/minute per IP

### Rate Limit Headers

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1735296000
```

### Rate Limit Exceeded (429)

```json
{
  "success": false,
  "error": {
    "code": "rate_limited",
    "message": "Rate limit exceeded. Try again in 45 seconds.",
    "details": {
      "limit": 1000,
      "reset_at": "2025-12-27T10:01:00Z"
    }
  }
}
```

---

## Phase 1: Core Routing API

### 1. Get Optimal Region

**Endpoint:** `POST /api/v1/route`

**Description:** Select the optimal region for a client based on geo-routing strategy.

**Request:**

```http
POST /api/v1/route
Content-Type: application/json

{
  "client_ip": "8.8.8.8",
  "service": "api",
  "strategy": "latency"  // optional: "latency" | "cost" | "availability"
}
```

**Request Parameters:**

| Field       | Type   | Required | Description                                    |
| ----------- | ------ | -------- | ---------------------------------------------- |
| `client_ip` | string | Yes      | Client IP address (IPv4 or IPv6)               |
| `service`   | string | Yes      | Service name (e.g., "api", "cdn", "websocket") |
| `strategy`  | string | No       | Routing strategy (default: "latency")          |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "region": {
      "id": "us-west-2",
      "name": "US West (Oregon)",
      "endpoint": "https://us-west-2.api.geostream.io",
      "location": {
        "latitude": 45.5152,
        "longitude": -122.6784
      }
    },
    "latency_ms": 45,
    "reason": "lowest_latency",
    "alternatives": [
      {
        "region_id": "us-east-1",
        "latency_ms": 120,
        "reason": "higher_latency"
      }
    ]
  },
  "metadata": {
    "timestamp": "2025-12-27T10:00:00Z",
    "request_id": "req_route_001"
  }
}
```

**Response Fields:**

| Field             | Type    | Description                            |
| ----------------- | ------- | -------------------------------------- |
| `region.id`       | string  | Region identifier                      |
| `region.name`     | string  | Human-readable region name             |
| `region.endpoint` | string  | Full URL to connect to                 |
| `region.location` | object  | Geographic coordinates                 |
| `latency_ms`      | integer | Estimated latency in milliseconds      |
| `reason`          | string  | Why this region was selected           |
| `alternatives[]`  | array   | Other viable regions (sorted by score) |

**Error Response (400 Bad Request):**

```json
{
  "success": false,
  "error": {
    "code": "invalid_request",
    "message": "Client IP address is required",
    "details": {
      "field": "client_ip",
      "reason": "missing_field"
    }
  }
}
```

**Error Response (503 Service Unavailable):**

```json
{
  "success": false,
  "error": {
    "code": "no_healthy_region",
    "message": "All regions are currently unhealthy"
  }
}
```

**Example (cURL):**

```bash
curl -X POST http://localhost:8080/api/v1/route \
  -H "Content-Type: application/json" \
  -d '{
    "client_ip": "8.8.8.8",
    "service": "api",
    "strategy": "latency"
  }'
```

---

### 2. List Regions

**Endpoint:** `GET /api/v1/regions`

**Description:** Get all available regions with their health status.

**Request:**

```http
GET /api/v1/regions?healthy_only=true
```

**Query Parameters:**

| Parameter      | Type    | Required | Description                    |
| -------------- | ------- | -------- | ------------------------------ |
| `healthy_only` | boolean | No       | Filter to only healthy regions |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "regions": [
      {
        "id": "us-west-2",
        "name": "US West (Oregon)",
        "location": {
          "latitude": 45.5152,
          "longitude": -122.6784
        },
        "endpoint": "https://us-west-2.api.geostream.io",
        "is_healthy": true,
        "capacity": 100,
        "current_load": 45,
        "last_health_check": "2025-12-27T09:59:30Z"
      },
      {
        "id": "us-east-1",
        "name": "US East (Virginia)",
        "location": {
          "latitude": 39.9526,
          "longitude": -75.1652
        },
        "endpoint": "https://us-east-1.api.geostream.io",
        "is_healthy": true,
        "capacity": 100,
        "current_load": 60,
        "last_health_check": "2025-12-27T09:59:25Z"
      }
    ],
    "total": 2
  }
}
```

**Example (cURL):**

```bash
curl http://localhost:8080/api/v1/regions?healthy_only=true
```

---

### 3. Get Region Details

**Endpoint:** `GET /api/v1/regions/{region_id}`

**Description:** Get detailed information about a specific region.

**Request:**

```http
GET /api/v1/regions/us-west-2
```

**Path Parameters:**

| Parameter   | Type   | Required | Description       |
| ----------- | ------ | -------- | ----------------- |
| `region_id` | string | Yes      | Region identifier |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "region": {
      "id": "us-west-2",
      "name": "US West (Oregon)",
      "location": {
        "latitude": 45.5152,
        "longitude": -122.6784
      },
      "endpoint": "https://us-west-2.api.geostream.io",
      "is_healthy": true,
      "capacity": 100,
      "current_load": 45,
      "last_health_check": "2025-12-27T09:59:30Z",
      "health_history": [
        {
          "timestamp": "2025-12-27T09:59:30Z",
          "is_healthy": true,
          "response_time_ms": 12
        },
        {
          "timestamp": "2025-12-27T09:59:00Z",
          "is_healthy": true,
          "response_time_ms": 15
        }
      ]
    }
  }
}
```

**Error Response (404 Not Found):**

```json
{
  "success": false,
  "error": {
    "code": "not_found",
    "message": "Region 'us-west-3' not found"
  }
}
```

---

## Phase 2: Health Monitoring API

### 4. Get Health Status

**Endpoint:** `GET /api/v1/health`

**Description:** Get overall system health status.

**Request:**

```http
GET /api/v1/health
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "status": "healthy",
    "version": "1.0.0",
    "uptime_seconds": 86400,
    "regions": {
      "total": 3,
      "healthy": 3,
      "unhealthy": 0
    },
    "services": {
      "database": "healthy",
      "cache": "healthy",
      "circuit_breaker": "closed"
    }
  }
}
```

**Response (503 Service Unavailable):**

```json
{
  "success": false,
  "data": {
    "status": "degraded",
    "regions": {
      "total": 3,
      "healthy": 1,
      "unhealthy": 2
    },
    "services": {
      "database": "healthy",
      "cache": "unhealthy",
      "circuit_breaker": "open"
    }
  },
  "error": {
    "code": "service_degraded",
    "message": "System is operating in degraded mode"
  }
}
```

---

### 5. Get Region Health History

**Endpoint:** `GET /api/v1/regions/{region_id}/health`

**Description:** Get historical health check data for a region.

**Request:**

```http
GET /api/v1/regions/us-west-2/health?duration=1h
```

**Query Parameters:**

| Parameter  | Type   | Required | Description                          |
| ---------- | ------ | -------- | ------------------------------------ |
| `duration` | string | No       | Time range (e.g., "1h", "24h", "7d") |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "region_id": "us-west-2",
    "duration": "1h",
    "health_checks": [
      {
        "timestamp": "2025-12-27T10:00:00Z",
        "is_healthy": true,
        "response_time_ms": 12,
        "error_count": 0
      },
      {
        "timestamp": "2025-12-27T09:59:30Z",
        "is_healthy": true,
        "response_time_ms": 15,
        "error_count": 0
      }
    ],
    "summary": {
      "uptime_percentage": 99.5,
      "average_response_time_ms": 13,
      "total_checks": 120,
      "failed_checks": 1
    }
  }
}
```

---

### 6. Trigger Manual Health Check

**Endpoint:** `POST /api/v1/regions/{region_id}/health/check`

**Description:** Manually trigger a health check for a specific region.

**Request:**

```http
POST /api/v1/regions/us-west-2/health/check
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "region_id": "us-west-2",
    "is_healthy": true,
    "response_time_ms": 14,
    "checked_at": "2025-12-27T10:05:00Z"
  }
}
```

**Error Response (503 Service Unavailable):**

```json
{
  "success": false,
  "data": {
    "region_id": "us-west-2",
    "is_healthy": false,
    "error": "connection timeout after 5000ms",
    "checked_at": "2025-12-27T10:05:00Z"
  },
  "error": {
    "code": "health_check_failed",
    "message": "Region us-west-2 is unhealthy"
  }
}
```

---

## Phase 3: Caching API

### 7. Get Cache Statistics

**Endpoint:** `GET /api/v1/cache/stats`

**Description:** Get cache performance metrics.

**Request:**

```http
GET /api/v1/cache/stats
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "cache_hit_ratio": 0.85,
    "total_requests": 10000,
    "cache_hits": 8500,
    "cache_misses": 1500,
    "average_hit_latency_ms": 2,
    "average_miss_latency_ms": 45,
    "evictions": 120,
    "size_bytes": 52428800,
    "max_size_bytes": 104857600,
    "keys_count": 1250
  }
}
```

---

### 8. Invalidate Cache

**Endpoint:** `DELETE /api/v1/cache/{key}`

**Description:** Manually invalidate a cache entry.

**Request:**

```http
DELETE /api/v1/cache/regions
```

**Path Parameters:**

| Parameter | Type   | Required | Description |
| --------- | ------ | -------- | ----------- |
| `key`     | string | Yes      | Cache key   |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "key": "regions",
    "invalidated": true,
    "invalidated_at": "2025-12-27T10:10:00Z"
  }
}
```

**Error Response (404 Not Found):**

```json
{
  "success": false,
  "error": {
    "code": "not_found",
    "message": "Cache key 'regions' not found"
  }
}
```

---

## Phase 4: Multi-Region API

### 9. Get Cross-Region Latencies

**Endpoint:** `GET /api/v1/latencies`

**Description:** Get measured latencies between all region pairs.

**Request:**

```http
GET /api/v1/latencies
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "latencies": [
      {
        "from": "us-west-2",
        "to": "us-east-1",
        "latency_ms": 70,
        "measured_at": "2025-12-27T10:00:00Z"
      },
      {
        "from": "us-west-2",
        "to": "eu-west-1",
        "latency_ms": 140,
        "measured_at": "2025-12-27T10:00:00Z"
      },
      {
        "from": "us-east-1",
        "to": "eu-west-1",
        "latency_ms": 100,
        "measured_at": "2025-12-27T10:00:00Z"
      }
    ]
  }
}
```

---

### 10. Get Region Metrics

**Endpoint:** `GET /api/v1/regions/{region_id}/metrics`

**Description:** Get detailed metrics for a region.

**Request:**

```http
GET /api/v1/regions/us-west-2/metrics?duration=1h
```

**Query Parameters:**

| Parameter  | Type   | Required | Description                |
| ---------- | ------ | -------- | -------------------------- |
| `duration` | string | No       | Time range (default: "1h") |

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "region_id": "us-west-2",
    "duration": "1h",
    "metrics": {
      "requests_per_second": 150,
      "p50_latency_ms": 15,
      "p95_latency_ms": 45,
      "p99_latency_ms": 85,
      "error_rate": 0.001,
      "cpu_usage_percent": 45,
      "memory_usage_percent": 60,
      "disk_usage_percent": 30
    },
    "timeseries": [
      {
        "timestamp": "2025-12-27T10:00:00Z",
        "requests_per_second": 148,
        "p99_latency_ms": 82
      },
      {
        "timestamp": "2025-12-27T09:59:00Z",
        "requests_per_second": 152,
        "p99_latency_ms": 88
      }
    ]
  }
}
```

---

## Webhooks (Future)

### 11. Region Health Event

**Event:** `region.health_changed`

**Description:** Triggered when a region's health status changes.

**Payload:**

```json
{
  "event": "region.health_changed",
  "timestamp": "2025-12-27T10:15:00Z",
  "data": {
    "region_id": "us-west-2",
    "previous_status": "healthy",
    "current_status": "unhealthy",
    "reason": "health_check_timeout",
    "failover_region": "us-east-1"
  }
}
```

---

## Prometheus Metrics Endpoint

### 12. Metrics Export

**Endpoint:** `GET /metrics`

**Description:** Export Prometheus-compatible metrics.

**Request:**

```http
GET /metrics
```

**Response (200 OK, text/plain):**

```
# HELP http_requests_total Total number of HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="POST",endpoint="/route",status="200"} 10000

# HELP http_request_duration_seconds HTTP request latency
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{method="POST",endpoint="/route",le="0.01"} 5000
http_request_duration_seconds_bucket{method="POST",endpoint="/route",le="0.05"} 8500
http_request_duration_seconds_bucket{method="POST",endpoint="/route",le="0.1"} 9500
http_request_duration_seconds_sum{method="POST",endpoint="/route"} 300.5
http_request_duration_seconds_count{method="POST",endpoint="/route"} 10000

# HELP cache_hit_ratio Cache hit ratio
# TYPE cache_hit_ratio gauge
cache_hit_ratio{cache="region_config"} 0.85

# HELP circuit_breaker_state Circuit breaker state (0=Closed, 1=Open, 2=HalfOpen)
# TYPE circuit_breaker_state gauge
circuit_breaker_state{region="us-west-2"} 0
circuit_breaker_state{region="us-east-1"} 0
```

---

## Code Examples

### Go Example

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

type RouteRequest struct {
    ClientIP string `json:"client_ip"`
    Service  string `json:"service"`
    Strategy string `json:"strategy,omitempty"`
}

type RouteResponse struct {
    Success bool `json:"success"`
    Data    struct {
        Region struct {
            ID       string `json:"id"`
            Name     string `json:"name"`
            Endpoint string `json:"endpoint"`
        } `json:"region"`
        LatencyMS int    `json:"latency_ms"`
        Reason    string `json:"reason"`
    } `json:"data"`
}

func main() {
    req := RouteRequest{
        ClientIP: "8.8.8.8",
        Service:  "api",
        Strategy: "latency",
    }

    body, _ := json.Marshal(req)
    resp, _ := http.Post(
        "http://localhost:8080/api/v1/route",
        "application/json",
        bytes.NewBuffer(body),
    )

    var result RouteResponse
    json.NewDecoder(resp.Body).Decode(&result)

    fmt.Printf("Optimal region: %s (latency: %dms)\n",
        result.Data.Region.ID,
        result.Data.LatencyMS)
}
```

### JavaScript Example

```javascript
async function getOptimalRegion(clientIP) {
  const response = await fetch("http://localhost:8080/api/v1/route", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      client_ip: clientIP,
      service: "api",
      strategy: "latency",
    }),
  });

  const result = await response.json();

  if (result.success) {
    console.log(`Optimal region: ${result.data.region.id}`);
    console.log(`Latency: ${result.data.latency_ms}ms`);
    return result.data.region.endpoint;
  } else {
    console.error(`Error: ${result.error.message}`);
    return null;
  }
}

getOptimalRegion("8.8.8.8");
```

### Python Example

```python
import requests

def get_optimal_region(client_ip: str) -> dict:
    url = "http://localhost:8080/api/v1/route"
    payload = {
        "client_ip": client_ip,
        "service": "api",
        "strategy": "latency"
    }

    response = requests.post(url, json=payload)
    result = response.json()

    if result["success"]:
        region = result["data"]["region"]
        print(f"Optimal region: {region['id']}")
        print(f"Latency: {result['data']['latency_ms']}ms")
        return region["endpoint"]
    else:
        print(f"Error: {result['error']['message']}")
        return None

get_optimal_region("8.8.8.8")
```

---

## OpenAPI Specification

Full OpenAPI 3.0 specification available at: `/api/v1/openapi.json`

---

<div align="center">

**Built with ❤️ by [@Nicholas Emmanuel](https://github.com/nickemma)**

[⬆ Back to Top](#geostream-api-reference)

</div>
