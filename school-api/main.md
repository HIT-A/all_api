# Main Campus API (HIT Harbin)

campus_id: main
status: active

## Scope
This doc is derived from the captures under the main-campus login bundle. It lists usable API/HTML endpoints and excludes static assets. It is intended to cover all endpoints needed to complete the login flow and reach teaching-system APIs.

## Auth and Session (WebVPN + CAS)
Primary access is via WebVPN (iVPN) and CAS. Keep a single cookie jar across all redirects.

Key hosts:
- `ids-hit-edu-cn-s.ivpn.hit.edu.cn` (CAS auth)
- `ivpn.hit.edu.cn` (CAS validate and portal scaffolding)
- `i-hit-edu-cn.ivpn.hit.edu.cn` (portal and portal_api)
- `jwts-hit-edu-cn.ivpn.hit.edu.cn` (teaching system)

Session rules:
- CAS cookies (`CASTGC`, `JSESSIONID`, `route`) do not equal business session cookies.
- Portal and teaching systems issue their own `JSESSIONID` after ticket consumption.
- Most portal endpoints require `sf_request_type=ajax` in query.

Minimal flow (HTML endpoints):
1. GET `/authserver/login?service=...` on `ids-hit-edu-cn-s.ivpn.hit.edu.cn`
2. Complete CAS login / MFA and obtain `ticket` via redirect
3. GET `/auth/cas_validate` on `ivpn.hit.edu.cn`
4. GET `/portal/` on `ivpn.hit.edu.cn`
5. GET `/portal?token=<JWT>` on `i-hit-edu-cn.ivpn.hit.edu.cn`
6. GET `/loginCAS?ticket=...` on `jwts-hit-edu-cn.ivpn.hit.edu.cn`

---

## 1. CAS and iVPN (Auth / HTML)

### 1.1 CAS login page
- Host: `ids-hit-edu-cn-s.ivpn.hit.edu.cn`
- Method: GET
- Path: `/authserver/login`
- Query: `service`

Request example:
```http
GET /authserver/login?service=<url-encoded-service> HTTP/1.1
Host: ids-hit-edu-cn-s.ivpn.hit.edu.cn
```

Response example:
```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>...CAS login page...</html>
```

Field notes:
- `service`: target service callback URL for CAS ticket.

### 1.2 Re-auth submit (MFA)
- Host: `ids-hit-edu-cn-s.ivpn.hit.edu.cn`
- Method: POST
- Path: `/authserver/reAuthCheck/reAuthSubmit.do`
- Query: `sf_request_type`
- Body: `answer1, answer2, dynamicCode, isMultifactor, otpCode, password, reAuthType, service, skipTmpReAuth, uuid`

Request example:
```http
POST /authserver/reAuthCheck/reAuthSubmit.do?sf_request_type=ajax HTTP/1.1
Host: ids-hit-edu-cn-s.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

password=REDACTED&otpCode=REDACTED&service=<service>
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

Field notes:
- Use form fields from the current login page.

### 1.3 Tenant info
- Host: `ids-hit-edu-cn-s.ivpn.hit.edu.cn`
- Method: GET
- Path: `/authserver/tenant/info`
- Query: `sf_request_type`

Request example:
```http
GET /authserver/tenant/info?sf_request_type=ajax HTTP/1.1
Host: ids-hit-edu-cn-s.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 1.4 Fingerprint info
- Host: `ids-hit-edu-cn-s.ivpn.hit.edu.cn`
- Method: GET
- Path: `/authserver/bfp/info`
- Query: `_, bfp, sf_request_type`

Request example:
```http
GET /authserver/bfp/info?sf_request_type=ajax&_ts=...&bfp=... HTTP/1.1
Host: ids-hit-edu-cn-s.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 1.5 CAS ticket validate
- Host: `ivpn.hit.edu.cn`
- Method: GET
- Path: `/auth/cas_validate`
- Query: `entry_id, redirect_uri, ticket`

Request example:
```http
GET /auth/cas_validate?entry_id=<id>&redirect_uri=<url>&ticket=ST-... HTTP/1.1
Host: ivpn.hit.edu.cn
```

Response example:
```http
HTTP/1.1 302 Found
Location: <redirect_uri>
```

### 1.6 Portal scaffolding pages
- Host: `ivpn.hit.edu.cn`
- Method: GET
- Paths:
  - `/portal/` (query: `data`)
  - `/por/jsdata.csp`
  - `/por/login_auth.csp` (query: `apiversion`)
  - `/portal/views/common_message/common_message.html`
  - `/portal/views/thirdparty_auth_judgment/thirdparty_auth_judgment.html`

Request example:
```http
GET /portal/ HTTP/1.1
Host: ivpn.hit.edu.cn
```

Response example:
```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>...portal scaffolding page...</html>
```

Field notes:
- These endpoints provide portal HTML or CSP scaffolding during the iVPN flow.

---

## 2. Portal (i-hit-edu-cn.ivpn.hit.edu.cn)

### 2.1 Portal entry
- Method: GET
- Path: `/portal`
- Query: `token`

Request example:
```http
GET /portal?token=<JWT> HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example:
```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>...portal app...</html>
```

### 2.2 Portal home
- Method: GET
- Path: `/portal/home/`
- Query: `sangfor_redirect, sangfor_sessid`

Request example:
```http
GET /portal/home/?sangfor_redirect=1&sangfor_sessid=<id> HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example:
```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>...portal home...</html>
```

### 2.3 Portal CAS entry
- Method: GET
- Path: `/cas/login_portal`

Request example:
```http
GET /cas/login_portal HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example:
```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>...CAS portal...</html>
```

### 2.4 WebVPN config
- Method: GET
- Path: `/sf-webproxy/api/vpn-config`

Request example:
```http
GET /sf-webproxy/api/vpn-config HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

---

## 3. Portal API (i-hit-edu-cn.ivpn.hit.edu.cn)

Common rules:
- Query parameter `sf_request_type=ajax` appears on most endpoints.
- Portal cookies and JWT session are required.

### 3.1 Security
- POST `/portal_api/security/validate_user`
- Query: `sf_request_type`

Request example:
```http
POST /portal_api/security/validate_user?sf_request_type=ajax HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 3.2 AI entry
- POST `/portal_api/ai/get_ai_url`
- Query: `sf_request_type`
- Body: `{"content":""}`

Request example:
```http
POST /portal_api/ai/get_ai_url?sf_request_type=ajax HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/json

{"content":""}
```

Response example (placeholder):
```json
{"data":"<signed-url>","errcode":"0","errmsg":"ok"}
```

### 3.3 Messages
- GET `/portal_api/message/count`
- GET `/portal_api/message/list`

Request example:
```http
GET /portal_api/message/count?sf_request_type=ajax HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"count":0}
```

### 3.4 Term and config
- GET `/portal_api/term/current`
- GET `/portal_api/config/version`

Request example:
```http
GET /portal_api/term/current?sf_request_type=ajax HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"term":"<current>"}
```

### 3.5 Services and banners
- GET `/portal_api/service/hot`
- GET `/portal_api/service/newest`
- GET `/portal_api/service_window/hot_card`
- GET `/portal_api/banner/home`
- GET `/portal_api/banner/service_card`
- GET `/portal_api/service/queryServiceSubject`

Request example:
```http
GET /portal_api/service/hot?sf_request_type=ajax HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"items":[]}
```

### 3.6 User center modules
- GET `/portal_api/user_center/module_card`
- GET `/portal_api/user_center/module_data/app`
- GET `/portal_api/user_center/module_data/email`
- GET `/portal_api/user_center/module_data/pan`
- GET `/portal_api/user_center/module_data/ykt`
- GET `/portal_api/user_center/module_data/library`

Request example:
```http
GET /portal_api/user_center/module_card?sf_request_type=ajax HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"modules":[]}
```

### 3.7 News
- GET `/portal_api/news/channels`
- GET `/portal_api/news/list_for_channel` (query: `channelId`)
- POST `/portal_api/news/query_fav` (body: `{"current":1,"size":100}`)

Request example:
```http
GET /portal_api/news/list_for_channel?sf_request_type=ajax&channelId=<id> HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"items":[]}
```

### 3.8 Apps and favorites
- GET `/portal_api/app/query_for_user`
- POST `/portal_api/app/query_fav` (body: `{"current":1,"size":100}`)
- POST `/portal_api/service/query_fav` (body: `{"current":1,"size":100}`)

Request example:
```http
POST /portal_api/app/query_fav?sf_request_type=ajax HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/json

{"current":1,"size":100}
```

Response example (placeholder):
```json
{"items":[]}
```

### 3.9 Misc
- GET `/portal_api/affair/todo_card` (query: `type`)
- GET `/portal_api/common/footer_links`
- GET `/portal_api/common/links`
- GET `/portal_api/portal_page/query_for_user`
- GET `/portal_api/email/unread`
- GET `/portal_api/email/unread_cnt`
- GET `/portal_api/resource/file` (query: `id`, binary)

Request example:
```http
GET /portal_api/resource/file?id=<file_id> HTTP/1.1
Host: i-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (file):
```http
HTTP/1.1 200 OK
Content-Type: application/octet-stream

<binary>
```

---

## 4. Teaching system (jwts-hit-edu-cn.ivpn.hit.edu.cn)

### 4.1 CAS entry
- Method: GET
- Path: `/loginCAS`
- Query: `ticket`

Request example:
```http
GET /loginCAS?ticket=ST-... HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
```

Response example:
```http
HTTP/1.1 302 Found
Set-Cookie: JSESSIONID=<business_session>
```

### 4.2 Empty page placeholder
- Method: GET
- Path: `/jsp/pub/pub/empty.jsp`

Request example:
```http
GET /jsp/pub/pub/empty.jsp HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
```

Response example:
```http
HTTP/1.1 200 OK
Content-Type: text/html

<html>...</html>
```

### 4.3 Training plan
- Method: GET or POST
- Path: `/zxjh/queryZxkc`
- Body (POST): `pageKcmc, pageKkxn, pageKkxn1, pageKkxq, pageKkxq1, pageNj, pageYxdm, pageZydm`

Request example (POST):
```http
POST /zxjh/queryZxkc HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

pageKcmc=&pageKkxn=&pageKkxn1=&pageKkxq=&pageKkxq1=&pageNj=&pageYxdm=&pageZydm=
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 4.4 Major list
- Method: POST
- Path: `/pub/queryYxzyList_x`
- Query: `sf_request_type`
- Body: `nj, yxdm`

Request example:
```http
POST /pub/queryYxzyList_x?sf_request_type=ajax HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

nj=2024&yxdm=<dept_code>
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 4.5 Empty rooms
- Method: POST
- Path: `/kjscx/queryKjs`
- Body: `pageCddm, pageLhdm, pageXiaoqu, pageXnxq, pageZc1, pageZc2`

Request example:
```http
POST /kjscx/queryKjs HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

pageCddm=&pageLhdm=&pageXiaoqu=&pageXnxq=&pageZc1=1&pageZc2=3
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 4.6 Personal timetable
- Method: POST
- Path: `/kbcx/queryGrkb`
- Body: `fhlj, xnxq, zc`

Request example:
```http
POST /kbcx/queryGrkb HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

fhlj=&xnxq=2025-20261&zc=1
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 4.7 Scores
- Method: POST
- Path: `/cjcx/queryQzcj` (midterm)
- Body: `pageKcmc, pageXnxq`

Request example:
```http
POST /cjcx/queryQzcj HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

pageKcmc=&pageXnxq=2025-20261
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

- Method: POST
- Path: `/cjcx/queryQmcj` (final)
- Body: `pageBkcxbj, pageKcmc, pageSfjg, pageXnxq`

Request example:
```http
POST /cjcx/queryQmcj HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

pageBkcxbj=&pageKcmc=&pageSfjg=&pageXnxq=2025-20261
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 4.8 Exam schedule
- Method: POST
- Path: `/kscx/queryKcForXs1`
- Body: `kssjd, xnxq`

Request example:
```http
POST /kscx/queryKcForXs1 HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

kssjd=&xnxq=2025-20261
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 4.9 Course selection list
- Method: POST
- Path: `/xsxk/queryYxkc`
- Body: `emzh, kcdmpx, kcmcpx, pageBjdm, pageKcmc, pageKkxiaoqu, pageKkyx, pageNj, pageXklb, pageXnxq, pageYxdm, pageZydm, qz, rlpx, sfyrl, token, zy`

Request example:
```http
POST /xsxk/queryYxkc HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
Content-Type: application/x-www-form-urlencoded

pageKcmc=&pageXnxq=2025-20261
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

### 4.10 Notices
- Method: GET
- Path: `/pubtzgg/queryTzggByMxxs`

Request example:
```http
GET /pubtzgg/queryTzggByMxxs HTTP/1.1
Host: jwts-hit-edu-cn.ivpn.hit.edu.cn
```

Response example (placeholder):
```json
{"_example":"response body not captured in bundle"}
```

---

## Pitfalls and TODO
- Portal and teaching-system response bodies are not included in the current bundle; placeholders are used.
- Teaching-system endpoints require the jwts business `JSESSIONID` cookie.
- Some endpoints require query flags such as `sf_request_type=ajax`.
