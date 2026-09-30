# Part 13: Forms และ Input Validation (Steps 231-250)

## บทนำ

Forms คือส่วนสำคัญที่สุดส่วนหนึ่งของเว็บแอปพลิเคชัน การ validate ข้อมูลบน client-side ช่วยให้ผู้ใช้ได้รับ feedback ทันที ก่อนส่งข้อมูลไปยัง server ทำให้ประสบการณ์ใช้งานดีขึ้นมาก

---

## Step 231: HTML Form Elements ใน JavaScript

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <form id="demo-form" name="demoForm" action="/submit" method="POST">
    <input type="text" id="username" name="username" value="John">
    <input type="email" id="email" name="email">
    <input type="password" id="password" name="password">
    <input type="number" id="age" name="age" min="0" max="120">
    <input type="checkbox" id="agree" name="agree" value="yes">
    <input type="radio" name="gender" value="male" id="male">
    <input type="radio" name="gender" value="female" id="female">
    <select id="country" name="country">
      <option value="th">ไทย</option>
      <option value="jp">ญี่ปุ่น</option>
    </select>
    <textarea id="bio" name="bio"></textarea>
    <input type="file" id="avatar" name="avatar">
    <button type="submit">Submit</button>
    <button type="reset">Reset</button>
    <button type="button" id="custom-btn">Custom</button>
  </form>

  <script>
    // เข้าถึง form
    const form = document.getElementById('demo-form');
    const formByName = document.forms['demoForm']; // by name
    const firstForm = document.forms[0]; // by index

    console.log(form === formByName); // true

    // Form properties
    console.log(form.action);   // URL action
    console.log(form.method);   // "post"
    console.log(form.name);     // "demoForm"
    console.log(form.elements); // HTMLFormControlsCollection
    console.log(form.length);   // จำนวน controls

    // เข้าถึง elements ใน form
    console.log(form.elements['username']); // input#username
    console.log(form.elements.email);       // input#email
    console.log(form.elements[0]);          // first element

    // เช่นเดียวกันกับ form.username (shortcut)
    console.log(form.username); // input#username

    // Form validation properties
    console.log(form.noValidate); // false (browser validation เปิดอยู่)
    console.log(form.checkValidity()); // true/false
    console.log(form.reportValidity()); // แสดง validation UI

    // ป้องกัน browser default validation
    form.noValidate = true; // หรือ <form novalidate>

    // Form elements
    Array.from(form.elements).forEach(el => {
      console.log(`${el.tagName} [${el.type}]: name=${el.name}, value=${el.value}`);
    });
  </script>
</body>
</html>
```

---

## Step 232: Getting และ Setting Values

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <form id="form">
    <!-- Text inputs -->
    <input type="text" id="name" value="สมชาย">
    <input type="number" id="age" value="25">
    <input type="email" id="email" value="test@example.com">
    <input type="password" id="pass" value="secret123">
    <input type="date" id="birthday" value="1999-05-15">
    <input type="range" id="volume" min="0" max="100" value="50">
    <input type="color" id="color" value="#ff0000">

    <!-- Checkbox -->
    <input type="checkbox" id="agree" name="agree" checked>
    <input type="checkbox" id="newsletter" name="newsletter">

    <!-- Radio -->
    <input type="radio" name="size" value="s" id="size-s">
    <input type="radio" name="size" value="m" id="size-m" checked>
    <input type="radio" name="size" value="l" id="size-l">

    <!-- Select -->
    <select id="city">
      <option value="">-- เลือก --</option>
      <option value="bkk" selected>กรุงเทพ</option>
      <option value="cmx">เชียงใหม่</option>
      <option value="pkt">ภูเก็ต</option>
    </select>

    <!-- Multiple select -->
    <select id="skills" multiple>
      <option value="js" selected>JavaScript</option>
      <option value="py">Python</option>
      <option value="go" selected>Go</option>
    </select>

    <!-- Textarea -->
    <textarea id="bio">นักพัฒนา Frontend</textarea>
  </form>

  <script>
    // Text inputs - ใช้ .value
    const nameInput = document.getElementById('name');
    console.log(nameInput.value); // "สมชาย"
    nameInput.value = 'สมหญิง';  // ตั้งค่า

    // Number input
    const ageInput = document.getElementById('age');
    console.log(ageInput.value);          // "25" (string!)
    console.log(Number(ageInput.value));  // 25 (number)
    console.log(ageInput.valueAsNumber);  // 25 (number, shortcut)

    // Date input
    const birthday = document.getElementById('birthday');
    console.log(birthday.value);            // "1999-05-15" (string)
    console.log(birthday.valueAsDate);      // Date object
    birthday.valueAsDate = new Date('2000-01-01');

    // Range
    const volume = document.getElementById('volume');
    console.log(volume.value);         // "50"
    console.log(volume.valueAsNumber); // 50

    // Color
    const color = document.getElementById('color');
    console.log(color.value); // "#ff0000"

    // Checkbox - ใช้ .checked
    const agree = document.getElementById('agree');
    console.log(agree.checked); // true
    console.log(agree.value);   // "agree" (HTML value attribute)
    agree.checked = false;

    // ดึงค่า checkboxes ทั้งหมดที่ checked
    function getCheckedValues(name) {
      return Array.from(document.querySelectorAll(`input[name="${name}"]:checked`))
        .map(el => el.value);
    }

    // Radio - ดึง selected value
    function getRadioValue(name) {
      const checked = document.querySelector(`input[name="${name}"]:checked`);
      return checked ? checked.value : null;
    }
    console.log(getRadioValue('size')); // "m"

    // ตั้งค่า radio
    document.getElementById('size-l').checked = true;
    console.log(getRadioValue('size')); // "l"

    // Select - .value และ .selectedIndex
    const city = document.getElementById('city');
    console.log(city.value);         // "bkk"
    console.log(city.selectedIndex); // 1 (index)
    console.log(city.options[city.selectedIndex].text); // "กรุงเทพ"

    city.value = 'cmx'; // เลือก option อื่น

    // Multiple select - ดึงค่าทั้งหมด
    const skills = document.getElementById('skills');
    const selectedSkills = Array.from(skills.selectedOptions).map(opt => opt.value);
    console.log(selectedSkills); // ["js", "go"]

    // textarea
    const bio = document.getElementById('bio');
    console.log(bio.value); // "นักพัฒนา Frontend"
    bio.value = 'Full Stack Developer';
  </script>
</body>
</html>
```

---

## Step 233: Form Submission Handling

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <form id="login-form" novalidate>
    <div class="form-group">
      <label for="username">Username:</label>
      <input type="text" id="username" name="username" required>
      <span class="error" id="username-error"></span>
    </div>
    <div class="form-group">
      <label for="password">Password:</label>
      <input type="password" id="password" name="password" required minlength="8">
      <span class="error" id="password-error"></span>
    </div>
    <button type="submit" id="submit-btn">เข้าสู่ระบบ</button>
    <div id="message"></div>
  </form>

  <script>
    const form = document.getElementById('login-form');
    const submitBtn = document.getElementById('submit-btn');
    const message = document.getElementById('message');

    function showError(fieldId, msg) {
      document.getElementById(`${fieldId}-error`).textContent = msg;
      document.getElementById(fieldId).style.borderColor = 'red';
    }

    function clearError(fieldId) {
      document.getElementById(`${fieldId}-error`).textContent = '';
      document.getElementById(fieldId).style.borderColor = '';
    }

    function validate() {
      let isValid = true;
      const username = document.getElementById('username').value.trim();
      const password = document.getElementById('password').value;

      clearError('username');
      clearError('password');

      if (!username) {
        showError('username', 'กรุณากรอก username');
        isValid = false;
      } else if (username.length < 3) {
        showError('username', 'Username ต้องมีอย่างน้อย 3 ตัวอักษร');
        isValid = false;
      }

      if (!password) {
        showError('password', 'กรุณากรอก password');
        isValid = false;
      } else if (password.length < 8) {
        showError('password', 'Password ต้องมีอย่างน้อย 8 ตัวอักษร');
        isValid = false;
      }

      return isValid;
    }

    form.addEventListener('submit', async (e) => {
      e.preventDefault(); // ป้องกัน default form submission

      if (!validate()) return;

      // Disable button เพื่อป้องกัน double submit
      submitBtn.disabled = true;
      submitBtn.textContent = 'กำลังเข้าสู่ระบบ...';
      message.textContent = '';

      try {
        // Simulate API call
        const formData = new FormData(form);
        const data = Object.fromEntries(formData);

        await new Promise(resolve => setTimeout(resolve, 1500));

        // Simulate success/failure
        if (data.username === 'admin' && data.password === 'password123') {
          message.textContent = 'เข้าสู่ระบบสำเร็จ!';
          message.style.color = 'green';
          form.reset();
        } else {
          throw new Error('ข้อมูลไม่ถูกต้อง');
        }
      } catch (error) {
        message.textContent = error.message;
        message.style.color = 'red';
      } finally {
        submitBtn.disabled = false;
        submitBtn.textContent = 'เข้าสู่ระบบ';
      }
    });

    // Real-time validation
    document.getElementById('username').addEventListener('input', () => clearError('username'));
    document.getElementById('password').addEventListener('input', () => clearError('password'));

    // Programmatic form submission
    // form.submit(); // ไม่ trigger submit event!
    // form.requestSubmit(); // trigger submit event (modern)
  </script>
</body>
</html>
```

---

## Step 234: Client-Side Validation Concepts

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <form id="form" novalidate>
    <input type="text" id="name" name="name" required minlength="2" maxlength="50">
    <input type="email" id="email" name="email" required>
    <input type="number" id="age" name="age" min="18" max="100">
    <input type="url" id="website" name="website">
    <input type="text" id="phone" name="phone" pattern="[0-9]{10}">
    <button type="submit">Submit</button>
  </form>

  <script>
    // HTML5 Constraint Validation API
    const nameInput = document.getElementById('name');
    const emailInput = document.getElementById('email');

    // validity object - ValidityState
    console.log(nameInput.validity);
    // {
    //   valueMissing: false,    // required แต่ว่าง
    //   tooShort: false,        // ต่ำกว่า minlength
    //   tooLong: false,         // เกิน maxlength
    //   typeMismatch: false,    // type ไม่ถูกต้อง (email, url)
    //   patternMismatch: false, // ไม่ match pattern
    //   rangeUnderflow: false,  // น้อยกว่า min
    //   rangeOverflow: false,   // เกิน max
    //   stepMismatch: false,    // ไม่ match step
    //   badInput: false,        // browser ไม่สามารถ convert ได้
    //   customError: false,     // setCustomValidity ตั้งค่า error
    //   valid: true             // ทุกอย่างถูกต้อง
    // }

    // checkValidity() - คืน boolean
    console.log(nameInput.checkValidity()); // true/false

    // validationMessage - message สำหรับ show ให้ user
    console.log(nameInput.validationMessage); // "" ถ้า valid

    // ทดสอบ validity states
    nameInput.value = '';
    console.log(nameInput.validity.valueMissing); // true
    console.log(nameInput.validationMessage);      // "โปรดกรอกข้อมูลในช่องนี้"

    nameInput.value = 'a';
    console.log(nameInput.validity.tooShort);  // true
    console.log(nameInput.validity.valid);     // false

    nameInput.value = 'John';
    console.log(nameInput.validity.valid);     // true

    emailInput.value = 'invalid-email';
    console.log(emailInput.validity.typeMismatch); // true

    // setCustomValidity - ตั้ง custom error message
    function validateUsername(input) {
      const value = input.value.trim();
      const reserved = ['admin', 'root', 'system'];

      if (reserved.includes(value.toLowerCase())) {
        input.setCustomValidity(`ชื่อ "${value}" ถูกสงวนไว้ ใช้ชื่ออื่นได้เลย`);
      } else {
        input.setCustomValidity(''); // clear error
      }
    }

    nameInput.addEventListener('input', () => validateUsername(nameInput));

    // Custom validation styles
    const style = document.createElement('style');
    style.textContent = `
      input:valid { border: 2px solid green; }
      input:invalid { border: 2px solid red; }
      input:placeholder-shown { border: 1px solid #ccc; } /* ยังไม่ได้แตะ */
    `;
    document.head.appendChild(style);

    // Validate all form fields
    function validateForm(form) {
      const errors = {};
      Array.from(form.elements).forEach(el => {
        if (el.name && !el.checkValidity()) {
          errors[el.name] = el.validationMessage;
        }
      });
      return {
        isValid: Object.keys(errors).length === 0,
        errors
      };
    }

    document.getElementById('form').addEventListener('submit', (e) => {
      e.preventDefault();
      const { isValid, errors } = validateForm(e.target);
      if (isValid) {
        console.log('Form is valid!');
      } else {
        console.log('Validation errors:', errors);
      }
    });
  </script>
</body>
</html>
```

---

## Step 235: Validating Required Fields

```javascript
// Helper functions สำหรับ validate required fields

function isEmpty(value) {
  if (typeof value === 'string') return value.trim() === '';
  if (Array.isArray(value)) return value.length === 0;
  return value === null || value === undefined;
}

function validateRequired(value, fieldName = 'ฟิลด์นี้') {
  if (isEmpty(value)) {
    return `${fieldName} จำเป็นต้องกรอก`;
  }
  return null;
}

// ตัวอย่างการใช้งาน
const formFields = {
  name: 'สมชาย',
  email: '',
  phone: '0812345678',
  bio: '   ',  // whitespace only
};

Object.entries(formFields).forEach(([field, value]) => {
  const error = validateRequired(value, field);
  if (error) console.log('Error:', error);
});

// Validate form ทั้งหมด
function validateRequiredFields(formData, requiredFields) {
  const errors = {};
  requiredFields.forEach(field => {
    const value = formData[field.name];
    if (isEmpty(value)) {
      errors[field.name] = field.message || `${field.label} จำเป็นต้องกรอก`;
    }
  });
  return errors;
}

const data = { username: '', email: 'test@test.com', password: '' };
const required = [
  { name: 'username', label: 'ชื่อผู้ใช้' },
  { name: 'email', label: 'Email' },
  { name: 'password', label: 'รหัสผ่าน', message: 'กรุณาตั้งรหัสผ่าน' },
];

const errors = validateRequiredFields(data, required);
console.log(errors);
// { username: "ชื่อผู้ใช้ จำเป็นต้องกรอก", password: "กรุณาตั้งรหัสผ่าน" }
```

---

## Step 236: Validating Email Format

```javascript
// Email validation ด้วย regex

// Basic email regex
const basicEmailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

// More strict email regex (RFC 5322 simplified)
const emailRegex = /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*\.[a-zA-Z]{2,}$/;

function validateEmail(email) {
  const trimmed = email.trim();
  if (!trimmed) return 'กรุณากรอก email';
  if (!emailRegex.test(trimmed)) return 'รูปแบบ email ไม่ถูกต้อง';
  if (trimmed.length > 254) return 'Email ยาวเกินไป';
  return null;
}

// ทดสอบ
const emails = [
  'user@example.com',     // valid
  'user.name+tag@example.co.th', // valid
  'invalid-email',        // invalid
  'missing@domain',       // invalid
  '@nodomain.com',        // invalid
  'spaces in@email.com',  // invalid
  '',                     // empty
];

emails.forEach(email => {
  const error = validateEmail(email);
  console.log(`"${email}": ${error || 'valid'}`);
});

// ตรวจสอบ domain ด้วย
function validateEmailDomain(email) {
  const basicError = validateEmail(email);
  if (basicError) return basicError;

  const domain = email.split('@')[1];
  const blockedDomains = ['tempmail.com', 'throwaway.com', '10minutemail.com'];

  if (blockedDomains.includes(domain.toLowerCase())) {
    return 'ไม่รองรับ email ชั่วคราว';
  }
  return null;
}

// HTML form integration
const emailInput = document.createElement('input');
emailInput.type = 'email';
emailInput.addEventListener('blur', (e) => {
  const error = validateEmail(e.target.value);
  if (error) {
    e.target.setCustomValidity(error);
    e.target.reportValidity();
  } else {
    e.target.setCustomValidity('');
  }
});
```

---

## Step 237: Validating Phone Numbers

```javascript
// Phone number validation สำหรับเบอร์โทรไทย

function validateThaiPhone(phone) {
  const cleaned = phone.replace(/[\s\-\(\)\.]/g, ''); // ลบ spaces, dashes, etc.

  if (!cleaned) return 'กรุณากรอกเบอร์โทรศัพท์';

  // เบอร์มือถือไทย: 06x, 08x, 09x (10 หลัก)
  const mobileRegex = /^(06|08|09)\d{8}$/;
  // เบอร์บ้านไทย: 02xxxxxxx (9 หลัก) หรือ 0x-xxxxxxx (9-10 หลัก)
  const landlineRegex = /^(02|03|04|05|07)\d{7,8}$/;
  // เบอร์ international (ไม่บังคับ)
  const intlRegex = /^\+\d{7,15}$/;

  if (!mobileRegex.test(cleaned) && !landlineRegex.test(cleaned) && !intlRegex.test(cleaned)) {
    return 'เบอร์โทรศัพท์ไม่ถูกต้อง (ตัวอย่าง: 0812345678)';
  }
  return null;
}

// Format phone number
function formatPhone(phone) {
  const cleaned = phone.replace(/\D/g, '');
  if (cleaned.length === 10) {
    return `${cleaned.slice(0, 3)}-${cleaned.slice(3, 6)}-${cleaned.slice(6)}`;
  }
  return phone;
}

// Phone input with auto-format
const phoneInput = document.createElement('input');
phoneInput.type = 'tel';
phoneInput.placeholder = '0812345678';

phoneInput.addEventListener('input', (e) => {
  let value = e.target.value.replace(/\D/g, '').slice(0, 10);
  e.target.value = value;
});

phoneInput.addEventListener('blur', (e) => {
  const error = validateThaiPhone(e.target.value);
  if (error) {
    console.log('Phone error:', error);
  } else {
    e.target.value = formatPhone(e.target.value);
  }
});

// ทดสอบ
const phones = [
  '0812345678',   // valid mobile
  '0234567890',   // valid landline
  '+66812345678', // valid international
  '12345',        // invalid
  '0612345678',   // valid mobile
];

phones.forEach(p => {
  const err = validateThaiPhone(p);
  console.log(`"${p}": ${err || 'valid'}`);
});
```

---

## Step 238: Password Strength Checker

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .strength-bar { height: 8px; border-radius: 4px; background: #ddd; margin: 5px 0; }
    .strength-fill { height: 100%; border-radius: 4px; transition: all 0.3s; }
    .strength-weak { background: red; width: 25%; }
    .strength-fair { background: orange; width: 50%; }
    .strength-good { background: #ffcc00; width: 75%; }
    .strength-strong { background: green; width: 100%; }
    .requirements { font-size: 0.85em; }
    .req { padding: 2px 0; }
    .req.met { color: green; }
    .req.unmet { color: #999; }
  </style>
</head>
<body>
  <input type="password" id="password" placeholder="กรอกรหัสผ่าน">
  <div class="strength-bar"><div class="strength-fill" id="strength-fill"></div></div>
  <p id="strength-text">กรุณากรอกรหัสผ่าน</p>
  <div class="requirements" id="requirements"></div>
  <input type="password" id="confirm-password" placeholder="ยืนยันรหัสผ่าน">
  <p id="match-text"></p>

  <script>
    const passwordInput = document.getElementById('password');
    const confirmInput = document.getElementById('confirm-password');
    const strengthFill = document.getElementById('strength-fill');
    const strengthText = document.getElementById('strength-text');
    const requirementsEl = document.getElementById('requirements');
    const matchText = document.getElementById('match-text');

    const requirements = [
      { id: 'length', label: 'อย่างน้อย 8 ตัวอักษร', test: p => p.length >= 8 },
      { id: 'uppercase', label: 'มีตัวพิมพ์ใหญ่ (A-Z)', test: p => /[A-Z]/.test(p) },
      { id: 'lowercase', label: 'มีตัวพิมพ์เล็ก (a-z)', test: p => /[a-z]/.test(p) },
      { id: 'number', label: 'มีตัวเลข (0-9)', test: p => /\d/.test(p) },
      { id: 'special', label: 'มีอักขระพิเศษ (!@#$%...)', test: p => /[!@#$%^&*(),.?":{}|<>]/.test(p) },
      { id: 'no-common', label: 'ไม่ใช่รหัสทั่วไป', test: p => !['password', '12345678', 'qwerty123'].includes(p.toLowerCase()) },
    ];

    // สร้าง requirements UI
    requirements.forEach(req => {
      const div = document.createElement('div');
      div.className = 'req unmet';
      div.id = `req-${req.id}`;
      div.textContent = `✗ ${req.label}`;
      requirementsEl.appendChild(div);
    });

    function checkPasswordStrength(password) {
      if (!password) {
        return { score: 0, level: 'none', label: 'กรุณากรอกรหัสผ่าน' };
      }

      let metCount = 0;
      requirements.forEach(req => {
        const met = req.test(password);
        const el = document.getElementById(`req-${req.id}`);
        if (met) {
          el.className = 'req met';
          el.textContent = `✓ ${req.label}`;
          metCount++;
        } else {
          el.className = 'req unmet';
          el.textContent = `✗ ${req.label}`;
        }
      });

      if (metCount <= 1) return { score: 25, level: 'weak', label: 'อ่อนแอมาก' };
      if (metCount <= 2) return { score: 50, level: 'fair', label: 'พอใช้' };
      if (metCount <= 4) return { score: 75, level: 'good', label: 'ดี' };
      return { score: 100, level: 'strong', label: 'แข็งแกร่งมาก' };
    }

    passwordInput.addEventListener('input', () => {
      const password = passwordInput.value;
      const { score, level, label } = checkPasswordStrength(password);

      strengthFill.className = `strength-fill strength-${level}`;
      if (level === 'none') strengthFill.style.width = '0';
      strengthText.textContent = label;

      checkMatch();
    });

    function checkMatch() {
      const p1 = passwordInput.value;
      const p2 = confirmInput.value;
      if (!p2) { matchText.textContent = ''; return; }
      if (p1 === p2) {
        matchText.textContent = '✓ รหัสผ่านตรงกัน';
        matchText.style.color = 'green';
      } else {
        matchText.textContent = '✗ รหัสผ่านไม่ตรงกัน';
        matchText.style.color = 'red';
      }
    }

    confirmInput.addEventListener('input', checkMatch);

    function validatePassword(password) {
      const strength = checkPasswordStrength(password);
      if (strength.level === 'none') return 'กรุณากรอกรหัสผ่าน';
      if (strength.level === 'weak') return 'รหัสผ่านอ่อนแอเกินไป';
      return null;
    }
  </script>
</body>
</html>
```

---

## Step 239: Validating Dates

```javascript
// Date validation functions

function isValidDate(dateStr) {
  if (!dateStr) return false;
  const date = new Date(dateStr);
  return !isNaN(date.getTime());
}

function validateDate(dateStr, options = {}) {
  const { min, max, label = 'วันที่', required = true } = options;

  if (!dateStr) {
    return required ? `กรุณากรอก${label}` : null;
  }

  if (!isValidDate(dateStr)) {
    return `รูปแบบ${label}ไม่ถูกต้อง (YYYY-MM-DD)`;
  }

  const date = new Date(dateStr);
  const today = new Date();
  today.setHours(0, 0, 0, 0);

  if (min) {
    const minDate = new Date(min);
    if (date < minDate) {
      return `${label} ต้องไม่ก่อน ${formatDateThai(min)}`;
    }
  }

  if (max) {
    const maxDate = new Date(max);
    if (date > maxDate) {
      return `${label} ต้องไม่หลัง ${formatDateThai(max)}`;
    }
  }

  return null;
}

function formatDateThai(dateStr) {
  const date = new Date(dateStr);
  return date.toLocaleDateString('th-TH', {
    year: 'numeric', month: 'long', day: 'numeric'
  });
}

function calculateAge(birthday) {
  const today = new Date();
  const birth = new Date(birthday);
  let age = today.getFullYear() - birth.getFullYear();
  const m = today.getMonth() - birth.getMonth();
  if (m < 0 || (m === 0 && today.getDate() < birth.getDate())) age--;
  return age;
}

function validateBirthday(dateStr) {
  const error = validateDate(dateStr, {
    label: 'วันเกิด',
    max: new Date().toISOString().split('T')[0], // ไม่เกินวันนี้
    min: '1900-01-01',
  });
  if (error) return error;

  const age = calculateAge(dateStr);
  if (age < 0) return 'วันเกิดต้องไม่ใช่วันในอนาคต';
  if (age < 18) return `คุณอายุ ${age} ปี ต้องมีอายุ 18 ปีขึ้นไป`;
  if (age > 120) return 'วันเกิดไม่ถูกต้อง';

  return null;
}

// ทดสอบ
console.log(validateBirthday('2010-01-01')); // อายุน้อยกว่า 18
console.log(validateBirthday('1990-05-15')); // null (valid)
console.log(validateBirthday('2030-01-01')); // วันในอนาคต
console.log(validateBirthday('invalid'));     // รูปแบบไม่ถูกต้อง

// Date range validation
function validateDateRange(startDate, endDate) {
  if (!startDate || !endDate) return 'กรุณากรอกวันที่ครบ';
  if (!isValidDate(startDate) || !isValidDate(endDate)) return 'รูปแบบวันที่ไม่ถูกต้อง';
  if (new Date(startDate) > new Date(endDate)) {
    return 'วันเริ่มต้องไม่เกินวันสิ้นสุด';
  }
  return null;
}
```

---

## Step 240: Real-time Validation

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .field { margin: 10px 0; }
    .field input { padding: 8px; width: 300px; }
    .field input.valid { border: 2px solid green; }
    .field input.invalid { border: 2px solid red; }
    .field input.touched { /* ผ่านการพิมพ์แล้ว */ }
    .error-msg { color: red; font-size: 0.85em; min-height: 18px; }
    .success-msg { color: green; font-size: 0.85em; }
  </style>
</head>
<body>
  <form id="realtime-form" novalidate>
    <div class="field">
      <label>Username</label>
      <input type="text" id="rt-username" placeholder="3-20 ตัวอักษร">
      <div class="error-msg" id="rt-username-error"></div>
    </div>
    <div class="field">
      <label>Email</label>
      <input type="email" id="rt-email" placeholder="user@example.com">
      <div class="error-msg" id="rt-email-error"></div>
    </div>
    <div class="field">
      <label>Password</label>
      <input type="password" id="rt-password" placeholder="อย่างน้อย 8 ตัวอักษร">
      <div class="error-msg" id="rt-password-error"></div>
    </div>
    <button type="submit">สมัครสมาชิก</button>
  </form>

  <script>
    // Validation rules
    const rules = {
      'rt-username': [
        { test: v => v.trim() !== '', msg: 'กรุณากรอก username' },
        { test: v => v.length >= 3, msg: 'Username ต้องมีอย่างน้อย 3 ตัวอักษร' },
        { test: v => v.length <= 20, msg: 'Username ต้องไม่เกิน 20 ตัวอักษร' },
        { test: v => /^[a-zA-Z0-9_]+$/.test(v), msg: 'ใช้ได้เฉพาะ a-z, 0-9, _' },
      ],
      'rt-email': [
        { test: v => v.trim() !== '', msg: 'กรุณากรอก email' },
        { test: v => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v), msg: 'รูปแบบ email ไม่ถูกต้อง' },
      ],
      'rt-password': [
        { test: v => v !== '', msg: 'กรุณากรอกรหัสผ่าน' },
        { test: v => v.length >= 8, msg: 'อย่างน้อย 8 ตัวอักษร' },
        { test: v => /[A-Z]/.test(v), msg: 'ต้องมีตัวพิมพ์ใหญ่' },
        { test: v => /\d/.test(v), msg: 'ต้องมีตัวเลข' },
      ],
    };

    function validateField(inputId, value) {
      const fieldRules = rules[inputId];
      if (!fieldRules) return null;

      for (const rule of fieldRules) {
        if (!rule.test(value)) {
          return rule.msg;
        }
      }
      return null;
    }

    function updateFieldUI(inputEl, errorEl, error) {
      if (error) {
        inputEl.classList.remove('valid');
        inputEl.classList.add('invalid');
        errorEl.textContent = error;
      } else {
        inputEl.classList.remove('invalid');
        inputEl.classList.add('valid');
        errorEl.textContent = '✓';
        errorEl.className = 'success-msg';
      }
    }

    // Setup real-time validation
    Object.keys(rules).forEach(inputId => {
      const input = document.getElementById(inputId);
      const errorEl = document.getElementById(`${inputId}-error`);

      // Validate on input (real-time)
      input.addEventListener('input', () => {
        const error = validateField(inputId, input.value);
        updateFieldUI(input, errorEl, error);
      });

      // Also validate on blur (when leaving field)
      input.addEventListener('blur', () => {
        const error = validateField(inputId, input.value);
        updateFieldUI(input, errorEl, error);
      });
    });

    // Form submit
    document.getElementById('realtime-form').addEventListener('submit', (e) => {
      e.preventDefault();
      let allValid = true;

      Object.keys(rules).forEach(inputId => {
        const input = document.getElementById(inputId);
        const errorEl = document.getElementById(`${inputId}-error`);
        const error = validateField(inputId, input.value);
        updateFieldUI(input, errorEl, error);
        if (error) allValid = false;
      });

      if (allValid) {
        alert('ข้อมูลถูกต้องทั้งหมด!');
      }
    });

    // Debounced validation (ไม่ validate ทุก keystroke)
    function debounce(fn, delay) {
      let timer;
      return (...args) => {
        clearTimeout(timer);
        timer = setTimeout(() => fn(...args), delay);
      };
    }

    // ตัวอย่าง: ตรวจ username ว่าซ้ำไหม (async)
    const checkUsernameAvailable = debounce(async (username) => {
      if (username.length < 3) return;
      // simulate API call
      await new Promise(r => setTimeout(r, 500));
      const taken = ['admin', 'user', 'test'].includes(username.toLowerCase());
      const errorEl = document.getElementById('rt-username-error');
      if (taken) {
        errorEl.textContent = 'Username นี้มีคนใช้แล้ว';
        errorEl.className = 'error-msg';
      }
    }, 500);

    document.getElementById('rt-username').addEventListener('input', (e) => {
      checkUsernameAvailable(e.target.value);
    });
  </script>
</body>
</html>
```

---

## Step 241: Showing/Hiding Error Messages

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .field { position: relative; margin: 15px 0; }
    .field input { padding: 8px 35px 8px 8px; width: 300px; box-sizing: border-box; }
    .field-icon {
      position: absolute;
      right: 10px;
      top: 50%;
      transform: translateY(-50%);
      font-size: 16px;
    }
    .error-message {
      display: none;
      color: red;
      font-size: 0.82em;
      margin-top: 3px;
      padding: 4px 8px;
      background: #fff0f0;
      border-left: 3px solid red;
      border-radius: 0 4px 4px 0;
    }
    .error-message.visible { display: block; animation: slideIn 0.2s ease; }
    .tooltip {
      display: none;
      position: absolute;
      background: #333;
      color: white;
      padding: 6px 10px;
      border-radius: 4px;
      font-size: 0.8em;
      bottom: 110%;
      left: 0;
      white-space: nowrap;
      z-index: 100;
    }
    .tooltip::after {
      content: '';
      position: absolute;
      top: 100%; left: 10px;
      border: 5px solid transparent;
      border-top-color: #333;
    }
    .field:focus-within .tooltip { display: block; }
    @keyframes slideIn {
      from { opacity: 0; transform: translateY(-5px); }
      to { opacity: 1; transform: translateY(0); }
    }
  </style>
</head>
<body>
  <form id="fancy-form">
    <div class="field">
      <label>Email</label>
      <input type="email" id="f-email" placeholder="your@email.com">
      <span class="field-icon" id="f-email-icon"></span>
      <div class="tooltip">กรอก email ของคุณ เช่น user@example.com</div>
      <div class="error-message" id="f-email-error"></div>
    </div>

    <div class="field">
      <label>Username</label>
      <input type="text" id="f-username" placeholder="3-20 ตัวอักษร">
      <span class="field-icon" id="f-username-icon"></span>
      <div class="tooltip">ใช้ได้เฉพาะ a-z, A-Z, 0-9 และ _</div>
      <div class="error-message" id="f-username-error"></div>
    </div>

    <button type="submit">Submit</button>
  </form>

  <script>
    function showError(fieldId, message) {
      const errorEl = document.getElementById(`${fieldId}-error`);
      const iconEl = document.getElementById(`${fieldId}-icon`);
      errorEl.textContent = message;
      errorEl.classList.add('visible');
      if (iconEl) iconEl.textContent = '✗';
    }

    function hideError(fieldId) {
      const errorEl = document.getElementById(`${fieldId}-error`);
      const iconEl = document.getElementById(`${fieldId}-icon`);
      errorEl.classList.remove('visible');
      errorEl.textContent = '';
    }

    function showSuccess(fieldId) {
      const iconEl = document.getElementById(`${fieldId}-icon`);
      if (iconEl) {
        iconEl.textContent = '✓';
        iconEl.style.color = 'green';
      }
      hideError(fieldId);
    }

    // Email validation
    const emailInput = document.getElementById('f-email');
    emailInput.addEventListener('blur', () => {
      const value = emailInput.value.trim();
      if (!value) {
        showError('f-email', 'กรุณากรอก email');
      } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
        showError('f-email', 'รูปแบบ email ไม่ถูกต้อง');
      } else {
        showSuccess('f-email');
      }
    });

    emailInput.addEventListener('input', () => {
      hideError('f-email');
    });

    // Username validation
    const usernameInput = document.getElementById('f-username');
    usernameInput.addEventListener('blur', () => {
      const value = usernameInput.value.trim();
      if (!value) {
        showError('f-username', 'กรุณากรอก username');
      } else if (value.length < 3) {
        showError('f-username', 'Username สั้นเกินไป');
      } else if (!/^[a-zA-Z0-9_]+$/.test(value)) {
        showError('f-username', 'ใช้ได้เฉพาะ a-z, 0-9, _');
      } else {
        showSuccess('f-username');
      }
    });

    // Form submit - แสดง error ทุก field
    document.getElementById('fancy-form').addEventListener('submit', (e) => {
      e.preventDefault();
      // Trigger blur on all fields
      document.querySelectorAll('#fancy-form input').forEach(input => {
        input.dispatchEvent(new Event('blur'));
      });
    });
  </script>
</body>
</html>
```

---

## Step 242: FormData API

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <form id="upload-form" enctype="multipart/form-data">
    <input type="text" name="title" value="My Post">
    <input type="text" name="author" value="สมชาย">
    <textarea name="content">เนื้อหาบทความ...</textarea>
    <input type="checkbox" name="tags" value="javascript"> JavaScript
    <input type="checkbox" name="tags" value="css" checked> CSS
    <input type="checkbox" name="tags" value="html" checked> HTML
    <input type="file" name="cover" id="cover-file">
    <button type="submit">Submit</button>
  </form>

  <script>
    const form = document.getElementById('upload-form');

    form.addEventListener('submit', async (e) => {
      e.preventDefault();

      // สร้าง FormData จาก form
      const formData = new FormData(form);

      // อ่านค่า
      console.log(formData.get('title'));   // "My Post" (ค่าแรก)
      console.log(formData.get('author'));  // "สมชาย"
      console.log(formData.getAll('tags')); // ["css", "html"] (ทุกค่า)

      // ตรวจสอบว่ามี key ไหม
      console.log(formData.has('title'));    // true
      console.log(formData.has('missing')); // false

      // ตั้งค่า
      formData.set('title', 'Updated Title');
      formData.set('date', new Date().toISOString());

      // append (เพิ่ม ไม่แทน)
      formData.append('tags', 'javascript');
      console.log(formData.getAll('tags')); // ["css", "html", "javascript"]

      // ลบ key
      formData.delete('author');

      // iterate
      for (const [key, value] of formData.entries()) {
        console.log(`${key}:`, value instanceof File ? `File(${value.name})` : value);
      }

      // แปลงเป็น object (จะไม่ได้ multiple values)
      const obj = Object.fromEntries(formData);
      console.log(obj);

      // แปลงให้ได้ multiple values
      const fullObj = {};
      for (const [key, value] of formData.entries()) {
        if (fullObj[key] !== undefined) {
          if (!Array.isArray(fullObj[key])) fullObj[key] = [fullObj[key]];
          fullObj[key].push(value);
        } else {
          fullObj[key] = value;
        }
      }
      console.log(fullObj);

      // ส่งด้วย fetch
      try {
        const response = await fetch('/api/posts', {
          method: 'POST',
          body: formData, // ส่ง FormData ตรงๆ (ไม่ต้อง set Content-Type)
        });
        const data = await response.json();
        console.log('Success:', data);
      } catch (error) {
        console.error('Error:', error);
      }
    });

    // สร้าง FormData ด้วยตัวเอง (ไม่จาก form)
    const manualFormData = new FormData();
    manualFormData.append('name', 'สมชาย');
    manualFormData.append('age', 25);

    // เพิ่ม file
    const fileInput = document.getElementById('cover-file');
    fileInput?.addEventListener('change', (e) => {
      const file = e.target.files[0];
      if (file) {
        const fd = new FormData();
        fd.append('file', file, file.name);
        console.log('File added to FormData:', file.name);
      }
    });
  </script>
</body>
</html>
```

---

## Step 243: Checkboxes และ Radio Buttons

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <form id="prefs-form">
    <!-- Checkboxes -->
    <fieldset>
      <legend>ความสนใจ</legend>
      <label><input type="checkbox" name="interests" value="sports"> กีฬา</label>
      <label><input type="checkbox" name="interests" value="music"> ดนตรี</label>
      <label><input type="checkbox" name="interests" value="tech"> เทคโนโลยี</label>
      <label><input type="checkbox" name="interests" value="travel"> ท่องเที่ยว</label>
    </fieldset>

    <!-- Select All -->
    <label><input type="checkbox" id="select-all"> เลือกทั้งหมด</label>

    <!-- Radio Buttons -->
    <fieldset>
      <legend>เพศ</legend>
      <label><input type="radio" name="gender" value="male" required> ชาย</label>
      <label><input type="radio" name="gender" value="female"> หญิง</label>
      <label><input type="radio" name="gender" value="other"> อื่นๆ</label>
    </fieldset>

    <!-- Rating radio -->
    <div id="rating">
      <label><input type="radio" name="rating" value="1"> ★</label>
      <label><input type="radio" name="rating" value="2"> ★★</label>
      <label><input type="radio" name="rating" value="3"> ★★★</label>
      <label><input type="radio" name="rating" value="4"> ★★★★</label>
      <label><input type="radio" name="rating" value="5"> ★★★★★</label>
    </div>

    <button type="submit">บันทึก</button>
  </form>

  <div id="output"></div>

  <script>
    const form = document.getElementById('prefs-form');
    const output = document.getElementById('output');

    // Select All checkbox
    const selectAll = document.getElementById('select-all');
    const interestCheckboxes = document.querySelectorAll('input[name="interests"]');

    selectAll.addEventListener('change', () => {
      interestCheckboxes.forEach(cb => cb.checked = selectAll.checked);
    });

    interestCheckboxes.forEach(cb => {
      cb.addEventListener('change', () => {
        const allChecked = Array.from(interestCheckboxes).every(c => c.checked);
        const someChecked = Array.from(interestCheckboxes).some(c => c.checked);
        selectAll.checked = allChecked;
        selectAll.indeterminate = someChecked && !allChecked;
      });
    });

    // อ่านค่า checkboxes
    function getCheckedValues(name) {
      return Array.from(document.querySelectorAll(`input[name="${name}"]:checked`))
        .map(el => el.value);
    }

    // อ่านค่า radio
    function getRadioValue(name) {
      const el = document.querySelector(`input[name="${name}"]:checked`);
      return el ? el.value : null;
    }

    // ตั้งค่า checkbox
    function setCheckboxValues(name, values) {
      document.querySelectorAll(`input[name="${name}"]`).forEach(cb => {
        cb.checked = values.includes(cb.value);
      });
    }

    // ตั้งค่า radio
    function setRadioValue(name, value) {
      const el = document.querySelector(`input[name="${name}"][value="${value}"]`);
      if (el) el.checked = true;
    }

    // โหลดค่าตัวอย่าง
    setCheckboxValues('interests', ['tech', 'travel']);
    setRadioValue('gender', 'male');
    setRadioValue('rating', '4');

    form.addEventListener('submit', (e) => {
      e.preventDefault();

      const data = {
        interests: getCheckedValues('interests'),
        gender: getRadioValue('gender'),
        rating: getRadioValue('rating'),
      };

      if (!data.gender) {
        alert('กรุณาเลือกเพศ');
        return;
      }
      if (data.interests.length === 0) {
        alert('กรุณาเลือกความสนใจอย่างน้อย 1 อย่าง');
        return;
      }

      output.innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
    });
  </script>
</body>
</html>
```

---

## Step 244: Select Elements

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <form>
    <!-- Single select -->
    <select id="province">
      <option value="">-- เลือกจังหวัด --</option>
      <optgroup label="ภาคกลาง">
        <option value="bkk">กรุงเทพมหานคร</option>
        <option value="nth">นนทบุรี</option>
        <option value="pth">ปทุมธานี</option>
      </optgroup>
      <optgroup label="ภาคเหนือ">
        <option value="cmi">เชียงใหม่</option>
        <option value="cri">เชียงราย</option>
      </optgroup>
      <optgroup label="ภาคใต้">
        <option value="pkt">ภูเก็ต</option>
        <option value="srt">สุราษฎร์ธานี</option>
      </optgroup>
    </select>

    <!-- Multiple select -->
    <select id="languages" multiple size="5">
      <option value="th">ไทย</option>
      <option value="en">English</option>
      <option value="jp">日本語</option>
      <option value="cn">中文</option>
      <option value="ko">한국어</option>
    </select>

    <!-- Dynamic select -->
    <select id="category"></select>
    <select id="subcategory"></select>
  </form>

  <script>
    const province = document.getElementById('province');

    // อ่าน selected value
    console.log(province.value); // ""
    console.log(province.selectedIndex); // 0

    // ตั้งค่า
    province.value = 'cmi';
    console.log(province.value);         // "cmi"
    console.log(province.selectedIndex); // 4

    // อ่าน selected option
    const selectedOption = province.options[province.selectedIndex];
    console.log(selectedOption.text);  // "เชียงใหม่"
    console.log(selectedOption.value); // "cmi"

    // iterate options
    Array.from(province.options).forEach(opt => {
      console.log(opt.value, opt.text, opt.selected);
    });

    // Multiple select
    const languages = document.getElementById('languages');
    languages.options[0].selected = true; // select ไทย
    languages.options[1].selected = true; // select English

    // อ่าน multiple selected
    const selectedLangs = Array.from(languages.selectedOptions).map(opt => ({
      value: opt.value,
      text: opt.text,
    }));
    console.log(selectedLangs);

    // Cascade selects (Category -> Subcategory)
    const categories = {
      'food': { label: 'อาหาร', subs: ['อาหารไทย', 'อาหารญี่ปุ่น', 'อาหารตะวันตก'] },
      'tech': { label: 'เทคโนโลยี', subs: ['มือถือ', 'คอมพิวเตอร์', 'AI'] },
      'sport': { label: 'กีฬา', subs: ['ฟุตบอล', 'บาสเกตบอล', 'ว่ายน้ำ'] },
    };

    const categorySelect = document.getElementById('category');
    const subcategorySelect = document.getElementById('subcategory');

    // Populate category
    categorySelect.innerHTML = '<option value="">-- เลือกหมวดหมู่ --</option>';
    Object.entries(categories).forEach(([value, { label }]) => {
      const opt = document.createElement('option');
      opt.value = value;
      opt.textContent = label;
      categorySelect.appendChild(opt);
    });

    // Update subcategory on change
    categorySelect.addEventListener('change', () => {
      const cat = categories[categorySelect.value];
      subcategorySelect.innerHTML = '<option value="">-- เลือกหมวดย่อย --</option>';
      subcategorySelect.disabled = !cat;

      if (cat) {
        cat.subs.forEach(sub => {
          const opt = document.createElement('option');
          opt.value = sub;
          opt.textContent = sub;
          subcategorySelect.appendChild(opt);
        });
      }
    });

    // Add option programmatically
    function addOption(selectEl, value, text, prepend = false) {
      const opt = new Option(text, value); // shorthand constructor
      if (prepend && selectEl.options.length > 0) {
        selectEl.insertBefore(opt, selectEl.options[0]);
      } else {
        selectEl.add(opt);
      }
    }

    // Remove option
    function removeOption(selectEl, value) {
      const opt = Array.from(selectEl.options).find(o => o.value === value);
      if (opt) opt.remove();
    }
  </script>
</body>
</html>
```

---

## Step 245: File Inputs

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .drop-zone {
      border: 2px dashed #ccc;
      padding: 30px;
      text-align: center;
      border-radius: 8px;
      cursor: pointer;
    }
    .drop-zone.dragover { border-color: blue; background: #e8f0fe; }
    .preview-img { max-width: 150px; max-height: 150px; border-radius: 4px; }
    .file-list { list-style: none; padding: 0; }
    .file-item { padding: 8px; background: #f5f5f5; margin: 4px 0; border-radius: 4px; }
  </style>
</head>
<body>
  <form id="file-form">
    <!-- Single file -->
    <input type="file" id="avatar" accept="image/*">
    <div id="avatar-preview"></div>

    <!-- Multiple files -->
    <input type="file" id="documents" multiple accept=".pdf,.doc,.docx,.txt">
    <ul id="file-list" class="file-list"></ul>

    <!-- Drag and Drop zone -->
    <div id="drop-zone" class="drop-zone">
      ลากไฟล์มาวางที่นี่ หรือ <label for="drop-input" style="color:blue;cursor:pointer">คลิกเพื่อเลือก</label>
      <input type="file" id="drop-input" multiple style="display:none">
    </div>
    <div id="dropped-files"></div>
  </form>

  <script>
    // Single file - Preview image
    const avatarInput = document.getElementById('avatar');
    const avatarPreview = document.getElementById('avatar-preview');

    avatarInput.addEventListener('change', (e) => {
      const file = e.target.files[0];
      if (!file) return;

      // Validate
      if (!file.type.startsWith('image/')) {
        alert('กรุณาเลือกไฟล์รูปภาพ');
        avatarInput.value = '';
        return;
      }
      if (file.size > 2 * 1024 * 1024) { // 2MB
        alert('ไฟล์ใหญ่เกิน 2MB');
        avatarInput.value = '';
        return;
      }

      // Preview
      const reader = new FileReader();
      reader.onload = (e) => {
        avatarPreview.innerHTML = `<img src="${e.target.result}" class="preview-img" alt="Preview">`;
      };
      reader.readAsDataURL(file);

      console.log('File info:', {
        name: file.name,
        size: file.size,
        type: file.type,
        lastModified: new Date(file.lastModified).toLocaleDateString('th-TH'),
      });
    });

    // Multiple files
    const documentsInput = document.getElementById('documents');
    const fileList = document.getElementById('file-list');

    documentsInput.addEventListener('change', (e) => {
      const files = Array.from(e.target.files);
      fileList.innerHTML = '';

      files.forEach(file => {
        const li = document.createElement('li');
        li.className = 'file-item';
        li.innerHTML = `
          <strong>${file.name}</strong>
          <span>${formatFileSize(file.size)}</span>
          <span>${file.type || 'unknown'}</span>
        `;
        fileList.appendChild(li);
      });
    });

    function formatFileSize(bytes) {
      if (bytes < 1024) return bytes + ' B';
      if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
      return (bytes / (1024 * 1024)).toFixed(1) + ' MB';
    }

    // Drag and Drop
    const dropZone = document.getElementById('drop-zone');
    const dropInput = document.getElementById('drop-input');
    const droppedFiles = document.getElementById('dropped-files');

    dropZone.addEventListener('click', () => dropInput.click());

    dropZone.addEventListener('dragover', (e) => {
      e.preventDefault();
      dropZone.classList.add('dragover');
    });

    dropZone.addEventListener('dragleave', () => {
      dropZone.classList.remove('dragover');
    });

    dropZone.addEventListener('drop', (e) => {
      e.preventDefault();
      dropZone.classList.remove('dragover');
      handleFiles(e.dataTransfer.files);
    });

    dropInput.addEventListener('change', (e) => {
      handleFiles(e.target.files);
    });

    function handleFiles(files) {
      droppedFiles.innerHTML = '';
      Array.from(files).forEach(file => {
        const div = document.createElement('div');
        div.className = 'file-item';
        div.textContent = `${file.name} (${formatFileSize(file.size)})`;
        droppedFiles.appendChild(div);
      });
    }

    // Upload with progress
    async function uploadFile(file) {
      const formData = new FormData();
      formData.append('file', file);

      const xhr = new XMLHttpRequest();
      xhr.upload.addEventListener('progress', (e) => {
        if (e.lengthComputable) {
          const percent = (e.loaded / e.total) * 100;
          console.log(`Upload progress: ${percent.toFixed(1)}%`);
        }
      });

      return new Promise((resolve, reject) => {
        xhr.onload = () => resolve(JSON.parse(xhr.responseText));
        xhr.onerror = () => reject(new Error('Upload failed'));
        xhr.open('POST', '/api/upload');
        xhr.send(formData);
      });
    }
  </script>
</body>
</html>
```

---

## Step 246-250: Complete Registration Form

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <title>Registration Form</title>
  <style>
    * { box-sizing: border-box; }
    body { font-family: 'Segoe UI', sans-serif; max-width: 600px; margin: 30px auto; padding: 0 20px; }
    h1 { text-align: center; color: #333; }
    .form-group { margin-bottom: 20px; }
    label { display: block; margin-bottom: 5px; font-weight: 600; color: #555; }
    label .required { color: red; }
    input, select, textarea { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 6px; font-size: 16px; transition: border 0.2s; }
    input:focus, select:focus, textarea:focus { outline: none; border-color: #4285f4; box-shadow: 0 0 0 3px rgba(66,133,244,0.2); }
    .input-wrapper { position: relative; }
    .input-icon { position: absolute; right: 12px; top: 50%; transform: translateY(-50%); font-size: 18px; }
    .error { display: none; color: #d32f2f; font-size: 0.82em; margin-top: 4px; }
    .error.show { display: block; }
    input.has-error { border-color: #d32f2f; }
    input.has-success { border-color: #388e3c; }
    .step-indicator { display: flex; justify-content: space-between; margin-bottom: 30px; }
    .step { flex: 1; text-align: center; padding: 10px; border-bottom: 3px solid #ddd; color: #999; font-size: 0.85em; }
    .step.active { border-color: #4285f4; color: #4285f4; font-weight: bold; }
    .step.done { border-color: #388e3c; color: #388e3c; }
    .form-section { display: none; }
    .form-section.active { display: block; }
    .btn { padding: 12px 24px; border: none; border-radius: 6px; cursor: pointer; font-size: 16px; }
    .btn-primary { background: #4285f4; color: white; }
    .btn-secondary { background: #ddd; }
    .btn:disabled { opacity: 0.6; cursor: not-allowed; }
    .form-actions { display: flex; justify-content: space-between; margin-top: 20px; }
    .password-strength { height: 6px; border-radius: 3px; background: #ddd; margin-top: 5px; }
    .password-strength-fill { height: 100%; border-radius: 3px; transition: all 0.3s; }
    .terms { font-size: 0.85em; color: #666; }
    #success-msg { display: none; text-align: center; padding: 30px; background: #e8f5e9; border-radius: 8px; }
  </style>
</head>
<body>
  <h1>สมัครสมาชิก</h1>

  <div class="step-indicator">
    <div class="step active" id="step-1-indicator">1. ข้อมูลส่วนตัว</div>
    <div class="step" id="step-2-indicator">2. บัญชีผู้ใช้</div>
    <div class="step" id="step-3-indicator">3. ยืนยัน</div>
  </div>

  <form id="reg-form" novalidate>
    <!-- Step 1: Personal Info -->
    <div class="form-section active" id="section-1">
      <div class="form-group">
        <label>ชื่อ <span class="required">*</span></label>
        <div class="input-wrapper">
          <input type="text" id="first-name" name="firstName" placeholder="ชื่อของคุณ">
          <span class="input-icon" id="fn-icon"></span>
        </div>
        <div class="error" id="fn-error">กรุณากรอกชื่อ</div>
      </div>
      <div class="form-group">
        <label>นามสกุล <span class="required">*</span></label>
        <div class="input-wrapper">
          <input type="text" id="last-name" name="lastName" placeholder="นามสกุลของคุณ">
          <span class="input-icon" id="ln-icon"></span>
        </div>
        <div class="error" id="ln-error">กรุณากรอกนามสกุล</div>
      </div>
      <div class="form-group">
        <label>วันเกิด <span class="required">*</span></label>
        <input type="date" id="birthday" name="birthday">
        <div class="error" id="bd-error"></div>
      </div>
      <div class="form-group">
        <label>เบอร์โทร</label>
        <input type="tel" id="phone" name="phone" placeholder="0812345678">
        <div class="error" id="phone-error"></div>
      </div>
    </div>

    <!-- Step 2: Account Info -->
    <div class="form-section" id="section-2">
      <div class="form-group">
        <label>Email <span class="required">*</span></label>
        <div class="input-wrapper">
          <input type="email" id="reg-email" name="email" placeholder="your@email.com">
          <span class="input-icon" id="email-icon"></span>
        </div>
        <div class="error" id="email-error"></div>
      </div>
      <div class="form-group">
        <label>Username <span class="required">*</span></label>
        <input type="text" id="reg-username" name="username" placeholder="3-20 ตัวอักษร">
        <div class="error" id="username-error"></div>
      </div>
      <div class="form-group">
        <label>รหัสผ่าน <span class="required">*</span></label>
        <input type="password" id="reg-password" name="password" placeholder="อย่างน้อย 8 ตัวอักษร">
        <div class="password-strength"><div class="password-strength-fill" id="pw-strength-fill"></div></div>
        <div class="error" id="pw-error"></div>
      </div>
      <div class="form-group">
        <label>ยืนยันรหัสผ่าน <span class="required">*</span></label>
        <input type="password" id="confirm-pw" name="confirmPassword" placeholder="กรอกรหัสผ่านอีกครั้ง">
        <div class="error" id="cpw-error"></div>
      </div>
    </div>

    <!-- Step 3: Confirmation -->
    <div class="form-section" id="section-3">
      <div id="summary"></div>
      <div class="form-group">
        <label>
          <input type="checkbox" id="terms-check">
          ฉันยอมรับ <a href="#">ข้อตกลงและเงื่อนไข</a>
        </label>
        <div class="error" id="terms-error">กรุณายอมรับข้อตกลง</div>
      </div>
      <div class="form-group">
        <label>
          <input type="checkbox" id="newsletter-check" name="newsletter">
          รับข่าวสารและโปรโมชั่น
        </label>
      </div>
    </div>

    <div class="form-actions">
      <button type="button" class="btn btn-secondary" id="prev-btn" style="display:none">← ย้อนกลับ</button>
      <button type="button" class="btn btn-primary" id="next-btn">ถัดไป →</button>
      <button type="submit" class="btn btn-primary" id="submit-btn" style="display:none">สมัครสมาชิก</button>
    </div>
  </form>

  <div id="success-msg">
    <h2>🎉 สมัครสมาชิกสำเร็จ!</h2>
    <p>ยินดีต้อนรับสู่ระบบ!</p>
  </div>

  <script>
    let currentStep = 1;
    const totalSteps = 3;

    // Validators
    const validators = {
      firstName: (v) => !v.trim() ? 'กรุณากรอกชื่อ' : null,
      lastName: (v) => !v.trim() ? 'กรุณากรอกนามสกุล' : null,
      birthday: (v) => {
        if (!v) return 'กรุณากรอกวันเกิด';
        const age = Math.floor((Date.now() - new Date(v)) / (365.25 * 24 * 3600 * 1000));
        if (age < 13) return 'ต้องมีอายุ 13 ปีขึ้นไป';
        if (age > 120) return 'วันเกิดไม่ถูกต้อง';
        return null;
      },
      phone: (v) => {
        if (!v) return null; // optional
        return /^[0-9]{9,10}$/.test(v.replace(/[\s\-]/g, '')) ? null : 'เบอร์โทรไม่ถูกต้อง';
      },
      email: (v) => {
        if (!v) return 'กรุณากรอก email';
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v) ? null : 'Email ไม่ถูกต้อง';
      },
      username: (v) => {
        if (!v) return 'กรุณากรอก username';
        if (v.length < 3) return 'อย่างน้อย 3 ตัวอักษร';
        if (!/^[a-zA-Z0-9_]+$/.test(v)) return 'ใช้ได้เฉพาะ a-z, 0-9, _';
        return null;
      },
      password: (v) => {
        if (!v) return 'กรุณากรอกรหัสผ่าน';
        if (v.length < 8) return 'อย่างน้อย 8 ตัวอักษร';
        return null;
      },
    };

    function setFieldState(errorId, iconId, error) {
      const errorEl = document.getElementById(errorId);
      if (errorEl) {
        if (error) {
          errorEl.textContent = error;
          errorEl.classList.add('show');
        } else {
          errorEl.classList.remove('show');
        }
      }
    }

    function validateStep(step) {
      let valid = true;

      if (step === 1) {
        const fields = [
          { id: 'first-name', key: 'firstName', errorId: 'fn-error' },
          { id: 'last-name', key: 'lastName', errorId: 'ln-error' },
          { id: 'birthday', key: 'birthday', errorId: 'bd-error' },
          { id: 'phone', key: 'phone', errorId: 'phone-error' },
        ];
        fields.forEach(({ id, key, errorId }) => {
          const val = document.getElementById(id).value;
          const error = validators[key]?.(val);
          setFieldState(errorId, null, error);
          if (error) valid = false;
        });
      }

      if (step === 2) {
        const fields = [
          { id: 'reg-email', key: 'email', errorId: 'email-error' },
          { id: 'reg-username', key: 'username', errorId: 'username-error' },
          { id: 'reg-password', key: 'password', errorId: 'pw-error' },
        ];
        fields.forEach(({ id, key, errorId }) => {
          const val = document.getElementById(id).value;
          const error = validators[key]?.(val);
          setFieldState(errorId, null, error);
          if (error) valid = false;
        });

        const pw = document.getElementById('reg-password').value;
        const cpw = document.getElementById('confirm-pw').value;
        if (cpw && pw !== cpw) {
          setFieldState('cpw-error', null, 'รหัสผ่านไม่ตรงกัน');
          valid = false;
        } else if (cpw) {
          setFieldState('cpw-error', null, null);
        }
      }

      if (step === 3) {
        if (!document.getElementById('terms-check').checked) {
          setFieldState('terms-error', null, 'กรุณายอมรับข้อตกลง');
          valid = false;
        } else {
          setFieldState('terms-error', null, null);
        }
      }

      return valid;
    }

    function showStep(step) {
      document.querySelectorAll('.form-section').forEach((s, i) => {
        s.classList.toggle('active', i + 1 === step);
      });
      document.querySelectorAll('.step').forEach((s, i) => {
        s.classList.remove('active', 'done');
        if (i + 1 < step) s.classList.add('done');
        if (i + 1 === step) s.classList.add('active');
      });

      document.getElementById('prev-btn').style.display = step > 1 ? '' : 'none';
      document.getElementById('next-btn').style.display = step < totalSteps ? '' : 'none';
      document.getElementById('submit-btn').style.display = step === totalSteps ? '' : 'none';

      if (step === 3) updateSummary();
    }

    function updateSummary() {
      const summary = document.getElementById('summary');
      summary.innerHTML = `
        <div style="background:#f5f5f5;padding:15px;border-radius:6px;margin-bottom:15px;">
          <h3>สรุปข้อมูล</h3>
          <p>ชื่อ: ${document.getElementById('first-name').value} ${document.getElementById('last-name').value}</p>
          <p>วันเกิด: ${document.getElementById('birthday').value}</p>
          <p>Email: ${document.getElementById('reg-email').value}</p>
          <p>Username: ${document.getElementById('reg-username').value}</p>
        </div>
      `;
    }

    document.getElementById('next-btn').addEventListener('click', () => {
      if (validateStep(currentStep)) {
        currentStep++;
        showStep(currentStep);
      }
    });

    document.getElementById('prev-btn').addEventListener('click', () => {
      currentStep--;
      showStep(currentStep);
    });

    document.getElementById('reg-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      if (!validateStep(3)) return;

      const btn = document.getElementById('submit-btn');
      btn.disabled = true;
      btn.textContent = 'กำลังสมัคร...';

      await new Promise(r => setTimeout(r, 1500));

      document.getElementById('reg-form').style.display = 'none';
      document.querySelector('.step-indicator').style.display = 'none';
      document.getElementById('success-msg').style.display = 'block';
    });

    // Password strength
    document.getElementById('reg-password').addEventListener('input', (e) => {
      const p = e.target.value;
      let score = 0;
      if (p.length >= 8) score++;
      if (/[A-Z]/.test(p)) score++;
      if (/\d/.test(p)) score++;
      if (/[!@#$%^&*]/.test(p)) score++;

      const fill = document.getElementById('pw-strength-fill');
      const colors = ['', '#f44336', '#ff9800', '#ffeb3b', '#4caf50'];
      const widths = ['0%', '25%', '50%', '75%', '100%'];
      fill.style.width = widths[score];
      fill.style.background = colors[score];
    });
  </script>
</body>
</html>
```

---

## สรุป Steps 231-250

| Step | หัวข้อ |
|------|--------|
| 231 | HTML Form Elements |
| 232 | Getting/Setting Values |
| 233 | Form Submission |
| 234 | Validation Concepts |
| 235 | Required Fields |
| 236 | Email Validation |
| 237 | Phone Validation |
| 238 | Password Strength |
| 239 | Date Validation |
| 240 | Real-time Validation |
| 241 | Error Messages |
| 242 | FormData API |
| 243 | Checkboxes/Radio |
| 244 | Select Elements |
| 245 | File Inputs |
| 246-250 | Complete Registration Form |

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Survey Form
สร้าง survey form ที่:
- มีคำถามหลายประเภท (text, radio, checkbox, select)
- validate ทุก field ก่อน submit
- แสดง summary ก่อน submit จริง

### แบบฝึกหัดที่ 2: Dynamic Form Builder
สร้าง form builder ที่:
- เพิ่ม/ลบ fields ได้
- กำหนด validation rules ได้
- Export เป็น JSON

### แบบฝึกหัดที่ 3: Multi-step Checkout
สร้าง checkout form:
- Step 1: Cart review
- Step 2: Shipping info
- Step 3: Payment info
- Step 4: Confirmation

### แบบฝึกหัดที่ 4: Image Upload Preview
สร้าง profile picture uploader:
- drag & drop
- preview ก่อน upload
- crop/resize ด้วย Canvas
- แสดง progress

### แบบฝึกหัดที่ 5: Autocomplete Input
สร้าง autocomplete ที่:
- fetch suggestions จาก API (debounced)
- keyboard navigation
- highlight matching text
