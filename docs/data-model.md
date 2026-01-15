# Data Model — DynamoDB + S3 (Wedding Face Gallery)

## S3 Keys

* Raw uploads:

  * `uploads/raw/{id}.{ext}` (photos/videos)
* Face thumbnails:

  * `faces/thumbs/{faceId}.jpg`

Bucket is private. All browser access is via presigned URLs.

---

## DynamoDB Tables

### 1) Photo

Maps photoId → S3 object and processing status.

**Primary key**

* PK: `photoId` (string UUID)

**Attributes**

* `s3KeyRaw` (string) e.g. `uploads/raw/{photoId}.jpg`
* `uploadedAt` (string ISO) e.g. `2026-01-15T18:40:12Z`
* `status` (string enum): `RAW | INDEXED | FAILED`
* `faceCount` (number)
* optional: `uploaderTag` (string), `contentType` (string), `error` (string)

**Example item**

```json
{
  "photoId": "2dd4c2b0-8b6b-4b3d-9c8a-60d1d5d3d8aa",
  "s3KeyRaw": "uploads/raw/2dd4c2b0-8b6b-4b3d-9c8a-60d1d5d3d8aa.jpg",
  "uploadedAt": "2026-01-15T18:40:12Z",
  "status": "INDEXED",
  "faceCount": 3
}
```

---

### 2) FaceOccurrence

Represents “faceId appears in photoId” + thumbnail + bbox.

**Primary key**

* PK: `faceId` (string; Rekognition FaceId)
* SK: `photoId` (string UUID)

**Attributes**

* `thumbS3Key` (string) e.g. `faces/thumbs/{faceId}.jpg`
* `bbox` (map) e.g. `{ "Left": 0.1, "Top": 0.2, "Width": 0.3, "Height": 0.3 }`
* `confidence` (number)
* `indexedAt` (string ISO)

**GSI1** (reverse lookup: photo → faces)

* GSI1PK: `gsi1pk` = `photoId`
* GSI1SK: `gsi1sk` = `faceId`

**Example item**

```json
{
  "faceId": "face-abc123",
  "photoId": "2dd4c2b0-8b6b-4b3d-9c8a-60d1d5d3d8aa",
  "thumbS3Key": "faces/thumbs/face-abc123.jpg",
  "bbox": {"Left":0.12,"Top":0.18,"Width":0.22,"Height":0.22},
  "confidence": 99.1,
  "indexedAt": "2026-01-15T18:40:20Z",
  "gsi1pk": "2dd4c2b0-8b6b-4b3d-9c8a-60d1d5d3d8aa",
  "gsi1sk": "face-abc123"
}
```

---

### 3) Cluster

Cluster summary for People page.

**Primary key**

* PK: `clusterId` (string) e.g. `c-0001`

**Attributes**

* `repFaceId` (string)
* `repThumbS3Key` (string)
* `memberCount` (number)
* `photoCount` (number)
* `updatedAt` (string ISO)
* optional: `canonicalClusterId` (string; only if implementing merge redirects)

**Example item**

```json
{
  "clusterId": "c-0012",
  "repFaceId": "face-abc123",
  "repThumbS3Key": "faces/thumbs/face-abc123.jpg",
  "memberCount": 17,
  "photoCount": 96,
  "updatedAt": "2026-01-15T19:21:52Z"
}
```

---

### 4) ClusterMember

Membership mapping cluster → faces.

**Primary key**

* PK: `clusterId` (string)
* SK: `faceId` (string)

**Attributes**

* `addedAt` (string ISO)
* optional: `repScore` (number)

**GSI1** (face → cluster)

* GSI1PK: `gsi1pk` = `faceId`
* GSI1SK: `gsi1sk` = `clusterId`

**Example item**

```json
{
  "clusterId": "c-0012",
  "faceId": "face-abc123",
  "addedAt": "2026-01-15T19:10:00Z",
  "gsi1pk": "face-abc123",
  "gsi1sk": "c-0012"
}
```

---

### 5) ClusterPhoto (recommended)

Fast path for cluster → photos.

**Primary key**

* PK: `clusterId` (string)
* SK: `photoId` (string)

**Attributes**

* `addedAt` (string ISO)
* optional: `bestFaceId` (string)

**Example item**

```json
{
  "clusterId": "c-0012",
  "photoId": "2dd4c2b0-8b6b-4b3d-9c8a-60d1d5d3d8aa",
  "addedAt": "2026-01-15T19:12:33Z"
}
```

---

### 6) JobState

Stores incremental rebuild checkpoint.

**Primary key**

* PK: `jobName` (string) e.g. `incremental-rebuild`

**Attributes**

* `lastProcessedAt` (string ISO)
* `lastRunAt` (string ISO)
* optional: `stats` (map)

**Example item**

```json
{
  "jobName": "incremental-rebuild",
  "lastProcessedAt": "2026-01-15T19:21:52Z",
  "lastRunAt": "2026-01-15T19:22:11Z"
}
```

---

## Notes on incremental rebuild inputs

The incremental job needs an efficient way to find new faces since `lastProcessedAt`.

Two approaches:

1. **Simple** (wedding scale): Scan FaceOccurrence with filter on `indexedAt > lastProcessedAt` and dedupe faceIds.
2. **Cleaner**: Add a `FaceIngest` table keyed by day bucket to query new faces without scans.

Start with #1; upgrade to #2 only if needed.

---

## Rekognition Collection usage

* On ingest: `IndexFaces` returns `FaceId` values stored in FaceOccurrence.
* For clustering: use similarity search (e.g., `SearchFaces`) to find edges between faces.

Threshold tuning (example):

* Similarity ≥ 90: candidate same person
* Similarity ≥ 95: strong same person
