# ตอนที่ 10: Date และ Time ใน JavaScript

## บทนำ

การทำงานกับวันที่และเวลาเป็นส่วนสำคัญในการพัฒนาโปรแกรม JavaScript มี `Date` object ที่ built-in สำหรับจัดการวันที่และเวลา และ `Intl.DateTimeFormat` สำหรับจัดรูปแบบการแสดงผลตามภาษาและภูมิภาค

ในบทนี้เราจะครอบคลุม Steps 171-190

---

## Step 171: การสร้าง Date Object

### new Date()

```javascript
// วันที่ปัจจุบัน
const now = new Date();
console.log(now);  // Mon Jan 15 2024 10:30:45 GMT+0700 (...)

// จาก timestamp (milliseconds since Jan 1, 1970 UTC)
const fromTimestamp = new Date(0);
console.log(fromTimestamp);  // Thu Jan 01 1970 07:00:00 GMT+0700

const specificTime = new Date(1705286445000);
console.log(specificTime);   // วันที่เฉพาะเจาะจง
```

```javascript
// จาก string
const fromString1 = new Date('2024-01-15');
const fromString2 = new Date('January 15, 2024');
const fromString3 = new Date('2024-01-15T10:30:00');
const fromString4 = new Date('2024-01-15T10:30:00Z');       // UTC
const fromString5 = new Date('2024-01-15T10:30:00+07:00'); // ไทย

console.log(fromString1);
console.log(fromString3);
```

```javascript
// จาก parameters (year, month, day, hours, minutes, seconds, ms)
// ระวัง! month เริ่มที่ 0 (0 = มกราคม, 11 = ธันวาคม)
const byParts = new Date(2024, 0, 15);           // 15 มกราคม 2024
const withTime = new Date(2024, 0, 15, 10, 30);  // + 10:30 น.
const full = new Date(2024, 0, 15, 10, 30, 45, 500); // + วินาที + ms

console.log(byParts);
console.log(withTime);

// ตัวอย่าง: เดือนทั้งหมด
const months = [
  'มกราคม', 'กุมภาพันธ์', 'มีนาคม', 'เมษายน',
  'พฤษภาคม', 'มิถุนายน', 'กรกฎาคม', 'สิงหาคม',
  'กันยายน', 'ตุลาคม', 'พฤศจิกายน', 'ธันวาคม'
];

for (let m = 0; m < 12; m++) {
  const date = new Date(2024, m, 1);
  console.log(`${months[m]}: month = ${date.getMonth()}`);
}
```

---

## Step 172: Date.now()

```javascript
// Date.now() คืน timestamp ปัจจุบัน (milliseconds)
const timestamp = Date.now();
console.log(timestamp);  // เช่น 1705286445123

// เหมือน new Date().getTime()
console.log(Date.now() === new Date().getTime());  // true (เกือบๆ)

// ใช้วัดเวลา execution
function measureTime(fn) {
  const start = Date.now();
  const result = fn();
  const elapsed = Date.now() - start;
  return { result, elapsed: `${elapsed}ms` };
}

const { result, elapsed } = measureTime(() => {
  let sum = 0;
  for (let i = 0; i < 1000000; i++) sum += i;
  return sum;
});

console.log(`ผลลัพธ์: ${result}, ใช้เวลา: ${elapsed}`);
```

```javascript
// performance.now() (ละเอียดกว่า)
// ใช้ใน browser หรือ Node.js
if (typeof performance !== 'undefined') {
  const start = performance.now();
  // ทำงานบางอย่าง
  const end = performance.now();
  console.log(`ใช้เวลา ${(end - start).toFixed(3)}ms`);
}

// ใน Node.js ใช้ process.hrtime.bigint()
// const start = process.hrtime.bigint();
// const end = process.hrtime.bigint();
// console.log(`${end - start}ns`);
```

---

## Step 173: Getting Date Parts - getFullYear, getMonth, getDate, getDay

```javascript
const date = new Date('2024-01-15');  // 15 มกราคม 2024 (วันจันทร์)

// ปี, เดือน, วัน
console.log(date.getFullYear());  // 2024
console.log(date.getMonth());     // 0 (มกราคม = 0!)
console.log(date.getDate());      // 15 (วันที่ 1-31)
console.log(date.getDay());       // 1 (วันในสัปดาห์ 0=อาทิตย์, 1=จันทร์)

// เดือนแบบเข้าใจง่าย
const monthNumber = date.getMonth() + 1;
console.log(monthNumber);  // 1 (มกราคม)
```

```javascript
// ชื่อวันและเดือนภาษาไทย
const thaiDays = ['อาทิตย์', 'จันทร์', 'อังคาร', 'พุธ', 'พฤหัสบดี', 'ศุกร์', 'เสาร์'];
const thaiMonths = [
  'มกราคม', 'กุมภาพันธ์', 'มีนาคม', 'เมษายน',
  'พฤษภาคม', 'มิถุนายน', 'กรกฎาคม', 'สิงหาคม',
  'กันยายน', 'ตุลาคม', 'พฤศจิกายน', 'ธันวาคม'
];

function formatThaiDate(date) {
  const day = thaiDays[date.getDay()];
  const dateNum = date.getDate();
  const month = thaiMonths[date.getMonth()];
  const year = date.getFullYear() + 543;  // แปลงเป็น พ.ศ.
  return `วัน${day}ที่ ${dateNum} ${month} พ.ศ. ${year}`;
}

const today = new Date('2024-01-15');
console.log(formatThaiDate(today));
// 'วันจันทร์ที่ 15 มกราคม พ.ศ. 2567'
```

```javascript
// UTC methods (ไม่คำนึงถึง timezone)
const date2 = new Date('2024-01-15T00:00:00Z');
console.log(date2.getUTCFullYear());  // 2024
console.log(date2.getUTCMonth());     // 0
console.log(date2.getUTCDate());      // 15
console.log(date2.getUTCDay());       // 1
```

---

## Step 174: Getting Time Parts

```javascript
const now = new Date('2024-01-15T14:30:45.500');

console.log(now.getHours());         // 14 (0-23)
console.log(now.getMinutes());       // 30 (0-59)
console.log(now.getSeconds());       // 45 (0-59)
console.log(now.getMilliseconds());  // 500 (0-999)

// UTC equivalents
console.log(now.getUTCHours());   // 7 (UTC+7)
console.log(now.getTimezoneOffset()); // -420 (minutes, -7 hours for UTC+7)
```

```javascript
// format time
function formatTime(date, use24h = true) {
  const h = date.getHours();
  const m = date.getMinutes();
  const s = date.getSeconds();
  
  if (use24h) {
    return [h, m, s].map(n => String(n).padStart(2, '0')).join(':');
  }
  
  const hour12 = h % 12 || 12;
  const ampm = h < 12 ? 'AM' : 'PM';
  return `${hour12}:${String(m).padStart(2, '0')} ${ampm}`;
}

const d = new Date('2024-01-15T14:30:45');
console.log(formatTime(d));         // '14:30:45'
console.log(formatTime(d, false));  // '2:30 PM'
```

```javascript
// getTime() - timestamp
const date = new Date('2024-01-15');
const timestamp = date.getTime();
console.log(timestamp);  // milliseconds since epoch

// แปลงกลับ
const restored = new Date(timestamp);
console.log(restored.toDateString() === date.toDateString());  // true
```

---

## Step 175: Setting Date Parts

```javascript
const date = new Date('2024-01-15T10:30:00');

// setFullYear, setMonth, setDate, setDay (ไม่มี setDay!)
date.setFullYear(2025);
console.log(date.getFullYear());  // 2025

date.setMonth(5);   // มิถุนายน (5)
console.log(date.getMonth());  // 5

date.setDate(20);
console.log(date.getDate());  // 20

// setHours, setMinutes, setSeconds, setMilliseconds
date.setHours(15);
date.setMinutes(45);
date.setSeconds(30);
console.log(date.toTimeString());
```

```javascript
// set methods คืน timestamp
const d = new Date();
const ts = d.setFullYear(2030);
console.log(ts);  // timestamp ใหม่

// ใช้ set เพื่อสร้าง date เฉพาะ
function createDate(year, month, day) {
  const date = new Date();
  date.setFullYear(year);
  date.setMonth(month - 1);  // ลบ 1 เพราะ month เริ่มที่ 0
  date.setDate(day);
  date.setHours(0, 0, 0, 0);  // รีเซ็ตเวลา
  return date;
}

const birthday = createDate(1990, 5, 20);
console.log(birthday.toDateString());
```

```javascript
// เพิ่ม/ลดวัน
function addDays(date, days) {
  const result = new Date(date);
  result.setDate(result.getDate() + days);
  return result;
}

function addMonths(date, months) {
  const result = new Date(date);
  result.setMonth(result.getMonth() + months);
  return result;
}

function addYears(date, years) {
  const result = new Date(date);
  result.setFullYear(result.getFullYear() + years);
  return result;
}

const today = new Date('2024-01-15');
console.log(addDays(today, 7).toDateString());     // 1 สัปดาห์ข้างหน้า
console.log(addMonths(today, 3).toDateString());   // 3 เดือนข้างหน้า
console.log(addYears(today, 1).toDateString());    // 1 ปีข้างหน้า
```

---

## Step 176: Date Comparison

```javascript
const date1 = new Date('2024-01-15');
const date2 = new Date('2024-06-20');
const date3 = new Date('2024-01-15');

// เปรียบเทียบด้วย < >
console.log(date1 < date2);   // true
console.log(date1 > date2);   // false
console.log(date1 <= date2);  // true

// ระวัง! === จะ false เพราะเป็น object
console.log(date1 === date3);           // false!
console.log(date1.getTime() === date3.getTime());  // true

// หรือใช้ toString
console.log(date1.toString() === date3.toString());  // true
```

```javascript
// ฟังก์ชันเปรียบเทียบ
function isSameDay(d1, d2) {
  return d1.getFullYear() === d2.getFullYear() &&
         d1.getMonth() === d2.getMonth() &&
         d1.getDate() === d2.getDate();
}

function isBefore(d1, d2) {
  return d1.getTime() < d2.getTime();
}

function isAfter(d1, d2) {
  return d1.getTime() > d2.getTime();
}

function isBetween(date, start, end) {
  return date >= start && date <= end;
}

const today = new Date();
const yesterday = addDays(today, -1);
const tomorrow = addDays(today, 1);

console.log(isBefore(yesterday, today));  // true
console.log(isAfter(tomorrow, today));    // true
console.log(isBetween(today, yesterday, tomorrow));  // true

function addDays(date, days) {
  const result = new Date(date);
  result.setDate(result.getDate() + days);
  return result;
}
```

```javascript
// หาความแตกต่างระหว่างวันที่
function daysBetween(d1, d2) {
  const oneDay = 24 * 60 * 60 * 1000;  // milliseconds in a day
  return Math.round(Math.abs((d1 - d2) / oneDay));
}

function monthsBetween(d1, d2) {
  const yearDiff = d2.getFullYear() - d1.getFullYear();
  const monthDiff = d2.getMonth() - d1.getMonth();
  return yearDiff * 12 + monthDiff;
}

const birth = new Date('1990-05-20');
const today = new Date('2024-01-15');

console.log(`อายุ ${daysBetween(birth, today)} วัน`);
console.log(`อายุ ${Math.floor(daysBetween(birth, today) / 365)} ปี`);

// คำนวณอายุ
function calculateAge(birthDate) {
  const today = new Date();
  let age = today.getFullYear() - birthDate.getFullYear();
  const monthDiff = today.getMonth() - birthDate.getMonth();
  
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birthDate.getDate())) {
    age--;
  }
  
  return age;
}

console.log(`อายุ: ${calculateAge(birth)} ปี`);
```

---

## Step 177: Date Formatting

### toLocaleDateString(), toLocaleTimeString(), toLocaleString()

```javascript
const date = new Date('2024-01-15T10:30:45');

// รูปแบบต่างๆ
console.log(date.toLocaleDateString());       // ตามระบบ
console.log(date.toLocaleDateString('th-TH')); // '15/1/2567' (พ.ศ.)
console.log(date.toLocaleDateString('en-US')); // '1/15/2024'
console.log(date.toLocaleDateString('ja-JP')); // '2024/1/15'

console.log(date.toLocaleTimeString());
console.log(date.toLocaleTimeString('th-TH')); // '10:30:45'
console.log(date.toLocaleTimeString('en-US')); // '10:30:45 AM'

console.log(date.toLocaleString('th-TH'));
// '15/1/2567 10:30:45'
```

```javascript
// options สำหรับ toLocaleDateString
const options = {
  weekday: 'long',   // 'วันจันทร์'
  year: 'numeric',   // '2567'
  month: 'long',     // 'มกราคม'
  day: 'numeric'     // '15'
};

console.log(date.toLocaleDateString('th-TH', options));
// 'วันจันทร์ที่ 15 มกราคม พ.ศ. 2567'

// แบบสั้น
const shortOptions = {
  year: '2-digit',
  month: '2-digit',
  day: '2-digit'
};

console.log(date.toLocaleDateString('th-TH', shortOptions));
// '15/01/67'
```

```javascript
// toString methods
const d = new Date('2024-01-15T10:30:45');

console.log(d.toString());           // Full string
console.log(d.toDateString());       // 'Mon Jan 15 2024'
console.log(d.toTimeString());       // '10:30:45 GMT+0700'
console.log(d.toISOString());        // '2024-01-15T03:30:45.000Z' (UTC)
console.log(d.toUTCString());        // 'Mon, 15 Jan 2024 03:30:45 GMT'
console.log(d.toJSON());             // เหมือน toISOString
```

---

## Step 178: Date Arithmetic

```javascript
// milliseconds ต่อหน่วยเวลา
const MS = {
  SECOND: 1000,
  MINUTE: 60 * 1000,
  HOUR: 60 * 60 * 1000,
  DAY: 24 * 60 * 60 * 1000,
  WEEK: 7 * 24 * 60 * 60 * 1000
};

// เพิ่ม/ลด milliseconds โดยตรง
function addTime(date, ms) {
  return new Date(date.getTime() + ms);
}

const now = new Date('2024-01-15T10:00:00');

console.log(addTime(now, MS.HOUR * 2).toTimeString());  // 12:00:00
console.log(addTime(now, MS.DAY).toDateString());       // วันถัดไป
console.log(addTime(now, -MS.WEEK).toDateString());     // 1 สัปดาห์ที่แล้ว
```

```javascript
// Duration class
class Duration {
  constructor(ms) {
    this.ms = ms;
  }
  
  static fromDays(days) { return new Duration(days * 86400000); }
  static fromHours(hours) { return new Duration(hours * 3600000); }
  static fromMinutes(minutes) { return new Duration(minutes * 60000); }
  static fromSeconds(seconds) { return new Duration(seconds * 1000); }
  
  get seconds() { return Math.floor(this.ms / 1000); }
  get minutes() { return Math.floor(this.ms / 60000); }
  get hours() { return Math.floor(this.ms / 3600000); }
  get days() { return Math.floor(this.ms / 86400000); }
  
  addTo(date) { return new Date(date.getTime() + this.ms); }
  subtractFrom(date) { return new Date(date.getTime() - this.ms); }
  
  format() {
    const d = Math.floor(this.ms / 86400000);
    const h = Math.floor((this.ms % 86400000) / 3600000);
    const m = Math.floor((this.ms % 3600000) / 60000);
    const s = Math.floor((this.ms % 60000) / 1000);
    
    const parts = [];
    if (d > 0) parts.push(`${d} วัน`);
    if (h > 0) parts.push(`${h} ชั่วโมง`);
    if (m > 0) parts.push(`${m} นาที`);
    if (s > 0) parts.push(`${s} วินาที`);
    
    return parts.join(' ') || '0 วินาที';
  }
  
  static between(d1, d2) {
    return new Duration(Math.abs(d2 - d1));
  }
}

const dur = Duration.between(new Date('2024-01-15'), new Date('2024-03-20'));
console.log(dur.days);    // 65
console.log(dur.format()); // '65 วัน'

const inThreeHours = Duration.fromHours(3).addTo(new Date());
console.log(inThreeHours.toTimeString());
```

---

## Step 179: Timestamps

```javascript
// Unix timestamp (วินาที)
const unixTimestamp = Math.floor(Date.now() / 1000);
console.log(unixTimestamp);  // เช่น 1705286445

// JavaScript timestamp (milliseconds)
const jsTimestamp = Date.now();

// แปลงระหว่างกัน
const fromUnix = new Date(unixTimestamp * 1000);
const toUnix = Math.floor(someDate.getTime() / 1000);

function someDate() { return new Date(); }
```

```javascript
// Relative time (time ago)
function timeAgo(date) {
  const now = new Date();
  const diff = now - date;  // milliseconds
  
  const seconds = Math.floor(diff / 1000);
  const minutes = Math.floor(seconds / 60);
  const hours = Math.floor(minutes / 60);
  const days = Math.floor(hours / 24);
  const months = Math.floor(days / 30);
  const years = Math.floor(months / 12);
  
  if (seconds < 60) return 'เมื่อสักครู่';
  if (minutes < 60) return `${minutes} นาทีที่แล้ว`;
  if (hours < 24) return `${hours} ชั่วโมงที่แล้ว`;
  if (days < 30) return `${days} วันที่แล้ว`;
  if (months < 12) return `${months} เดือนที่แล้ว`;
  return `${years} ปีที่แล้ว`;
}

const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000);
const yesterday = new Date(Date.now() - 86400000);
const lastYear = new Date(Date.now() - 365 * 86400000);

console.log(timeAgo(fiveMinutesAgo));  // '5 นาทีที่แล้ว'
console.log(timeAgo(yesterday));        // '1 วันที่แล้ว'
console.log(timeAgo(lastYear));         // '1 ปีที่แล้ว'
```

```javascript
// countdown
function countdown(targetDate) {
  const now = new Date();
  const diff = targetDate - now;
  
  if (diff < 0) return 'เลยกำหนดแล้ว';
  
  const days = Math.floor(diff / 86400000);
  const hours = Math.floor((diff % 86400000) / 3600000);
  const minutes = Math.floor((diff % 3600000) / 60000);
  const seconds = Math.floor((diff % 60000) / 1000);
  
  return `${days} วัน ${hours} ชั่วโมง ${minutes} นาที ${seconds} วินาที`;
}

const newYear = new Date('2025-01-01T00:00:00');
console.log(`ปีใหม่ใน: ${countdown(newYear)}`);
```

---

## Step 180: Intl.DateTimeFormat

```javascript
// สร้าง formatter ที่ใช้ซ้ำได้
const thaiFormatter = new Intl.DateTimeFormat('th-TH', {
  dateStyle: 'full',
  timeStyle: 'short'
});

const engFormatter = new Intl.DateTimeFormat('en-US', {
  dateStyle: 'medium',
  timeStyle: 'medium'
});

const date = new Date('2024-01-15T10:30:00');
console.log(thaiFormatter.format(date));  // 'วันจันทร์ที่ 15 มกราคม พ.ศ. 2567 เวลา 10:30'
console.log(engFormatter.format(date));   // 'Jan 15, 2024, 10:30:00 AM'
```

```javascript
// options ละเอียด
const detailedFormatter = new Intl.DateTimeFormat('th-TH', {
  year: 'numeric',
  month: 'long',
  day: 'numeric',
  weekday: 'long',
  hour: '2-digit',
  minute: '2-digit',
  second: '2-digit',
  hour12: false,
  timeZone: 'Asia/Bangkok'
});

console.log(detailedFormatter.format(new Date()));

// formatToParts - แยกส่วนได้
const parts = detailedFormatter.formatToParts(new Date());
parts.forEach(({ type, value }) => {
  console.log(`${type}: ${value}`);
});
```

```javascript
// Relative time formatting
const relativeFormatter = new Intl.RelativeTimeFormat('th', {
  style: 'long',
  numeric: 'auto'
});

console.log(relativeFormatter.format(-1, 'day'));      // 'เมื่อวาน'
console.log(relativeFormatter.format(-3, 'day'));      // '3 วันที่แล้ว'
console.log(relativeFormatter.format(1, 'day'));       // 'พรุ่งนี้'
console.log(relativeFormatter.format(2, 'month'));     // 'ใน 2 เดือน'
console.log(relativeFormatter.format(-1, 'year'));     // 'ปีที่แล้ว'

// ใช้กับ Date objects
function relativeTime(date) {
  const formatter = new Intl.RelativeTimeFormat('th', { numeric: 'auto' });
  const diff = date - new Date();
  const diffSeconds = diff / 1000;
  
  const units = [
    { limit: 60, divisor: 1, unit: 'second' },
    { limit: 3600, divisor: 60, unit: 'minute' },
    { limit: 86400, divisor: 3600, unit: 'hour' },
    { limit: 86400 * 7, divisor: 86400, unit: 'day' },
    { limit: 86400 * 30, divisor: 86400 * 7, unit: 'week' },
    { limit: 86400 * 365, divisor: 86400 * 30, unit: 'month' },
    { limit: Infinity, divisor: 86400 * 365, unit: 'year' }
  ];
  
  for (const { limit, divisor, unit } of units) {
    if (Math.abs(diffSeconds) < limit) {
      return formatter.format(Math.round(diffSeconds / divisor), unit);
    }
  }
}

console.log(relativeTime(new Date(Date.now() + 86400000 * 2)));  // ใน 2 วัน
console.log(relativeTime(new Date(Date.now() - 3600000)));        // 1 ชั่วโมงที่แล้ว
```

---

## Step 181: Timezone Handling

```javascript
// Date เก็บเวลาเป็น UTC เสมอ แต่แสดงตาม local timezone
const date = new Date('2024-01-15T10:00:00Z');  // UTC

// Local time (ไทย UTC+7)
console.log(date.getHours());     // 17 (10 + 7 = 17)
console.log(date.getUTCHours());  // 10 (UTC)

// Timezone offset
const offset = date.getTimezoneOffset();  // -420 (minutes)
console.log(`UTC${offset > 0 ? '-' : '+'}${Math.abs(offset / 60)}`);  // UTC+7
```

```javascript
// แสดงเวลาใน timezone ต่างๆ
function formatInTimezone(date, timezone) {
  return date.toLocaleString('en-US', {
    timeZone: timezone,
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  });
}

const now = new Date();
const timezones = ['Asia/Bangkok', 'UTC', 'America/New_York', 'Europe/London', 'Asia/Tokyo'];

timezones.forEach(tz => {
  console.log(`${tz}: ${formatInTimezone(now, tz)}`);
});
```

```javascript
// แปลงระหว่าง timezone
function convertTimezone(date, fromTZ, toTZ) {
  const fromDate = new Date(date.toLocaleString('en-US', { timeZone: fromTZ }));
  const toDate = new Date(date.toLocaleString('en-US', { timeZone: toTZ }));
  const diff = date - fromDate;
  return new Date(toDate.getTime() + diff);
}
```

---

## Step 182: Calendar Calculations

```javascript
// จำนวนวันในแต่ละเดือน
function daysInMonth(year, month) {
  return new Date(year, month, 0).getDate();
}

// month เป็น 1-12 (ไม่ใช่ 0-11)
console.log(daysInMonth(2024, 1));   // 31 (มกราคม)
console.log(daysInMonth(2024, 2));   // 29 (กุมภาพันธ์ ปีอธิกสุรทิน)
console.log(daysInMonth(2023, 2));   // 28
console.log(daysInMonth(2024, 4));   // 30 (เมษายน)
```

```javascript
// ปีอธิกสุรทิน
function isLeapYear(year) {
  return (year % 4 === 0 && year % 100 !== 0) || year % 400 === 0;
}

console.log(isLeapYear(2024));  // true
console.log(isLeapYear(1900));  // false
console.log(isLeapYear(2000));  // true

// หรือใช้ Date
function isLeapYearAlt(year) {
  return new Date(year, 1, 29).getMonth() === 1;
}
```

```javascript
// วันที่ใน range
function getDatesInRange(start, end) {
  const dates = [];
  const current = new Date(start);
  
  while (current <= end) {
    dates.push(new Date(current));
    current.setDate(current.getDate() + 1);
  }
  
  return dates;
}

const startDate = new Date('2024-01-01');
const endDate = new Date('2024-01-07');
const week = getDatesInRange(startDate, endDate);
week.forEach(d => console.log(d.toDateString()));
```

```javascript
// วันทำงาน (weekdays)
function getWorkdays(start, end) {
  const dates = getDatesInRange(start, end);
  return dates.filter(d => {
    const day = d.getDay();
    return day !== 0 && day !== 6;  // ไม่ใช่ อาทิตย์ หรือ เสาร์
  });
}

// เพิ่มวันทำงาน
function addWorkdays(date, days) {
  const result = new Date(date);
  let added = 0;
  
  while (added < days) {
    result.setDate(result.getDate() + 1);
    const day = result.getDay();
    if (day !== 0 && day !== 6) added++;
  }
  
  return result;
}

const monday = new Date('2024-01-15');  // วันจันทร์
const nextFriday = addWorkdays(monday, 4);
console.log(nextFriday.toDateString());  // Friday Jan 19 2024
```

---

## Step 183: Thai Calendar (พ.ศ.)

```javascript
// แปลง ค.ศ. เป็น พ.ศ.
function adToBe(year) {
  return year + 543;
}

function beToAd(year) {
  return year - 543;
}

// แสดงวันที่แบบไทย
function formatThaiFullDate(date) {
  const thaiDayNames = ['อาทิตย์', 'จันทร์', 'อังคาร', 'พุธ', 'พฤหัสบดี', 'ศุกร์', 'เสาร์'];
  const thaiMonthNames = [
    'มกราคม', 'กุมภาพันธ์', 'มีนาคม', 'เมษายน',
    'พฤษภาคม', 'มิถุนายน', 'กรกฎาคม', 'สิงหาคม',
    'กันยายน', 'ตุลาคม', 'พฤศจิกายน', 'ธันวาคม'
  ];
  
  const dayName = thaiDayNames[date.getDay()];
  const day = date.getDate();
  const month = thaiMonthNames[date.getMonth()];
  const year = adToBe(date.getFullYear());
  
  return `${dayName}ที่ ${day} ${month} พ.ศ. ${year}`;
}

const nationalDay = new Date('2024-12-05');
console.log(formatThaiFullDate(nationalDay));
// 'พฤหัสบดีที่ 5 ธันวาคม พ.ศ. 2567'
```

```javascript
// Intl สำหรับปฏิทินไทย
const thaiCalendarFormatter = new Intl.DateTimeFormat('th-TH-u-ca-buddhist', {
  year: 'numeric',
  month: 'long',
  day: 'numeric',
  weekday: 'long'
});

const date = new Date('2024-01-15');
console.log(thaiCalendarFormatter.format(date));
// 'วันจันทร์ที่ 15 มกราคม พ.ศ. 2567'

// เฉพาะปี
const yearFormatter = new Intl.DateTimeFormat('th-TH-u-ca-buddhist', { year: 'numeric' });
console.log(yearFormatter.format(new Date()));  // '2567'
```

---

## Step 184: Date Parsing

```javascript
// parse ISO 8601
function parseISO(str) {
  return new Date(str);
}

// parse รูปแบบไทย (dd/mm/yyyy)
function parseThaiDate(str) {
  const [day, month, year] = str.split('/').map(Number);
  // ปรับปี ถ้าเป็น พ.ศ.
  const adYear = year > 2500 ? year - 543 : year;
  return new Date(adYear, month - 1, day);
}

console.log(parseThaiDate('15/1/2567'));  // Jan 15 2024
console.log(parseThaiDate('20/5/1990')); // May 20 1990

// parse หลาย format
function smartParse(str) {
  // ISO format
  if (/^\d{4}-\d{2}-\d{2}/.test(str)) {
    return new Date(str);
  }
  
  // Thai format dd/mm/yyyy
  if (/^\d{1,2}\/\d{1,2}\/\d{4}$/.test(str)) {
    return parseThaiDate(str);
  }
  
  // ลอง parse ทั่วไป
  const date = new Date(str);
  return isNaN(date) ? null : date;
}
```

---

## Step 185: Date Validation

```javascript
function isValidDate(year, month, day) {
  const date = new Date(year, month - 1, day);
  return date.getFullYear() === year &&
         date.getMonth() === month - 1 &&
         date.getDate() === day;
}

console.log(isValidDate(2024, 1, 31));   // true
console.log(isValidDate(2024, 2, 29));   // true (ปีอธิกสุรทิน)
console.log(isValidDate(2023, 2, 29));   // false
console.log(isValidDate(2024, 4, 31));   // false (เมษาแค่ 30 วัน)
```

```javascript
// ตรวจสอบ date string
function isValidDateString(str) {
  const date = new Date(str);
  return !isNaN(date.getTime());
}

console.log(isValidDateString('2024-01-15'));  // true
console.log(isValidDateString('2024-13-01'));  // false
console.log(isValidDateString('invalid'));     // false

// form validation
function validateDateForm(day, month, year) {
  const errors = {};
  
  if (!Number.isInteger(Number(day)) || day < 1 || day > 31) {
    errors.day = 'วันที่ไม่ถูกต้อง (1-31)';
  }
  
  if (!Number.isInteger(Number(month)) || month < 1 || month > 12) {
    errors.month = 'เดือนไม่ถูกต้อง (1-12)';
  }
  
  if (!Number.isInteger(Number(year)) || year < 1900 || year > 2100) {
    errors.year = 'ปีไม่ถูกต้อง';
  }
  
  if (!errors.day && !errors.month && !errors.year) {
    if (!isValidDate(Number(year), Number(month), Number(day))) {
      errors.date = 'วันที่ไม่มีในปฏิทิน';
    }
  }
  
  return errors;
}

console.log(validateDateForm(29, 2, 2023));  // { date: 'วันที่ไม่มีในปฏิทิน' }
console.log(validateDateForm(15, 1, 2024));  // {}
```

---

## Step 186: Working Hours Calculator

```javascript
class WorkingHoursCalculator {
  constructor(workStart = '09:00', workEnd = '18:00', holidays = []) {
    this.workStart = workStart;
    this.workEnd = workEnd;
    this.holidays = new Set(holidays.map(d => new Date(d).toDateString()));
  }
  
  isWorkday(date) {
    const day = date.getDay();
    return day !== 0 && day !== 6 && !this.holidays.has(date.toDateString());
  }
  
  getWorkHours(date) {
    if (!this.isWorkday(date)) return 0;
    
    const [startH, startM] = this.workStart.split(':').map(Number);
    const [endH, endM] = this.workEnd.split(':').map(Number);
    
    return (endH + endM/60) - (startH + startM/60);
  }
  
  calculateHours(startDate, endDate) {
    let totalHours = 0;
    const current = new Date(startDate);
    current.setHours(0, 0, 0, 0);
    
    while (current <= endDate) {
      totalHours += this.getWorkHours(current);
      current.setDate(current.getDate() + 1);
    }
    
    return totalHours;
  }
  
  getDeadline(startDate, hours) {
    const result = new Date(startDate);
    let remaining = hours;
    
    while (remaining > 0) {
      if (this.isWorkday(result)) {
        const dailyHours = this.getWorkHours(result);
        remaining -= dailyHours;
      }
      if (remaining > 0) result.setDate(result.getDate() + 1);
    }
    
    return result;
  }
}

const calc = new WorkingHoursCalculator('09:00', '18:00', ['2024-01-01']);

const start = new Date('2024-01-15');
const end = new Date('2024-01-19');  // จันทร์ - ศุกร์

console.log(`ชั่วโมงทำงาน: ${calc.calculateHours(start, end)} ชั่วโมง`);  // 45

const deadline = calc.getDeadline(start, 20);
console.log(`Deadline: ${deadline.toDateString()}`);
```

---

## Step 187: Event Scheduler

```javascript
class EventScheduler {
  #events = [];
  
  addEvent(title, date, duration = 60, recurring = null) {
    this.#events.push({
      id: Date.now(),
      title,
      date: new Date(date),
      duration,  // minutes
      recurring  // 'daily' | 'weekly' | 'monthly' | null
    });
    return this;
  }
  
  getEventsOnDate(targetDate) {
    const target = new Date(targetDate);
    target.setHours(0, 0, 0, 0);
    const nextDay = new Date(target);
    nextDay.setDate(nextDay.getDate() + 1);
    
    return this.#events.filter(event => {
      const eventDate = new Date(event.date);
      
      // Check direct match
      if (eventDate >= target && eventDate < nextDay) return true;
      
      // Check recurring
      if (event.recurring === 'daily') return eventDate <= target;
      if (event.recurring === 'weekly') {
        return eventDate <= target && 
               event.date.getDay() === target.getDay();
      }
      if (event.recurring === 'monthly') {
        return eventDate <= target && 
               event.date.getDate() === target.getDate();
      }
      
      return false;
    }).sort((a, b) => a.date - b.date);
  }
  
  getUpcoming(days = 7) {
    const now = new Date();
    const end = new Date();
    end.setDate(end.getDate() + days);
    
    const upcoming = [];
    const current = new Date(now);
    
    while (current <= end) {
      const events = this.getEventsOnDate(current);
      events.forEach(e => upcoming.push({ ...e, scheduledDate: new Date(current) }));
      current.setDate(current.getDate() + 1);
    }
    
    return upcoming;
  }
  
  removeEvent(id) {
    this.#events = this.#events.filter(e => e.id !== id);
    return this;
  }
}

const scheduler = new EventScheduler();

scheduler
  .addEvent('ประชุมทีม', '2024-01-15T09:00:00', 60, 'weekly')
  .addEvent('กินข้าวเที่ยง', '2024-01-15T12:00:00', 60, 'daily')
  .addEvent('ส่งรายงาน', '2024-01-19T17:00:00', 30);

const monday = new Date('2024-01-15');
const events = scheduler.getEventsOnDate(monday);
console.log('Events on Monday:');
events.forEach(e => {
  console.log(`  ${e.date.toLocaleTimeString('th-TH')} - ${e.title}`);
});
```

---

## Step 188: Date Range ต่างๆ

```javascript
// วันแรก/สุดท้ายของเดือน
function startOfMonth(date) {
  return new Date(date.getFullYear(), date.getMonth(), 1);
}

function endOfMonth(date) {
  return new Date(date.getFullYear(), date.getMonth() + 1, 0);
}

// วันแรก/สุดท้ายของสัปดาห์ (เริ่มจันทร์)
function startOfWeek(date) {
  const d = new Date(date);
  const day = d.getDay();
  const diff = day === 0 ? -6 : 1 - day;  // จันทร์ = 1
  d.setDate(d.getDate() + diff);
  d.setHours(0, 0, 0, 0);
  return d;
}

function endOfWeek(date) {
  const start = startOfWeek(date);
  const end = new Date(start);
  end.setDate(end.getDate() + 6);
  end.setHours(23, 59, 59, 999);
  return end;
}

// วันแรก/สุดท้ายของปี
function startOfYear(date) {
  return new Date(date.getFullYear(), 0, 1);
}

function endOfYear(date) {
  return new Date(date.getFullYear(), 11, 31);
}

const today = new Date('2024-01-15');
console.log('เริ่มต้นเดือน:', startOfMonth(today).toDateString());
console.log('สิ้นเดือน:', endOfMonth(today).toDateString());
console.log('เริ่มต้นสัปดาห์:', startOfWeek(today).toDateString());
console.log('สิ้นสัปดาห์:', endOfWeek(today).toDateString());
```

---

## Step 189: Date Utilities Library

```javascript
// Mini date utility library
const DateUtils = {
  // Formatting
  format(date, pattern) {
    const d = new Date(date);
    const map = {
      'YYYY': d.getFullYear(),
      'YY': String(d.getFullYear()).slice(-2),
      'MM': String(d.getMonth() + 1).padStart(2, '0'),
      'M': d.getMonth() + 1,
      'DD': String(d.getDate()).padStart(2, '0'),
      'D': d.getDate(),
      'HH': String(d.getHours()).padStart(2, '0'),
      'H': d.getHours(),
      'mm': String(d.getMinutes()).padStart(2, '0'),
      'ss': String(d.getSeconds()).padStart(2, '0')
    };
    
    return pattern.replace(/YYYY|YY|MM|M|DD|D|HH|H|mm|ss/g, match => map[match]);
  },
  
  // Parsing
  parse(str, format) {
    // Simple implementation
    const parts = {};
    const regexMap = {
      'YYYY': '(\\d{4})',
      'MM': '(\\d{2})',
      'DD': '(\\d{2})',
      'HH': '(\\d{2})',
      'mm': '(\\d{2})',
      'ss': '(\\d{2})'
    };
    
    let regex = format;
    const keys = [];
    
    ['YYYY', 'MM', 'DD', 'HH', 'mm', 'ss'].forEach(key => {
      if (format.includes(key)) {
        regex = regex.replace(key, regexMap[key]);
        keys.push(key);
      }
    });
    
    const match = str.match(new RegExp('^' + regex + '$'));
    if (!match) return null;
    
    keys.forEach((key, i) => parts[key] = parseInt(match[i + 1]));
    
    return new Date(
      parts.YYYY || 2000,
      (parts.MM || 1) - 1,
      parts.DD || 1,
      parts.HH || 0,
      parts.mm || 0,
      parts.ss || 0
    );
  },
  
  // Comparison
  isSameDay: (d1, d2) => DateUtils.format(d1, 'YYYY-MM-DD') === DateUtils.format(d2, 'YYYY-MM-DD'),
  isBefore: (d1, d2) => new Date(d1) < new Date(d2),
  isAfter: (d1, d2) => new Date(d1) > new Date(d2),
  
  // Manipulation
  add(date, value, unit) {
    const d = new Date(date);
    const units = {
      day: () => d.setDate(d.getDate() + value),
      month: () => d.setMonth(d.getMonth() + value),
      year: () => d.setFullYear(d.getFullYear() + value),
      hour: () => d.setHours(d.getHours() + value),
      minute: () => d.setMinutes(d.getMinutes() + value)
    };
    units[unit]?.();
    return d;
  },
  
  diff(d1, d2, unit = 'day') {
    const diff = new Date(d2) - new Date(d1);
    const divisors = {
      ms: 1,
      second: 1000,
      minute: 60000,
      hour: 3600000,
      day: 86400000
    };
    return Math.floor(diff / (divisors[unit] || 86400000));
  }
};

// ใช้งาน
const now = new Date('2024-01-15T10:30:45');
console.log(DateUtils.format(now, 'DD/MM/YYYY'));          // '15/01/2024'
console.log(DateUtils.format(now, 'YYYY-MM-DD HH:mm:ss')); // '2024-01-15 10:30:45'

const parsed = DateUtils.parse('15/01/2024', 'DD/MM/YYYY');
console.log(parsed.toDateString());  // 'Mon Jan 15 2024'

const futureDate = DateUtils.add(now, 30, 'day');
console.log(DateUtils.diff(now, futureDate));  // 30
```

---

## Step 190: ตัวอย่างโปรเจกต์ - Meeting Planner

```javascript
class MeetingPlanner {
  #meetings = [];
  #workHours = { start: 9, end: 18 };
  
  scheduleMeeting(title, startDate, durationMinutes, attendees = []) {
    const start = new Date(startDate);
    const end = new Date(start.getTime() + durationMinutes * 60000);
    
    // ตรวจสอบ conflict
    const conflicts = this.#checkConflicts(start, end);
    if (conflicts.length > 0) {
      return {
        success: false,
        conflicts,
        suggestions: this.#suggestAlternatives(start, durationMinutes)
      };
    }
    
    // ตรวจสอบว่าอยู่ในเวลาทำงาน
    if (!this.#isWithinWorkHours(start, end)) {
      return {
        success: false,
        error: 'เวลานอกเหนือจากชั่วโมงทำงาน',
        workHours: `${this.#workHours.start}:00 - ${this.#workHours.end}:00`
      };
    }
    
    const meeting = {
      id: Date.now(),
      title,
      start,
      end,
      duration: durationMinutes,
      attendees,
      createdAt: new Date()
    };
    
    this.#meetings.push(meeting);
    return { success: true, meeting };
  }
  
  #checkConflicts(start, end) {
    return this.#meetings.filter(m => 
      (start >= m.start && start < m.end) ||
      (end > m.start && end <= m.end) ||
      (start <= m.start && end >= m.end)
    );
  }
  
  #isWithinWorkHours(start, end) {
    const startH = start.getHours() + start.getMinutes() / 60;
    const endH = end.getHours() + end.getMinutes() / 60;
    return startH >= this.#workHours.start && endH <= this.#workHours.end;
  }
  
  #suggestAlternatives(date, duration) {
    const suggestions = [];
    const workStart = new Date(date);
    workStart.setHours(this.#workHours.start, 0, 0, 0);
    
    // หาช่วงว่างในวันนั้น
    const dayMeetings = this.getMeetingsOnDate(date)
      .sort((a, b) => a.start - b.start);
    
    let lastEnd = new Date(workStart);
    
    for (const m of dayMeetings) {
      const gapMs = m.start - lastEnd;
      if (gapMs >= duration * 60000) {
        suggestions.push(new Date(lastEnd));
      }
      lastEnd = new Date(m.end);
    }
    
    // หลังประชุมสุดท้าย
    const workEnd = new Date(date);
    workEnd.setHours(this.#workHours.end, 0, 0, 0);
    if (workEnd - lastEnd >= duration * 60000) {
      suggestions.push(new Date(lastEnd));
    }
    
    return suggestions.slice(0, 3);  // แนะนำ 3 ช่วง
  }
  
  getMeetingsOnDate(date) {
    const target = new Date(date);
    target.setHours(0, 0, 0, 0);
    const nextDay = new Date(target);
    nextDay.setDate(nextDay.getDate() + 1);
    
    return this.#meetings.filter(m => m.start >= target && m.start < nextDay);
  }
  
  cancelMeeting(id) {
    this.#meetings = this.#meetings.filter(m => m.id !== id);
  }
  
  getSchedule(startDate, endDate) {
    return this.#meetings
      .filter(m => m.start >= startDate && m.start <= endDate)
      .sort((a, b) => a.start - b.start);
  }
}

// ใช้งาน
const planner = new MeetingPlanner();

const r1 = planner.scheduleMeeting(
  'Sprint Planning',
  '2024-01-15T09:00:00',
  120,
  ['สมชาย', 'สมหญิง']
);
console.log(r1.success ? 'นัดได้!' : 'มีปัญหา');

const r2 = planner.scheduleMeeting(
  'Code Review',
  '2024-01-15T10:00:00',  // ซ้อนทับ Sprint Planning!
  60,
  ['สมศักดิ์']
);

if (!r2.success) {
  console.log('ไม่สามารถนัดได้ เนื่องจากมีการประชุมซ้อน');
  console.log('เวลาที่แนะนำ:');
  r2.suggestions.forEach(s => {
    console.log(`  - ${s.toLocaleTimeString('th-TH')}`);
  });
}
```

---

## แบบฝึกหัด (Exercises)

### ระดับง่าย

**แบบฝึกหัด 1:** เขียน function `formatDate(date, format)` ที่ format วันที่ตามรูปแบบที่กำหนด เช่น 'DD/MM/YYYY'

**แบบฝึกหัด 2:** เขียน function `calculateAge(birthdate)` ที่คำนวณอายุจากวันเกิด

**แบบฝึกหัด 3:** เขียน function `isWeekend(date)` ตรวจสอบว่าเป็นวันหยุดสุดสัปดาห์

**แบบฝึกหัด 4:** เขียน function `nextMonday(date)` ที่คืนวันจันทร์ถัดไป

**แบบฝึกหัด 5:** เขียน function `quarterOf(date)` ที่บอกว่าวันนั้นอยู่ใน Q ที่เท่าไหร่ (Q1-Q4)

### ระดับกลาง

**แบบฝึกหัด 6:** สร้าง Calendar component ที่แสดงตารางเดือน

**แบบฝึกหัด 7:** เขียน function `businessDaysBetween(start, end, holidays=[])` นับวันทำงาน

**แบบฝึกหัด 8:** สร้าง Timer class ที่รองรับ start, stop, pause, resume, lap

**แบบฝึกหัด 9:** เขียน function `parseRelativeDate(str)` เช่น 'yesterday', 'next week', '3 days ago'

### ระดับยาก

**แบบฝึกหัด 10:** สร้าง Recurring Event ที่รองรับ RRULE (iCal format)

### เฉลยบางส่วน

```javascript
// แบบฝึกหัด 1
function formatDate(date, format) {
  const d = new Date(date);
  return format
    .replace('YYYY', d.getFullYear())
    .replace('MM', String(d.getMonth() + 1).padStart(2, '0'))
    .replace('DD', String(d.getDate()).padStart(2, '0'))
    .replace('HH', String(d.getHours()).padStart(2, '0'))
    .replace('mm', String(d.getMinutes()).padStart(2, '0'))
    .replace('ss', String(d.getSeconds()).padStart(2, '0'));
}

console.log(formatDate(new Date('2024-01-15'), 'DD/MM/YYYY'));  // '15/01/2024'

// แบบฝึกหัด 3
function isWeekend(date) {
  const day = new Date(date).getDay();
  return day === 0 || day === 6;
}

// แบบฝึกหัด 4
function nextMonday(date) {
  const d = new Date(date);
  const day = d.getDay();
  const daysUntilMonday = day === 0 ? 1 : 8 - day;
  d.setDate(d.getDate() + daysUntilMonday);
  return d;
}

// แบบฝึกหัด 5
function quarterOf(date) {
  return Math.floor(new Date(date).getMonth() / 3) + 1;
}

console.log(quarterOf(new Date('2024-01-15')));   // Q1
console.log(quarterOf(new Date('2024-07-15')));   // Q3

// แบบฝึกหัด 7
function businessDaysBetween(start, end, holidays = []) {
  const holidaySet = new Set(holidays.map(h => new Date(h).toDateString()));
  let count = 0;
  const current = new Date(start);
  
  while (current <= end) {
    const day = current.getDay();
    if (day !== 0 && day !== 6 && !holidaySet.has(current.toDateString())) {
      count++;
    }
    current.setDate(current.getDate() + 1);
  }
  
  return count;
}

// แบบฝึกหัด 8 - Timer
class Timer {
  #startTime = null;
  #elapsed = 0;
  #running = false;
  #laps = [];
  
  start() {
    if (!this.#running) {
      this.#startTime = Date.now() - this.#elapsed;
      this.#running = true;
    }
    return this;
  }
  
  stop() {
    if (this.#running) {
      this.#elapsed = Date.now() - this.#startTime;
      this.#running = false;
    }
    return this;
  }
  
  pause() { return this.stop(); }
  
  resume() { return this.start(); }
  
  reset() {
    this.#startTime = null;
    this.#elapsed = 0;
    this.#running = false;
    this.#laps = [];
    return this;
  }
  
  lap() {
    this.#laps.push(this.getElapsed());
    return this;
  }
  
  getElapsed() {
    if (this.#running) return Date.now() - this.#startTime;
    return this.#elapsed;
  }
  
  format() {
    const ms = this.getElapsed();
    const m = Math.floor(ms / 60000);
    const s = Math.floor((ms % 60000) / 1000);
    const centiseconds = Math.floor((ms % 1000) / 10);
    return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}.${String(centiseconds).padStart(2, '0')}`;
  }
  
  getLaps() { return [...this.#laps]; }
}

const timer = new Timer();
timer.start();
// หลังจาก 1 วินาที...
// timer.lap();
// timer.stop();
console.log(timer.format());
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- การสร้าง Date objects ด้วยวิธีต่างๆ
- Date.now() และ timestamps
- Getting date parts: getFullYear, getMonth, getDate, getDay
- Getting time parts: getHours, getMinutes, getSeconds, getMilliseconds
- Setting date parts ด้วย set methods
- การเปรียบเทียบวันที่
- Date formatting ด้วย toLocaleDateString, toLocaleTimeString
- Date arithmetic: เพิ่ม/ลดวัน เดือน ปี
- Intl.DateTimeFormat และ Intl.RelativeTimeFormat
- Timezone handling
- Calendar calculations: leap year, days in month
- Thai calendar (พ.ศ.)
- Practical applications: Timer, Scheduler, Calendar

การทำงานกับ Date ใน JavaScript มีความซับซ้อน โดยเฉพาะเรื่อง timezone และ month ที่เริ่มจาก 0 การใช้ library อย่าง date-fns หรือ Day.js สำหรับโปรเจกต์ใหญ่จะช่วยลดข้อผิดพลาดได้มาก
