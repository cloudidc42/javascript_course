# Part 66: Testing - E2E Tests (Steps 1291-1310)

## บทนำ: การทดสอบแบบ End-to-End

การทดสอบแบบ End-to-End (E2E) คือการทดสอบที่จำลองการใช้งานจริงของผู้ใช้ตั้งแต่ต้นจนจบ โดยทดสอบทั้งระบบรวมกัน ตั้งแต่ UI ไปจนถึง Database เหมือนกับที่ผู้ใช้จริงๆ ใช้งาน

```
Unit Tests → Integration Tests → E2E Tests
(เร็วที่สุด)                      (ช้าที่สุดแต่ครอบคลุมที่สุด)
```

---

## Step 1291: E2E Testing คืออะไร

### ประเภทของการทดสอบ

```
Testing Pyramid
      /\
     /E2E\          ← น้อย แต่ครอบคลุม
    /------\
   /  Integration  \ ← ปานกลาง
  /------------------\
 /    Unit Tests      \ ← มาก, เร็ว, แยกส่วน
/______________________\
```

### ทำไมต้องใช้ E2E Tests

```javascript
// Unit Test - ทดสอบฟังก์ชันแยก
function add(a, b) {
  return a + b;
}
test('add works', () => {
  expect(add(1, 2)).toBe(3);
});

// Integration Test - ทดสอบหลายส่วนรวมกัน
test('user service creates user', async () => {
  const user = await userService.create({ name: 'Alice' });
  expect(user.id).toBeDefined();
});

// E2E Test - ทดสอบเหมือนผู้ใช้จริง
test('user can register and login', async ({ page }) => {
  await page.goto('/register');
  await page.fill('#name', 'Alice');
  await page.fill('#email', 'alice@example.com');
  await page.fill('#password', 'SecurePass123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('h1')).toHaveText('ยินดีต้อนรับ Alice');
});
```

---

## Step 1292: Playwright คืออะไรและทำไมเลือกใช้

Playwright คือ framework สำหรับ E2E testing ที่พัฒนาโดย Microsoft รองรับหลาย browser

### เปรียบเทียบ E2E Frameworks

| Feature | Playwright | Cypress | Selenium |
|---------|-----------|---------|----------|
| Speed | เร็วมาก | เร็ว | ช้า |
| Multi-browser | ✅ | จำกัด | ✅ |
| Auto-wait | ✅ | ✅ | ❌ |
| Network intercept | ✅ | ✅ | จำกัด |
| Parallel tests | ✅ | Pro only | ✅ |
| TypeScript | ✅ | ✅ | ✅ |

---

## Step 1293: การติดตั้ง Playwright

### ติดตั้งและตั้งค่า

```bash
# สร้างโปรเจค Playwright ใหม่
npm init playwright@latest

# ตอบคำถาม:
# ? Where to put your end-to-end tests? › tests
# ? Add a GitHub Actions workflow? › false
# ? Install Playwright browsers? › true

# หรือติดตั้งในโปรเจคที่มีอยู่แล้ว
npm install -D @playwright/test
npx playwright install
```

### โครงสร้างโปรเจค

```
my-project/
├── tests/
│   ├── e2e/
│   │   ├── auth.spec.ts
│   │   ├── checkout.spec.ts
│   │   └── home.spec.ts
│   └── fixtures/
│       └── test-data.ts
├── playwright.config.ts
├── package.json
└── ...
```

---

## Step 1294: playwright.config.ts การตั้งค่า

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // ที่อยู่ไฟล์ test
  testDir: './tests',
  
  // รัน tests แบบ parallel
  fullyParallel: true,
  
  // หยุดทันทีถ้า test fail บน CI
  forbidOnly: !!process.env.CI,
  
  // จำนวนครั้งที่ retry เมื่อ fail
  retries: process.env.CI ? 2 : 0,
  
  // จำนวน workers (parallel)
  workers: process.env.CI ? 1 : undefined,
  
  // Reporter
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['json', { outputFile: 'test-results.json' }],
    ['list']
  ],
  
  // Global settings
  use: {
    // URL หลักสำหรับทดสอบ
    baseURL: 'http://localhost:3000',
    
    // เก็บ trace เมื่อ retry
    trace: 'on-first-retry',
    
    // เก็บ screenshot เมื่อ fail
    screenshot: 'only-on-failure',
    
    // เก็บ video เมื่อ fail  
    video: 'retain-on-failure',
    
    // Timeout สำหรับแต่ละ action
    actionTimeout: 10000,
    
    // Timeout สำหรับ navigation
    navigationTimeout: 30000,
  },
  
  // ทดสอบบน browsers ต่างๆ
  projects: [
    // Desktop browsers
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    
    // Mobile
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
    
    // Tablet
    {
      name: 'iPad',
      use: { ...devices['iPad Pro'] },
    },
  ],
  
  // เริ่ม dev server ก่อนรัน tests
  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

---

## Step 1295: การเขียน Test พื้นฐาน

```typescript
// tests/e2e/home.spec.ts
import { test, expect } from '@playwright/test';

// Test suite
test.describe('หน้าแรก', () => {
  
  // ทำก่อนทุก test
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });
  
  // Test 1
  test('แสดงชื่อเว็บไซต์', async ({ page }) => {
    await expect(page).toHaveTitle(/My App/);
  });
  
  // Test 2
  test('มีปุ่ม Login', async ({ page }) => {
    const loginButton = page.getByRole('button', { name: 'เข้าสู่ระบบ' });
    await expect(loginButton).toBeVisible();
  });
  
  // Test 3
  test('navigation links ทำงานได้', async ({ page }) => {
    await page.getByRole('link', { name: 'เกี่ยวกับเรา' }).click();
    await expect(page).toHaveURL('/about');
  });
  
});

// Test แบบ standalone
test('ผู้ใช้สามารถค้นหาสินค้าได้', async ({ page }) => {
  await page.goto('/');
  
  const searchBox = page.getByPlaceholder('ค้นหาสินค้า...');
  await searchBox.fill('laptop');
  await searchBox.press('Enter');
  
  await expect(page).toHaveURL('/search?q=laptop');
  await expect(page.getByRole('heading', { name: /ผลการค้นหา/ })).toBeVisible();
});
```

---

## Step 1296: Locators - วิธีหา Elements

Locators เป็นวิธีที่ Playwright ใช้ระบุ element บนหน้าเว็บ

### getByRole - แนะนำใช้มากที่สุด

```typescript
// tests/e2e/locators.spec.ts
import { test, expect } from '@playwright/test';

test('getByRole examples', async ({ page }) => {
  await page.goto('/');
  
  // Buttons
  const submitBtn = page.getByRole('button', { name: 'ส่งข้อมูล' });
  const cancelBtn = page.getByRole('button', { name: /ยกเลิก/i }); // case-insensitive
  
  // Links
  const homeLink = page.getByRole('link', { name: 'หน้าแรก' });
  
  // Input
  const emailInput = page.getByRole('textbox', { name: 'อีเมล' });
  const passwordInput = page.getByRole('textbox', { name: 'รหัสผ่าน' });
  
  // Checkbox
  const rememberMe = page.getByRole('checkbox', { name: 'จำฉันไว้' });
  
  // Radio
  const option1 = page.getByRole('radio', { name: 'ตัวเลือกที่ 1' });
  
  // Heading
  const title = page.getByRole('heading', { name: 'ยินดีต้อนรับ', level: 1 });
  
  // List items
  const listItems = page.getByRole('listitem');
  
  // Table
  const table = page.getByRole('table');
  const rows = page.getByRole('row');
  
  // Images
  const logo = page.getByRole('img', { name: 'โลโก้บริษัท' });
  
  // Combobox (select)
  const dropdown = page.getByRole('combobox', { name: 'เลือกหมวดหมู่' });
  
  // Dialog
  const modal = page.getByRole('dialog', { name: 'ยืนยันการลบ' });
  
  await expect(title).toBeVisible();
  await expect(submitBtn).toBeEnabled();
});
```

### getByText

```typescript
test('getByText examples', async ({ page }) => {
  await page.goto('/products');
  
  // หา element ที่มี text ตรงกัน
  const exactText = page.getByText('สินค้าทั้งหมด');
  
  // หาแบบ substring
  const partialText = page.getByText('สินค้า'); // จะ match หลาย elements
  
  // หาแบบ case-insensitive
  const caseInsensitive = page.getByText(/ราคา/i);
  
  // หาใน context
  const productCard = page.locator('.product-card').first();
  const priceInCard = productCard.getByText(/฿\d+/);
  
  await expect(exactText).toBeVisible();
  await expect(priceInCard).toHaveText(/฿1,299/);
});
```

### getByLabel, getByPlaceholder, getByTestId

```typescript
test('form locators', async ({ page }) => {
  await page.goto('/register');
  
  // getByLabel - หา input จาก label ที่เชื่อมกัน
  const nameInput = page.getByLabel('ชื่อ-นามสกุล');
  const emailInput = page.getByLabel('อีเมล');
  
  // getByPlaceholder - หา input จาก placeholder
  const searchInput = page.getByPlaceholder('ค้นหาสินค้า...');
  const messageArea = page.getByPlaceholder('พิมพ์ข้อความของคุณ');
  
  // getByTestId - หาจาก data-testid attribute (แนะนำสำหรับ test)
  // HTML: <button data-testid="submit-btn">ส่ง</button>
  const submitBtn = page.getByTestId('submit-btn');
  const userAvatar = page.getByTestId('user-avatar');
  const navMenu = page.getByTestId('nav-menu');
  
  // Custom test id attribute (ตั้งค่าใน playwright.config.ts)
  // testIdAttribute: 'data-cy'  → ใช้ data-cy="submit-btn"
  
  await nameInput.fill('สมชาย ใจดี');
  await emailInput.fill('somchai@example.com');
  await submitBtn.click();
});
```

### Locator Chaining และ Filtering

```typescript
test('chaining locators', async ({ page }) => {
  await page.goto('/products');
  
  // Chain locators
  const productList = page.locator('.product-list');
  const firstProduct = productList.locator('.product-card').first();
  const productTitle = firstProduct.locator('h3');
  
  // Filter
  const products = page.locator('.product-card');
  const laptops = products.filter({ hasText: 'laptop' });
  const saleProducts = products.filter({ has: page.locator('.sale-badge') });
  
  // nth
  const secondProduct = page.locator('.product-card').nth(1);
  
  // last
  const lastProduct = page.locator('.product-card').last();
  
  // Count
  const count = await products.count();
  console.log(`จำนวนสินค้า: ${count}`);
  
  // All (ดึงทุก locators)
  const allProducts = await products.all();
  for (const product of allProducts) {
    const title = await product.locator('h3').textContent();
    console.log(title);
  }
  
  await expect(laptops.first()).toBeVisible();
});
```

---

## Step 1297: Actions - การโต้ตอบกับ Elements

### click, fill, type, press

```typescript
test('basic actions', async ({ page }) => {
  await page.goto('/login');
  
  // fill - เคลียร์แล้วพิมพ์ (แนะนำ)
  await page.getByLabel('อีเมล').fill('user@example.com');
  
  // type - พิมพ์ทีละตัวอักษร (simulate typing)
  await page.getByLabel('รหัสผ่าน').type('MyPassword123', { delay: 100 });
  
  // press - กดปุ่ม
  await page.getByLabel('อีเมล').press('Tab');
  await page.keyboard.press('Enter');
  await page.keyboard.press('Control+A'); // Ctrl+A
  await page.keyboard.press('Meta+C');    // Cmd+C (Mac)
  
  // click
  await page.getByRole('button', { name: 'เข้าสู่ระบบ' }).click();
  
  // double click
  await page.locator('.text-area').dblclick();
  
  // right click
  await page.locator('.item').click({ button: 'right' });
  
  // click with modifier
  await page.locator('.link').click({ modifiers: ['Shift'] }); // Shift+Click
  
  // click at position
  await page.locator('.canvas').click({ position: { x: 100, y: 150 } });
  
  // hover
  await page.getByRole('button', { name: 'เมนู' }).hover();
  
  // focus
  await page.getByLabel('ชื่อ').focus();
  
  // blur
  await page.getByLabel('ชื่อ').blur();
  
  // clear
  await page.getByLabel('ชื่อ').clear();
  
  // select option
  await page.getByLabel('จังหวัด').selectOption('Bangkok');
  await page.getByLabel('จังหวัด').selectOption({ label: 'กรุงเทพมหานคร' });
  await page.getByLabel('สี').selectOption(['red', 'blue']); // multiple
  
  // check/uncheck checkbox
  await page.getByLabel('ยอมรับเงื่อนไข').check();
  await page.getByLabel('รับข่าวสาร').uncheck();
  
  // file upload
  await page.getByLabel('อัพโหลดรูปภาพ').setInputFiles('photo.jpg');
  await page.getByLabel('อัพโหลดไฟล์').setInputFiles(['file1.pdf', 'file2.pdf']);
  
  // drag and drop
  await page.locator('#source').dragTo(page.locator('#target'));
});
```

### Keyboard และ Mouse

```typescript
test('keyboard and mouse', async ({ page }) => {
  await page.goto('/editor');
  
  // Keyboard shortcuts
  await page.keyboard.down('Shift');
  await page.keyboard.press('ArrowRight');
  await page.keyboard.press('ArrowRight');
  await page.keyboard.up('Shift');
  
  // Type text
  await page.keyboard.type('สวัสดีโลก');
  
  // Insert text (ไม่ simulate key events)
  await page.locator('#editor').pressSequentially('Hello World');
  
  // Mouse
  await page.mouse.move(100, 100);
  await page.mouse.down();
  await page.mouse.move(200, 200);
  await page.mouse.up();
  
  // Scroll
  await page.mouse.wheel(0, 500); // scroll down 500px
  
  // Scroll to element
  await page.locator('#footer').scrollIntoViewIfNeeded();
  
  // Scroll in container
  await page.locator('.scroll-container').hover();
  await page.mouse.wheel(0, 300);
});
```

---

## Step 1298: Assertions - การตรวจสอบผลลัพธ์

```typescript
// tests/e2e/assertions.spec.ts
import { test, expect } from '@playwright/test';

test('assertions examples', async ({ page }) => {
  await page.goto('/');
  
  // ===== Page Assertions =====
  
  // URL
  await expect(page).toHaveURL('http://localhost:3000/');
  await expect(page).toHaveURL(/dashboard/);
  await expect(page).not.toHaveURL('/login');
  
  // Title
  await expect(page).toHaveTitle('My App - หน้าแรก');
  await expect(page).toHaveTitle(/My App/);
  
  // ===== Element Assertions =====
  
  const heading = page.getByRole('heading', { level: 1 });
  const loginBtn = page.getByRole('button', { name: 'Login' });
  const emailInput = page.getByLabel('Email');
  const checkbox = page.getByLabel('Remember me');
  const image = page.getByRole('img', { name: 'Profile' });
  
  // Visibility
  await expect(heading).toBeVisible();
  await expect(page.locator('.spinner')).not.toBeVisible();
  await expect(page.locator('.hidden-element')).toBeHidden();
  
  // Text content
  await expect(heading).toHaveText('ยินดีต้อนรับสู่ My App');
  await expect(heading).toHaveText(/ยินดีต้อนรับ/); // regex
  await expect(heading).toContainText('My App');
  
  // Attribute
  await expect(emailInput).toHaveAttribute('type', 'email');
  await expect(emailInput).toHaveAttribute('placeholder', /กรอกอีเมล/);
  
  // Value
  await emailInput.fill('test@example.com');
  await expect(emailInput).toHaveValue('test@example.com');
  await expect(page.getByLabel('อายุ')).toHaveValue('25');
  
  // Class
  await expect(loginBtn).toHaveClass('btn-primary');
  await expect(loginBtn).toHaveClass(/btn/);
  
  // State
  await expect(loginBtn).toBeEnabled();
  await expect(page.locator('button[disabled]')).toBeDisabled();
  await expect(checkbox).toBeChecked();
  await expect(checkbox).not.toBeChecked();
  
  // Focus
  await emailInput.focus();
  await expect(emailInput).toBeFocused();
  
  // Count
  await expect(page.locator('.product-card')).toHaveCount(10);
  
  // CSS properties
  await expect(heading).toHaveCSS('color', 'rgb(0, 0, 0)');
  await expect(loginBtn).toHaveCSS('background-color', 'rgb(59, 130, 246)');
  
  // Screenshot comparison
  await expect(page).toMatchAriaSnapshot(`
    - heading "ยินดีต้อนรับ" [level=1]
    - button "Login"
  `);
});

// Soft assertions - ไม่หยุด test เมื่อ fail
test('soft assertions', async ({ page }) => {
  await page.goto('/products');
  
  // Soft assertions รวบรวม errors ทั้งหมด
  await expect.soft(page.locator('h1')).toHaveText('สินค้าทั้งหมด');
  await expect.soft(page.locator('.product-count')).toHaveText('100 รายการ');
  await expect.soft(page.locator('.filter-btn')).toBeVisible();
  
  // Check all soft assertions
  // ถ้า soft assertion fail, test จะ fail ตอนจบ
  await page.locator('.load-more-btn').click();
});
```

---

## Step 1299: Page Navigation

```typescript
// tests/e2e/navigation.spec.ts
import { test, expect } from '@playwright/test';

test('navigation examples', async ({ page }) => {
  // goto - ไปที่ URL
  await page.goto('/');
  await page.goto('http://localhost:3000/products');
  await page.goto('/products?category=electronics');
  
  // goto options
  await page.goto('/slow-page', {
    waitUntil: 'networkidle', // รอจนไม่มี network requests
    timeout: 60000,
  });
  
  // go back/forward
  await page.goto('/page1');
  await page.goto('/page2');
  await page.goBack();    // กลับไป /page1
  await page.goForward(); // ไป /page2 อีกครั้ง
  
  // reload
  await page.reload();
  await page.reload({ waitUntil: 'networkidle' });
  
  // Wait for URL
  await page.getByRole('button', { name: 'Go to Dashboard' }).click();
  await page.waitForURL('/dashboard');
  await page.waitForURL(/dashboard/, { timeout: 10000 });
  
  // Wait for navigation
  const [response] = await Promise.all([
    page.waitForNavigation(),
    page.getByRole('link', { name: 'Next Page' }).click(),
  ]);
  
  // Current URL
  const currentURL = page.url();
  console.log('URL ปัจจุบัน:', currentURL);
  
  // Intercept navigation
  page.on('framenavigated', (frame) => {
    if (frame === page.mainFrame()) {
      console.log('Navigate to:', frame.url());
    }
  });
});

test('เปิด new tab', async ({ page, context }) => {
  await page.goto('/');
  
  // รอ new page ที่เปิดจากการ click
  const newPagePromise = context.waitForEvent('page');
  await page.getByRole('link', { name: 'เปิดในแท็บใหม่' }).click();
  const newPage = await newPagePromise;
  
  await newPage.waitForLoadState();
  expect(newPage.url()).toContain('/new-page');
  
  await newPage.close();
});
```

---

## Step 1300: Screenshots และ Videos

```typescript
// tests/e2e/media.spec.ts
import { test, expect } from '@playwright/test';
import path from 'path';

test('screenshots', async ({ page }) => {
  await page.goto('/dashboard');
  
  // Screenshot ทั้งหน้า
  await page.screenshot({ path: 'screenshots/dashboard.png' });
  
  // Screenshot เฉพาะส่วน (viewport only)
  await page.screenshot({
    path: 'screenshots/dashboard-viewport.png',
    clip: { x: 0, y: 0, width: 800, height: 600 }
  });
  
  // Full page screenshot
  await page.screenshot({
    path: 'screenshots/dashboard-full.png',
    fullPage: true
  });
  
  // Screenshot เฉพาะ element
  await page.locator('.chart-container').screenshot({
    path: 'screenshots/chart.png'
  });
  
  // Screenshot เป็น buffer (ไม่บันทึกไฟล์)
  const buffer = await page.screenshot();
  // ใช้ buffer กับ visual comparison tools
  
  // Visual comparison (snapshot testing)
  await expect(page).toHaveScreenshot('dashboard.png');
  await expect(page.locator('.hero-section')).toHaveScreenshot('hero.png');
  
  // Visual comparison with options
  await expect(page).toHaveScreenshot('dashboard.png', {
    maxDiffPixels: 100,        // อนุญาต pixel diff สูงสุด
    threshold: 0.1,             // tolerance 10%
    animations: 'disabled',     // ปิด animations
  });
});

// Video ถูกบันทึกอัตโนมัติตาม config
// video: 'on' | 'off' | 'retain-on-failure' | 'on-first-retry'
test('with video recording', async ({ page }) => {
  // Video จะถูกบันทึกตาม config
  await page.goto('/checkout');
  await page.getByLabel('ชื่อบนบัตร').fill('John Doe');
  await page.getByLabel('หมายเลขบัตร').fill('4111111111111111');
  await page.getByRole('button', { name: 'ชำระเงิน' }).click();
  // Video จะถูกบันทึกใน test-results/
});
```

---

## Step 1301: Multiple Browsers Testing

```typescript
// playwright.config.ts - กำหนด projects
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    // Desktop
    {
      name: 'Desktop Chrome',
      use: {
        ...devices['Desktop Chrome'],
        viewport: { width: 1280, height: 720 },
      },
    },
    {
      name: 'Desktop Firefox',
      use: {
        ...devices['Desktop Firefox'],
      },
    },
    {
      name: 'Desktop Safari',
      use: {
        ...devices['Desktop Safari'],
      },
    },
    
    // Mobile
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
    
    // Tablet
    {
      name: 'iPad',
      use: { ...devices['iPad Pro'] },
    },
    
    // Branded browsers
    {
      name: 'Microsoft Edge',
      use: {
        ...devices['Desktop Edge'],
        channel: 'msedge',
      },
    },
    {
      name: 'Google Chrome',
      use: {
        ...devices['Desktop Chrome'],
        channel: 'chrome',
      },
    },
  ],
});
```

### ทดสอบเฉพาะบาง browsers

```typescript
// รันเฉพาะ chromium
// npx playwright test --project=chromium

// รันเฉพาะ mobile
// npx playwright test --project="Mobile Chrome" --project="Mobile Safari"

// Skip บาง browsers ใน test
import { test, expect } from '@playwright/test';

test('works on all browsers', async ({ page, browserName }) => {
  // ข้ามถ้าเป็น Firefox
  test.skip(browserName === 'firefox', 'Firefox has different behavior');
  
  await page.goto('/');
  await expect(page.locator('h1')).toBeVisible();
});

test.describe('Desktop only features', () => {
  test.skip(({ browserName }) => browserName === 'webkit', 
    'Safari on iOS does not support this');
  
  test('desktop menu', async ({ page }) => {
    await page.goto('/');
    await expect(page.locator('.desktop-menu')).toBeVisible();
  });
});
```

---

## Step 1302: Mobile Viewport Testing

```typescript
// tests/e2e/mobile.spec.ts
import { test, expect, devices } from '@playwright/test';

// ทดสอบบน mobile viewport
test.use({ ...devices['iPhone 12'] });

test('mobile navigation', async ({ page }) => {
  await page.goto('/');
  
  // Mobile menu button ควรมองเห็น
  const menuBtn = page.getByRole('button', { name: 'เมนู' });
  await expect(menuBtn).toBeVisible();
  
  // Desktop nav ไม่ควรมองเห็นบน mobile
  await expect(page.locator('.desktop-nav')).not.toBeVisible();
  
  // เปิด mobile menu
  await menuBtn.click();
  await expect(page.locator('.mobile-menu')).toBeVisible();
  
  // Navigate
  await page.getByRole('link', { name: 'สินค้า' }).click();
  await expect(page).toHaveURL('/products');
});

// ทดสอบ responsive design
test.describe('Responsive Design', () => {
  
  test('mobile layout', async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 812 });
    await page.goto('/');
    
    // ตรวจสอบ mobile layout
    await expect(page.locator('.mobile-header')).toBeVisible();
    await expect(page.locator('.desktop-header')).not.toBeVisible();
  });
  
  test('tablet layout', async ({ page }) => {
    await page.setViewportSize({ width: 768, height: 1024 });
    await page.goto('/');
    
    // ตรวจสอบ tablet layout
    await expect(page.locator('.tablet-sidebar')).toBeVisible();
  });
  
  test('desktop layout', async ({ page }) => {
    await page.setViewportSize({ width: 1280, height: 720 });
    await page.goto('/');
    
    // ตรวจสอบ desktop layout
    await expect(page.locator('.desktop-nav')).toBeVisible();
    await expect(page.locator('.mobile-menu-btn')).not.toBeVisible();
  });
});

// Touch gestures
test('touch interactions', async ({ page }) => {
  await page.goto('/gallery');
  
  // Swipe simulation
  const slider = page.locator('.image-slider');
  const box = await slider.boundingBox();
  
  if (box) {
    // Swipe left
    await page.touchscreen.tap(box.x + box.width * 0.8, box.y + box.height / 2);
    await page.mouse.move(box.x + box.width * 0.8, box.y + box.height / 2);
    await page.mouse.down();
    await page.mouse.move(box.x + box.width * 0.2, box.y + box.height / 2);
    await page.mouse.up();
  }
});
```

---

## Step 1303: Network Interception

```typescript
// tests/e2e/network.spec.ts
import { test, expect } from '@playwright/test';

test('mock API responses', async ({ page }) => {
  // Mock GET request
  await page.route('/api/products', async (route) => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({
        products: [
          { id: 1, name: 'Laptop', price: 25000 },
          { id: 2, name: 'Mouse', price: 500 },
        ],
        total: 2,
      }),
    });
  });
  
  await page.goto('/products');
  
  // ตรวจสอบว่า mock data ถูกแสดง
  await expect(page.getByText('Laptop')).toBeVisible();
  await expect(page.getByText('Mouse')).toBeVisible();
});

test('mock error response', async ({ page }) => {
  // Mock 500 error
  await page.route('/api/products', async (route) => {
    await route.fulfill({
      status: 500,
      body: JSON.stringify({ message: 'Internal Server Error' }),
    });
  });
  
  await page.goto('/products');
  
  // ตรวจสอบ error state
  await expect(page.getByText('เกิดข้อผิดพลาด')).toBeVisible();
});

test('intercept and modify request', async ({ page }) => {
  await page.route('/api/search*', async (route, request) => {
    // อ่าน request เดิม
    const url = new URL(request.url());
    console.log('Search query:', url.searchParams.get('q'));
    
    // ส่ง request ต่อ (แต่เปลี่ยน header)
    await route.continue({
      headers: {
        ...request.headers(),
        'X-Test-Mode': 'true',
      },
    });
  });
  
  await page.goto('/search?q=laptop');
});

test('wait for specific request', async ({ page }) => {
  // รอ request ที่ต้องการ
  const requestPromise = page.waitForRequest('/api/cart');
  
  await page.goto('/products');
  await page.getByRole('button', { name: 'เพิ่มลงตะกร้า' }).first().click();
  
  const request = await requestPromise;
  expect(request.method()).toBe('POST');
  
  // รอ response
  const responsePromise = page.waitForResponse('/api/cart');
  await page.getByRole('button', { name: 'เพิ่มลงตะกร้า' }).first().click();
  
  const response = await responsePromise;
  expect(response.status()).toBe(200);
  
  const data = await response.json();
  expect(data.itemCount).toBe(1);
});

test('block resources', async ({ page }) => {
  // Block images เพื่อเร่งความเร็ว test
  await page.route('**/*.{png,jpg,jpeg,gif,svg}', route => route.abort());
  
  // Block analytics
  await page.route('**/google-analytics.com/**', route => route.abort());
  await page.route('**/facebook.com/tr*', route => route.abort());
  
  await page.goto('/');
  // เว็บจะโหลดเร็วขึ้นเนื่องจาก block รูปภาพ
});

test('HAR recording', async ({ page }) => {
  // บันทึก network traffic เป็น HAR file
  await page.routeFromHAR('./har-files/api-responses.har', {
    url: /api/,
    update: false, // false = replay, true = record new
  });
  
  await page.goto('/dashboard');
  // ใช้ responses จาก HAR file
});
```

---

## Step 1304: Authentication ใน E2E Tests

```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

// วิธีที่ 1: Login ผ่าน UI ทุกครั้ง (ช้า)
test('login via UI', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('อีเมล').fill('user@example.com');
  await page.getByLabel('รหัสผ่าน').fill('password123');
  await page.getByRole('button', { name: 'เข้าสู่ระบบ' }).click();
  await expect(page).toHaveURL('/dashboard');
});

// วิธีที่ 2: Storage State - Login ครั้งเดียว แล้ว reuse (เร็วกว่า)
// setup/auth.ts - Login และบันทึก state
import { chromium, type FullConfig } from '@playwright/test';

async function globalSetup(config: FullConfig) {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();
  
  await page.goto('http://localhost:3000/login');
  await page.getByLabel('อีเมล').fill('user@example.com');
  await page.getByLabel('รหัสผ่าน').fill('password123');
  await page.getByRole('button', { name: 'เข้าสู่ระบบ' }).click();
  await page.waitForURL('/dashboard');
  
  // บันทึก storage state (cookies, localStorage)
  await context.storageState({ path: './auth-state/user.json' });
  
  await browser.close();
}

export default globalSetup;
```

```typescript
// playwright.config.ts - ใช้ globalSetup
export default defineConfig({
  globalSetup: require.resolve('./setup/auth.ts'),
  use: {
    storageState: './auth-state/user.json', // ใช้สำหรับทุก test
  },
  projects: [
    {
      name: 'authenticated',
      use: { storageState: './auth-state/user.json' },
      testMatch: '**/*.auth.spec.ts',
    },
    {
      name: 'unauthenticated',
      testMatch: '**/*.public.spec.ts',
    },
  ],
});
```

```typescript
// tests/e2e/dashboard.auth.spec.ts
import { test, expect } from '@playwright/test';

// test นี้ใช้ storageState จาก config อัตโนมัติ
test('authenticated user can see dashboard', async ({ page }) => {
  await page.goto('/dashboard');
  
  // ไม่ต้อง login เพราะใช้ stored state
  await expect(page.getByText('ยินดีต้อนรับ')).toBeVisible();
  await expect(page.locator('.user-menu')).toBeVisible();
});

// วิธีที่ 3: API Login - เร็วที่สุด
test('login via API', async ({ page, request }) => {
  // Login ผ่าน API
  const loginResponse = await request.post('/api/auth/login', {
    data: {
      email: 'user@example.com',
      password: 'password123',
    },
  });
  
  const { token } = await loginResponse.json();
  
  // ตั้งค่า cookie/localStorage
  await page.goto('/');
  await page.evaluate((token) => {
    localStorage.setItem('authToken', token);
  }, token);
  
  // ไปหน้าที่ต้อง auth
  await page.goto('/dashboard');
  await expect(page).toHaveURL('/dashboard');
});
```

---

## Step 1305: Page Object Model (POM)

Page Object Model เป็น design pattern ที่แยก logic ของ page ออกจาก test logic

```typescript
// pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  
  // Locators
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;
  readonly forgotPasswordLink: Locator;
  
  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('อีเมล');
    this.passwordInput = page.getByLabel('รหัสผ่าน');
    this.loginButton = page.getByRole('button', { name: 'เข้าสู่ระบบ' });
    this.errorMessage = page.getByRole('alert');
    this.forgotPasswordLink = page.getByRole('link', { name: 'ลืมรหัสผ่าน?' });
  }
  
  async goto() {
    await this.page.goto('/login');
  }
  
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
  
  async expectErrorMessage(message: string) {
    await expect(this.errorMessage).toHaveText(message);
  }
  
  async expectLoginSuccess() {
    await expect(this.page).toHaveURL('/dashboard');
  }
}
```

```typescript
// pages/ProductsPage.ts
import { Page, Locator, expect } from '@playwright/test';

export class ProductsPage {
  readonly page: Page;
  readonly searchInput: Locator;
  readonly productCards: Locator;
  readonly filterButton: Locator;
  readonly sortDropdown: Locator;
  
  constructor(page: Page) {
    this.page = page;
    this.searchInput = page.getByPlaceholder('ค้นหาสินค้า');
    this.productCards = page.locator('[data-testid="product-card"]');
    this.filterButton = page.getByRole('button', { name: 'กรองสินค้า' });
    this.sortDropdown = page.getByLabel('เรียงตาม');
  }
  
  async goto() {
    await this.page.goto('/products');
  }
  
  async search(query: string) {
    await this.searchInput.fill(query);
    await this.searchInput.press('Enter');
    await this.page.waitForLoadState('networkidle');
  }
  
  async filterByCategory(category: string) {
    await this.filterButton.click();
    await this.page.getByLabel(`หมวด ${category}`).check();
    await this.page.getByRole('button', { name: 'ยืนยัน' }).click();
  }
  
  async sortBy(option: string) {
    await this.sortDropdown.selectOption(option);
  }
  
  async getProductCount(): Promise<number> {
    return await this.productCards.count();
  }
  
  async addToCart(productName: string) {
    const product = this.productCards.filter({ hasText: productName });
    await product.getByRole('button', { name: 'เพิ่มลงตะกร้า' }).click();
  }
  
  async expectProductCount(count: number) {
    await expect(this.productCards).toHaveCount(count);
  }
}
```

```typescript
// tests/e2e/login.spec.ts - ใช้ POM
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../pages/LoginPage';
import { DashboardPage } from '../../pages/DashboardPage';

test.describe('Login Flow', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;
  
  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
    await loginPage.goto();
  });
  
  test('login สำเร็จด้วย credentials ที่ถูกต้อง', async ({ page }) => {
    await loginPage.login('user@example.com', 'password123');
    await loginPage.expectLoginSuccess();
    await expect(dashboardPage.welcomeMessage).toContainText('ยินดีต้อนรับ');
  });
  
  test('แสดง error เมื่อ password ผิด', async ({ page }) => {
    await loginPage.login('user@example.com', 'wrongpassword');
    await loginPage.expectErrorMessage('อีเมลหรือรหัสผ่านไม่ถูกต้อง');
    await expect(page).toHaveURL('/login');
  });
  
  test('แสดง error เมื่อ email ไม่ถูกต้อง', async ({ page }) => {
    await loginPage.login('notanemail', 'password123');
    await loginPage.expectErrorMessage('กรุณากรอกอีเมลให้ถูกต้อง');
  });
  
  test('ลืมรหัสผ่าน link ทำงาน', async ({ page }) => {
    await loginPage.forgotPasswordLink.click();
    await expect(page).toHaveURL('/forgot-password');
  });
});
```

---

## Step 1306: Test Fixtures

Fixtures คือ mechanism สำหรับ setup และ teardown ที่ reusable

```typescript
// fixtures/index.ts
import { test as base, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { ProductsPage } from '../pages/ProductsPage';
import { CartPage } from '../pages/CartPage';

// กำหนด types
type Pages = {
  loginPage: LoginPage;
  productsPage: ProductsPage;
  cartPage: CartPage;
};

type TestData = {
  testUser: {
    email: string;
    password: string;
    name: string;
  };
  testProduct: {
    name: string;
    price: number;
  };
};

// Extend base test
export const test = base.extend<Pages & TestData>({
  // Page fixtures
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },
  
  productsPage: async ({ page }, use) => {
    const productsPage = new ProductsPage(page);
    await use(productsPage);
  },
  
  cartPage: async ({ page }, use) => {
    const cartPage = new CartPage(page);
    await use(cartPage);
  },
  
  // Data fixtures
  testUser: async ({}, use) => {
    await use({
      email: 'test@example.com',
      password: 'TestPass123',
      name: 'Test User',
    });
  },
  
  testProduct: async ({}, use) => {
    await use({
      name: 'Test Laptop',
      price: 25000,
    });
  },
});

// Fixture ที่ login อัตโนมัติ
export const authenticatedTest = base.extend<Pages>({
  page: async ({ page }, use) => {
    // Login ก่อน
    await page.goto('/login');
    await page.getByLabel('อีเมล').fill('user@example.com');
    await page.getByLabel('รหัสผ่าน').fill('password123');
    await page.getByRole('button', { name: 'เข้าสู่ระบบ' }).click();
    await page.waitForURL('/dashboard');
    
    // ใช้ page
    await use(page);
    
    // Cleanup (logout)
    await page.goto('/logout');
  },
  
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
  
  productsPage: async ({ page }, use) => {
    await use(new ProductsPage(page));
  },
  
  cartPage: async ({ page }, use) => {
    await use(new CartPage(page));
  },
});

export { expect };
```

```typescript
// tests/e2e/checkout.spec.ts
import { test, expect } from '../../fixtures';

test('ผู้ใช้สามารถ checkout ได้', async ({
  loginPage,
  productsPage,
  cartPage,
  testUser,
  testProduct,
  page,
}) => {
  // Login
  await loginPage.goto();
  await loginPage.login(testUser.email, testUser.password);
  
  // ค้นหาและเพิ่มสินค้า
  await productsPage.goto();
  await productsPage.search(testProduct.name);
  await productsPage.addToCart(testProduct.name);
  
  // ไป cart
  await cartPage.goto();
  await cartPage.expectItemCount(1);
  
  // Checkout
  await cartPage.checkout();
  await expect(page).toHaveURL('/checkout');
});
```

---

## Step 1307: Playwright UI Mode

```bash
# รัน Playwright UI Mode
npx playwright test --ui

# หรือ
npx playwright test --ui --project=chromium
```

UI Mode ช่วยให้:
- ดู tests ทั้งหมดในรูปแบบ tree
- รัน tests ทีละตัวหรือทั้งหมด
- ดู timeline ของ actions
- Debug tests ได้ง่ายขึ้น
- Watch mode - auto-rerun เมื่อไฟล์เปลี่ยน

```typescript
// Debug ใน test
test('debug example', async ({ page }) => {
  await page.goto('/');
  
  // หยุด execution (ใช้กับ --debug flag)
  await page.pause();
  
  // หรือใช้ PWDEBUG=1
  // PWDEBUG=1 npx playwright test login.spec.ts
});
```

### Playwright Inspector

```bash
# เปิด Inspector mode
npx playwright test --debug

# Record new tests
npx playwright codegen http://localhost:3000
```

---

## Step 1308: Running Tests ใน CI

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    
    - name: Run E2E tests
      run: npx playwright test
      env:
        CI: true
        BASE_URL: http://localhost:3000
    
    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()  # อัพโหลดแม้ test fail
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
    
    - name: Upload test videos
      uses: actions/upload-artifact@v3
      if: failure()  # อัพโหลดเมื่อ test fail
      with:
        name: test-videos
        path: test-results/
```

```typescript
// playwright.config.ts - CI configuration
export default defineConfig({
  // Strict mode for CI
  forbidOnly: !!process.env.CI,
  
  // More retries on CI
  retries: process.env.CI ? 2 : 0,
  
  // Fewer workers on CI (resource constraints)
  workers: process.env.CI ? 2 : undefined,
  
  // CI reporter
  reporter: process.env.CI
    ? [['github'], ['html']]
    : [['list'], ['html']],
  
  use: {
    // Capture trace on CI for debugging
    trace: process.env.CI ? 'on' : 'on-first-retry',
    
    // Video on CI
    video: process.env.CI ? 'retain-on-failure' : 'off',
    
    // Screenshot on CI
    screenshot: process.env.CI ? 'only-on-failure' : 'off',
    
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
  },
  
  // Only run chromium on CI for speed
  projects: process.env.CI
    ? [{ name: 'chromium', use: devices['Desktop Chrome'] }]
    : [
        { name: 'chromium', use: devices['Desktop Chrome'] },
        { name: 'firefox', use: devices['Desktop Firefox'] },
        { name: 'webkit', use: devices['Desktop Safari'] },
      ],
});
```

---

## Step 1309: Advanced Patterns

### Test Data Management

```typescript
// helpers/test-data.ts
export const createTestUser = () => ({
  email: `test-${Date.now()}@example.com`,
  password: 'TestPass123!',
  name: `Test User ${Date.now()}`,
});

export const testProducts = [
  { id: 1, name: 'Laptop Pro', price: 45000, category: 'Electronics' },
  { id: 2, name: 'Wireless Mouse', price: 1200, category: 'Accessories' },
  { id: 3, name: 'Mechanical Keyboard', price: 3500, category: 'Accessories' },
];

// API Helpers
export async function createUserViaAPI(request: APIRequestContext) {
  const user = createTestUser();
  
  const response = await request.post('/api/users', {
    data: user,
    headers: {
      'Authorization': `Bearer ${process.env.ADMIN_TOKEN}`,
    },
  });
  
  const created = await response.json();
  return { ...user, id: created.id };
}
```

### Parallel Test Execution

```typescript
// tests/e2e/parallel.spec.ts
import { test, expect } from '@playwright/test';

// กำหนดให้ tests รันแบบ parallel
test.describe.configure({ mode: 'parallel' });

test('test 1', async ({ page }) => {
  await page.goto('/page1');
  // ...
});

test('test 2', async ({ page }) => {
  await page.goto('/page2');
  // ...
});

// กำหนดให้ tests รันแบบ serial (ตามลำดับ)
test.describe.configure({ mode: 'serial' });

test.describe('Sequential tests', () => {
  let userId: string;
  
  test('สร้าง user', async ({ page }) => {
    // ...
    userId = 'created-user-id';
  });
  
  test('อัปเดต user (ใช้ userId จาก test ก่อน)', async ({ page }) => {
    // userId มาจาก test ก่อน
    await page.goto(`/users/${userId}`);
  });
});
```

### Custom Reporters

```typescript
// reporters/custom-reporter.ts
import type {
  Reporter,
  FullConfig,
  Suite,
  TestCase,
  TestResult,
} from '@playwright/test/reporter';

class CustomReporter implements Reporter {
  onBegin(config: FullConfig, suite: Suite) {
    console.log(`\n🚀 เริ่มต้น ${suite.allTests().length} tests\n`);
  }
  
  onTestEnd(test: TestCase, result: TestResult) {
    const status = result.status === 'passed' ? '✅' : '❌';
    const duration = (result.duration / 1000).toFixed(2);
    console.log(`${status} ${test.title} (${duration}s)`);
  }
  
  onEnd(result: { status: string }) {
    console.log(`\n📊 ผลลัพธ์: ${result.status}\n`);
  }
}

export default CustomReporter;
```

---

## Step 1310: Full E2E Test Example

```typescript
// tests/e2e/shopping-flow.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../pages/LoginPage';
import { ProductsPage } from '../../pages/ProductsPage';
import { CartPage } from '../../pages/CartPage';
import { CheckoutPage } from '../../pages/CheckoutPage';

test.describe('Shopping Flow', () => {
  
  test('ผู้ใช้สามารถซื้อสินค้าได้ตั้งแต่ต้นจนจบ', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const productsPage = new ProductsPage(page);
    const cartPage = new CartPage(page);
    const checkoutPage = new CheckoutPage(page);
    
    // Step 1: Login
    await test.step('Login', async () => {
      await loginPage.goto();
      await loginPage.login('customer@example.com', 'password123');
      await expect(page).toHaveURL('/dashboard');
    });
    
    // Step 2: ค้นหาสินค้า
    await test.step('ค้นหาสินค้า', async () => {
      await productsPage.goto();
      await productsPage.search('laptop');
      await expect(productsPage.productCards).toHaveCount(5);
    });
    
    // Step 3: เพิ่มสินค้าลง Cart
    await test.step('เพิ่มสินค้าลง Cart', async () => {
      await productsPage.addToCart('Laptop Pro');
      
      // ตรวจสอบ notification
      const notification = page.getByRole('alert');
      await expect(notification).toHaveText('เพิ่มสินค้าลงตะกร้าแล้ว');
      
      // ตรวจสอบ cart count
      const cartBadge = page.getByTestId('cart-count');
      await expect(cartBadge).toHaveText('1');
    });
    
    // Step 4: ตรวจสอบ Cart
    await test.step('ตรวจสอบ Cart', async () => {
      await cartPage.goto();
      await expect(cartPage.items).toHaveCount(1);
      await expect(cartPage.totalPrice).toContainText('45,000');
    });
    
    // Step 5: Checkout
    await test.step('กรอกข้อมูลการจัดส่ง', async () => {
      await cartPage.proceedToCheckout();
      await expect(page).toHaveURL('/checkout');
      
      await checkoutPage.fillShippingInfo({
        firstName: 'สมชาย',
        lastName: 'ใจดี',
        address: '123 ถนนสุขุมวิท',
        city: 'กรุงเทพมหานคร',
        zipCode: '10110',
        phone: '0812345678',
      });
    });
    
    // Step 6: ชำระเงิน
    await test.step('ชำระเงิน', async () => {
      await checkoutPage.fillPaymentInfo({
        cardNumber: '4111111111111111',
        expiry: '12/25',
        cvv: '123',
        cardName: 'SOMCHAI JAIDEE',
      });
      
      await checkoutPage.placeOrder();
    });
    
    // Step 7: ตรวจสอบ Order Confirmation
    await test.step('ตรวจสอบ Order Confirmation', async () => {
      await expect(page).toHaveURL(/\/order-confirmation\/\d+/);
      await expect(page.getByRole('heading')).toContainText('ขอบคุณสำหรับการสั่งซื้อ');
      
      const orderNumber = await page.getByTestId('order-number').textContent();
      expect(orderNumber).toMatch(/^ORD-\d+$/);
    });
  });
  
  test('แสดง error เมื่อ credit card ไม่ถูกต้อง', async ({ page }) => {
    const checkoutPage = new CheckoutPage(page);
    
    // Setup: Login และเพิ่มสินค้าลง cart
    await page.goto('/');
    await page.evaluate(() => {
      localStorage.setItem('cart', JSON.stringify([
        { id: 1, name: 'Laptop', price: 45000, qty: 1 }
      ]));
    });
    
    await page.goto('/checkout');
    
    await checkoutPage.fillShippingInfo({
      firstName: 'Test',
      lastName: 'User',
      address: '123 Test St',
      city: 'Bangkok',
      zipCode: '10110',
      phone: '0812345678',
    });
    
    // ใส่ card ไม่ถูกต้อง
    await checkoutPage.fillPaymentInfo({
      cardNumber: '4111111111111112', // invalid
      expiry: '12/25',
      cvv: '123',
      cardName: 'TEST USER',
    });
    
    await checkoutPage.placeOrder();
    
    // ตรวจสอบ error
    await expect(page.getByRole('alert')).toContainText('บัตรเครดิตไม่ถูกต้อง');
    await expect(page).toHaveURL('/checkout'); // ยังอยู่หน้า checkout
  });
});
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Basic E2E Test
สร้าง test สำหรับ contact form:
- ไปหน้า `/contact`
- กรอก: ชื่อ, อีเมล, ข้อความ
- คลิก Submit
- ตรวจสอบว่าแสดง "ส่งข้อความสำเร็จ"

### แบบฝึกหัดที่ 2: Authentication Flow
- ทดสอบ Register → Verify Email → Login
- ทดสอบ Logout แล้วพยายาม access หน้าที่ต้อง auth
- ทดสอบ "จำฉันไว้" feature

### แบบฝึกหัดที่ 3: Page Object Model
สร้าง POM สำหรับ:
- `DashboardPage`
- `ProfilePage`
- `SettingsPage`

แล้วเขียน test ที่ใช้ทั้ง 3 pages

### แบบฝึกหัดที่ 4: API Mocking
- Mock API `/api/weather` ที่คืนค่า mock data
- ตรวจสอบว่าหน้าเว็บแสดงข้อมูล weather ถูกต้อง
- Mock error response แล้วตรวจสอบ error handling

### แบบฝึกหัดที่ 5: Cross-Browser Testing
- เขียน test ที่รันบน Chromium, Firefox, WebKit
- เพิ่ม mobile viewport test
- ตรวจสอบ responsive behavior

---

## สรุป

| แนวคิด | คำอธิบาย |
|--------|----------|
| E2E Test | ทดสอบทั้งระบบเหมือนผู้ใช้จริง |
| Playwright | Framework E2E ที่รองรับหลาย browser |
| Locators | วิธีหา elements: getByRole, getByText, getByLabel |
| Actions | การโต้ตอบ: click, fill, press, select |
| Assertions | การตรวจสอบ: toBeVisible, toHaveText, toHaveURL |
| POM | Design pattern แยก page logic จาก test |
| Fixtures | Setup/teardown ที่ reusable |
| Network Mock | จำลอง API responses |
| Storage State | เก็บ session เพื่อ reuse ใน tests |

```bash
# คำสั่งที่ใช้บ่อย
npx playwright test                    # รัน tests ทั้งหมด
npx playwright test login.spec.ts      # รัน test เฉพาะไฟล์
npx playwright test --headed           # แสดง browser
npx playwright test --debug            # debug mode
npx playwright test --ui               # UI mode
npx playwright codegen http://localhost:3000  # record tests
npx playwright show-report             # ดู HTML report
npx playwright test --project=chromium # เฉพาะ browser
```
