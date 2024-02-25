# Page: DOMContentLoaded, load, beforeunload, unload

چرخه زندگی (lifecycle) یک صفحه HTML سه رویداد اصلی را شامل می‌شود:

- `DOMContentLoaded` -- مرورگر به صورت کامل لود شده و درخت DOM ساخته شده ولی منابع خارجی مانند تصاویر `<img>` و استایل شیت‌ها ممکن است در این مرحله هنوز لود نشده باشند.
- `load` -- نه تنها HTML لود شده بلکه همه‌ی منابع خارجی مانند تصاویر و استایل‌ها هم لود شده اند.
- `beforeunload/unload` -- یوزر صفحه را ترک میکند.

هر یک از این رویدادها میتوانند کاربردی باشند:

- رویداد `DOMContentLoaded` -- درخت DOM آماده است پس هندلر میتواند در نودهای DOM جستجو کرده و اینترفیس را بسازند.
- `load` event -- external resources are loaded, so styles are applied, image sizes are known etc.
- رویداد `load` -- منابع خارجی بارگذاری شده اند بنابراین استایلها اعمال شده و اندازه تصاویر را میتوان به دست آورد و مواردی مثل آن.
- رویداد `beforeunload` -- کاربر درحال خروج است: میتوانیم بررسی کنیم که آیا یوزر تغییرات را ذخیره کرده و از آنها بپرسیم که آیا واقعا قصد خروج از صفحه را دارند؟
- `unload` -- کاربر تقریبا از صفحه خارج شده اما ما هنوز میتوانیم بعضی از عملیات‌ها مانند فرستادن آمار به بیرون را انجام دهیم.

بیاید جزئیات این رویدادهارا بررسی کنیم.

## DOMContentLoaded

رویداد `DOMContentLoaded` برروی آبجکت `document` اتفاق می‌افتد.

ما باید از `addEventListener` برای گرفتن آن استفاده کنیم:

```js
document.addEventListener("DOMContentLoaded", ready);
// not "document.onDOMContentLoaded = ..."
```

برای مثال:

```html run height=200 refresh
<script>
  function ready() {
    alert('DOM is ready');

    // image is not yet loaded (unless it was cached), so the size is 0x0
    alert(`Image size: ${img.offsetWidth}x${img.offsetHeight}`);
  }

*!*
  document.addEventListener("DOMContentLoaded", ready);
*/!*
</script>

<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

در این مثال هندلر `DOMContentLoaded` زمانی که داکیومنت لود شده باشد اجرا میشود و بنابراین می‌تواند همه‌ی المان‌ها شامل `<img>` را ببیند.

اما این رویداد منتظر لود شدن عکس نمیشود پس `alert` اندازه صفر را نمایش می‌دهد.

در نگاه اول رویداد `DOMContentLoaded` خیلی ساده به نظر میرسد. درخت DOM آماده است. با این وجود موارد عجیبی وجود دارند.

### DOMContentLoaded and scripts

زمانی که مرورگر یک اچ‌تی‌ام‌ال داکیومنت را پردازش میکند و در حین انجام این کار به تگ `<script>` برمیخورد این تگ نیاز دارد پیش از ساختن DOM آنرا اجرا کند. اینجا باید احتیاط به خرج داد چون اسکریپت‌ها ممکن است بخواهند تا DOM را تغییر دهند و حتی `document.write` را برروی آن اعمال کنند پس `DOMContentLoaded` باید صبرکند.

بنابراین DOMContentLoaded قطعا پس از چنین اسکریپتی اتفاق می‌افتد:

```html run
<script>
  document.addEventListener("DOMContentLoaded", () => {
    alert("DOM ready!");
  });
</script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.3.0/lodash.js"></script>

<script>
  alert("Library loaded, inline script executed");
</script>
```

در مثال بالا ما ابتدا "Library loaded..." را میبینیم و پس از آن "DOM ready!" (همه‌ی اسکریپت‌ها اجرا شده‌اند).

```warn header="Scripts that don't block DOMContentLoaded"
دو استثنا برای این قانون وجود داند:
1. اسکریپت‌های دارای اتریبیوت `async` که [کمی بعد](info:script-async-defer) آن را بررسی میکنیم، رویداد `DOMContentLoaded` را بلاک نمیکنند.
2. اسکریپت‌هایی که به شکل پویا توسط `document.createElement('script')` ساخته میشوند و سپس به صفحه وب اضافه میشوند هم این رویداد را بلاک نمیکنند.
```

### DOMContentLoaded and styles

استایل شیت‌های خارجی هم روی DOM اثری نمیگذارند بنابراین `DOMContentLoaded` روی آنها تاثیری نمیگذارد.

اما اینجا نکته‌ای هست که باید به آن توجه کرد، اگر ما اسکریپتی بعد از استایل داشته باشیم آنگاه آن اسکریپت تا لود شدن استایل باید صبر کند.

```html run
<link type="text/css" rel="stylesheet" href="style.css">
<script>
  // the script doesn't execute until the stylesheet is loaded
  alert(getComputedStyle(document.body).marginTop);
</script>
```

این اتفاق به این علت می‌افتد که اسکریپت ممکن است که بخواهد مختصات و دیگر پراپرتی‌های وابسته به استایل المان را بخواند درست مانند مثال بالا. طبیعتا برای این کار باید منتظر لود شدن استایلها باقی بماند.

بنابراین از آنجا که `DOMContentLoaded` منتظر لود شدن اسکریپت‌ها میماند پس حالا باید منتظر لود شدن استایل‌ها هم بماند.

### Built-in browser autofill

مروگرهای فایرفاکس، کروم و اپرا فرم‌هارا هنگام `DOMContentLoaded` پر میکنند.

برای مثال اگر صفحه یک فرم با فیلدهای لاگین و پسورد داشته باشد و مرورگر این مقادیر را به خاطر سپرده باشد آنگاه هنگام رویداد `DOMContentLoaded` ممکن است بخواهد این فیلدهارا به صورت خودکار پر کند (اگر که توسط کاربر تایید شده باشد).

بنابراین اگر `DOMContentLoaded` توسط اسکریپت‌هایی با زمان لود بالا به تعویق افتاده باشد انگاه فرآیند پرکردن خودکار فیلدها هم به تعویق خواهد افتاد. احتمالا شما هم این رخداد را در بعضی از سایت‌ها دیده باشید (البته اگر از فرآیند پر کردن خودکار فرمها استفاده میکنید.) -- فیلدهای لاگین/پسورد بلافاصله و تا زمانی که صفحه کامل لود شود پر نخواهند شد. این تاخیر درواقع به اندازه رخ دادن `DOMContentLoaded` طول خواهد کشید.


## window.onload [#window-onload]

رویداد `load` برروی آبجکت `window` زمانی اتفاق خواهد افتاد که کل صفحه شامل استایل‌ها, تصاویر و دیگر منابع لود شده باشد. این رویداد با پراپرتی `onload` قابل دسترسی است.

مثال زیر به درستی اندازه تصاویر را نشان میدهد چون `window.onload` تا لود شدن همه‌ی تصاویر صبر کرده و بعد اجرا میشود.

```html run height=200 refresh
<script>
  window.onload = function() { // can also use window.addEventListener('load', (event) => {
    alert('Page loaded');

    // image is loaded at this time
    alert(`Image size: ${img.offsetWidth}x${img.offsetHeight}`);
  };
</script>

<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

## window.onunload

زمانی که یک بازدیدکننده صفحه را ترک میکند، رویداد `unload` بر روی `window` اجرا میشود. در این رویداد میتوانیم کارهایی که تاخیر ندارند مانند بستن پنجره‌های پاپ‌آپ مرتبط را انجام دهیم.

یک مثال قابل توجه فرستادن آمار است.

Let's say we gather data about how the page is used: mouse clicks, scrolls, viewed page areas, and so on.

Naturally, `unload` event is when the user leaves us, and we'd like to save the data on our server.

There exists a special `navigator.sendBeacon(url, data)` method for such needs, described in the specification <https://w3c.github.io/beacon/>.

It sends the data in background. The transition to another page is not delayed: the browser leaves the page, but still performs `sendBeacon`.

Here's how to use it:
```js
let analyticsData = { /* object with gathered data */ };

window.addEventListener("unload", function() {
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
});
```

- The request is sent as POST.
- We can send not only a string, but also forms and other formats, as described in the chapter <info:fetch>, but usually it's a stringified object.
- The data is limited by 64kb.

When the `sendBeacon` request is finished, the browser probably has already left the document, so there's no way to get server response (which is usually empty for analytics).

There's also a `keepalive` flag for doing such "after-page-left" requests in  [fetch](info:fetch) method for generic network requests. You can find more information in the chapter <info:fetch-api>.


If we want to cancel the transition to another page, we can't do it here. But we can use another event -- `onbeforeunload`.

## window.onbeforeunload [#window.onbeforeunload]

If a visitor initiated navigation away from the page or tries to close the window, the `beforeunload` handler asks for additional confirmation.

If we cancel the event, the browser may ask the visitor if they are sure.

You can try it by running this code and then reloading the page:

```js run
window.onbeforeunload = function() {
  return false;
};
```

For historical reasons, returning a non-empty string also counts as canceling the event. Some time ago browsers used to show it as a message, but as the [modern specification](https://html.spec.whatwg.org/#unloading-documents) says, they shouldn't.

Here's an example:

```js run
window.onbeforeunload = function() {
  return "There are unsaved changes. Leave now?";
};
```

The behavior was changed, because some webmasters abused this event handler by showing misleading and annoying messages. So right now old browsers still may show it as a message, but aside of that -- there's no way to customize the message shown to the user.

````warn header="The `event.preventDefault()` doesn't work from a `beforeunload` handler"
That may sound weird, but most browsers ignore `event.preventDefault()`.

Which means, following code may not work:
```js run
window.addEventListener("beforeunload", (event) => {
  // doesn't work, so this event handler doesn't do anything
	event.preventDefault();
});
```

Instead, in such handlers one should set `event.returnValue` to a string to get the result similar to the code above:
```js run
window.addEventListener("beforeunload", (event) => {
  // works, same as returning from window.onbeforeunload
	event.returnValue = "There are unsaved changes. Leave now?";
});
```
````

## readyState

What happens if we set the `DOMContentLoaded` handler after the document is loaded?

Naturally, it never runs.

There are cases when we are not sure whether the document is ready or not. We'd like our function to execute when the DOM is loaded, be it now or later.

The `document.readyState` property tells us about the current loading state.

There are 3 possible values:

- `"loading"` -- the document is loading.
- `"interactive"` -- the document was fully read.
- `"complete"` -- the document was fully read and all resources (like images) are loaded too.

So we can check `document.readyState` and setup a handler or execute the code immediately if it's ready.

Like this:

```js
function work() { /*...*/ }

if (document.readyState == 'loading') {
  // still loading, wait for the event
  document.addEventListener('DOMContentLoaded', work);
} else {
  // DOM is ready!
  work();
}
```

There's also the `readystatechange` event that triggers when the state changes, so we can print all these states like this:

```js run
// current state
console.log(document.readyState);

// print state changes
document.addEventListener('readystatechange', () => console.log(document.readyState));
```

The `readystatechange` event is an alternative mechanics of tracking the document loading state, it appeared long ago. Nowadays, it is rarely used.

Let's see the full events flow for the completeness.

Here's a document with `<iframe>`, `<img>` and handlers that log events:

```html
<script>
  log('initial readyState:' + document.readyState);

  document.addEventListener('readystatechange', () => log('readyState:' + document.readyState));
  document.addEventListener('DOMContentLoaded', () => log('DOMContentLoaded'));

  window.onload = () => log('window onload');
</script>

<iframe src="iframe.html" onload="log('iframe onload')"></iframe>

<img src="https://en.js.cx/clipart/train.gif" id="img">
<script>
  img.onload = () => log('img onload');
</script>
```

The working example is [in the sandbox](sandbox:readystate).

The typical output:
1. [1] initial readyState:loading
2. [2] readyState:interactive
3. [2] DOMContentLoaded
4. [3] iframe onload
5. [4] img onload
6. [4] readyState:complete
7. [4] window onload

The numbers in square brackets denote the approximate time of when it happens. Events labeled with the same digit happen approximately at the same time (+- a few ms).

- `document.readyState` becomes `interactive` right before `DOMContentLoaded`. These two things actually mean the same.
- `document.readyState` becomes `complete` when all resources (`iframe` and `img`) are loaded. Here we can see that it happens in about the same time as `img.onload` (`img` is the last resource) and `window.onload`. Switching to `complete` state means the same as `window.onload`. The difference is that `window.onload` always works after all other `load` handlers.


## Summary

Page load events:

- The `DOMContentLoaded` event triggers on `document` when the DOM is ready. We can apply JavaScript to elements at this stage.
  - Script such as `<script>...</script>` or `<script src="..."></script>` block DOMContentLoaded, the browser waits for them to execute.
  - Images and other resources may also still continue loading.
- The `load` event on `window` triggers when the page and all resources are loaded. We rarely use it, because there's usually no need to wait for so long.
- The `beforeunload` event on `window` triggers when the user wants to leave the page. If we cancel the event, browser asks whether the user really wants to leave (e.g we have unsaved changes).
- The `unload` event on `window` triggers when the user is finally leaving, in the handler we can only do simple things that do not involve delays or asking a user. Because of that limitation, it's rarely used. We can send out a network request with `navigator.sendBeacon`.
- `document.readyState` is the current state of the document, changes can be tracked in the `readystatechange` event:
  - `loading` -- the document is loading.
  - `interactive` -- the document is parsed, happens at about the same time as `DOMContentLoaded`, but before it.
  - `complete` -- the document and resources are loaded, happens at about the same time as `window.onload`, but before it.
