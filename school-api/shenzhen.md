# Shenzhen Campus API (HITSZ)

campus_id: shenzhen

## Scope
This doc covers all usable endpoints from `api_bundle` for Shenzhen. Static assets are excluded. The focus is on login/session, timetable, scores, empty rooms, and course selection.

## Auth and Session
Base host: `mjw.hitsz.edu.cn`
Base path: `/incoSpringBoot`

Pre-login:
- Authorization: `Basic aW5jb246MTIzNDU=`
- Content-Type: `application/x-www-form-urlencoded`

Post-login:
- Authorization: `bearer <access_token>`
- Keep cookies from the same HTTP session:
  - `route` (from init endpoint)
  - `JSESSIONID` (from login and refreshed by some endpoints)
  - `org.springframework.web.servlet.i18n.CookieLocaleResolver.LOCALE` (optional)

Minimal login flow:
1. POST `/component/queryApplicationSetting/rsa`
2. POST `/c_raskey`
3. POST `/authentication/ldap`
4. Persist `access_token` and cookies (route + JSESSIONID)

---

## 1. Login

### 1.1 Init route cookie
- Method: POST
- Path: `/component/queryApplicationSetting/rsa`
- Auth: Basic
- Content-Type: `application/x-www-form-urlencoded`
- Request body: empty

Request example:
```http
POST /incoSpringBoot/component/queryApplicationSetting/rsa HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: Basic aW5jb246MTIzNDU=
Content-Type: application/x-www-form-urlencoded

```

Response example:
```json
{"code":0,"msg":null,"msg_en":null,"content":"0"}
```

Field notes:
- `code`: success indicator (0 in captured sample)
- `content`: string flag (meaning unclear)

### 1.2 Get RSA params
- Method: POST
- Path: `/c_raskey`
- Auth: Basic
- Content-Type: `application/x-www-form-urlencoded`
- Request body: empty

Request example:
```http
POST /incoSpringBoot/c_raskey HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: Basic aW5jb246MTIzNDU=
Content-Type: application/x-www-form-urlencoded

```

Response example:
```json
{
  "CLIENT_RSA_EXPONENT":"<base64>",
  "CLIENT_RSA_MODULUS":"10001"
}
```

Field notes:
- The field names appear swapped vs typical RSA semantics; follow captured behavior.
- Login request currently uses plaintext password in form data.

### 1.3 LDAP login
- Method: POST
- Path: `/authentication/ldap`
- Auth: Basic
- Content-Type: `application/x-www-form-urlencoded`

Request body format:
```
username=<student_id>&password=<password>
```

Request example:
```http
POST /incoSpringBoot/authentication/ldap HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: Basic aW5jb246MTIzNDU=
Content-Type: application/x-www-form-urlencoded

username=2024XXXXXX&password=REDACTED
```

Response example:
```json
{
  "access_token":"<token>",
  "refresh_token":"<refresh_token>",
  "token_type":"bearer",
  "expires_in":7327462,
  "scope":"all",
  "data":{
    "pylx":"1",
    "lxdh":"<redacted>",
    "yhlx":"1",
    "yhxm":"<redacted>",
    "sfjxyz":false,
    "bmmc":"<dept_name>"
  },
  "info":{
    "yhdm":"<student_id>",
    "xm":"<name>",
    "roles":["01"]
  }
}
```

Field notes:
- `access_token`: bearer token for all post-login requests.
- `refresh_token`: present but refresh endpoint not confirmed.
- `info.yhdm`: user id (student id).

---

## 2. Timetable

### 2.1 Daily timetable
- Method: POST
- Path: `/app/kbrcbyapp/querykbrcbyday`
- Auth: bearer
- Content-Type: `application/x-www-form-urlencoded`

Request body:
```
nyr=YYYY-MM-DD
```

Request example:
```http
POST /incoSpringBoot/app/kbrcbyapp/querykbrcbyday HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/x-www-form-urlencoded

nyr=2026-03-09
```

Response example:
```json
{
  "code":200,
  "msg":"SUCCESS",
  "msg_en":"SUCCESS_EN",
  "content":[
    {"KSJC":1,"JSJC":2,"DJ":1,"XJ":"1 2 ","JTSJ":" - "},
    {"KSJC":3,"JSJC":4,"DJ":2,"XJ":"3 4 ","JTSJ":" - "},
    {"KSJC":5,"JSJC":6,"DJ":3,"XJ":"5 6 ","JTSJ":" - ","kbrc":[{"KCMC":"Machine Learning","CDMC":"A309","XB":2}]}
  ]
}
```

Field notes:
- `KSJC` / `JSJC`: start/end period (inclusive).
- `DJ`: block index (1..6).
- `kbrc[]`: courses within that block.

### 2.2 Timetable overview (by date range)
- Method: POST
- Path: `/app/kbrcbyapp/querykbrczong`
- Auth: bearer
- Content-Type: `application/x-www-form-urlencoded`

Request example:
```http
POST /incoSpringBoot/app/kbrcbyapp/querykbrczong HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/x-www-form-urlencoded

nyr=2026-03-09
```

Response example (shape):
```json
{
  "code":200,
  "msg":"SUCCESS",
  "content":[
    {
      "XN":"2025-2026",
      "XQ":"2",
      "XQJ":"1",
      "RQ":"2026-03-09",
      "kbrc":[{"KCMC":"<course>","RQ":"2026-03-09"}]
    }
  ]
}
```

Field notes:
- `XQJ`: day of week as string 1..7.
- `RQ`: date string.

### 2.3 Term list
- Method: POST
- Path: `/app/commapp/queryxnxqlist`
- Auth: bearer
- Content-Type: `application/x-www-form-urlencoded`

Request example:
```http
POST /incoSpringBoot/app/commapp/queryxnxqlist HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/x-www-form-urlencoded

```

Response example:
```json
{
  "code":200,
  "msg":"SUCCESS",
  "content":[
    {
      "SFDQXQ":"1",
      "XN":"2025-2026",
      "XNXQ":"2025-20262",
      "XQ":"2",
      "XNXQMC":"2026Spring",
      "XNXQMC_EN":"2026Spring"
    }
  ]
}
```

Field notes:
- `SFDQXQ`: current term flag (`1` = current).

### 2.4 Week list by term
- Method: POST
- Path: `/app/commapp/queryzclistbyxnxq`
- Auth: bearer
- Content-Type: `application/json`

Request example:
```http
POST /incoSpringBoot/app/commapp/queryzclistbyxnxq HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/json

{"xn":"2025-2026","xq":"2"}
```

Response example:
```json
{
  "code":200,
  "msg":"SUCCESS",
  "content":[
    {"ZCMC":"Week 1","ZC":1},
    {"ZCMC":"Week 2","ZC":2}
  ]
}
```

Field notes:
- `ZC`: week number.

### 2.5 Weekly timetable matrix
- Method: POST
- Path: `/app/Kbcx/query`
- Auth: bearer
- Content-Type: `application/json`

Request example:
```http
POST /incoSpringBoot/app/Kbcx/query HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/json

{"xn":"2025-2026","xq":"2","zc":"1","type":"json"}
```

Response example:
```json
{
  "code":200,
  "msg":"SUCCESS",
  "content":[
    {"rqList":[{"XQMC":"Mon","XQMC_EN":"Mon","XQDM":1,"RQ":"2026-03-09"}]},
    {"jcList":[{"KSJC":1,"JSJC":2,"DJ":1,"SJ":"08:30-10:15"}]},
    {"kcxxList":[{"XQJ":1,"KCDM":"COMP2054","KBXX":"CourseName [Room]","DJ":3,"XB":2}]}
  ]
}
```

Field notes:
- `rqList`: dates within the week.
- `jcList`: time blocks.
- `kcxxList`: course cells within the grid.

---

## 3. Scores

### 3.1 Score list
- Method: POST
- Path: `/app/cjgl/xscjList?_lang=zh_CN`
- Auth: bearer
- Content-Type: `application/json`

Request example:
```http
POST /incoSpringBoot/app/cjgl/xscjList?_lang=zh_CN HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/json

{"xn":"2025-2026","xq":"1","qzqmFlag":"qm","type":"json"}
```

Response example:
```json
{
  "code":200,
  "msg":null,
  "msg_en":null,
  "content":[
    {
      "id":"<id>",
      "xnxq":"2025Fall",
      "xnxqdm":"2025-20261",
      "kcdm":"22MX44001",
      "kcmc":"Intro to Labor",
      "kcmc_en":"Introduction to Labor Education",
      "xf":0,
      "zf":"99",
      "kcxz":"Required",
      "rwh":"2025-2026-1-22MX44001-001",
      "fxcj":[]
    }
  ]
}
```

Field notes:
- `zf` may be numeric or grade text (A/A- etc).
- `fxcj[]`: breakdown items (may be empty).

### 3.2 GPA and rank
- Method: POST
- Path: `/app/cjgl/xfj`
- Auth: bearer
- Content-Type: `application/json`

Request example:
```http
POST /incoSpringBoot/app/cjgl/xfj HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/json

{"type":"json","ksxnxq":"-1-1","jsxnxq":"-1-1","pylx":"1"}
```

Response example:
```json
{
  "code":200,
  "msg":null,
  "content":{"xfj":{"ZYZRS":616,"RANK":"61","XFJ":90.165}}
}
```

Field notes:
- `XFJ`: GPA-like score.
- `RANK`: rank within major.
- `ZYZRS`: total count in major.

---

## 4. Empty rooms

### 4.1 Building list
- Method: POST
- Path: `/app/commapp/queryjxllist`
- Auth: bearer
- Content-Type: `application/x-www-form-urlencoded`
- Request body: empty

Request example:
```http
POST /incoSpringBoot/app/commapp/queryjxllist HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/x-www-form-urlencoded

```

Response example:
```json
{
  "code":200,
  "msg":"SUCCESS",
  "content":[
    {"MC":"Teaching Building II","DM":"14","MC_EN":"Teaching Building II"}
  ]
}
```

Field notes:
- `DM`: building code used by empty room query.

### 4.2 Empty room occupancy by date and building
- Method: POST
- Path: `/app/kbrcbyapp/querycdzyxx`
- Auth: bearer
- Content-Type: `application/x-www-form-urlencoded`

Request example:
```http
POST /incoSpringBoot/app/kbrcbyapp/querycdzyxx HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/x-www-form-urlencoded

nyr=2026-03-09&jxl=14
```

Response example:
```json
{
  "code":200,
  "msg":"SUCCESS",
  "content":[
    {"CDMC":"T2102","CDMC_EN":"T2102","DJ1":"0","DJ2":"0","DJ3":"0","DJ4":"0","DJ5":"0","DJ6":"0"}
  ]
}
```

Field notes:
- `DJ1`..`DJ6`: block occupancy flags. `0` means free in captured samples.
- Time mapping:
  - DJ1: 08:30-10:15
  - DJ2: 10:30-12:15
  - DJ3: 14:00-15:45
  - DJ4: 16:00-17:45
  - DJ5: 18:45-20:30
  - DJ6: 20:45-22:30

---

## 5. Course selection (not in current product scope, but usable)

### 5.1 Selected courses
- Method: POST
- Path: `/app/Xsxk/queryYxkc?_lang=zh_CN`
- Auth: bearer
- Content-Type: `application/json`

Request example:
```http
POST /incoSpringBoot/app/Xsxk/queryYxkc?_lang=zh_CN HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/json

{"RoleCode":"01","p_pylx":"1","p_xn":"2025-2026","p_xq":"2","p_xnxq":"2025-20262","p_gjz":"","p_kc_gjz":"","p_xkfsdm":"yixuan"}
```

Response example (placeholder; response is large):
```json
{"code":200,"msg":null,"content":{"yxkcList":[{"kcdm":"<course_code>","kcmc":"<course_name>","dgjsmc":"<teachers>"}]}}
```

Field notes:
- Response includes both selected courses and configuration payloads.
- `kcxx` may include HTML.

### 5.2 Available course tasks
- Method: POST
- Path: `/app/Xsxk/queryKxrw?_lang=zh_CN`
- Auth: bearer
- Content-Type: `application/json`

Request example:
```http
POST /incoSpringBoot/app/Xsxk/queryKxrw?_lang=zh_CN HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/json

{"RoleCode":"01","p_pylx":"1","p_xn":"2025-2026","p_xq":"2","p_xnxq":"2025-20262","p_gjz":"","p_kc_gjz":"","p_xkfsdm":"bx-b-b","pageNum":1,"pageSize":10}
```

Response example (placeholder):
```json
{"code":200,"msg":null,"content":{"yxkcList":[{"rwh":"<task_id>","kcdm":"<course_code>","kcmc":"<course_name>","xf":3}]}}
```

Field notes:
- `p_xkfsdm` selects course pools (examples: `bx-b-b`, `xx-b-b`, `ty-b-b`).
- `kcxx` often contains HTML with instructor/time info.

### 5.3 Add to cart
- Method: POST
- Path: `/app/Xsxk/addGouwuche?_lang=zh_CN`
- Auth: bearer
- Content-Type: `application/json`

Request example:
```http
POST /incoSpringBoot/app/Xsxk/addGouwuche?_lang=zh_CN HTTP/1.1
Host: mjw.hitsz.edu.cn
Authorization: bearer <access_token>
Content-Type: application/json

{"RoleCode":"01","p_pylx":"1","p_xn":"2025-2026","p_xq":"2","p_xkfsdm":"bx-b-b","p_xktjz":"rwtjzyx","p_id":"<task_id>"}
```

Response example (failure):
```json
{"code":200,"msg":null,"content":{"message":"NOT_IN_TIME_WINDOW","jg":"-1","gjhczztm":"XKGL.OPERATE.RESULT_BZXKSJN"}}
```

Field notes:
- Failure responses still return HTTP 200 with `jg=-1`.
- Rate limit errors appear as `msg` and `content.message`.

---

## Pitfalls and TODO
- Keep the same session; some endpoints refresh `JSESSIONID`.
- RSA params are present but login currently succeeds with plaintext password.
- Some responses contain HTML strings; avoid parsing as plain text only.
