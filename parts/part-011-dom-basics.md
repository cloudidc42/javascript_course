# Part 11: DOM Manipulation พื้นฐาน (Steps 191-210)

## บทนำ

DOM (Document Object Model) คือ API ที่ทำให้ JavaScript สามารถเข้าถึงและแก้ไข HTML Document ได้ เมื่อ Browser โหลด HTML จะสร้าง DOM Tree ที่เป็น Tree Structure แสดงโครงสร้างของ Document ทำให้เราสามารถ:

- อ่านและแก้ไขเนื้อหาใน HTML
- เพิ่มหรือลบ Elements
- เปลี่ยน Style และ Attributes
- ตอบสนองต่อ User Interactions

---

## Step 191: What is the DOM (Document Object Model)

DOM คือ Interface ที่แสดง HTML Document ในรูปแบบ Tree ของ Objects โดยแต่ละ Node ใน Tree คือ Object ที่มี Properties และ Methods

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <title>DOM คืออะไร</title>
</head>
<body>
  <h1 id="title">สวัสดี DOM</h1>
  <p class="intro">นี่คือบทเรียน DOM</p>
  <script>
    // document คือ Entry Point สู่ DOM
    console.log(document); // HTMLDocument object

    // document.documentElement คือ <html>
    console.log(document.documentElement); // <html>

    // document.head คือ <head>
    console.log(document.head); // <head>

    // document.body คือ <body>
    console.log(document.body); // <body>

    // document.title คือ title ของ page
    console.log(document.title); // "DOM คืออะไร"

    // document.URL คือ URL ปัจจุบัน
    console.log(document.URL);

    // document.domain
    console.log(document.domain);
  </script>
</body>
</html>
```

### ประเภทของ Node

```javascript
// Node Types
console.log(Node.ELEMENT_NODE);       // 1 - element เช่น <div>
console.log(Node.TEXT_NODE);          // 3 - text content
console.log(Node.COMMENT_NODE);       // 8 - <!-- comment -->
console.log(Node.DOCUMENT_NODE);      // 9 - document
console.log(Node.DOCUMENT_TYPE_NODE); // 10 - <!DOCTYPE>
console.log(Node.DOCUMENT_FRAGMENT_NODE); // 11 - fragment

// ตรวจสอบประเภทของ Node
const h1 = document.querySelector('h1');
console.log(h1.nodeType);    // 1 (ELEMENT_NODE)
console.log(h1.nodeName);    // "H1"
console.log(h1.nodeValue);   // null (สำหรับ element nodes)

const textNode = h1.firstChild;
console.log(textNode.nodeType);  // 3 (TEXT_NODE)
console.log(textNode.nodeName);  // "#text"
console.log(textNode.nodeValue); // "สวัสดี DOM"
```

---

## Step 192: DOM Tree Structure

DOM Tree มีโครงสร้างแบบ Tree โดยมี document เป็น Root

```html
<!DOCTYPE html>
<html>
<head>
  <title>Tree Structure</title>
</head>
<body>
  <div id="container">
    <h1>หัวข้อ</h1>
    <p>ย่อหน้า <span>พิเศษ</span></p>
    <ul>
      <li>รายการที่ 1</li>
      <li>รายการที่ 2</li>
    </ul>
  </div>
  <script>
    // แสดง DOM Tree structure
    function showTree(node, indent = 0) {
      const spaces = '  '.repeat(indent);
      if (node.nodeType === Node.TEXT_NODE) {
        const text = node.nodeValue.trim();
        if (text) console.log(`${spaces}[Text: "${text}"]`);
      } else if (node.nodeType === Node.ELEMENT_NODE) {
        console.log(`${spaces}<${node.nodeName.toLowerCase()}>`);
        node.childNodes.forEach(child => showTree(child, indent + 1));
      }
    }

    showTree(document.getElementById('container'));
    // Output:
    // <div>
    //   <h1>
    //     [Text: "หัวข้อ"]
    //   <p>
    //     [Text: "ย่อหน้า "]
    //     <span>
    //       [Text: "พิเศษ"]
    //   <ul>
    //     <li>
    //       [Text: "รายการที่ 1"]
    //     <li>
    //       [Text: "รายการที่ 2"]
  </script>
</body>
</html>
```

### Relationship ใน DOM Tree

```javascript
// Parent - Child - Sibling relationships
const container = document.getElementById('container');
const h1 = document.querySelector('h1');
const p = document.querySelector('p');
const ul = document.querySelector('ul');
const li1 = document.querySelectorAll('li')[0];
const li2 = document.querySelectorAll('li')[1];

// Parent
console.log(h1.parentNode);    // <div id="container">
console.log(h1.parentElement); // <div id="container">

// Children
console.log(container.childNodes);  // NodeList [text, h1, text, p, text, ul, text]
console.log(container.children);    // HTMLCollection [h1, p, ul]

// Siblings
console.log(h1.nextSibling);        // text node (\n)
console.log(h1.nextElementSibling); // <p>
console.log(p.previousSibling);     // text node (\n)
console.log(p.previousElementSibling); // <h1>

// First / Last
console.log(container.firstChild);        // text node
console.log(container.firstElementChild); // <h1>
console.log(container.lastChild);         // text node
console.log(container.lastElementChild);  // <ul>
```

---

## Step 193: getElementById

วิธีที่เร็วที่สุดในการเลือก Element ด้วย ID

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="main-header">
    <h1 id="page-title">ยินดีต้อนรับ</h1>
    <nav id="navigation">
      <a href="#">หน้าแรก</a>
      <a href="#">เกี่ยวกับ</a>
    </nav>
  </div>

  <section id="content">
    <p id="intro-text">นี่คือ introduction</p>
  </section>

  <script>
    // getElementById - คืนค่า Element เดียว หรือ null
    const header = document.getElementById('main-header');
    console.log(header); // <div id="main-header">

    const title = document.getElementById('page-title');
    console.log(title);        // <h1 id="page-title">
    console.log(title.id);     // "page-title"
    console.log(title.tagName); // "H1"

    // ถ้าไม่พบ คืนค่า null
    const notFound = document.getElementById('non-existent');
    console.log(notFound); // null

    // ป้องกัน null error ด้วย optional chaining
    console.log(notFound?.textContent); // undefined (ไม่ error)

    // แก้ไขค่าผ่าน Element ที่เลือกได้
    title.textContent = 'ยินดีต้อนรับสู่ JavaScript';
    title.style.color = 'blue';

    // getElementById ใช้บน document เท่านั้น (ไม่ใช่บน element อื่น)
    const content = document.getElementById('content');
    // content.getElementById('intro-text'); // Error! 
    // ต้องใช้ document.getElementById เสมอ
  </script>
</body>
</html>
```

---

## Step 194: getElementsByClassName และ getElementsByTagName

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div class="card featured">Card 1</div>
  <div class="card">Card 2</div>
  <div class="card featured">Card 3</div>
  <div class="other">Other</div>

  <h2>หัวข้อ 1</h2>
  <p>ย่อหน้า 1</p>
  <h2>หัวข้อ 2</h2>
  <p>ย่อหน้า 2</p>
  <h3>หัวข้อย่อย</h3>

  <script>
    // getElementsByClassName - คืน HTMLCollection (live)
    const cards = document.getElementsByClassName('card');
    console.log(cards);        // HTMLCollection(3) [div, div, div]
    console.log(cards.length); // 3
    console.log(cards[0]);     // <div class="card featured">

    // หลาย class (คั่นด้วย space)
    const featured = document.getElementsByClassName('card featured');
    console.log(featured.length); // 2

    // ต้อง convert เป็น Array เพื่อใช้ Array methods
    const cardsArray = Array.from(cards);
    cardsArray.forEach(card => {
      card.style.border = '1px solid #ccc';
    });

    // หรือใช้ spread operator
    [...cards].forEach(card => {
      card.style.padding = '10px';
    });

    // getElementsByTagName - เลือกด้วย tag name
    const headings = document.getElementsByTagName('h2');
    console.log(headings.length); // 2

    const paragraphs = document.getElementsByTagName('p');
    console.log(paragraphs.length); // 2

    // ทุก elements
    const allElements = document.getElementsByTagName('*');
    console.log(allElements.length); // จำนวนทั้งหมด

    // HTMLCollection เป็น Live Collection
    // ถ้าเพิ่ม element ใหม่ที่ match, collection จะ update อัตโนมัติ
    const divs = document.getElementsByTagName('div');
    console.log(divs.length); // 4

    const newDiv = document.createElement('div');
    document.body.appendChild(newDiv);
    console.log(divs.length); // 5 (เพิ่มขึ้นอัตโนมัติ!)

    // เปรียบเทียบกับ querySelectorAll ที่เป็น Static
    const staticDivs = document.querySelectorAll('div');
    const newDiv2 = document.createElement('div');
    document.body.appendChild(newDiv2);
    console.log(staticDivs.length); // ยังคงเดิม (ไม่ update)
  </script>
</body>
</html>
```

---

## Step 195: querySelector และ querySelectorAll

`querySelector` และ `querySelectorAll` ใช้ CSS Selector syntax ทำให้ยืดหยุ่นกว่ามาก

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <header id="main-header">
    <nav class="navbar">
      <ul>
        <li class="nav-item active"><a href="#">หน้าแรก</a></li>
        <li class="nav-item"><a href="#">สินค้า</a></li>
        <li class="nav-item"><a href="#">ติดต่อ</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <article class="post featured">
      <h2 class="post-title">บทความที่ 1</h2>
      <p class="post-body">เนื้อหา...</p>
      <span class="tag">JavaScript</span>
      <span class="tag">DOM</span>
    </article>
    <article class="post">
      <h2 class="post-title">บทความที่ 2</h2>
      <p class="post-body">เนื้อหา...</p>
      <span class="tag">CSS</span>
    </article>
  </main>

  <form id="search-form">
    <input type="text" name="query" placeholder="ค้นหา...">
    <button type="submit">ค้นหา</button>
  </form>

  <script>
    // querySelector - คืน Element แรกที่ตรงกัน หรือ null

    // เลือกด้วย ID
    const header = document.querySelector('#main-header');

    // เลือกด้วย class
    const navbar = document.querySelector('.navbar');

    // เลือกด้วย tag
    const firstH2 = document.querySelector('h2');

    // เลือกด้วย attribute
    const textInput = document.querySelector('input[type="text"]');

    // เลือกด้วย pseudo-class
    const firstLi = document.querySelector('li:first-child');
    const lastLi = document.querySelector('li:last-child');

    // Descendant selector
    const navLink = document.querySelector('nav a');

    // Child combinator
    const directChild = document.querySelector('#main-header > nav');

    // Complex selectors
    const activeItem = document.querySelector('.nav-item.active');
    const featured = document.querySelector('.post.featured');
    console.log(featured.querySelector('.post-title').textContent); // "บทความที่ 1"

    // querySelectorAll - คืน NodeList (static)
    const allPosts = document.querySelectorAll('.post');
    console.log(allPosts.length); // 2

    const allTags = document.querySelectorAll('.tag');
    console.log(allTags.length); // 3

    // NodeList มี forEach ในตัว
    allTags.forEach((tag, index) => {
      console.log(`Tag ${index}: ${tag.textContent}`);
    });

    // แต่ไม่มี map, filter ต้องแปลงก่อน
    const tagTexts = [...allTags].map(tag => tag.textContent);
    console.log(tagTexts); // ["JavaScript", "DOM", "CSS"]

    // :not() selector
    const nonFeatured = document.querySelectorAll('.post:not(.featured)');
    console.log(nonFeatured.length); // 1

    // nth-child
    const secondNavItem = document.querySelector('.nav-item:nth-child(2)');
    console.log(secondNavItem.textContent.trim()); // "สินค้า"

    // เรียกบน element อื่น (scoped)
    const firstPost = document.querySelector('.post');
    const tagsInFirstPost = firstPost.querySelectorAll('.tag');
    console.log(tagsInFirstPost.length); // 2 (เฉพาะใน post แรก)
  </script>
</body>
</html>
```

---

## Step 196: การ Navigate DOM - parentNode, childNodes, firstChild, lastChild

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="family">
    <p id="child1">ลูกคนที่ 1</p>
    <p id="child2">ลูกคนที่ 2</p>
    <p id="child3">ลูกคนที่ 3</p>
  </div>

  <script>
    const family = document.getElementById('family');
    const child2 = document.getElementById('child2');

    // parentNode - คืน parent node (ได้ทุก node type)
    console.log(child2.parentNode);    // <div id="family">
    console.log(child2.parentNode.id); // "family"

    // parent ของ <body> คือ <html>
    console.log(document.body.parentNode); // <html>
    // parent ของ <html> คือ document
    console.log(document.documentElement.parentNode); // document
    // parent ของ document คือ null
    console.log(document.parentNode); // null

    // childNodes - คืน NodeList ของ child nodes ทั้งหมด (รวม text nodes)
    console.log(family.childNodes);
    // NodeList(7) [text, p#child1, text, p#child2, text, p#child3, text]
    // text nodes คือ whitespace (\n และ spaces) ระหว่าง elements

    console.log(family.childNodes.length); // 7

    // ตรวจสอบประเภท node
    family.childNodes.forEach(node => {
      if (node.nodeType === Node.TEXT_NODE) {
        console.log('Text:', JSON.stringify(node.nodeValue));
      } else {
        console.log('Element:', node.tagName, node.id);
      }
    });

    // firstChild - child node แรก (อาจเป็น text node)
    console.log(family.firstChild); // text node (whitespace)
    console.log(family.firstChild.nodeType); // 3 (TEXT_NODE)

    // lastChild - child node สุดท้าย
    console.log(family.lastChild); // text node (whitespace)

    // nextSibling - sibling node ถัดไป
    console.log(child2.nextSibling); // text node
    console.log(child2.nextSibling.nextSibling); // <p id="child3">

    // previousSibling - sibling node ก่อนหน้า
    console.log(child2.previousSibling); // text node
    console.log(child2.previousSibling.previousSibling); // <p id="child1">

    // ตัวอย่างการหา parent สูงสุดที่มี class
    function findAncestorWithClass(element, className) {
      let current = element.parentNode;
      while (current && current !== document) {
        if (current.classList && current.classList.contains(className)) {
          return current;
        }
        current = current.parentNode;
      }
      return null;
    }
  </script>
</body>
</html>
```

---

## Step 197: parentElement, children, firstElementChild, lastElementChild

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="parent">
    <!-- comment node -->
    <div class="child" id="c1">Child 1</div>
    <div class="child" id="c2">Child 2</div>
    <div class="child" id="c3">Child 3</div>
  </div>

  <script>
    const parent = document.getElementById('parent');
    const c2 = document.getElementById('c2');

    // parentElement - คืน parent Element (null ถ้า parent ไม่ใช่ element)
    console.log(c2.parentElement); // <div id="parent">
    // parentNode vs parentElement
    // document.documentElement.parentNode = document (Document node)
    // document.documentElement.parentElement = null (เพราะ parent ไม่ใช่ Element)
    console.log(document.documentElement.parentNode);    // document
    console.log(document.documentElement.parentElement); // null

    // children - คืน HTMLCollection ของ child elements (ไม่รวม text/comment)
    console.log(parent.children);
    // HTMLCollection(3) [div#c1, div#c2, div#c3]
    console.log(parent.children.length); // 3 (ไม่นับ comment และ text)

    // children เป็น live collection
    const newChild = document.createElement('div');
    parent.appendChild(newChild);
    console.log(parent.children.length); // 4

    // firstElementChild - child element แรก (ข้าม text/comment)
    console.log(parent.firstElementChild); // <div id="c1">
    console.log(parent.firstElementChild.id); // "c1"

    // lastElementChild - child element สุดท้าย
    console.log(parent.lastElementChild); // div ที่เพิ่งเพิ่ม
    parent.removeChild(newChild); // เอาออก

    // nextElementSibling - sibling element ถัดไป (ข้าม text/comment)
    console.log(c2.nextElementSibling); // <div id="c3">
    console.log(c2.nextElementSibling.id); // "c3"

    // previousElementSibling
    console.log(c2.previousElementSibling); // <div id="c1">

    // ตัวอย่างการ iterate children อย่างมีประสิทธิภาพ
    // วิธีที่ 1: for loop กับ children
    for (let i = 0; i < parent.children.length; i++) {
      console.log(parent.children[i].id);
    }

    // วิธีที่ 2: for...of
    for (const child of parent.children) {
      console.log(child.textContent);
    }

    // วิธีที่ 3: spread + forEach
    [...parent.children].forEach(child => {
      child.style.color = 'blue';
    });

    // นับจำนวน levels ถึง root
    function getDepth(element) {
      let depth = 0;
      let current = element;
      while (current.parentElement) {
        depth++;
        current = current.parentElement;
      }
      return depth;
    }
    console.log(getDepth(c2)); // 3 (body > div#parent > div#c2)
  </script>
</body>
</html>
```

---

## Step 198: createElement และ createTextNode

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="container"></div>

  <script>
    const container = document.getElementById('container');

    // createElement - สร้าง element ใหม่
    const div = document.createElement('div');
    console.log(div); // <div></div>
    console.log(div.tagName); // "DIV"
    console.log(div.parentNode); // null (ยังไม่ได้เพิ่มใน DOM)

    // ตั้งค่า attributes และ properties
    div.id = 'new-div';
    div.className = 'card highlight';
    div.textContent = 'Card ใหม่';
    div.style.backgroundColor = '#f0f0f0';
    div.style.padding = '10px';

    // createTextNode - สร้าง text node
    const textNode = document.createTextNode('Hello World สวัสดีโลก');
    console.log(textNode.nodeType); // 3
    console.log(textNode.nodeValue); // "Hello World สวัสดีโลก"

    // ตัวอย่างสร้าง element หลายระดับ
    const article = document.createElement('article');
    article.className = 'post';

    const h2 = document.createElement('h2');
    h2.textContent = 'หัวข้อบทความ';
    h2.style.color = '#333';

    const p = document.createElement('p');
    const text = document.createTextNode('นี่คือเนื้อหาของบทความ...');
    p.appendChild(text);

    const btn = document.createElement('button');
    btn.textContent = 'อ่านเพิ่มเติม';
    btn.className = 'btn btn-primary';
    btn.addEventListener('click', () => alert('คลิกแล้ว!'));

    // สร้าง list
    const ul = document.createElement('ul');
    const items = ['JavaScript', 'Python', 'Go', 'Rust'];
    items.forEach(item => {
      const li = document.createElement('li');
      li.textContent = item;
      ul.appendChild(li);
    });

    // Function สร้าง card
    function createCard(title, body, tags = []) {
      const card = document.createElement('div');
      card.className = 'card';

      const cardTitle = document.createElement('h3');
      cardTitle.textContent = title;

      const cardBody = document.createElement('p');
      cardBody.textContent = body;

      const tagContainer = document.createElement('div');
      tagContainer.className = 'tags';
      tags.forEach(tag => {
        const tagEl = document.createElement('span');
        tagEl.className = 'tag';
        tagEl.textContent = tag;
        tagContainer.appendChild(tagEl);
      });

      card.appendChild(cardTitle);
      card.appendChild(cardBody);
      card.appendChild(tagContainer);

      return card;
    }

    const myCard = createCard(
      'DOM Manipulation',
      'การจัดการ DOM ด้วย JavaScript',
      ['JavaScript', 'DOM', 'Frontend']
    );

    container.appendChild(myCard);
  </script>
</body>
</html>
```

---

## Step 199: appendChild, insertBefore, insertAdjacentElement

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <ul id="list">
    <li id="item1">รายการที่ 1</li>
    <li id="item2">รายการที่ 2</li>
    <li id="item3">รายการที่ 3</li>
  </ul>

  <div id="target">
    <p id="existing">Existing content</p>
  </div>

  <script>
    const list = document.getElementById('list');
    const item2 = document.getElementById('item2');
    const target = document.getElementById('target');
    const existing = document.getElementById('existing');

    // appendChild - เพิ่ม element เป็น last child
    const newItem = document.createElement('li');
    newItem.textContent = 'รายการใหม่ (appendChild)';
    list.appendChild(newItem);
    // list: item1, item2, item3, newItem

    // appendChild คืน element ที่เพิ่ม
    const returned = list.appendChild(document.createElement('li'));
    returned.textContent = 'อีกรายการ';

    // insertBefore(newNode, referenceNode)
    // เพิ่ม newNode ก่อน referenceNode
    const insertedItem = document.createElement('li');
    insertedItem.textContent = 'รายการแทรก (insertBefore item2)';
    list.insertBefore(insertedItem, item2);
    // list: item1, insertedItem, item2, item3, newItem, returned

    // insertBefore กับ null = appendChild
    const lastItem = document.createElement('li');
    lastItem.textContent = 'สุดท้าย (insertBefore null)';
    list.insertBefore(lastItem, null);

    // insertAdjacentElement(position, element)
    // position: 'beforebegin', 'afterbegin', 'beforeend', 'afterend'
    const before = document.createElement('p');
    before.textContent = 'beforebegin - ก่อน existing (sibling)';
    existing.insertAdjacentElement('beforebegin', before);

    const afterBegin = document.createElement('span');
    afterBegin.textContent = 'afterbegin - first child ของ existing';
    existing.insertAdjacentElement('afterbegin', afterBegin);

    const beforeEnd = document.createElement('span');
    beforeEnd.textContent = 'beforeend - last child ของ existing';
    existing.insertAdjacentElement('beforeend', beforeEnd);

    const after = document.createElement('p');
    after.textContent = 'afterend - หลัง existing (sibling)';
    existing.insertAdjacentElement('afterend', after);

    // insertAdjacentHTML - เพิ่ม HTML string
    existing.insertAdjacentHTML('beforeend', '<strong> [bold text] </strong>');
    existing.insertAdjacentHTML('afterend', '<p class="note">Note paragraph</p>');

    // insertAdjacentText - เพิ่ม text (safe จาก XSS)
    existing.insertAdjacentText('beforeend', ' <script>alert("xss")</script> ');
    // text จะถูก escape อัตโนมัติ ไม่ถูก execute

    // prepend และ append (modern methods)
    const newDiv = document.createElement('div');
    newDiv.id = 'modern-div';

    // append - เพิ่มหลาย nodes/strings พร้อมกัน
    newDiv.append('Text first, ', document.createElement('span'), ' text last');
    document.body.append(newDiv);

    // prepend - เพิ่มเป็น first child
    const header = document.createElement('h1');
    header.textContent = 'Header เพิ่มด้วย prepend';
    newDiv.prepend(header);
  </script>
</body>
</html>
```

---

## Step 200: insertAdjacentHTML และ append/prepend

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="demo">
    <p id="center">ตรงกลาง</p>
  </div>

  <script>
    const demo = document.getElementById('demo');
    const center = document.getElementById('center');

    // insertAdjacentHTML positions:
    // 'beforebegin': ก่อน element (outside, before)
    // 'afterbegin':  ก่อน first child (inside, beginning)
    // 'beforeend':   หลัง last child (inside, end)
    // 'afterend':    หลัง element (outside, after)

    center.insertAdjacentHTML('beforebegin', '<p style="color:red">beforebegin</p>');
    center.insertAdjacentHTML('afterbegin', '<span style="color:green">afterbegin</span> ');
    center.insertAdjacentHTML('beforeend', ' <span style="color:blue">beforeend</span>');
    center.insertAdjacentHTML('afterend', '<p style="color:purple">afterend</p>');

    // ผลลัพธ์ใน demo:
    // <p style="color:red">beforebegin</p>
    // <p id="center">
    //   <span style="color:green">afterbegin</span>
    //   ตรงกลาง
    //   <span style="color:blue">beforeend</span>
    // </p>
    // <p style="color:purple">afterend</p>

    // สร้าง HTML template ด้วย insertAdjacentHTML
    function addCard(container, data) {
      const html = `
        <div class="card" data-id="${data.id}">
          <img src="${data.image}" alt="${data.title}">
          <div class="card-body">
            <h3 class="card-title">${data.title}</h3>
            <p class="card-text">${data.description}</p>
            <button class="btn" onclick="viewCard(${data.id})">ดูรายละเอียด</button>
          </div>
        </div>
      `;
      container.insertAdjacentHTML('beforeend', html);
    }

    // before(), after() - modern methods
    const refEl = document.createElement('div');
    refEl.textContent = 'Reference';
    demo.appendChild(refEl);

    const newBefore = document.createElement('p');
    newBefore.textContent = 'inserted before refEl';
    refEl.before(newBefore);

    const newAfter = document.createElement('p');
    newAfter.textContent = 'inserted after refEl';
    refEl.after(newAfter);

    // before/after ยอมรับหลาย arguments
    refEl.before('text1', document.createElement('span'), 'text2');
  </script>
</body>
</html>
```

---

## Step 201: removeChild และ remove

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <ul id="task-list">
    <li id="task1">งานที่ 1</li>
    <li id="task2">งานที่ 2</li>
    <li id="task3">งานที่ 3</li>
    <li id="task4">งานที่ 4</li>
  </ul>

  <div id="modal" class="modal">
    <div class="modal-content">
      <h2>Modal Title</h2>
      <p>Modal content here</p>
    </div>
  </div>

  <script>
    const taskList = document.getElementById('task-list');
    const task2 = document.getElementById('task2');
    const modal = document.getElementById('modal');

    // removeChild(child) - ลบ child node จาก parent
    // ต้องเรียกจาก parent และส่ง child เป็น argument
    const removedTask2 = taskList.removeChild(task2);
    console.log(removedTask2); // element ที่ถูกลบยังอยู่ใน memory
    console.log(removedTask2.textContent); // "งานที่ 2"
    // สามารถนำกลับมาใส่ใหม่ได้
    taskList.appendChild(removedTask2); // ใส่กลับเป็น last child

    // remove() - ลบ element ตัวเอง (modern)
    const task3 = document.getElementById('task3');
    task3.remove(); // ลบ task3 ออกจาก DOM

    // ลบ modal
    modal.remove();

    // ลบ element ด้วย parent.removeChild แบบ safe
    function safeRemove(element) {
      if (element && element.parentNode) {
        element.parentNode.removeChild(element);
        return true;
      }
      return false;
    }

    // ลบ elements ที่ match selector
    function removeAll(selector, context = document) {
      const elements = context.querySelectorAll(selector);
      elements.forEach(el => el.remove());
      return elements.length;
    }

    // ลบ list items ทั้งหมด
    removeAll('#task-list li');

    // ล้าง children ทั้งหมด (เร็วกว่า remove all children)
    function clearChildren(element) {
      // วิธีที่ 1: innerHTML = ''
      // element.innerHTML = '';

      // วิธีที่ 2: ลบทีละอัน
      // while (element.firstChild) {
      //   element.removeChild(element.firstChild);
      // }

      // วิธีที่ 3: replaceChildren() (modern)
      element.replaceChildren();
    }

    clearChildren(taskList);
    console.log(taskList.children.length); // 0

    // Detach and reattach pattern (เพิ่ม performance)
    // ลบออกจาก DOM ก่อน แก้ไข แล้วค่อยใส่คืน
    function updateList(listEl, items) {
      const parent = listEl.parentNode;
      const nextSibling = listEl.nextSibling;

      // Detach
      parent.removeChild(listEl);

      // แก้ไข (ไม่ trigger reflow)
      listEl.innerHTML = '';
      items.forEach(item => {
        const li = document.createElement('li');
        li.textContent = item;
        listEl.appendChild(li);
      });

      // Reattach
      parent.insertBefore(listEl, nextSibling);
    }
  </script>
</body>
</html>
```

---

## Step 202: replaceChild และ replaceWith

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="content">
    <h1 id="old-title">หัวข้อเก่า</h1>
    <p>เนื้อหา</p>
  </div>

  <ul id="menu">
    <li id="old-item">รายการเก่า</li>
    <li>รายการอื่น</li>
  </ul>

  <script>
    const content = document.getElementById('content');
    const oldTitle = document.getElementById('old-title');
    const oldItem = document.getElementById('old-item');

    // replaceChild(newChild, oldChild) - แทน oldChild ด้วย newChild
    const newTitle = document.createElement('h2');
    newTitle.textContent = 'หัวข้อใหม่ (h2)';
    newTitle.style.color = 'green';

    const replaced = content.replaceChild(newTitle, oldTitle);
    console.log(replaced); // oldTitle element ที่ถูกแทน
    console.log(replaced.textContent); // "หัวข้อเก่า"

    // replaceWith() - modern method, แทน element ตัวเอง
    const newItem = document.createElement('li');
    newItem.textContent = 'รายการใหม่ (replaceWith)';
    newItem.style.fontWeight = 'bold';

    oldItem.replaceWith(newItem);

    // replaceWith ยอมรับหลาย arguments
    const extra1 = document.createElement('li');
    extra1.textContent = 'Extra 1';
    const extra2 = document.createElement('li');
    extra2.textContent = 'Extra 2';
    newItem.replaceWith(extra1, 'plain text', extra2);

    // replaceChildren() - แทน children ทั้งหมด
    const ul = document.getElementById('menu');
    const items = ['ข้อมูล', 'รูปภาพ', 'วิดีโอ'];
    const newItems = items.map(text => {
      const li = document.createElement('li');
      li.textContent = text;
      return li;
    });
    ul.replaceChildren(...newItems);
    // ul ตอนนี้มีแค่ items ใหม่ (ไม่มี children เก่า)

    // ตัวอย่าง dynamic content replacement
    function replaceContent(selector, newHTML) {
      const element = document.querySelector(selector);
      if (!element) return;

      const temp = document.createElement('div');
      temp.innerHTML = newHTML;

      element.replaceWith(...temp.children);
    }
  </script>
</body>
</html>
```

---

## Step 203: cloneNode

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="template-card" class="card">
    <h3>Card Title</h3>
    <p>Card description</p>
    <ul>
      <li>Item 1</li>
      <li>Item 2</li>
    </ul>
    <button onclick="alert('Original!')">Click me</button>
  </div>

  <div id="cards-container"></div>

  <script>
    const template = document.getElementById('template-card');
    const container = document.getElementById('cards-container');

    // cloneNode(deep)
    // deep = false: copy เฉพาะ element ไม่รวม children
    // deep = true: copy ทั้ง element และ children ทั้งหมด

    // Shallow clone
    const shallowClone = template.cloneNode(false);
    console.log(shallowClone.children.length); // 0 (ไม่มี children)
    console.log(shallowClone.className); // "card" (attributes copy มาด้วย)

    // Deep clone
    const deepClone = template.cloneNode(true);
    console.log(deepClone.children.length); // 4 (h3, p, ul, button)
    console.log(deepClone.innerHTML); // copy ทั้งหมด

    // แก้ไข clone (ไม่กระทบ original)
    deepClone.id = 'clone-1'; // เปลี่ยน id (unique)
    deepClone.querySelector('h3').textContent = 'Cloned Card';
    deepClone.querySelector('p').textContent = 'This is a clone';
    container.appendChild(deepClone);

    // สร้าง cards หลายอัน
    const cardsData = [
      { id: 1, title: 'JavaScript', desc: 'ภาษา Programming', items: ['ES6+', 'Async', 'DOM'] },
      { id: 2, title: 'CSS', desc: 'Styling language', items: ['Flexbox', 'Grid', 'Animation'] },
      { id: 3, title: 'HTML', desc: 'Markup language', items: ['Semantic', 'Forms', 'Media'] },
    ];

    cardsData.forEach(data => {
      const card = template.cloneNode(true);
      card.id = `card-${data.id}`;
      card.querySelector('h3').textContent = data.title;
      card.querySelector('p').textContent = data.desc;

      const ul = card.querySelector('ul');
      ul.innerHTML = '';
      data.items.forEach(item => {
        const li = document.createElement('li');
        li.textContent = item;
        ul.appendChild(li);
      });

      const btn = card.querySelector('button');
      btn.onclick = () => alert(`Card: ${data.title}`);

      container.appendChild(card);
    });

    // สิ่งที่ cloneNode COPY:
    // - Tag name
    // - Attributes (id, class, style, data-*, etc.)
    // - innerHTML (ถ้า deep = true)
    // - Event handlers ที่ set ผ่าน attribute (onclick="...")

    // สิ่งที่ cloneNode ไม่ COPY:
    // - Event listeners ที่ add ด้วย addEventListener
    // - ค่า .value ของ input elements
    const input = document.createElement('input');
    input.value = 'test value';
    input.addEventListener('click', () => console.log('clicked'));

    const inputClone = input.cloneNode(true);
    console.log(inputClone.value); // '' (ไม่ copy value)
    // click event listener ก็ไม่ถูก copy
  </script>
</body>
</html>
```

---

## Step 204: innerHTML, innerText, textContent, outerHTML

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="demo">
    <p>ย่อหน้าแรก</p>
    <p style="display:none">ย่อหน้าที่ซ่อน</p>
    <script>var x = 1;</script>
    ข้อความโดยตรง
    <span>span text</span>
  </div>

  <script>
    const demo = document.getElementById('demo');

    // innerHTML - อ่าน/เขียน HTML content ภายใน
    console.log(demo.innerHTML);
    // จะแสดง HTML ทั้งหมดภายใน <div>

    // เขียน innerHTML
    demo.innerHTML = '<h2>New Content</h2><p>New paragraph</p>';
    // ลบ content เก่าทั้งหมดและแทนด้วยใหม่

    // innerHTML เสี่ยง XSS ถ้ารับ input จาก user
    const userInput = '<img src=x onerror="alert(\'XSS\')">';
    // อย่าทำแบบนี้:
    // demo.innerHTML = userInput; // DANGEROUS!

    // ใช้ textContent หรือ createTextNode แทน
    demo.textContent = userInput; // จะ escape HTML entities
    // แสดง: <img src=x onerror="alert('XSS')"> (as text)

    // Reset demo
    demo.innerHTML = `
      <p>ย่อหน้าแรก</p>
      <p style="display:none">ย่อหน้าที่ซ่อน</p>
      ข้อความโดยตรง
      <span>span text</span>
    `;

    // textContent - อ่าน/เขียน text content ทั้งหมด (รวมถึงที่ซ่อน)
    console.log(demo.textContent);
    // "ย่อหน้าแรกย่อหน้าที่ซ่อนข้อความโดยตรงspan text"
    // รวมทั้ง hidden elements และ whitespace

    // innerText - อ่าน/เขียน text ที่แสดงจริง (คล้าย copy-paste)
    console.log(demo.innerText);
    // "ย่อหน้าแรก\nข้อความโดยตรงspan text"
    // ไม่รวม hidden elements (display:none)
    // จัดการ whitespace เหมือน CSS

    // ความแตกต่าง textContent vs innerText
    const p = document.createElement('p');
    p.innerHTML = 'Hello   <span>World</span>   !';
    document.body.appendChild(p);

    console.log(p.textContent); // "Hello   World   !" (ได้ raw text)
    console.log(p.innerText);   // "Hello World !" (collapsed whitespace)

    // textContent เร็วกว่า innerText เพราะไม่ต้อง calculate layout

    // outerHTML - รวม element ตัวเองด้วย
    const span = document.querySelector('span');
    console.log(span.innerHTML);  // "span text"
    console.log(span.outerHTML);  // "<span>span text</span>"

    // แทน element ด้วย outerHTML
    span.outerHTML = '<strong>แทนด้วย strong</strong>';
    // span ถูกแทนด้วย strong (span reference เก่าชี้ไปที่ detached node)

    // outerHTML ของ document.documentElement
    // console.log(document.documentElement.outerHTML);
    // จะแสดง HTML ทั้งหมดของ page

    // ใช้ innerHTML อย่างปลอดภัย ด้วย sanitization
    function safeHTML(html) {
      const div = document.createElement('div');
      div.textContent = html;
      return div.innerHTML;
    }

    const cleanHTML = safeHTML('<script>alert("xss")<\/script>');
    console.log(cleanHTML); // "&lt;script&gt;alert(&quot;xss&quot;)&lt;/script&gt;"
  </script>
</body>
</html>
```

---

## Step 205: getAttribute, setAttribute, removeAttribute, hasAttribute

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <img id="hero-img" 
       src="hero.jpg" 
       alt="Hero Image" 
       width="800" 
       height="400"
       data-category="banner"
       class="hero featured">
  
  <a id="link" href="https://example.com" target="_blank" rel="noopener">Link</a>
  
  <input id="email" type="email" required disabled value="test@example.com">

  <script>
    const img = document.getElementById('hero-img');
    const link = document.getElementById('link');
    const email = document.getElementById('email');

    // getAttribute(name) - อ่าน attribute value
    console.log(img.getAttribute('src'));      // "hero.jpg"
    console.log(img.getAttribute('alt'));      // "Hero Image"
    console.log(img.getAttribute('width'));    // "800" (string เสมอ)
    console.log(img.getAttribute('class'));    // "hero featured"
    console.log(img.getAttribute('data-category')); // "banner"
    console.log(img.getAttribute('nonexistent')); // null

    // getAttribute vs property
    // getAttribute ได้ original HTML attribute value
    // property ได้ processed/current value
    console.log(link.getAttribute('href')); // "https://example.com" (as written)
    console.log(link.href);                 // "https://example.com" (full URL)

    const relativeLink = document.createElement('a');
    relativeLink.setAttribute('href', '/page');
    console.log(relativeLink.getAttribute('href')); // "/page"
    console.log(relativeLink.href);                 // "https://current-domain.com/page"

    // setAttribute(name, value) - ตั้งค่า attribute
    img.setAttribute('src', 'new-hero.jpg');
    img.setAttribute('alt', 'New Hero Image');
    img.setAttribute('title', 'Hero Image Title'); // เพิ่ม attribute ใหม่
    img.setAttribute('width', 600); // number จะถูก convert เป็น string

    // Boolean attributes
    // ถ้ามี attribute = true, ไม่มี = false
    console.log(email.getAttribute('required')); // "" (empty string, not null)
    console.log(email.getAttribute('disabled')); // ""
    email.setAttribute('disabled', ''); // ตั้งค่า disabled
    email.removeAttribute('disabled');  // เอา disabled ออก

    // removeAttribute(name) - ลบ attribute
    img.removeAttribute('title');
    console.log(img.getAttribute('title')); // null

    // hasAttribute(name) - ตรวจสอบว่ามี attribute ไหม
    console.log(img.hasAttribute('src'));        // true
    console.log(img.hasAttribute('title'));      // false
    console.log(email.hasAttribute('required')); // true
    console.log(email.hasAttribute('disabled')); // false (เพิ่งลบไป)

    // attributes property - ดู attributes ทั้งหมด
    console.log(img.attributes); // NamedNodeMap
    for (const attr of img.attributes) {
      console.log(`${attr.name}: ${attr.value}`);
    }

    // hasAttributes() - ตรวจว่ามี attributes ใดๆ ไหม
    console.log(img.hasAttributes()); // true

    // ตัวอย่างการใช้งาน
    function toggleAttribute(element, attrName) {
      if (element.hasAttribute(attrName)) {
        element.removeAttribute(attrName);
      } else {
        element.setAttribute(attrName, '');
      }
    }

    toggleAttribute(email, 'disabled'); // enable
    toggleAttribute(email, 'disabled'); // disable again
  </script>
</body>
</html>
```

---

## Step 206: classList: add, remove, toggle, contains, replace

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .box { width: 100px; height: 100px; }
    .red { background-color: red; }
    .blue { background-color: blue; }
    .green { background-color: green; }
    .active { border: 3px solid black; }
    .hidden { display: none; }
    .large { width: 200px; height: 200px; }
    .rounded { border-radius: 50%; }
  </style>
</head>
<body>
  <div id="box" class="box red">กล่อง</div>
  <button id="toggle-btn">Toggle Active</button>
  <button id="change-color">เปลี่ยนสี</button>

  <script>
    const box = document.getElementById('box');
    const toggleBtn = document.getElementById('toggle-btn');
    const changeBtn = document.getElementById('change-color');

    // classList เป็น DOMTokenList
    console.log(box.classList);           // DOMTokenList ["box", "red"]
    console.log(box.classList.length);    // 2
    console.log(box.classList[0]);        // "box"

    // add() - เพิ่ม class (หลาย class ได้)
    box.classList.add('active');
    box.classList.add('large', 'rounded'); // เพิ่มหลายอัน
    console.log(box.className); // "box red active large rounded"

    // add ซ้ำไม่เพิ่มซ้ำ
    box.classList.add('active'); // ไม่มีผล
    console.log(box.classList.length); // 5 (ไม่เพิ่มขึ้น)

    // remove() - ลบ class (หลาย class ได้)
    box.classList.remove('large');
    box.classList.remove('rounded', 'active'); // ลบหลายอัน
    console.log(box.className); // "box red"

    // remove class ที่ไม่มีไม่ error
    box.classList.remove('nonexistent'); // ไม่ error

    // toggle() - ถ้ามีให้ลบ, ถ้าไม่มีให้เพิ่ม
    box.classList.toggle('active'); // เพิ่ม (ไม่มีอยู่)
    console.log(box.classList.contains('active')); // true
    box.classList.toggle('active'); // ลบ (มีอยู่แล้ว)
    console.log(box.classList.contains('active')); // false

    // toggle กับ force argument
    box.classList.toggle('active', true);  // force add (ไม่ toggle)
    box.classList.toggle('active', true);  // ยังคง add (ไม่ลบ)
    box.classList.toggle('active', false); // force remove

    // contains() - ตรวจสอบว่ามี class ไหม
    console.log(box.classList.contains('red'));   // true
    console.log(box.classList.contains('blue'));  // false

    // replace(oldClass, newClass) - แทน class
    box.classList.replace('red', 'blue');  // แทน red ด้วย blue
    console.log(box.classList.contains('red'));  // false
    console.log(box.classList.contains('blue')); // true

    // replace คืน true ถ้าสำเร็จ, false ถ้าไม่พบ oldClass
    const result = box.classList.replace('nonexistent', 'green');
    console.log(result); // false

    // item(index) - อ่าน class ที่ index
    console.log(box.classList.item(0)); // "box"
    console.log(box.classList.item(1)); // "blue"

    // forEach, values, entries
    box.classList.forEach(cls => console.log(cls));

    for (const [index, cls] of box.classList.entries()) {
      console.log(index, cls);
    }

    // ตัวอย่างปฏิบัติจริง
    toggleBtn.addEventListener('click', () => {
      box.classList.toggle('active');
      toggleBtn.textContent = box.classList.contains('active')
        ? 'Deactivate'
        : 'Activate';
    });

    let colorIndex = 0;
    const colors = ['red', 'blue', 'green'];
    changeBtn.addEventListener('click', () => {
      const currentColor = colors[colorIndex];
      colorIndex = (colorIndex + 1) % colors.length;
      const nextColor = colors[colorIndex];
      box.classList.replace(currentColor, nextColor);
    });
  </script>
</body>
</html>
```

---

## Step 207: style property

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="box" style="width:100px; height:100px; background:red;">Box</div>

  <script>
    const box = document.getElementById('box');

    // อ่าน inline style
    console.log(box.style.width);      // "100px"
    console.log(box.style.height);     // "100px"
    console.log(box.style.background); // "red"
    console.log(box.style.color);      // "" (ไม่ได้ set inline)

    // ตั้งค่า style
    box.style.color = 'white';
    box.style.fontSize = '16px';        // camelCase สำหรับ property ที่มี -
    box.style.backgroundColor = 'blue'; // background-color -> backgroundColor
    box.style.borderRadius = '8px';     // border-radius -> borderRadius
    box.style.marginTop = '20px';       // margin-top -> marginTop
    box.style.zIndex = '100';           // z-index -> zIndex

    // property names กับ hyphenated
    box.style['font-size'] = '18px';    // ใช้ bracket notation ก็ได้
    box.style.setProperty('font-size', '20px'); // หรือ setProperty
    box.style.setProperty('--custom-var', 'red'); // CSS variables

    // ลบ inline style
    box.style.color = ''; // ลบ (กลับไปใช้ stylesheet)
    box.style.removeProperty('font-size'); // ลบด้วย removeProperty

    // cssText - อ่าน/เขียน style ทั้งหมดเป็น string
    console.log(box.style.cssText);
    // "width: 100px; height: 100px; background: blue; ..."

    // เขียน cssText (แทน style ทั้งหมด)
    box.style.cssText = 'width: 200px; height: 200px; background: green; color: white;';
    // style เก่าทั้งหมดถูกแทน

    // เพิ่ม style โดยไม่ลบของเก่า
    box.style.cssText += '; border: 2px solid black;';

    // getComputedStyle - อ่าน computed style (ทั้ง inline + stylesheet)
    const computed = window.getComputedStyle(box);
    console.log(computed.width);           // computed value เช่น "200px"
    console.log(computed.fontSize);        // computed font size
    console.log(computed.backgroundColor); // computed background color

    // getComputedStyle เป็น read-only
    // computed.width = '300px'; // Error!

    // getPropertyValue
    console.log(computed.getPropertyValue('background-color'));
    // "rgb(0, 128, 0)"

    // Animation ง่ายๆ ด้วย style
    let pos = 0;
    let direction = 1;
    function animate() {
      pos += direction * 2;
      if (pos >= 300 || pos <= 0) direction *= -1;
      box.style.transform = `translateX(${pos}px)`;
      requestAnimationFrame(animate);
    }
    // animate(); // เริ่ม animation
  </script>
</body>
</html>
```

---

## Step 208: data-* attributes และ dataset

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="user-card"
       data-user-id="123"
       data-user-name="สมชาย"
       data-user-role="admin"
       data-is-active="true"
       data-score="95.5">
    สมชาย (admin)
  </div>

  <ul id="product-list">
    <li data-id="1" data-price="150" data-category="food">ข้าวผัด - 150 บาท</li>
    <li data-id="2" data-price="200" data-category="drink">กาแฟ - 200 บาท</li>
    <li data-id="3" data-price="120" data-category="food">ต้มยำ - 120 บาท</li>
  </ul>

  <button data-action="save" data-target="form1">บันทึก</button>
  <button data-action="delete" data-target="item5">ลบ</button>

  <script>
    const card = document.getElementById('user-card');

    // dataset - ใช้อ่าน/เขียน data-* attributes
    console.log(card.dataset);
    // DOMStringMap {userId: "123", userName: "สมชาย", userRole: "admin", ...}

    // อ่าน data attributes
    console.log(card.dataset.userId);    // "123" (data-user-id)
    console.log(card.dataset.userName);  // "สมชาย" (data-user-name)
    console.log(card.dataset.userRole);  // "admin"
    console.log(card.dataset.isActive);  // "true" (ยังเป็น string)
    console.log(card.dataset.score);     // "95.5" (ยังเป็น string)

    // ต้อง parse ถ้าต้องการ type ที่ถูกต้อง
    const userId = Number(card.dataset.userId);      // 123
    const isActive = card.dataset.isActive === 'true'; // true (boolean)
    const score = parseFloat(card.dataset.score);     // 95.5

    // kebab-case -> camelCase
    // data-user-id -> dataset.userId
    // data-is-active -> dataset.isActive
    // data-my-long-attr -> dataset.myLongAttr

    // เขียน data attributes
    card.dataset.userId = '456'; // เปลี่ยน data-user-id
    card.dataset.lastLogin = '2024-01-15'; // เพิ่ม data-last-login
    console.log(card.getAttribute('data-last-login')); // "2024-01-15"

    // ลบ data attribute
    delete card.dataset.userRole;
    console.log(card.hasAttribute('data-user-role')); // false

    // ใช้ dataset กับ product list
    const products = document.querySelectorAll('#product-list li');
    let total = 0;
    products.forEach(product => {
      const price = Number(product.dataset.price);
      const category = product.dataset.category;
      total += price;
      console.log(`ID: ${product.dataset.id}, Category: ${category}, Price: ${price}`);
    });
    console.log('Total:', total); // 470

    // Filter ด้วย data attributes
    function filterByCategory(category) {
      products.forEach(product => {
        if (category === 'all' || product.dataset.category === category) {
          product.style.display = '';
        } else {
          product.style.display = 'none';
        }
      });
    }

    filterByCategory('food'); // แสดงเฉพาะ food

    // Event delegation ด้วย data attributes
    document.addEventListener('click', (e) => {
      const { action, target } = e.target.dataset;
      if (action === 'save') {
        console.log(`Saving ${target}`);
      } else if (action === 'delete') {
        console.log(`Deleting ${target}`);
      }
    });

    // ใช้ CSS selector กับ data attributes
    const adminCards = document.querySelectorAll('[data-user-role="admin"]');
    const activeUsers = document.querySelectorAll('[data-is-active="true"]');
    console.log(adminCards.length, activeUsers.length);
  </script>
</body>
</html>
```

---

## Step 209: ตัวอย่างการใช้งาน DOM จริง - Dynamic List

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <title>Dynamic Todo List</title>
  <style>
    body { font-family: sans-serif; max-width: 500px; margin: 20px auto; }
    .todo-item { display: flex; align-items: center; padding: 8px; border-bottom: 1px solid #eee; }
    .todo-item.completed span { text-decoration: line-through; color: #999; }
    .todo-item span { flex: 1; }
    button { margin-left: 5px; cursor: pointer; }
    .add-form { display: flex; margin-bottom: 10px; }
    .add-form input { flex: 1; padding: 8px; }
    .stats { color: #666; font-size: 0.9em; margin-top: 10px; }
  </style>
</head>
<body>
  <h1>Todo List</h1>
  <div class="add-form">
    <input type="text" id="new-todo" placeholder="เพิ่มรายการใหม่..." />
    <button id="add-btn">เพิ่ม</button>
  </div>
  <ul id="todo-list"></ul>
  <p class="stats" id="stats"></p>

  <script>
    const todoList = document.getElementById('todo-list');
    const newTodoInput = document.getElementById('new-todo');
    const addBtn = document.getElementById('add-btn');
    const stats = document.getElementById('stats');

    let todos = [
      { id: 1, text: 'เรียน JavaScript', completed: false },
      { id: 2, text: 'ทำ Project', completed: true },
    ];
    let nextId = 3;

    function createTodoElement(todo) {
      const li = document.createElement('li');
      li.className = `todo-item${todo.completed ? ' completed' : ''}`;
      li.dataset.id = todo.id;

      const checkbox = document.createElement('input');
      checkbox.type = 'checkbox';
      checkbox.checked = todo.completed;
      checkbox.addEventListener('change', () => toggleTodo(todo.id));

      const span = document.createElement('span');
      span.textContent = todo.text;

      const editBtn = document.createElement('button');
      editBtn.textContent = 'แก้ไข';
      editBtn.addEventListener('click', () => editTodo(todo.id, li));

      const deleteBtn = document.createElement('button');
      deleteBtn.textContent = 'ลบ';
      deleteBtn.style.backgroundColor = '#ff4444';
      deleteBtn.style.color = 'white';
      deleteBtn.addEventListener('click', () => deleteTodo(todo.id));

      li.append(checkbox, span, editBtn, deleteBtn);
      return li;
    }

    function render() {
      todoList.innerHTML = '';
      todos.forEach(todo => {
        todoList.appendChild(createTodoElement(todo));
      });
      updateStats();
    }

    function updateStats() {
      const total = todos.length;
      const done = todos.filter(t => t.completed).length;
      stats.textContent = `เสร็จแล้ว ${done}/${total} รายการ`;
    }

    function addTodo(text) {
      if (!text.trim()) return;
      todos.push({ id: nextId++, text: text.trim(), completed: false });
      render();
    }

    function toggleTodo(id) {
      const todo = todos.find(t => t.id === id);
      if (todo) {
        todo.completed = !todo.completed;
        // แค่ update element ที่เปลี่ยน แทน render ทั้งหมด
        const li = todoList.querySelector(`[data-id="${id}"]`);
        li.classList.toggle('completed', todo.completed);
        li.querySelector('input').checked = todo.completed;
        updateStats();
      }
    }

    function deleteTodo(id) {
      todos = todos.filter(t => t.id !== id);
      const li = todoList.querySelector(`[data-id="${id}"]`);
      li.remove();
      updateStats();
    }

    function editTodo(id, li) {
      const todo = todos.find(t => t.id === id);
      const span = li.querySelector('span');
      const editBtn = li.querySelector('button');

      if (editBtn.textContent === 'แก้ไข') {
        const input = document.createElement('input');
        input.value = todo.text;
        input.style.flex = '1';
        span.replaceWith(input);
        editBtn.textContent = 'บันทึก';
        input.focus();
      } else {
        const input = li.querySelector('input[type="text"]');
        todo.text = input.value.trim() || todo.text;
        const newSpan = document.createElement('span');
        newSpan.textContent = todo.text;
        input.replaceWith(newSpan);
        editBtn.textContent = 'แก้ไข';
      }
    }

    addBtn.addEventListener('click', () => {
      addTodo(newTodoInput.value);
      newTodoInput.value = '';
      newTodoInput.focus();
    });

    newTodoInput.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') {
        addTodo(newTodoInput.value);
        newTodoInput.value = '';
      }
    });

    render();
  </script>
</body>
</html>
```

---

## Step 210: ตัวอย่างสรุป - Complete DOM Manipulation

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <title>DOM Summary</title>
  <style>
    .highlight { background-color: yellow; }
    .selected { outline: 2px solid blue; }
    .dragging { opacity: 0.5; }
    #gallery { display: flex; flex-wrap: wrap; gap: 10px; }
    .thumb { width: 100px; height: 100px; background: #ddd; cursor: pointer; display: flex; align-items: center; justify-content: center; border-radius: 4px; }
    .thumb.selected { outline: 3px solid blue; }
    #info { margin-top: 10px; padding: 10px; background: #f9f9f9; border-radius: 4px; }
  </style>
</head>
<body>
  <h1 id="page-title">DOM Manipulation</h1>

  <div id="controls">
    <input type="text" id="search" placeholder="ค้นหา...">
    <button id="add-item">เพิ่มรายการ</button>
    <button id="remove-selected">ลบที่เลือก</button>
    <button id="select-all">เลือกทั้งหมด</button>
  </div>

  <div id="gallery"></div>
  <div id="info">เลือก item เพื่อดูรายละเอียด</div>

  <script>
    const gallery = document.getElementById('gallery');
    const info = document.getElementById('info');
    const searchInput = document.getElementById('search');
    let selectedItems = new Set();
    let itemCount = 0;

    function createItem(id, label) {
      const div = document.createElement('div');
      div.className = 'thumb';
      div.dataset.id = id;
      div.textContent = label;

      div.addEventListener('click', (e) => {
        if (e.ctrlKey || e.metaKey) {
          div.classList.toggle('selected');
          if (div.classList.contains('selected')) {
            selectedItems.add(id);
          } else {
            selectedItems.delete(id);
          }
        } else {
          document.querySelectorAll('.thumb.selected').forEach(el => {
            el.classList.remove('selected');
            selectedItems.delete(Number(el.dataset.id));
          });
          div.classList.add('selected');
          selectedItems.add(id);
        }
        updateInfo();
      });

      return div;
    }

    function updateInfo() {
      if (selectedItems.size === 0) {
        info.textContent = 'เลือก item เพื่อดูรายละเอียด';
      } else {
        info.textContent = `เลือกแล้ว ${selectedItems.size} items: ${[...selectedItems].join(', ')}`;
      }
    }

    // เพิ่ม items เริ่มต้น
    for (let i = 1; i <= 6; i++) {
      gallery.appendChild(createItem(i, `Item ${i}`));
      itemCount = i;
    }

    // เพิ่มรายการใหม่
    document.getElementById('add-item').addEventListener('click', () => {
      itemCount++;
      const newItem = createItem(itemCount, `Item ${itemCount}`);
      newItem.style.background = '#c8e6c9';
      gallery.appendChild(newItem);
    });

    // ลบที่เลือก
    document.getElementById('remove-selected').addEventListener('click', () => {
      selectedItems.forEach(id => {
        const el = gallery.querySelector(`[data-id="${id}"]`);
        if (el) el.remove();
      });
      selectedItems.clear();
      updateInfo();
    });

    // เลือกทั้งหมด
    document.getElementById('select-all').addEventListener('click', () => {
      gallery.querySelectorAll('.thumb').forEach(el => {
        el.classList.add('selected');
        selectedItems.add(Number(el.dataset.id));
      });
      updateInfo();
    });

    // ค้นหา
    searchInput.addEventListener('input', (e) => {
      const query = e.target.value.toLowerCase();
      gallery.querySelectorAll('.thumb').forEach(el => {
        const match = el.textContent.toLowerCase().includes(query);
        el.style.display = match ? '' : 'none';
      });
    });
  </script>
</body>
</html>
```

---

## สรุป Steps 191-210

| Step | หัวข้อ | สิ่งที่เรียนรู้ |
|------|--------|----------------|
| 191 | What is DOM | document object, node types |
| 192 | DOM Tree | parent, child, sibling relationships |
| 193 | getElementById | การเลือก element ด้วย ID |
| 194 | getElementsBy* | HTMLCollection, live collections |
| 195 | querySelector | CSS selector syntax |
| 196 | Navigation | parentNode, childNodes, firstChild |
| 197 | Element Navigation | parentElement, children, firstElementChild |
| 198 | createElement | สร้าง elements ใหม่ |
| 199 | appendChild/insertBefore | เพิ่ม elements |
| 200 | insertAdjacentHTML | เพิ่ม HTML อย่างยืดหยุ่น |
| 201 | removeChild/remove | ลบ elements |
| 202 | replaceChild/replaceWith | แทน elements |
| 203 | cloneNode | copy elements |
| 204 | innerHTML/textContent | อ่าน/เขียน content |
| 205 | getAttribute/setAttribute | จัดการ attributes |
| 206 | classList | จัดการ CSS classes |
| 207 | style property | inline styles |
| 208 | dataset | data-* attributes |
| 209 | Dynamic List | ตัวอย่างจริง |
| 210 | Complete Example | รวมทุกอย่าง |

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Dynamic Table
สร้าง table ที่:
- มีปุ่มเพิ่มแถว
- สามารถแก้ไขเซลล์ได้ด้วยการ double click
- มีปุ่มลบแต่ละแถว
- แสดงจำนวนแถวทั้งหมด

### แบบฝึกหัดที่ 2: Image Gallery
สร้าง gallery ที่:
- แสดงรูปภาพในรูปแบบ grid
- คลิกเพื่อ select (เปลี่ยน border)
- มีปุ่ม "เลือกทั้งหมด" และ "ยกเลิกทั้งหมด"
- แสดงจำนวน items ที่เลือก

### แบบฝึกหัดที่ 3: Accordion
สร้าง accordion component ที่:
- มีหลาย sections
- คลิก header เพื่อ expand/collapse
- ใช้ classList.toggle สำหรับ animation
- เก็บ state ด้วย data attributes

### แบบฝึกหัดที่ 4: Drag and Drop List
สร้าง list ที่:
- ลาก items เพื่อ reorder
- ใช้ DOM methods ในการย้าย elements
- แสดง visual feedback ระหว่าง drag

### แบบฝึกหัดที่ 5: Mini Spreadsheet
สร้าง spreadsheet ขนาดเล็ก ที่:
- สร้าง table 5x5 ด้วย createElement
- คลิกเพื่อ select cell
- พิมพ์ข้อมูลลงใน cell
- คำนวณ sum ของ column อัตโนมัติ
