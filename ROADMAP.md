# خارطة الطريق

آخر تحديث: 2026-08-05

## المرحلة الحالية: تثبيت Knowledge approval workflow

### PR #35

المطلوب قبل الدمج:

- دفع إصلاح idempotency key لعملية “Save and add another”.
- تشغيل CI على الرأس الجديد.
- مراجعة مستقلة نهائية.
- squash merge عند عدم وجود High/Medium findings.
- تحقق إنتاجي من الأدوار، review queue، retries، ومنع duplicates.

## المرحلة التالية مباشرة

### 1. إكمال الترجمة الأساسية

المسارات المؤجلة أو غير المكتملة:

- Contacts
- Analytics
- Subscription
- Appearance & Language
- Privacy inside Settings

### 2. إظهار Language selector

لا يظهر إلا بعد اكتمال ترجمة الصفحات الأساسية.

المطلوب:

- Arabic / English
- حفظ preference
- SSR صحيح
- RTL/LTR
- فصل UI language عن AI reply language

### 3. Theme controls

- Light
- Dark
- System
- ترحيل الألوان الثابتة إلى semantic tokens
- Visual regression tests
- عدم إظهار Dark قبل اكتمال component migration

### 4. Meta readiness

بعد Business Verification:

- إكمال Independent Tech Provider qualification.
- مراجعة إعدادات التطبيق.
- تسجيل فيديوهات App Review.
- طلب Advanced Access:
  - whatsapp_business_management
  - whatsapp_business_messaging
- إضافة App icon معتمد 1024×1024.
- تثبيت Privacy/Terms/Data Deletion URLs في Meta.
- إعادة اختبار Embedded Signup config المنتهي بـ1674.

### 5. WhatsApp production customer onboarding

- اختبار FINISH event الحقيقي.
- التحقق من WABA/phone metadata.
- code exchange.
- token storage في dedicated Secret Manager project.
- app subscription وphone registration.
- reconnect/disconnect.
- token expiration lifecycle.
- account_update handling.

## Backlog قريب

### Knowledge Phase 2

- Unified business profile editor.
- Review queue refinements.
- Add multiple FAQs faster.
- Attention-needed panel.
- Logo upload بعد تصميم storage صحيح.
- Product images.
- Product variants/categories.
- Bulk import.
- First-class exchange policy enum.

### Product experience

- First-run experience.
- Guided checklist.
- AI employee test chat.
- Knowledge gaps suggestions.
- Better “what next?” CTAs.
- Product favicon.

### Team and organization

- Team member management UI.
- Invitations.
- Roles and permissions UX.
- Activity history.
- Workspace switching إذا دُعم أكثر من Workspace.

### Commercial

- Subscription page.
- Pricing model.
- Payment provider.
- Usage limits.
- Upgrade/downgrade.
- Invoices.

### Analytics

- Conversation volume.
- Response time.
- AI vs human handoff.
- Top customer questions.
- Knowledge gaps.
- Leads and conversions.
- Failed/uncertain replies.

## Backlog متوسط المدى

- Instagram integration.
- Facebook Messenger.
- Web chat widget.
- CRM integrations.
- Calendar/booking.
- Ecommerce catalogs.
- Templates by business type.
- Multi-agent architecture.

## قواعد ترتيب الأولويات

1. الأمان وعدم فقدان البيانات.
2. إكمال تدفق العميل الأساسي.
3. إزالة الالتباس قبل إضافة ميزات جديدة.
4. تجربة صاحب المشروع غير التقني.
5. الجاهزية للإيرادات.
6. التوسع متعدد القنوات بعد استقرار WhatsApp.

## تعريف MVP الحقيقي

لا نعتبر المنتج MVP جاهزًا للبيع حتى يتحقق:

- تسجيل ودخول موثوق.
- Workspace onboarding واضح.
- AI employee customization.
- Knowledge سهلة ومنظمة.
- WhatsApp Embedded Signup يعمل للعميل الخارجي.
- إرسال واستقبال وردود آمنة.
- Review/auto modes واضحة.
- صفحات قانونية.
- Subscription/payment path.
- دعم وتشخيص أساسي.
