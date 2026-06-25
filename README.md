# Session-based auth
## 1. تنظیمات پروژه (`settings.py`)

### 1.1. برنامه‌های ضروری (Essential Apps)

برای فعال‌سازی سیستم احراز هویت و مدیریت سشن‌ها، اطمینان حاصل کنید که برنامه‌های زیر در `INSTALLED_APPS` وجود دارند:

```python
INSTALLED_APPS = [
    # ... other apps
    'django.contrib.auth',       # هسته اصلی احراز هویت جنگو
    'django.contrib.sessions',   # برای مدیریت سشن‌های کاربران
    'accounts',                  # اپلیکیشن خودتان (یا هر نامی که انتخاب کردید)
]
```

**چرا؟**
- `django.contrib.auth`: تمام مدل‌ها (مانند `User`)، ویوها، فرم‌ها و مجوزهای لازم را فراهم می‌کند.
- `django.contrib.sessions`: جنگو از سشن‌ها برای حفظ وضعیت ورود کاربر استفاده می‌کند. کوکی سشن در مرورگر ذخیره می‌شود و سرور با استفاده از آن کاربر را شناسایی می‌کند.
- `accounts`: اپلیکیشن شما که ویوها و URLهای مربوط به احراز هویت را در خود جای می‌دهد.

### 1.2. Middlewareهای لازم (Required Middleware)

این middlewareها برای پردازش درخواست‌های احراز هویت و سشن ضروری هستند:

```python
MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',  # پردازش سشن‌ها
    'django.middleware.csrf.CsrfViewMiddleware',             # محافظت از فرم ها 
    'django.contrib.auth.middleware.AuthenticationMiddleware', # request.user
]
```

**چرا؟**
- `SessionMiddleware`: امکان خواندن و نوشتن داده‌های سشن را فراهم می‌کند.
- `AuthenticationMiddleware`: به صورت خودکار `request.user` را بر اساس کوکی سشن پر می‌کند. این مهم‌ترین بخش برای دسترسی به کاربر فعلی در ویوها و تمپلیت‌هاست.
- `CsrfViewMiddleware`: از حملات Cross-Site Request Forgery (CSRF) جلوگیری می‌کند، خصوصاً برای درخواست‌های `POST` مانند logout.

### 1.3. تنظیمات ریدایرکت (Redirect Settings)

این تنظیمات تعیین می‌کنند کاربر پس از ورود یا خروج به کدام صفحه هدایت شود:

```python
LOGIN_REDIRECT_URL = 'home'  # نام URL مقصد پس از ورود موفق
LOGOUT_REDIRECT_URL = 'login' # نام URL مقصد پس از خروج موفق
```

**چرا؟**
- `LOGIN_REDIRECT_URL`: بعد از ورود موفق، کاربر به صفحه‌ای که با نام `home` تعریف شده هدایت می‌شود. این تجربه کاربری خوبی را ایجاد می‌کند.
- `LOGOUT_REDIRECT_URL`: پس از خروج، کاربر به صفحه `login` فرستاده می‌شود تا بتواند دوباره وارد شود.

### 1.4. تنظیمات امنیتی کوکی (Cookie Security)

برای افزایش امنیت، این تنظیمات را در نظر بگیرید:

```python
SESSION_COOKIE_HTTPONLY = True  # کوکی سشن فقط از طریق سرور قابل دسترسی باشد
# SESSION_COOKIE_SECURE = True  # کوکی سشن فقط روی HTTPS ارسال شود (برای محیط production)
```

**چرا؟**
- `SESSION_COOKIE_HTTPONLY = True`: از دسترسی اسکریپت‌های جاوااسکریپت در مرورگر به کوکی سشن جلوگیری می‌کند. این یک لایه امنیتی مهم در برابر حملات XSS (Cross-Site Scripting) است.
- `SESSION_COOKIE_SECURE = True`: در محیط production که سایت از HTTPS استفاده می‌کند، این تنظیم اطمینان می‌دهد که کوکی سشن فقط روی ارتباطات امن ارسال شود. در محیط توسعه (localhost) معمولاً `False` است تا session کار کند.

---

## 2. اعمال Migrationها

قبل از هر کاری، دیتابیس را با migrationهای جنگو همگام‌سازی کنید:

```bash
python manage.py migrate
```

---

## 3. ساخت Superuser

برای دسترسی به پنل ادمین و تست کردن سیستم ورود، یک کاربر ادمین بسازید:

```bash
python manage.py createsuperuser
```

---

## 4. تعریف URLها (`accounts/urls.py`)

در اپلیکیشن `accounts`، یک فایل `urls.py` بسازید و URLهای مربوط به احراز هویت را در آن تعریف کنید:

```python
# accounts/urls.py
from django.urls import path
from django.contrib.auth.views import LoginView, LogoutView
from .views import home

urlpatterns = [
    path('login/', LoginView.as_view(template_name='accounts/login.html'), name='login'),
    path('logout/', LogoutView.as_view(next_page='login'), name='logout'),
    path('', home, name='home'),
]
```


---

## 5. تعریف ویوها (`accounts/views.py`)

برای صفحه اصلی که نیاز به ورود دارد:

```python
# accounts/views.py
from django.shortcuts import render
from django.contrib.auth.decorators import login_required

@login_required(login_url='login')
def home(request):
    return render(request, 'accounts/home.html')
```

**چرا؟**
- **`@login_required(login_url='login')`**: این decorator تضمین می‌کند که فقط کاربرانی که لاگین کرده‌اند می‌توانند به این ویو دسترسی داشته باشند.
    - اگر کاربری که لاگین نکرده سعی کند به این صفحه دسترسی پیدا کند، به URL با نام `login` (که در `settings.py` یا `urls.py` تعریف شده) هدایت می‌شود.


---

## 7. قالب‌های HTML (Templates)

### 7.1. صفحه ورود (`templates/accounts/login.html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>
    <h2>Login</h2>

    {% if form.errors %}
        <p style="color: red;">Username or password is not correct.</p>
    {% endif %}

    <form method="post" action="{% url 'login' %}"> {# ارجاع به URL با نام login #}
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

### 7.2. صفحه اصلی (`templates/accounts/home.html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
</head>
<body>
    <h1>Welcome {{ user.username }}</h1>
    <p>Name: {{ user.username }}</p>
    <p>Email: {{ user.email }}</p>
    <p>Admin permission: {{ user.is_staff }}</p>

    {# فرم خروج #}
    <form method="post" action="{% url 'logout' %}">
        {% csrf_token %}
        <button type="submit">Log out</button>
    </form>
</body>
</html>
```

---

# روند اجرای عملیاتی (Workflow)

## سناریوی ورود (Login)

1.  کاربر به آدرس `/accounts/login/` مراجعه می‌کند.
2.  `LoginView` اجرا شده و تمپلیت `accounts/login.html` را با فرم خالی نمایش می‌دهد.
3.  کاربر نام کاربری و رمز عبور را وارد کرده و دکمه Login را می‌زند.
4.  فرم با متد `POST` به آدرس `login` ارسال می‌شود.
5.  `LoginView` اطلاعات را اعتبارسنجی می‌کند.
6.  در صورت صحت اطلاعات:
    *   سشن کاربر ایجاد و ذخیره می‌شود.
    *   `AuthenticationMiddleware` کوکی سشن را در مرورگر کاربر قرار می‌دهد.
    - کاربر به URL تعیین شده توسط `LOGIN_REDIRECT_URL` (که `home` است) هدایت می‌شود.
    - بنابراین کاربر به آدرس `/accounts/` می‌رسد.
7.  `home` view اجرا می‌شود. چون کاربر لاگین کرده، `login_required` اجازه ورود می‌دهد و تمپلیت `home.html` نمایش داده می‌شود.

## سناریوی خروج (Logout)

1.  کاربر در صفحه home روی دکمه "Log out" کلیک می‌کند.
2.  فرم `POST` به آدرس `logout` (یعنی `/accounts/logout/`) ارسال می‌شود.
3.  `LogoutView` اجرا می‌شود.
4.  سشن کاربر پاک می‌شود.
5.  کاربر به URL تعیین شده توسط `next_page` در `LogoutView` (که `login` است) هدایت می‌شود.
6.  کاربر به آدرس `/accounts/login/` می‌رسد و صفحه login نمایش داده می‌شود.

## سناریوی دسترسی به صفحه اصلی بدون ورود (Unauthorized Access to Home)

1.  کاربر به آدرس `/accounts/` (که همان `home` است) مراجعه می‌کند.
2.  `AuthenticationMiddleware` بررسی می‌کند که کاربر لاگین نکرده است.
3.  Decorator `@login_required` فعال شده و کاربر را به URL با نام `login` (یعنی `/accounts/login/`) هدایت می‌کند.
4.  کاربر صفحه login را می‌بیند.
