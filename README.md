# photo-pool

photo-pool/
│
├── frontend/                      # GitHub Pages frontend
│   ├── index.html                 # Main entry point for the website
│   ├── scripts/                   # JavaScript files
│   │   └── app.js                 # Frontend logic
│   ├── styles/                    # Stylesheets
│   │   └── app.css                # Main CSS file
│   ├── assets/                    # Static assets (images, icons, etc.)
│       ├── images/
│       ├── icons/
│
├── backend/                       # Serverless backend
│   ├── apis/                      # API endpoints for Lambda
│   │   ├── finalize_upload.py     # Logic to finalize uploads
│   │   ├── presign_upload.py      # Generate presigned URLs
│   ├── jobs/                      # Background jobs
│   │   ├── incremental_rebuild.py # Incremental cluster rebuild logic
│   ├── utils/                     # Utility scripts
│       ├── rekognition_client.py  # AWS Rekognition client helper
│
├── infrastructure/                # Infrastructure-as-code files
│   ├── template.yaml              # CloudFormation / SAM template
│
├── tests/                         # Test cases for backend and frontend
│   ├── backend/                   # Backend-specific tests
│   ├── frontend/                  # Frontend-specific tests
│
├── docs/                          # Project documentation
│   └── README.md                  # Documentation for different components
│
├── .github/                       # GitHub-specific workflows and configs
│   ├── workflows/                 # GitHub Actions CI/CD workflows
│       └── deploy.yml             # Workflow for deployment
│
└── blueprint.md                   # The blueprint document
