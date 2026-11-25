
<h1 dir="rtl" align="right">سناریوی عملی Kubernetes: مدیریت Pod و Namespace</h1>

<p dir="rtl" align="right">دو محیط مستقل برای توسعه و عملیات بساز؛ دو وب‌سرور با نام یکسان اجرا کن و از یک پاد آزمایشی به هر دو متصل شو. در این پروژه فقط از منابع <code>Pod</code> و <code>Namespace</code> استفاده می‌شود.</p>

<p dir="rtl" align="right"><strong>خروجی نهایی:</strong> دو صفحه وب متفاوت، یک کلاینت آزمایشی، مدیریت منابع، ارتباط مستقیم بین پادها و یک آزمون حذف و بازیابی.</p>

<h2 dir="rtl" align="right">معماری نهایی</h2>

<p dir="rtl" align="right"><strong>چیدمان محیط:</strong> در فضای نام <code>lab-dev</code> دو پاد <code>web</code> (Nginx) و <code>toolbox</code> (BusyBox) قرار دارند. در فضای نام <code>lab-prod</code> یک پاد <code>web</code> (Nginx) وجود دارد. پاد <code>toolbox</code> با استفاده از IP مستقیم هر یک از دو وب‌سرور، درخواست HTTP ارسال می‌کند.</p>

<p dir="rtl" align="right">در محیط توسعه، پاد <code>web</code> با ایمیج <code>nginx:stable</code> و پاد <code>toolbox</code> با ایمیج <code>busybox:1.36</code> قرار دارند. در محیط عملیاتی نیز یک پاد <code>web</code> با ایمیج <code>nginx:stable</code> اجرا می‌شود. هر دو وب‌سرور روی پورت ۸۰ پاسخ می‌دهند.</p>

<p dir="rtl" align="right">کلاینت آزمایشی از داخل <code>toolbox</code> مستقیماً با آدرس IP هر دو پاد وب‌سرور ارتباط برقرار می‌کند. محدوده‌های نام‌گذاری جدا هستند؛ بنابراین داشتن دو پاد با نام <code>web</code> در دو Namespace مختلف مجاز است. در این سناریو هیچ Service یا کنترلی مانند Deployment تعریف نمی‌شود.</p>

<p dir="rtl" align="right"><strong>اجزای سناریو:</strong> ۲ فضای نام، ۳ پاد (۲ پاد Nginx و ۱ پاد BusyBox) و بدون Deployment، Service یا StatefulSet.</p>

---

<h2 dir="rtl" align="right">بخش اول: مراحل تمرین (بدون پاسخ)</h2>

<p dir="rtl" align="right">ابتدا تلاش کن همه مراحل را بدون دیدن پاسخ‌ها انجام بدهی. هدف، فهم رفتار خود پادها و فضای نام است؛ نه حفظ کردن YAML.</p>

<h3 dir="rtl" align="right">مرحله ۱: بررسی سلامت کلاستر</h3>

<p dir="rtl" align="right">نودهای کلاستر را مشاهده کن. وضعیت، نام و IP آن‌ها را بررسی کن؛ همه نودهای موردنیاز باید آماده باشند.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> بتوانی Master و Workerها را شناسایی کنی.</p>

<h3 dir="rtl" align="right">مرحله ۲: ساخت دو Namespace</h3>

<p dir="rtl" align="right">دو فضای نام <code>lab-dev</code> و <code>lab-prod</code> بساز. وضعیت هر دو باید <code>Active</code> باشد.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> دو Namespace مستقل در فهرست وجود دارند.</p>

<h3 dir="rtl" align="right">مرحله ۳: پاد وب محیط توسعه</h3>

<p dir="rtl" align="right">در فضای توسعه، پاد <code>web</code> با یک کانتینر Nginx بساز؛ پورت ۸۰ و برچسب‌های <code>app=web</code> و <code>env=dev</code> داشته باشد.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> پاد اجرا می‌شود و یک کانتینر دارد.</p>

<h3 dir="rtl" align="right">مرحله ۴: پاد وب محیط عملیاتی</h3>

<p dir="rtl" align="right">در فضای عملیاتی نیز یک پاد با همان نام <code>web</code> و یک کانتینر Nginx بساز؛ برچسب <code>env=prod</code> داشته باشد.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> دو پاد همنام در دو Namespace متفاوت بدون تداخل وجود دارند.</p>

<h3 dir="rtl" align="right">مرحله ۵: درخواست و محدودیت منابع</h3>

<p dir="rtl" align="right">برای هر پاد وب: درخواست CPU برابر ۵۰ میلی‌کور، محدودیت CPU برابر ۲۵۰ میلی‌کور، درخواست حافظه <code>64Mi</code> و محدودیت حافظه <code>128Mi</code> قرار بده.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> با مشاهده جزئیات پاد، مقادیر Requests و Limits درست هستند.</p>

<h3 dir="rtl" align="right">مرحله ۶: ساخت پاد ابزار آزمایش</h3>

<p dir="rtl" align="right">در <code>lab-dev</code> پاد <code>toolbox</code> با ایمیج BusyBox بساز؛ کانتینر باید برای ورود و آزمایش فعال بماند.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> وارد Shell کانتینر می‌شوی و پاد به‌صورت عادی <code>Running</code> است.</p>

<h3 dir="rtl" align="right">مرحله ۷: تغییر صفحه هر وب‌سرور</h3>

<p dir="rtl" align="right">با اجرای دستور داخل کانتینر، محتوای صفحه اصلی Nginx را تغییر بده: توسعه <code>DEV ENVIRONMENT</code> و عملیات <code>PRODUCTION ENVIRONMENT</code> را نشان دهد.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> دو پاد پاسخ‌های متفاوت دارند.</p>

<h3 dir="rtl" align="right">مرحله ۸: تست ارتباط بین Namespaceها</h3>

<p dir="rtl" align="right">IP هر پاد وب را پیدا کن. از داخل پاد <code>toolbox</code> ابتدا به وب‌سرور توسعه و سپس به وب‌سرور عملیاتی درخواست HTTP بفرست.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> بدون Kubernetes Service و فقط با Pod IP هر دو پاسخ را دریافت می‌کنی.</p>

<h3 dir="rtl" align="right">مرحله ۹: حذف و بازیابی دستی</h3>

<p dir="rtl" align="right">پاد وب توسعه را حذف کن. ببین آیا خودش برمی‌گردد. بعد همان پاد را دوباره از Manifest ایجاد کن و اتصال را آزمایش کن.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> توضیح بدهی چرا پاد معمولی برخلاف Deployment خودکار بازسازی نمی‌شود.</p>

<h3 dir="rtl" align="right">مرحله ۱۰: پاک‌سازی انتخابی محیط توسعه</h3>

<p dir="rtl" align="right">کل Namespace توسعه را حذف کن. بررسی کن پادهای آن حذف شده‌اند اما وب‌سرور محیط عملیاتی سالم مانده است.</p>
<p dir="rtl" align="right"><strong>معیار قبولی:</strong> Namespace عملیاتی و پاد آن بدون تغییر باقی می‌مانند.</p>

<p dir="rtl" align="right"><strong>مرز سناریو:</strong> تنها منابع موردنیاز Namespace و Pod هستند. ابزارهای خط فرمان برای ورود به پاد، ویرایش فایل و آزمایش HTTP مجازند. هیچ Service، Deployment، Ingress یا ConfigMap نساز.</p>

---

<h2 dir="rtl" align="right">بخش دوم: پاسخ کامل منیفست‌ها</h2>

<p dir="rtl" align="right">چهار فایل زیر منابع موردنیاز سناریو را تعریف می‌کنند. ابتدا خودت تمرین را انجام بده و سپس این پاسخ‌ها را بررسی کن.</p>

<h3 dir="rtl" align="right">فایل اول: 01-namespaces.yaml</h3>

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: lab-dev
  labels:
    environment: development
---
apiVersion: v1
kind: Namespace
metadata:
  name: lab-prod
  labels:
    environment: production
```

<h3 dir="rtl" align="right">فایل دوم: 02-dev-web.yaml</h3>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
  namespace: lab-dev
  labels:
    app: web
    env: dev
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "50m"
          memory: "64Mi"
        limits:
          cpu: "250m"
          memory: "128Mi"
```

<p dir="rtl" align="right"><strong>نکته:</strong> هر دو پاد می‌توانند <code>web</code> نام داشته باشند، چون یکتایی نام Pod در محدوده همان Namespace سنجیده می‌شود، نه در کل کلاستر.</p>

<h3 dir="rtl" align="right">فایل سوم: 03-prod-web.yaml</h3>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
  namespace: lab-prod
  labels:
    app: web
    env: prod
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "50m"
          memory: "64Mi"
        limits:
          cpu: "250m"
          memory: "128Mi"
```

<h3 dir="rtl" align="right">فایل چهارم: 04-dev-toolbox.yaml</h3>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toolbox
  namespace: lab-dev
  labels:
    app: toolbox
    env: dev
spec:
  containers:
    - name: busybox
      image: busybox:1.36
      command:
        - /bin/sh
        - -c
        - "while true; do sleep 3600; done"
```

<p dir="rtl" align="right">در BusyBox یک فرمان خواب تکرارشونده گذاشته‌ایم تا پردازش اصلی کانتینر فوراً تمام نشود و بتوانیم با <code>kubectl exec</code> وارد آن شویم.</p>

---

<h2 dir="rtl" align="right">بخش سوم: پاسخ اجرایی و روش بررسی</h2>

<h3 dir="rtl" align="right">۱. ایجاد منابع</h3>

```bash
kubectl apply -f 01-namespaces.yaml
kubectl apply -f 02-dev-web.yaml
kubectl apply -f 03-prod-web.yaml
kubectl apply -f 04-dev-toolbox.yaml
```

<h3 dir="rtl" align="right">۲. بررسی Namespaceها و پادها</h3>

```bash
kubectl get namespaces
kubectl get pods -n lab-dev -o wide
kubectl get pods -n lab-prod -o wide
kubectl describe pod web -n lab-dev
```

<h3 dir="rtl" align="right">۳. تغییر محتوای صفحه توسعه و عملیات</h3>

```bash
kubectl exec -n lab-dev web -- sh -c \
  'echo "DEV ENVIRONMENT" > /usr/share/nginx/html/index.html'

kubectl exec -n lab-prod web -- sh -c \
  'echo "PRODUCTION ENVIRONMENT" > /usr/share/nginx/html/index.html'
```

<h3 dir="rtl" align="right">۴. مشاهده آدرس‌های واقعی پادهای وب</h3>

```bash
kubectl get pod web -n lab-dev -o wide
kubectl get pod web -n lab-prod -o wide
```

<h3 dir="rtl" align="right">۵. ورود به پاد ابزار آزمایش</h3>

```bash
kubectl exec -it -n lab-dev toolbox -- /bin/sh
```

<p dir="rtl" align="right">آدرس‌های فرضی زیر را با IP واقعی هر پاد جایگزین کن. دو دستور را <strong>داخل BusyBox</strong> اجرا کن:</p>

```bash
wget -qO- http://<DEV_POD_IP>:80
wget -qO- http://<PROD_POD_IP>:80
```

<p dir="rtl" align="right"><strong>نتیجه:</strong> پاسخ موفق ارتباط میان Namespaceها را نشان می‌دهد؛ خود Namespace به‌تنهایی جداسازی شبکه یا محدودیت دسترسی ایجاد نمی‌کند.</p>

<h3 dir="rtl" align="right">۶. حذف عمدی پاد وب توسعه</h3>

```bash
kubectl delete pod web -n lab-dev
kubectl get pods -n lab-dev
```

<p dir="rtl" align="right"><strong>انتظار:</strong> پاد <code>web</code> از فهرست حذف می‌شود و به‌صورت خودکار برنمی‌گردد؛ چون هیچ کنترلری برای بازسازی آن تعریف نکرده‌ایم.</p>

<h3 dir="rtl" align="right">۷. بازیابی پاد از فایل Manifest</h3>

```bash
kubectl apply -f 02-dev-web.yaml
kubectl get pod web -n lab-dev -o wide
```

<p dir="rtl" align="right">با ایجاد مجدد پاد، احتمالاً آدرس IP تغییر می‌کند. همچنین تغییر دستی صفحه HTML از بین رفته است، چون در فایل‌سیستم گذرای کانتینر انجام شده بود. برای تکرار آزمایش باید دوباره صفحه را تغییر بدهی.</p>

<h3 dir="rtl" align="right">۸. پاک‌سازی فقط محیط توسعه</h3>

```bash
kubectl delete namespace lab-dev
kubectl get namespaces
kubectl get pods -n lab-prod
```

<p dir="rtl" align="right"><strong>انتظار:</strong> منابع داخل Namespace توسعه حذف شوند و پاد وب محیط عملیاتی باقی بماند.</p>

---

<h2 dir="rtl" align="right">چک‌لیست پایان سناریو</h2>

- [ ] دو Namespace مستقل ایجاد شدند.
- [ ] دو پاد با نام یکسان، ولی در Namespaceهای متفاوت اجرا شدند.
- [ ] مقدار CPU و حافظه برای دو پاد وب تعریف شد.
- [ ] از داخل BusyBox به هر دو Nginx با IP مستقیم متصل شدیم.
- [ ] حذف پاد معمولی و بازیابی دستی آن آزمایش شد.
- [ ] حذف Namespace توسعه روی Namespace عملیاتی اثر نداشت.

<p dir="rtl" align="right"><strong>یادگیری کلیدی:</strong> Namespace محدوده نام‌گذاری و سازمان‌دهی منابع است؛ Pod واحد اجرای کانتینر است. بدون کنترلر، Pod حذف‌شده خودکار جایگزین نمی‌شود. بدون Service نیز می‌توان با IP پاد ارتباط گرفت، ولی آن IP پایدار نیست.</p>
