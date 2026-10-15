<div dir="rtl" align="right">


# 📚 مصاحبه Full Stack Developer شرکت فناوران اطلاعات خبره
## ASP.NET Core + Angular + DDD + Microservices + RabbitMQ

---

> ### در اینجا سوالات مصاحبه در تاریخ 140506  آمده البته با توضیحات و سوالات تکمیلی تر بنده به همراه پاسخ های صحیح که chatgpt جواب داده 


---

# بخش اول: Angular و مدیریت وضعیت

## ۱. State Management چیست؟

### ❓ سؤال
State Management در Angular چیست و چه زمانی به آن نیاز داریم؟

### ✅ پاسخ
State Management یعنی مدیریت داده‌هایی که وضعیت فعلی برنامه را مشخص می‌کنند.

مثلاً در یک فروشگاه اینترنتی:

- اطلاعات کاربر واردشده
- سبد خرید
- فهرست محصولات
- فیلترهای جست‌وجو
- اطلاعات سفارش

اگر وضعیت فقط در یک کامپوننت استفاده شود، معمولاً متغیرهای محلی کافی هستند. اگر چند کامپوننت به دادهٔ مشترک نیاز داشته باشند، بهتر است از یک روش اشتراکی استفاده کنیم.

### روش‌های رایج

| روش | کاربرد |
|---|---|
| Component State | وضعیت یک کامپوننت |
| Angular Signals | وضعیت واکنش‌پذیر و محاسبات وابسته |
| Service + RxJS | اشتراک وضعیت میان کامپوننت‌ها |
| ComponentStore | مدیریت وضعیت محدود به یک بخش |
| NgRx Store | مدیریت وضعیت پیچیده و سراسری |
| NgRx Signal Store | مدیریت وضعیت با رویکرد Signals |

### مثال: Signal

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>تعداد: {{ count() }}</p>
    <p>دو برابر: {{ doubled() }}</p>
    <button (click)="increase()">افزایش</button>
  `
})
export class CounterComponent {
  count = signal(0);

  doubled = computed(() => this.count() * 2);

  increase() {
    this.count.update(value => value + 1);
  }
}
```

**نکتهٔ مصاحبه:** برای هر پروژه‌ای لازم نیست NgRx نصب کنیم. پیچیدگی واقعی وضعیت برنامه تعیین می‌کند که چه ابزاری مناسب‌تر است.

---

## ۲. تفاوت Signal و RxJS چیست؟

### ❓ سؤال
Signal چه تفاوتی با Observable دارد؟

### ✅ پاسخ

| ویژگی | Signal | Observable |
|---|---|---|
| مدل اصلی | مقدار فعلی واکنش‌پذیر | جریان داده در طول زمان |
| خواندن مقدار | با `count()` | با Subscribe یا تبدیل مناسب |
| محاسبهٔ وابسته | `computed()` | اپراتورهایی مانند `map` |
| لغو عملیات | به‌صورت ذاتی ابزار لغو درخواست نیست | امکان لغو اشتراک وجود دارد |
| کاربرد مهم | وضعیت UI | HTTP، رویدادها و جریان‌های زمانی |

مثال Signal:

```typescript
count = signal(0);
doubleCount = computed(() => this.count() * 2);
```

مثال RxJS:

```typescript
this.http.get<Product[]>('/api/products')
  .subscribe(products => {
    console.log(products);
  });
```

**قاعدهٔ کاربردی:** برای وضعیت فعلی رابط کاربری، Signal گزینهٔ مناسبی است؛ برای جست‌وجوی لحظه‌ای، ترکیب چند درخواست و مدیریت جریان‌های زمانی، RxJS ابزار قدرتمندی است.

---

## ۳. Dependency Injection در Angular چیست؟

### ❓ سؤال
DI چیست و چرا از آن استفاده می‌کنیم؟

### ✅ پاسخ
Dependency Injection یعنی یک کلاس وابستگی‌های موردنیازش را دریافت کند، به‌جای اینکه خودش آن‌ها را بسازد.

روش نامناسب:

```typescript
export class ProductComponent {
  service = new ProductService();
}
```

مشکل این است که کامپوننت مستقیماً به نحوهٔ ساخت سرویس وابسته می‌شود.

روش بهتر:

```typescript
import { Component, inject } from '@angular/core';
import { ProductService } from './product.service';

@Component({
  selector: 'app-products',
  standalone: true,
  template: `محصولات`
})
export class ProductComponent {
  private productService = inject(ProductService);
}
```

### مزایا

- کاهش وابستگی مستقیم
- تست‌پذیری بیشتر
- امکان جایگزینی پیاده‌سازی
- مدیریت چرخهٔ عمر سرویس‌ها

---

## ۴. تفاوت providedIn: root و providers کامپوننت چیست؟

### ❓ سؤال
اگر سرویس را در `root` ثبت کنیم چه می‌شود؟

### ✅ پاسخ

```typescript
@Injectable({
  providedIn: 'root'
})
export class AuthService {}
```

در حالت معمول، سرویس در سطح برنامه در دسترس است و یک نمونهٔ مشترک در اختیار مصرف‌کنندگان همان ثبت قرار می‌گیرد.

اگر سرویس را در `providers` یک کامپوننت ثبت کنیم، آن کامپوننت و محدودهٔ فرزندانش می‌توانند نمونهٔ جداگانه‌ای دریافت کنند.

```typescript
@Component({
  selector: 'app-profile',
  providers: [ProfileStateService],
  template: `پروفایل`
})
export class ProfileComponent {}
```

**مثال واقعی:** اطلاعات ورود کاربر معمولاً در یک سرویس سراسری نگهداری می‌شود؛ اما وضعیت یک فرم یا صفحه می‌تواند محلی باشد.

---

## ۵. تفاوت useClass، useValue، useFactory و useExisting چیست؟

### ❓ سؤال
در Angular DI چه زمانی از هرکدام استفاده می‌کنیم؟

### ✅ پاسخ

**`useClass` — انتخاب یک کلاس برای پیاده‌سازی**

```typescript
{
  provide: PaymentService,
  useClass: OnlinePaymentService
}
```

**`useValue` — تزریق یک مقدار ثابت**

```typescript
{
  provide: 'API_URL',
  useValue: 'https://api.example.com'
}
```

**`useFactory` — ساخت وابستگی با یک تابع**

```typescript
{
  provide: 'APP_CONFIG',
  useFactory: () => ({
    language: 'fa',
    pageSize: 20
  })
}
```

**`useExisting` — استفاده از ثبت موجود**

```typescript
{
  provide: AbstractLogger,
  useExisting: ConsoleLogger
}
```

در حالت آخر، هر دو توکن به همان نمونهٔ ثبت‌شدهٔ `ConsoleLogger` اشاره می‌کنند؛ نه اینکه لزوماً دو نمونه بسازند.

---

# بخش دوم: RxJS و اپراتورها

## ۶. تفاوت map، tap و switchMap چیست؟

### ❓ سؤال
هرکدام چه کاری انجام می‌دهند؟

### ✅ پاسخ

**`map`** خروجی را تبدیل می‌کند.

```typescript
of(2, 4, 6).pipe(
  map(x => x * 2)
);
```

خروجی: `4, 8, 12`

**`tap`** برای کارهای جانبی است و مقدار جریان را تغییر نمی‌دهد.

```typescript
this.http.get<Product[]>('/api/products').pipe(
  tap(products => console.log(products))
);
```

**`switchMap`** جریان قبلی را کنار می‌گذارد و به Observable جدید سوئیچ می‌کند.

```typescript
searchTerms$.pipe(
  switchMap(term =>
    this.http.get<Product[]>(
      `/api/products?search=${term}`
    )
  )
);
```

این روش برای جست‌وجوی لحظه‌ای مناسب است؛ چون پاسخ جست‌وجوی قدیمی نباید جای نتیجهٔ جست‌وجوی جدید را بگیرد.

**نکته:** لغو اشتراک، تضمین نمی‌کند که سرور حتماً عملیات پردازش را متوقف کرده باشد.

---

## ۷. تفاوت switchMap، mergeMap، concatMap و exhaustMap چیست؟

### ❓ سؤال
این چهار اپراتور چه تفاوتی دارند؟

### ✅ پاسخ

| اپراتور | رفتار | سناریوی واقعی |
|---|---|---|
| `switchMap` | کنار گذاشتن جریان قبلی هنگام شروع جریان جدید | جست‌وجوی زنده |
| `mergeMap` | اجرای هم‌زمان چند جریان داخلی | دریافت جزئیات چند آیتم |
| `concatMap` | اجرای ترتیبی و صف‌شده | ذخیرهٔ عملیات به ترتیب |
| `exhaustMap` | نادیده گرفتن ورودی جدید تا پایان عملیات جاری | جلوگیری از چند ارسال هم‌زمان فرم |

مثال `concatMap`:

```typescript
from(orderIds).pipe(
  concatMap(id =>
    this.http.post('/api/orders/process', { id })
  )
);
```

هر درخواست بعد از تکمیل درخواست قبلی شروع می‌شود.

مثال `exhaustMap`:

```typescript
submitClicks$.pipe(
  exhaustMap(() =>
    this.http.post('/api/orders', formValue)
  )
);
```

اگر درخواست قبلی هنوز تمام نشده باشد، کلیک‌های جدید نادیده گرفته می‌شوند.

**نکتهٔ مصاحبه:** این اپراتورها جای یکدیگر نیستند؛ انتخاب آن‌ها به نیاز کسب‌وکار بستگی دارد.

---

## ۸. تفاوت debounceTime و delay چیست؟

### ❓ سؤال
چرا برای جست‌وجو از debounceTime استفاده می‌کنیم؟

### ✅ پاسخ
`debounceTime` پس از آخرین رویداد، تا مدت مشخصی صبر می‌کند و اگر رویداد جدیدی نرسید، مقدار را منتشر می‌کند.

```typescript
searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term =>
    this.http.get('/api/products', {
      params: { search: term ?? '' }
    })
  )
);
```

در این مثال، کاربر می‌تواند تایپ کند و درخواست پس از ۳۰۰ میلی‌ثانیه توقف ارسال شود.

اما `delay(300)` داده را با تأخیر منتشر می‌کند؛ رویدادهای ورودی را مانند `debounceTime` تجمیع نمی‌کند.

---

## ۹. تفاوت some، every و filter در JavaScript چیست؟

### ❓ سؤال
چه زمانی از این متدها استفاده می‌کنیم؟

### ✅ پاسخ

```typescript
const prices = [100, 200, 300];
```

**some:** آیا حداقل یک عنصر شرط را دارد؟

```typescript
prices.some(price => price > 250); // true
```

**every:** آیا همهٔ عناصر شرط را دارند؟

```typescript
prices.every(price => price > 50); // true
```

**filter:** تمام عناصر منطبق را برمی‌گرداند.

```typescript
prices.filter(price => price > 150);
// [200, 300]
```

| متد | خروجی |
|---|---|
| `some` | Boolean |
| `every` | Boolean |
| `filter` | آرایه |

---

# بخش سوم: ASP.NET Core و Dependency Injection

## ۱۰. تفاوت Transient، Scoped و Singleton چیست؟

### ❓ سؤال
طول عمر سرویس‌ها در ASP.NET Core چگونه است؟

### ✅ پاسخ

| طول عمر | رفتار | کاربرد |
|---|---|---|
| Transient | هر بار درخواست وابستگی، نمونهٔ جدید | سرویس سبک و بدون وضعیت مشترک |
| Scoped | یک نمونه در هر Scope؛ در وب معمولاً هر درخواست HTTP | سرویس‌های وابسته به درخواست |
| Singleton | یک نمونه در طول عمر Container | کش یا سرویس مشترک امن برای استفادهٔ هم‌زمان |

مثال:

```csharp
builder.Services.AddTransient<IEmailFormatter, EmailFormatter>();

builder.Services.AddScoped<IOrderService, OrderService>();

builder.Services.AddSingleton<ISystemClock, SystemClock>();
```

**نکته:** Singleton الزاماً Thread-Safe نیست. اگر چند درخواست هم‌زمان از آن استفاده کنند، وضعیت داخلی قابل‌تغییر آن باید ایمن باشد.

---

## ۱۱. مشکل Captive Dependency چیست؟

### ❓ سؤال
چرا نباید سرویس Scoped را مستقیماً داخل Singleton تزریق کنیم؟

### ✅ پاسخ
فرض کنید یک سرویس Singleton به سرویسی Scoped وابسته باشد. وابستگی Scoped ممکن است عملاً بیشتر از محدودهٔ عمر مورد انتظارش نگه داشته شود.

مثال نامناسب:

```csharp
public class ReportCache
{
    private readonly AppDbContext _db;

    public ReportCache(AppDbContext db)
    {
        _db = db;
    }
}
```

اگر `ReportCache` به‌صورت Singleton ثبت شود، نگهداری `DbContext` در آن اشتباه است.

راه‌حل به نیاز بستگی دارد؛ مثلاً برای ساخت Scope مستقل می‌توان از `IServiceScopeFactory` استفاده کرد، یا برای ایجاد Context مستقل از `IDbContextFactory<TContext>` کمک گرفت.

---

## ۱۲. چرا DbContext را Singleton نمی‌کنیم؟

### ❓ سؤال
چرا `DbContext` معمولاً Scoped است؟

### ✅ پاسخ

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

`DbContext` برای استفادهٔ هم‌زمان چند عملیات طراحی نشده است و وضعیت داخلی مربوط به تغییرات و Tracking را نگه می‌دارد.

اگر یک نمونه بین درخواست‌های هم‌زمان به اشتراک گذاشته شود، احتمال خطا، تداخل تغییرات و مشکلات امنیتی داده وجود دارد.

### اگر دو عملیات موازی لازم داشته باشیم چه کنیم؟

```csharp
builder.Services.AddDbContextFactory<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

سپس برای هر عملیات، Context مستقل بسازیم:

```csharp
public class ProductReader
{
    private readonly IDbContextFactory<AppDbContext> _factory;

    public ProductReader(
        IDbContextFactory<AppDbContext> factory)
    {
        _factory = factory;
    }

    public async Task<int> CountAsync()
    {
        await using var db =
            await _factory.CreateDbContextAsync();

        return await db.Products.CountAsync();
    }
}
```

---

## ۱۳. تفاوت IoC، DI و IoC Container چیست؟

### ❓ سؤال
این مفاهیم چه ارتباطی دارند؟

### ✅ پاسخ

- **IoC:** اصل کلی واگذاری کنترل ساخت یا مدیریت وابستگی‌ها به سازوکاری بیرونی.
- **DI:** یکی از روش‌های پیاده‌سازی IoC؛ وابستگی از بیرون به کلاس داده می‌شود.
- **IoC Container:** ابزاری که ثبت، ساخت و مدیریت عمر وابستگی‌ها را انجام می‌دهد.

مثال:

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }
}
```

کلاس `OrderService` خودش Repository را نمی‌سازد؛ آن را از بیرون دریافت می‌کند.

---

## ۱۴. Autofac چیست؟

### ❓ سؤال
چرا ممکن است در یک پروژه از Autofac استفاده کنیم؟

### ✅ پاسخ
Autofac یک IoC Container برای .NET است که امکاناتی مانند ثبت وابستگی‌ها، مدیریت Lifetime و ماژول‌بندی ثبت‌ها را فراهم می‌کند.

برای مثال، می‌توان ثبت وابستگی‌ها را در Moduleهای جدا قرار داد. بااین‌حال، DI داخلی ASP.NET Core برای بسیاری از پروژه‌ها کافی است و استفاده از Autofac الزام معماری نیست.

---

# بخش چهارم: C# و مباحث عمیق

## ۱۵. تفاوت Class و Struct چیست؟

### ❓ سؤال
تفاوت اصلی Class و Struct چیست؟

### ✅ پاسخ

| ویژگی | Class | Struct |
|---|---|---|
| نوع | Reference Type | Value Type |
| انتساب | معمولاً کپی مرجع | کپی مقدار |
| وراثت از کلاس | دارد | ندارد |
| کاربرد متداول | اشیای دارای هویت و رفتار | مقادیر کوچک و مستقل |

مثال:

```csharp
public class PersonClass
{
    public int Age { get; set; }
}

public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
}
```

وقتی یک متغیر از نوع کلاس را به متغیر دیگری نسبت می‌دهیم، معمولاً هر دو به یک شیء اشاره می‌کنند. اما با کپی کردن یک Struct، مقدار آن کپی می‌شود.

**نکته:** Value Type بودن Struct به معنی قرار گرفتن همیشگی آن روی Stack نیست؛ محل ذخیره‌سازی به زمینهٔ استفاده بستگی دارد.

---

## ۱۶. تفاوت init و readonly چیست؟

### ❓ سؤال
هرکدام چه زمانی استفاده می‌شوند؟

### ✅ پاسخ

**`init`** اجازه می‌دهد ویژگی در زمان مقداردهی اولیه تنظیم شود، اما بعد از آن از طریق Setter عادی قابل تغییر نباشد.

```csharp
public class User
{
    public string Name { get; init; } = "";
}

var user = new User { Name = "Ali" };
```

**`readonly`** برای فیلد استفاده می‌شود و اجازه می‌دهد مقدار آن در اعلان یا Constructor تنظیم شود.

```csharp
public class UserService
{
    private readonly string _connectionName;

    public UserService(string connectionName)
    {
        _connectionName = connectionName;
    }
}
```

**خلاصه:** `init` برای ویژگی‌هایی مناسب است که هنگام ساخت شیء تنظیم می‌شوند؛ `readonly` محدودیت تغییر فیلد را اعمال می‌کند.

---

## ۱۷. تفاوت IEnumerable، IQueryable، IList و ICollection چیست؟

### ❓ سؤال
چه زمانی از هرکدام استفاده کنیم؟

### ✅ پاسخ

| نوع | کاربرد |
|---|---|
| `IEnumerable<T>` | پیمایش عناصر؛ مناسب عملیات سمت برنامه |
| `IQueryable<T>` | ساخت Query قابل ترجمه توسط Provider، مانند EF Core |
| `ICollection<T>` | مجموعه‌ای با امکاناتی مانند Count، Add و Remove |
| `IList<T>` | مجموعه‌ای با دسترسی اندیسی و عملیات مرتبط |

مثال EF Core:

```csharp
IQueryable<Product> query =
    db.Products.Where(p => p.Price > 100);

var products = await query
    .OrderBy(p => p.Name)
    .ToListAsync();
```

فیلتر و مرتب‌سازی پیش از `ToListAsync` معمولاً به SQL ترجمه می‌شوند.

اگر ابتدا داده‌ها را به حافظه منتقل کنیم:

```csharp
var products = await db.Products.ToListAsync();

var result = products
    .Where(p => p.Price > 100);
```

فیلتر دوم در حافظه اجرا می‌شود.

**قاعده:** برای Query پایگاه داده، تا وقتی نیاز است از `IQueryable` استفاده کنید؛ در مرزهایی که می‌خواهید دادهٔ نهایی را به مصرف‌کننده بدهید، خروجی مادی‌شده یا `IEnumerable` مناسب‌تر است.

---

## ۱۸. تفاوت Task و Thread چیست؟

### ❓ سؤال
آیا هر Task یک Thread جداگانه دارد؟

### ✅ پاسخ
خیر.

- `Thread` نمایندهٔ یک Thread اجرایی است.
- `Task` نمایندهٔ یک عملیات ناهمگام یا کاری است که ممکن است روی Thread اجرا شود.
- عملیات I/O ناهمگام معمولاً در زمان انتظار به Thread اختصاصی نیاز ندارد.

مثال:

```csharp
public async Task<Product?> GetProductAsync(int id)
{
    return await _db.Products
        .FirstOrDefaultAsync(p => p.Id == id);
}
```

در زمان انتظار برای پاسخ پایگاه داده، Thread درخواست می‌تواند آزاد شود تا کار دیگری انجام دهد.

**نکته:** `async` به‌خودی‌خود CPU-bound را سریع‌تر نمی‌کند.

---

## ۱۹. تفاوت I/O-bound و CPU-bound چیست؟

### ❓ سؤال
برای هرکدام از چه روشی استفاده می‌کنیم؟

### ✅ پاسخ

**I/O-bound:** بیشتر زمان صرف انتظار برای شبکه، فایل یا دیتابیس می‌شود.

مثال: دریافت اطلاعات از API.

راهکار: استفاده از APIهای Async مانند `HttpClient.GetAsync`.

**CPU-bound:** بیشتر زمان صرف محاسبات پردازنده می‌شود.

مثال: پردازش سنگین تصویر یا محاسبات ریاضی.

راهکار: در صورت نیاز به اجرای موازی یا آزاد نگه‌داشتن Thread درخواست، ممکن است از Background Worker یا `Task.Run` در جای مناسب استفاده کنیم.

در ASP.NET Core معمولاً نباید صرفاً برای یک فراخوانی I/O، آن را داخل `Task.Run` قرار دهیم.

---

## ۲۰. تفاوت lock و SemaphoreSlim چیست؟

### ❓ سؤال
برای کنترل دسترسی هم‌زمان از کدام استفاده کنیم؟

### ✅ پاسخ

`lock` دسترسی هم‌زمان به یک بخش کد را بین Threadهای همان فرایند کنترل می‌کند.

```csharp
private readonly object _lock = new();

public void Update()
{
    lock (_lock)
    {
        // تغییر وضعیت مشترک
    }
}
```

اما نمی‌توان `await` را مستقیماً داخل بدنهٔ `lock` معمولی قرار داد.

برای کنترل هم‌زمانی در عملیات Async می‌توان از `SemaphoreSlim` استفاده کرد:

```csharp
private readonly SemaphoreSlim _semaphore = new(1, 1);

public async Task UpdateAsync()
{
    await _semaphore.WaitAsync();

    try
    {
        await SaveAsync();
    }
    finally
    {
        _semaphore.Release();
    }
}
```

**محدودیت مهم:** `lock` و `SemaphoreSlim` معمولی فقط داخل همان فرایند اثر دارند. برای هماهنگی چند سرور باید از سازوکارهایی مانند قفل توزیع‌شده، تراکنش دیتابیس یا کنترل هم‌زمانی خوش‌بینانه استفاده کرد.

---

## ۲۱. Race Condition چیست؟

### ❓ سؤال
Race Condition را با مثال توضیح دهید.

### ✅ پاسخ
وقتی نتیجه به ترتیب اجرای عملیات هم‌زمان وابسته شود، ممکن است Race Condition رخ دهد.

فرض کنید موجودی کالا ۱ عدد است و دو درخواست هم‌زمان آن را می‌خوانند:

1. درخواست A موجودی ۱ را می‌خواند.
2. درخواست B هم موجودی ۱ را می‌خواند.
3. هر دو تصمیم به فروش می‌گیرند.
4. موجودی ممکن است منفی شود یا دو سفارش برای یک واحد ثبت شود.

این مشکل فقط با کنترل ساده در حافظهٔ یک سرور حل نمی‌شود؛ چون ممکن است چند سرور یا چند فرایند درگیر باشند.

---

## ۲۲. چرا برای موجودی انبار نباید همه‌چیز را با یک Lock کنترل کنیم؟

### ❓ سؤال
اگر موجودی ماشین لباس‌شویی را قفل کنیم، چرا موجودی یخچال هم باید منتظر بماند؟

### ✅ پاسخ
اگر یک قفل سراسری برای تمام محصولات داشته باشیم، عملیات مستقل نیز پشت سر هم اجرا می‌شوند.

این روش باعث کاهش هم‌زمانی و افزایش زمان انتظار می‌شود.

راهکارهای بهتر:

- قفل مجزا برای هر محصول، در صورت مناسب بودن مدل اجرا
- تراکنش دیتابیس
- شرط اتمیک هنگام کاهش موجودی
- کنترل هم‌زمانی خوش‌بینانه

مثال SQL:

```sql
UPDATE Products
SET Stock = Stock - 1
WHERE Id = @ProductId
  AND Stock > 0;
```

سپس تعداد ردیف‌های تغییرکرده را بررسی می‌کنیم:

- اگر یک ردیف تغییر کرد، موجودی با موفقیت کم شده است.
- اگر صفر ردیف تغییر کرد، محصول وجود ندارد یا موجودی کافی نیست.

**نکته:** این الگو باید در تراکنش و منطق ثبت سفارش متناسب با نیاز سیستم استفاده شود.

---

## ۲۳. Collectionهای Thread-Safe چیستند؟

### ❓ سؤال
چه زمانی از ConcurrentDictionary یا Channel استفاده می‌کنیم؟

### ✅ پاسخ

| ابزار | کاربرد |
|---|---|
| `ConcurrentDictionary<TKey,TValue>` | Dictionary با عملیات ایمن برای استفادهٔ هم‌زمان |
| `ConcurrentQueue<T>` | صف هم‌زمان |
| `ConcurrentStack<T>` | پشتهٔ هم‌زمان |
| `ConcurrentBag<T>` | مجموعهٔ بدون ترتیب مشخص با تمرکز بر دسترسی هم‌زمان |
| `BlockingCollection<T>` | الگوی Producer/Consumer با امکان انتظار |
| `Channel<T>` | انتقال ناهمگام داده میان تولیدکننده و مصرف‌کننده |

مثال:

```csharp
private readonly ConcurrentDictionary<int, string> _cache = new();

public void AddOrUpdate(int id, string name)
{
    _cache.AddOrUpdate(id, name, (_, _) => name);
}
```

**نکته:** Thread-Safe بودن یک Collection لزوماً به معنی اتمیک بودن چند عملیات ترکیبی کسب‌وکار نیست.

---

## ۲۴. تفاوت decimal و double چیست؟

### ❓ سؤال
چرا برای محاسبات مالی از decimal استفاده می‌کنیم؟

### ✅ پاسخ

```csharp
double d1 = 0.1;
double d2 = 0.2;

Console.WriteLine(d1 + d2 == 0.3);

decimal m1 = 0.1m;
decimal m2 = 0.2m;

Console.WriteLine(m1 + m2 == 0.3m);
```

`double` برای نمایش بسیاری از اعداد اعشاری از تقریب دودویی استفاده می‌کند؛ بنابراین بعضی محاسبات ممکن است دقیقاً برابر مقدار مورد انتظار نشوند.

`decimal` برای مقادیر اعشاری مالی معمولاً مناسب‌تر است.

**نکته:** انتخاب نوع عددی، سیاست گرد کردن و تعداد رقم اعشار باید با قواعد کسب‌وکار هماهنگ باشد.

---

# بخش پنجم: EF Core و SQL Server

## ۲۵. مشکل N+1 در EF Core چیست؟

### ❓ سؤال
N+1 Query Problem را توضیح دهید.

### ✅ پاسخ
فرض کنید ابتدا سفارش‌ها را می‌گیریم و سپس برای هر سفارش، مشتری را جداگانه از دیتابیس می‌خوانیم.

در این حالت ممکن است یک Query برای سفارش‌ها و N Query دیگر برای مشتریان اجرا شود.

```csharp
var orders = await db.Orders.ToListAsync();

foreach (var order in orders)
{
    var customer = await db.Customers
        .FirstAsync(c => c.Id == order.CustomerId);
}
```

راهکار به نیاز بستگی دارد؛ برای مثال:

```csharp
var orders = await db.Orders
    .Select(o => new OrderDto
    {
        Id = o.Id,
        CustomerName = o.Customer.Name
    })
    .ToListAsync();
```

در این مثال، EF Core می‌تواند دادهٔ موردنیاز را در یک Query مناسب دریافت کند.

**نکته:** صرف استفاده از `Include` همیشه بهترین انتخاب نیست؛ برای نمایش چند فیلد، Projection معمولاً انتخاب کم‌هزینه‌تری است.

---

## ۲۶. Include در EF Core چیست؟

### ❓ سؤال
آیا Include همیشه LEFT JOIN می‌سازد؟

### ✅ پاسخ
`Include` برای بارگذاری Navigation Propertyها استفاده می‌شود. SQL نهایی به نوع رابطه، Provider و شکل Query بستگی دارد.

```csharp
var orders = await db.Orders
    .Include(o => o.Customer)
    .ToListAsync();
```

برای رابطهٔ اختیاری، ممکن است `LEFT JOIN` تولید شود؛ اما نباید فرض کنیم همهٔ Includeها همیشه دقیقاً یک شکل SQL دارند.

برای گزارش‌ها، Projection معمولاً مناسب‌تر است:

```csharp
var result = await db.Orders
    .Select(o => new
    {
        o.Id,
        CustomerName = o.Customer.Name
    })
    .ToListAsync();
```

---

## ۲۷. تفاوت NOT EXISTS، LEFT JOIN و NOT IN چیست؟

### ❓ سؤال
برای پیدا کردن مشتریانی که سفارش ندارند، کدام روش مناسب است؟

### ✅ پاسخ

**روش NOT EXISTS:**

```sql
SELECT c.*
FROM Customers c
WHERE NOT EXISTS
(
    SELECT 1
    FROM Orders o
    WHERE o.CustomerId = c.Id
);
```

**روش LEFT JOIN:**

```sql
SELECT c.*
FROM Customers c
LEFT JOIN Orders o
    ON o.CustomerId = c.Id
WHERE o.Id IS NULL;
```

هر دو می‌توانند این مسئله را حل کنند. انتخاب نهایی به خوانایی، ساختار Query و Execution Plan بستگی دارد.

**نکته درباره NOT IN:** اگر زیرپرس‌وجو `NULL` برگرداند، رفتار سه‌ارزشی SQL می‌تواند نتیجه را غیرمنتظره کند. در این سناریو، `NOT EXISTS` معمولاً انتخاب مطمئن‌تری است.

---

## ۲۸. Compiled Query چیست؟

### ❓ سؤال
چه زمانی از EF.CompileQuery استفاده می‌کنیم؟

### ✅ پاسخ
EF Core معمولاً Queryها را Cache می‌کند، اما در مسیرهای پرتکرار و پس از اندازه‌گیری می‌توان از Compiled Query برای کاهش بخشی از سربار پردازش Query استفاده کرد.

```csharp
private static readonly
    Func<AppDbContext, int, Product?> GetProduct =
        EF.CompileQuery(
            (AppDbContext db, int id) =>
                db.Products.FirstOrDefault(p => p.Id == id)
        );
```

استفاده:

```csharp
var product = GetProduct(db, productId);
```

**نکته:** Compiled Query تضمین افزایش محسوس سرعت در تمام پروژه‌ها نیست؛ ابتدا باید با Profiling و Benchmark گلوگاه را مشخص کرد.

---

# بخش ششم: معماری نرم‌افزار، DDD و Clean Architecture

## ۲۹. DDD چیست؟

### ❓ سؤال
Domain-Driven Design چه مشکلی را حل می‌کند؟

### ✅ پاسخ
DDD رویکردی برای طراحی نرم‌افزار بر اساس مفاهیم و قواعد واقعی کسب‌وکار است.

مثلاً در سامانهٔ فروش موبایل:

- محصول و موجودی
- سبد خرید
- سفارش
- تخصیص کالا
- تأیید مدیر
- فاکتور فروش

به‌جای اینکه منطق این فرایندها فقط در Controller یا چند سرویس عمومی پراکنده شود، قواعد اصلی در مدل دامنه و اجزای مناسب معماری قرار می‌گیرند.

---

## ۳۰. Aggregate و Aggregate Root چیست؟

### ❓ سؤال
Aggregate چه تفاوتی با Entity دارد؟

### ✅ پاسخ
Aggregate مجموعه‌ای از اشیای دامنه است که تحت یک مرز سازگاری و قواعد مشخص مدیریت می‌شوند.

Aggregate Root نقطهٔ ورود اصلی برای تغییر وضعیت آن مجموعه است.

مثال: `Order` می‌تواند Aggregate Root باشد و `OrderItem`ها داخل آن مدیریت شوند.

```csharp
public class Order
{
    private readonly List<OrderItem> _items = [];

    public IReadOnlyCollection<OrderItem> Items => _items;

    public void AddItem(int productId, int quantity)
    {
        if (quantity <= 0)
            throw new ArgumentOutOfRangeException(nameof(quantity));

        var item = _items.FirstOrDefault(
            x => x.ProductId == productId);

        if (item is null)
            _items.Add(new OrderItem(productId, quantity));
        else
            item.IncreaseQuantity(quantity);
    }
}
```

در این مدل، تغییر اقلام از طریق رفتارهای `Order` انجام می‌شود تا قواعد سفارش قابل‌کنترل باشند.

**نکته:** Aggregate نباید آن‌قدر بزرگ شود که تمام تغییرات سیستم را به یک مرز سازگاری وابسته کند.

---

## ۳۱. تفاوت Entity و Value Object چیست؟

### ❓ سؤال
آدرس یا مبلغ پول باید Entity باشد یا Value Object؟

### ✅ پاسخ

**Entity** هویت دارد؛ حتی اگر ویژگی‌هایش تغییر کنند، همچنان همان موجودیت است.

مثال: مشتری با `CustomerId`.

**Value Object** معمولاً هویت مستقل ندارد و با مقدار ویژگی‌هایش شناخته می‌شود.

مثال: مبلغ پول با مبلغ و واحد پول.

```csharp
public record Money(decimal Amount, string Currency);
```

مثال:

```csharp
var a = new Money(100, "IRR");
var b = new Money(100, "IRR");

Console.WriteLine(a == b); // true
```

**نکته:** در طراحی واقعی، قواعد اعتبارسنجی مبلغ، واحد پول و عملیات مالی نیز باید در نظر گرفته شوند.

---

## ۳۲. آدرس مشتری چه زمانی Entity و چه زمانی Value Object است؟

### ❓ سؤال
آدرس را چطور مدل می‌کنیم؟

### ✅ پاسخ
اگر آدرس در پروفایل مشتری هویت مستقل، تاریخچه یا عملیات مدیریتی داشته باشد، می‌تواند Entity باشد.

اما آدرس ارسال سفارش معمولاً باید تصویری از اطلاعات مقصد در زمان ثبت سفارش باشد؛ حتی اگر مشتری بعداً آدرس پروفایل خود را تغییر دهد.

```csharp
public record ShippingAddress(
    string Province,
    string City,
    string Street,
    string PostalCode);
```

سفارش می‌تواند این اطلاعات را به‌صورت Snapshot نگه دارد.

**قاعده:** مدل‌سازی را بر اساس قواعد کسب‌وکار انجام می‌دهیم، نه صرفاً بر اساس اسم جدول.

---

## ۳۳. آیا هر میکروسرویس فقط یک Aggregate Root دارد؟

### ❓ سؤال
یک Microservice چند Aggregate Root می‌تواند داشته باشد؟

### ✅ پاسخ
یک میکروسرویس می‌تواند چند Aggregate Root داشته باشد. مرز Aggregate با مرز Microservice یکسان نیست.

- Microservice مرز استقرار و مالکیت قابلیت‌های یک بخش کسب‌وکار است.
- Aggregate مرز سازگاری و تغییر وضعیت در مدل دامنه است.
- Bounded Context مرزی است که مدل و زبان دامنه در آن معنای مشخص دارند.

مثلاً یک سرویس فروش می‌تواند Aggregateهای سفارش و سبد خرید داشته باشد، اگر از نظر مرز دامنه و مالکیت داده چنین طراحی‌ای مناسب باشد.

---

## ۳۴. مرز میکروسرویس را چطور تعیین می‌کنیم؟

### ❓ سؤال
آیا باید برای هر جدول یک Microservice بسازیم؟

### ✅ پاسخ
خیر. ساخت میکروسرویس برای هر جدول معمولاً باعث پیچیدگی ارتباطات و وابستگی زیاد می‌شود.

مرز سرویس باید بر اساس قابلیت‌های کسب‌وکار، قواعد دامنه، مالکیت داده و نیازهای مستقل استقرار مشخص شود.

مثال:

- Sales Service
- Inventory Service
- Payment Service
- Identity Service

اگر دو قابلیت به‌شدت به هم وابسته باشند و همواره با هم تغییر کنند، ممکن است بهتر باشد ابتدا در یک مرز منطقی قرار گیرند.

---

## ۳۵. تفاوت Domain، Application، Infrastructure و Presentation چیست؟

### ❓ سؤال
در Clean Architecture مسئولیت هر لایه چیست؟

### ✅ پاسخ

**Domain**
- مدل دامنه
- قواعد کسب‌وکار اصلی
- Entityها و Value Objectها
- رفتارهای دامنه

**Application**
- هماهنگ‌سازی Use Caseها
- Command و Query Handlerها
- اجرای گردش کار
- تعریف نیازمندی‌های بیرونی از طریق Interfaceها

**Infrastructure**
- پیاده‌سازی Repositoryها
- ارتباط با دیتابیس
- RabbitMQ، سرویس‌های خارجی و فایل‌ها

**Presentation**
- API Controllerها یا Endpointها
- دریافت ورودی
- احراز هویت و پاسخ HTTP

### مثال

درخواست «ثبت سفارش» وارد API می‌شود. Application فرایند ثبت را هماهنگ می‌کند، Domain قواعد سفارش را اعمال می‌کند و Infrastructure اطلاعات را ذخیره می‌کند.

---

## ۳۶. چرا منطق کسب‌وکار را در Controller قرار نمی‌دهیم؟

### ❓ سؤال
اگر Controller بتواند همهٔ کارها را انجام دهد، چرا لایهٔ Application لازم است؟

### ✅ پاسخ
Controller بهتر است مسئولیت دریافت HTTP، اعتبارسنجی ورودی‌های سطح API و تولید پاسخ را بر عهده داشته باشد.

اگر قواعد سفارش، محاسبهٔ قیمت، کنترل موجودی و تأیید مدیر در Controller قرار گیرند:

- تست‌کردن سخت‌تر می‌شود.
- منطق میان Endpointها تکرار می‌شود.
- تغییر رابط کاربری یا نوع ارتباط دشوارتر خواهد شد.

Application Use Case را هماهنگ می‌کند و Domain قواعد اصلی را نگه می‌دارد.

**نکته:** هر Query ساده لزوماً به Domain Service نیاز ندارد. خواندن داده برای نمایش می‌تواند با یک Query Handler ساده و Projection انجام شود.

---

# بخش هفتم: CQRS و الگوهای طراحی

## ۳۷. CQRS چیست؟

### ❓ سؤال
تفاوت Command و Query چیست؟

### ✅ پاسخ
CQRS مسئولیت تغییر وضعیت را از خواندن اطلاعات جدا می‌کند.

- **Command:** درخواست تغییر وضعیت؛ مانند ثبت سفارش.
- **Query:** درخواست خواندن؛ مانند نمایش سفارش‌ها.

مثال:

```csharp
public record CreateOrderCommand(
    int CustomerId,
    List<CreateOrderItem> Items);

public record GetOrderByIdQuery(int OrderId);
```

این جداسازی لزوماً به معنی داشتن دو دیتابیس نیست. می‌توان در یک دیتابیس هم Command و Queryهای جدا داشت.

---

## ۳۸. Repository Pattern چیست؟

### ❓ سؤال
Repository چه مسئولیتی دارد؟

### ✅ پاسخ
Repository دسترسی به Aggregateهای دامنه را از جزئیات ذخیره‌سازی جدا می‌کند.

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(
        int id,
        CancellationToken cancellationToken);

    Task AddAsync(
        Order order,
        CancellationToken cancellationToken);
}
```

پیاده‌سازی می‌تواند در Infrastructure با EF Core انجام شود.

**نکته:** EF Core خودش امکاناتی شبیه Repository و Unit of Work دارد. بنابراین ساخت یک لایهٔ Repository عمومی و بدون ارزش افزوده برای تمام Entityها همیشه ضروری نیست.

---

## ۳۹. Unit of Work چیست؟

### ❓ سؤال
چرا از Unit of Work استفاده می‌کنیم؟

### ✅ پاسخ
Unit of Work مجموعه‌ای از تغییرات را به‌صورت یک واحد هماهنگ ذخیره می‌کند.

در EF Core، `DbContext.SaveChangesAsync()` تغییرات Trackingشده را ذخیره می‌کند و در شرایط معمول عملیات ذخیره‌سازی را در یک تراکنش انجام می‌دهد.

```csharp
order.AddItem(productId, quantity);

await db.SaveChangesAsync(cancellationToken);
```

اگر چند مرحله باید اتمیک باشند، باید مرز تراکنش را متناسب با نیاز تعیین کنیم.

---

## ۴۰. تفاوت Factory، Strategy و Decorator چیست؟

### ❓ سؤال
این Design Patternها را با مثال توضیح دهید.

### ✅ پاسخ

**Factory:** ساخت شیء را از مصرف‌کننده جدا می‌کند.

مثال: ساخت نوع مناسب درگاه پرداخت.

**Strategy:** الگوریتم قابل‌جایگزینی را از مصرف‌کننده جدا می‌کند.

مثال: محاسبهٔ تخفیف برای مشتری عادی و مشتری ویژه.

**Decorator:** رفتار جدیدی را بدون تغییر مستقیم کلاس اصلی اضافه می‌کند.

مثال: افزودن Logging یا Caching به یک سرویس.

این الگوها زمانی ارزشمندند که تنوع واقعی رفتار یا نیاز به جداسازی وجود داشته باشد؛ استفادهٔ بی‌دلیل از آن‌ها پیچیدگی را افزایش می‌دهد.

---

# بخش هشتم: Microservices و ارتباط سرویس‌ها

## ۴۱. ارتباط هم‌زمان و ناهم‌زمان چه تفاوتی دارند؟

### ❓ سؤال
REST و RabbitMQ چه زمانی مناسب‌اند؟

### ✅ پاسخ

**ارتباط هم‌زمان** مانند REST یا gRPC:

- پاسخ مستقیم می‌دهد.
- برای Queryها و عملیات نیازمند پاسخ فوری مناسب است.
- در صورت زنجیره‌شدن سرویس‌ها، وابستگی زمانی ایجاد می‌کند.

**ارتباط ناهم‌زمان** مانند RabbitMQ:

- فرستنده پیام را منتشر می‌کند.
- مصرف‌کننده ممکن است بعداً آن را پردازش کند.
- برای رویدادهای کسب‌وکار و کارهای پس‌زمینه مناسب است.

مثال: برای نمایش جزئیات محصول از API استفاده می‌کنیم؛ برای اطلاع‌رسانی «سفارش ثبت شد» می‌توان رویداد منتشر کرد.

---

## ۴۲. آیا میکروسرویس‌ها باید دیتابیس مشترک داشته باشند؟

### ❓ سؤال
چرا هر سرویس مالک داده‌های خودش است؟

### ✅ پاسخ
اصل رایج در معماری میکروسرویس این است که هر سرویس مالک داده‌های خود باشد و سرویس‌های دیگر از طریق قراردادهای مشخص به آن دسترسی پیدا کنند.

اگر چند سرویس مستقیماً جداول مشترک را تغییر دهند:

- تغییر Schema پرریسک می‌شود.
- وابستگی میان سرویس‌ها افزایش می‌یابد.
- مرزهای مالکیت داده از بین می‌روند.

این به معنی الزام دیتابیس فیزیکی جداگانه برای هر سرویس در همهٔ شرایط نیست؛ مهم، مالکیت و مرز دسترسی مستقل است.

---

## ۴۳. Saga Pattern چیست؟

### ❓ سؤال
اگر ثبت سفارش به چند سرویس وابسته باشد، چه می‌کنیم؟

### ✅ پاسخ
فرض کنید ثبت سفارش به این مراحل نیاز دارد:

1. ثبت سفارش
2. رزرو موجودی
3. دریافت تأیید پرداخت

در میکروسرویس‌ها نمی‌توان همیشه یک تراکنش دیتابیسی واحد روی تمام سرویس‌ها داشت.

Saga فرایند را به تراکنش‌های محلی تقسیم می‌کند. اگر مرحله‌ای شکست بخورد، ممکن است عملیات جبرانی اجرا شود.

مثال:

- سفارش ثبت می‌شود.
- موجودی رزرو می‌شود.
- پرداخت ناموفق می‌شود.
- رزرو موجودی آزاد می‌شود.
- سفارش به وضعیت ناموفق یا لغوشده می‌رود.

**نکته:** عملیات جبرانی لزوماً پاک‌کردن کامل آثار قبلی نیست؛ ممکن است یک تغییر وضعیت یا ثبت رویداد جبرانی باشد.

---

# بخش نهم: RabbitMQ، قابلیت اطمینان و پیام‌رسانی

## ۴۴. RabbitMQ چیست؟

### ❓ سؤال
چرا از RabbitMQ استفاده می‌کنیم؟

### ✅ پاسخ
RabbitMQ یک Message Broker است که پیام‌ها را میان تولیدکنندگان و مصرف‌کنندگان منتقل می‌کند.

مثال:

- Sales Service سفارش را ثبت می‌کند.
- رویداد `OrderCreated` منتشر می‌شود.
- Inventory Service آن را مصرف می‌کند.
- Notification Service پیام مناسب را برای مشتری ارسال می‌کند.

این معماری وابستگی مستقیم و هم‌زمان میان سرویس‌ها را کاهش می‌دهد.

---

## ۴۵. Exchange، Queue و Binding چیست؟

### ❓ سؤال
پیام در RabbitMQ چگونه مسیریابی می‌شود؟

### ✅ پاسخ

- **Producer:** پیام را منتشر می‌کند.
- **Exchange:** پیام را براساس نوع و قواعد مسیریابی هدایت می‌کند.
- **Binding:** رابطهٔ مسیریابی میان Exchange و Queue را مشخص می‌کند.
- **Queue:** پیام‌ها را تا زمان مصرف نگه می‌دارد.
- **Consumer:** پیام را دریافت و پردازش می‌کند.

انواع متداول Exchange:

- Direct
- Topic
- Fanout
- Headers

مثال: رویداد `OrderCreated` می‌تواند به Queueهای مختلف هدایت شود تا موجودی و اعلان‌ها هرکدام پردازش مستقل داشته باشند.

---

## ۴۶. چطور پیام را گم نکنیم؟

### ❓ سؤال
برای قابل‌اعتمادتر کردن RabbitMQ چه کارهایی انجام می‌دهیم؟

### ✅ پاسخ

- Queue مناسب را Durable تعریف می‌کنیم.
- برای پیام‌های لازم، Persistent Delivery Mode را تنظیم می‌کنیم.
- از Publisher Confirms استفاده می‌کنیم.
- مصرف‌کننده را با Manual Acknowledgement تنظیم می‌کنیم.
- پیام را بعد از پردازش موفق ACK می‌کنیم.
- برای خطاها Retry محدود و Dead Letter Queue در نظر می‌گیریم.
- پردازش تکراری را با Idempotency کنترل می‌کنیم.

**نکته:** Durable Queue و پیام Persistent به‌تنهایی تضمین کامل تحویل و پردازش موفق نیستند. Publisher Confirm و Acknowledgement نیز دو بخش متفاوت از قابلیت اطمینان‌اند.

---

## ۴۷. ACK و NACK چه تفاوتی دارند؟

### ❓ سؤال
چه زمانی پیام را ACK می‌کنیم؟

### ✅ پاسخ

- **ACK:** پیام با موفقیت پردازش شده است.
- **NACK:** پیام پردازش موفق نداشته است؛ بسته به تنظیمات می‌توان آن را دوباره در صف قرار داد یا به مسیر دیگری هدایت کرد.

پیام نباید پیش از انجام کار اصلی ACK شود؛ در غیر این صورت، اگر برنامه بعد از ACK و قبل از ذخیرهٔ داده متوقف شود، احتمال از دست رفتن اثر پردازش وجود دارد.

---

## ۴۸. Idempotency چیست؟

### ❓ سؤال
اگر RabbitMQ یک پیام را دوبار تحویل دهد چه می‌کنیم؟

### ✅ پاسخ
Idempotency یعنی اجرای دوبارهٔ یک عملیات، اثر تکراری ناخواسته ایجاد نکند.

مثال: اگر پیام کاهش موجودی دوبار پردازش شود، نباید موجودی دو بار کم شود.

راهکار متداول:

1. اختصاص شناسهٔ یکتا به پیام، مانند `MessageId`.
2. ذخیرهٔ سابقهٔ پردازش پیام.
3. ایجاد قید یکتا در دیتابیس.
4. انجام ثبت اثر کسب‌وکار و ثبت سابقهٔ پردازش در یک تراکنش محلی.
5. در صورت برخورد با پیام تکراری، از اجرای دوبارهٔ اثر جلوگیری شود.

نمونهٔ مفهومی:

```csharp
public async Task HandleAsync(
    OrderCreated message,
    CancellationToken cancellationToken)
{
    var alreadyProcessed =
        await db.ProcessedMessages.AnyAsync(
            x => x.MessageId == message.MessageId,
            cancellationToken);

    if (alreadyProcessed)
        return;

    // اعمال تغییر کسب‌وکار

    db.ProcessedMessages.Add(
        new ProcessedMessage(message.MessageId));

    await db.SaveChangesAsync(cancellationToken);
}
```

**هشدار مهم:** این مثال به‌تنهایی در برابر دو پردازش هم‌زمان ایمن نیست. باید قید یکتا و مدیریت برخورد تراکنش نیز در نظر گرفته شود.

---

## ۴۹. Outbox Pattern چیست؟

### ❓ سؤال
اگر سفارش ذخیره شود ولی انتشار پیام شکست بخورد، چه اتفاقی می‌افتد؟

### ✅ پاسخ
این مشکل ممکن است زمانی رخ دهد که سفارش در دیتابیس ثبت شده باشد اما RabbitMQ در دسترس نباشد.

Outbox Pattern کمک می‌کند ثبت داده و ثبت پیام موردنیاز برای انتشار، در یک تراکنش دیتابیسی انجام شوند.

مراحل:

1. سفارش ذخیره می‌شود.
2. پیام Outbox در همان تراکنش ثبت می‌شود.
3. Worker پیام‌های منتشرنشده را می‌خواند.
4. پیام به RabbitMQ منتشر می‌شود.
5. پس از تأیید مناسب، وضعیت پیام به‌روزرسانی می‌شود.
6. در صورت خطا، انتشار دوباره تلاش می‌شود.

مصرف‌کننده همچنان باید Idempotent باشد؛ چون ممکن است پیام بیش از یک بار منتشر یا تحویل داده شود.

---

## ۵۰. تفاوت Retry و Dead Letter Queue چیست؟

### ❓ سؤال
اگر پردازش پیام شکست بخورد چه می‌کنیم؟

### ✅ پاسخ

**Retry:** تلاش دوباره برای پردازش پیام.

برای خطاهای موقت مانند قطعی کوتاه‌مدت دیتابیس مناسب است.

**Dead Letter Queue:** انتقال پیام‌هایی که دیگر نباید در مسیر عادی باقی بمانند، به صف جداگانه برای بررسی و رسیدگی.

مثال:

- خطای موقت شبکه: Retry محدود با فاصلهٔ زمانی.
- پیام نامعتبر یا شکست دائمی: بررسی در DLQ.
- پس از اصلاح علت، پیام می‌تواند طبق رویهٔ کنترل‌شده دوباره پردازش شود.

**نکته:** Retry نامحدود می‌تواند باعث ازدحام صف و فشار بیشتر بر سرویس خراب شود.

---

# بخش دهم: امنیت، JWT و احراز هویت

## ۵۱. Authentication و Authorization چه تفاوتی دارند؟

### ❓ سؤال
این دو مفهوم را توضیح دهید.

### ✅ پاسخ

- **Authentication:** کاربر چه کسی است؟
- **Authorization:** این کاربر اجازهٔ انجام چه کاری را دارد؟

مثال: کاربر با JWT وارد سیستم می‌شود؛ سپس Policy بررسی می‌کند که آیا نقش یا مجوز لازم برای تأیید سفارش را دارد یا خیر.

---

## ۵۲. JWT چیست؟

### ❓ سؤال
JWT چگونه استفاده می‌شود؟

### ✅ پاسخ
JWT معمولاً شامل Header، Payload و Signature است.

در یک API:

1. کاربر احراز هویت می‌شود.
2. توکن صادر می‌شود.
3. کلاینت توکن را طبق روش امن انتخاب‌شده ارسال می‌کند.
4. API امضا، زمان اعتبار و سایر تنظیمات اعتبارسنجی را بررسی می‌کند.
5. دسترسی بر اساس Claims و Policy تعیین می‌شود.

**نکته:** محتوای معمول JWT رمزگذاری نشده است؛ بنابراین نباید اطلاعات محرمانه را در Payload قرار داد.

---

## ۵۳. چرا نقش کاربر از JWT درست خوانده نمی‌شود؟

### ❓ سؤال
اگر `User.IsInRole("Manager")` مقدار False بدهد چه چیزهایی را بررسی می‌کنیم؟

### ✅ پاسخ

- آیا Claim نقش در توکن وجود دارد؟
- نام Claim با تنظیمات `RoleClaimType` سازگار است؟
- آیا نقش در Middleware احراز هویت به `ClaimsPrincipal` تبدیل شده است؟
- آیا `UseAuthentication()` پیش از `UseAuthorization()` قرار دارد؟
- آیا Policy و نام Role درست تعریف شده‌اند؟

نمونهٔ بررسی:

```csharp
builder.Services
    .AddAuthentication()
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters =
            new TokenValidationParameters
            {
                RoleClaimType = ClaimTypes.Role
            };
    });
```

مقدار `RoleClaimType` باید با شکل Claim واقعی موجود در توکن هماهنگ باشد.

---

# بخش یازدهم: Performance و Observability

## ۵۴. Metrics، Logs و Traces چه تفاوتی دارند؟

### ❓ سؤال
Observability چیست؟

### ✅ پاسخ
Observability یعنی بتوانیم از خروجی‌ها و نشانه‌های سیستم، وضعیت داخلی و علت مشکلات را بهتر درک کنیم.

| ابزار | چه چیزی را نشان می‌دهد؟ |
|---|---|
| Logs | چه رویدادی رخ داده است؟ |
| Metrics | سیستم چه وضعیتی دارد و مقادیر آن چقدرند؟ |
| Traces | یک درخواست از چه مسیرهایی عبور کرده است؟ |

مثال:

- Log: سفارش شمارهٔ ۱۲۳ ثبت نشد.
- Metric: نرخ خطای API افزایش یافته است.
- Trace: درخواست در Inventory Service تأخیر زیادی داشته است.

---

## ۵۵. OpenTelemetry چیست؟

### ❓ سؤال
چرا از OpenTelemetry استفاده می‌کنیم؟

### ✅ پاسخ
OpenTelemetry مجموعه‌ای از ابزارها و استانداردها برای تولید، جمع‌آوری و انتقال Telemetry شامل Metrics، Traces و Logs است.

در معماری میکروسرویس می‌توان از آن برای دنبال کردن یک درخواست از API تا دیتابیس و سرویس‌های دیگر استفاده کرد.

ابزارهای متداول:

- OpenTelemetry SDK
- Prometheus برای جمع‌آوری Metrics
- Grafana برای داشبورد و نمایش
- سیستم‌های ذخیره و تحلیل Trace، متناسب با زیرساخت

**نکته:** برای سنجش کارایی، شاخص‌هایی مانند نرخ درخواست، زمان پاسخ، نرخ خطا، زمان اجرای Query و تأخیر صف اهمیت دارند.

---

## ۵۶. اگر بخواهیم IP، صفحات مشاهده‌شده و مرورگر کاربر را ذخیره کنیم چه می‌کنیم؟

### ❓ سؤال
این اطلاعات را کجا ذخیره می‌کنیم؟

### ✅ پاسخ
ابتدا هدف را مشخص می‌کنیم: تحلیل استفاده از سیستم، امنیت، عیب‌یابی یا گزارش‌گیری.

سپس می‌توانیم Middleware یا سامانهٔ Telemetry مناسبی برای ثبت رویدادهای لازم پیاده‌سازی کنیم.

اطلاعات احتمالی:

- شناسهٔ ناشناس نشست
- مسیر صفحه
- زمان مشاهده
- نوع مرورگر
- IP در صورت نیاز واقعی و مجاز

باید اطلاعات حساس را حداقل‌سازی کنیم، سیاست نگهداری مشخص داشته باشیم و IP را بدون ضرورت ذخیره نکنیم. پشت Proxy نیز باید فقط به Headerهای Forwarded از Proxyهای مورداعتماد اتکا کرد.

---

# بخش دوازدهم: سناریوهای واقعی مصاحبه

## ۵۷. سناریو: دو کاربر آخرین موجودی یک محصول را می‌خرند

### ❓ سؤال
چطور جلوی فروش بیش از موجودی را می‌گیرید؟

### ✅ پاسخ پیشنهادی
«من کنترل موجودی را صرفاً در حافظهٔ API انجام نمی‌دهم؛ چون چند درخواست یا چند نمونهٔ سرویس ممکن است هم‌زمان اجرا شوند. در دیتابیس از Update شرطی، تراکنش یا کنترل هم‌زمانی استفاده می‌کنم. تعداد ردیف‌های تغییرکرده را بررسی می‌کنم و اگر موجودی کافی نباشد، سفارش را به وضعیت مناسب می‌برم. در معماری میکروسرویس نیز رزرو موجودی می‌تواند از طریق Saga مدیریت شود.»

---

## ۵۸. سناریو: درخواست‌ها کند شده‌اند

### ❓ سؤال
چطور مشکل را پیدا می‌کنید؟

### ✅ پاسخ پیشنهادی

1. نرخ درخواست، زمان پاسخ و نرخ خطا را در Metrics بررسی می‌کنم.
2. Trace یک درخواست کند را مشاهده می‌کنم.
3. زمان دیتابیس، سرویس‌های خارجی و پردازش CPU را جدا می‌کنم.
4. Queryهای سنگین، N+1، قفل‌ها و صف‌های طولانی را بررسی می‌کنم.
5. پس از اصلاح، نتیجه را با Benchmark و Metrics مقایسه می‌کنم.

**نکته:** قبل از اندازه‌گیری، حدس نمی‌زنم که مشکل حتماً از دیتابیس یا کد است.

---

## ۵۹. سناریو: پیام سفارش دوبار پردازش شده است

### ❓ سؤال
چطور جلوی تکرار را می‌گیرید؟

### ✅ پاسخ پیشنهادی
«مصرف‌کننده را Idempotent می‌کنم. برای هر پیام شناسهٔ یکتا دارم و سابقهٔ پردازش را در دیتابیس ثبت می‌کنم. ثبت سابقه و تغییر کسب‌وکار را در یک تراکنش محلی انجام می‌دهم و قید یکتا می‌گذارم تا دو پردازش هم‌زمان نتوانند اثر تکراری ایجاد کنند. ACK نیز پس از تکمیل موفق پردازش انجام می‌شود.»

---

## ۶۰. سناریو: ثبت سفارش موفق است اما پیام RabbitMQ منتشر نمی‌شود

### ❓ سؤال
چطور این مشکل را حل می‌کنید؟

### ✅ پاسخ پیشنهادی
«برای جلوگیری از ناسازگاری میان ثبت سفارش و انتشار پیام از Outbox Pattern استفاده می‌کنم. سفارش و پیام Outbox در یک تراکنش ذخیره می‌شوند. یک Worker پیام‌های منتشرنشده را با Retry کنترل‌شده منتشر می‌کند. چون انتشار ممکن است تکراری شود، مصرف‌کننده نیز باید Idempotent باشد.»

---

## ۶۱. سناریو: سفارش باید تأیید مدیر بگیرد

### ❓ سؤال
این گردش کار را در معماری چگونه پیاده‌سازی می‌کنید؟

### ✅ پاسخ پیشنهادی
«ثبت سفارش و وضعیت تأیید را در مدل دامنه مشخص می‌کنم. Application Use Case ثبت یا تأیید را هماهنگ می‌کند و مجوز نقش مدیر در لایهٔ مناسب بررسی می‌شود. تغییر وضعیت فقط از مسیر مجاز انجام می‌شود. اگر مراحل به چند سرویس وابسته باشند، از رویدادها و در صورت نیاز Saga استفاده می‌کنم.»

---

# بخش سیزدهم: جمع‌بندی سریع مصاحبه

## ۶۲. جواب‌های کوتاه و مهم

**❓ چرا DbContext نباید Singleton باشد؟**

چون Thread-Safe نیست و وضعیت Tracking دارد؛ معمولاً Scoped است.

**❓ تفاوت Task و Thread چیست؟**

Thread واحد اجرایی است؛ Task نمایندهٔ یک عملیات است و الزاماً Thread اختصاصی ندارد.

**❓ تفاوت switchMap و mergeMap چیست؟**

`switchMap` به جریان جدید سوئیچ می‌کند؛ `mergeMap` اجازهٔ اجرای هم‌زمان چند جریان را می‌دهد.

**❓ چرا از decimal برای پول استفاده می‌کنیم؟**

برای محاسبات اعشاری مالی، رفتار عددی مناسب‌تری از double دارد؛ سیاست گرد کردن نیز مهم است.

**❓ Aggregate Root چیست؟**

نقطهٔ ورود اصلی برای تغییر وضعیت یک Aggregate و حفظ قواعد سازگاری آن است.

**❓ آیا هر Microservice یک Aggregate Root دارد؟**

خیر، ممکن است چند Aggregate Root داشته باشد.

**❓ CQRS چیست؟**

جدا کردن مسئولیت Commandهای تغییردهنده از Queryهای خواندنی.

**❓ Outbox چه مشکلی را حل می‌کند؟**

ناهماهنگی میان ذخیرهٔ داده در دیتابیس و انتشار پیام را کاهش می‌دهد.

**❓ Idempotency چیست؟**

جلوگیری از اثر تکراری ناخواسته هنگام اجرای مجدد یک عملیات.

**❓ چرا از SemaphoreSlim استفاده می‌کنیم؟**

برای کنترل هم‌زمانی، به‌خصوص در مسیرهای Async؛ اما به‌تنهایی قفل توزیع‌شده نیست.

**❓ N+1 چیست؟**

اجرای یک Query اصلی و سپس Queryهای متعدد برای هر ردیف؛ با Projection یا بارگذاری مناسب قابل‌پیشگیری است.

**❓ تفاوت Metrics و Logs چیست؟**

Metrics وضعیت عددی و روند سیستم را نشان می‌دهد؛ Logs رویدادهای رخ‌داده را ثبت می‌کند.

---

# 🧭 نقشهٔ مرور پیش از مصاحبه

1. **C# و ASP.NET Core:** OOP، DI، Lifetime، Task، Thread، Async، Lock
2. **EF Core و SQL:** Tracking، Include، N+1، Projection، Transactions، Index
3. **Angular:** DI، Component، Signals، RxJS، State Management
4. **DDD:** Entity، Value Object، Aggregate، Bounded Context
5. **معماری:** Clean Architecture، CQRS، Repository، Unit of Work
6. **Microservices:** مالکیت داده، REST، RabbitMQ، Saga، Outbox
7. **امنیت:** JWT، Claims، Authentication، Authorization
8. **کارایی و عیب‌یابی:** Metrics، Logs، Traces، OpenTelemetry

## 🎯 توصیهٔ نهایی

در مصاحبه فقط تعریف حفظ نکن. برای هر مفهوم سه چیز را آماده داشته باش:

- **تعریف کوتاه:** مفهوم چیست؟
- **سناریوی واقعی:** چه مشکلی را حل می‌کند؟
- **تصمیم معماری:** چه زمانی از آن استفاده نمی‌کنی و چرا؟

این سه مورد معمولاً پاسخ تو را از یک جواب حفظی به یک پاسخ حرفه‌ای تبدیل می‌کنند.


</div>

