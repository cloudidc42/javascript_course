# Part 77: CI/CD Pipeline (Steps 1511-1530)

## บทนำ: CI/CD คืออะไร?

**CI/CD** ย่อมาจาก:
- **CI** = Continuous Integration (การรวมโค้ดอย่างต่อเนื่อง)
- **CD** = Continuous Delivery/Deployment (การส่งมอบ/deploy อย่างต่อเนื่อง)

เป็นแนวปฏิบัติในการพัฒนา Software ที่ automation ขั้นตอนต่างๆ ตั้งแต่การ commit โค้ด ไปจนถึงการ deploy ขึ้น production

```
โดยไม่มี CI/CD (Manual Process):
Developer → commit → "หวังว่ามันจะ work" → manual test → manual build → manual deploy

ด้วย CI/CD (Automated Process):
Developer → commit → CI ทดสอบอัตโนมัติ → CD build อัตโนมัติ → Deploy อัตโนมัติ
```

---

## Step 1511: Continuous Integration (CI) คืออะไร?

**CI** คือการ merge โค้ดจาก developer หลายคนเข้า repository หลักบ่อยๆ (อย่างน้อยวันละครั้ง) พร้อมทดสอบอัตโนมัติ

### ปัญหาก่อนมี CI: "Integration Hell"

```
ทีม 5 คน ทำงาน 2 สัปดาห์แยกกัน:
├── Developer A: เพิ่มฟีเจอร์ A
├── Developer B: เพิ่มฟีเจอร์ B
├── Developer C: แก้ bug C
├── Developer D: refactor module X
└── Developer E: เปลี่ยน API contract

เมื่อ merge ทุกอย่างพร้อมกัน = 🔥 CHAOS
- Merge conflicts ไม่สิ้นสุด
- Tests ล้มเหลวเป็นร้อย
- "มันทำงานในเครื่องฉัน" ← ปัญหาคลาสสิก
```

### ประโยชน์ของ CI

1. **Detect bugs เร็ว**: พบปัญหาเมื่อ code เพิ่งถูกเขียน
2. **Reduce integration risk**: merge บ่อยๆ = conflict น้อย
3. **Consistent environment**: รันบน server เดียวกันทุกครั้ง
4. **Confidence**: ทีมรู้ว่า main branch ทำงาน

### CI Workflow พื้นฐาน

```
1. Developer push code
2. CI server detect change
3. CI run automated tests
4. CI report results
5. Developer fix if failed
6. PR approved เฉพาะเมื่อ CI pass
```

---

## Step 1512: Continuous Delivery vs Continuous Deployment

```
Continuous Integration:
  code → build → test
  
Continuous Delivery:
  code → build → test → staging → [MANUAL APPROVAL] → production
  
Continuous Deployment:
  code → build → test → staging → [AUTOMATED] → production
```

### Continuous Delivery

```
คุณสมบัติ:
✓ ทุก commit พร้อม deploy ได้ตลอดเวลา
✓ มีขั้นตอน manual approval ก่อน production
✓ Business ตัดสินใจว่าจะ release เมื่อไหร่
✓ ลด deployment risk เพราะทำบ่อย
```

### Continuous Deployment

```
คุณสมบัติ:
✓ ทุก commit ที่ผ่าน tests → deploy อัตโนมัติ
✓ ต้องการ test coverage สูงมาก
✓ เหมาะกับ SaaS products
✓ ต้องการ feature flags สำหรับ gradual rollout
```

---

## Step 1513: GitHub Actions - Overview

**GitHub Actions** คือ CI/CD platform ที่ built-in ใน GitHub ฟรีสำหรับ public repos

### แนวคิดหลัก

```yaml
Workflow:
  ├── Trigger (on)     ← เมื่อไหรจะรัน?
  ├── Jobs             ← งานหลัก (parallel ได้)
  │   ├── Steps        ← ขั้นตอนย่อยๆ (sequential)
  │   │   ├── Action  ← reusable unit
  │   │   └── Command ← shell command
  │   └── Runner       ← machine ที่รันงาน
  └── Artifacts        ← ผลลัพธ์จาก job
```

### โครงสร้างไฟล์

```
.github/
└── workflows/
    ├── ci.yml        ← CI pipeline
    ├── deploy.yml    ← Deploy pipeline
    └── release.yml   ← Release pipeline
```

---

## Step 1514: Workflow YAML Syntax

```yaml
# .github/workflows/ci.yml

# ชื่อ workflow
name: CI Pipeline

# Triggers: เมื่อไหร่จะรัน
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # ทุกวันจันทร์ 02:00 UTC
  workflow_dispatch:       # trigger ด้วยตัวเอง (manual)

# Jobs: งานที่รันใน workflow
jobs:
  test:                    # ชื่อ job
    name: Run Tests
    
    # Runner OS
    runs-on: ubuntu-latest
    
    # Strategy matrix (รันหลาย variants)
    strategy:
      matrix:
        node-version: [18, 20, 21]
        os: [ubuntu-latest, windows-latest]
    
    # Steps: ขั้นตอนใน job
    steps:
      # 1. Checkout code
      - name: Checkout repository
        uses: actions/checkout@v4
      
      # 2. Setup Node.js
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      # 3. Install dependencies
      - name: Install dependencies
        run: npm ci  # ci (clean install) แทน install
      
      # 4. Run linter
      - name: Lint code
        run: npm run lint
      
      # 5. Run tests
      - name: Run tests
        run: npm test
      
      # 6. Upload coverage
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
```

---

## Step 1515: Triggers ต่างๆ

### Push Trigger

```yaml
on:
  push:
    branches:
      - main
      - 'release/*'      # wildcard
      - '!release/draft' # ยกเว้น
    tags:
      - 'v*'             # tags ที่ขึ้นต้นด้วย v
    paths:
      - 'src/**'         # เฉพาะไฟล์ใน src/
      - 'package.json'
    paths-ignore:
      - '**.md'          # ยกเว้นไฟล์ markdown
```

### Pull Request Trigger

```yaml
on:
  pull_request:
    types:
      - opened
      - synchronize      # push ใหม่เข้า PR
      - reopened
    branches:
      - main
```

### Schedule Trigger

```yaml
on:
  schedule:
    # รัน tests ทุกวัน 03:00 UTC
    - cron: '0 3 * * *'
    # ทุกวันจันทร์-ศุกร์ 09:00 UTC
    - cron: '0 9 * * 1-5'
```

### Manual Trigger (workflow_dispatch)

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: false
        type: string
```

---

## Step 1516: Jobs และ Steps

### Job Dependencies

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
  
  test:
    runs-on: ubuntu-latest
    needs: build    # รอ build เสร็จก่อน
    steps:
      - run: echo "Testing..."
  
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build, test]  # รอทั้ง build และ test
    steps:
      - run: echo "Deploy to staging..."
  
  deploy-prod:
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'  # เฉพาะ main branch
    steps:
      - run: echo "Deploy to production..."
```

### Job Outputs

```yaml
jobs:
  prepare:
    runs-on: ubuntu-latest
    
    outputs:
      version: ${{ steps.get-version.outputs.version }}
    
    steps:
      - id: get-version
        run: echo "version=$(node -p "require('./package.json').version")" >> $GITHUB_OUTPUT
  
  deploy:
    needs: prepare
    runs-on: ubuntu-latest
    steps:
      - name: Deploy version
        run: echo "Deploying version ${{ needs.prepare.outputs.version }}"
```

---

## Step 1517: Runners

**Runners** คือ servers ที่รัน workflows

### GitHub-hosted Runners

```yaml
runs-on: ubuntu-latest    # Ubuntu 22.04
runs-on: ubuntu-22.04     # Ubuntu 22.04 (pinned)
runs-on: windows-latest   # Windows Server 2022
runs-on: macos-latest     # macOS 14 (M1)
```

### Self-hosted Runners

```yaml
runs-on: self-hosted
runs-on: [self-hosted, linux, x64]
runs-on: [self-hosted, gpu]      # Custom labels
```

```bash
# ติดตั้ง self-hosted runner
# 1. ไปที่ GitHub Repo Settings → Actions → Runners
# 2. New self-hosted runner
# 3. รัน script ที่ GitHub ให้

./config.sh --url https://github.com/OWNER/REPO \
            --token YOUR_TOKEN \
            --name my-runner \
            --labels ubuntu,x64,custom

./run.sh  # เริ่มต้น runner
```

---

## Step 1518: Environment Variables และ Secrets

### Environment Variables

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    
    # Environment variables สำหรับ job ทั้งหมด
    env:
      NODE_ENV: production
      API_URL: https://api.example.com
    
    steps:
      - name: Build with env vars
        run: echo "Building for $NODE_ENV"
        env:
          # Override เฉพาะ step นี้
          API_URL: https://staging-api.example.com
```

### Secrets

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: ./deploy.sh
```

```
การเพิ่ม Secrets:
1. GitHub Repository → Settings → Secrets and Variables → Actions
2. New repository secret
3. ใส่ชื่อและ value

Organization Secrets:
- Shared ระหว่าง repositories ทั้งหมดใน organization
- Manage ที่ Organization → Settings → Secrets
```

### Environment Secrets

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment: production  # ← ใช้ environment-specific secrets
    
    steps:
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}  # production API key
        run: ./deploy-prod.sh
```

---

## Step 1519: Caching Dependencies

Caching ลด build time โดย cache node_modules

```yaml
steps:
  - uses: actions/checkout@v4
  
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'  # Built-in caching
  
  - run: npm ci
```

### Manual Caching

```yaml
steps:
  - uses: actions/checkout@v4
  
  - name: Cache node_modules
    uses: actions/cache@v3
    id: cache-npm
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-
  
  - name: Install dependencies
    if: steps.cache-npm.outputs.cache-hit != 'true'
    run: npm ci
```

### Caching Build Output

```yaml
steps:
  - name: Cache build
    uses: actions/cache@v3
    with:
      path: .next/cache
      key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**.[jt]s', '**.[jt]sx') }}
      restore-keys: |
        ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-
```

---

## Step 1520: Complete CI Workflow - Run Tests on PR

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  setup:
    name: Setup & Validate
    runs-on: ubuntu-latest
    
    outputs:
      node-version: ${{ steps.node.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
      
      - id: node
        run: echo "version=$(cat .nvmrc || echo '20')" >> $GITHUB_OUTPUT
  
  lint:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    needs: setup
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ needs.setup.outputs.node-version }}
          cache: 'npm'
      
      - run: npm ci
      
      - name: TypeScript check
        run: npx tsc --noEmit
      
      - name: ESLint
        run: npm run lint -- --max-warnings 0
      
      - name: Prettier check
        run: npx prettier --check src
  
  test:
    name: Unit & Integration Tests
    runs-on: ubuntu-latest
    needs: setup
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ needs.setup.outputs.node-version }}
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
        env:
          CI: true
      
      - name: Upload coverage report
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: coverage/
          retention-days: 7
      
      - name: Coverage comment on PR
        uses: davelosert/vitest-coverage-report-action@v2
        if: github.event_name == 'pull_request'
  
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: [lint, test]
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium
      
      - name: Build app
        run: npm run build
      
      - name: Run E2E tests
        run: npx playwright test
      
      - name: Upload Playwright report
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
  
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/
          retention-days: 1
```

---

## Step 1521: Deploy to Vercel

```yaml
# .github/workflows/deploy-vercel.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Vercel CLI
        run: npm install -g vercel@latest
      
      - name: Pull Vercel environment info
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Build Project
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Deploy to Vercel
        id: deploy
        run: |
          URL=$(vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }})
          echo "url=$URL" >> $GITHUB_OUTPUT
      
      - name: Comment PR with URL
        uses: actions/github-script@v7
        if: github.event_name == 'pull_request'
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🚀 Deploy Preview: ${{ steps.deploy.outputs.url }}`
            })
```

### Vercel Setup

```bash
# ติดตั้ง Vercel CLI
npm install -g vercel

# เชื่อมต่อโปรเจค
vercel link

# ดู project ID
cat .vercel/project.json
# { "orgId": "team_xxx", "projectId": "prj_xxx" }

# Secrets ที่ต้องการ:
# VERCEL_TOKEN - จาก vercel.com/account/tokens
# VERCEL_ORG_ID - จาก .vercel/project.json
# VERCEL_PROJECT_ID - จาก .vercel/project.json
```

---

## Step 1522: Deploy to AWS

```yaml
# .github/workflows/deploy-aws.yml
name: Deploy to AWS

on:
  push:
    branches: [main]

env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: my-app
  ECS_SERVICE: my-service
  ECS_CLUSTER: my-cluster
  CONTAINER_NAME: my-app

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build, tag, and push Docker image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Build Docker image
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      - name: Download ECS task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition my-task-def \
            --query taskDefinition > task-definition.json
      
      - name: Update ECS task definition with new image
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}
      
      - name: Deploy to Amazon ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
      
      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "✅ Deployed to production: ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## Step 1523: Publish npm Package

```yaml
# .github/workflows/publish-npm.yml
name: Publish to npm

on:
  push:
    tags:
      - 'v*'  # รัน เมื่อ push tag v1.0.0 เป็นต้น

jobs:
  publish:
    runs-on: ubuntu-latest
    
    permissions:
      contents: write   # สร้าง GitHub Release
      packages: write   # publish ไปยัง GitHub Packages
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'
      
      - run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
      
      - name: Get version from tag
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
      
      - name: Publish to npm
        run: npm publish --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      
      - name: Create GitHub Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ steps.version.outputs.version }}
          body: |
            ## Changes
            See [CHANGELOG.md](CHANGELOG.md) for details.
          draft: false
          prerelease: false
      
      - name: Publish to GitHub Packages
        uses: actions/setup-node@v4
        with:
          registry-url: 'https://npm.pkg.github.com'
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## Step 1524: Branch Protection Rules

```
ตั้งค่า Branch Protection ใน GitHub:
Repository → Settings → Branches → Add rule

สำหรับ main branch:
✓ Require a pull request before merging
  - Require approvals: 2
  - Dismiss stale pull request approvals when new commits are pushed
  
✓ Require status checks to pass before merging
  - Require branches to be up to date before merging
  - Status checks: ci/test, ci/lint, ci/build
  
✓ Require conversation resolution before merging

✓ Require linear history (no merge commits)

✓ Do not allow bypassing the above settings
```

```yaml
# CODEOWNERS file (.github/CODEOWNERS)
# Global owner - ต้อง approve ทุก PR
* @team-lead

# Specific directories
/src/api/ @backend-team
/src/ui/  @frontend-team
/.github/ @devops-team

# Specific file types
*.sql @database-team
```

---

## Step 1525: Complete Deployment Workflow

```yaml
# .github/workflows/full-pipeline.yml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  ci:
    name: CI Tests
    uses: ./.github/workflows/ci.yml  # Reusable workflow
  
  deploy-staging:
    name: Deploy to Staging
    needs: ci
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to staging
        run: ./scripts/deploy.sh staging
        env:
          DEPLOY_KEY: ${{ secrets.STAGING_DEPLOY_KEY }}
  
  deploy-production:
    name: Deploy to Production
    needs: [ci, deploy-staging]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/
      
      - name: Deploy to production
        run: ./scripts/deploy.sh production
        env:
          DEPLOY_KEY: ${{ secrets.PROD_DEPLOY_KEY }}
      
      - name: Tag release
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          VERSION=$(node -p "require('./package.json').version")
          git tag -a "v$VERSION" -m "Release v$VERSION"
          git push origin "v$VERSION"
      
      - name: Notify team
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "🚀 Production deployed: https://example.com",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Production Deployment*\nCommit: `${{ github.sha }}`\nBy: @${{ github.actor }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## Step 1526: Reusable Workflows

```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
      run-e2e:
        required: false
        type: boolean
        default: false
    secrets:
      NPM_TOKEN:
        required: false
    outputs:
      test-result:
        description: 'Test result'
        value: ${{ jobs.test.outputs.result }}

jobs:
  test:
    runs-on: ubuntu-latest
    outputs:
      result: ${{ steps.run.outputs.result }}
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
      
      - run: npm ci
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      
      - id: run
        name: Run tests
        run: |
          npm test
          echo "result=success" >> $GITHUB_OUTPUT
```

```yaml
# .github/workflows/main-pipeline.yml
name: Main Pipeline

on:
  push:
    branches: [main]

jobs:
  test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20'
      run-e2e: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
  
  deploy:
    needs: test
    uses: ./.github/workflows/reusable-deploy.yml
```

---

## Step 1527: GitLab CI Overview

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "20"
  CACHE_KEY: "${CI_COMMIT_REF_SLUG}"

# Template
.node-setup:
  image: node:${NODE_VERSION}
  before_script:
    - npm ci

# Build job
build:
  extends: .node-setup
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 day

# Test jobs
lint:
  extends: .node-setup
  stage: test
  script:
    - npm run lint

test:unit:
  extends: .node-setup
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Deploy to staging
deploy:staging:
  stage: deploy
  script:
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# Deploy to production (manual)
deploy:production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
  when: manual  # ต้องกด manual
  only:
    - main
```

---

## Step 1528: Deployment Strategies

### Rolling Deployment

```
เดิม: 4 instances รัน v1.0
Rolling update เป็น v2.0:

t=0: [v1] [v1] [v1] [v1]
t=1: [v2] [v1] [v1] [v1]  ← replace 1 instance
t=2: [v2] [v2] [v1] [v1]
t=3: [v2] [v2] [v2] [v1]
t=4: [v2] [v2] [v2] [v2]  ← สำเร็จ

ข้อดี: zero downtime
ข้อเสีย: หลาย versions รันพร้อมกันชั่วคราว
```

### Blue-Green Deployment

```
Blue (current): production traffic
Green (new): เตรียม environment ใหม่

t=0: Users → Blue (v1.0)
              Green (v2.0) ← กำลัง test

t=1: Test green สำเร็จ
     Users → Green (v2.0)  ← switch traffic
              Blue (v1.0)  ← standby (rollback ได้ทันที)

ข้อดี: rollback ง่าย, ไม่มี downtime
ข้อเสีย: ต้องการ infrastructure 2x
```

```yaml
# GitHub Actions - Blue-Green with AWS
- name: Blue-Green Deployment
  run: |
    # หา color ปัจจุบัน
    CURRENT=$(aws ec2 describe-tags \
      --filters "Name=key,Values=Slot" \
      --query "Tags[0].Value" --output text)
    
    if [ "$CURRENT" = "blue" ]; then
      TARGET="green"
    else
      TARGET="blue"
    fi
    
    # Deploy ไปยัง target slot
    aws ecs update-service \
      --cluster my-cluster \
      --service my-service-${TARGET} \
      --task-definition my-task:latest
    
    # Switch ALB target group
    aws elbv2 modify-listener \
      --listener-arn $LISTENER_ARN \
      --default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN}
```

### Canary Deployment

```
Canary: ส่ง traffic บางส่วนไปยัง version ใหม่

t=0: Users → v1.0 (100%)
t=1: Users → v1.0 (90%) + v2.0 (10%)  ← ทดสอบกับ users จริง
     Monitor metrics (error rate, latency)
t=2: Users → v1.0 (50%) + v2.0 (50%)  ← ขยายถ้า OK
t=3: Users → v2.0 (100%)              ← full rollout

ข้อดี: risk ต่ำสุด, ทดสอบกับ users จริง
ข้อเสีย: ซับซ้อนกว่า, ต้องการ feature flags
```

---

## Step 1529: Monitoring and Notifications

```yaml
# .github/workflows/deploy-with-monitoring.yml
jobs:
  deploy:
    steps:
      - name: Deploy
        id: deploy
        run: ./deploy.sh
      
      # Notify Slack on success
      - name: Notify success
        if: success()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "✅ Deployment successful",
              "attachments": [{
                "color": "good",
                "fields": [
                  {"title": "Branch", "value": "${{ github.ref_name }}", "short": true},
                  {"title": "Commit", "value": "${{ github.sha }}", "short": true},
                  {"title": "Author", "value": "${{ github.actor }}", "short": true}
                ]
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      
      # Notify Slack on failure
      - name: Notify failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "❌ Deployment FAILED! @channel",
              "attachments": [{
                "color": "danger",
                "text": "Check: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## Step 1530: Security Scanning

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # ทุกวันจันทร์

jobs:
  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # ตรวจสอบ vulnerable dependencies
      - name: Run npm audit
        run: npm audit --audit-level=high
      
      # Snyk scan
      - name: Run Snyk
        uses: snyk/actions/node@master
        with:
          args: --severity-threshold=high
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  
  code-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # GitHub CodeQL
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: javascript, typescript
      
      - name: Autobuild
        uses: github/codeql-action/autobuild@v2
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
  
  container-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build image
        run: docker build -t my-app:latest .
      
      # Trivy scan
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'my-app:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Basic CI Pipeline

สร้าง `.github/workflows/ci.yml` ที่:
1. รัน tests ทุกครั้งที่ push
2. รัน linter
3. Build โปรเจค
4. Report ผลลัพธ์

### แบบฝึกหัดที่ 2: Deploy Pipeline

เพิ่ม deployment workflow ที่:
1. Deploy ไปยัง staging เมื่อ push ไปยัง `develop`
2. Deploy ไปยัง production เมื่อ push ไปยัง `main`
3. Notify Slack ทุกครั้ง

### แบบฝึกหัดที่ 3: Branch Protection

1. ตั้งค่า branch protection rules สำหรับ `main`
2. ต้องการ CI pass ก่อน merge
3. ต้องการ code review 1 คน
4. ทดสอบด้วยการสร้าง PR

---

## สรุป Part 77

1. **CI/CD** = automation ตั้งแต่ commit ถึง production
2. **GitHub Actions**: workflows, triggers, jobs, steps
3. **Secrets**: จัดการ sensitive data อย่างปลอดภัย
4. **Caching**: ลด build time ด้วย dependency caching
5. **Deployment strategies**: rolling, blue-green, canary
6. **Security scanning**: ตรวจสอบ vulnerabilities อัตโนมัติ
