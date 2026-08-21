# AI Handoff — تعليمات لأي مساعد ذكاء اصطناعي

## الهدف

هذا الملف يتيح لـChatGPT أو Claude أو Gemini أو Copilot أو Codex فهم DBL بسرعة دون إعادة بناء القصة من الصفر.

## اقرأ أولًا

1. `CURRENT_STATUS.md`
2. `STRATEGY_PIVOT_2026-08-19.md`
3. `VISION.md`
4. `ROADMAP.md`
5. `DECISIONS.md`
6. `PROBLEMS_AND_SOLUTIONS.md`
7. `LATE_DISCOVERED_ISSUES.md`
8. `CHANGELOG.md`
9. `BLOCKERS.md`

## الهوية

- المنتج: DBL Employee AI
- المظلة: Digital Blueprint Lab (DBL)
- التطبيق: `elias-mujally/dbl-employee-ai`
- الإنتاج: `https://dbl-employee-ai.vercel.app`
- الذاكرة: `elias-mujally/dbl-product-memory`

## الوصف الحالي في سطر واحد

DBL يتجه من موظف AI يركز على المحادثة إلى **موظف تشغيلي موثوق** يفهم الطلب، يقرأ أنظمة النشاط، يجهز فعلًا تجاريًا، يطبق سياسات deterministic، يطلب موافقة عند الحاجة، ينفذ، يتحقق، ويسجل النتيجة.

> هذه North Star قيد التحقق، وليست تصريحًا لبناء منصة Execution عامة الآن. أول Wedge مقترح هو Commerce / Sales Execution، وSalla أول Provider محتمل إذا توفر الوصول الرسمي.

## لماذا تغير الاتجاه

المنتج القديم كان قابلًا للبناء لكنه ضعيف دفاعيًا إذا ظلت القيمة الأساسية:

`AI replies + WhatsApp + Knowledge + automation`

المنافسون ومنصة Meta نفسها يضغطون طبقة المحادثة. لذلك القرار الحالي:

> Meta قناة وليست المنتج. Salla Provider أول وليست هوية المنتج. القيمة التي يجب أن يملكها DBL هي orchestration + policy + approval + execution + verification + reconciliation + audit.

اقرأ `STRATEGY_PIVOT_2026-08-19.md` قبل اقتراح CRM/ERP/omnichannel/workflow builder أو توسع عام.

## الحالة التنفيذية المختصرة

- آخر functional WhatsApp merge هو PR #42 عند `ede862e60bf1ace67a1e3fbb49d1e3f3d4bc7caf`.
- `main` في مستودع التطبيق تقدم بعد ذلك بcommits توثيقية فقط؛ تحقق من GitHub قبل الاعتماد على SHA محفوظ هنا.
- PR #43 `codex/test-whatsapp-scope-cleanup` ما يزال Draft وغير مدمج عند آخر تحقق، head `b9b4c0ed490aeeba0c4dc65fb5e4eaaf48661396`، وقاعدته أقدم من main الحالي؛ لا تدمجه دون إعادة التحقق والمراجعة.
- اتصال WhatsApp الحالي في Production ثبت أنه Test WABA/Test Number وبياناته التشغيلية Test-only.
- الرقم الحقيقي مختلف ومسجل في WhatsApp Business App؛ Standard Embedded Signup ليس المسار الصحيح له حاليًا.
- Coexistence هو الاتجاه المفضل معماريًا إذا ثبتت أهلية Meta، لكن لا يبدأ تنفيذ live onboarding قبل إزالة/تنظيف test scope بأمان أو اتخاذ قرار أحدث.
- Meta webhook ما يزال على legacy Cloud Run، والرد الثابت القديم مشكلة تشغيلية منفصلة. Webhook cutover = NO-GO حتى اتصال Production الحقيقي وmodern credential readiness.
- Contacts PR 2 وPR C لا يعودان أولوية المنتج الحالية ما لم يثبت أنهما prerequisite مباشر للـvalidation wedge.

## الوضع الاستراتيجي الحالي

`Planning freeze -> merchant validation + provider access/read spike -> evidence -> narrow build decision`

أهم الأسئلة الآن:

1. هل يوجد Merchant pain متكرر في تحويل محادثة/طلب إلى business action؟
2. هل Salla أو Provider أول يسمح بقراءة موثوقة وwrite منخفض المخاطر؟
3. ما أقل Action له قيمة اقتصادية ويمكن وضع Human Approval قبله؟
4. هل يمكن اختبار الفرضية دون انتظار Meta؟

## قواعد الثقة

1. لا تفترض أن PR مدمج لأن تقريرًا قال “جاهز”. تحقق من GitHub.
2. لا تفترض أن Production يطابق local branch.
3. Vercel canonical Production alias هو المرجع للتطبيق المنشور.
4. لا تغيّر Production/Meta/Supabase data دون موافقة صريحة.
5. لا تضع secrets في commits أو logs أو الذاكرة.
6. عند تعارض الاختبارات مع الاستخدام الفعلي، دليل Production له الأولوية في التشخيص.
7. Product Memory توثق القرار والسياق؛ التطبيق ومصادر Production هي مصدر الحقيقة للتنفيذ.
8. Merchant evidence يسبق تقارير مستشارين إضافية عندما يكون السؤال Product-Market Fit.

## قواعد هندسية ثابتة

- Forward-only migrations.
- لا تعديل migration تاريخية.
- RLS وworkspace isolation لا تُضعف.
- SECURITY DEFINER يحتاج empty/safe search_path وfully qualified references.
- browser لا يحدد actor/workspace/provider authority.
- deterministic policy تفرض الأموال والصلاحيات والحالات الحرجة؛ الـLLM لا يمنح نفسه صلاحية من نص Knowledge.
- أي external write يحتاج ambiguous/unknown state وreconciliation قبل retry.
- reuse architectural pattern، لا تنسخ implementation خاص بواتساب بلا دليل.
- implement first, abstract second.
- Human approval قبل autonomy.

## أسلوب تنفيذ PRs الكبيرة

1. Audit/read-only إذا كان السبب غير واضح.
2. Draft PR ضيق.
3. Full validation.
4. Adversarial self-review.
5. إصلاح High/Medium.
6. Independent final review.
7. Squash merge مع expected head SHA.
8. Production verification منفصلة.
9. تحديث Product Memory.

## ما لا ينبغي فعله الآن

- لا تبنِ CRM أو ERP داخل DBL.
- لا تبنِ universal execution platform أو policy DSL.
- لا تضف قنوات جديدة لمجرد أن الرؤية طويلة المدى omnichannel.
- لا توسع multi-agent أو marketplace أو model routing.
- لا تجعل Salla تبعية وجودية جديدة.
- لا تعتبر WhatsApp شرطًا لاختبار الـexecution hypothesis؛ استخدم web/internal/Wizard-of-Oz إذا لزم.
- لا تغيّر Test WABA identifiers إلى الرقم الحقيقي مباشرة.
- لا تعمل webhook cutover قبل اتصال Production حقيقي modern/readiness-verified.
- لا تنفذ cleanup Production من PR #43 لمجرد دمج migration؛ التنفيذ يحتاج إذنًا وتشغيلًا منفصلًا.

## أسئلة يجب طرحها عند أي قرار جديد

- ما Business outcome الذي يتحسن؟
- هل هناك Merchant evidence أم مجرد منطق نظري؟
- هل هذا prerequisite للفرضية الحالية أم feature جانبية؟
- هل يمكن اختباره يدويًا أو شبه يدوي قبل البناء الكامل؟
- هل Provider API يسمح بالفعل المطلوب بأمان؟
- هل الفعل reversible أو يحتاج reconciliation؟
- هل Human Approval واضح؟
- هل القرار يزيد lock-in مع منصة خارجية؟
- هل نحن نعيد نقاشًا حُسم دون دليل جديد؟

## تحديث الذاكرة

- `CURRENT_STATUS.md`: نقطة الاستئناف الحالية.
- `ROADMAP.md`: ترتيب العمل الفعلي.
- `VISION.md`: North Star بعيدة المدى.
- `DECISIONS.md`: القرارات الدائمة.
- `BLOCKERS.md`: العوائق الحالية فقط.
- `CHANGELOG.md`: المراحل التاريخية.
- `STRATEGY_PIVOT_2026-08-19.md`: يُحدّث فقط إذا تغيرت الفرضية الاستراتيجية بدليل جديد.
