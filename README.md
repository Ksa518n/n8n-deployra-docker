# دليل نشر n8n على deployra.com باستخدام Docker

هذا الدليل يوضح كيفية نشر منصة أتمتة سير العمل **n8n** على **deployra.com** باستخدام حاوية Docker مخصصة.

## المتطلبات الأساسية

1.  حساب على [deployra.com](https://deployra.com).
2.  حساب على GitHub أو GitLab أو Bitbucket.
3.  تثبيت Git على جهازك المحلي.

## الخطوة 1: إعداد المشروع المحلي

قم بإنشاء مجلد للمشروع وانسخ الملفات التالية إليه:

1.  **`Dockerfile`**: ملف إعداد Docker.
2.  **`.env`**: ملف متغيرات البيئة.

### محتوى `Dockerfile`

تم تعديل هذا الملف ليتوافق مع متطلبات `deployra.com` التي تتوقع أن يستمع التطبيق على المنفذ `3000` (المتغير الافتراضي `PORT`).

```dockerfile
# استخدم الصورة الرسمية لـ n8n
FROM n8nio/n8n

# n8n يعمل افتراضياً على المنفذ 5678.
# deployra.com تتوقع أن يعمل التطبيق على المنفذ المحدد في متغير البيئة PORT، والافتراضي هو 3000.
# سنقوم بتغيير المنفذ الذي يستمع إليه n8n داخل الحاوية إلى 3000.
# n8n يستخدم متغير البيئة N8N_PORT لتحديد المنفذ الداخلي.
ENV N8N_PORT 3000

# كشف المنفذ 3000
EXPOSE 3000

# لا حاجة لتغيير نقطة الدخول (ENTRYPOINT) أو الأمر (CMD) الافتراضيين.
```

### محتوى ملف `.env`

**هام:** يجب عليك تعديل القيم الافتراضية، خاصةً `N8N_BASIC_AUTH_PASSWORD`، قبل النشر.

```ini
# ملف .env لمتغيرات بيئة n8n

# المنطقة الزمنية
GENERIC_TIMEZONE=Africa/Cairo

# تفعيل المصادقة الأساسية
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=n8nuser
N8N_BASIC_AUTH_PASSWORD=n8npassword_change_me

# متغيرات قاعدة البيانات (يوصى بها للإنتاج)
# إذا لم يتم تعيينها، سيتم استخدام SQLite، وهو مناسب للاختبارات الصغيرة.

# إعدادات Supabase (قاعدة بيانات خارجية مجانية وموثوقة):
# قم بتعطيل المتغيرات أدناه إذا كنت تستخدم SQLite.
DB_TYPE=postgres
DB_POSTGRES_HOST=db.bypuaavvyvvkknbcyfnp.supabase.co
DB_POSTGRES_DATABASE=postgres
DB_POSTGRES_USER=postgres
DB_POSTGRES_PASSWORD=Asd0556626357
# DB_POSTGRES_PORT=5432 (المنفذ الافتراضي 5432، لا حاجة لتعيينه عادةً)
# إذا كنت تخطط لاستخدام قاعدة بيانات خارجية (مثل PostgreSQL أو MySQL) على deployra.com،
# يجب عليك إنشاء خدمة قاعدة بيانات خاصة وتحديث هذه المتغيرات.

**ملاحظة هامة: تم إعداد المتغيرات أدناه للاتصال بقاعدة بيانات Supabase الخاصة بك.**
**يجب نسخ هذه المتغيرات إلى إعدادات خدمة n8n على deployra.com.**

| المتغير | القيمة |
| :--- | :--- |
| `DB_TYPE` | `postgres` |
| `DB_POSTGRES_HOST` | `db.bypuaavvyvvkknbcyfnp.supabase.co` |
| `DB_POSTGRES_DATABASE` | `postgres` |
| `DB_POSTGRES_USER` | `postgres` |
| `DB_POSTGRES_PASSWORD` | `Asd0556626357` |

**تنبيه:** `DB_POSTGRES_PASSWORD` هو كلمة مرور حساسة، يجب التعامل معها بأمان.

# مثال لـ PostgreSQL على deployra:
# DB_TYPE=postgres
# DB_POSTGRES_HOST=اسم-خدمة-قاعدة-البيانات.service
# DB_POSTGRES_DATABASE=n8n
# DB_POSTGRES_USER=n8nuser
# DB_POSTGRES_PASSWORD=كلمة-مرور-قاعدة-البيانات

# متغيرات إضافية
# deployra.com ستقوم بتعيين متغيرات N8N_HOST و WEBHOOK_URL تلقائيًا.
# إذا كنت بحاجة إلى تعيينها يدوياً، يمكنك إضافتها هنا.
# N8N_HOST=your-n8n-subdomain.deployra.app
# WEBHOOK_URL=https://${N8N_HOST}/
```

## الخطوة 2: رفع المشروع إلى GitHub

1.  قم بإنشاء مستودع Git جديد (Repository) على GitHub.
2.  قم بتهيئة المشروع المحلي وربطه بالمستودع:

    ```bash
    cd /home/ubuntu/n8n_deployra
    git init
    git add .
    git commit -m "Initial commit for n8n deployra setup"
    # استبدل [YOUR_REPO_URL] برابط مستودعك
    git remote add origin [YOUR_REPO_URL]
    git push -u origin master
    ```

## الخطوة 3: النشر على deployra.com

1.  **تسجيل الدخول**: قم بتسجيل الدخول إلى حسابك في deployra.com.
2.  **إنشاء خدمة جديدة**:
    *   انتقل إلى لوحة التحكم الخاصة بك.
    *   انقر على **"New Service"** (خدمة جديدة).
3.  **ربط المستودع**:
    *   اختر **"Web Service"** (خدمة ويب).
    *   اختر المستودع الذي قمت برفعه في الخطوة 2.
4.  **تكوين الإعدادات**:
    *   **Build Type**: اختر **"Docker"** (سيتم اكتشاف `Dockerfile` تلقائيًا).
    *   **Port**: تأكد من أن المنفذ المحدد هو **`3000`**.
    *   **Environment Variables**:
        *   يمكنك نسخ المتغيرات من ملف `.env` ولصقها في واجهة deployra.com الرسومية.
        *   **الأهم**: تأكد من تعيين متغيرات `N8N_BASIC_AUTH_USER` و `N8N_BASIC_AUTH_PASSWORD` بقيم قوية.
        *   **ملاحظة**: deployra.com ستقوم تلقائيًا بتعيين متغير `PORT` وستقوم n8n باستخدامه بفضل التعديل في `Dockerfile`.
5.  **النشر**:
    *   انقر على **"Deploy"** (نشر). ستقوم deployra.com بسحب الكود، بناء صورة Docker، ونشر التطبيق.

## الخطوة 4: الوصول إلى n8n

بمجرد اكتمال النشر بنجاح، يمكنك الوصول إلى n8n عبر النطاق الفرعي `deployra.app` الذي تم تعيينه لخدمتك. سيُطلب منك اسم المستخدم وكلمة المرور التي قمت بتعيينها في متغيرات البيئة (`N8N_BASIC_AUTH_USER` و `N8N_BASIC_AUTH_PASSWORD`).
