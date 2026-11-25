
<h1 dir="rtl" align="right">
سناریوی عملی Kubernetes: Pod و Namespace
</h1>

<p dir="rtl" align="right">
هدف این سناریو ایجاد دو محیط مستقل
برای توسعه و محیط عملیاتی است.
</p>

## Architecture

![Architecture](./architecture.png)

<h2 dir="rtl" align="right">
مراحل تمرین
</h2>

<p dir="rtl" align="right">
مرحله اول: دو Namespace ایجاد کن.
</p>

<p dir="rtl" align="right">
مرحله دوم: یک Pod با ایمیج Nginx بساز.
</p>

<h2 dir="rtl" align="right">
پاسخ و منیفست‌ها
</h2>

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: lab-dev
```

[مشاهده فایل PDF کامل](./scenario.pdf)
