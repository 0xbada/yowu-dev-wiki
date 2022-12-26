---
title: 이벤트 루프 (Event Loop)
description: 
published: true
date: 2022-12-26T14:12:18.189Z
tags: nodejs, v8
editor: markdown
dateCreated: 2022-12-26T12:35:42.101Z
---

- [Event Loop***English** version of this document is available*](/en/dev/Nodejs/event-loop)
{.links-list}

## 개요

Node.js 이벤트 루프는 Node.js가 비차단(Non-Blocking) I/O 작업을 효율적으로 수행할 수 있도록 하는 런타임의 기본 부분입니다. 이벤트 대기열을 지속적으로 모니터링하고 처리할 준비가 된 이벤트를 실행하는 방식으로 작동합니다.

- 이벤트 루프는 싱글 스레드 이벤트 루프입니다. 즉, 한 번에 하나의 이벤트만 처리할 수 있으며 이벤트는 순서대로 처리됩니다. 그러나 Node.js는 CPU를 많이 사용하는 작업을 수행하거나 시스템 호출을 차단하는 것과 같은 특정 유형의 작업에 여러 스레드를 사용할 수도 있습니다.
- Node.js 애플리케이션에서 실행되는 대부분의 JavaScript 코드 실행을 이벤트 루프가 담당합니다. 여기에는 사용자 코드와 Node.js 런타임 및 해당 표준 라이브러리에서 제공하는 코드가 모두 포함됩니다.
- 또한, 비차단형으로 설계되어 각 이벤트가 완료될 때까지 기다리지 않고 다음 이벤트로 넘어갈 때까지 이벤트를 동시에 처리할 수 있습니다. 이는 비차단 I/O 작업 및 비동기 콜백 함수를 사용하여 달성됩니다.
- JavaScript와 네이티브 코드의 조합을 사용하여 이벤트 루프가 구현됩니다. 이벤트 루프의 JavaScript 부분은 C++로 작성되어 V8 JavaScript 엔진을 사용, 네이티브 코드로 컴파일되는 반면 네이티브 부분은 플랫폼별 API를 사용하여 구현됩니다.
- 이벤트 루프에는 여러 단계가 있으며 각 단계는 특정 유형의 이벤트 처리를 담당합니다. 단계가 실행되는 순서는 사용 중인 Node.js의 특정 버전에 따라 다를 수 있습니다.

> ### Non-Blocking I/O 작업에 대하여
>
> Node.js에서 비차단 I/O 작업은 작업이 완료되기를 기다리는 동안 프로그램 실행을 차단하지 않는 작업입니다. 이는 I/O 작업이 백그라운드에서 수행되는 동안 프로그램이 계속 실행되고 다른 작업을 수행할 수 있음을 의미합니다.
>
> Node.js는 서버 측에서 JavaScript를 실행하도록 설계된 Chrome V8 JavaScript 엔진 위에 Built 됩니다. Node.js의 주요 기능 중 하나는 비차단 I/O를 사용하여 많은 동시 연결을 효율적으로 처리하고 작업을 비동기식(asynchronously)으로 수행할 수 있다는 것입니다.

> ### V8 JavaScript 엔진에 대하여
>
> 위에서 언급한 것과 같이 Node.js는 Google에서 개발하는 V8 JavaScript 엔진 위에 Built 됩니다. V8은 JavaScript 코드를 빠르고 효율적으로 실행하도록 설계된 고성능 오픈 소스 JavaScript 엔진입니다. C++로 작성되었으며 Google Chrome 웹 브라우저 및 기타 여러 프로젝트에서 사용됩니다.
>
> V8은 ECMAScript 버전 6 이상으로 작성된 코드를 포함하여 최신 JavaScript 코드를 실행하도록 설계되었습니다. 여기에는 Class, 화살표 함수(Arrow Function) 및 Promise 같은 기능과 JIT(Just-In-Time) 컴파일이 포함되어 런타임에 코드를 네이티브 머신 코드로 변환하여 코드를 더 빠르게 실행할 수 있습니다.
>
> V8은 Node.js 애플리케이션에서 실행되는 JavaScript 코드 실행을 담당하므로 Node.js 런타임의 중요한 구성 요소입니다. 이는 Node.js 런타임과 긴밀하게 통합되며 Node.js를 확장 가능한 네트워크 애플리케이션 구축에 적합하게 만드는 많은 기능을 제공합니다.
>
> 다음은 V8 엔진의 작동 방식을 간략하게 나타낸 것입니다.
>
> 1. V8 엔진은 JavaScript 코드를 입력받습니다.
> 1. 코드가 파싱되고 V8 엔진이 이해하기 쉬운 중간 표현(IR)으로 변환됩니다.
> 1. IR은 Constant Folding, Inlining 등 다양한 최적화 기법을 적용하여 빠른 실행이 가능하도록 최적화합니다.
> 1. 최적화된 IR은 JIT 컴파일러를 사용하여 네이티브 코드로 컴파일됩니다.
> 1. 네이티브 코드는 CPU에 의해 실행됩니다.

이벤트 루프에는 다음과 같은 여러 단계가 있습니다.

![nodejs-eventloop.png](/nodejs-eventloop.png =700x){.align-center}

- **타이머 (Timers)**: `setTimeout()` 및 `setInterval()`에 의해 예약된 콜백 함수를 실행합니다.
- **보류 중인 콜백 (Pending callbacks)**: Blocking 작업을 완료한 I/O 콜백을 실행합니다.
- **유휴 및 준비 (Idle, prepare)**: 이벤트 루프에서 사용하는 내부 단계입니다.
- **선출 (Poll)**: 새로운 I/O 이벤트를 검색하고 콜백을 실행합니다. 처리할 I/O 이벤트가 없는 경우 이벤트 루프는 이 단계에서 Blocking되고 새 이벤트가 도착할 때까지 기다립니다.
- **확인 (Check)**: `setImmediate()`에 의해 예약된 콜백을 실행합니다.
- **콜백 닫기 (Close callbacks)**: `socket.on('close', ...)`에 의해 예약된 것과 같은 콜백 닫기를 실행합니다.

> **요우의 노트 😎**
> 
> 이 곳에서 표현한 각 단계의 한글명은 공식적인 한글명이 없어 제가 임의로 붙인 것 입니다. 그러므로 여러분들은 영어 원문에 익숙해지는 것이 낫습니다.

이벤트 루프의 각 반복에서 이벤트 루프는 위에서 설명한 단계를 지정된 순서대로 처리합니다. 이벤트가 처리되면 연결된 콜백 함수가 실행되고 이벤트가 대기열에서 제거됩니다.

이벤트 루프는 다음 작업으로 이동하기 전에 각 작업이 완료될 때까지 차단(Blocking)하고 기다리는 대신 런타임이 많은 작업을 동시에 수행할 수 있도록 하기 때문에 Node.js에서 중요한 개념입니다. 이를 통해 Node.js로 확장 가능한 고성능 애플리케이션을 구축할 수 있습니다.

## 각 단계에 대한 자세한 설명

### 타이머 (Timers)

`setTimeout()` 및 `setInterval()`을 사용하여 예약된 콜백 함수를 실행합니다. 타이머가 예약되면 타이머 대기열에 추가되고 지정된 시간이 경과하면 이벤트 루프가 콜백을 실행합니다. 여러 타이머가 동시에 만료되도록 예약된 경우 콜백은 대기열에 추가된 순서대로 실행됩니다.

> **Q. 이벤트 루프의 타이머가 정확한 실행 시간을 보장합니까?**
>
> 아니요, Node.js에서 이벤트 루프의 `tick`은 `setTimeout()` 및 `setInterval()` 와 같은 타이머 함수의 정확한 실행 시간을 보장하지 않습니다.
>
> Node.js 이벤트 루프의 `tick`은 이벤트 루프의 두 연속 반복 사이에 가능한 가장 작은 지연을 나타내는 시간 단위입니다. 일반적으로 밀리초 또는 초와 같이 타이머 기능에서 사용하는 시간 단위보다 훨씬 짧습니다.
>
> 그러나 이벤트 루프의 `tick`을 사용하여 예약된 콜백 함수의 실제 실행 시간은 시스템 부하 및 이벤트 큐에 다른 작업이 존재하는 등의 요인으로 인해 달라질 수 있습니다. 따라서 이벤트 루프의 `tick`을 사용하여 예약된 작업에 대한 정확한 실행 시간을 보장할 수 없습니다.
>
> 일반적으로 이로 인한 타이밍 오류는 대부분의 응용 프로그램에서 작은 영향을 끼치며, 눈에 보이지 않습니다. 그러나 특정 시간, 또는 특정 수준의 정확도로 작업을 실행해야 하는 경우 이벤트 루프의 타이머 기능과 이벤트 루프의 `tick` 기능이 사용자의 요구에 적합하지 않을 수 있음을 인식하는 것이 중요합니다.
>
> 이러한 경우 하드웨어 타이머를 사용하거나 자체 타이밍 메커니즘을 구현하는 것과 같은 다른 접근 방식을 사용해야 할 수 있습니다. 애플리케이션의 요구 사항과 필요한 정확도 수준을 신중하게 고려하여 적절한 접근 방식을 선택하는 것도 중요합니다.

### 보류 중인 콜백 (Pending callbacks)

파일 읽기 또는 HTTP 요청 생성와 같은 차단(Blocking) 작업을 완료한 콜백을 실행합니다. 이러한 콜백은 차단 작업이 시작될 때 Node.js 런타임에 의해 "보류 중인 콜백" 대기열에 추가되며 대기열에 추가된 순서대로 실행됩니다.

### 유휴 및 준비 (Idle, prepare)

파일 시스템 이벤트에 대한 Watcher 리스트 업데이트 및 내부 데이터 구조 정리와 같은 다양한 작업을 수행하는 데 사용되는 이벤트 루프의 내부 단계입니다.

### 선출 (Poll)

새로운 I/O 이벤트를 검색하고 콜백을 실행합니다. 이벤트 루프는 처리할 이벤트가 없으면 이 단계에서 차단(Blocking)되고 새 이벤트가 도착할 때까지 기다립니다. 이 단계에서 이벤트 루프가 차단되는 시간은 구성 가능한 값인 `poll` Phase 타임 아웃에 의해 결정됩니다. 이벤트 루프가 차단 해제되면 도착한 새 이벤트에 대한 콜백을 실행한 다음 폴링 단계로 돌아가 더 많은 이벤트를 확인합니다.

### 확인 (Check)

`setImmediate()`를 사용하여 예약된 콜백을 실행합니다. 이러한 콜백은 "확인 (Check)" 대기열에 추가되고 대기열에 추가된 순서대로 실행됩니다.

### 콜백 닫기 (Close callbacks)

`socket.on('close', ...)`을 사용하여 예약된 콜백과 같은 닫기 콜백을 실행합니다. 이러한 콜백은 소켓이나 다른 리소스가 닫힐 때 실행되며 일반적으로 리소스를 정리하거나 리소스 닫기와 관련된 다른 작업을 수행하는 데 사용됩니다.

## 이벤트 루프 예제

```js
setTimeout(function() {
  console.log('Timeout callback');
}, 1000);

console.log('Start');

// "Start" 먼저 출력된 후 1초 뒤 "Timeout callback" 출력
```

이 예제에서는 `setTimeout()`을 사용하여 1초(1000밀리초) 후에 실행되도록 콜백 함수를 예약합니다. 이벤트 루프는 1초가 경과한 후 "타이머" 단계에 들어가고 콜백 함수를 실행합니다.

---


```js
const fs = require('fs');

fs.readFile('file.txt', function(err, data) {
  console.log('File read callback');
});

console.log('Start');
// "Start" 먼저 출력된 후 파일 읽기가 완료되면 "File read callback" 출력
```

이 예제에서는 fs 모듈을 사용하여 파일을 읽어 Blocking I/O 작업을 시작합니다. 파일 읽기 작업이 완료되면 콜백 함수가 "보류 중인 콜백" 대기열에 추가되고 다음 반복에서 이벤트 루프에 의해 실행됩니다.

---

```js
setImmediate(function() {
  console.log('Immediate callback');
});

console.log('Start');

// "Start" 먼저 출력된 후 바로 "Imediate callback" 출력
```

이 예제에서는 가능한 한 빨리 실행되도록 `setImmediate()`를 사용하여 콜백 함수를 예약합니다. 이벤트 루프는 다음 반복에서 "확인" 단계에 진입하고 콜백 함수를 실행합니다.

## Deep dive with pseudo code

```js
const fs = require('fs');

// 보류 중인 콜백
const pendingCallbacks = [];

// 타이머
const timers = [];

// SetImmediate 콜백
const immediateCallbacks = [];

// 이벤트 핸들러에 대한 파일 디스크립터의 맵
const fdHandlers = new Map();

// 처리할 파일 디스크립터 이벤트 큐
const fdEvents = [];

// 보류 중인 콜백 큐에 콜백을 추가
function addPendingCallback(callback) {
  pendingCallbacks.push(callback);
}

// 타이머 큐에 콜백을 추가
function addTimer(callback, timeout) {
  const expiration = Date.now() + timeout;
  timers.push({ callback, expiration });
  timers.sort((a, b) => a.expiration - b.expiration);
}
// SetImmediate 큐에 콜백을 추가
// Add a SetImmediate callback to the queue
function addImmediateCallback(callback) {
  immediateCallbacks.push(callback);
}

// 파일 디스크립터에 대한 이벤트 핸들러 추가
function addFdHandler(fd, handler) {
  fdHandlers.set(fd, handler);
}

// 큐에 파일 디스크립터 이벤트 추가
function addFdEvent(fd, eventType) {
  fdEvents.push({ fd, eventType });
}

// 메인 이벤트 루프
function eventLoop() {
  // 보류 중인 콜백을 실행
  while (pendingCallbacks.length > 0) {
    const callback = pendingCallbacks.shift();
    callback();
  }

  // 만료된 타이머 실행
  const now = Date.now();
  while (timers.length > 0 && timers[0].expiration <= now) {
    const timer = timers.shift();
    timer.callback();
  }

  // 모든 SetImmediate 콜백 실행
  while (immediateCallbacks.length > 0) {
    const callback = immediateCallbacks.shift();
    callback();
  }

  // 모든 파일 디스크립터 이벤트 처리
  while (fdEvents.length > 0) {
    const { fd, eventType } = fdEvents.shift();
    const handler = fdHandlers.get(fd);
    if (handler) {
      handler(fd, eventType);
    }
  }

  // 새로운 이벤트 도착 대기
  waitForEvents();
}

// 새로운 이벤트 도착 대기
function waitForEvents() {
   // epoll() 또는 select()와 같은 Blocking System call 사용하여 새로운 이벤트를 기다립니다.
   // 이벤트를 사용할 수 있을 때 적절한 큐에 추가합니다(보류 중인 콜백, 타이머 등).
   // 그리고 process.nextTick()을 사용하여 이벤트 루프가 다시 실행되도록 예약합니다.
}

// 이벤트 루프 실행
function run() {
  process.nextTick(eventLoop);
}

run();

// 사용 예제
fs.readFile('file.txt', function(err, data) {
  addPendingCallback(function() {
    console.log('File read callback');
  });
});

addTimer(function() {
  console.log('Timeout callback');
}, 1000);

setImmediate(function() {
  console.log('Immediate callback');
});
```

이 코드는 다양한 유형의 이벤트를 저장하기 위해 여러 배열과 맵을 정의합니다.

- `pendingCallbacks`: Blocking 작업을 완료하고 실행할 준비가 된 콜백의 배열입니다.
- `timers`: `setTimeout()` 또는 `setInterval()`을 사용하여 예약된 타이머 배열입니다.
- `immediateCallbacks`: `setImmediate()`를 사용하여 예약되고 실행할 준비가 된 콜백의 배열입니다.
- `fdHandlers`: 이벤트 핸들러에 대한 파일 디스크립터의 맵입니다.
- `fdEvents`: 처리할 준비가 된 파일 디스크립터 이벤트의 배열입니다.

이 코드는 또한 배열 및 맵에 이벤트를 추가하는 여러 함수를 정의합니다.

- `addPendingCallback(callback)`: `pendingCallbacks` 배열에 콜백을 추가합니다.
- `addTimer(callback, timeout)`: `timers` 배열에 타이머 콜백을 추가합니다.
- `addImmediateCallback(callback)`: `immediateCallbacks` 배열에 콜백을 추가합니다.
- `addFdHandler(fd, handler)`: 파일 디스크립터에 대한 이벤트 핸들러를 `fdHandlers` 맵에 추가합니다.
- `addFdEvent(fd, eventType)`: 파일 디스크립터 이벤트를 `fdEvents` 배열에 추가합니다.

메인 이벤트 루프는 `eventLoop()` 함수에서 정의됩니다. 이 함수는 다음 순서로 이벤트를 처리합니다.

- **Pending callbacks**: `pendingCallbacks` 배열의 콜백은 추가된 순서대로 실행됩니다.
- **Timer**: `timers` 배열의 타이머 콜백은 추가된 순서대로 실행됩니다.
- **SetImmediate callbacks**: `immediateCallbacks` 배열의 콜백은 추가된 순서대로 실행됩니다.
- **File descriptor events**: `fdEvents` 배열의 이벤트는 `fdHandlers` 맵에서 해당 이벤트 핸들러를 실행하여 처리됩니다.

마지막으로 `waitForEvents()` 함수가 호출되어 새 이벤트가 도착하기를 기다립니다. 이 함수는 `setTimeout()`을 사용하거나 Linux에서 `epoll()`과 같은 차단 시스템 호출을 사용하는 등의 다양한 기술을 사용하여 구현할 수 있습니다.

> ### Linux의 epoll() 함수에 대하여
>
> `epoll`은 확장 가능한 I/O 이벤트 알림 메커니즘을 제공하는 Linux 커널 시스템 콜입니다. 데이터 가용성 또는 연결 준비와 같은 이벤트에 대한 파일 디스크립터 집합을 모니터링하는 데 사용되며 네트워크 응용 프로그램에서 효율적인 I/O 다중화를 구현하는 데 사용할 수 있습니다.
>
> `epoll`은 에지 트리거(edge-triggered) 또는 레벨 트리거(level-triggerd)의 두 가지 모드 중 하나로 작동합니다. 에지 트리거 모드에서는 파일 디스크립터의 상태가 변경될 때만 이벤트가 생성됩니다. 레벨 트리거 모드에서는 데이터를 읽을 수 있는 경우와 같이 파일 디스크립터가 특정 상태에 있을 때마다 이벤트가 생성됩니다.
>
> `epoll`은 종종 `select()` 시스템 호출과 함께 사용되어 프로그램이 이벤트에 대한 여러 파일 디스크립터를 모니터링할 수 있도록 합니다. `epoll`은 많은 수의 파일 디스크립터를 모니터링하는 보다 효율적인 방법을 제공하며 특히 높은 동시성 서버에 유용합니다.
>
> Node.js에서 사용하는 크로스 플랫폼 라이브러리인 `libuv`는 `epoll` 및 기타 플랫폼별 API에 대한 래퍼를 제공하여 비동기식 I/O 작업을 위한 일관된 인터페이스를 제공합니다. 이를 통해 Node.js 애플리케이션은 `epoll` 시스템 호출을 직접 사용하지 않고도 Linux에서 `epoll`의 확장성과 효율성을 활용할 수 있습니다.
>
> MacOS/BSD에는 `epoll` 과 동일한 역할을 하는 `kqueue()`가 있고, Windows 에는 `IOCP`(Input/Output Completion Ports)가 있습니다.

> ### libuv에 대하여
>
> libuv는 비동기 I/O 및 기타 유틸리티 기능을 지원하는 크로스 플랫폼 라이브러리입니다. Node.js의 기본 플랫폼 계층으로 사용되며 Node.js 런타임에서 이벤트 루프 및 기타 비동기 메커니즘 구현을 담당합니다.
>
> libuv는 C언어로 작성되었으며 파일 I/O, 네트워크 작업, 타이머 및 프로세스 관리와 같은 다양한 유형의 비동기 작업을 위한 일관된 API를 제공합니다. Linux의 `epoll` 및 macOS의 `kqueue`와 같은 다양한 플랫폼별 API 간의 차이점을 추상화하여 여러 운영 체제에서 작동하는 교차 플랫폼 코드를 더 쉽게 작성할 수 있습니다.
>
> libuv는 Node.js 런타임의 중요한 구성 요소이며 Node.js가 확장 가능한 네트워크 애플리케이션을 구축하는 데 적합하도록 만드는 많은 기능을 제공합니다.

## More

- [The Node.js Event Loop, Timers, and process.nextTick()*nodejs.org*](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
{.links-list}