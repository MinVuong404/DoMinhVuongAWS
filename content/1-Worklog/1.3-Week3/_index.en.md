---
title: "Worklog Week 3"
date: 2026-08-17
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Worklog Week 3: Cloud Networking Architecture & Amazon API Gateway v2 HTTP API

### 1. General Information & Core Objectives
* **Duration:** August 17, 2026 – August 23, 2026 (Week 3).
* **Location:** AWS Hanoi Office — 7th Floor, Grand Terra Building, 36 Cat Linh, Dong Da, Hanoi.
* **Field Supervisor:** Pham Van Phong / Do Tuan Anh (Solutions Architect).
* **Mentor Lead:** Nguyen Gia Hung (Senior Solutions Architect - AWS Vietnam).
* **Core Technical Objectives:**
  1. Investigate cloud networking topology in **Amazon Virtual Private Cloud (VPC)**, subnets, route tables, and Internet Gateways.
  2. Perform an empirical architectural evaluation comparing **Amazon API Gateway REST API (v1)** and **HTTP API (v2)**.
  3. Formalize the JSON payload schema for the real-time inference interface, specifying input contract (`text`) and output telemetry (`prediction`, `confidence`, `latency_ms`).
  4. Master and resolve browser **Cross-Origin Resource Sharing (CORS)** enforcement when issuing requests from external browser contexts (such as `mail.google.com`).
  5. Configure **Lambda Proxy Integration** utilizing the streamlined Payload Format Version 2.0.

---

### 2. Implementation Schedule & Daily Work Breakdown

| Day | Technical Tasks & Objectives | Start Date | End Date | References | Outcomes & Evidence |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Mon** | • Studied Amazon VPC topology: CIDR block planning `10.0.0.0/16`, public vs private subnet partitioning.<br>• Analyzed Lambda VPC attachment mechanisms: Hyperplane ENI latency overhead on cold starts. | 08/17/2026 | 08/17/2026 | [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)<br>[Improved VPC Networking for AWS Lambda](https://aws.amazon.com/blogs/compute/announcing-improved-vpc-networking-for-aws-lambda-functions/) | Selected non-VPC execution for Lambda to guarantee minimum execution latency without internal RDS dependencies. |
| **Tue** | • Benchmarked REST APIs against HTTP APIs v2: HTTP APIs provide ~60% lower latency overhead and are 71% cheaper ($1.00/million vs $3.50/million requests). | 08/18/2026 | 08/18/2026 | [Choosing Between REST APIs and HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html) | Formally adopted **HTTP API v2** as the ingestion gateway for the Chrome Extension client. |
| **Wed** | • Designed schema contract for request/response serialization between Client and API Gateway.<br>• Drafted OpenAPI documentation for `POST /predict`. | 08/19/2026 | 08/19/2026 | [OpenAPI 3.0 Specification](https://swagger.io/specification/)<br>[JSON Schema Core](https://json-schema.org/) | Finalized input payload contracts and HTTP status codes (200 OK, 400 Bad Request, 500 Internal Server Error). |
| **Thu** | • Implemented CORS configuration on API Gateway: Allowed HTTP methods (`POST`, `OPTIONS`), authorized headers (`Content-Type`, `Authorization`).<br>• Tested preflight OPTIONS validation via cURL. | 08/20/2026 | 08/20/2026 | [Configuring CORS for an HTTP API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-cors.html)<br>[MDN Web Docs: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) | Header `Access-Control-Allow-Origin: *` returned successfully in API Gateway preflight response. |
| **Fri** | • Authored Python event parser handling raw vs JSON payloads within `event['body']`.<br>• Implemented base64 payload decoding fallback logic (`event.get('isBase64Encoded')`). | 08/21/2026 | 08/21/2026 | [Working with AWS Lambda Proxy Integrations](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html) | Lambda function reliably parses inputs whether submitted as raw string or structured JSON. |
| **Sat - Sun**| • Conducted synthetic stress tests using Postman and cURL measuring API Gateway proxy latency.<br>• Captured baseline network latency: Average API Gateway routing time was only ~8 ms. | 08/22/2026 | 08/23/2026 | [Postman API Platform](https://www.postman.com/)<br>[cURL Documentation](https://curl.se/docs/) | Gateway routing demonstrated 100% availability with 0% error rate. |

---

### 3. Practical Configurations & Interface Specifications

#### 3.1. API Contract Schema for `POST /predict`:
* **Request Payload (Client → API Gateway):**
```json
{
  "email_text": "URGENT SECURITY NOTICE: Your online banking account has been locked due to suspicious login activity. Click https://secure-bank-login.xyz/verify immediately to confirm your identity within 24 hours!"
}
```
* **Response Payload (Lambda → API Gateway → Client):**
```json
{
  "request_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "prediction": "Phishing",
  "confidence": 0.9842,
  "confidence_percent": "98.42%",
  "risk_level": "CRITICAL",
  "inference_latency_ms": 11.45,
  "timestamp": "2026-08-20T10:15:30.124Z"
}
```

#### 3.2. CORS Preflight Testing via cURL:
```bash
curl -X OPTIONS https://p7ailap9ci.execute-api.ap-southeast-1.amazonaws.com/predict \
  -H "Origin: https://mail.google.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type" \
  -i
```
*Output HTTP response headers:*
```http
HTTP/2 204
date: Thu, 20 Aug 2026 10:15:30 GMT
access-control-allow-origin: *
access-control-allow-methods: POST,OPTIONS
access-control-allow-headers: content-type
access-control-max-age: 300
```

---

### 4. Technical Challenges & Troubleshooting

* **Issue: `CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource` from Chrome Extension.**
  * *Symptom:* When the Chrome Extension executing on `https://mail.google.com` triggered a `fetch()` call to API Gateway, the browser blocked the response and threw a red CORS console error.
  * *Root Cause Analysis:* HTTP API v2 had not enabled automated preflight response handling. When the browser detects a `POST` request with `Content-Type: application/json`, it automatically dispatches a preflight `OPTIONS` probe. Without an explicit CORS configuration, API Gateway rejects the handshake.
  * *Resolution:* In AWS API Gateway Console → Select API → Navigate to **CORS**:
    * **Access-Control-Allow-Origin:** `*`
    * **Access-Control-Allow-Headers:** `Content-Type,X-Amz-Date,Authorization,X-Api-Key`
    * **Access-Control-Allow-Methods:** `POST,OPTIONS`
    * Furthermore, added redundant fallback headers within the Lambda response payload:
      ```python
      headers = {
          "Content-Type": "application/json",
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "POST,OPTIONS"
      }
      ```
    Dual-layer configuration completely eliminated all cross-origin errors.

---

### 5. Deliverables & Milestones
1. **Network Architecture Decision:** Non-VPC serverless deployment selected for sub-15ms warm invocation latencies.
2. **Standardized HTTP API v2 Specification:** Clear, robust schema contract governing all system components.
3. **Multi-layer CORS Enforcement:** Verified communication between Google Chrome browser contexts and AWS backend endpoints.
