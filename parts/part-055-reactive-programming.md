# Part 55: Reactive Programming (Steps 1071-1090)

## การเขียนโปรแกรมแบบ Reactive

**คำอธิบาย:** Reactive Programming เป็นแนวคิดที่จัดการ data streams และการเปลี่ยนแปลงแบบ asynchronous ด้วยวิธีที่ declarative และ composable เราจะสร้าง Observable จากพื้นฐาน เรียนรู้ operators และนำไปใช้กับ RxJS

---

## Step 1071: Reactive Programming คืออะไร?

```javascript
// ========================
// ปัญหาที่ Reactive Programming แก้
// ========================

// Imperative (traditional)
function fetchUserData(userId) {
  fetch(`/api/users/${userId}`)
    .then(res => res.json())
    .then(user => {
      document.getElementById('name').textContent = user.name;
      fetch(`/api/users/${userId}/orders`)
        .then(res => res.json())
        .then(orders => {
          document.getElementById('orders').textContent = orders.length;
        });
    })
    .catch(err => console.error(err));
}
// Callback hell ปัญหา, ยากต่อการ compose

// Reactive (declarative)
// userStream.pipe(
//   switchMap(user => combineLatest([of(user), getOrders(user.id)])),
//   map(([user, orders]) => ({ ...user, orderCount: orders.length }))
// ).subscribe(renderUser);

// แนวคิดหลัก
console.log('Reactive Programming = Everything is a Stream');

// ตัวอย่าง streams ในชีวิตจริง
const streams = {
  userEvents: 'clicks, keypresses, mouse moves',
  data: 'API responses, WebSocket messages',
  time: 'setTimeout, setInterval, animations',
  state: 'form inputs, store changes'
};

// Marble Diagram
// --1--2--3--4--5--|  (values over time, | = complete)
// map(x => x * 2)
// --2--4--6--8--10-| (transformed stream)
```

---

## Step 1072: สร้าง Observable จากพื้นฐาน

```javascript
// ========================
// Observable จากพื้นฐาน
// ========================

class Observable {
  constructor(subscribeFn) {
    this._subscribeFn = subscribeFn;
  }
  
  // Subscribe เริ่มการทำงาน
  subscribe(observerOrNext, onError, onComplete) {
    // รองรับทั้ง Observer object และ callbacks
    const observer = typeof observerOrNext === 'function'
      ? {
          next: observerOrNext,
          error: onError || ((err) => console.error('Unhandled error:', err)),
          complete: onComplete || (() => {})
        }
      : {
          next: observerOrNext.next || (() => {}),
          error: observerOrNext.error || ((err) => console.error('Unhandled:', err)),
          complete: observerOrNext.complete || (() => {})
        };
    
    // Wrap observer เพื่อป้องกันหลังจาก complete/error
    let isComplete = false;
    const safeObserver = {
      next: (value) => {
        if (!isComplete) observer.next(value);
      },
      error: (err) => {
        if (!isComplete) {
          isComplete = true;
          observer.error(err);
        }
      },
      complete: () => {
        if (!isComplete) {
          isComplete = true;
          observer.complete();
        }
      }
    };
    
    // รัน subscribe function
    let cleanup;
    try {
      cleanup = this._subscribeFn(safeObserver);
    } catch (err) {
      safeObserver.error(err);
    }
    
    // Return subscription object
    return {
      unsubscribe: () => {
        isComplete = true;
        if (cleanup && typeof cleanup === 'function') cleanup();
      }
    };
  }
  
  // ========================
  // Static factory methods
  // ========================
  
  // สร้างจาก values
  static of(...values) {
    return new Observable(observer => {
      values.forEach(value => observer.next(value));
      observer.complete();
    });
  }
  
  // สร้างจาก array/iterable
  static from(iterable) {
    return new Observable(observer => {
      try {
        for (const item of iterable) {
          observer.next(item);
        }
        observer.complete();
      } catch (err) {
        observer.error(err);
      }
    });
  }
  
  // สร้าง interval
  static interval(ms) {
    return new Observable(observer => {
      let count = 0;
      const id = setInterval(() => {
        observer.next(count++);
      }, ms);
      
      return () => clearInterval(id); // cleanup
    });
  }
  
  // สร้าง timer
  static timer(delay, period = null) {
    return new Observable(observer => {
      const timeout = setTimeout(() => {
        observer.next(0);
        
        if (period !== null) {
          let count = 1;
          const id = setInterval(() => {
            observer.next(count++);
          }, period);
          // Cannot return cleanup here - limitation
        } else {
          observer.complete();
        }
      }, delay);
      
      return () => clearTimeout(timeout);
    });
  }
  
  // สร้างจาก Promise
  static fromPromise(promise) {
    return new Observable(observer => {
      promise
        .then(value => {
          observer.next(value);
          observer.complete();
        })
        .catch(err => observer.error(err));
    });
  }
  
  // Observable ที่ complete ทันที (no values)
  static empty() {
    return new Observable(observer => {
      observer.complete();
    });
  }
  
  // Observable ที่ไม่ complete เลย
  static never() {
    return new Observable(() => {}); // ไม่ emit อะไร
  }
  
  // Observable ที่ throw error ทันที
  static throwError(error) {
    return new Observable(observer => {
      observer.error(error);
    });
  }
}

// ========================
// ทดสอบ Observable
// ========================

// 1. จาก values
const values$ = Observable.of(1, 2, 3, 4, 5);
values$.subscribe({
  next: v => console.log('Value:', v),
  complete: () => console.log('Complete!')
});

// 2. จาก array
const arr$ = Observable.from([10, 20, 30]);
arr$.subscribe(v => console.log('Array item:', v));

// 3. Interval (จะ unsubscribe หลัง 5 ครั้ง)
const interval$ = Observable.interval(100);
let count = 0;
const sub = interval$.subscribe(i => {
  console.log('Tick:', i);
  if (++count >= 3) sub.unsubscribe();
});

// 4. จาก Promise
const promise$ = Observable.fromPromise(
  fetch('https://jsonplaceholder.typicode.com/todos/1')
    .then(r => r.json())
    .catch(() => ({ id: 1, title: 'Mock data' }))
);

promise$.subscribe({
  next: data => console.log('Promise data:', data),
  error: err => console.error('Error:', err)
});
```

---

## Step 1073: Operators พื้นฐาน

```javascript
// ========================
// Operators
// ========================

// เพิ่ม pipe method และ operators ให้ Observable
class Observable2 extends Observable {
  pipe(...operators) {
    return operators.reduce((obs, op) => op(obs), this);
  }
}

// Operator factories
function map(fn) {
  return (source) => new Observable2(observer => {
    return source.subscribe({
      next: value => {
        try {
          observer.next(fn(value));
        } catch (err) {
          observer.error(err);
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

function filter(predicate) {
  return (source) => new Observable2(observer => {
    return source.subscribe({
      next: value => {
        try {
          if (predicate(value)) observer.next(value);
        } catch (err) {
          observer.error(err);
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

function take(n) {
  return (source) => new Observable2(observer => {
    let count = 0;
    const sub = source.subscribe({
      next: value => {
        if (count < n) {
          observer.next(value);
          count++;
          if (count >= n) {
            observer.complete();
            sub.unsubscribe();
          }
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
    
    return () => sub.unsubscribe();
  });
}

function skip(n) {
  return (source) => new Observable2(observer => {
    let count = 0;
    return source.subscribe({
      next: value => {
        if (count >= n) observer.next(value);
        count++;
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

function scan(fn, seed) {
  return (source) => new Observable2(observer => {
    let acc = seed;
    let hasSeed = arguments.length >= 2;
    
    return source.subscribe({
      next: value => {
        if (!hasSeed) {
          acc = value;
          hasSeed = true;
          return;
        }
        try {
          acc = fn(acc, value);
          observer.next(acc);
        } catch (err) {
          observer.error(err);
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

function reduce(fn, seed) {
  return (source) => new Observable2(observer => {
    let acc = seed;
    let hasSeed = seed !== undefined;
    
    return source.subscribe({
      next: value => {
        if (!hasSeed) {
          acc = value;
          hasSeed = true;
          return;
        }
        acc = fn(acc, value);
      },
      error: err => observer.error(err),
      complete: () => {
        observer.next(acc);
        observer.complete();
      }
    });
  });
}

function tap(fn) {
  return (source) => new Observable2(observer => {
    return source.subscribe({
      next: value => {
        try { fn(value); } catch {}
        observer.next(value);
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

function distinct() {
  return (source) => new Observable2(observer => {
    const seen = new Set();
    return source.subscribe({
      next: value => {
        if (!seen.has(value)) {
          seen.add(value);
          observer.next(value);
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

function distinctUntilChanged(comparator = (a, b) => a === b) {
  return (source) => new Observable2(observer => {
    let prevValue;
    let hasValue = false;
    
    return source.subscribe({
      next: value => {
        if (!hasValue || !comparator(prevValue, value)) {
          prevValue = value;
          hasValue = true;
          observer.next(value);
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

// ========================
// ใช้งาน Operators
// ========================

// สร้าง Observable ที่ใช้ pipe ได้
function of2(...values) {
  return new Observable2(observer => {
    values.forEach(v => observer.next(v));
    observer.complete();
  });
}

function from2(iterable) {
  return new Observable2(observer => {
    for (const item of iterable) observer.next(item);
    observer.complete();
  });
}

// ทดสอบ
const result = of2(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
  .pipe(
    tap(v => console.log(`Before filter: ${v}`)),
    filter(v => v % 2 === 0),
    map(v => v * v),
    take(3)
  );

result.subscribe({
  next: v => console.log('Result:', v),
  complete: () => console.log('Done')
});

// Scan สำหรับ running total
console.log('\nRunning total:');
from2([1, 2, 3, 4, 5])
  .pipe(scan((acc, v) => acc + v, 0))
  .subscribe(v => console.log(v)); // 1, 3, 6, 10, 15

// Distinct
console.log('\nDistinct:');
from2([1, 2, 2, 3, 3, 3, 4])
  .pipe(distinct())
  .subscribe(v => console.log(v)); // 1, 2, 3, 4
```

---

## Step 1074: mergeMap, switchMap, concatMap

```javascript
// ========================
// Higher-order Observables
// ========================

// mergeMap: รัน inner observables พร้อมกัน
function mergeMap(project) {
  return (source) => new Observable2(observer => {
    let active = 0;
    let sourceComplete = false;
    const subscriptions = new Set();
    
    const sourceSubscription = source.subscribe({
      next: value => {
        active++;
        const innerObs = project(value);
        const innerSub = innerObs.subscribe({
          next: innerValue => observer.next(innerValue),
          error: err => observer.error(err),
          complete: () => {
            active--;
            subscriptions.delete(innerSub);
            if (sourceComplete && active === 0) observer.complete();
          }
        });
        subscriptions.add(innerSub);
      },
      error: err => observer.error(err),
      complete: () => {
        sourceComplete = true;
        if (active === 0) observer.complete();
      }
    });
    
    return () => {
      sourceSubscription.unsubscribe();
      subscriptions.forEach(sub => sub.unsubscribe());
    };
  });
}

// switchMap: cancel previous, start new
function switchMap(project) {
  return (source) => new Observable2(observer => {
    let innerSubscription = null;
    let sourceComplete = false;
    
    const sourceSubscription = source.subscribe({
      next: value => {
        // Cancel previous inner observable
        if (innerSubscription) innerSubscription.unsubscribe();
        
        const innerObs = project(value);
        innerSubscription = innerObs.subscribe({
          next: innerValue => observer.next(innerValue),
          error: err => observer.error(err),
          complete: () => {
            innerSubscription = null;
            if (sourceComplete) observer.complete();
          }
        });
      },
      error: err => observer.error(err),
      complete: () => {
        sourceComplete = true;
        if (!innerSubscription) observer.complete();
      }
    });
    
    return () => {
      sourceSubscription.unsubscribe();
      if (innerSubscription) innerSubscription.unsubscribe();
    };
  });
}

// concatMap: รอให้ inner complete ก่อน start next
function concatMap(project) {
  return (source) => new Observable2(observer => {
    const queue = [];
    let active = false;
    let sourceComplete = false;
    
    function processNext() {
      if (active || queue.length === 0) return;
      
      active = true;
      const value = queue.shift();
      const innerObs = project(value);
      
      innerObs.subscribe({
        next: innerValue => observer.next(innerValue),
        error: err => observer.error(err),
        complete: () => {
          active = false;
          if (queue.length > 0) {
            processNext();
          } else if (sourceComplete) {
            observer.complete();
          }
        }
      });
    }
    
    return source.subscribe({
      next: value => {
        queue.push(value);
        processNext();
      },
      error: err => observer.error(err),
      complete: () => {
        sourceComplete = true;
        if (!active && queue.length === 0) observer.complete();
      }
    });
  });
}

// ========================
// ใช้งาน
// ========================

function createDelay(value, delay) {
  return new Observable2(observer => {
    const id = setTimeout(() => {
      observer.next(value);
      observer.complete();
    }, delay);
    return () => clearTimeout(id);
  });
}

// mergeMap: ทำพร้อมกัน
console.log('=== mergeMap ===');
from2([1, 2, 3])
  .pipe(
    mergeMap(n => createDelay(`response-${n}`, n * 100))
  )
  .subscribe(v => console.log('mergeMap:', v));
// order: response-1, response-2, response-3 (by delay time)

// switchMap: cancel เมื่อมีค่าใหม่
console.log('\n=== switchMap ===');
from2(['a', 'ab', 'abc', 'abcd'])
  .pipe(
    switchMap(query => {
      // จำลอง search API
      return createDelay(`results for "${query}"`, 150);
    })
  )
  .subscribe(v => console.log('switchMap:', v));
// ได้แค่ "results for abcd" เพราะ cancel ของก่อนหน้า

// concatMap: ทำทีละอัน
console.log('\n=== concatMap ===');
from2([1, 2, 3])
  .pipe(
    concatMap(n => createDelay(`sequential-${n}`, 100))
  )
  .subscribe(v => console.log('concatMap:', v));
// sequential-1 แล้ว sequential-2 แล้ว sequential-3
```

---

## Step 1075: combineLatest, zip, merge, concat

```javascript
// ========================
// Combination Operators
// ========================

// merge: รวม observables ทำงานพร้อมกัน
function merge(...observables) {
  return new Observable2(observer => {
    let active = observables.length;
    if (active === 0) {
      observer.complete();
      return;
    }
    
    const subscriptions = observables.map(obs => 
      obs.subscribe({
        next: value => observer.next(value),
        error: err => observer.error(err),
        complete: () => {
          active--;
          if (active === 0) observer.complete();
        }
      })
    );
    
    return () => subscriptions.forEach(sub => sub.unsubscribe());
  });
}

// concat: รัน observables ทีละอัน
function concat(...observables) {
  return new Observable2(observer => {
    if (observables.length === 0) {
      observer.complete();
      return;
    }
    
    let index = 0;
    let currentSub;
    
    function subscribeToNext() {
      if (index >= observables.length) {
        observer.complete();
        return;
      }
      
      const obs = observables[index++];
      currentSub = obs.subscribe({
        next: value => observer.next(value),
        error: err => observer.error(err),
        complete: () => subscribeToNext()
      });
    }
    
    subscribeToNext();
    return () => { if (currentSub) currentSub.unsubscribe(); };
  });
}

// combineLatest: รวมค่าล่าสุดจากทุก observable
function combineLatest(observables) {
  return new Observable2(observer => {
    const values = new Array(observables.length).fill(undefined);
    const hasValue = new Array(observables.length).fill(false);
    let completedCount = 0;
    
    const subscriptions = observables.map((obs, i) => 
      obs.subscribe({
        next: value => {
          values[i] = value;
          hasValue[i] = true;
          
          if (hasValue.every(Boolean)) {
            observer.next([...values]);
          }
        },
        error: err => observer.error(err),
        complete: () => {
          completedCount++;
          if (completedCount === observables.length) {
            observer.complete();
          }
        }
      })
    );
    
    return () => subscriptions.forEach(sub => sub.unsubscribe());
  });
}

// zip: จับคู่ values ที่ตำแหน่งเดียวกัน
function zip(observables) {
  return new Observable2(observer => {
    const queues = observables.map(() => []);
    let completedCount = 0;
    
    const subscriptions = observables.map((obs, i) => 
      obs.subscribe({
        next: value => {
          queues[i].push(value);
          
          if (queues.every(q => q.length > 0)) {
            observer.next(queues.map(q => q.shift()));
          }
        },
        error: err => observer.error(err),
        complete: () => {
          completedCount++;
          if (completedCount === observables.length) {
            observer.complete();
          }
        }
      })
    );
    
    return () => subscriptions.forEach(sub => sub.unsubscribe());
  });
}

// withLatestFrom
function withLatestFrom(other) {
  return (source) => new Observable2(observer => {
    let latestOther;
    let hasOther = false;
    
    const otherSub = other.subscribe({
      next: value => {
        latestOther = value;
        hasOther = true;
      },
      error: err => observer.error(err)
    });
    
    const sourceSub = source.subscribe({
      next: value => {
        if (hasOther) {
          observer.next([value, latestOther]);
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
    
    return () => {
      sourceSub.unsubscribe();
      otherSub.unsubscribe();
    };
  });
}

// ========================
// ใช้งาน
// ========================

// merge
console.log('=== merge ===');
const obs1 = of2(1, 2, 3);
const obs2 = of2('a', 'b', 'c');
merge(obs1, obs2).subscribe(v => console.log('merged:', v));

// concat
console.log('\n=== concat ===');
concat(of2(1, 2, 3), of2('a', 'b', 'c')).subscribe(v => console.log('concat:', v));

// combineLatest
console.log('\n=== combineLatest ===');
const firstName$ = of2('สมชาย');
const lastName$ = of2('ใจดี');
combineLatest([firstName$, lastName$]).subscribe(
  ([first, last]) => console.log('Combined:', `${first} ${last}`)
);

// zip
console.log('\n=== zip ===');
const names$ = of2('สมชาย', 'สมหญิง', 'วิชัย');
const ages$ = of2(25, 30, 28);
zip([names$, ages$]).subscribe(
  ([name, age]) => console.log(`${name}: ${age}`)
);
```

---

## Step 1076: Subject - Hot Observable

```javascript
// ========================
// Subject - Multicast Observable
// ========================

class Subject extends Observable2 {
  constructor() {
    super(observer => {
      this._observers.add(observer);
      return () => this._observers.delete(observer);
    });
    
    this._observers = new Set();
    this._isComplete = false;
    this._error = null;
  }
  
  // สามารถ emit ค่าได้จากภายนอก
  next(value) {
    if (this._isComplete) return;
    this._observers.forEach(obs => {
      try { obs.next(value); } catch {}
    });
  }
  
  error(err) {
    if (this._isComplete) return;
    this._isComplete = true;
    this._error = err;
    this._observers.forEach(obs => {
      try { obs.error(err); } catch {}
    });
    this._observers.clear();
  }
  
  complete() {
    if (this._isComplete) return;
    this._isComplete = true;
    this._observers.forEach(obs => {
      try { obs.complete(); } catch {}
    });
    this._observers.clear();
  }
  
  asObservable() {
    return new Observable2(observer => {
      return this.subscribe(observer);
    });
  }
}

// BehaviorSubject - มี current value
class BehaviorSubject extends Subject {
  constructor(initialValue) {
    super();
    this._currentValue = initialValue;
  }
  
  getValue() {
    return this._currentValue;
  }
  
  next(value) {
    this._currentValue = value;
    super.next(value);
  }
  
  subscribe(observer) {
    // ส่ง current value ให้ subscriber ใหม่ทันที
    const sub = super.subscribe(observer);
    
    const obs = typeof observer === 'function' 
      ? { next: observer }
      : observer;
    
    if (!this._isComplete) {
      obs.next && obs.next(this._currentValue);
    }
    
    return sub;
  }
}

// ReplaySubject - replay N values ให้ subscribers ใหม่
class ReplaySubject extends Subject {
  constructor(bufferSize = Infinity) {
    super();
    this._buffer = [];
    this._bufferSize = bufferSize;
  }
  
  next(value) {
    this._buffer.push(value);
    if (this._buffer.length > this._bufferSize) {
      this._buffer.shift();
    }
    super.next(value);
  }
  
  subscribe(observer) {
    const obs = typeof observer === 'function'
      ? { next: observer, error: () => {}, complete: () => {} }
      : { next: () => {}, error: () => {}, complete: () => {}, ...observer };
    
    // Replay buffered values
    this._buffer.forEach(value => {
      try { obs.next(value); } catch {}
    });
    
    if (this._isComplete) {
      obs.complete();
      return { unsubscribe: () => {} };
    }
    
    return super.subscribe(observer);
  }
}

// ========================
// ใช้งาน Subjects
// ========================

// Subject ใช้สำหรับ EventBus
const eventBus = new Subject();

// Subscriber 1
const sub1 = eventBus.subscribe(event => {
  console.log('[Sub1] received:', event);
});

eventBus.next({ type: 'USER_LOGGED_IN', user: 'สมชาย' });

// Subscriber 2 - เข้ามาทีหลัง จะไม่เห็น USER_LOGGED_IN
const sub2 = eventBus.subscribe(event => {
  console.log('[Sub2] received:', event);
});

eventBus.next({ type: 'CART_UPDATED', items: 3 });
eventBus.next({ type: 'PAGE_VIEWED', page: '/home' });

sub1.unsubscribe();
eventBus.next({ type: 'USER_LOGGED_OUT' }); // sub1 จะไม่เห็น

// BehaviorSubject
console.log('\n=== BehaviorSubject ===');
const currentUser$ = new BehaviorSubject(null);

currentUser$.subscribe(user => {
  console.log('Current user:', user ? user.name : 'Not logged in');
});

currentUser$.next({ id: 1, name: 'สมชาย' });
currentUser$.next({ id: 1, name: 'สมชาย', role: 'admin' });

// Subscriber ใหม่จะได้รับ current value ทันที
currentUser$.subscribe(user => {
  console.log('New subscriber sees:', user ? user.name : 'Not logged in');
});

console.log('getValue():', currentUser$.getValue());

// ReplaySubject
console.log('\n=== ReplaySubject ===');
const recentEvents$ = new ReplaySubject(3); // buffer 3 events

recentEvents$.next('event1');
recentEvents$.next('event2');
recentEvents$.next('event3');
recentEvents$.next('event4'); // event1 ออกจาก buffer

// Subscriber ใหม่จะได้รับ 3 events ล่าสุด
recentEvents$.subscribe(e => console.log('Replayed:', e));
// Gets event2, event3, event4
```

---

## Step 1077: Error Handling ใน Reactive Programming

```javascript
// ========================
// Error Handling
// ========================

function catchError(handler) {
  return (source) => new Observable2(observer => {
    return source.subscribe({
      next: value => observer.next(value),
      error: err => {
        try {
          const recovery = handler(err, source);
          recovery.subscribe(observer);
        } catch (handlerError) {
          observer.error(handlerError);
        }
      },
      complete: () => observer.complete()
    });
  });
}

function retry(count) {
  return (source) => new Observable2(observer => {
    let attempts = 0;
    
    function subscribe() {
      return source.subscribe({
        next: value => observer.next(value),
        error: err => {
          if (attempts < count) {
            attempts++;
            console.log(`Retry attempt ${attempts}...`);
            subscribe();
          } else {
            observer.error(err);
          }
        },
        complete: () => observer.complete()
      });
    }
    
    return subscribe();
  });
}

function retryWhen(notifier) {
  return (source) => new Observable2(observer => {
    let attempts = 0;
    
    function subscribe() {
      return source.subscribe({
        next: value => observer.next(value),
        error: err => {
          attempts++;
          const notification$ = notifier(err, attempts);
          notification$.subscribe({
            next: () => subscribe(),
            error: innerErr => observer.error(innerErr),
            complete: () => observer.error(err)
          });
        },
        complete: () => observer.complete()
      });
    }
    
    return subscribe();
  });
}

function timeout(ms) {
  return (source) => new Observable2(observer => {
    let lastActivity = Date.now();
    
    const timeoutId = setInterval(() => {
      if (Date.now() - lastActivity >= ms) {
        clearInterval(timeoutId);
        observer.error(new Error(`Observable timed out after ${ms}ms`));
        sub.unsubscribe();
      }
    }, 10);
    
    const sub = source.subscribe({
      next: value => {
        lastActivity = Date.now();
        observer.next(value);
      },
      error: err => {
        clearInterval(timeoutId);
        observer.error(err);
      },
      complete: () => {
        clearInterval(timeoutId);
        observer.complete();
      }
    });
    
    return () => {
      clearInterval(timeoutId);
      sub.unsubscribe();
    };
  });
}

function defaultIfEmpty(defaultValue) {
  return (source) => new Observable2(observer => {
    let hasValue = false;
    
    return source.subscribe({
      next: value => {
        hasValue = true;
        observer.next(value);
      },
      error: err => observer.error(err),
      complete: () => {
        if (!hasValue) observer.next(defaultValue);
        observer.complete();
      }
    });
  });
}

// ========================
// ใช้งาน Error Handling
// ========================

// catchError
function createFailingObservable(n) {
  return new Observable2(observer => {
    if (n < 3) {
      observer.error(new Error(`Failed at ${n}`));
    } else {
      observer.next(n);
      observer.complete();
    }
  });
}

from2([1, 2, 3, 4])
  .pipe(
    mergeMap(n => createFailingObservable(n).pipe(
      catchError(err => of2(`Recovered from: ${err.message}`))
    ))
  )
  .subscribe(v => console.log(v));

// retry
let attempt = 0;
const flaky$ = new Observable2(observer => {
  attempt++;
  if (attempt < 3) {
    observer.error(new Error(`Attempt ${attempt} failed`));
  } else {
    observer.next('Success on attempt 3!');
    observer.complete();
  }
});

flaky$.pipe(retry(3)).subscribe({
  next: v => console.log(v),
  error: err => console.error('Final error:', err.message)
});

// defaultIfEmpty
Observable2.empty()
  .pipe(defaultIfEmpty('No data available'))
  .subscribe(v => console.log(v));
```

---

## Step 1078: Debounce, Throttle, และ Time Operators

```javascript
// ========================
// Time-based Operators
// ========================

function debounceTime(ms) {
  return (source) => new Observable2(observer => {
    let timeoutId;
    
    const sub = source.subscribe({
      next: value => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
          observer.next(value);
        }, ms);
      },
      error: err => {
        clearTimeout(timeoutId);
        observer.error(err);
      },
      complete: () => {
        clearTimeout(timeoutId);
        observer.complete();
      }
    });
    
    return () => {
      clearTimeout(timeoutId);
      sub.unsubscribe();
    };
  });
}

function throttleTime(ms) {
  return (source) => new Observable2(observer => {
    let lastEmit = 0;
    
    return source.subscribe({
      next: value => {
        const now = Date.now();
        if (now - lastEmit >= ms) {
          lastEmit = now;
          observer.next(value);
        }
      },
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

function delay(ms) {
  return (source) => new Observable2(observer => {
    const subs = new Set();
    
    const sub = source.subscribe({
      next: value => {
        const id = setTimeout(() => {
          subs.delete(id);
          observer.next(value);
        }, ms);
        subs.add(id);
      },
      error: err => observer.error(err),
      complete: () => {
        const id = setTimeout(() => {
          subs.delete(id);
          observer.complete();
        }, ms);
        subs.add(id);
      }
    });
    
    return () => {
      sub.unsubscribe();
      subs.forEach(id => clearTimeout(id));
    };
  });
}

function bufferTime(ms) {
  return (source) => new Observable2(observer => {
    let buffer = [];
    
    const intervalId = setInterval(() => {
      if (buffer.length > 0) {
        observer.next([...buffer]);
        buffer = [];
      }
    }, ms);
    
    const sub = source.subscribe({
      next: value => buffer.push(value),
      error: err => {
        clearInterval(intervalId);
        observer.error(err);
      },
      complete: () => {
        clearInterval(intervalId);
        if (buffer.length > 0) observer.next([...buffer]);
        observer.complete();
      }
    });
    
    return () => {
      clearInterval(intervalId);
      sub.unsubscribe();
    };
  });
}

function auditTime(ms) {
  return (source) => new Observable2(observer => {
    let lastValue;
    let hasValue = false;
    let timeoutId;
    
    const sub = source.subscribe({
      next: value => {
        lastValue = value;
        hasValue = true;
        
        if (!timeoutId) {
          timeoutId = setTimeout(() => {
            timeoutId = null;
            if (hasValue) {
              hasValue = false;
              observer.next(lastValue);
            }
          }, ms);
        }
      },
      error: err => {
        clearTimeout(timeoutId);
        observer.error(err);
      },
      complete: () => {
        clearTimeout(timeoutId);
        observer.complete();
      }
    });
    
    return () => {
      clearTimeout(timeoutId);
      sub.unsubscribe();
    };
  });
}

// ========================
// ตัวอย่างการใช้งาน
// ========================

// จำลอง keystroke events
const searchInput$ = new Subject();
let keystrokeCount = 0;

// Debounce - wait until user stops typing
const debouncedSearch$ = searchInput$.pipe(
  tap(v => console.log(`Keystroke: "${v}"`)),
  debounceTime(300) // รอ 300ms หลังจาก keystroke สุดท้าย
);

debouncedSearch$.subscribe(query => {
  console.log(`Searching for: "${query}"`);
});

// จำลองการพิมพ์
const typing = ['s', 'sa', 'sal', 'sale', 'sale '];

let i = 0;
const typeInterval = setInterval(() => {
  if (i < typing.length) {
    searchInput$.next(typing[i++]);
  } else {
    clearInterval(typeInterval);
    searchInput$.complete();
  }
}, 100);

// Throttle - limit rate
console.log('\n=== Throttle ===');
const scroll$ = new Subject();

scroll$.pipe(throttleTime(500)).subscribe(
  position => console.log('Scroll position:', position)
);

// จำลอง scroll events
[100, 200, 300, 400, 500].forEach((pos, i) => {
  setTimeout(() => scroll$.next(pos), i * 50);
});

// Buffer - collect values
console.log('\n=== Buffer ===');
const clicks$ = new Subject();

clicks$.pipe(
  bufferTime(500)
).subscribe(clickBatch => {
  if (clickBatch.length > 0) {
    console.log(`${clickBatch.length} clicks in last 500ms`);
  }
});

// จำลอง rapid clicks
['click1', 'click2', 'click3', 'click4'].forEach((c, i) => {
  setTimeout(() => clicks$.next(c), i * 100);
});
```

---

## Step 1079: RxJS Overview

```javascript
// ========================
// RxJS - The Real Deal
// ========================

// ตัวอย่างนี้ใช้ RxJS API (ต้องติดตั้ง: npm install rxjs)

/*
import { Observable, Subject, BehaviorSubject, ReplaySubject,
         of, from, interval, timer, fromEvent, 
         combineLatest, merge, zip, concat, forkJoin } from 'rxjs';

import { map, filter, tap, take, skip, scan, reduce,
         debounceTime, throttleTime, distinctUntilChanged,
         switchMap, mergeMap, concatMap, exhaustMap,
         catchError, retry, retryWhen, timeout,
         delay, bufferTime, auditTime,
         startWith, endWith, pairwise,
         withLatestFrom, share, shareReplay,
         takeUntil, takeWhile, skipWhile, skipUntil,
         first, last, find, findIndex,
         min, max, count, toArray } from 'rxjs/operators';
*/

// ========================
// RxJS เหมือนกับที่เราสร้าง แต่สมบูรณ์กว่า
// ========================

// จำลอง RxJS API สำหรับตัวอย่าง
const RxJS = {
  // Operators เพิ่มเติม
  
  startWith(value) {
    return (source) => new Observable2(observer => {
      observer.next(value);
      return source.subscribe(observer);
    });
  },
  
  endWith(value) {
    return (source) => new Observable2(observer => {
      return source.subscribe({
        next: v => observer.next(v),
        error: err => observer.error(err),
        complete: () => {
          observer.next(value);
          observer.complete();
        }
      });
    });
  },
  
  pairwise() {
    return (source) => new Observable2(observer => {
      let prev;
      let hasPrev = false;
      
      return source.subscribe({
        next: value => {
          if (hasPrev) {
            observer.next([prev, value]);
          }
          prev = value;
          hasPrev = true;
        },
        error: err => observer.error(err),
        complete: () => observer.complete()
      });
    });
  },
  
  first(predicate = null) {
    return (source) => new Observable2(observer => {
      const sub = source.subscribe({
        next: value => {
          if (!predicate || predicate(value)) {
            observer.next(value);
            observer.complete();
            sub.unsubscribe();
          }
        },
        error: err => observer.error(err),
        complete: () => observer.complete()
      });
      return () => sub.unsubscribe();
    });
  },
  
  last(predicate = null) {
    return (source) => new Observable2(observer => {
      let lastValue;
      let hasValue = false;
      
      return source.subscribe({
        next: value => {
          if (!predicate || predicate(value)) {
            lastValue = value;
            hasValue = true;
          }
        },
        error: err => observer.error(err),
        complete: () => {
          if (hasValue) observer.next(lastValue);
          observer.complete();
        }
      });
    });
  },
  
  toArray() {
    return (source) => new Observable2(observer => {
      const arr = [];
      return source.subscribe({
        next: value => arr.push(value),
        error: err => observer.error(err),
        complete: () => {
          observer.next(arr);
          observer.complete();
        }
      });
    });
  },
  
  count(predicate = null) {
    return (source) => new Observable2(observer => {
      let count = 0;
      return source.subscribe({
        next: value => {
          if (!predicate || predicate(value)) count++;
        },
        error: err => observer.error(err),
        complete: () => {
          observer.next(count);
          observer.complete();
        }
      });
    });
  },
  
  share() {
    return (source) => {
      const subject = new Subject();
      let refCount = 0;
      let subscription;
      
      return new Observable2(observer => {
        refCount++;
        const sub = subject.subscribe(observer);
        
        if (refCount === 1) {
          subscription = source.subscribe(subject);
        }
        
        return () => {
          refCount--;
          sub.unsubscribe();
          if (refCount === 0 && subscription) {
            subscription.unsubscribe();
          }
        };
      });
    };
  },
  
  shareReplay(bufferSize = 1) {
    return (source) => {
      const subject = new ReplaySubject(bufferSize);
      let subscription;
      
      return new Observable2(observer => {
        if (!subscription) {
          subscription = source.subscribe(subject);
        }
        return subject.subscribe(observer);
      });
    };
  }
};

// ========================
// ตัวอย่าง RxJS patterns
// ========================

// 1. pairwise - previous + current
from2([1, 2, 3, 4, 5])
  .pipe(RxJS.pairwise())
  .subscribe(([prev, curr]) => console.log(`${prev} → ${curr}`));

// 2. toArray
from2([1, 2, 3, 4, 5])
  .pipe(
    filter(x => x % 2 === 0),
    RxJS.toArray()
  )
  .subscribe(arr => console.log('Array:', arr));

// 3. startWith
from2(['B', 'C', 'D'])
  .pipe(RxJS.startWith('A'))
  .subscribe(v => console.log(v));

// 4. share - multicast
const expensive$ = new Observable2(observer => {
  console.log('Executing expensive operation...');
  observer.next(Math.random());
  observer.complete();
}).pipe(RxJS.share());

// สอง subscribers จะ share ผลลัพธ์เดียวกัน
expensive$.subscribe(v => console.log('Sub1:', v));
expensive$.subscribe(v => console.log('Sub2:', v));
```

---

## Step 1080: Reactive Programming กับ HTTP

```javascript
// ========================
// Reactive HTTP with Observable
// ========================

class HttpClient {
  static get(url, options = {}) {
    return new Observable2(observer => {
      const controller = new AbortController();
      
      fetch(url, { ...options, signal: controller.signal })
        .then(response => {
          if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
          }
          return response.json();
        })
        .then(data => {
          observer.next(data);
          observer.complete();
        })
        .catch(err => {
          if (err.name !== 'AbortError') {
            observer.error(err);
          }
        });
      
      return () => controller.abort(); // cleanup = cancel request
    });
  }
  
  static post(url, body, options = {}) {
    return HttpClient.get(url, {
      ...options,
      method: 'POST',
      headers: { 'Content-Type': 'application/json', ...options.headers },
      body: JSON.stringify(body)
    });
  }
}

// Search Component ที่ reactive
class SearchComponent {
  constructor() {
    this._searchSubject = new Subject();
    this._results$ = new BehaviorSubject([]);
    this._loading$ = new BehaviorSubject(false);
    this._error$ = new BehaviorSubject(null);
    
    this._setupSearch();
  }
  
  _setupSearch() {
    this._searchSubject
      .pipe(
        debounceTime(300),           // รอ 300ms
        distinctUntilChanged(),       // ไม่ส่งถ้าค่าเหมือนเดิม
        filter(query => query.length >= 2), // ต้องมีอย่างน้อย 2 ตัวอักษร
        tap(() => {
          this._loading$.next(true);
          this._error$.next(null);
        }),
        switchMap(query =>            // cancel previous request
          HttpClient.get(`https://api.example.com/search?q=${query}`).pipe(
            catchError(err => {
              this._error$.next(err.message);
              return of2([]);
            })
          )
        ),
        tap(() => this._loading$.next(false))
      )
      .subscribe(results => {
        this._results$.next(results);
      });
  }
  
  search(query) {
    this._searchSubject.next(query);
  }
  
  get results$() { return this._results$.asObservable ? this._results$.asObservable() : this._results$; }
  get loading$() { return this._loading$; }
  get error$() { return this._error$; }
  
  destroy() {
    this._searchSubject.complete();
  }
}

// ========================
// Reactive Form Validation
// ========================

class ReactiveForm {
  constructor() {
    this._fields = {};
    this._formValue$ = new BehaviorSubject({});
    this._errors$ = new BehaviorSubject({});
    this._valid$ = new BehaviorSubject(false);
  }
  
  addField(name, validators = []) {
    const field$ = new BehaviorSubject('');
    this._fields[name] = { value$: field$, validators };
    
    // Validate on change
    field$.pipe(
      debounceTime(300),
      map(value => {
        const errors = validators
          .map(v => v(value))
          .filter(Boolean);
        return errors;
      })
    ).subscribe(errors => {
      const currentErrors = this._errors$.getValue();
      this._errors$.next({ ...currentErrors, [name]: errors });
      this._updateValidity();
    });
    
    // Update form value
    field$.subscribe(value => {
      const currentValue = this._formValue$.getValue();
      this._formValue$.next({ ...currentValue, [name]: value });
    });
    
    return this;
  }
  
  setValue(name, value) {
    if (this._fields[name]) {
      this._fields[name].value$.next(value);
    }
    return this;
  }
  
  _updateValidity() {
    const errors = this._errors$.getValue();
    const hasErrors = Object.values(errors).some(errs => errs.length > 0);
    this._valid$.next(!hasErrors);
  }
  
  get value$() { return this._formValue$; }
  get errors$() { return this._errors$; }
  get valid$() { return this._valid$; }
}

// Validators
const required = (value) => 
  !value || value.toString().trim() === '' ? 'Required' : null;

const minLength = (min) => (value) =>
  value && value.length < min ? `Minimum ${min} characters` : null;

const pattern = (regex, message) => (value) =>
  value && !regex.test(value) ? message : null;

const emailValidator = pattern(/^[^\s@]+@[^\s@]+\.[^\s@]+$/, 'Invalid email');

// ใช้งาน
const registrationForm = new ReactiveForm();
registrationForm
  .addField('name', [required, minLength(2)])
  .addField('email', [required, emailValidator])
  .addField('password', [required, minLength(8)]);

// Subscribe to changes
registrationForm.valid$.subscribe(valid => {
  console.log('Form valid:', valid);
});

registrationForm.errors$.subscribe(errors => {
  Object.entries(errors).forEach(([field, errs]) => {
    if (errs.length > 0) {
      console.log(`${field} errors:`, errs);
    }
  });
});

// จำลองการกรอกฟอร์ม
registrationForm.setValue('name', 'สมชาย');
registrationForm.setValue('email', 'somchai@example.com');
registrationForm.setValue('password', 'securepassword123');
```

---

## Step 1081: Real-time Data ด้วย Reactive

```javascript
// ========================
// WebSocket Observable
// ========================

class WebSocketObservable {
  static create(url) {
    return new Observable2(observer => {
      let ws = null;
      
      try {
        ws = new WebSocket(url);
        
        ws.onopen = () => {
          observer.next({ type: 'open' });
        };
        
        ws.onmessage = (event) => {
          try {
            const data = JSON.parse(event.data);
            observer.next({ type: 'message', data });
          } catch {
            observer.next({ type: 'message', data: event.data });
          }
        };
        
        ws.onerror = (error) => {
          observer.error(error);
        };
        
        ws.onclose = (event) => {
          if (event.code === 1000) {
            observer.complete();
          } else {
            observer.error(new Error(`WebSocket closed: ${event.code}`));
          }
        };
      } catch (err) {
        observer.error(err);
      }
      
      // Cleanup: close WebSocket
      return () => {
        if (ws && ws.readyState === WebSocket.OPEN) {
          ws.close(1000, 'Unsubscribed');
        }
      };
    });
  }
}

// Stock Price Stream (Simulated)
class StockPriceSimulator {
  static createStream(symbol, initialPrice) {
    return new Observable2(observer => {
      let price = initialPrice;
      let tick = 0;
      
      const id = setInterval(() => {
        const change = (Math.random() - 0.5) * 2;
        price = Math.max(0.01, price + change);
        
        observer.next({
          symbol,
          price: parseFloat(price.toFixed(2)),
          change: parseFloat(change.toFixed(2)),
          changePercent: parseFloat((change / price * 100).toFixed(2)),
          timestamp: new Date().toISOString(),
          tick: ++tick
        });
      }, 500);
      
      return () => clearInterval(id);
    });
  }
}

// Portfolio Dashboard
class PortfolioDashboard {
  constructor(holdings) {
    this._holdings = holdings;
    this._streams = new Map();
    this._portfolio$ = new BehaviorSubject({});
  }
  
  start() {
    // สร้าง stream สำหรับแต่ละ stock
    const stockStreams = this._holdings.map(({ symbol, shares, buyPrice }) => {
      const stream$ = StockPriceSimulator.createStream(symbol, buyPrice).pipe(
        map(tick => ({
          symbol: tick.symbol,
          shares,
          currentPrice: tick.price,
          value: tick.price * shares,
          gainLoss: (tick.price - buyPrice) * shares,
          gainLossPercent: ((tick.price - buyPrice) / buyPrice * 100).toFixed(2)
        }))
      );
      
      return stream$;
    });
    
    // รวมทุก stock streams
    merge(...stockStreams).subscribe(stock => {
      const current = this._portfolio$.getValue();
      this._portfolio$.next({
        ...current,
        [stock.symbol]: stock
      });
    });
    
    // Log portfolio value
    this._portfolio$.pipe(
      filter(p => Object.keys(p).length === this._holdings.length),
      map(p => {
        const stocks = Object.values(p);
        return {
          totalValue: stocks.reduce((sum, s) => sum + s.value, 0).toFixed(2),
          totalGainLoss: stocks.reduce((sum, s) => sum + s.gainLoss, 0).toFixed(2),
          stocks
        };
      }),
      RxJS.pairwise(),
      filter(([prev, curr]) => prev.totalValue !== curr.totalValue)
    ).subscribe(([, summary]) => {
      console.log(`Portfolio: ฿${summary.totalValue} (${parseFloat(summary.totalGainLoss) >= 0 ? '+' : ''}฿${summary.totalGainLoss})`);
    });
  }
}

// จำลอง
const dashboard = new PortfolioDashboard([
  { symbol: 'KBANK', shares: 100, buyPrice: 150.00 },
  { symbol: 'PTT', shares: 200, buyPrice: 45.00 },
  { symbol: 'SCB', shares: 150, buyPrice: 120.00 }
]);

// ถ้าต้องการทดสอบ: dashboard.start();
console.log('Portfolio dashboard ready');

// ========================
// Server-Sent Events Observable
// ========================

class SSEObservable {
  static create(url) {
    return new Observable2(observer => {
      const eventSource = new EventSource(url);
      
      eventSource.onmessage = (event) => {
        try {
          observer.next(JSON.parse(event.data));
        } catch {
          observer.next(event.data);
        }
      };
      
      eventSource.onerror = (err) => {
        observer.error(err);
      };
      
      return () => eventSource.close();
    });
  }
}

// ========================
// Polling Observable
// ========================

function createPolling(fetchFn, intervalMs, options = {}) {
  const { immediate = true, retries = 3 } = options;
  
  return new Observable2(observer => {
    let subscription;
    let isActive = true;
    
    async function poll() {
      while (isActive) {
        try {
          const data = await fetchFn();
          if (isActive) observer.next(data);
        } catch (err) {
          if (isActive) observer.error(err);
          break;
        }
        
        if (isActive) {
          await new Promise(resolve => setTimeout(resolve, intervalMs));
        }
      }
    }
    
    if (immediate) {
      poll();
    } else {
      setTimeout(poll, intervalMs);
    }
    
    return () => { isActive = false; };
  }).pipe(
    distinctUntilChanged((a, b) => JSON.stringify(a) === JSON.stringify(b))
  );
}

// จำลอง polling API
let apiData = { count: 0, status: 'active' };
const polling$ = createPolling(
  () => {
    apiData.count++;
    return { ...apiData };
  },
  500
);

const sub = polling$.pipe(take(5)).subscribe({
  next: data => console.log('Poll result:', data),
  complete: () => console.log('Polling stopped')
});
```

---

## Step 1082-1090: Exercises และสรุป

```javascript
// ========================
// Observable Patterns Summary
// ========================

// 1. Search with debounce
function createSearchStream(input$) {
  return input$.pipe(
    debounceTime(300),
    distinctUntilChanged(),
    filter(q => q.length >= 2),
    switchMap(query => HttpClient.get(`/api/search?q=${query}`).pipe(
      catchError(() => of2([]))
    ))
  );
}

// 2. Infinite scroll
function createInfiniteScroll(scroll$, fetchPage) {
  const page$ = new BehaviorSubject(1);
  
  return scroll$.pipe(
    filter(({ scrollTop, clientHeight, scrollHeight }) =>
      scrollTop + clientHeight >= scrollHeight - 100
    ),
    throttleTime(300),
    withLatestFrom(page$),
    map(([, page]) => page),
    concatMap(page => fetchPage(page).pipe(
      tap(() => page$.next(page + 1))
    ))
  );
}

// 3. Auto-save with debounce
function createAutoSave(content$, saveFunction) {
  return content$.pipe(
    debounceTime(1000),
    distinctUntilChanged(),
    switchMap(content => 
      Observable2.fromPromise(saveFunction(content)).pipe(
        map(() => ({ saved: true, content })),
        catchError(err => of2({ saved: false, error: err.message }))
      )
    )
  );
}

// 4. Real-time collaboration
function createCollaboration(userId, documentId) {
  const localChanges$ = new Subject();
  const remoteChanges$ = WebSocketObservable.create(`/ws/docs/${documentId}`);
  
  return merge(
    localChanges$.pipe(map(change => ({ ...change, source: 'local' }))),
    remoteChanges$.pipe(
      filter(msg => msg.type === 'message' && msg.data.userId !== userId),
      map(msg => ({ ...msg.data, source: 'remote' }))
    )
  );
}

// 5. Retry with exponential backoff
function withExponentialBackoff(maxRetries = 3, baseDelay = 1000) {
  return retryWhen(err => {
    let attempts = 0;
    return new Observable2(observer => {
      if (attempts >= maxRetries) {
        observer.error(err);
        return;
      }
      const delayMs = baseDelay * Math.pow(2, attempts);
      console.log(`Retry in ${delayMs}ms (attempt ${++attempts})`);
      const id = setTimeout(() => {
        observer.next();
        observer.complete();
      }, delayMs);
      return () => clearTimeout(id);
    });
  });
}
```

---

## แบบฝึกหัด (Exercises)

**Exercise 1:** Observable Basic
```javascript
// สร้าง Observable ที่:
// 1. Emit ค่า fibonacci แต่ละตัวทุก 200ms
// 2. หยุดหลังจาก 10 ค่า
// 3. map แต่ละค่า → 'Fibonacci #n: value'
// 4. สามารถ unsubscribe ได้

function createFibonacciStream() {
  // TODO: return Observable
}

const fib$ = createFibonacciStream();
const sub = fib$.subscribe(console.log);
setTimeout(() => sub.unsubscribe(), 3000);
```

**Exercise 2:** Combination Operators
```javascript
// สร้าง Weather Dashboard ที่:
// - ดึงข้อมูลอุณหภูมิและความชื้นพร้อมกัน (combineLatest)
// - แสดง "Loading..." ขณะรอ
// - ถ้า request ใดสำเร็จ แสดงผลทันที
// - retry 3 ครั้งถ้า error

function getTemperature(city) { /* returns Observable<number> */ }
function getHumidity(city) { /* returns Observable<number> */ }

function getWeatherDashboard(city) {
  return combineLatest([
    // TODO: combine temperature and humidity
  ]).pipe(
    // TODO: map to { temperature, humidity, feelsLike }
    // TODO: retry on error
  );
}
```

**Exercise 3:** Subject สำหรับ State Management
```javascript
// สร้าง Shopping Cart ด้วย Reactive State:
// - BehaviorSubject เก็บ cart state
// - Methods: addItem, removeItem, updateQuantity, clearCart
// - Observables: items$, total$, itemCount$, isEmpty$
// - Auto-save to localStorage เมื่อ cart เปลี่ยน

class ReactiveCart {
  constructor() {
    // TODO: initialize with BehaviorSubject
  }
  
  addItem(product) { /* TODO */ }
  removeItem(productId) { /* TODO */ }
  updateQuantity(productId, qty) { /* TODO */ }
  clearCart() { /* TODO */ }
  
  get items$() { /* TODO */ }
  get total$() { /* TODO */ }
  get itemCount$() { /* TODO */ }
  get isEmpty$() { /* TODO */ }
}
```

**Exercise 4:** WebSocket Chat
```javascript
// สร้าง Chat Observable ที่:
// - เชื่อมต่อ WebSocket
// - Auto-reconnect เมื่อ disconnect (retry with delay)
// - Filter messages by room
// - Track online users (scan)
// - Notify เมื่อมี user เข้า/ออก

class ChatService {
  constructor(wsUrl) {
    this._ws$ = WebSocketObservable.create(wsUrl);
    this._setupStreams();
  }
  
  _setupStreams() {
    // TODO: setup message$, onlineUsers$, notifications$
  }
  
  joinRoom(roomId) { /* TODO */ }
  leaveRoom(roomId) { /* TODO */ }
  sendMessage(roomId, message) { /* TODO */ }
  
  get messages$() { /* TODO */ }
  get onlineUsers$() { /* TODO */ }
}
```

---

## สรุป Reactive Programming

| แนวคิด | คำอธิบาย | ตัวอย่าง |
|--------|---------|---------|
| Observable | Stream of values over time | click events, API calls |
| Observer | ผู้รับ values | { next, error, complete } |
| Subscription | การ subscribe ที่ cancel ได้ | sub.unsubscribe() |
| Subject | Observable + Observer | EventBus, Store |
| BehaviorSubject | มี current value | App state |
| Operators | transform streams | map, filter, debounce |

```javascript
// RxJS Cheat Sheet

// Create
of(1, 2, 3)              // sync values
from([1, 2, 3])          // from iterable
interval(1000)           // every 1 second
timer(1000, 500)         // delay then interval
fromEvent(btn, 'click')  // DOM events
fromPromise(fetch(...))  // from Promise

// Transform
map(x => x * 2)          // transform each value
filter(x => x > 0)       // keep matching
scan((acc, x) => acc+x)  // running accumulation
switchMap(query => ...)   // cancel prev, start new
mergeMap(item => ...)     // parallel
concatMap(item => ...)    // sequential

// Combine
merge(a$, b$)            // concurrent streams
concat(a$, b$)           // sequential
combineLatest([a$, b$])  // latest from all
zip([a$, b$])            // pair by index

// Filter
take(5)                  // first 5
skip(3)                  // skip first 3
debounceTime(300)        // wait for pause
throttleTime(500)        // limit rate
distinctUntilChanged()   // skip duplicates

// Error Handling
catchError(err => ...)   // recover from error
retry(3)                 // retry N times
```

---

**ยินดีด้วย!** คุณได้เรียนรู้:
- Reactive Programming paradigm
- การสร้าง Observable จากพื้นฐาน
- Operators: map, filter, switchMap, mergeMap, combineLatest
- Hot vs Cold Observables
- Subject, BehaviorSubject, ReplaySubject
- Error handling ใน Reactive programming
- Real-time applications ด้วย Reactive

**ขั้นตอนต่อไป:** Part 56 จะพาไปเรียน **Testing กับ JavaScript** ครอบคลุม Unit Tests, Integration Tests และ E2E Testing
