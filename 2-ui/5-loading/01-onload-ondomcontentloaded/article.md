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

تصور کنید که میخواهیم راجع به اینکه صفحه چطور استفاده شده اطلاعات جمع‌آوری کنیم: مواردی مثل کلیک‌های ماوس، اسکرول‌ها، بخش‌هایی از صفحه که مشاهده شده‌اند و مانند آن.

طبیعتا رویداد `unload` زمانی رخ میدهد که کاربر صفحه‌ی ما را ترک میکند و ما میخواهیم تا اطلاعات جمع‌آوری شده را روی سرورهای خودمون ذخیره کنیم.

یک متد خاص به نام `navigator.sendBeacon(url, data)` برای برطرف کردن چنین نیازهایی وجود دارد که در بخش <https://w3c.github.io/beacon/> توضیح داده شده است.

این متد اطلاعات را در پس‌زمینه ارسال میکند. با این متد فرآیند تغییر صفحه به تعویق نخواهد افتاد و مرورگر صفحه را ترک خواهد کرد اما همچنان `sendBeacon` را اجرا میکند.

به این شکل میتوانیم از این متد استفاده کنیم:
```js
let analyticsData = { /* object with gathered data */ };

window.addEventListener("unload", function() {
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
});
```

- ریکوئست به شکل POST ارسال میشود.
- ما میتوانیم تنها یک رشته یا دیگر فرمت‌ها را به شکلی که در فصل <info:fetch> بیان شده است ارسال کنیم اما معمولا به شکل یک آبجکت که به رشته تبدیل شده ارسال خواهد شد.
- دیتای قابل ارسال دارای محدودیت حجمی 64 کیلوبایت میباشد.

ممکن است مرورگر پیش از به پایان رسیدن درخواست `sendBeacon` صفحه را ترک کرده باشد پس راهی برای دریافت پاسخ سرور وجود ندارد (که معمولا برای اطلاعات آماری خالی است.)

همچنین یک فلگ `keepalive` برای انجام ریکوئست‌هایی مانند "after-page-left" در متد [fetch](info:fetch) وجود دارد. میتوانیم اطلاعات بیشتری در اینباره در فصل <info:fetch-api> به دست آوریم.

اگر بخواهیم تغییر صفحه را لغو کنیم نمیتوانیم آنرا اینجا انجام دهیم. اما میتوانیم از رویداد دیگری استفاده کنیم: `onbeforeunload`

## window.onbeforeunload [#window.onbeforeunload]

اگر بازدید کننده فرآیند خروج از صفحه را آغاز کند یا تلاش کند تا پنجره را ببند، هندلر رویداد `beforeunload` درخواست تایید خواهد کرد.

اگر رویداد را لغو کنیم آنگاه مرورگر از بازدیدکننده خواهد پرسید که آیا مطمئن هستند.

میتوانید با اجرای این کد و سپس بارگذاری دوباره‌ی صفحه آنرا امتحان کنید:

```js run
window.onbeforeunload = function() {
  return false;
};
```

بنابر دلایل تاریخی ارسال یک رشته غیرخالی هم به عنوان یک لغو کننده رویداد به حساب می‌آید. در گذشته مرورگرها از آن برای نمایش یک پیام استفاده میکردند اما طبق [آیین نامه مرورگرهای جدید](https://html.spec.whatwg.org/#unloading-documents) دیگر نباید اینکار انجام شود.

مثال:

```js run
window.onbeforeunload = function() {
  return "There are unsaved changes. Leave now?";
};
```

از آنجا که بعضی از وب‌مسترها از این ایونت هندلر برای نمایش پیام‌های آزاردهنده استفاده کردند این رفتار تغییر کرد. بنابراین مروگرهای قدیمی ممکن است که هنوز آنرا تحت یک پیام نمایش دهند اما اگر آنرا کنار بگذاریم راهی وجود ندارد که پیام نمایش داده شده به کاربر را شخصی سازی کنیم.

````warn header="The `event.preventDefault()` doesn't work from a `beforeunload` handler"
ممکن است عجیب به نظر برسد ولی اغلب مرورگرها `event.preventDefault()` را نادیده میگیرند.

که به معنی آن است که کد زیر ممکن است عمل نکند:
```js run
window.addEventListener("beforeunload", (event) => {
  // doesn't work, so this event handler doesn't do anything
	event.preventDefault();
});
```

به عنوان جایگزین در این هندلرها میتوان با جایگذاری `event.returnValue` به یک رشته نتیجه‌ای مشابه کد بالا بدست آورد:
```js run
window.addEventListener("beforeunload", (event) => {
  // works, same as returning from window.onbeforeunload
	event.returnValue = "There are unsaved changes. Leave now?";
});
```
````

## readyState

چه اتفاقی می‌افتد اگر ما هندلر `DOMContentLoaded` را پس از لود شدن داکیومنت ست کنیم؟

طبیعتا هرگز اجرا نخواهد شد.

در شرایطی ممکن است که ما از اینکه آیا داکیومنت آماده است یا نه اطمینان نداشته باشیم. ما میخواهیم تا فانکشن ما پس از بارگذاری DOM اجرا شود خواه حالا باشد یا بعدا.

پراپرتی `document.readyState` به ما راجع به ما راجع به وضعیت بارگذاری DOM اطلاعات میدهد.

سه مقدار ممکن است برگردد:

- `"loading"` -- داکیومنت در حال بارگذاری است..
- `"interactive"` -- داکیومنت کاملا خوانده شده است.
- `"complete"` -- داکیومنت شکل کامل خوانده شده و همه‌ی منابع (مانند تصاویر) هم بارگذاری شده‌اند.

بنابراین ما میتوانیم `document.readyState` را بررسی کرده و یک هندلر را برای آن مشخص کنیم یا اینکه کد را بلافاصله پس از اینکه آماده شد اجرا کنیم.

به اینصورت:

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

همچنین رویدادی به نام `readystatechange` هم وجود دارد  که در هنگام تغییر وضعیت اتفاق می‌افتد. بنابراین ما میتوانیم همه‌ی وضعیت‌ها را به اینصورت نمایش دهیم.

```js run
// وضعیت کنونی
console.log(document.readyState);

// چاپ تغییرات وضعیت
document.addEventListener('readystatechange', () => console.log(document.readyState));
```

رویداد `readystatechange` جایگزینی برای ردگیری وضعیت بارگذاری میباشد. این رویداد مدت‌ها پیش به وجود آمد. اینروزها خیلی به ندرت استفاده می‌شود.

بیاید تا برای جمع بندی فلوی کلی رویدادها را ببینیم.

اینجا داکیومنتی با تگ‌های `<iframe>`, `<img>` و هندلرهایی برای چاپ رویدادها داریم:
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

مثال عملی آن [در سندباکس](sandbox:readystate) وجود دارد.

خروجی معمول:
1. [1] initial readyState:loading
2. [2] readyState:interactive
3. [2] DOMContentLoaded
4. [3] iframe onload
5. [4] img onload
6. [4] readyState:complete
7. [4] window onload

شماره‌های داخل براکت نشان‌دهنده زمان تقریبی اتفاق افتادن آن هستند. رویدادهایی که با ارقام مشابه تقریبا همزمان اتفاق می‌افتند (+- میلی ثانیه).

- `document.readyState` becomes `interactive` right before `DOMContentLoaded`. These two things actually mean the same.
- `document.readyState` becomes `complete` when all resources (`iframe` and `img`) are loaded. Here we can see that it happens in about the same time as `img.onload` (`img` is the last resource) and `window.onload`. Switching to `complete` state means the same as `window.onload`. The difference is that `window.onload` always works after all other `load` handlers.


## خلاصه

رویدادهای بارگذاری صفحه:

- رویداد `DOMContentLoaded` برروی زمانی که DOM آماده باشد اجرا میشود. ما میتوانیم جاوااسکریپت را در این حالت به المان‌ها اپلای کنیم.
  - تگ‌های اسکریپت همانند `<script>...</script>` یا `<script src="..."></script>` رویداد DOMContentLoaded را بلاک میکنند و مرورگر برای اجرای آنها صبر میکند.
  - تصاویر و دیگر منابع هم ممکن است همچنان درحال لود باشند
- رویداد `load` زمانی که صفحه و همه‌ی منابع لود شده باشند اجرا میشود. ما به ندرت از این ایونت استفاده میکنیم چون معمولا نیازی به اینکه مدتی طولانی منتظر بمانیم نیست.
- رویداد `beforeunload` زمانی برروی `window` اجرا میشود که کاربر بخواهد صفحه را ترک کند. اگر ما رویداد را لغو کنیم مرورگر از یوزر میپرسد که آیا واقعا قصد خروج دارد یا نه (برای مثال زمانی که ما تغییرات ذخیره نشده داریم).
- The `unload` event on `window` triggers when the user is finally leaving, in the handler we can only do simple things that do not involve delays or asking a user. Because of that limitation, it's rarely used. We can send out a network request with `navigator.sendBeacon`.
- `document.readyState` is the current state of the document, changes can be tracked in the `readystatechange` event:
  - `loading` -- the document is loading.
  - `interactive` -- the document is parsed, happens at about the same time as `DOMContentLoaded`, but before it.
  - `complete` -- the document and resources are loaded, happens at about the same time as `window.onload`, but before it.
