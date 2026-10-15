
<div dir="rtl">

# 📘 مصاحبه استخدامی شرکت هواپیمایی آتا برای موقعیت شغلی Full Stack .Net
---
> ### در اینجا سوالات مصاحبه در تاریخ 14050722  آمده البته با توضیحات و سوالات تکمیلی تر بنده به همراه پاسخ های صحیح که chatgpt جواب داده 



## ❓ سؤال ۱: اگر سرویس قبلاً در ۲۰۰ میلی‌ثانیه پاسخ می‌داد، اما حالا چند ثانیه طول می‌کشد، چه می‌کنی؟

### ✅ پاسخ

ابتدا حدس نمی‌زنم و کد را بدون بررسی تغییر نمی‌دهم. با Metrics، لاگ‌ها و Distributed Tracing محل دقیق کندی را پیدا می‌کنم.

**مراحل عیب‌یابی:**

1. بررسی زمان شروع مشکل و تغییرات اخیر در کد یا زیرساخت
2. بررسی Response Time، نرخ خطا و شاخص‌های P95 و P99
3. بررسی CPU، RAM، Garbage Collection و Thread Pool
4. بررسی SQL Server، کوئری‌های کند، Indexها، Blocking و Connection Pool
5. بررسی Redis، شبکه و سرویس‌های خارجی
6. اصلاح علت اصلی و اجرای مجدد تست بار

### 🔧 مثال: مشکل N+1 در EF Core

```csharp
var orders = await _context.Orders.ToListAsync();

foreach (var order in orders)
{
    order.Customer = await _context.Customers
        .FirstOrDefaultAsync(c => c.Id == order.CustomerId);
}
```

برای هر سفارش یک Query جدا اجرا می‌شود و ممکن است مشکل N+1 ایجاد شود. راهکار، استفاده از Projection یا Query مناسب برای دریافت اطلاعات موردنیاز با تعداد Query کمتر است.

### 🎯 پاسخ مناسب مصاحبه

«ابتدا با Metrics و Distributed Tracing مشخص می‌کنم تأخیر در کدام بخش است. سپس دیتابیس، کد، منابع سرور و سرویس‌های خارجی را بررسی می‌کنم. بعد از اصلاح علت اصلی، با تست بار و بررسی P95 مطمئن می‌شوم که عملکرد بهبود یافته است.»

---


## ❓ سؤال ۲: چطور لاگ‌ها را Trace می‌کنی؟

### ✅ پاسخ

برای دنبال کردن یک درخواست در چند بخش سیستم از **Trace ID و Distributed Tracing** استفاده می‌کنم.

```text
Client
  |
  v
API Gateway
  |
  v
Order Service
  |
  v
RabbitMQ
  |
  v
Payment Service
  |
  v
SQL Server
```

می‌خواهم بفهمم کدام مرحله باعث تأخیر شده است.

### 🔧 مثال در ASP.NET Core

```csharp
using System.Diagnostics;

var traceId = Activity.Current?.TraceId.ToString();

_logger.LogInformation(
    "Processing order. TraceId: {TraceId}",
    traceId);
```

در این مثال، شناسه Trace جاری در لاگ ثبت می‌شود تا اطلاعات مربوط به همان درخواست را جست‌وجو کنم.

**نکته:** در پردازش‌های غیرهم‌زمان مانند RabbitMQ باید Context ردیابی را همراه پیام منتقل و در مصرف‌کننده بازیابی کرد. صرفاً ثبت Trace ID در یک لاگ کافی نیست.

## ❓ سؤال ۳: چطور لاگ‌ها را مدیریت می‌کنی و از چه ابزارهایی استفاده می‌کنی؟

### ✅ پاسخ

از لاگ‌های ساختاریافته استفاده می‌کنم و آن‌ها را در یک سیستم متمرکز جمع‌آوری می‌کنم.

| ابزار | کاربرد |
|---|---|
| Serilog | تولید لاگ ساختاریافته |
| Elasticsearch + Kibana | ذخیره، جست‌وجو و تحلیل لاگ |
| Seq | مشاهده و فیلتر لاگ‌های برنامه |
| Grafana Loki | جمع‌آوری و جست‌وجوی لاگ |

### 🔧 مثال

```csharp
_logger.LogInformation(
    "Order {OrderId} created by user {UserId}",
    orderId,
    userId);
```

در این روش، `OrderId` و `UserId` به‌عنوان فیلدهای مجزا ثبت می‌شوند؛ بنابراین می‌توان لاگ‌ها را بر اساس آن‌ها فیلتر کرد.

- `Information`: رویدادهای عادی سیستم
- `Warning`: شرایط غیرعادی که لزوماً عملیات را متوقف نکرده‌اند
- `Error`: خطا در انجام یک عملیات
- `Critical`: خطاهای بسیار جدی که ممکن است کل سیستم را تحت تأثیر قرار دهند

### 🎯 پاسخ مناسب مصاحبه

«با Serilog لاگ‌های ساختاریافته تولید می‌کنم و آن‌ها را در ابزار متمرکزی مانند ELK یا Seq ذخیره می‌کنم. هنگام بروز مشکل، لاگ‌ها را بر اساس زمان، سطح خطا، شناسه درخواست و Trace ID فیلتر می‌کنم. همچنین رمز عبور، توکن و اطلاعات حساس را ثبت نمی‌کنم.»

## ❓ سؤال ۴: تفاوت Log، Metric و Trace چیست؟

### ✅ پاسخ

| مفهوم | به چه سؤالی پاسخ می‌دهد؟ | مثال |
|---|---|---|
| Log | چه اتفاقی افتاد؟ | خطای اتصال به دیتابیس |
| Metric | وضعیت سیستم چگونه است؟ | افزایش P95 زمان پاسخ |
| Trace | درخواست در کدام مرحله کند شد؟ | ۲٫۵ ثانیه تأخیر در SQL |

این سه مکمل یکدیگر هستند. Metric افزایش زمان پاسخ را نشان می‌دهد، Trace محل تأخیر را مشخص می‌کند و Log جزئیات خطا را ارائه می‌دهد.

---


## ❓ سؤال ۵: OpenTelemetry، Prometheus و Grafana چه هستند و چگونه با هم کار می‌کنند؟

### ✅ پاسخ

- **OpenTelemetry:** استاندارد و ابزار جمع‌آوری Telemetry
- **Prometheus:** پایش و ذخیره Metrics
- **Grafana:** نمایش نمودارها و تعریف Alert

```text
ASP.NET Core
     |
     v
OpenTelemetry
     |
     v
Prometheus
     |
     v
Grafana
```

برای مشاهده Traceهای توزیع‌شده می‌توانم از Jaeger یا Grafana Tempo هم استفاده کنم.

## ❓ سؤال ۶: چطور OpenTelemetry را در ASP.NET Core تنظیم می‌کنی؟

### ✅ پاسخ

ابتدا پکیج‌های لازم را نصب می‌کنم:

```bash
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Runtime
dotnet add package OpenTelemetry.Exporter.Prometheus.AspNetCore
```

سپس در `Program.cs`:

```csharp
using OpenTelemetry.Metrics;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services
    .AddOpenTelemetry()
    .WithMetrics(metrics =>
    {
        metrics
            .AddAspNetCoreInstrumentation()
            .AddRuntimeInstrumentation()
            .AddPrometheusExporter();
    });

var app = builder.Build();

app.MapControllers();
app.UseOpenTelemetryPrometheusScrapingEndpoint();

app.Run();
```

**توضیح:**

- `AddAspNetCoreInstrumentation()` اطلاعات عملکرد درخواست‌های HTTP را جمع‌آوری می‌کند.
- `AddRuntimeInstrumentation()` شاخص‌های Runtime دات‌نت مانند حافظه و Garbage Collection را فراهم می‌کند.
- `AddPrometheusExporter()` خروجی Metrics را برای Prometheus آماده می‌کند.
- `UseOpenTelemetryPrometheusScrapingEndpoint()` Endpoint مربوط به Metrics را فعال می‌کند.

با تنظیم پیش‌فرض، Endpoint معمولاً `/metrics` است.

## ❓ سؤال ۷: چطور Prometheus را تنظیم می‌کنی؟

### ✅ پاسخ

یک فایل `prometheus.yml` می‌سازم:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "orders-api"
    metrics_path: /metrics
    static_configs:
      - targets: ["host.docker.internal:5000"]
```

Prometheus هر ۵ ثانیه Metrics را از API دریافت می‌کند. آدرس `host.docker.internal:5000` برای حالتی است که Prometheus در Docker و API روی میزبان اجرا شود؛ باید آدرس و پورت را با محیط واقعی تطبیق دهم.

در بخش `Status → Targets` بررسی می‌کنم که وضعیت Target برابر `UP` باشد.

## ❓ سؤال ۸: چطور Grafana را به Prometheus وصل می‌کنی؟

### ✅ پاسخ

1. وارد Grafana می‌شوم.
2. از بخش Data Sources، گزینه Prometheus را انتخاب می‌کنم.
3. آدرس Prometheus را وارد می‌کنم؛ مثلاً `http://prometheus:9090` اگر هر دو در یک شبکه Docker باشند.
4. اتصال را تست می‌کنم.
5. داشبوردهایی برای زمان پاسخ، نرخ درخواست، خطا و مصرف منابع می‌سازم.

## ❓ سؤال ۹: چه شاخص‌هایی را مانیتور می‌کنی؟

### ✅ پاسخ

| شاخص | کاربرد |
|---|---|
| Response Time | مدت پاسخ درخواست |
| P95 / P99 | شناسایی درخواست‌های کند |
| Error Rate | درصد درخواست‌های ناموفق |
| Throughput | تعداد درخواست در ثانیه |
| CPU / RAM | بررسی مصرف منابع |
| Database Duration | زمان اجرای کوئری‌ها |
| Redis Latency | زمان پاسخ Redis |
| Queue Lag | تأخیر پردازش پیام‌ها |

### 🔧 مثال PromQL برای میانگین زمان پاسخ

```promql
sum(rate(http_server_request_duration_seconds_sum[5m]))
/
sum(rate(http_server_request_duration_seconds_count[5m]))
```

این Query زمانی معتبر است که Metricهای Histogram با این نام‌ها موجود باشند. نام و نوع Metricها به نسخه ابزار و تنظیمات وابسته است؛ ابتدا خروجی واقعی `/metrics` را بررسی می‌کنم.

---


## ❓ سؤال ۱۰: Health Check چیست و چطور از آن استفاده می‌کنی؟

### ✅ پاسخ

Health Check برای بررسی سلامت برنامه و وابستگی‌های آن مانند SQL Server و Redis استفاده می‌شود.

برای SQL Server می‌توانم پکیج زیر را نصب کنم:

```bash
dotnet add package AspNetCore.HealthChecks.SqlServer
```

سپس در `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHealthChecks()
    .AddSqlServer(
        builder.Configuration
            .GetConnectionString("DefaultConnection")!,
        name: "sql");

var app = builder.Build();

app.MapHealthChecks("/health");

app.Run();
```

در صورت موفقیت بررسی‌ها، پاسخ معمولاً `200 OK` است و اگر بررسی ثبت‌شده ناموفق شود، پاسخ پیش‌فرض معمولاً `503 Service Unavailable` خواهد بود.

## ❓ سؤال ۱۱: تفاوت Liveness و Readiness چیست؟

### ✅ پاسخ

| نوع | کاربرد |
|---|---|
| Liveness | آیا برنامه زنده است و باید اجرا بماند؟ |
| Readiness | آیا برنامه آماده دریافت ترافیک است؟ |

اگر SQL Server موقتاً قطع شود، ممکن است Readiness ناموفق شود؛ اما Liveness همچنان موفق باشد، چون خود برنامه زنده است.

### 🔧 مثال

```csharp
using Microsoft.Extensions.Diagnostics.HealthChecks;

builder.Services.AddHealthChecks()
    .AddCheck(
        "self",
        () => HealthCheckResult.Healthy());

app.MapHealthChecks("/health/live", new()
{
    Predicate = _ => false
});

app.MapHealthChecks("/health/ready");

```

در این نمونه، `/health/live` بررسی‌های ثبت‌شده را اجرا نمی‌کند و برای بررسی ساده زنده بودن استفاده می‌شود. `/health/ready` بررسی‌های ثبت‌شده را اجرا می‌کند. برای Production باید بررسی‌های واقعی وابستگی‌ها را ثبت و خروجی عمومی را از جزئیات حساس محافظت کنم.

### 🎯 پاسخ مناسب مصاحبه

«Health Check را برای تشخیص سلامت و آمادگی سرویس استفاده می‌کنم. Liveness برای تشخیص زنده بودن برنامه است و Readiness مشخص می‌کند که برنامه آماده دریافت ترافیک است یا نه. سپس این Endpointها را به Kubernetes یا سیستم مانیتورینگ متصل می‌کنم.»

---


## ❓ سؤال ۱۲: چطور Redis را با SQL Server سینک می‌کنی؟

### ✅ پاسخ

SQL Server را منبع اصلی داده در نظر می‌گیرم و Redis را برای Cache یا ذخیره‌سازی سریع داده‌های لحظه‌ای استفاده می‌کنم.

در حالت Cache-Aside:

1. اطلاعات در SQL Server ذخیره می‌شود.
2. پس از Commit موفق، کلید Cache حذف می‌شود.
3. درخواست بعدی اطلاعات جدید را از SQL می‌خواند و در Redis قرار می‌دهد.

### 🔧 مثال

```csharp
public async Task UpdateStockAsync(
    int productId,
    int quantity)
{
    var product = await _db.Products
        .FindAsync(productId);

    if (product is null)
        throw new InvalidOperationException(
            "Product not found");

    product.Stock = quantity;

    await _db.SaveChangesAsync();

    await _redis.KeyDeleteAsync(
        $"product:{productId}");
}
```

این مثال ساده است و مشکل شکست Redis بعد از Commit دیتابیس را به‌تنهایی حل نمی‌کند.

## ❓ سؤال ۱۳: اگر SQL ذخیره شود ولی Redis در دسترس نباشد، چه می‌کنی؟

### ✅ پاسخ

برای سیستم‌های حساس از **Transactional Outbox** استفاده می‌کنم.

```text
SQL Transaction
      |
      +--> Update Product
      |
      +--> Insert Outbox Event
                   |
                   v
             Background Worker
                   |
                   v
             Redis Invalidation
```

در یک تراکنش SQL، هم اطلاعات اصلی را تغییر می‌دهم و هم رویداد را در Outbox ثبت می‌کنم.

سپس Worker رویداد را پردازش می‌کند و کش را حذف یا به‌روزرسانی می‌کند. اگر Redis قطع باشد، پردازش مجدد پس از بازیابی امکان‌پذیر است.

**نکات مهم:**

- پردازش رویداد باید Idempotent باشد.
- Retry باید محدود و کنترل‌شده باشد.
- TTL به کاهش ماندگاری کش قدیمی کمک می‌کند.
- Outbox به‌تنهایی SQL و Redis را اتمیک نمی‌کند؛ سازگاری نهایی ایجاد می‌شود.

## ❓ سؤال ۱۴: کدام روش را انتخاب می‌کنی؟

### ✅ پاسخ

| سناریو | راهکار |
|---|---|
| Cache ساده محصول | Cache-Aside و حذف کلید |
| داده حساس و تغییرات مهم | Transactional Outbox |
| ارتباط چند سرویس | Outbox + RabbitMQ |
| جلوگیری از ماندن Cache قدیمی | TTL به‌عنوان لایه کمکی |

### 🎯 پاسخ مناسب مصاحبه

«SQL Server را منبع اصلی داده می‌دانم. در حالت ساده، بعد از Commit موفق کلید Redis را حذف می‌کنم. اگر بازیابی در شرایط خطا مهم باشد، از Transactional Outbox و Background Worker استفاده می‌کنم تا تغییرات قابل تکرار و بازیابی باشند.»

---


## ❓ سؤال ۱۵: اگر بخواهی سیستمی مثل اسنپ طراحی کنی که موقعیت ناوگان را داشته باشد، چه معماری انتخاب می‌کنی؟

### ✅ پاسخ

نیازمندی‌های اصلی را مشخص می‌کنم:

- دریافت موقعیت لحظه‌ای رانندگان
- نمایش موقعیت روی نقشه
- پیدا کردن راننده مناسب
- تخصیص راننده به سفر
- نمایش زنده حرکت راننده
- پشتیبانی از تعداد زیاد درخواست‌ها
- تحمل خطا و حفظ سازگاری داده

### 🏗️ معماری پیشنهادی

```text
Driver App             Passenger App
    |                        |
    v                        v
    API Gateway / Load Balancer
                |
       +--------+---------+
       |                  |
       v                  v
 Location Service     Trip Service
       |                  |
       v                  v
    Redis GEO          SQL Server
       |
       v
 Matching Service
       |
       v
 Realtime Gateway
       |
       v
 Passenger App

Trip Events --> Outbox --> RabbitMQ
                            |
                            v
                 Notification Services
```

این معماری مفهومی است؛ در نسخه اولیه لازم نیست همه بخش‌ها Microservice مستقل باشند.

## ❓ سؤال ۱۶: هر سرویس چه مسئولیتی دارد؟

### ✅ پاسخ

| سرویس | مسئولیت |
|---|---|
| Location Service | دریافت و اعتبارسنجی GPS |
| Driver Service | اطلاعات و وضعیت راننده |
| Matching Service | پیدا کردن راننده مناسب |
| Trip Service | مدیریت چرخه عمر سفر |
| Realtime Gateway | انتقال موقعیت زنده |
| Payment Service | پرداخت |
| Notification Service | اعلان‌ها |

برای شروع، ماژول‌ها را در یک Modular Monolith پیاده‌سازی می‌کنم و بخش‌های پرترافیک را در صورت نیاز جدا می‌کنم.

## ❓ سؤال ۱۷: اگر ۱۰۰ هزار راننده داشته باشیم و هرکدام هر ۲ ثانیه موقعیت ارسال کنند، چه می‌کنی؟

### ✅ پاسخ

بار تقریبی ورودی را محاسبه می‌کنم:

```text
100,000 / 2 = 50,000 updates/sec
```

پس باید حدود ۵۰ هزار به‌روزرسانی موقعیت در ثانیه را مدیریت کنیم.

راهکار:

- جدا کردن Location Service از عملیات تراکنشی سفر
- استفاده از Redis GEO برای موقعیت فعلی
- جلوگیری از ذخیره غیرضروری تمام مختصات در SQL
- تست بار و ظرفیت‌سنجی
- استفاده از Kafka یا ابزار پردازش جریان در صورت نیاز واقعی

این محاسبه فقط نرخ ورودی تقریبی است؛ ظرفیت واقعی به زیرساخت، اندازه پیام و الگوی ترافیک وابسته است.

## ❓ سؤال ۱۸: موقعیت رانندگان را کجا ذخیره می‌کنی؟

### ✅ پاسخ

| فناوری | کاربرد |
|---|---|
| Redis GEO | موقعیت فعلی و جست‌وجوی مکانی |
| SQL Server | سفرها و داده‌های تراکنشی |
| ذخیره‌سازی تاریخچه | مسیرهای موردنیاز و سوابق |

SQL Server منبع اصلی داده‌های تراکنشی است. Redis برای دسترسی سریع به موقعیت‌های لحظه‌ای استفاده می‌شود.

## ❓ سؤال ۱۹: چطور موقعیت راننده را دریافت می‌کنی؟

### ✅ پاسخ

اپ راننده مختصات و زمان ثبت را ارسال می‌کند:

```json
{
  "latitude": 35.6892,
  "longitude": 51.3890,
  "recordedAt": "2026-10-15T08:30:00Z",
  "sequence": 120
}
```

سرور:

1. هویت راننده را اعتبارسنجی می‌کند.
2. مختصات و زمان را بررسی می‌کند.
3. با Sequence Number یا زمان معتبر از پذیرش داده قدیمی جلوگیری می‌کند.
4. آخرین موقعیت را در Redis ثبت می‌کند.
5. در صورت نیاز، اطلاعات را برای مسافر دارای سفر فعال منتشر می‌کند.

**نکته:** عملیات بررسی ترتیب و به‌روزرسانی موقعیت باید در برابر درخواست‌های هم‌زمان نیز ایمن باشد.

## ❓ سؤال ۲۰: چطور نزدیک‌ترین راننده را پیدا می‌کنی؟

### ✅ پاسخ

از جست‌وجوی مکانی Redis GEO استفاده می‌کنم:

1. مختصات مسافر را دریافت می‌کنم.
2. رانندگان داخل شعاع موردنظر را پیدا می‌کنم.
3. رانندگان مشغول، آفلاین یا دارای موقعیت قدیمی را حذف می‌کنم.
4. زمان رسیدن گزینه‌های باقی‌مانده را با موتور مسیریابی محاسبه می‌کنم.
5. راننده مناسب را انتخاب و به سفر تخصیص می‌دهم.

فاصله مستقیم به‌تنهایی کافی نیست؛ مسیر خیابان و ترافیک نیز مهم‌اند.

## ❓ سؤال ۲۱: اگر دو مسافر هم‌زمان یک راننده را انتخاب کنند، چه می‌کنی؟

### ✅ پاسخ

این یک مسئله Concurrency است.

راهکار:

- کنترل تخصیص در Trip Service
- استفاده از تراکنش و کنترل هم‌زمانی
- رزرو اتمیک راننده یا تغییر وضعیت شرطی
- استفاده از قیدهای مناسب در SQL Server
- پیاده‌سازی Idempotency برای درخواست‌های تکراری

Redis GEO فقط نامزدها را پیدا می‌کند؛ تخصیص نهایی باید به‌صورت کنترل‌شده انجام شود.

## ❓ سؤال ۲۲: چطور موقعیت راننده را زنده به مسافر نشان می‌دهی؟

### ✅ پاسخ

از SignalR و WebSocket استفاده می‌کنم.

```text
Driver App
    |
    v
Location Service
    |
    v
Redis GEO
    |
    v
Realtime Gateway
    |
    v
Passenger App
```

باید دسترسی را محدود کنم تا فقط مسافر مجاز بتواند موقعیت راننده سفر فعال خود را مشاهده کند.

در مقیاس بالا، برای انتشار پیام بین چند نمونه Realtime Gateway از زیرساخت مناسب SignalR استفاده می‌کنم.

## ❓ سؤال ۲۳: اگر Redis یا RabbitMQ قطع شود، چه می‌کنی؟

### ✅ پاسخ

**در صورت قطع Redis:**

- وضعیت موقعیت را نامشخص یا قدیمی در نظر می‌گیرم.
- از نمایش مختصات قدیمی به‌عنوان موقعیت زنده جلوگیری می‌کنم.
- خطاها و زمان بازیابی را مانیتور می‌کنم.

**در صورت قطع RabbitMQ:**

- رویدادهای ثبت‌شده در Outbox حفظ می‌شوند.
- پس از بازیابی، Worker انتشار مجدد را انجام می‌دهد.
- مصرف‌کننده‌ها را Idempotent طراحی می‌کنم.

## ❓ سؤال ۲۴: مانیتورینگ این سیستم را چطور انجام می‌دهی؟

### ✅ پاسخ

از OpenTelemetry، Prometheus و Grafana استفاده می‌کنم.

| شاخص | کاربرد |
|---|---|
| Request Latency | زمان پاسخ API |
| P95 / P99 | درخواست‌های کند |
| Error Rate | نرخ خطا |
| GPS Update Rate | تعداد به‌روزرسانی موقعیت |
| Location Freshness | تازگی موقعیت راننده |
| Redis Latency | تأخیر Redis |
| Queue Lag | تأخیر پردازش پیام‌ها |

با Metrics مشکل را شناسایی می‌کنم، با Trace محل تأخیر را پیدا می‌کنم و با Log علت را بررسی می‌کنم.

## ❓ سؤال ۲۵: چرا از ابتدا همه بخش‌ها را Microservice نمی‌کنی؟

### ✅ پاسخ

ابتدا یک Modular Monolith با مرزهای مشخص طراحی می‌کنم. زمانی که مقیاس، نیاز به استقرار مستقل یا نیازهای متفاوت پردازشی توجیه داشته باشند، بخش‌هایی مانند Location Service و Realtime Gateway را جدا می‌کنم.

این روش پیچیدگی شبکه، استقرار، مانیتورینگ و سازگاری داده را در ابتدای پروژه کاهش می‌دهد.

---

</div>
