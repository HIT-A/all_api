# Weihai Campus API (HITWH)

campus_id: weihai
status: deferred

## Scope
This doc lists usable API/HTML endpoints from `api_bundle` for Weihai. Static assets are excluded. The current capture focuses on WebVPN QR login flow and a few teaching-system endpoints.

## Auth and Session (WebVPN QR)
Base host: `webvpn.hitwh.edu.cn`
Auth base (dynamic): `https://webvpn.hitwh.edu.cn/https/{id}/authserver`

Rules:
- `{id}` changes over time; extract it from redirects.
- Keep a single cookie jar across all requests.
- Login flow is QR-based (2FA).

### 1.1 Get QR token
- Method: GET
- Path: `{AUTH_BASE}/qrCode/getToken`
- Query: `vpn-12-o2-ids.hit.edu.cn, ts`

Request example:
```http
GET /https/{id}/authserver/qrCode/getToken?vpn-12-o2-ids.hit.edu.cn&ts=1700000000000 HTTP/1.1
Host: webvpn.hitwh.edu.cn
```

Response example (placeholder):
```json
{"uuid":"<uuid>"}
```

Field notes:
- Response body is not captured; expected to contain a UUID token.

### 1.2 Get QR code image
- Method: GET
- Path: `{AUTH_BASE}/qrCode/getCode`
- Query: `vpn-1, uuid`

Request example:
```http
GET /https/{id}/authserver/qrCode/getCode?vpn-1&uuid=<uuid> HTTP/1.1
Host: webvpn.hitwh.edu.cn
```

Response example:
```http
HTTP/1.1 200 OK
Content-Type: image/png

<binary>
```

### 1.3 Poll QR status
- Method: GET
- Path: `{AUTH_BASE}/qrCode/getStatus.htl`
- Query: `vpn-12-o2-ids.hit.edu.cn, ts, uuid`

Request example:
```http
GET /https/{id}/authserver/qrCode/getStatus.htl?vpn-12-o2-ids.hit.edu.cn&ts=1700000000000&uuid=<uuid> HTTP/1.1
Host: webvpn.hitwh.edu.cn
```

Response example:
```text
1
```

Field notes:
- Status values (from api_bundle notes):
  - `1` confirmed
  - `2` scanned, not confirmed
  - `3` expired

### 1.4 Submit login form
- Method: POST
- Path: `{AUTH_BASE}/login`
- Query: `display=qrLogin&service=<encoded_webvpn_service>`
- Body fields: `lt, uuid, cllt, dllt, _eventId, rmShown, execution`

Request example:
```http
POST /https/{id}/authserver/login?display=qrLogin&service=https%3A%2F%2Fwebvpn.hitwh.edu.cn%2Flogin%3Fcas_login%3Dtrue HTTP/1.1
Host: webvpn.hitwh.edu.cn
Content-Type: application/x-www-form-urlencoded

lt=&uuid=<uuid>&cllt=qrLogin&dllt=generalLogin&_eventId=submit&rmShown=1&execution=<execution>
```

Response example:
```http
HTTP/1.1 302 Found
Set-Cookie: JSESSIONID=<webvpn_session>
```

Field notes:
- `execution` must be parsed from login page.
- Keep cookies from the login page through the submit.

---

## 2. Teaching system (via WebVPN proxy)

The proxy path includes a hash and may change:
- `https://webvpn.hitwh.edu.cn/http/{hash}/...`

### 2.1 CAS entry
- Method: GET
- Path: `/http/{hash}/loginCAS`

Request example:
```http
GET /http/{hash}/loginCAS HTTP/1.1
Host: webvpn.hitwh.edu.cn
```

Response example:
```http
HTTP/1.1 302 Found
Set-Cookie: JSESSIONID=<business_session>
```

### 2.2 Course selection list
- Method: POST
- Path: `/http/{hash}/xsxk/queryXsxkList`

Request example:
```http
POST /http/{hash}/xsxk/queryXsxkList HTTP/1.1
Host: webvpn.hitwh.edu.cn
Content-Type: application/x-www-form-urlencoded

page=1&pageSize=20
```

Response example (placeholder):
```json
{"_example":"response body not captured in api_bundle"}
```

### 2.3 Course selection submit
- Method: POST
- Path: `/http/{hash}/xsxk/saveXsxk`

Request example:
```http
POST /http/{hash}/xsxk/saveXsxk HTTP/1.1
Host: webvpn.hitwh.edu.cn
Content-Type: application/x-www-form-urlencoded

kcid=<course_id>
```

Response example (placeholder):
```json
{"_example":"response body not captured in api_bundle"}
```

---

## Pitfalls and TODO
- Auth base `{id}` and proxy `{hash}` are dynamic; extract from redirects.
- Response bodies for Weihai endpoints are not captured in `api_bundle`.
