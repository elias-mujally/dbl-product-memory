# AI Handoff — تعليمات لأي مساعد ذكاء اصطناعي

## الهدف

هذا الملف يتيح لـChatGPT أو Claude أو Gemini أو Copilot أو Codex فهم مشروع DBL بسرعة دون إعادة سرد القصة من الصفر.

## اقرأ أولًا

بالترتيب:

1. `CURRENT_STATUS.md`
2. `VISION.md`
3. `ROADMAP.md`
4. `DECISIONS.md`
5. `PROBLEMS_AND_SOLUTIONS.md`
6. `CHANGELOG.md`
7. `BLOCKERS.md`

## الهوية

- المنتج: DBL Employee AI
- المظلة: Digital Blueprint Lab (DBL)
- المستودع البرمجي: `elias-mujally/dbl-employee-ai`
- الإنتاج: `https://dbl-employee-ai.vercel.app`
- Memory repo: `elias-mujally/dbl-product-memory`

## وصف المنتج في سطر واحد

SaaS لبناء موظف AI قابل للتخصيص يتعلم من معرفة النشاط المعتمدة ويتعامل مع العملاء، بدءًا من WhatsApp ثم قنوات متعددة لاحقًا.

## الحالة الحالية المختصرة جدًا

- Main يتضمن Knowledge Hub الجديد من PR #34.
- PR #35 مفتوح Draft لإصلاح approval UX وإضافة durable create idempotency.
- آخر head مدفوع لـPR #35: `d79dc723f45499e4b14c93af60e645eff4a018d3`.
- هناك إصلاح محلي غير مدفوع في ملفين بسبب Codex external-command limit.
- لا تدمج PR #35 قبل دفع الإصلاح وتشغيل CI ومراجعة مستقلة.
- Meta Business Verification وTech Provider qualification ما زالا pending.

## قواعد الثقة

1. لا تفترض أن PR مدمج لأن التقرير قال “جاهز”. تحقق من GitHub.
2. لا تفترض أن local fix موجود على GitHub.
3. Production alias هو المرجع، لا immutable deployment قديم.
4. لا تغيّر production/Meta/Supabase data دون موافقة صريحة.
5. لا تضع secrets في commits أو logs أو هذه الذاكرة.
6. عند وجود فرق بين الاختبارات وما يراه المستخدم، تجربة الإنتاج الفعلية لها الأولوية في التشخيص.

## قواعد هندسية

- Forward-only migrations.
- لا تعديل migration تاريخية.
- RLS وworkspace isolation لا تُضعف.
- SECURITY DEFINER يحتاج safe search_path وأسماء fully qualified.
- browser لا يحدد actor/workspace/phone/provider authority.
- approved Knowledge فقط للردود التجارية.
- client locks ليست ضمان idempotency.
- provider/database raw errors لا تصل للمستخدم.
- Arabic/English + RTL/LTR + bidi isolation.
- AI reply language منفصلة عن UI locale.

## أسلوب تنفيذ المهام

لأي PR كبير:

1. Audit/read-only أولًا عند عدم وضوح السبب.
2. Draft PR.
3. Full validation.
4. Critical self-review.
5. إصلاح High وMedium.
6. Independent final review.
7. Squash merge مع expected head SHA.
8. Production verification.
9. تحديث هذا المستودع.

## نقطة الاستئناف لـPR #35

عندما تتوفر صلاحية الدفع:

- لا تعيد بناء الحل.
- افحص الـdiff المحلي في:
  - `components/knowledge-task-form-actions.tsx`
  - `tests/e2e/knowledge-idempotency.spec.ts`
- تحقق أن:
  - نفس العملية تستخدم نفس idempotency key عند retry.
  - “Save and add another” يولد key جديدًا للعملية الجديدة.
  - لا توجد ملفات أخرى معدلة.
- شغّل checks.
- commit/push إلى branch الحالي.
- انتظر CI.
- final review.

## ما لا ينبغي فعله

- لا تستخدم PR جديدًا لتجاوز PR #35 إلا إذا أصبح الفرع غير قابل للاسترداد.
- لا تدمج head القديم.
- لا تحذف replay ledger أو تقلل الأمان لتجنب migration.
- لا تجعل Owner يشعر أن DBL يراجع علامته.
- لا تجعل Documents شرطًا لـ100% readiness.
- لا تعرض technical Knowledge schema في المسار الطبيعي.
- لا تظهر language/theme selector قبل جاهزية الصفحات.

## أسئلة يجب طرحها عند قرار جديد

- هل يحسن تجربة صاحب مشروع غير تقني؟
- هل يحافظ على سيطرة العميل؟
- هل يمنع فقدان البيانات أو التكرار؟
- هل يمكن تنفيذه بلا schema change؟
- إذا احتاج migration، هل الضمان يستحقها؟
- هل الرسالة واضحة بالعربية والإنجليزية؟
- هل الاختبارات تفتح التجربة فعليًا، أم تبحث عن نص في source فقط؟

## تحديث الذاكرة

بعد إنجاز مهمة:

- `CURRENT_STATUS.md`: النقطة الحالية.
- `CHANGELOG.md`: ماذا حدث وتاريخه.
- `DECISIONS.md`: لماذا اتخذ القرار.
- `PROBLEMS_AND_SOLUTIONS.md`: المشكلة والدرس.
- `BLOCKERS.md`: إزالة/إضافة العائق.
