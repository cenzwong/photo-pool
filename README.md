# wedding-face-gallery

Placeholder repository skeleton for the wedding "Pick-a-Face" photo system.

```
wedding-face-gallery/
├─ README.md
├─ .gitignore
├─ LICENSE
├─ docs/
│  ├─ architecture.md
│  ├─ api-contract.md
│  ├─ data-model.md
│  ├─ runbooks.md
│  └─ decisions.md
├─ frontend/
│  ├─ README.md
│  ├─ index.html
│  ├─ assets/
│  │  ├─ styles.css
│  │  └─ logo.svg
│  └─ src/
│     ├─ app.js
│     ├─ api.js
│     ├─ upload.js
│     ├─ clusters.js
│     └─ download.js
├─ sam/
│  ├─ README.md
│  ├─ template.yaml
│  ├─ samconfig.toml
│  ├─ env.example.json
│  ├─ events/
│  │  ├─ s3_object_created.json
│  │  ├─ api_presign.json
│  │  └─ api_list_clusters.json
│  └─ policies/
│     └─ rekognition_notes.md
├─ backend/
│  ├─ README.md
│  ├─ pyproject.toml
│  ├─ src/
│  │  └─ wedding_backend/
│  │     ├─ __init__.py
│  │     ├─ config.py
│  │     ├─ logging.py
│  │     ├─ aws/
│  │     │  ├─ s3.py
│  │     │  ├─ dynamo.py
│  │     │  └─ rekognition.py
│  │     ├─ domain/
│  │     │  ├─ models.py
│  │     │  ├─ clustering.py
│  │     │  └─ thumbnails.py
│  │     └─ handlers/
│  │        ├─ presign_upload.py
│  │        ├─ finalize_upload.py
│  │        ├─ list_clusters.py
│  │        ├─ cluster_photos.py
│  │        ├─ ingest_s3_event.py
│  │        └─ rebuild_incremental.py
│  ├─ tests/
│  │  ├─ unit/
│  │  └─ integration/
│  └─ scripts/
│     ├─ local_smoke_test.sh
│     └─ seed_test_data.py
├─ tools/
│  ├─ format.sh
│  └─ make_release_notes.py
└─ .github/
   └─ workflows/
      ├─ deploy-frontend.yml
      └─ deploy-sam.yml
```
