# Cartfy

منصة لإدارة بطاقات العمل الرقمية (NFC) بنمط **الإدارة المركزية (Admin-Managed)**.

## 1) الرؤية
Cartfy تمكّن صاحب المنصة (Admin) من إنشاء وإدارة بروفايلات الزبائن من مكان واحد، مع منح كل زبون وصولاً محدودًا لتعديل بياناته الأساسية فقط.

## 2) الأطراف والصلاحيات

### Admin
- إنشاء حسابات الزبائن.
- توليد الرابط المختصر لكل زبون (مثل: `cartfy.me/user`).
- ربط دومين خاص بالبروفايل (Custom Domain).
- إدخال أكواد **HTML/CSS مخصصة لكل زبون** لتصميم واجهته برمجيًا.
- تفعيل/تعطيل البروفايل.

### Customer
- تسجيل دخول بكلمة مرور.
- تعديل:
  - رقم الهاتف.
  - روابط السوشيال.
  - الصورة الشخصية.
  - لون/ألوان الخلفية فقط.
- لا يمكنه تعديل القالب البرمجي (HTML/CSS injected by admin).

## 3) سلوك المنتج
- لا توجد صفحة رئيسية عامة (No Public Homepage).
- الوصول للبروفايل يتم فقط عبر رابط مباشر أو الدومين المخصص.
- صفحة البروفايل يجب أن تكون سريعة ومناسبة للموبايل (لأنها تُفتح غالبًا بعد مسح NFC).
- تتضمن زر **حفظ جهة اتصال (vCard)**.

## 4) المتطلبات الوظيفية (MVP)
1. نظام مصادقة Admin.
2. لوحة Admin لإدارة العملاء والبروفايلات.
3. نظام مصادقة Customer (بحساب واحد لكل بروفايل).
4. لوحة Customer محدودة الصلاحيات.
5. صفحة بروفايل عامة لكل عميل وفق slug.
6. تحميل صورة البروفايل.
7. توليد ملف vCard ديناميكيًا وتحميله بنقرة واحدة.
8. دعم Custom Domain عبر mapping في قاعدة البيانات.
9. تخزين أكواد HTML/CSS مخصصة لكل بروفايل (مع تعقيم مدخلات).

## 5) نموذج البيانات المقترح

### users
- `id`
- `email` (unique)
- `password_hash`
- `role` (`admin` | `customer`)
- `created_at`

### profiles
- `id`
- `user_id` (FK -> users)
- `slug` (unique)
- `display_name`
- `job_title`
- `phone`
- `email_public`
- `avatar_url`
- `bio`
- `bg_color_start`
- `bg_color_end`
- `is_active`
- `custom_html` (nullable)
- `custom_css` (nullable)
- `created_at`
- `updated_at`

### social_links
- `id`
- `profile_id` (FK -> profiles)
- `platform`
- `url`
- `sort_order`

### domains
- `id`
- `profile_id` (FK -> profiles)
- `hostname` (unique)
- `verified_at` (nullable)
- `created_at`

## 6) مسارات التطبيق (Routing)
- `GET /admin/login`
- `GET /admin/dashboard`
- `GET /admin/customers/:id`
- `GET /customer/login`
- `GET /customer/profile`
- `GET /:slug`  → صفحة البروفايل
- `GET /vcard/:slug.vcf` → تنزيل ملف vCard

> ملاحظة: عند تفعيل custom domain، نفس الـ profile يُخدم اعتمادًا على `Host` header بدل slug.

## 7) متطلبات الأمان
- تعقيم (sanitize) أكواد HTML/CSS المدخلة من لوحة Admin.
- حماية ضد XSS وCSRF.
- rate-limiting على login endpoints.
- كلمات المرور محفوظة بـ hash قوي (Argon2/Bcrypt).
- سجل تدقيق (Audit log) لعمليات Admin الحرجة.

## 8) تجربة المستخدم (Profile Page)
- صورة شخصية + الاسم + المسمى الوظيفي.
- أزرار سريعة: اتصال، واتساب (اختياري)، بريد.
- روابط السوشيال.
- زر واضح: **Save Contact (vCard)**.
- خلفية بألوان يحددها العميل.
- توافق ممتاز مع الجوال.

## 9) خطة تنفيذ مختصرة
1. إعداد المصادقة والأدوار.
2. بناء CRUD للبروفايلات من لوحة Admin.
3. بناء لوحة Customer محدودة.
4. بناء public renderer للبروفايل + vCard endpoint.
5. إضافة custom domain resolution.
6. اختبارات أمنية ووظيفية أساسية.

## 10) معايير قبول MVP
- Admin قادر ينشئ زبونًا جديدًا مع slug مباشر.
- Customer قادر يعدّل البيانات المسموحة فقط.
- فتح الرابط المباشر يعرض صفحة بروفايل احترافية.
- زر vCard يُنزّل ملف اتصال صحيح.
- custom domain (إن وُجد) يعرض نفس البروفايل بشكل صحيح.
