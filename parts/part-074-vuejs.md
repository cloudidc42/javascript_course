# Part 74: Vue.js พื้นฐาน (Steps 1451-1470)

## บทนำ

Vue.js คือ progressive JavaScript framework สำหรับสร้าง UI พัฒนาโดย Evan You ในปี 2014 Vue โดดเด่นด้วยความเรียบง่าย เส้นการเรียนรู้ที่ไม่ชัน และ documentation ที่ยอดเยี่ยม เหมาะสำหรับทั้งโปรเจคเล็กและใหญ่

Vue 3 (ออกมาในปี 2020) นำ Composition API มาซึ่งทำให้เขียน logic ได้ยืดหยุ่นกว่าเดิมมาก

---

## Step 1451: Vue.js คืออะไร

### ลักษณะสำคัญของ Vue

```
Vue.js:
✓ Progressive Framework (เพิ่มทีละนิดได้)
✓ Reactivity System ที่ทรงพลัง
✓ Component-based Architecture
✓ Official ecosystem (Router, Pinia)
✓ Excellent DevTools
✓ Great documentation
```

### เปรียบเทียบ Vue vs React

```
┌─────────────────────┬──────────────┬──────────────────┐
│ Feature             │ Vue 3        │ React 18         │
├─────────────────────┼──────────────┼──────────────────┤
│ Learning Curve      │ Easier       │ Moderate         │
│ Template Syntax     │ HTML-like    │ JSX              │
│ Reactivity          │ Proxy-based  │ State hooks      │
│ Two-way Binding     │ v-model      │ Manual           │
│ State Management    │ Pinia        │ Redux/Zustand    │
│ Router              │ Vue Router   │ React Router     │
│ Performance         │ Excellent    │ Excellent        │
│ Ecosystem           │ Good         │ Very Large       │
│ TypeScript          │ Excellent    │ Excellent        │
│ Mobile             │ NativeScript │ React Native     │
└─────────────────────┴──────────────┴──────────────────┘
```

---

## Step 1452: Vue 3 กับ Vite Setup

```bash
# สร้างโปรเจค Vue 3 + Vite
npm create vite@latest my-vue-app -- --template vue
cd my-vue-app
npm install
npm run dev

# หรือด้วย TypeScript
npm create vite@latest my-vue-app -- --template vue-ts

# โครงสร้างโปรเจค
my-vue-app/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   │   └── vue.svg
│   ├── components/
│   │   └── HelloWorld.vue
│   ├── App.vue
│   ├── main.js
│   └── style.css
├── index.html
├── package.json
└── vite.config.js
```

```javascript
// main.js - Entry Point
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'

const app = createApp(App)
app.mount('#app')
```

```vue
<!-- App.vue - Root Component -->
<template>
  <div class="app">
    <h1>สวัสดี Vue!</h1>
    <p>{{ message }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const message = ref('ยินดีต้อนรับสู่ Vue 3')
</script>

<style scoped>
.app {
  text-align: center;
  padding: 20px;
}
</style>
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="th">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My Vue App</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

---

## Step 1453: Options API vs Composition API

```vue
<!-- Options API (Vue 2 style) -->
<template>
  <div>
    <p>{{ count }}</p>
    <p>{{ doubled }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
  </div>
</template>

<script>
export default {
  name: 'Counter',
  
  data() {
    return {
      count: 0,
      step: 1
    }
  },
  
  computed: {
    doubled() {
      return this.count * 2
    }
  },
  
  methods: {
    increment() {
      this.count += this.step
    },
    decrement() {
      this.count -= this.step
    }
  },
  
  mounted() {
    console.log('Component mounted')
  }
}
</script>
```

```vue
<!-- Composition API (Vue 3 แนะนำ) -->
<template>
  <div>
    <p>{{ count }}</p>
    <p>{{ doubled }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// Data
const count = ref(0)
const step = ref(1)

// Computed
const doubled = computed(() => count.value * 2)

// Methods
function increment() {
  count.value += step.value
}

function decrement() {
  count.value -= step.value
}

// Lifecycle
onMounted(() => {
  console.log('Component mounted')
})
</script>
```

---

## Step 1454: ref() และ reactive()

```vue
<template>
  <div>
    <!-- ref: ใช้ได้กับทุก type -->
    <p>count: {{ count }}</p>
    <p>name: {{ name }}</p>
    <p>items: {{ items.join(', ') }}</p>
    
    <!-- reactive: ใช้กับ objects/arrays -->
    <p>user: {{ user.name }} ({{ user.age }})</p>
    <p>state items: {{ state.items.length }}</p>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

// ref() - สำหรับ primitive values และ objects
const count = ref(0)
const name = ref('สมชาย')
const items = ref(['แอปเปิล', 'กล้วย'])

// เข้าถึงค่าด้วย .value (ใน script)
console.log(count.value)  // 0
count.value = 5  // อัพเดทค่า

// reactive() - สำหรับ objects (ไม่ต้อง .value)
const user = reactive({
  name: 'สมหญิง',
  age: 25,
  email: 'somying@example.com'
})

// เข้าถึงโดยตรง (ไม่มี .value)
console.log(user.name)  // 'สมหญิง'
user.age = 26  // อัพเดทโดยตรง

// reactive กับ nested objects - reactive อยู่ลึกได้
const state = reactive({
  items: [],
  filters: { category: 'all', search: '' },
  pagination: { page: 1, perPage: 10, total: 0 }
})

state.items.push({ id: 1, name: 'สินค้า' })
state.filters.search = 'vue'
</script>
```

```vue
<!-- ref vs reactive - ความแตกต่าง -->
<script setup>
import { ref, reactive } from 'vue'

// ref: destructure แล้วสูญเสีย reactivity
const state1 = ref({ count: 0, name: 'test' })
// ❌ ผิด
const { count } = state1.value  // count ไม่ reactive แล้ว

// reactive: destructure แล้วสูญเสีย reactivity
const state2 = reactive({ count: 0, name: 'test' })
// ❌ ผิด
const { count: count2 } = state2  // count2 ไม่ reactive แล้ว

// ✅ แก้ด้วย toRefs
import { toRefs } from 'vue'
const { count: count3, name } = toRefs(state2)
console.log(count3.value)  // ต้องใช้ .value
</script>
```

---

## Step 1455: computed() Properties

```vue
<template>
  <div>
    <h2>ร้านค้า</h2>
    
    <!-- computed ใช้ได้เหมือน data -->
    <p>จำนวนสินค้า: {{ productCount }}</p>
    <p>ราคารวม: ฿{{ totalPrice.toLocaleString() }}</p>
    <p>ราคาเฉลี่ย: ฿{{ averagePrice.toFixed(2) }}</p>
    <p>สินค้าแพงสุด: {{ mostExpensive?.name }}</p>
    
    <!-- Filter products -->
    <input v-model="search" placeholder="ค้นหา..." />
    <ul>
      <li v-for="product in filteredProducts" :key="product.id">
        {{ product.name }} - ฿{{ product.price }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const search = ref('')
const products = ref([
  { id: 1, name: 'MacBook', price: 89000, category: 'laptop' },
  { id: 2, name: 'iPhone', price: 35000, category: 'phone' },
  { id: 3, name: 'iPad', price: 25000, category: 'tablet' },
  { id: 4, name: 'AirPods', price: 8000, category: 'accessory' }
])

// Computed properties (cache ผล ไม่คำนวณใหม่ถ้า dependency ไม่เปลี่ยน)
const productCount = computed(() => products.value.length)

const totalPrice = computed(() =>
  products.value.reduce((sum, p) => sum + p.price, 0)
)

const averagePrice = computed(() =>
  products.value.length ? totalPrice.value / products.value.length : 0
)

const mostExpensive = computed(() =>
  [...products.value].sort((a, b) => b.price - a.price)[0]
)

const filteredProducts = computed(() => {
  if (!search.value.trim()) return products.value
  const query = search.value.toLowerCase()
  return products.value.filter(p =>
    p.name.toLowerCase().includes(query)
  )
})
</script>
```

```vue
<!-- Writable computed -->
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('สมชาย')
const lastName = ref('ใจดี')

// Computed getter + setter
const fullName = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(value) {
    const parts = value.split(' ')
    firstName.value = parts[0] || ''
    lastName.value = parts[1] || ''
  }
})

// ใช้งาน
fullName.value = 'สมหญิง รักดี'
console.log(firstName.value)  // 'สมหญิง'
console.log(lastName.value)   // 'รักดี'
</script>
```

---

## Step 1456: watch() และ watchEffect()

```vue
<script setup>
import { ref, watch, watchEffect } from 'vue'

const count = ref(0)
const name = ref('')
const user = ref({ name: '', email: '' })

// watch: ดู specific source
watch(count, (newValue, oldValue) => {
  console.log(`count เปลี่ยนจาก ${oldValue} ไป ${newValue}`)
})

// watch กับ options
watch(count, (newVal) => {
  console.log('count:', newVal)
}, {
  immediate: true,   // รันทันทีเมื่อ component mount
  deep: false        // ไม่ลึก (สำหรับ objects ใช้ deep: true)
})

// watch object แบบ deep
watch(user, (newVal) => {
  console.log('user เปลี่ยน:', newVal)
}, { deep: true })

// watch หลาย sources
watch([count, name], ([newCount, newName], [oldCount, oldName]) => {
  console.log('count:', newCount, 'name:', newName)
})

// watch getter function
watch(
  () => user.value.name,  // เฉพาะ user.name
  (newName) => {
    console.log('ชื่อเปลี่ยนเป็น:', newName)
  }
)

// watchEffect: อัตโนมัติ track dependencies
const page = ref(1)
const pageSize = ref(10)

watchEffect(async () => {
  // ติดตาม page และ pageSize อัตโนมัติ
  const response = await fetch(
    `/api/items?page=${page.value}&size=${pageSize.value}`
  )
  // ...
})

// Stop watcher
const stop = watchEffect(() => {
  console.log('effect:', count.value)
})
stop()  // หยุด watcher
</script>
```

```vue
<!-- watch สำหรับ fetch data -->
<template>
  <div>
    <select v-model="userId">
      <option v-for="id in [1, 2, 3]" :key="id" :value="id">
        ผู้ใช้ {{ id }}
      </option>
    </select>
    
    <div v-if="loading">กำลังโหลด...</div>
    <div v-else-if="error">{{ error }}</div>
    <div v-else-if="userData">
      <h3>{{ userData.name }}</h3>
      <p>{{ userData.email }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const userId = ref(1)
const userData = ref(null)
const loading = ref(false)
const error = ref(null)

watch(userId, async (newId) => {
  loading.value = true
  error.value = null
  
  try {
    const res = await fetch(`https://jsonplaceholder.typicode.com/users/${newId}`)
    userData.value = await res.json()
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
}, { immediate: true })
</script>
```

---

## Step 1457: Template Syntax

```vue
<template>
  <div>
    <!-- Text Interpolation -->
    <p>{{ message }}</p>
    <p>{{ count + 1 }}</p>
    <p>{{ isActive ? 'เปิดใช้' : 'ปิดใช้' }}</p>
    <p>{{ user.name.toUpperCase() }}</p>
    
    <!-- Raw HTML (ระวัง XSS!) -->
    <div v-html="rawHtml"></div>
    
    <!-- v-bind: bind attribute -->
    <img :src="imageSrc" :alt="imageAlt" />
    <div :class="dynamicClass"></div>
    <input :disabled="isDisabled" />
    
    <!-- Dynamic attribute name -->
    <button :[attributeName]="value">ปุ่ม</button>
    
    <!-- v-on: event handler -->
    <button @click="handleClick">คลิก</button>
    <input @input="handleInput" @keyup.enter="handleEnter" />
    
    <!-- v-model: two-way binding -->
    <input v-model="text" />
    <select v-model="selected"></select>
    <input type="checkbox" v-model="checked" />
  </div>
</template>

<script setup>
import { ref } from 'vue'

const message = ref('สวัสดี Vue!')
const count = ref(0)
const isActive = ref(true)
const user = ref({ name: 'สมชาย' })
const rawHtml = ref('<strong>ตัวหนา</strong>')
const imageSrc = ref('/avatar.jpg')
const imageAlt = ref('รูปโปรไฟล์')
const isDisabled = ref(false)
const attributeName = ref('title')
const value = ref('ข้อความ tooltip')
const text = ref('')
const selected = ref('')
const checked = ref(false)

function handleClick(event) {
  console.log('คลิก:', event.target)
  count.value++
}

function handleInput(event) {
  console.log('input:', event.target.value)
}

function handleEnter(event) {
  console.log('กด Enter:', event.target.value)
}
</script>
```

```vue
<!-- Class และ Style Bindings -->
<template>
  <div>
    <!-- Dynamic Class -->
    <!-- Object syntax -->
    <div :class="{ active: isActive, disabled: isDisabled }">Object syntax</div>
    
    <!-- Array syntax -->
    <div :class="[baseClass, isActive ? 'active' : '', extraClass]">Array syntax</div>
    
    <!-- Mixed -->
    <div :class="['btn', { 'btn-primary': isPrimary }]">Mixed</div>
    
    <!-- Dynamic Style -->
    <!-- Object syntax -->
    <div :style="{ color: textColor, fontSize: fontSize + 'px' }">Inline style</div>
    
    <!-- Object reference -->
    <div :style="styleObject">Style object</div>
    
    <!-- Array syntax -->
    <div :style="[baseStyles, additionalStyles]">Style array</div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const isActive = ref(true)
const isDisabled = ref(false)
const isPrimary = ref(true)
const baseClass = ref('container')
const extraClass = ref('highlight')
const textColor = ref('red')
const fontSize = ref(16)

const styleObject = reactive({
  backgroundColor: '#f5f5f5',
  padding: '10px',
  borderRadius: '4px'
})

const baseStyles = { margin: '10px' }
const additionalStyles = { padding: '20px', color: 'blue' }
</script>
```

---

## Step 1458: Directives

```vue
<!-- v-if, v-else-if, v-else -->
<template>
  <div>
    <div v-if="score >= 80">เกรด A</div>
    <div v-else-if="score >= 70">เกรด B</div>
    <div v-else-if="score >= 60">เกรด C</div>
    <div v-else>เกรด F</div>
    
    <!-- v-show: แสดง/ซ่อนด้วย CSS (ไม่ destroy element) -->
    <div v-show="isVisible">เห็นฉันมั้ย</div>
    
    <!-- v-if vs v-show:
         v-if: destroy/create element จริง (เหมาะกับเงื่อนไขที่เปลี่ยนน้อย)
         v-show: toggle display:none (เหมาะกับเงื่อนไขที่เปลี่ยนบ่อย) -->
  </div>
</template>

<script setup>
import { ref } from 'vue'

const score = ref(75)
const isVisible = ref(true)
</script>
```

```vue
<!-- v-for: การ render list -->
<template>
  <div>
    <!-- Array -->
    <ul>
      <li v-for="(item, index) in items" :key="item.id">
        {{ index + 1 }}. {{ item.name }}
      </li>
    </ul>
    
    <!-- Object -->
    <div v-for="(value, key, index) in person" :key="key">
      {{ index }}. {{ key }}: {{ value }}
    </div>
    
    <!-- Range -->
    <span v-for="n in 5" :key="n">{{ n }} </span>
    
    <!-- Nested v-for -->
    <div v-for="category in categories" :key="category.id">
      <h3>{{ category.name }}</h3>
      <ul>
        <li v-for="product in category.products" :key="product.id">
          {{ product.name }}
        </li>
      </ul>
    </div>
    
    <!-- v-for กับ v-if (ไม่แนะนำบน element เดียวกัน) -->
    <!-- ❌ ไม่ดี -->
    <li v-for="user in users" v-if="user.active" :key="user.id">
      {{ user.name }}
    </li>
    
    <!-- ✅ ดี: filter ก่อน -->
    <li v-for="user in activeUsers" :key="user.id">{{ user.name }}</li>
    <!-- หรือใช้ computed -->
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const items = ref([
  { id: 1, name: 'แอปเปิล' },
  { id: 2, name: 'กล้วย' },
  { id: 3, name: 'ส้ม' }
])

const person = ref({
  name: 'สมชาย',
  age: 25,
  city: 'กรุงเทพ'
})

const categories = ref([
  {
    id: 1,
    name: 'ผลไม้',
    products: [
      { id: 1, name: 'มะม่วง' },
      { id: 2, name: 'สับปะรด' }
    ]
  },
  {
    id: 2,
    name: 'ผัก',
    products: [
      { id: 3, name: 'กะหล่ำปลี' },
      { id: 4, name: 'แตงกวา' }
    ]
  }
])

const users = ref([
  { id: 1, name: 'สมชาย', active: true },
  { id: 2, name: 'สมหญิง', active: false },
  { id: 3, name: 'สมศักดิ์', active: true }
])

const activeUsers = computed(() => users.value.filter(u => u.active))
</script>
```

```vue
<!-- v-model เชิงลึก -->
<template>
  <div>
    <!-- text input -->
    <input v-model="text" type="text" />
    
    <!-- modifiers -->
    <input v-model.trim="trimmedText" />      <!-- trim whitespace -->
    <input v-model.number="numValue" type="number" />  <!-- convert to number -->
    <input v-model.lazy="lazyText" />          <!-- update on change not input -->
    
    <!-- checkbox -->
    <input v-model="checked" type="checkbox" />
    
    <!-- checkbox กับ array -->
    <input v-model="selectedFruits" type="checkbox" value="apple" />
    <input v-model="selectedFruits" type="checkbox" value="banana" />
    
    <!-- radio -->
    <input v-model="gender" type="radio" value="male" /> ชาย
    <input v-model="gender" type="radio" value="female" /> หญิง
    
    <!-- select -->
    <select v-model="country">
      <option value="">เลือกประเทศ</option>
      <option value="th">ไทย</option>
      <option value="us">สหรัฐอเมริกา</option>
    </select>
    
    <!-- textarea -->
    <textarea v-model="bio"></textarea>
    
    <pre>{{ JSON.stringify({ text, checked, selectedFruits, gender, country }, null, 2) }}</pre>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const text = ref('')
const trimmedText = ref('')
const numValue = ref(0)
const lazyText = ref('')
const checked = ref(false)
const selectedFruits = ref([])
const gender = ref('')
const country = ref('')
const bio = ref('')
</script>
```

---

## Step 1459: Components ใน Vue

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="child">
    <h3>{{ title }}</h3>
    <p>{{ description }}</p>
    <slot>เนื้อหา default</slot>
  </div>
</template>

<script setup>
const props = defineProps({
  title: String,
  description: {
    type: String,
    default: 'ไม่มีคำอธิบาย'
  }
})
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <div>
    <ChildComponent
      title="หัวข้อจาก Parent"
      description="คำอธิบายจาก Parent"
    />
    
    <!-- ส่ง dynamic values -->
    <ChildComponent
      :title="dynamicTitle"
      :description="dynamicDesc"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const dynamicTitle = ref('หัวข้อ Dynamic')
const dynamicDesc = ref('คำอธิบาย Dynamic')
</script>
```

```vue
<!-- การ import และใช้ component -->
<template>
  <div>
    <!-- Auto-import (Vite + @vitejs/plugin-vue) -->
    <Button variant="primary" @click="handleClick">คลิก</Button>
    <Card title="บัตรข้อมูล">
      <p>เนื้อหา</p>
    </Card>
    <Modal v-if="showModal" @close="showModal = false">
      <h2>Modal</h2>
    </Modal>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Button from './components/Button.vue'
import Card from './components/Card.vue'
import Modal from './components/Modal.vue'

const showModal = ref(false)

function handleClick() {
  showModal.value = true
}
</script>
```

---

## Step 1460: Props ใน Vue

```vue
<!-- UserCard.vue -->
<template>
  <div class="user-card">
    <img :src="avatar" :alt="name" />
    <h2>{{ name }}</h2>
    <p>{{ role }}</p>
    <p v-if="email">{{ email }}</p>
    <div v-if="skills.length">
      <span v-for="skill in skills" :key="skill" class="tag">
        {{ skill }}
      </span>
    </div>
    <p>สถานะ: {{ isActive ? 'ใช้งาน' : 'ไม่ใช้งาน' }}</p>
  </div>
</template>

<script setup>
// defineProps - กำหนด props
const props = defineProps({
  name: {
    type: String,
    required: true
  },
  avatar: {
    type: String,
    default: 'https://via.placeholder.com/100'
  },
  role: {
    type: String,
    default: 'user',
    validator: (value) => ['admin', 'user', 'moderator'].includes(value)
  },
  email: String,
  skills: {
    type: Array,
    default: () => []  // ต้องใช้ factory function สำหรับ array/object
  },
  isActive: {
    type: Boolean,
    default: true
  },
  score: {
    type: Number,
    validator: (value) => value >= 0 && value <= 100
  }
})

// ใช้ props ใน script
console.log(props.name)
</script>
```

```vue
<!-- การใช้ Props ใน Parent -->
<template>
  <UserCard
    name="สมชาย ใจดี"
    role="admin"
    email="somchai@example.com"
    :skills="['Vue', 'React', 'Node.js']"
    :is-active="true"
    :score="95"
  />
</template>
```

---

## Step 1461: Emits ใน Vue

```vue
<!-- FormInput.vue -->
<template>
  <div class="form-input">
    <label :for="id">{{ label }}</label>
    <input
      :id="id"
      :type="type"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      @blur="$emit('blur', $event)"
      @focus="$emit('focus', $event)"
    />
    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>

<script setup>
// defineEmits - กำหนด events ที่ emit ได้
const emit = defineEmits({
  // Event แบบไม่ validate
  blur: null,
  focus: null,
  
  // Event แบบ validate
  'update:modelValue': (value) => {
    return typeof value === 'string'
  },
  
  submit: (data) => {
    return data !== null && typeof data === 'object'
  }
})

const props = defineProps({
  modelValue: String,
  label: String,
  type: { type: String, default: 'text' },
  id: String,
  error: String
})

function handleSubmit(data) {
  emit('submit', data)
}
</script>
```

```vue
<!-- Parent ใช้ FormInput -->
<template>
  <div>
    <!-- v-model ทำงานกับ component ที่ emit 'update:modelValue' -->
    <FormInput
      v-model="email"
      label="อีเมล"
      type="email"
      id="email"
      :error="emailError"
      @blur="validateEmail"
    />
    
    <!-- v-model กับ multiple bindings (Vue 3) -->
    <CustomInput
      v-model:firstName="firstName"
      v-model:lastName="lastName"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import FormInput from './FormInput.vue'

const email = ref('')
const emailError = ref('')
const firstName = ref('')
const lastName = ref('')

function validateEmail() {
  if (!email.value.includes('@')) {
    emailError.value = 'รูปแบบอีเมลไม่ถูกต้อง'
  } else {
    emailError.value = ''
  }
}
</script>
```

---

## Step 1462: Slots

```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <!-- Named slot: header -->
    <div class="card-header">
      <slot name="header">
        <h3>หัวข้อ Default</h3>
      </slot>
    </div>
    
    <!-- Default slot -->
    <div class="card-body">
      <slot></slot>
    </div>
    
    <!-- Named slot: footer -->
    <div class="card-footer">
      <slot name="footer">
        <p>ส่วนท้าย Default</p>
      </slot>
    </div>
  </div>
</template>
```

```vue
<!-- การใช้ Named Slots -->
<template>
  <Card>
    <template #header>
      <h2>หัวข้อพิเศษ</h2>
      <span class="badge">ใหม่</span>
    </template>
    
    <!-- default slot -->
    <p>เนื้อหาของบัตร</p>
    <ul>
      <li>รายการ 1</li>
      <li>รายการ 2</li>
    </ul>
    
    <template #footer>
      <button>บันทึก</button>
      <button>ยกเลิก</button>
    </template>
  </Card>
</template>
```

```vue
<!-- Scoped Slots - ส่งข้อมูลจาก child ไป slot -->
<!-- DataTable.vue -->
<template>
  <table>
    <thead>
      <tr>
        <th v-for="col in columns" :key="col.key">{{ col.label }}</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="row in data" :key="row.id">
        <td v-for="col in columns" :key="col.key">
          <!-- Scoped slot: ส่ง row และ col ไปให้ parent จัดการ -->
          <slot :name="col.key" :row="row" :value="row[col.key]">
            {{ row[col.key] }}
          </slot>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script setup>
defineProps({
  columns: Array,
  data: Array
})
</script>
```

```vue
<!-- ใช้ DataTable กับ scoped slots -->
<template>
  <DataTable :columns="columns" :data="users">
    <!-- Override column 'status' -->
    <template #status="{ row, value }">
      <span :class="value === 'active' ? 'green' : 'red'">
        {{ value === 'active' ? 'ใช้งาน' : 'ไม่ใช้งาน' }}
      </span>
    </template>
    
    <!-- Override column 'actions' -->
    <template #actions="{ row }">
      <button @click="editUser(row)">แก้ไข</button>
      <button @click="deleteUser(row.id)">ลบ</button>
    </template>
  </DataTable>
</template>
```

---

## Step 1463: Lifecycle Hooks

```vue
<script setup>
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onErrorCaptured,
  ref
} from 'vue'

const count = ref(0)
const data = ref(null)

// onBeforeMount: ก่อน DOM render
onBeforeMount(() => {
  console.log('ก่อน mount - DOM ยังไม่มี')
})

// onMounted: หลัง DOM render (ใช้บ่อยที่สุด)
onMounted(async () => {
  console.log('Mounted - DOM พร้อมใช้งาน')
  
  // Fetch data
  const res = await fetch('https://jsonplaceholder.typicode.com/posts/1')
  data.value = await res.json()
  
  // Access DOM
  const el = document.querySelector('.my-element')
  console.log(el)
})

// onBeforeUpdate: ก่อน re-render
onBeforeUpdate(() => {
  console.log('ก่อน update')
})

// onUpdated: หลัง re-render
onUpdated(() => {
  console.log('อัพเดทแล้ว')
})

// onBeforeUnmount: ก่อน unmount
onBeforeUnmount(() => {
  console.log('ก่อน unmount - cleanup ได้เลย')
  // ล้าง event listeners, timers ที่นี่
})

// onUnmounted: หลัง unmount
onUnmounted(() => {
  console.log('Unmounted แล้ว')
})

// onErrorCaptured: ดัก error จาก child
onErrorCaptured((error, instance, info) => {
  console.error('เกิด error:', error, info)
  return false  // ป้องกัน error propagation
})
</script>
```

---

## Step 1464: Pinia สำหรับ State Management

```bash
npm install pinia
```

```javascript
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)
const pinia = createPinia()

app.use(pinia)
app.mount('#app')
```

```javascript
// stores/counter.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

// Composition API style (แนะนำ)
export const useCounterStore = defineStore('counter', () => {
  // State
  const count = ref(0)
  const step = ref(1)
  
  // Getters (computed)
  const doubled = computed(() => count.value * 2)
  const isPositive = computed(() => count.value > 0)
  
  // Actions
  function increment() {
    count.value += step.value
  }
  
  function decrement() {
    count.value -= step.value
  }
  
  function reset() {
    count.value = 0
  }
  
  function setStep(value) {
    step.value = value
  }
  
  return { count, step, doubled, isPositive, increment, decrement, reset, setStep }
})
```

```javascript
// stores/cart.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCartStore = defineStore('cart', () => {
  const items = ref([])
  
  const totalItems = computed(() =>
    items.value.reduce((sum, item) => sum + item.quantity, 0)
  )
  
  const total = computed(() =>
    items.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
  )
  
  function addItem(product) {
    const existing = items.value.find(item => item.id === product.id)
    if (existing) {
      existing.quantity++
    } else {
      items.value.push({ ...product, quantity: 1 })
    }
  }
  
  function removeItem(id) {
    const index = items.value.findIndex(item => item.id === id)
    if (index !== -1) items.value.splice(index, 1)
  }
  
  function updateQuantity(id, quantity) {
    const item = items.value.find(item => item.id === id)
    if (item) {
      if (quantity <= 0) removeItem(id)
      else item.quantity = quantity
    }
  }
  
  function clearCart() {
    items.value = []
  }
  
  return { items, totalItems, total, addItem, removeItem, updateQuantity, clearCart }
}, {
  persist: true  // npm install pinia-plugin-persistedstate
})
```

```vue
<!-- ใช้ Pinia store ใน component -->
<template>
  <div>
    <p>นับ: {{ counter.count }}</p>
    <p>คูณสอง: {{ counter.doubled }}</p>
    <button @click="counter.increment()">+</button>
    <button @click="counter.decrement()">-</button>
    
    <hr />
    
    <p>ตะกร้า: {{ cart.totalItems }} ชิ้น</p>
    <p>รวม: ฿{{ cart.total.toLocaleString() }}</p>
    <button @click="cart.addItem({ id: 1, name: 'สินค้า', price: 100 })">
      เพิ่มสินค้า
    </button>
  </div>
</template>

<script setup>
import { useCounterStore } from './stores/counter'
import { useCartStore } from './stores/cart'

const counter = useCounterStore()
const cart = useCartStore()
</script>
```

---

## Step 1465: Vue Router Basics

```bash
npm install vue-router@4
```

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'
import AboutView from '../views/AboutView.vue'
import UserView from '../views/UserView.vue'

const routes = [
  {
    path: '/',
    name: 'home',
    component: HomeView
  },
  {
    path: '/about',
    name: 'about',
    component: AboutView,
    meta: { requiresAuth: false }
  },
  {
    // Dynamic route
    path: '/users/:id',
    name: 'user',
    component: UserView,
    props: true  // ส่ง params เป็น props
  },
  {
    // Nested routes
    path: '/admin',
    component: () => import('../views/AdminLayout.vue'),
    meta: { requiresAuth: true },
    children: [
      {
        path: '',  // /admin
        component: () => import('../views/admin/Dashboard.vue')
      },
      {
        path: 'users',  // /admin/users
        component: () => import('../views/admin/Users.vue')
      }
    ]
  },
  {
    // Catch-all
    path: '/:pathMatch(.*)*',
    name: '404',
    component: () => import('../views/NotFound.vue')
  }
]

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes
})

// Navigation Guard
router.beforeEach((to, from, next) => {
  const isAuthenticated = localStorage.getItem('token')
  
  if (to.meta.requiresAuth && !isAuthenticated) {
    next({ name: 'login', query: { redirect: to.fullPath } })
  } else {
    next()
  }
})

export default router
```

```javascript
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import router from './router'
import App from './App.vue'

createApp(App)
  .use(createPinia())
  .use(router)
  .mount('#app')
```

```vue
<!-- App.vue -->
<template>
  <div>
    <!-- Navigation -->
    <nav>
      <RouterLink to="/">หน้าแรก</RouterLink>
      <RouterLink to="/about">เกี่ยวกับ</RouterLink>
      <RouterLink :to="{ name: 'user', params: { id: 1 } }">ผู้ใช้ 1</RouterLink>
    </nav>
    
    <!-- Router View -->
    <RouterView />
  </div>
</template>
```

```vue
<!-- Views/UserView.vue -->
<template>
  <div>
    <h1>ผู้ใช้ #{{ id }}</h1>
    <button @click="goBack">ย้อนกลับ</button>
    <button @click="goToUser(2)">ผู้ใช้ 2</button>
  </div>
</template>

<script setup>
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

// รับ params จาก URL
const id = route.params.id
const { page, search } = route.query  // query string

function goBack() {
  router.back()
}

function goToUser(userId) {
  router.push({ name: 'user', params: { id: userId } })
}

function goHome() {
  router.push('/')
  // หรือ
  router.push({ name: 'home' })
}
</script>
```

---

## Step 1466: Composables (Vue Custom Hooks)

```javascript
// composables/useFetch.js
import { ref, watch, toValue } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const loading = ref(false)
  const error = ref(null)
  
  async function fetchData() {
    const resolvedUrl = toValue(url)  // รองรับทั้ง ref และ plain value
    if (!resolvedUrl) return
    
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch(resolvedUrl)
      if (!response.ok) throw new Error(`HTTP ${response.status}`)
      data.value = await response.json()
    } catch (e) {
      error.value = e.message
    } finally {
      loading.value = false
    }
  }
  
  // Watch URL changes
  watch(
    () => toValue(url),
    fetchData,
    { immediate: true }
  )
  
  return { data, loading, error, refetch: fetchData }
}
```

```javascript
// composables/useLocalStorage.js
import { ref, watch } from 'vue'

export function useLocalStorage(key, defaultValue) {
  const storedValue = localStorage.getItem(key)
  const value = ref(storedValue ? JSON.parse(storedValue) : defaultValue)
  
  watch(value, (newValue) => {
    localStorage.setItem(key, JSON.stringify(newValue))
  }, { deep: true })
  
  return value
}
```

```javascript
// composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0, options = {}) {
  const { min = -Infinity, max = Infinity, step = 1 } = options
  
  const count = ref(initialValue)
  const isAtMin = computed(() => count.value <= min)
  const isAtMax = computed(() => count.value >= max)
  
  function increment() {
    count.value = Math.min(count.value + step, max)
  }
  
  function decrement() {
    count.value = Math.max(count.value - step, min)
  }
  
  function reset() {
    count.value = initialValue
  }
  
  function set(value) {
    count.value = Math.min(Math.max(value, min), max)
  }
  
  return { count, isAtMin, isAtMax, increment, decrement, reset, set }
}
```

```javascript
// composables/useMousePosition.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useMousePosition() {
  const x = ref(0)
  const y = ref(0)
  
  function update(event) {
    x.value = event.clientX
    y.value = event.clientY
  }
  
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))
  
  return { x, y }
}
```

```vue
<!-- ใช้งาน Composables -->
<template>
  <div>
    <p>เมาส์: {{ x }}, {{ y }}</p>
    
    <div v-if="loading">กำลังโหลด...</div>
    <div v-else-if="error">Error: {{ error }}</div>
    <div v-else>{{ data?.name }}</div>
    
    <p>นับ: {{ count }}</p>
    <button @click="increment" :disabled="isAtMax">+</button>
    <button @click="decrement" :disabled="isAtMin">-</button>
    
    <p>ธีม: {{ theme }}</p>
    <button @click="theme = theme === 'light' ? 'dark' : 'light'">Toggle</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useFetch } from './composables/useFetch'
import { useMousePosition } from './composables/useMousePosition'
import { useCounter } from './composables/useCounter'
import { useLocalStorage } from './composables/useLocalStorage'

const userId = ref(1)
const { data, loading, error } = useFetch(
  () => `https://jsonplaceholder.typicode.com/users/${userId.value}`
)

const { x, y } = useMousePosition()

const { count, isAtMin, isAtMax, increment, decrement } = useCounter(0, {
  min: 0, max: 10
})

const theme = useLocalStorage('theme', 'light')
</script>
```

---

## Step 1467: Provide / Inject

```vue
<!-- GrandParent.vue -->
<script setup>
import { provide, ref, readonly } from 'vue'

const theme = ref('light')
const user = ref({ name: 'สมชาย', role: 'admin' })

// provide ค่าให้ descendants ทั้งหมด
provide('theme', readonly(theme))  // readonly ป้องกัน mutation
provide('user', user)
provide('toggleTheme', () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
})
</script>
```

```vue
<!-- DeepChild.vue (ลูกหลาน) -->
<template>
  <div :class="`container ${theme}`">
    <p>ผู้ใช้: {{ user.name }}</p>
    <button @click="toggleTheme">Toggle Theme</button>
  </div>
</template>

<script setup>
import { inject } from 'vue'

const theme = inject('theme', 'light')  // default value
const user = inject('user')
const toggleTheme = inject('toggleTheme')
</script>
```

---

## Step 1468: Template Refs

```vue
<template>
  <div>
    <!-- ref บน element -->
    <input ref="inputRef" type="text" />
    <canvas ref="canvasRef"></canvas>
    
    <!-- ref บน component -->
    <ChildComponent ref="childRef" />
    
    <!-- ref ใน v-for -->
    <li v-for="item in items" :key="item.id" :ref="el => setRef(el, item.id)">
      {{ item.name }}
    </li>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ChildComponent from './ChildComponent.vue'

// สร้าง ref สำหรับ DOM elements
const inputRef = ref(null)
const canvasRef = ref(null)
const childRef = ref(null)
const itemRefs = ref({})

function setRef(el, id) {
  if (el) itemRefs.value[id] = el
}

onMounted(() => {
  // focus input
  inputRef.value?.focus()
  
  // access canvas
  const ctx = canvasRef.value?.getContext('2d')
  
  // call method on child
  childRef.value?.someMethod()
})

const items = ref([
  { id: 1, name: 'รายการ 1' },
  { id: 2, name: 'รายการ 2' }
])
</script>
```

---

## Step 1469: Teleport และ Suspense

```vue
<!-- Teleport: render element ที่ตำแหน่งอื่นใน DOM -->
<template>
  <div>
    <button @click="showModal = true">เปิด Modal</button>
    
    <!-- Teleport ไปที่ body -->
    <Teleport to="body">
      <div v-if="showModal" class="modal-overlay">
        <div class="modal">
          <h2>Modal</h2>
          <p>เนื้อหา modal</p>
          <button @click="showModal = false">ปิด</button>
        </div>
      </div>
    </Teleport>
    
    <!-- Teleport ไปที่ element อื่น -->
    <Teleport to="#notifications">
      <div class="notification">แจ้งเตือน!</div>
    </Teleport>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const showModal = ref(false)
</script>
```

```vue
<!-- Suspense: รอ async components -->
<template>
  <Suspense>
    <!-- Content to show when resolved -->
    <template #default>
      <AsyncDataComponent />
    </template>
    
    <!-- Fallback while loading -->
    <template #fallback>
      <div>กำลังโหลด...</div>
    </template>
  </Suspense>
</template>
```

```vue
<!-- AsyncDataComponent.vue -->
<script setup>
// async setup() จะทำให้ Suspense ทำงาน
const response = await fetch('https://jsonplaceholder.typicode.com/posts/1')
const post = await response.json()
</script>

<template>
  <div>
    <h2>{{ post.title }}</h2>
    <p>{{ post.body }}</p>
  </div>
</template>
```

---

## Step 1470: Complete Vue App Example

```vue
<!-- App.vue - Complete Example -->
<template>
  <div :class="['app', theme]">
    <header>
      <h1>📝 Vue Todo App</h1>
      <button @click="toggleTheme" class="theme-btn">
        {{ theme === 'light' ? '🌙' : '☀️' }}
      </button>
    </header>
    
    <main>
      <!-- Add Todo Form -->
      <form @submit.prevent="addTodo" class="add-form">
        <input
          v-model.trim="newTodo"
          placeholder="เพิ่มรายการ..."
          :disabled="isLoading"
        />
        <select v-model="newCategory">
          <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
        </select>
        <button type="submit" :disabled="!newTodo || isLoading">เพิ่ม</button>
      </form>
      
      <!-- Filters -->
      <div class="filters">
        <button
          v-for="filter in ['all', 'active', 'done']"
          :key="filter"
          :class="{ active: currentFilter === filter }"
          @click="currentFilter = filter"
        >
          {{ filterLabels[filter] }}
          <span class="count">{{ filterCounts[filter] }}</span>
        </button>
      </div>
      
      <!-- Todo List -->
      <TransitionGroup name="todo-list" tag="ul" class="todo-list">
        <li
          v-for="todo in filteredTodos"
          :key="todo.id"
          :class="{ done: todo.done }"
        >
          <input
            type="checkbox"
            :checked="todo.done"
            @change="toggleTodo(todo.id)"
          />
          
          <span v-if="editingId !== todo.id" @dblclick="startEdit(todo)">
            {{ todo.text }}
          </span>
          <input
            v-else
            :ref="el => { if (el) el.focus() }"
            :value="todo.text"
            @blur="finishEdit(todo.id, $event.target.value)"
            @keyup.enter="finishEdit(todo.id, $event.target.value)"
            @keyup.escape="editingId = null"
          />
          
          <span class="category">{{ todo.category }}</span>
          <button @click="deleteTodo(todo.id)" class="delete-btn">✕</button>
        </li>
      </TransitionGroup>
      
      <!-- Empty State -->
      <div v-if="filteredTodos.length === 0" class="empty">
        {{ currentFilter === 'all' ? 'ยังไม่มีรายการ' : 'ไม่มีรายการที่ตรงกัน' }}
      </div>
      
      <!-- Footer -->
      <footer v-if="todos.length > 0">
        <span>เหลือ {{ activeCount }} รายการ</span>
        <button
          v-if="doneCount > 0"
          @click="clearDone"
          class="clear-btn"
        >
          ล้างรายการที่เสร็จ ({{ doneCount }})
        </button>
      </footer>
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from './composables/useLocalStorage'

// State
const todos = useLocalStorage('vue-todos', [])
const newTodo = ref('')
const newCategory = ref('ทั่วไป')
const currentFilter = ref('all')
const editingId = ref(null)
const isLoading = ref(false)
const theme = useLocalStorage('theme', 'light')

const categories = ['ทั่วไป', 'งาน', 'ส่วนตัว', 'ซื้อของ']

const filterLabels = {
  all: 'ทั้งหมด',
  active: 'ยังไม่เสร็จ',
  done: 'เสร็จแล้ว'
}

// Computed
const filteredTodos = computed(() => {
  switch (currentFilter.value) {
    case 'active': return todos.value.filter(t => !t.done)
    case 'done': return todos.value.filter(t => t.done)
    default: return todos.value
  }
})

const activeCount = computed(() => todos.value.filter(t => !t.done).length)
const doneCount = computed(() => todos.value.filter(t => t.done).length)

const filterCounts = computed(() => ({
  all: todos.value.length,
  active: activeCount.value,
  done: doneCount.value
}))

// Methods
function addTodo() {
  if (!newTodo.value.trim()) return
  todos.value.push({
    id: Date.now(),
    text: newTodo.value.trim(),
    category: newCategory.value,
    done: false,
    createdAt: new Date().toISOString()
  })
  newTodo.value = ''
}

function toggleTodo(id) {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.done = !todo.done
}

function deleteTodo(id) {
  todos.value = todos.value.filter(t => t.id !== id)
}

function startEdit(todo) {
  editingId.value = todo.id
}

function finishEdit(id, text) {
  const todo = todos.value.find(t => t.id === id)
  if (todo && text.trim()) {
    todo.text = text.trim()
  } else if (todo && !text.trim()) {
    deleteTodo(id)
  }
  editingId.value = null
}

function clearDone() {
  todos.value = todos.value.filter(t => !t.done)
}

function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}
</script>

<style scoped>
.todo-list-enter-active,
.todo-list-leave-active {
  transition: all 0.3s ease;
}
.todo-list-enter-from,
.todo-list-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}
</style>
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Profile Page
สร้างหน้า user profile ด้วย Vue ที่มี:
- แสดงข้อมูล user
- แก้ไขชื่อและ bio ได้
- อัพโหลดรูปโปรไฟล์
- แสดง list โพสต์ของ user

### แบบฝึกหัดที่ 2: Shopping Cart
สร้าง shopping app ด้วย Pinia ที่มี:
- Product list (fetch จาก API)
- เพิ่ม/ลบสินค้า
- ใส่ coupon code
- checkout form

### แบบฝึกหัดที่ 3: Weather App
สร้าง weather app ที่:
- ค้นหาเมือง
- แสดงอากาศวันนี้
- แสดงพยากรณ์ 5 วัน
- บันทึกเมืองที่ค้นหาล่าสุด

### แบบฝึกหัดที่ 4: Chat App
สร้าง mock chat app ที่มี:
- รายชื่อสนทนา
- แสดงข้อความ
- ส่งข้อความ
- animation

### แบบฝึกหัดที่ 5: Blog CMS
สร้าง mini blog CMS ด้วย Vue Router ที่มี:
- หน้า list โพสต์ (ดึงจาก API)
- หน้า detail โพสต์
- หน้า search
- Navigation

---

## สรุปท้ายส่วน

ใน Part 74 นี้เราได้เรียนรู้:

- **Vue.js คืออะไร**: progressive framework, component-based
- **Setup**: ใช้ Vite สร้างโปรเจค Vue 3
- **Options vs Composition API**: Composition API เหมาะสำหรับ Vue 3
- **ref() และ reactive()**: reactive state management
- **computed()**: derived values ที่ cache ไว้
- **watch() และ watchEffect()**: ตอบสนองต่อการเปลี่ยนแปลง
- **Template Syntax**: interpolation, v-bind, v-on, v-model
- **Directives**: v-if, v-show, v-for, v-model, v-slot
- **Components**: Props, Emits, Slots
- **Pinia**: state management สำหรับ Vue
- **Vue Router**: routing ใน Vue
- **Composables**: custom hooks สำหรับ Vue

Part 75 จะเรียน Next.js!
