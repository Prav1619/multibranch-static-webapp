# multibranch-static-webapp

# Static Website CI/CD – Jenkins Multibranch Pipeline

This is a static website deployed to EC2 (Amazon Linux) using Jenkins Multibranch Pipeline.

## Branch Strategy
- `feature/*` → CI only
- `dev` → Deploys to DEV EC2
- `qa` → Deploys to QA EC2
- `main` → Deploys to PROD EC2 (with approval)

## Build
```bash
npm install
npm run build
