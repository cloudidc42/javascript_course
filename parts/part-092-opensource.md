# Part 92: Open Source Contribution (Steps 1811-1830)

## การมีส่วนร่วมกับ Open Source - จากผู้ใช้สู่ผู้สร้าง

ในส่วนนี้เราจะเรียนรู้วิธีการมีส่วนร่วมกับ open source projects อย่างมืออาชีพ ตั้งแต่การหา project ที่เหมาะสม การ set up local environment จนถึงการสร้างและเผยแพร่ npm package ของตัวเอง

---

## Step 1811: Finding Projects to Contribute To

```bash
# วิธีหา open source projects ที่เหมาะสม

# 1. GitHub Explore
# https://github.com/explore

# 2. Good First Issues
# https://goodfirstissues.com
# https://goodfirstissue.dev

# 3. Up For Grabs
# https://up-for-grabs.net

# 4. CodeTriage
# https://www.codetriage.com

# 5. ค้นหาใน GitHub
# github.com/search?q=label%3A"good+first+issue"+language%3AJavaScript&type=issues

# ค้นหาด้วย GitHub CLI
gh issue list --repo expressjs/express --label "good first issue" --state open

# ตัวอย่าง projects ที่เหมาะสำหรับ beginners
# - expressjs/express
# - axios/axios
# - prettier/prettier
# - eslint/eslint
# - jest-community/jest
# - webpack/webpack
# - vitejs/vite
```

```javascript
// เกณฑ์การเลือก project
const projectSelectionCriteria = {
  // ความสดใหม่ของ activity
  lastCommit: 'ภายใน 3 เดือน',
  
  // มี documentation ชัดเจน
  docs: [
    'CONTRIBUTING.md',
    'CODE_OF_CONDUCT.md',
    'README.md',
    'CHANGELOG.md',
  ],
  
  // ขนาด codebase ที่เหมาะสม
  codebaseSize: 'ไม่ใหญ่เกินไปสำหรับผู้เริ่มต้น',
  
  // community ที่ friendly
  community: [
    'ตอบ issues เร็ว',
    'Review PR ไม่นานเกินไป',
    'ใช้ภาษาสุภาพ',
  ],
  
  // issues ที่เหมาะสม
  issues: [
    '"good first issue" label',
    '"help wanted" label',
    '"beginner friendly" label',
  ],
};

// สิ่งที่ควรตรวจสอบก่อน contribute
async function evaluateProject(repoUrl) {
  // ตรวจสอบ
  const checks = {
    hasReadme: false,
    hasContributing: false,
    hasTests: false,
    recentActivity: false,
    openIssues: 0,
    prMergeTime: 'unknown',
  };
  
  // ใช้ GitHub API
  const response = await fetch(`https://api.github.com/repos/${repoUrl}`);
  const data = await response.json();
  
  checks.recentActivity = new Date(data.pushed_at) > 
    new Date(Date.now() - 90 * 24 * 60 * 60 * 1000);
  checks.openIssues = data.open_issues_count;
  
  return checks;
}
```

---

## Step 1812: Reading Project Documentation

```bash
# อ่าน documentation สำคัญก่อน contribute

# 1. README.md - ภาพรวมโปรเจค
# 2. CONTRIBUTING.md - วิธีการ contribute
# 3. CODE_OF_CONDUCT.md - กฎการ community
# 4. CHANGELOG.md - ประวัติการเปลี่ยนแปลง
# 5. Architecture docs (ถ้ามี)

# ตัวอย่าง CONTRIBUTING.md ของ project จริง
# cat CONTRIBUTING.md
```

```markdown
<!-- ตัวอย่าง CONTRIBUTING.md ที่ดี -->

# Contributing to MyProject

## Code of Conduct
กรุณาอ่าน [Code of Conduct](./CODE_OF_CONDUCT.md) ก่อน

## Getting Started

### Prerequisites
- Node.js >= 16
- npm >= 8

### Setup
\`\`\`bash
git clone https://github.com/username/myproject
cd myproject
npm install
npm test
\`\`\`

## How to Contribute

### Reporting Bugs
1. ตรวจสอบว่ามีรายงานแล้วหรือยัง
2. สร้าง issue ใหม่พร้อม:
   - Version ที่ใช้
   - Steps to reproduce
   - Expected vs actual behavior
   - Code sample

### Suggesting Features
- เปิด Discussion ก่อน
- อธิบาย use case ให้ชัดเจน
- ตรวจสอบว่าสอดคล้องกับ project scope

### Pull Requests
1. Fork repository
2. สร้าง feature branch: `git checkout -b feat/my-feature`
3. Commit ด้วย Conventional Commits
4. เขียน tests
5. Push และเปิด PR

## Commit Message Format
เราใช้ [Conventional Commits](https://conventionalcommits.org)

\`\`\`
type(scope): description

[optional body]

[optional footer]
\`\`\`

Types: feat, fix, docs, style, refactor, perf, test, chore
```

---

## Step 1813: Setting Up Project Locally

```bash
# ขั้นตอนการ set up project

# 1. Fork repo ใน GitHub
# คลิก Fork button บน repo page

# 2. Clone fork ของเรา
git clone https://github.com/YOUR_USERNAME/PROJECT_NAME.git
cd PROJECT_NAME

# 3. เพิ่ม upstream remote
git remote add upstream https://github.com/ORIGINAL_OWNER/PROJECT_NAME.git

# ตรวจสอบ remotes
git remote -v
# origin    https://github.com/YOUR_USERNAME/PROJECT_NAME.git (fetch)
# origin    https://github.com/YOUR_USERNAME/PROJECT_NAME.git (push)
# upstream  https://github.com/ORIGINAL_OWNER/PROJECT_NAME.git (fetch)
# upstream  https://github.com/ORIGINAL_OWNER/PROJECT_NAME.git (push)

# 4. Install dependencies
npm install
# หรือ
yarn install
# หรือ
pnpm install

# 5. รัน tests เพื่อตรวจสอบว่า setup ถูกต้อง
npm test

# 6. รัน development server
npm run dev

# 7. ตรวจสอบ code style
npm run lint

# 8. Build project
npm run build
```

```javascript
// ตัวอย่าง package.json ของ open source project
{
  "name": "my-open-source-lib",
  "version": "1.0.0",
  "description": "A useful JavaScript library",
  "main": "dist/index.js",
  "module": "dist/index.esm.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "rollup -c",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src --ext .js,.ts",
    "lint:fix": "eslint src --ext .js,.ts --fix",
    "format": "prettier --write src",
    "prepare": "husky install",
    "release": "semantic-release"
  },
  "devDependencies": {
    "@commitlint/cli": "^17.0.0",
    "@commitlint/config-conventional": "^17.0.0",
    "eslint": "^8.0.0",
    "husky": "^8.0.0",
    "jest": "^29.0.0",
    "lint-staged": "^13.0.0",
    "prettier": "^2.0.0",
    "rollup": "^3.0.0",
    "semantic-release": "^20.0.0"
  }
}
```

---

## Step 1814: Understanding Contribution Guidelines

```bash
# อ่านและทำความเข้าใจ guidelines

# ตรวจสอบ .github/ directory
ls .github/
# ISSUE_TEMPLATE/
# PULL_REQUEST_TEMPLATE.md
# workflows/
# CONTRIBUTING.md

# Issue Templates
cat .github/ISSUE_TEMPLATE/bug_report.md
cat .github/ISSUE_TEMPLATE/feature_request.md

# PR Template
cat .github/PULL_REQUEST_TEMPLATE.md

# GitHub Actions
cat .github/workflows/ci.yml
```

```yaml
# ตัวอย่าง .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Test
        run: npm run test:coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        if: matrix.node-version == '18.x'

  build:
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18.x'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - name: Check bundle size
        run: npx bundlesize
```

---

## Step 1815: Git Workflow for Open Source

```bash
# Git Workflow ที่ดีสำหรับ Open Source

# 1. Sync กับ upstream ก่อนเริ่มทำงาน
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# 2. สร้าง branch ใหม่สำหรับ feature/fix
# Naming conventions:
# feat/add-dark-mode
# fix/button-click-bug
# docs/update-readme
# refactor/user-service
# test/add-auth-tests

git checkout -b feat/add-search-functionality

# 3. ทำงาน, commit บ่อยๆ
git add src/search.js
git commit -m "feat(search): add basic text search functionality"

git add tests/search.test.js
git commit -m "test(search): add unit tests for search module"

# 4. Sync กับ upstream ระหว่างทำงาน
git fetch upstream
git rebase upstream/main
# แก้ conflicts ถ้ามี

# 5. Push ไปยัง fork
git push origin feat/add-search-functionality

# 6. เปิด Pull Request ใน GitHub
```

```bash
# Interactive Rebase สำหรับ clean commit history
# ก่อน submit PR อาจต้องการ squash commits

git log --oneline
# abc123 test(search): fix test cases
# def456 feat(search): add pagination
# ghi789 feat(search): add basic search
# jkl012 feat(search): initial structure

# Squash 4 commits เป็น 1
git rebase -i HEAD~4

# Editor จะเปิด:
# pick jkl012 feat(search): initial structure
# pick ghi789 feat(search): add basic search
# pick def456 feat(search): add pagination
# pick abc123 test(search): fix test cases

# เปลี่ยนเป็น:
# pick jkl012 feat(search): initial structure
# squash ghi789 feat(search): add basic search
# squash def456 feat(search): add pagination
# squash abc123 test(search): fix test cases

# บันทึก, จากนั้นแก้ commit message
# ผลลัพธ์: commit เดียวที่ clean
```

---

## Step 1816: Writing Conventional Commits

```bash
# Conventional Commits Format
# <type>(<scope>): <description>
# 
# [optional body]
# 
# [optional footer(s)]

# Types:
# feat:     ฟีเจอร์ใหม่
# fix:      แก้ bug
# docs:     เปลี่ยน documentation เท่านั้น
# style:    เปลี่ยน formatting (ไม่กระทบ logic)
# refactor: เปลี่ยนโค้ดที่ไม่ใช่ bug fix หรือ feature
# perf:     ปรับปรุง performance
# test:     เพิ่มหรือแก้ tests
# build:    เปลี่ยน build system หรือ dependencies
# ci:       เปลี่ยน CI configuration
# chore:    งานอื่นๆ

# ตัวอย่าง commits ที่ดี
git commit -m "feat(auth): add Google OAuth integration

Implements OAuth 2.0 flow with Google provider.
Users can now sign in with their Google account.

Closes #123"

git commit -m "fix(cart): prevent negative quantity in cart items

When users decrease quantity below 1, 
the item is removed instead of showing negative.

Fixes #456"

git commit -m "docs(readme): add installation prerequisites section

Added Node.js version requirements and 
optional environment variables explanation."

git commit -m "perf(search): add database index for search queries

Added compound index on title and content fields
to improve search query performance by ~70%."

# Breaking Changes
git commit -m "feat(api)!: change user endpoint response format

BREAKING CHANGE: User object no longer includes 'password' field.
Clients should update to use the new format.

Migration guide: docs/migration/v2.md"
```

```javascript
// commitlint configuration
// .commitlintrc.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'style', 'refactor',
      'perf', 'test', 'build', 'ci', 'chore', 'revert',
    ]],
    'subject-max-length': [2, 'always', 72],
    'body-max-line-length': [2, 'always', 100],
  },
};

// husky setup
// .husky/commit-msg
// npx --no -- commitlint --edit ${1}

// ติดตั้ง
// npm install @commitlint/cli @commitlint/config-conventional husky --save-dev
// npx husky install
// npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

---

## Step 1817: Writing Good PR Descriptions

```markdown
<!-- ตัวอย่าง PR Description ที่ดี -->

## Description

เพิ่มฟีเจอร์ค้นหาแบบ full-text ให้กับ product catalog

### Changes Made
- เพิ่ม `SearchService` สำหรับ full-text search logic
- เพิ่ม database indexes สำหรับ title และ description
- เพิ่ม pagination support ใน search results
- เพิ่ม search API endpoint `GET /api/products/search`

### Type of Change
- [x] New feature (non-breaking change adds functionality)
- [ ] Bug fix (non-breaking change fixes an issue)
- [ ] Breaking change
- [ ] Documentation update

### How Has This Been Tested?
- [x] Unit tests (SearchService)
- [x] Integration tests (API endpoint)
- [ ] E2E tests

### Test Results
\`\`\`
PASS tests/search.test.js
  SearchService
    ✓ returns results for matching query (45ms)
    ✓ returns empty array when no results (12ms)
    ✓ handles pagination correctly (23ms)
    ✓ sanitizes search query (8ms)

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
\`\`\`

### Screenshots (if UI change)
<!-- เพิ่ม screenshots ถ้ามี -->

### Checklist
- [x] My code follows the project's code style
- [x] I've added/updated tests
- [x] I've updated documentation
- [x] All existing tests pass
- [ ] I've added entries to CHANGELOG.md (maintainer does this)

### Related Issues
Closes #123
Related to #124

### Notes for Reviewers
- `SearchService.ts` line 45: ใช้ debounce เพื่อลด API calls
- Performance benchmark: 65ms average response time (tested with 10k products)
```

---

## Step 1818: Code Review Process

```javascript
// การรับ code review อย่างมืออาชีพ

// ✅ การตอบสนองที่ดีต่อ review comments

// Reviewer comment: "This function is doing too much"
// ❌ ตอบสนองที่ไม่ดี
// "มันทำงานได้ดีอยู่แล้ว ไม่จำเป็นต้องเปลี่ยน"

// ✅ ตอบสนองที่ดี
// "ขอบคุณสำหรับ feedback ครับ ผมเห็นด้วย
//  ผมจะแยก function ออกเป็น 3 ส่วน:
//  1. validateInput
//  2. processData
//  3. saveResults
//  ผมจะ push update ให้เร็วๆ นี้ครับ"

// การ request changes
// ถ้า reviewer request changes:
// 1. อ่านอย่างละเอียด
// 2. ถามถ้าไม่เข้าใจ
// 3. แก้ไขแล้ว comment ตอบกลับ
// 4. Push commits ใหม่

// GitHub comment commands
// @reviewer ผมได้แก้ไขตาม comment ของคุณแล้วครับ
// ดูที่ commit abc123 ครับ

// Resolving conversations
// เมื่อแก้ไขแล้ว ให้ mark conversation ว่า resolved

// การทำ code review ให้คนอื่น
function codeReviewTips() {
  return {
    beKind: 'ใช้ภาษาสุภาพและสร้างสรรค์',
    beSpecific: 'ระบุปัญหาให้ชัดเจน อย่า vague',
    suggestSolutions: 'เสนอวิธีแก้ไขด้วย ไม่ใช่แค่วิจารณ์',
    distinguishSeverity: 'แยก blocking issue กับ nitpick ให้ชัด',
    praise: 'ชมเมื่อเห็นโค้ดที่ดี',
    
    // ตัวอย่าง comment ที่ดี
    goodComments: [
      'นิดหน่อย: ตัวแปรนี้ชื่ออาจจะ confusing เล็กน้อย `userCount` อาจชัดกว่า `n`',
      'Blocking: มี SQL injection ที่บรรทัดนี้ ควรใช้ parameterized query',
      'Question: เราต้องการ deep copy หรือ shallow copy ตรงนี้ครับ?',
      'Suggestion: อาจใช้ Array.from() แทน [...spread] เพื่อ clarity',
      'Nice: ชอบที่แยก validation ออกมา clean มากเลย!',
    ],
  };
}
```

---

## Step 1819: Creating Your Own npm Package

```bash
# ขั้นตอนการสร้าง npm package

# 1. สร้างโฟลเดอร์
mkdir my-awesome-package
cd my-awesome-package

# 2. Initialize
npm init -y
# หรือ interactive mode
npm init

# 3. โครงสร้างที่แนะนำ
# my-awesome-package/
#   src/
#     index.js          <- main source
#     utils.js
#   tests/
#     index.test.js
#   dist/               <- compiled output (gitignore)
#   .gitignore
#   .npmignore
#   package.json
#   README.md
#   CHANGELOG.md
#   LICENSE

# 4. ติดตั้ง dev dependencies
npm install --save-dev jest eslint prettier rollup
```

```javascript
// package.json สำหรับ npm package ที่ดี
{
  "name": "@username/my-awesome-package",
  "version": "1.0.0",
  "description": "A helpful utility package for JavaScript developers",
  "keywords": ["javascript", "utility", "helper"],
  "author": "Your Name <your.email@example.com>",
  "license": "MIT",
  "homepage": "https://github.com/username/my-awesome-package#readme",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/username/my-awesome-package.git"
  },
  "bugs": {
    "url": "https://github.com/username/my-awesome-package/issues"
  },
  
  // Entry points
  "main": "dist/index.cjs.js",      // CommonJS (require)
  "module": "dist/index.esm.js",    // ES Modules (import)
  "types": "dist/index.d.ts",       // TypeScript types
  "exports": {
    ".": {
      "require": "./dist/index.cjs.js",
      "import": "./dist/index.esm.js",
      "types": "./dist/index.d.ts"
    },
    "./utils": {
      "require": "./dist/utils.cjs.js",
      "import": "./dist/utils.esm.js"
    }
  },
  
  // Files to publish
  "files": [
    "dist",
    "README.md",
    "CHANGELOG.md"
  ],
  
  "engines": {
    "node": ">=16.0.0"
  },
  
  "scripts": {
    "build": "rollup -c rollup.config.js",
    "test": "jest",
    "test:coverage": "jest --coverage",
    "lint": "eslint src",
    "format": "prettier --write src",
    "prepublishOnly": "npm run build && npm test",
    "prepare": "husky install"
  },
  
  "devDependencies": {
    "jest": "^29.0.0",
    "rollup": "^3.0.0",
    "eslint": "^8.0.0"
  },
  
  "peerDependencies": {
    // dependencies ที่ user ต้องมีเอง
  }
}
```

---

## Step 1820: Package Structure and Code

```javascript
// src/index.js - Main entry point

/**
 * @module my-awesome-package
 * @description A collection of useful JavaScript utilities
 */

// Re-export everything
export * from './string-utils.js';
export * from './array-utils.js';
export * from './object-utils.js';
export * from './async-utils.js';

// Named exports
export { default as deepClone } from './deep-clone.js';
export { default as debounce } from './debounce.js';
export { default as throttle } from './throttle.js';
```

```javascript
// src/string-utils.js

/**
 * แปลง string เป็น camelCase
 * @param {string} str - Input string
 * @returns {string} camelCase string
 * @example
 * toCamelCase('hello world') // 'helloWorld'
 * toCamelCase('foo-bar-baz') // 'fooBarBaz'
 * toCamelCase('MY_CONSTANT') // 'mYCONSTANT'
 */
export function toCamelCase(str) {
  if (typeof str !== 'string') {
    throw new TypeError(`Expected string, got ${typeof str}`);
  }
  return str
    .replace(/[-_\s]+(.)/g, (_, char) => char.toUpperCase())
    .replace(/^(.)/, char => char.toLowerCase());
}

/**
 * แปลง string เป็น snake_case
 * @param {string} str - Input string
 * @returns {string} snake_case string
 */
export function toSnakeCase(str) {
  if (typeof str !== 'string') {
    throw new TypeError(`Expected string, got ${typeof str}`);
  }
  return str
    .replace(/([A-Z])/g, '_$1')
    .replace(/[-\s]+/g, '_')
    .toLowerCase()
    .replace(/^_/, '');
}

/**
 * แปลง string เป็น kebab-case
 */
export function toKebabCase(str) {
  if (typeof str !== 'string') {
    throw new TypeError(`Expected string, got ${typeof str}`);
  }
  return str
    .replace(/([A-Z])/g, '-$1')
    .replace(/[-_\s]+/g, '-')
    .toLowerCase()
    .replace(/^-/, '');
}

/**
 * ตัด string ให้เหลือความยาวที่กำหนดพร้อม ellipsis
 * @param {string} str - Input string
 * @param {number} maxLength - Maximum length
 * @param {string} [ellipsis='...'] - Ellipsis string
 * @returns {string} Truncated string
 */
export function truncate(str, maxLength, ellipsis = '...') {
  if (typeof str !== 'string') {
    throw new TypeError(`Expected string, got ${typeof str}`);
  }
  if (str.length <= maxLength) return str;
  return str.slice(0, maxLength - ellipsis.length) + ellipsis;
}

/**
 * สร้าง slug จาก string
 */
export function toSlug(str) {
  if (typeof str !== 'string') {
    throw new TypeError(`Expected string, got ${typeof str}`);
  }
  return str
    .toLowerCase()
    .trim()
    .replace(/[^\w\s-]/g, '')
    .replace(/[\s_-]+/g, '-')
    .replace(/^-+|-+$/g, '');
}

/**
 * Count words in string
 */
export function wordCount(str) {
  if (typeof str !== 'string') return 0;
  return str.trim().split(/\s+/).filter(Boolean).length;
}

/**
 * Capitalize first letter of each word
 */
export function titleCase(str) {
  if (typeof str !== 'string') {
    throw new TypeError(`Expected string, got ${typeof str}`);
  }
  return str.replace(/\b\w/g, char => char.toUpperCase());
}

/**
 * Template literal tag สำหรับ HTML escaping
 */
export function html(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i - 1];
    const escaped = String(value)
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#39;');
    return result + escaped + str;
  });
}
```

```javascript
// src/array-utils.js

/**
 * แบ่ง array เป็น chunks ขนาดที่กำหนด
 * @param {Array} arr - Input array
 * @param {number} size - Chunk size
 * @returns {Array[]} Array of chunks
 */
export function chunk(arr, size) {
  if (!Array.isArray(arr)) throw new TypeError('Expected array');
  if (size < 1) throw new RangeError('Chunk size must be >= 1');
  
  const chunks = [];
  for (let i = 0; i < arr.length; i += size) {
    chunks.push(arr.slice(i, i + size));
  }
  return chunks;
}

/**
 * ลบ duplicates ออกจาก array
 */
export function unique(arr) {
  if (!Array.isArray(arr)) throw new TypeError('Expected array');
  return [...new Set(arr)];
}

/**
 * ลบ duplicates โดยใช้ key function
 */
export function uniqueBy(arr, keyFn) {
  if (!Array.isArray(arr)) throw new TypeError('Expected array');
  const seen = new Set();
  return arr.filter(item => {
    const key = keyFn(item);
    if (seen.has(key)) return false;
    seen.add(key);
    return true;
  });
}

/**
 * จัดกลุ่ม array ตาม key function
 */
export function groupBy(arr, keyFn) {
  if (!Array.isArray(arr)) throw new TypeError('Expected array');
  return arr.reduce((groups, item) => {
    const key = keyFn(item);
    (groups[key] = groups[key] || []).push(item);
    return groups;
  }, {});
}

/**
 * Flatten array หลายระดับ
 */
export function flattenDeep(arr) {
  if (!Array.isArray(arr)) throw new TypeError('Expected array');
  return arr.reduce((flat, item) =>
    flat.concat(Array.isArray(item) ? flattenDeep(item) : item), []);
}

/**
 * สุ่ม array
 */
export function shuffle(arr) {
  if (!Array.isArray(arr)) throw new TypeError('Expected array');
  const result = [...arr];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
}

/**
 * สุ่มเลือก n items จาก array
 */
export function sample(arr, n = 1) {
  if (!Array.isArray(arr)) throw new TypeError('Expected array');
  return shuffle(arr).slice(0, n);
}

/**
 * หา intersection ของ arrays
 */
export function intersection(...arrays) {
  if (!arrays.every(Array.isArray)) throw new TypeError('Expected arrays');
  return arrays.reduce((a, b) => a.filter(x => b.includes(x)));
}

/**
 * หา difference ระหว่าง arrays
 */
export function difference(arr, ...others) {
  if (!Array.isArray(arr)) throw new TypeError('Expected array');
  const otherSet = new Set(others.flat());
  return arr.filter(x => !otherSet.has(x));
}

/**
 * Zip arrays เข้าด้วยกัน
 */
export function zip(...arrays) {
  const maxLength = Math.max(...arrays.map(a => a.length));
  return Array.from({ length: maxLength }, (_, i) =>
    arrays.map(arr => arr[i])
  );
}
```

```javascript
// src/async-utils.js

/**
 * Sleep ตามเวลาที่กำหนด
 * @param {number} ms - Milliseconds to sleep
 * @returns {Promise<void>}
 */
export function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

/**
 * Retry function พร้อม exponential backoff
 * @param {Function} fn - Async function to retry
 * @param {Object} options - Options
 */
export async function retry(fn, {
  retries = 3,
  delay = 1000,
  factor = 2,
  maxDelay = 30000,
  onRetry = null,
} = {}) {
  let lastError;
  
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      return await fn(attempt);
    } catch (error) {
      lastError = error;
      
      if (attempt < retries - 1) {
        const waitTime = Math.min(delay * Math.pow(factor, attempt), maxDelay);
        
        if (onRetry) {
          onRetry(error, attempt + 1, waitTime);
        }
        
        await sleep(waitTime);
      }
    }
  }
  
  throw lastError;
}

/**
 * Timeout wrapper สำหรับ async function
 */
export function withTimeout(fn, timeoutMs) {
  return function(...args) {
    return Promise.race([
      fn.apply(this, args),
      new Promise((_, reject) =>
        setTimeout(() => reject(new Error(`Timeout after ${timeoutMs}ms`)), timeoutMs)
      ),
    ]);
  };
}

/**
 * Debounce async function
 */
export function debounceAsync(fn, delay) {
  let timeout;
  let resolvers = [];
  
  return function(...args) {
    return new Promise((resolve, reject) => {
      resolvers.push({ resolve, reject });
      
      clearTimeout(timeout);
      timeout = setTimeout(async () => {
        const currentResolvers = resolvers;
        resolvers = [];
        
        try {
          const result = await fn.apply(this, args);
          currentResolvers.forEach(({ resolve }) => resolve(result));
        } catch (error) {
          currentResolvers.forEach(({ reject }) => reject(error));
        }
      }, delay);
    });
  };
}

/**
 * Run promises concurrently with limit
 */
export async function mapConcurrent(items, fn, concurrency = 5) {
  const results = [];
  const executing = [];
  
  for (const item of items) {
    const promise = fn(item).then(result => {
      results.push(result);
      executing.splice(executing.indexOf(promise), 1);
    });
    
    executing.push(promise);
    
    if (executing.length >= concurrency) {
      await Promise.race(executing);
    }
  }
  
  await Promise.all(executing);
  return results;
}

/**
 * Memoize async function
 */
export function memoizeAsync(fn, { ttl = Infinity } = {}) {
  const cache = new Map();
  
  return async function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      const { value, timestamp } = cache.get(key);
      if (Date.now() - timestamp < ttl) {
        return value;
      }
    }
    
    const value = await fn.apply(this, args);
    cache.set(key, { value, timestamp: Date.now() });
    return value;
  };
}
```

---

## Step 1821: Testing Your Package

```javascript
// tests/string-utils.test.js
const { toCamelCase, toSnakeCase, toKebabCase, truncate, toSlug } = require('../src/string-utils');

describe('String Utilities', () => {
  describe('toCamelCase', () => {
    test.each([
      ['hello world', 'helloWorld'],
      ['foo-bar-baz', 'fooBarBaz'],
      ['foo_bar_baz', 'fooBarBaz'],
      ['FOO BAR', 'fOOBAR'],
      ['', ''],
    ])('toCamelCase(%s) => %s', (input, expected) => {
      expect(toCamelCase(input)).toBe(expected);
    });

    test('throws TypeError for non-string input', () => {
      expect(() => toCamelCase(123)).toThrow(TypeError);
      expect(() => toCamelCase(null)).toThrow(TypeError);
      expect(() => toCamelCase(undefined)).toThrow(TypeError);
    });
  });

  describe('truncate', () => {
    test('ตัด string ที่ยาวเกิน', () => {
      expect(truncate('Hello World', 8)).toBe('Hello...');
    });

    test('ไม่ตัด string ที่สั้นกว่า maxLength', () => {
      expect(truncate('Hi', 10)).toBe('Hi');
    });

    test('ใช้ custom ellipsis ได้', () => {
      expect(truncate('Hello World', 8, '…')).toBe('Hello W…');
    });
  });

  describe('toSlug', () => {
    test.each([
      ['Hello World', 'hello-world'],
      ['  Trim Spaces  ', 'trim-spaces'],
      ['Special @#$% Chars', 'special-chars'],
      ['Multiple   Spaces', 'multiple-spaces'],
    ])('toSlug(%s) => %s', (input, expected) => {
      expect(toSlug(input)).toBe(expected);
    });
  });
});

// tests/array-utils.test.js
const { chunk, unique, groupBy, flatten, shuffle } = require('../src/array-utils');

describe('Array Utilities', () => {
  describe('chunk', () => {
    test('แบ่ง array เท่าๆ กัน', () => {
      expect(chunk([1, 2, 3, 4], 2)).toEqual([[1, 2], [3, 4]]);
    });

    test('chunk สุดท้ายอาจเล็กกว่า', () => {
      expect(chunk([1, 2, 3, 4, 5], 2)).toEqual([[1, 2], [3, 4], [5]]);
    });

    test('chunk size = 1', () => {
      expect(chunk([1, 2, 3], 1)).toEqual([[1], [2], [3]]);
    });

    test('throws for invalid size', () => {
      expect(() => chunk([1, 2, 3], 0)).toThrow(RangeError);
    });
  });

  describe('groupBy', () => {
    test('จัดกลุ่มตาม property', () => {
      const people = [
        { name: 'Alice', age: 25 },
        { name: 'Bob', age: 30 },
        { name: 'Charlie', age: 25 },
      ];
      
      const grouped = groupBy(people, p => p.age);
      expect(grouped[25]).toHaveLength(2);
      expect(grouped[30]).toHaveLength(1);
    });
  });
});

// tests/async-utils.test.js
const { retry, sleep, mapConcurrent } = require('../src/async-utils');

describe('Async Utilities', () => {
  describe('retry', () => {
    test('retry สำเร็จหลังจาก fail ครั้งแรก', async () => {
      let attempts = 0;
      const fn = jest.fn().mockImplementation(() => {
        attempts++;
        if (attempts < 2) throw new Error('fail');
        return 'success';
      });

      const result = await retry(fn, { retries: 3, delay: 10 });
      expect(result).toBe('success');
      expect(fn).toHaveBeenCalledTimes(2);
    });

    test('throw error หลัง retry หมด', async () => {
      const fn = jest.fn().mockRejectedValue(new Error('always fails'));
      await expect(retry(fn, { retries: 3, delay: 10 })).rejects.toThrow('always fails');
      expect(fn).toHaveBeenCalledTimes(3);
    });
  });

  describe('mapConcurrent', () => {
    test('รัน tasks พร้อมกันตาม concurrency limit', async () => {
      const concurrent = [];
      let maxConcurrent = 0;

      const tasks = Array.from({ length: 10 }, (_, i) => i);
      
      await mapConcurrent(tasks, async (item) => {
        concurrent.push(item);
        maxConcurrent = Math.max(maxConcurrent, concurrent.length);
        await sleep(10);
        concurrent.pop();
        return item * 2;
      }, 3);

      expect(maxConcurrent).toBeLessThanOrEqual(3);
    });
  });
});
```

---

## Step 1822: Building with Rollup

```javascript
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import { babel } from '@rollup/plugin-babel';
import terser from '@rollup/plugin-terser';
import pkg from './package.json' assert { type: 'json' };

const banner = `/**
 * ${pkg.name} v${pkg.version}
 * ${pkg.description}
 * ${pkg.homepage}
 * 
 * Copyright (c) ${new Date().getFullYear()} ${pkg.author}
 * Released under the ${pkg.license} License
 */`;

export default [
  // UMD build (สำหรับ browser และ Node.js)
  {
    input: 'src/index.js',
    output: {
      name: 'MyAwesomePackage',
      file: pkg.unpkg || 'dist/index.umd.js',
      format: 'umd',
      banner,
      sourcemap: true,
    },
    plugins: [
      resolve(),
      commonjs(),
      babel({ babelHelpers: 'bundled' }),
      terser(),
    ],
  },
  
  // ESM build (สำหรับ modern bundlers)
  {
    input: 'src/index.js',
    output: [
      {
        file: pkg.module,
        format: 'esm',
        banner,
        sourcemap: true,
      },
      {
        file: pkg.main,
        format: 'cjs',
        banner,
        exports: 'named',
        sourcemap: true,
      },
    ],
    plugins: [
      resolve(),
      commonjs(),
      babel({ babelHelpers: 'bundled' }),
    ],
  },
];
```

---

## Step 1823: Publishing to npm

```bash
# ขั้นตอนการ publish npm package

# 1. สร้าง account ที่ npmjs.com
# 2. Login ใน terminal
npm login
# หรือ
npm login --scope=@username

# 3. ตรวจสอบ package ก่อน publish
npm pack --dry-run
# จะแสดง files ที่จะถูก include

# 4. ตรวจสอบ bundle size
npx bundlesize

# 5. Build
npm run build

# 6. Test ครั้งสุดท้าย
npm test

# 7. Publish
npm publish
# หรือ scoped package
npm publish --access public

# 8. ตรวจสอบว่า publish สำเร็จ
npm info @username/my-awesome-package

# publish beta version
npm publish --tag beta

# ใช้ beta version
npm install @username/my-awesome-package@beta
```

```bash
# .npmignore - ระบุ files ที่ไม่ต้องการใน npm package
# (ถ้าไม่มี .npmignore จะใช้ .gitignore แทน)

# Test files
tests/
coverage/
*.test.js
*.spec.js

# Development configs
.eslintrc*
.prettierrc*
.babelrc*
rollup.config.js
jest.config.js

# Dev files
src/
.husky/
.github/

# Local files
.env
.env.local
*.log
```

---

## Step 1824: Semantic Versioning

```javascript
// Semantic Versioning (SemVer): MAJOR.MINOR.PATCH

// MAJOR: Breaking changes (1.0.0 -> 2.0.0)
// MINOR: New features, backward compatible (1.0.0 -> 1.1.0)
// PATCH: Bug fixes, backward compatible (1.0.0 -> 1.0.1)

// Pre-release versions
// 1.0.0-alpha.1
// 1.0.0-beta.1
// 1.0.0-rc.1

// bump version commands
// npm version patch    // 1.0.0 -> 1.0.1
// npm version minor    // 1.0.0 -> 1.1.0
// npm version major    // 1.0.0 -> 2.0.0
// npm version prerelease --preid=beta  // 1.0.0 -> 1.0.1-beta.0

// Automate with standard-version
// npm install --save-dev standard-version

// package.json scripts
{
  "scripts": {
    "release": "standard-version",
    "release:minor": "standard-version --release-as minor",
    "release:major": "standard-version --release-as major",
    "release:beta": "standard-version --prerelease beta"
  }
}

// standard-version อ่าน Conventional Commits และ:
// 1. Bumps version ตาม commit types
// 2. Updates CHANGELOG.md
// 3. Creates git tag

// Automate ด้วย semantic-release
// npm install --save-dev semantic-release @semantic-release/changelog @semantic-release/git

// .releaserc.js
module.exports = {
  branches: ['main'],
  plugins: [
    '@semantic-release/commit-analyzer',
    '@semantic-release/release-notes-generator',
    '@semantic-release/changelog',
    '@semantic-release/npm',
    '@semantic-release/github',
    [
      '@semantic-release/git',
      {
        assets: ['package.json', 'CHANGELOG.md'],
        message: 'chore(release): ${nextRelease.version} [skip ci]',
      },
    ],
  ],
};
```

---

## Step 1825: Writing a Good README

```markdown
<!-- README.md template สำหรับ npm package -->

# my-awesome-package

[![npm version](https://badge.fury.io/js/my-awesome-package.svg)](https://badge.fury.io/js/my-awesome-package)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Test Coverage](https://codecov.io/gh/username/my-awesome-package/badge.svg)](https://codecov.io/gh/username/my-awesome-package)

> ประโยค description สั้นๆ ว่า package ทำอะไร

## Features

- ✨ Feature 1
- 🚀 Feature 2
- 🔒 Feature 3

## Installation

\`\`\`bash
npm install my-awesome-package
# หรือ
yarn add my-awesome-package
# หรือ
pnpm add my-awesome-package
\`\`\`

## Quick Start

\`\`\`javascript
import { toCamelCase, chunk, retry } from 'my-awesome-package';

// String utilities
toCamelCase('hello world') // 'helloWorld'

// Array utilities
chunk([1, 2, 3, 4, 5], 2) // [[1,2], [3,4], [5]]

// Async utilities
await retry(fetchData, { retries: 3, delay: 1000 })
\`\`\`

## API Reference

### String Utilities

#### `toCamelCase(str)`
แปลง string เป็น camelCase

**Parameters:**
- `str` (string): Input string

**Returns:** string

**Example:**
\`\`\`javascript
toCamelCase('hello world')  // 'helloWorld'
toCamelCase('foo-bar')      // 'fooBar'
\`\`\`

### Array Utilities

#### `chunk(arr, size)`
แบ่ง array เป็น chunks

...

## Browser Support

| Chrome | Firefox | Safari | Edge |
|--------|---------|--------|------|
| Latest | Latest  | Latest | Latest |

## Contributing

กรุณาดู [CONTRIBUTING.md](./CONTRIBUTING.md)

## Changelog

ดู [CHANGELOG.md](./CHANGELOG.md)

## License

MIT © [Your Name](https://github.com/username)
```

---

## Step 1826: Changelog

```markdown
<!-- CHANGELOG.md -->
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [2.1.0] - 2024-03-15

### Added
- เพิ่ม `mapConcurrent` ใน async utilities
- เพิ่ม `memoizeAsync` function
- รองรับ TypeScript 5.x

### Changed
- ปรับปรุง performance ของ `groupBy` function
- อัพเดท peer dependencies

### Fixed
- แก้ bug ใน `uniqueBy` เมื่อใช้กับ null values

## [2.0.0] - 2024-01-01

### BREAKING CHANGES
- เปลี่ยน `chunk` signature จาก `chunk(size, arr)` เป็น `chunk(arr, size)`
- ลบ deprecated `flatten` ออก ใช้ `flattenDeep` แทน

### Added
- เพิ่ม `zip` function
- เพิ่ม `difference` function
- เพิ่ม TypeScript types

### Deprecated
- `flatten` deprecated, จะถูกลบใน v3.0.0

## [1.0.0] - 2023-06-01

### Added
- Initial release
- String utilities: toCamelCase, toSnakeCase, toKebabCase, truncate
- Array utilities: chunk, unique, groupBy, flatten
- Async utilities: sleep, retry, withTimeout

[Unreleased]: https://github.com/username/package/compare/v2.1.0...HEAD
[2.1.0]: https://github.com/username/package/compare/v2.0.0...v2.1.0
[2.0.0]: https://github.com/username/package/compare/v1.0.0...v2.0.0
[1.0.0]: https://github.com/username/package/releases/tag/v1.0.0
```

---

## Step 1827: Maintaining a Project

```javascript
// การดูแลรักษา open source project

// 1. Issue Templates
// .github/ISSUE_TEMPLATE/bug_report.yml
const bugReportTemplate = `
name: Bug Report
description: File a bug report
title: "[Bug]: "
labels: ["bug", "triage"]
assignees:
  - maintainer-username

body:
  - type: markdown
    attributes:
      value: |
        Thanks for taking the time to fill out this bug report!
  
  - type: input
    id: version
    attributes:
      label: Package Version
      placeholder: ex. 2.1.0
    validations:
      required: true
  
  - type: textarea
    id: description
    attributes:
      label: Describe the bug
      description: A clear description of what the bug is
    validations:
      required: true
  
  - type: textarea
    id: reproduce
    attributes:
      label: Steps to reproduce
      value: |
        1. 
        2. 
        3. 
    validations:
      required: true
  
  - type: textarea
    id: code
    attributes:
      label: Code Sample
      render: javascript
`;

// 2. Stale Bot Configuration
// .github/stale.yml
const staleConfig = `
daysUntilStale: 60
daysUntilClose: 7
staleLabel: stale
staleComment: >
  This issue has been automatically marked as stale because it has 
  not had recent activity. It will be closed if no further activity occurs.
closeComment: >
  This issue has been automatically closed due to inactivity.
exemptLabels:
  - pinned
  - security
  - "good first issue"
`;

// 3. Dependabot Configuration
// .github/dependabot.yml
const dependabotConfig = `
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    labels:
      - "dependencies"
    reviewers:
      - "maintainer-username"
    groups:
      dev-dependencies:
        dependency-type: "development"
      production-dependencies:
        dependency-type: "production"
`;

// 4. Security Policy
// SECURITY.md
const securityPolicy = `
# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| 2.x.x   | ✅        |
| 1.x.x   | ❌        |

## Reporting a Vulnerability

DO NOT create a public GitHub issue for security vulnerabilities.

Email us at security@example.com with:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

We will respond within 48 hours and aim to fix critical issues within 7 days.
`;
```

---

## Step 1828: Advanced GitHub Workflows

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
          persist-credentials: false
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
      
      - name: Semantic Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release

# .github/workflows/pr-check.yml
name: PR Checks

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  check:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Check commit messages
        uses: wagoid/commitlint-github-action@v5
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'
      
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
      
      - name: Check bundle size
        uses: andresz1/size-limit-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Step 1829: Publishing Scoped Packages

```bash
# Scoped packages (@username/package-name)

# สร้าง organization ใน npm (optional)
# npmjs.com/org/create

# Login
npm login

# สร้าง package
npm init --scope=@myorg

# package.json ที่ได้
# "name": "@myorg/my-package"

# Publish public scoped package
npm publish --access public

# Publish private scoped package (ต้องมี paid plan)
npm publish --access restricted

# ตรวจสอบ access
npm access ls-packages @myorg

# เปลี่ยน access
npm access public @myorg/my-package
npm access restricted @myorg/my-package

# Team management
npm access grant read-only @myorg:developers @myorg/my-package
npm access grant read-write @myorg:maintainers @myorg/my-package
```

```javascript
// การใช้ .npmrc สำหรับ scoped packages
// .npmrc
// @myorg:registry=https://registry.npmjs.org/
// //registry.npmjs.org/:_authToken=${NPM_TOKEN}

// สำหรับ private registry (เช่น GitHub Packages)
// .npmrc
// @myorg:registry=https://npm.pkg.github.com
// //npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}

// package.json
{
  "name": "@myorg/my-package",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```

---

## Step 1830: Best Practices Summary

```javascript
// สรุป Open Source Best Practices

const openSourceBestPractices = {
  
  // ก่อน contribute
  before: [
    'อ่าน CONTRIBUTING.md ก่อนเสมอ',
    'ตรวจสอบว่ามี issue หรือ PR ที่เหมือนกันหรือยัง',
    'เปิด issue ขอความเห็นก่อนเริ่มงานใหญ่',
    'Setup local environment ให้พร้อม',
  ],
  
  // ระหว่าง contribute
  during: [
    'ใช้ Conventional Commits',
    'เขียน tests สำหรับทุก change',
    'อัพเดท documentation',
    'Sync กับ upstream บ่อยๆ',
    'Commit บ่อยๆ แต่ squash ก่อน PR',
  ],
  
  // ใน PR
  pullRequest: [
    'อธิบายว่าทำอะไรและทำไม',
    'Reference related issues',
    'เพิ่ม screenshots ถ้ามี UI changes',
    'ตอบ review comments เร็วๆ',
    'ขอบคุณ reviewer',
  ],
  
  // การ maintain package ของตัวเอง
  maintaining: [
    'ตอบ issues ภายใน 48 ชั่วโมง',
    'เขียน comprehensive documentation',
    'อัพเดท dependencies เป็นประจำ',
    'ใช้ semantic versioning',
    'อัพเดท CHANGELOG.md',
    'มี security policy',
  ],
  
  // npm package quality
  packageQuality: [
    'README ที่ดีพร้อม examples',
    'TypeScript types',
    'Unit tests > 90% coverage',
    'CI/CD pipeline',
    'Automated releases',
    'Bundle size ที่เหมาะสม',
  ],
};

// Tools ที่ควรรู้จัก
const usefulTools = {
  versionManagement: ['standard-version', 'semantic-release', 'changesets'],
  codeQuality: ['eslint', 'prettier', 'husky', 'lint-staged', 'commitlint'],
  bundling: ['rollup', 'tsup', 'microbundle', 'esbuild'],
  testing: ['jest', 'vitest', 'mocha', 'c8'],
  documentation: ['jsdoc', 'typedoc', 'docusaurus'],
  sizeCheck: ['bundlesize', 'size-limit'],
};

console.log('Open Source Best Practices:', openSourceBestPractices);
console.log('Useful Tools:', usefulTools);
```

---

## แบบฝึกหัด (Exercises)

### Exercise 1: Contribute to Existing Project
1. หา open source project ที่ใช้ JavaScript
2. ดู "good first issue" labels
3. Fork, clone, setup
4. แก้ issue เล็กๆ เช่น bug fix หรือ doc improvement
5. เปิด PR

### Exercise 2: Create npm Package - Thai Date Utilities
สร้าง package `thai-date-utils` ที่มี:
- `toBuddhistYear(date)` - แปลงเป็น พ.ศ.
- `toThaiDate(date)` - แปลงเป็น วันที่ภาษาไทย
- `getDayInThai(date)` - ชื่อวันภาษาไทย
- `getMonthInThai(date)` - ชื่อเดือนภาษาไทย
- Tests ครบถ้วน
- README ภาษาไทย/อังกฤษ

### Exercise 3: Implement Complete Package Structure
สร้าง npm package พร้อม:
- Rollup build (CJS + ESM)
- TypeScript types (d.ts files)
- Jest tests + 90% coverage
- GitHub Actions CI
- Conventional commits + commitlint
- Auto-release ด้วย semantic-release

### Exercise 4: Improve Existing Package
เลือก npm package ที่คุณใช้บ่อยๆ:
1. Report a bug (สร้าง minimal reproduction)
2. Submit a PR พร้อม fix + test
3. หรือ improve documentation

### Exercise 5: Code Review Practice
1. หา PR ที่เปิดอยู่ใน open source project
2. Review code ให้ constructive feedback
3. เรียนรู้จากการ review ของคนอื่น

---

*จบ Part 92: Open Source Contribution*
*ต่อไป Part 93: Performance at Scale*
