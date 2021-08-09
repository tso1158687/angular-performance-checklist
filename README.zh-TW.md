# Angular 效能清單

<img src="./assets/flash.png" width="1000">

- [中文版](./README.zh-CN.md)
- [Русский](./README.ru-RU.md)
- [Português](./README.pt-BR.md)
- [Español](./README.es-ES.md)
- [日本語](./README.ja-JP.md)
- [正體中文](./README.zh-TW.md)

## 介紹

本文列出許多改善 Angular 應用程式的性能的方法。「Angular 效能清單」涵蓋諸多主題，從伺服器端的渲染、預先渲染、建構應用程式到執行時的效能與框架變更檢測性能的最佳化

本文主要分為兩大部分:

- 網路效能 - 列出許多可以改善應用程式載入時間的方法，包含網路延遲或頻寬不足的對應方法。
- 執行效能 - 改善應用程式執行時的效能，包含變更檢測和渲染相關的最佳化。

有些例子會影響這兩大部分，因此可能有些交集，但是將會明確提到兩者之間在使用案例和影響的差異。

大多數小節所列出來與特定例子相關的工具，可以幫助我們透過自動化的開發流程提高效率。

注意大多數的例子都適用於HTTP/1.1 和 HTTP/2。通過指定它們可以應用於哪個版本的協議來提及例外的例子。

## 目錄

- [Angular 效能清單](#angular-2-performance-checklist)
  - [介紹](#introduction)
  - [目錄](#table-of-content)
  - [網路效能](#network-performance)
    - [打包](#bundling)
    - [最小化與清除無用程式碼](#minification-and-dead-code-elimination)
    - [移除模板空白](#移除模板空白)
    - [搖樹優化](#tree-shaking)
    - [搖樹優化提供者](#tree-shakeable-providers)
    - [預先（AOT）編譯器](#ahead-of-time-aot-compilation)
    - [壓縮](#compression)
    - [預先載入資源](#預先載入資源)
    - [惰性載入資源](#惰性載入資源資源)
    - [Don't lazy-load default route](#dont-lazy-load-the-default-route)
    - [快取](#caching)
    - [使用 Application Shell](#use-application-shell)
    - [使用 Service Workers](#use-service-workers)
  - [運行時最佳化](#runtime-optimizations)
    - [使用 `enableProdMode`](#use-enableprodmode)
    - [預先（AOT）編譯器](#ahead-of-time-compilation)
    - [Web Workers](#web-workers)
    - [伺服器端渲染](#server-side-rendering)
    - [變更檢測](#change-detection)
      - [`ChangeDetectionStrategy.OnPush`](#changedetectionstrategyonpush)
      - [Detaching the Change Detector](#detaching-the-change-detector)
      - [Run outside Angular](#run-outside-angular)
      - [Coalescing event change detections](#coalescing-event-change-detections)
    - [使用純管道](#use-pure-pipes)
    - [`*ngFor` 指令](#ngfor-directive)
      - [使用 `trackBy` 選項](#use-trackby-option)
      - [最小化 DOM 元素](#最小化 DOM 元素)
    - [最佳化模板語法](#optimize-template-expressions)
- [結論](#conclusion)
- [貢獻](#貢獻)

## 網路效能

在本章節的部分工具因為仍在開發中，所以可能有更動。 Angular 核心團隊正在努力讓
Some of the tools in this section are still in development and are subject to change. The Angular core team is working on automating the build process for our applications as much as possible so a lot of things will happen transparently.

### 打包

打包是一個標準的動作，目的在降低瀏覽器在使用者執行應用程式的各種動作的多次請求的請求數
 In essence, the bundler receives as an input a list of entry points and produces one or more bundles. 這樣瀏覽器執行幾個請求就可以取得整個應用程式，而非單獨請求個別資源。

隨著你的應用程式不斷成長，且將所有的東西都打包進單一龐大的應用程式中反而是適得其反。這時候需要使用 Webpack 去拆分程式碼

**Additional http requests will not be a concern with HTTP/2 because of the [server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push) feature.**

**工具**
幫助我們更有效率打包應用程式的工具：

- [Webpack](https://webpack.js.org) - provides efficient bundling by performing [tree-shaking](#tree-shaking).
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - Techniques to split your code.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - Need for splitting with http2.
- [Rollup](https://github.com/rollup/rollup) - provides bundling by performing efficient tree-shaking, taking advantage of the static nature of the ES2015 modules.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs plenty of optimizations and provides bundling support. Originally written in Java, since recently it also has a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler) that can be [found here](https://www.npmjs.com/package/google-closure-compiler).
- [SystemJS Builder](https://github.com/systemjs/builder) - provides a single-file build for SystemJS of mixed-dependency module trees.
- [Browserify](http://browserify.org/).

**資源**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 最小化與清除無用程式碼

這些例子可以減少我們應用程式的負荷以淺少頻寬的消耗

**工具**

- [Uglify](https://github.com/mishoo/UglifyJS) - performs minification such as mangling variables, 移除註解與空白或刪除無用程式碼等等。 Written completely in JavaScript, has plugins for all popular task runners.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs similar to uglify type of minification. In advanced mode, it transforms the AST of our program aggressively in order to be able to perform even more sophisticated optimizations. It has also a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler) that can be [found here](https://www.npmjs.com/package/google-closure-compiler). GCC also supports *most of the ES2015 modules syntax* so it can [perform tree-shaking](#tree-shaking).

**資源**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 移除模板空白

雖然我們看不到空白字元 （與 `\s` 正則表達式相符的字元，仍然會佔用位元且通過網路傳輸。如果將模板中的空白降低到最小，將能降低在 AOT 模式下程式碼打包的大小。

幸好我們不用手動做這件事情。
 The `ComponentMetadata` interface provides the property `preserveWhitespaces` which by default has value `false` meaning that by default the Angular compiler will trim whitespaces to further reduce the size of our application. In case we set the property to `true` Angular will preserve the whitespace.

- [preserveWhitespaces in the Angular docs](https://angular.io/api/core/Component#preserveWhitespaces)

### 搖樹優化

我們通常不會在應用程式的最終版本使用 Angular 或任何第三方套件，甚至不會使用所有我們自己寫的程式碼。幸好有 ES2015 模組的靜態特性，我們可以剔除未引用至我們應用程式的程式碼

**範例**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```
當使用搖樹優化且打包至 `app.js` 會得到

```javascript
let foo = () => 'foo';
console.log(foo());
```

也就是說，沒有用到的匯出 `bar` 將不會被包含至最終的打包

**工具**

- [Webpack](https://webpack.js.org) - provides efficient bundling by performing [tree-shaking](#tree-shaking). Once the application has been bundled, it does not export the unused code so it can be safely considered as dead code and removed by Uglify.
- [Rollup](https://github.com/rollup/rollup) - provides bundling by performing an efficient tree-shaking, taking advantage of the static nature of the ES2015 modules.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - performs plenty of optimizations and provides bundling support. Originally written in Java, since recently it has also a [JavaScript version](https://www.npmjs.com/package/google-closure-compiler) that can be [found here](https://www.npmjs.com/package/google-closure-compiler).

*注意:* GCC does not support `export *` yet, which is essential for building Angular applications because of the heavy usage of the "barrel" pattern.

**資源**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["Using pipeable operators in RxJS"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)

### 搖樹優化提供者

自從 Angular 6 發布之後， Angular 團隊提供一項新特色允許將服務 (service) 搖樹優化，這表示除非你的服務在其他服務或元件中使用到，不然將不會包含在最終的打包當中。刪除未使用的程式碼，將有助於縮小打包過後的大小。

你可以使用 `providedIn` 屬性讓你的服務可以被搖樹。再來就可以從 `NgModule` 
You can make your services tree-shakeable by using the `providedIn` attribute to define where the service should be initialized when using the `@Injectable()` decorator. Then you should remove it from the `providers` attribute of your `NgModule` declaration as well as its import statement as follows.

Before:

```ts
// app.module.ts
import { NgModule } from '@angular/core'
import { AppRoutingModule } from './app-routing.module'
import { AppComponent } from './app.component'
import { environment } from '../environments/environment'
import { MyService } from './app.service'

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    ...
  ],
  providers: [MyService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
// my-service.service.ts
import { Injectable } from '@angular/core'

@Injectable()
export class MyService { }
```

After:

```ts
// app.module.ts
import { NgModule } from '@angular/core'
import { AppRoutingModule } from './app-routing.module'
import { AppComponent } from './app.component'
import { environment } from '../environments/environment'

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    ...
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
// my-service.service.ts
import { Injectable } from '@angular/core'

@Injectable({
  providedIn: 'root'
})
export class MyService { }
```

If `MyService` is not injected in any component/service, then it will not be included in the bundle.

**資源**

- [Angular Providers](https://angular.io/guide/providers)

### 預先（AOT）編譯器

A challenge for the available in the wild tools (像是 GCC、 Rollup 等等) are the HTML-like templates of the Angular components, which cannot be analyzed with their capabilities. This makes their tree-shaking support less efficient because they're not sure which directives are referenced within the templates. The AoT compiler transpiles the Angular HTML-like templates to JavaScript or TypeScript with ES2015 module imports. This way we are able to efficiently tree-shake during bundling and remove all the unused directives defined by Angular, third-party libraries or by ourselves.

**資源**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Compression

Compression of the responses' payload standard practice for bandwidth usage reduction. By specifying the value of the header `Accept-Encoding`, the browser hints the server which compression algorithms are available on the client's machine. On the other hand, the server sets the value for the `Content-Encoding` header of the response in order to tell the browser which algorithm has been chosen for compressing the response.

**工具**

The 工具 here is not Angular-specific and entirely depends on the web/application server that we're using. Typical compression algorithms are:

- deflate - a data compression algorithm and associated file format that uses a combination of the LZ77 algorithm and Huffman coding.
- [brotli](https://github.com/google/brotli) - a generic-purpose lossless compression algorithm that compresses data using a combination of a modern variant of the LZ77 algorithm, Huffman coding, and 2nd order context modeling, with a compression ratio comparable to the best currently available general-purpose compression methods. It is similar in speed with deflate but offers more dense compression.

**資源**

- ["Better than Gzip Compression with Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 預先載入資源
預先載入資源是一個很讚改進使用者體驗的方法。我們可以預先取得檔案（圖片、樣式或想要[惰性載入](#lazy-loading-of-資源)的模組等等）或資料。這裡有不同預先載入的策略，但大多數取決於應用程式的個別情形

### 惰性載入資源

In case the target application has a huge code base with hundreds of dependencies, the practices listed above may not help us reduce the bundle to a reasonable size (reasonable might be 100K or 2M, it again, completely depends on the business goals).

In such cases, a good solution might be to load some of the application's modules lazily. For instance, let's suppose we're building an e-commerce system. In this case, we might want to load the admin panel independently from the user-facing UI. Once the administrator has to add a new product we'd want to provide the UI required for that. This could be either only the "Add product page" or the entire admin panel, depending on our use case/business requirements.

**工具**

- [Webpack](https://github.com/webpack/webpack) - allows asynchronous module loading.
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - router preloading strategy which automatically downloads the lazy-loaded modules associated with all the visible links on the screen

### 請勿使用惰性載入預設的路由

假設有以下的路由設定

```ts
// 不好的例子
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: () => import('./dashboard.module').then(mod => mod.DashboardModule) },
  { path: 'heroes', loadChildren: () => import('./heroes.module').then(mod => mod.HeroesModule) }
];
```
當使用者第一次打開應用程式的網址為: https://example.com/ ，將會被導向至 `/dashboard`
The first time the user opens the application using the url: https://範例.com/ they will be redirected to `/dashboard` which will trigger the lazy-route with path `dashboard`. In order Angular to render the bootstrap component of the module, it will has to download the file `dashboard.module` and all of its dependencies. Later, the file needs to be parsed by the JavaScript VM and evaluated.

Triggering extra HTTP requests and performing unnecessary computations during the initial page load is a bad practice since it slows down the initial page rendering. Consider declaring the default page route as non-lazy.

### 快取

Caching is another common practice intending to speed-up our application by taking advantage of the heuristic that if one resource was recently been requested, it might be requested again in the near future.

For caching data we usually use a custom caching mechanism. For caching static assets, we can either use the standard browser caching or Service Workers with the [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).

### Use Application Shell

To make the perceived performance of your application faster, use an [Application Shell](https://developers.google.com/web/updates/2015/11/app-shell).

The application shell is the minimum user interface that we show to the users in order to indicate them that the application will be delivered soon. For generating an application shell dynamically you can use Angular Universal with custom directives which conditionally show elements depending on the used rendering platform (i.e. hide everything except the App Shell when using `platform-server`).

**工具**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (isomorphic) JavaScript support for Angular.

**資源**

- ["Instant Loading Web Apps with an Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)

### Use Service Workers

We can think of the Service Worker as an HTTP proxy which is located in the browser. All requests sent from the client are first intercepted by the Service Worker which can either handle them or pass them through the network.

You can add a Service Worker to your Angular project by running
``` ng add @angular/pwa ```

**工具**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Offline Plugin for Webpack](https://github.com/NekR/offline-plugin) - Webpack plugin that adds support for Service Worker with a fall-back to AppCache.

**資源**

- ["The offline cookbook"](https://jakearchibald.com/2014/offline-cookbook/)
- ["Getting started with service workers"](https://angular.io/guide/service-worker-getting-started)

## 運行時最佳化
本章節包含一些應用的實例以提供更順暢的60幀 (fps) 使用者體驗。

### 使用 `enableProdMode`

在開發模式 Angular 執行額外的檢查去確認檢測不會導致對任何綁定的任何額外更改。這樣一來，框架可以確保遵循單向資料流

在正式產品要關閉這些檢測，別忘記呼叫 `enableProdMode`:

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### Ahead-of-Time Compilation

<!-- TODO:還沒好 -->
AOT 不僅可以在執行搖樹的時候更有效率，更可以增加應用程式在執行時的效能。相較於 AOT 的另外一個選擇是在執行期間執行的 JIT ，因此我們可以當作建構應用程式的流程，降低
The alternative of AoT is Just-in-Time compilation (JiT) which is performed runtime, therefore we can reduce the amount of computations required for the rendering of our application by performing the compilation as part of our build process.

**工具**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - a starter project which includes support for AoT compilation.
- [angular-cli](https://cli.angular.io) Using the `ng serve --prod`

**資源**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Web Workers

我們最常在典型的 SPA 遇到的問題是，我們的程式碼執行在單執行緒裡面。也就是說我們如果想要在 60 FPS 達到絲滑柔順的使用者體驗，**最多只有16毫秒**的時間去渲染各個畫面，不然幀數將會下降一半。

在元件樹龐大的複雜應用程式裡面，每秒需要執行數以萬計的變更偵測檢查，這樣很容易讓幀數下降。幸好 Angular 將 DOM 解構進行了解構，可以在 Web Worker中執行整個應用程式（包含變更檢測），讓主 UI 執行緒只是則渲染。

**工具**

- 由核心團隊打造的模組，讓我們可以的應用程式可以在 Web Worker 執行。可以在[這裡](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers)找到使用範例。
（譯者註：找不到，網頁404）
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - webpack 的 Web Worker 載入器

**資源**

- ["Using Web Workers for more responsive apps"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### 伺服器端渲染

傳統 SPA 網站最大的問題是在尚未取得所有渲染頁面所需 Javascript 之前，無法渲染任何東西。這會導致兩大問題： 

- 不是所有的搜尋引擎都會執行頁面所需的 Javascript，因此會無法正確索引到動態應用程式的內容。
- 糟糕的使用者體驗，因為在載入頁面所需的 Javascript 下載、解析、執行完之前，使用者只會看到空白或載入的畫面。

伺服器端渲染藉由在伺服器收到請求時，預先渲染頁面的內容來解決問題且在初始頁面載入提供頁面標記來解決問題

**工具**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Angular 統一平台（同構應用）對 Javascript 的支援。
- [Preboot](https://github.com/angular/preboot) - Library to help manage the transition of state (i.e. events, focus, data) from a server-generated web view to a client-generated web view.
- [Scully](https://github.com/scullyio/scully) - 擁抱 JAMStack 的 Angular 專案靜態網站產生器。

**資源**

- ["Angular Server Rendering"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular Universal Patterns"](https://www.youtube.com/watch?v=TCj_oC3m6_U)
- ["Create a Static Site Using Angular & Scully"](https://www.youtube.com/watch?v=ugTx-14jRrI)

### Change Detection

On each asynchronous event, Angular performs change detection over the entire component tree. Although the code which detects for changes is optimized for [inline-caching](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html), this still can be a heavy computation in complex applications. A way to improve the performance of the change detection is to not perform it for subtrees which are not supposed to be changed based on the recent actions.

#### `ChangeDetectionStrategy.OnPush`

The `OnPush` change detection strategy allows us to disable the change detection mechanism for subtrees of the component tree. By setting the change detection strategy to any component to the value `ChangeDetectionStrategy.OnPush` will make the change detection perform **only** when the component has received different inputs. Angular will consider inputs as different when it compares them with the previous inputs by reference, and the result of the reference check is `false`. In combination with [immutable data structures](https://facebook.github.io/immutable-js/), `OnPush` can bring great performance implications for such "pure" components.

**資源**

- ["Change Detection in Angular"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- ["Everything you need to know about change detection in Angular"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

#### Detaching the Change Detector

Another way of implementing a custom change detection mechanism is by `detach`ing and `reattach`ing the change detector (CD) for given a component. Once we `detach` the CD Angular will not perform check for the entire component subtree.

This practice is typically used when user actions or interactions with external services trigger the change detection more often than required. In such cases we may want to consider detaching the change detector and reattaching it only when performing change detection is required.

#### Run outside Angular

The Angular's change detection mechanism is being triggered thanks to [zone.js](https://github.com/angular/zone.js). Zone.js monkey patches all asynchronous APIs in the browser and triggers the change detection at the end of the execution of any async callback. In **rare cases**, we may want the given code to be executed outside the context of the Angular Zone and thus, without running change detection mechanism. In such cases, we can use the method `runOutsideAngular` of the `NgZone` instance.

**範例**

In the snippet below, you can see an 範例 for a component that uses this practice. When the `_incrementPoints` method is called the component will start incrementing the `_points` property every 10ms (by default). The incrementation will make the illusion of an animation. Since in this case, we don't want to trigger the change detection mechanism for the entire component tree, every 10ms, we can run `_incrementPoints` outside the context of the Angular's zone and update the DOM manually (see the `points` setter).

```ts
@Component({
  template: '<span #label></span>'
})
class PointAnimationComponent {

  @Input() duration = 1000;
  @Input() stepDuration = 10;
  @ViewChild('label') label: ElementRef;

  @Input() set points(val: number) {
    this._points = val;
    if (this.label) {
      this.label.nativeElement.innerText = this._pipe.transform(this.points, '1.0-0');
    }
  }
  get points() {
    return this._points;
  }

  private _incrementInterval: any;
  private _points: number = 0;

  constructor(private _zone: NgZone, private _pipe: DecimalPipe) {}

  ngOnChanges(changes: any) {
    const change = changes.points;
    if (!change) {
      return;
    }
    if (typeof change.previousValue !== 'number') {
      this.points = change.currentValue;
    } else {
      this.points = change.previousValue;
      this._ngZone.runOutsideAngular(() => {
        this._incrementPoints(change.currentValue);
      });
    }
  }

  private _incrementPoints(newVal: number) {
    const diff = newVal - this.points;
    const step = this.stepDuration * (diff / this.duration);
    const initialPoints = this.points;
    this._incrementInterval = setInterval(() => {
      let nextPoints = Math.ceil(initialPoints + diff);
      if (this.points >= nextPoints) {
        this.points = initialPoints + diff;
        clearInterval(this._incrementInterval);
      } else {
        this.points += step;
      }
    }, this.stepDuration);
  }
}
```

**Warning**: Use this practice **very carefully only when you're sure what you are doing** because if not used properly it can lead to an inconsistent state of the DOM. Also, note that the code above is not going to run in WebWorkers. In order to make it WebWorker-compatible, you need to set the label's value by using the Angular's renderer.

#### Coalescing event change detections

Angular 使用 zone.js to intercept events that occurred in the application and runs a change detection automatically. By default this happens when the [microtask queue](https://www.youtube.com/watch?v=cCOL7MC4Pl0) of the browser is empty, which in some cases may call redundant cycles.
From v9, Angular provides a way to coalesce event change detections by turning `ngZoneEventCoalescing` on, i.e
```typescript
platformBrowser()
  .bootstrapModule(AppModule, { ngZoneEventCoalescing: true });
```
The above configuration will schedule change detection with `requestAnimationFrame`, instead of plugging into the microtask queue, which will run checks less frequently and consume fewer computational cycles.

> Warning: **ngZoneEventCoalescing: true** may break existing apps that relay on consistently running change detection. 


**資源**
- [ngZoneEventCoalescing BootstrapOption](https://github.com/angular/angular/blob/master/packages/core/src/application_ref.ts#L268) - source code for BootstrapOptions interface
- [Reduce Change Detection Cycles with Event Coalescing in Angular](https://netbasal.com/reduce-change-detection-cycles-with-event-coalescing-in-angular-c4037199859f)
- [Simple Angular context help component or how global event listener can affect your perfomance](https://medium.com/@a.yurich.zuev/simple-angular-context-help-component-or-how-global-event-listener-can-affect-your-perfomance-75b67dba197f)

### Use pure pipes

As argument the `@Pipe` decorator accepts an object literal with the following format:

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

The pure flag indicates that the pipe is not dependent on any global state and does not produce side-effects. This means that the pipe will return the same output when invoked with the same input. This way Angular can cache the outputs for all the input parameters the pipe has been invoked with, and reuse them in order to not have to recompute them on each evaluation.

The default value of the `pure` property is `true`.

### `*ngFor` directive

The `*ngFor` directive is used for rendering a collection.

#### Use `trackBy` option

By default `*ngFor` identifies object uniqueness by reference.

Which means when a developer breaks reference to object during updating item's content Angular treats it as removal of the old object and addition of the new object. This effects in destroying old DOM node in the list and adding new DOM node on its place.

The developer can provide a hint for angular how to identify object uniqueness: custom tracking function as the `trackBy` option for the `*ngFor` directive. The tracking function takes two arguments: `index` and `item`. Angular uses the value returned from the tracking function to track items identity. It is very common to use the ID of the particular record as the unique key.

**範例**

```typescript
@Component({
  selector: 'yt-feed',
  template: `
  <h1>Your video feed</h1>
  <yt-player *ngFor="let video of feed; trackBy: trackById" [video]="video"></yt-player>
`
})
export class YtFeedComponent {
  feed = [
    {
      id: 3849, // note "id" field, we refer to it in "trackById" function
      title: "Angular in 60 minutes",
      url: "http://youtube.com/ng2-in-60-min",
      likes: "29345"
    },
    // ...
  ];

  trackById(index, item) {
    return item.id;
  }
}
```

#### 最小化 DOM 元素

當增加元素到畫面上的時候，渲染 DOM 元素通常是最成本最高的操作。最主要的工作通常來自於插入元素到 DOM 當中與應用樣式。假設 `*ngFor` 渲染大量的元素，瀏覽器（特別是那些舊的）可能會變慢且需要更多時間去完成渲染所有的元素。這不是只有 Angular 獨有的問題

為了降低渲染時間，試試看以下這些：
- 使用 [CDK](https://material.angular.io/cdk/scrolling/overview)的 virtual scrolling 或 [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller)
- 減少你的模板在 `*ngFor` 區域裡面渲染出來的 DOM 元素總量。通常不需要或未使用的 DOM 元素來自於你一次又一次擴充模板。好好思考一下怎樣的結構可以讓事情簡單一點。
- 如果可以的話，使用 [`ng-container`](https://angular.io/guide/structural-directives#ngcontainer)

**資源**

- ["NgFor directive"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - official documentation for `*ngFor`
- ["Angular — Improve performance with trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - shows gif demonstration of the approach
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - API description
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) - 顯示虛擬「無限的」列表

### Optimize template expressions

Angular executes template expressions after every change detection cycle. Change detection cycles are triggered by many asynchronous activities such as promise resolutions, http results, timer events, keypresses, and mouse moves.

Expressions should finish quickly or the user experience may drag, especially on slower devices. Consider caching values when their computation is expensive.

**資源**
- [quick-execution](https://angular.tw/guide/template-syntax#quick-execution) - 官方範本語法的文件
- [Increasing Performance - more than a pipe dream](https://youtu.be/I6ZvpdRM1eQ) - youtube 上的 ng-conf 影片。 在插值表達式裡面使用管道取代 function。

# 結論

The list of practices will dynamically evolve over time with new/updated practices. In case you notice something missing or you think that any of the practices can be improved do not hesitate to fire an issue and/or a PR. For more information please take a look at the "[Contributing](#contributing)" section below.

# 貢獻

假如你發現有缺少、不完整或不正確的內容，非常樂意看到你的 PR。關於本文未包含的實務討論，請 [open an issue](https://github.com/mgechev/angular2-performance-checklist/issues).

# 授權

MIT
