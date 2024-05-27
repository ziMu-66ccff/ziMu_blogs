---
title: 前端埋点sdk搭建实践总结
link: sdkSumary
catalog: true
date: 2024-03-31 14:23:47
tags:
  - 前端
  - 前端监控
  - sendSeacon
categories:
  - [笔记, 前端, 前端监控]
---

# 前端埋点 sdk 搭建实践总结

## 前端搭建监控体系是为了什么？

### 及时发现问题

1. 能够通过收集的性能指标，及时发现存在的性能问题
2. 能够通过白屏监控，及时发现出现了白屏问题
3. 能够通过通过 pv，uv 的变化，及时发现活跃量的变化；通过用户行为的监控，收集到用户的行为信息，从而进行针对性推荐和优化
4. 能够异常上报，及时发现各种异常、
5. 能够通过接口调用情况监控，及时发现流量大的后端服务，高频出错的后端服务

### 精确定位问题

1. 通过收集的页面加载事件的关键时间段，精准的定位到首屏时间主要是卡在哪个时间段
2. 通过用户行为，路由跳转的监控，及时定位到发生错误前后用户的操作，助力错误重现，修复
3. 通过异常监控，准确定位到发生错误的原因，所在文件，行，列，
4. 通过接口调用情况监控，及时定位出错的后端服务

总结起来就简单的两句话`如何及时的发现问题`, `如何准确的定位问题`

## 前端监控需要监控哪些模块

### 性能监控：

#### 性能监控究竟要监控什么数据，从哪里入手？

##### 用户体验为王，以用户为中心的性能指标：

###### `加载速度` 用户什么时候可以感受到界面开始加载，加载完成：

1. FP：首次非网页背景像素渲染的时间

   - 建议改为收集 FCP

   - **采集方法**：

   - 1. 基于 PerformanceObserver

        ```typescript
        new PerformanceObserver((entryList) => {
          for (const entry of entryList.getEntriesByName('first-paint')) {
            console.log('fp', entry);
          }
        }).observe({ type: 'paint', buffered: true });
        ```

2. FCP：首次内容渲染的时间

   - 收集的原因：用户首次感知到界面开始有东西

   - **采集方法**：

   - 1. 基于 PerformanceObserver

     ```typescript
     new PerformanceObserver((entryList) => {
       for (const entry of entryList.getEntriesByName('first-contentful-paint')) {
         console.log('fcp', entry);
       }
     }).observe({ type: 'paint', buffered: true });
     ```

   - 2. 基于 web-vitals 包

     ```typescript
     import { getFCP } from 'web-vitals';
     // 当 FCP 可用时立即进行测量和记录。
     getFCP(console.log);
     ```

3. LCP：可视区域的最大图像或者文本块渲染出来的时间

   - 收集的原因：用户觉得页面基本上块渲染完毕的时间

   - **采集方法**

   - 1. 基于 PerformanceObserver

        ```ts
        new PerformanceObserver((entryList) => {
          const entries = entryList.getEntries();
          const entry = entries[entries.length - 1];
          console.log('lcp', entry);
        }).observe({ type: 'largest-contentful-paint', buffered: true });
        ```

     2. 基于 web-vitals 包

        ```
        import { getLCP } from 'web-vitals';
        // 当 LCP 可用时立即进行测量和记录。
        getLCP(console.log);

        ```

   -

###### `交互延迟` 用户什么时候可以感受到界面可以开始交互，交互后需要等待多久才有回应：

1. FID：用户首次于页面交互到浏览器做出相应开始执行时间处理程序的经过的时间

   - 收集的原因：用户的第一次交互往往是很重要的，第一次交互迟迟没有响应，用户就可能放弃

   - **采集方法**：

   - 1. 基于 PerformanceObserver

        ```ts
        new PerformanceObserver((entryList) => {
          const entries = entryList.getEntries();
          const entry = entries[entries.length - 1];
          const delay = entry.processingStart - entry.startTime;
          console.log('FID:', delay, entry);
        }).observe({ type: 'first-input', buffered: true });
        ```

     2. 基于 web-vitals

        ```ts
        import { getFID } from 'web-vitals';
        // 当 FID 可用时立即进行测量和记录。
        getFID(console.log);
        ```

2. INP：页面生命周期中用户与网页交互中最长的一次持续时间

   - 收集的原因：用户浏览网页过程中，如果存在某次交互迟迟没有响应，用户就可能放弃

   - **采集方法：**

   - 1. 基于 web-vitals

        ```ts
        import { getINP } from 'web-vitals';
        // 当 INP 可用时立即进行测量和记录。
        getINP(console.log);
        ```

3. TTI：最早可交互时间：

   - **详细定义**：

   - TTI 指标用于衡量从网页开始加载到其主要子资源加载完成所用的时间，并且能够快速可靠地响应用户输入

   - **标准计算方式**：

   - 1. 从 FCP 开始。
     2. 向前搜索一个至少 5 秒的静默窗口，其中*静默窗口*的定义为：没有长任务且不超过两个进行中的网络 GET 请求。
     3. 向后搜索静默窗口之前的最后一个长任务，如果找不到长任务，则停止在 FCP 处停止。
     4. TTI 是安静窗口之前的最后一个长任务的结束时间（如果未找到长任务，则与 FCP 值相同）。

     **开发中的计算方式：**

     ```typescript
     const navigation =
       // W3C Level2  PerformanceNavigationTiming
       // 使用了High-Resolution Time，时间精度可以达毫秒的小数点好几位。
       performance.getEntriesByType('navigation').length > 0
         ? performance.getEntriesByType('navigation')[0]
         : performance.timing; // W3C Level1  (目前兼容性高，仍然可使用，未来可能被废弃)。
     const TTI = navigation.domInteractive - navigation.fetchStart;
     ```

   > **注意**：是一项实验室指标，虽然可以衡量现场 TTI，但不建议这样做，因为用户互动可能会对网页的 TTI 造成影响，导致报告中出现大量差异。若要了解网页在该字段中的互动情况，应该衡量 INP。

###### `视觉稳定`：界面视觉上的变化对用户造成的负面影响大小

1. CLS：网页生命周期内发生的每次意外布局偏移的最大*布局偏移得分*，每当一个已渲染的可见元素的位置从一个可见位置变更到下一个可见位置时，就发生了 **布局偏移**

   - 收集的原因：用户很讨厌页面布局偏移过大，比如，当用户点击一个按钮的时候，突然按钮的位置变了，导致用户点到了其他的东西，比如广告之类的。

   - **采集方法：**

   - 1. 基于 web-vitals：

        ```ts
        import { getCLS } from 'web-vitals';
        // 当 CLS 可用时立即进行测量和记录。
        getCLS(console.log);
        ```

##### 精准定位问题，以技术为中心的关键时间点，时间段：

###### 页面加载过程关键数据获取方式：

利用`performance.getEntriesByType('navigation')[0]` or `performance.timing`(如果前者浏览器不支持，使用这个)得到的数据对象身上的字段：

`navigationStart`：表示上一个文档卸载结束时的 unix 时间戳，如果没有上一个文档，则等于 fetchStart。

`unloadEventStart`：表示前一个网页（与当前页面同域）unload 的时间戳，如无前一个网页 unloade 或前一个网页与当前不同域，则为 0。

`unloadEventEnd`: 返回前一个 unload 时间绑定的回调执行完毕的时间戳。

`redirectStart`：前一个 Http 重定向发送时的时间。有跳转且是同域名内重定向，否则为 0。

`redirectEnd`：前一个 Http 重定向完成时的时间。有跳转且是同域名内重定向，否则为 0。

`fetchStart`：浏览器准备使用 http 请求文档的时间，在检查本地缓存之前。

`domainLookupStart/domainLookupEnd`：DNS 域名查询开始/结束的时间，如果使用本地缓存（则无需 DNC 查询）或持久链接，则和 fetchStart 一致。

`connectStart`：HTTP（TCP）开始或重新建立链接的时间，如果是持久链接，则和 fetchStart 一致。

`connectEnd`：HTTP（TCP）完成建立链接的时间（完成握手），如果是持久链接，则和 fetchStart 一致。

`secureConnectionStart`：Https 链接开始的时间，如果不是安全链接则为 0。

`requestStart`：http 在建立链接之后，正式开始请求真实文档的时间，包括从本地读取缓存。

`responseStart`：http 开始接收响应的时间（获取第一个字节），包括从本地读取缓存。

`responseEnd`：http 响应接收完全的时间（最后一个字节），包括从本地读取缓存。

`domLoading`：开始解析渲染 DOM 树的时间。

`domInteractive`：完成解析 DOM 树的时间(这个时候放在 html 头部的脚本已经执行完毕，async，defter 属性的脚本已经开始加载)。

`domContentLoadedEventStart`：DOM 解析完成，同步的脚本，defter，async 属性的脚本下载并执行完毕后，页面内资源加载开始的时间，此时 dom 依赖的图片、子框架和异步脚本，css 样式表等其他内容不一定完成加载加载。

`domContentLoadedEventEnd`：domContentLoadedEvent 事件结束

`domComplete`：当文档的 DOM 树已经构建完成、所有的子资源（如样式表、脚本、图片等）都已经下载完成时。这意味着文档的结构已经完全准备好，但是可能一些外部资源（如图片等）还在进行下载或者脚本在执行。在这个阶段，页面已经可以被渲染和展示给用户了，但可能还有些许的异步加载的资源没有完成加载。

`loadEventStart`：整个页面及所有依赖资源如样式表和图片都已完成加载时触发，这个时候可以认为 dom 渲染完毕

`loadEventEnd`：load 函数执行完毕的事件、

###### 可以计算出来的页面加载过程的关键时间段

- 页面卸载阶段：
  1. 上一个页面卸载耗时：unloadEventEnd - unloadEventStart
- 网络部分：
  1. 重定向耗时：redirectEnd - redirectStart
  2. 文档开始获取时间时间：fetchStart
  3. 取缓存耗时：domainLookupStart - fetchStart
  4. DNS 查询耗时：domainLookupEnd - domainLookupStart
  5. ssl 四次握手耗时：connectEnd - secureConnectionStart
  6. TCP 建立耗时：connectEnd - connectStart
  7. 请求响应耗时：responseStart - requestStart
  8. 响应传输耗时：responseEnd - responseStart。
- 页面解析渲染部分：
  1. dom 开始解析时间（dom 树开始构建的时间）：domLoading
  2. dom 解析完毕时间（dom 树构建完成的时间）：domInteractive
  3. dom 解析完毕并且脚本执行完毕的时间：domContendLoadEventStart
  4. 所有资源加载完成，dom 完全渲染的时间：loadEventStart

###### 资源加载过程中关键数据的获取方式：

利用`performance.getEntriesByType('resource')`获取到所有静态资源加载的关键信息

###### 可以得到的静态资源信息和加载中的关键时间段

- 静态资源信息
  1. `name`: 资源地址
  2. `initiatorType`: 资源类型
  3. `transferSize`:资源大小
  4. `startTime`: 开始时间
  5. `responseEnd`: 结束时间
  6. `duration`: 消耗时间
- 网络部分
  1. 重定向耗时：redirectEnd - redirectStart
  2. 文档开始获取时间时间：fetchStart
  3. 取缓存耗时：domainLookupStart - fetchStart
  4. DNS 查询耗时：domainLookupEnd - domainLookupStart
  5. ssl 四次握手耗时：connectEnd - secureConnectionStart
  6. TCP 建立耗时：connectEnd - connectStart
  7. 请求响应耗时：responseStart - requestStart
  8. 响应传输耗时：responseEnd - responseStart。

###### 静态资源加载的缓存命中率的计算

1. 直接判断`performance.getEntriesByType('resource')`返回的资源数据对象数组中的元素的`deliveryType`属性是否为`cache`
2. 同时满足`durantion === 0` 和`transferSize === 0`

### 行为监控：

#### 行为监控中需要监控一些什么，提供一些什么功能？

##### 监控 pv，uv，从而得到流量变化，统计出热点页面

###### pv: 页面访问量

**获取方法**：路由跳转就说明访问了页面，在路由跳转的时候上报也就是`popState`事件触发时，上报此时的事件，页面信息，用户来路

###### uv：用户访问量

获取方法：对外暴露一个设置用户唯一标识的方法，让用户在合适的时候调用这个方法设置唯一标识，后续每次数据上报时都带上这个唯一标识，后端来通过时间戳和这个唯一标识，来统计不同时间段的 uv

##### 监控用户设备信息，浏览器信息，从而得到大多数用户的屏幕数据，浏览器类型从而更好的适配

利用`window.navigation`获取到 userAgent,再利用`broser`和`ua-parser-js`这两个包对其进行解析

```ts
import parser from 'ua-parser-js';
import Bowser from 'bowser';

function resolveUserAgent(userAgent: string) {
  const browserData = Bowser.parse(userAgent);
  const parserData = parser(userAgent);
  const browserName = browserData.browser.name ?? parserData.browser.name; // 浏览器名
  const browserVersion = browserData.browser.version ?? parserData.browser.version; // 浏览器版本号
  const osName = browserData.os.name ?? parserData.os.name; // 操作系统名
  const osVersion = parserData.os.version ?? browserData.os.version; // 操作系统版本号
  const deviceType = browserData.platform.type ?? parserData.device.type; // 设备类型
  const deviceVendor = browserData.platform.vendor ?? parserData.device.vendor ?? ''; // 设备所属公司
  const deviceModel = browserData.platform.model ?? parserData.device.model ?? ''; // 设备型号
  const engineName = browserData.engine.name ?? parserData.engine.name; // engine名
  const engineVersion = browserData.engine.version ?? parserData.engine.version; // engine版本号
  return {
    browserName,
    browserVersion,
    osName,
    osVersion,
    deviceType,
    deviceVendor,
    deviceModel,
    engineName,
    engineVersion,
  };
}
```

##### 监控用户的行为，路由跳转，接口请求，当前访问的页面信息， 并按顺序存入用户行为栈，从而准确定位用户的操作

###### 用户行为的监控：

考虑到监听所有元素的所有方法会过于消耗性能，所以支持用户通过配置对象的方式在初始化 sdk 的时候配置`elementTrackedList`, `classTrackedList`, `eventTrackedList`从而监听指定元素 和 指定 class 的元素的指定事件。

```ts
private initDomHandler() {
    this.eventTrackedList.forEach((eventName) => {
      window.addEventListener(
        eventName,
        (event) => {
          let target = this.elementTrackedList.includes(
            (event.target as HTMLElement)?.tagName?.toLocaleLowerCase(),
          )
            ? (event.target as HTMLElement)
            : undefined;
          target =
            target ??
            Array.from((event.target as HTMLElement)?.classList).find((className) =>
              this.classTrackedList.includes(className),
            )
              ? (event.target as HTMLElement)
              : undefined;

          if (!target) return;

          const domData = {
            tagInfo: {
              id: target.id,
              classList: Array.from(target.classList),
              tagName: target.tagName,
              text: target.textContent,
            },
            pageInfo: getPageInformation(),
            time: new Date().getTime(),
            timeFormat: formatDate(new Date()),
          };

          if (!this.data[UserActionMetricsName.DBR]) {
            this.data[UserActionMetricsName.DBR] = { [eventName]: [domData] };
          } else if (!this.data[UserActionMetricsName.DBR][eventName]) {
            this.data[UserActionMetricsName.DBR][eventName] = [domData];
          } else {
            this.data[UserActionMetricsName.DBR][eventName].push(domData);
          }

          const hehaviorStackData = {
            name: eventName,
            page: getPageInformation().pathname,
            value: {
              tagInfo: {
                id: target.id,
                classList: Array.from(target.classList),
                tagName: target.tagName,
                text: target.textContent,
              },
              pageInfo: getPageInformation(),
            },
            time: new Date().getTime(),
            timeFormat: formatDate(new Date()),
          };
          this.hehaviorStack.push(hehaviorStackData);
        },
        true,
      );
    });
  }
```

###### 页面基本信息获取：

1. 通过`window.location`获取到当前页面的`protocol`, `host`,`port` `origin`, `href`, `pathname`, `search`
2. 利用`window.screen`获取到用户屏幕的宽高
3. 通过`document.title`获取到网页的 title

###### 路由跳转的监控

- 监控`hash`路由

  `hash`路由的`hash`变化会触发`hashchange`时间，但是也会触发`popstate`事件，所以我们统一监听`popstate`事件即可

- 监控`history`路由

  1. `history`路由的`back`， `forword`， `go`方法都会触发`popstate`事件，所以我们可以监听`popstate`事件

  2. 但是`pushState`， `replaceState`方法不会触发对应的事件和`popstate`事件，所以我们需要对其进行重写，在原有功能的基础上通过`window.dispatch`派发事件

     ```ts
     export function writePushStateAndReplaceState() {
       history.pushState = writeHistoryEvent('pushState');
       history.replaceState = writeHistoryEvent('replaceState');
     }

     function writeHistoryEvent(type: keyof History) {
       const originEvent = history[type];
       return function (this: any) {
         const result = originEvent.apply(this, arguments);
         const event = new Event(type);
         window.dispatchEvent(event);
         return result;
       };
     }
     ```

###### 界面停留事件的监控

1. 在 load 事件里记录初次进入界面的事件，把该界面的信息（url， startTime， endTime）推送进栈中
2. 在 popSate 事件触发，也就是路由跳转时，记录上一个界面也就是栈顶的哪个界面信息的结束时间
3. 在 beforeunload 中计算出最后一个界面的结束时间

###### 访客来路的监控：

监控访客来路的原因：

1. 获取到流量是从哪个界面来的

   - 获取方法：

   - 通过`document.referer`来获取到网页前一个地址

   - > 注意：以下场景获取到的值为空
     >
     > - 直接在地址栏中输入地址跳转
     > - 直接通过浏览器收藏夹打开
     > - 从 https 的网站直接进入一个 http 协议的网站

2. 当发生 404 等问题的时候，知道是从哪个界面调过来的

   获取方法：

   利用`window.performance.navigation.type`属性，它返回一个整数，有以下 4 种情况

   - - `0`: 点击链接、地址栏输入、表单提交、脚本操作等。
   - - `1`: 点击重新加载按钮、location.reload。
   - - `2`: 点击前进或后退按钮。
   - - `255`: 任何其他来源。即非刷新/非前进后退、非点击链接/地址栏输入/表单提交/脚本操作等。

###### 对外暴露上传数据的接口，从而支持用户自己手动埋点，提高灵活性

1. 通过对外暴露`report`方法，支持用户自定义埋点，上传数据和数据类型
2. 并且每次的上报都会默认带上`appID`来区别不同的 app, `uid`来区别不同的用户统计 uv, `time`来记录时间，`extra`来支持用户通过对外暴露的`setExtra`方法来配置每次请求都必须带上的数据

```ts
public report<T extends Record<string, any>>(data: T, type: string) {
    const params = Object.assign(
      { data, type },
      {
        appId: this.appId,
        uid: this.uid,
        extra: this.extra,
        time: new Date().getTime(),
        timeFormat: formatDate(new Date()),
      },
    );
    const blob = new Blob([JSON.stringify(params)]);
    navigator.sendBeacon(this.options.requestUrl as string, blob);
  }
```

### 异常监控：

#### js 运行错误监控

1. 在全局监控`error`事件
2. 根据错误类型，文件名，信息生成一个随机 id，避免相同的错误重复上传
3. 将 event.message, event.error.name（错误类型）,event.error(错误堆栈，可以让 gpt 写一个函数来解析这个错误堆栈)， event.filename, event.colno, event.lineno 上传

#### 资源加载错误（src or href 错了）

1. 在全局监控 error 事件，通过 event.targrt.tagName 是不是 link or script or img or a 来判断是不是资源加载错误
2. 根据错误类型，src 生成一个随机 id，避免相同的错误重复上传
3. 将 event.tatget.src, event.target.tagName, outerHTMl 上传

#### promise 错误监控

1. 在全局监控`unhandledRejection`事件
2. 根据错误信息，错误名字，生成一个随机 id，避免相同的错误重复上传
3. 上传 event.reason.message, event.reason.name, event.reason(错误堆栈，可以让 gpt 写一个函数来解析这个错误堆栈)

#### 跨域错误

1. 在全局监听`error`事件
2. 通过 event.message 是否为 `Script error`来判断是否是跨域请求
3. 上报跨域请求错误

### http 请求监控

我们通常需要监控 http 请求的各种信息，`请求地址`， `方法`， `状态码`， `请求耗时`， `请求体`， `响应`， 从而在接口报错时及时发现，统计出流量大的后端服务，经常出问题的后端服务。

#### XMLHTTPRequest 的劫持：

1. 重写 xhr 对象的`open`方法，在里面采集到请求的 url 和 method

2. 重写 xhr 对象的`send`方法，在里面采集到请求的 body，并记录时间，为后续记录接口耗时做准备

3. 监听 xhr 对象的 loadend 事件，然后从 xhr 对象中得到，状态码，状态码短语，response, 然后执行传递的函数来讲数据添加进 sdk 的 data 等待上报

   ```ts
   export function proxyXmlHttp(loadHandler: (...args: any[]) => any) {
     if ('XMLHttpRequest' in window && typeof window.XMLHttpRequest === 'function') {
       const oXMLHttpRequest = window.XMLHttpRequest;
       if (!(window as any).oXMLHttpRequest) {
         // oXMLHttpRequest 为原生的 XMLHttpRequest，可以用以 SDK 进行数据上报，区分业务
         (window as any).oXMLHttpRequest = oXMLHttpRequest;
       }
       (window as any).XMLHttpRequest = function () {
         // 覆写 window.XMLHttpRequest
         // eslint-disable-next-line new-cap
         const xhr = new oXMLHttpRequest();
         // eslint-disable-next-line @typescript-eslint/unbound-method
         const { open, send } = xhr;
         // eslint-disable-next-line @typescript-eslint/consistent-type-assertions
         let httpMetrics: HttpMetrics = {} as HttpMetrics;
         xhr.open = (method, url) => {
           httpMetrics.method = method;
           httpMetrics.url = url;
           open.call(xhr, method, url, true);
         };
         xhr.send = (body) => {
           httpMetrics.body = body ?? '';
           httpMetrics.requestTime = new Date().getTime();
           httpMetrics.requestTimeFormat = formatDate(new Date());
           send.call(xhr, body);
         };
         xhr.addEventListener('loadend', () => {
           const { status, statusText, response } = xhr;
           httpMetrics = {
             ...httpMetrics,
             status,
             statusText,
             response: response && JSON.parse(response),
             responseTime: new Date().getTime(),
             responseTimeFormat: formatDate(new Date()),
           };
           if (typeof loadHandler === 'function') loadHandler(httpMetrics);
         });
         return xhr;
       };
     }
   }
   ```

## 前端监控需要注意的点：

### 数据的上报方式：

#### 使用的上报方法：

1. 优先使用 sendBeacon 方法

   **原因：**

   - 是异步发送数据，不会阻塞页面的卸载和跳转

   - 可靠性很高，不会因为页面的关闭 or 跳转 而 终止发送

   - 虽然只支持 post 请求，但是用来发送埋点的统计数据，却足够了，也极为合适

2. 如果浏览器不支持 sendBeacon 方法再降级使用 ajax 方法

### 数据的上报时机：

#### 需要实时上报的：

1. 异常上报，需要实时上报
2. pv，用户行为都是触发就上报

#### 不需要实时上报的

1. 用户行为栈，http 请求记录都放到各自的一个队列里，满了再集中上报
2. 其他不会很多的数据在界面卸载前上报

### 当流量爆炸的时候，怎么进行削峰限流

#### 1. 简单方法：

直接丢弃百分之 20 的用户的数据，具体方法，Math.radom（）生成一个随机的方法，丢其小于 0.2 的用户的数据

### 异常上报的时候，需要根据异常的信息生成一个唯一性标识，避免同一错误的重复上报

## 前端监控提供的监控模块，应该支持用户按需选择其需要的：

1. 支持用户通过配置对象，按需选择它需要的性能监控，行为监控，异常监控，http 监控功能
2. 当用户开启性能监控的时候，应支持用户通过配置对象，按需选择自己需要收集的性能指标和关键时间段，缓存命中率
3. 当用户开启行为监控的时候，应支持用户通过配置对象，按需选择自己是否需要收集，pv，uv， 设备信息，界面信息，来源信息，路由跳转信息，用户行为栈，页面停留时间。
4. 当用户开启异常监控的时候应支持用户通过配置对象，按需选择自己是否需要 js 运行错误监控，资源加载错误监控，promise 错误监控，跨域错误监控，白屏错误监控
