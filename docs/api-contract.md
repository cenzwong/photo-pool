# API Contract — Wedding Face Gallery

## Overview

Frontend (GitHub Pages) talks to API Gateway endpoints. API returns JSON and uses **presigned S3 URLs** for uploads/downloads.

Base URL:

* `https://{api_id}.execute-api.{region}.amazonaws.com/{stage}`

Auth:

* MVP: `X-Wedding-Code: <shared_code>` header required on all endpoints.
* Admin endpoints additionally require `X-Admin-Token: <admin_token>`.

Common response headers:

* `Content-Type: application/json`

Error format (all endpoints):

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "requestId": "string"
  }
}
```

---

## 1) GET /presign-upload

Generate presigned PUT URL for direct upload to S3.

### Request

Query params:

* `kind` (required): `photo` | `video`
* `contentType` (required): e.g. `image/jpeg`, `image/png`, `video/mp4`
* `ext` (optional): e.g. `jpg`, `png`, `mp4`

Headers:

* `X-Wedding-Code: ...`

### Response 200

```json
{
  "id": "uuid-string",
  "kind": "photo",
  "s3Key": "uploads/raw/uuid-string.jpg",
  "putUrl": "https://s3-presigned-put-url",
  "expiresInSeconds": 900,
  "maxBytes": 52428800
}
```

Notes:

* `id` becomes `photoId`/`videoId` across the system.
* `maxBytes` is an enforced frontend limit; backend may also validate metadata on finalize (optional).

### Response 4xx examples

* 400 invalid params
* 401 missing/invalid wedding code

---

## 2) POST /finalize-upload (optional but recommended)

Tell backend an upload finished and should be processed. Helps if S3 events are delayed.

### Request

Headers:

* `X-Wedding-Code: ...`

Body:

```json
{
  "id": "uuid-string",
  "kind": "photo",
  "s3Key": "uploads/raw/uuid-string.jpg"
}
```

### Response 202

```json
{
  "accepted": true,
  "id": "uuid-string",
  "kind": "photo"
}
```

---

## 3) GET /clusters

List current clusters for People page.

### Request

Headers:

* `X-Wedding-Code: ...`

Query params (optional):

* `limit`: default 50, max 200

### Response 200

```json
{
  "updatedAt": "2026-01-15T19:22:11Z",
  "clusters": [
    {
      "clusterId": "c-0012",
      "repFaceId": "face-abc",
      "repThumbUrl": "https://s3-presigned-get-url",
      "memberCount": 17,
      "photoCount": 96,
      "lastUpdatedAt": "2026-01-15T19:21:52Z"
    }
  ]
}
```

Notes:

* `repThumbUrl` is short-lived presigned GET (e.g. 10–30 minutes).

---

## 4) GET /clusters/{clusterId}/photos

List photos belonging to a cluster.

### Request

Headers:

* `X-Wedding-Code: ...`

Path:

* `clusterId` required

Query params (optional):

* `limit`: default 200, max 1000
* `cursor`: opaque pagination token (optional for MVP)

### Response 200

```json
{
  "clusterId": "c-0012",
  "photos": [
    {
      "photoId": "uuid-photo-1",
      "s3Key": "uploads/raw/uuid-photo-1.jpg",
      "downloadUrl": "https://s3-presigned-get-url",
      "uploadedAt": "2026-01-15T18:40:12Z"
    }
  ],
  "nextCursor": null
}
```

---

## 5) POST /admin/rebuild/incremental (admin)

Trigger incremental rebuild manually (mostly for testing). Normally invoked by schedule.

### Request

Headers:

* `X-Wedding-Code: ...`
* `X-Admin-Token: ...`

Body (optional):

```json
{
  "maxFaces": 500
}
```

### Response 200

```json
{
  "ok": true,
  "processedFaces": 42,
  "createdClusters": 1,
  "updatedClusters": 6,
  "checkpoint": "2026-01-15T19:21:52Z"
}
```

---

## 6) POST /admin/rebuild/full (admin, optional)

Full rebuild from all faces for best quality.

### Request

Headers:

* `X-Wedding-Code: ...`
* `X-Admin-Token: ...`

### Response 200

```json
{
  "ok": true,
  "clusters": 14,
  "faces": 312,
  "photos": 821,
  "finishedAt": "2026-01-16T01:05:33Z"
}
```

---

## Status codes summary

* 200 OK: success
* 202 Accepted: async processing accepted
* 400 Bad Request: invalid input
* 401 Unauthorized: missing/invalid wedding code
* 403 Forbidden: missing/invalid admin token
* 404 Not Found: clusterId not found
* 429 Too Many Requests: rate limit
* 500 Internal Error: unexpected

