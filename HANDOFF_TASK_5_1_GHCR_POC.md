# Task 5.1 — تسليم إثبات مفهوم سجل GHCR

**تاريخ التحديث:** 2026-10-03  
**الحالة:** جزئية؛ اختُبر نموذج OCI محليًا، ولم يُختبر GHCR مباشرة.  
**التفويض:** يُسمح بـPoC مستقل لـGHCR؛ يُعزل عن Titan/Titan-3 وتُستخدم فيه بيانات اصطناعية فقط.

## الحالة الحالية
اجتازت فحوص نموذج OCI/layout المحلية، لكن لم تُنشأ أي حزمة GHCR ولم تُرفع أو تُسحب. الأدلة الحالية غير كافية لإثبات الملاءمة للإنتاج؛ والتصنيف النهائي المؤقت هو D — التجربة غير كافية ويلزم اختبار موجّه آخر.

## ما اختُبر محليًا
أنشأ سكربت مؤقت بـNode.js يدويًا OCI Image Layout v1.0.0 داخل `/tmp`، ثم أزاله بعد الاختبار. لم تتغير ملفات مصدر Titan/المشروع.
- أُنشئ artifact واحد للجذر واثنان للشظايا؛ وقُرئت 3 أغلفة manifest من OCI.
- نوع وسائط manifest: `application/vnd.oci.image.manifest.v1+json`. أنواع artifact المخصصة: `application/vnd.titan.registry.root.v1+json` و`application/vnd.titan.registry.shard.v1+json`. نوع Config: `application/vnd.oci.empty.v1+json`. أُدرجت تعليقات `ref-name` للجذر/الشظايا.
- أشار payload الجذر إلى manifests الشظايا باستخدام digest. اجتازت محليًا 16 عملية تحقق من digest/size للواصفات.
- قرأ lookup اصطناعي مفهرس شظية واحدة محددة من أصل اثنتين؛ وعُثر على record نشط، وعُرض tombstone كحالة tombstone.
- حُسب hash لـblob اصطناعي بحجم 100 KiB وتحققت صحته.
- حُوكي Retry/dedup في الذاكرة: إخفاق واحد قبل commit، ثم إعادة المحاولة وإعادة التشغيل؛ digest ثابت وإيصال واحد. هذا سلوك لنموذج publisher وليس دليلًا على GHCR.
- لم تُستخدم ORAS أو أداة عميل registry، ولم تُختبر طبقة مضغوطة أو عملية pull فعلية أو Registry API.

## النتائج وحدود الأدلة
| الموضوع | الأدلة الحالية | الحالة |
|---|---|---|
| بنية OCI والواصفات وأنواع media type المخصصة | layout محلي أُنشئ يدويًا فقط | CONFIRMED محليًا؛ قبول GHCR يحتاج تجربة NEEDS EXPERIMENT |
| digests من root إلى shards وlookup انتقائي | قُرئت شظية محلية واحدة من اثنتين | CONFIRMED في النموذج؛ مسار pull البعيد غير مختبر |
| ثبات SHA-256 | حساب hash للمحتوى محليًا | CONFIRMED محليًا؛ سلوك GHCR في dedup/tag غير مختبر |
| Retry/idempotency والإيصالات | محاكاة داخل الذاكرة فقط | NEEDS EXPERIMENT على GHCR؛ لا تنسب سلوك publisher إلى GHCR |
| تحديثات root/generations المتزامنة | لم يُنفّذ | UNRESOLVED |
| pull عام مجهول الهوية/ظهور الحزمة | لا توجد حزمة PoC | FEASIBLE بحسب وثائق GitHub؛ لم يُؤكّد تجريبيًا هنا |
| الحجم وbatching | blob محلي بحجم 100 KiB فقط | أحجام/أزمنة النقل الفعلية من 100 KiB إلى 1 GiB ما زالت UNRESOLVED |
| tombstone والحذف والاحتفاظ | إسقاط محلي فقط | محو التخزين واستمراريته UNRESOLVED |
| Pages Explorer وCORS/proxy/cache | لم يُنفّذ | UNRESOLVED |
| GHCR بوصفه Registry كاملًا | ليس الدور الذي اختُبر؛ query وpolicy وidentity وprivacy وmoderation طبقات منفصلة | REJECTED لهذه المعمارية |
| publisher/relay خارجي | تحليل معماري فقط | FEASIBLE، لم يُنفّذ أو يُختبر |

## القيود التقنية وبيانات الاعتماد
- البيئة المرصودة: Node 24.13.0 وPython 3.13.11 وNix؛ إصدار Docker CLI هو 27.5.1 لكن Docker daemon غير متاح؛ وORAS غير موجود.
- يملك اتصال GitHub App صلاحية repo، لكنه لا يملك packages scope. أعاد استعراض حزم المستخدم في container packages استجابة 403 التي تتطلب `read:packages`، حتى بعد إعادة توصيل واحدة وإعادة المحاولة.
- لا يوجد secure registry credential path متاح، ولم يُستخدم أو يُخزّن أي token.
- تتطلب عمليات GHCR مسارًا آمنًا لاعتماد registry؛ وتوضح وثائق GitHub نطاقات classic PAT الخاصة بالحزم: `read:packages` للبيانات الوصفية/القراءة، و`write:packages` للرفع، و`delete:packages` للتنظيف. استخدم نطاقات الحزم المطلوبة فقط، وتجنّب نطاق repo الواسع إن أمكن.
- كان المستودع `WaheedFox/Titan-3` متاحًا بصلاحية push/admin، لكنه بقي دون تغيير.
- سبق الاطلاع على الوثائق الرسمية: يدعم GHCR الوصول العام المجهول ويوثّق حدًا أقصى قدره 10 GB لكل layer ومهلة رفع قدرها 10 دقائق؛ وهذه حدود موثقة وليست نتائج مقاسة. وتشير وثائق GitHub Pages إلى حد حجم موقع قدره 1 GB وحد نطاق ترددي مرن قدره 100 GB/month؛ ولم يُختبر التكامل بين GHCR وPages.

## النطاق المسموح والمحظورات
- تُستخدم بيانات اصطناعية فقط وحزمة مستقلة واحدة باسم فريد، مثل `ghcr.io/waheedfox/titan-ghcr-poc-<unique>`، إذا أصبح مسار اعتماد آمن متاحًا.
- يُحظر النشر إلى `WaheedFox/Titan` أو `Titan-3` أو تعديلهما. لا تغييرات على Titan source أو ADRs أو CONTRACT أو ROADMAP أو التحقيق الحالي؛ ولا بنية تحتية للإنتاج.
- أجاز المستخدم اختبار القراءة العامة المجهولة وتنظيف الحزمة التجريبية المُنشأة فقط. تُجعل عامة لهذا الاختبار وحده؛ وقد تبقى النسخ المنزّلة وcaches. لم تُنشأ أي حزمة حتى الآن.

## السياق المعماري
التحقيق القائم: `docs/internal/investigations/shared-community-message-links-registry.md` (لقطة المصدر `7c43f11627a06c4341bc8aab45ba0fc3ce9b8369`). يصف SQLite محلية لكل runtime، من دون نشر بعيد أو fallback لـ`/link`، ويعامل `titan_id` كمعرّف محلي لا عالمي فريد. تسجيل الهوية عبر `ctx.send()`/`ctx.reply()` يحدث بعد الإرسال وبأفضل جهد. لا يستطيع GitHub/GHCR إثبات تسليم Telegram أو فرض runtime ملتزم بالعقد.
يبقى الشكل الموصى به:
```text
Titan Runtime → هوية/outbox محلية دائمة → publisher/relay موثوق ومحايد تجاه المزوّد → تخزين GHCR (الشظايا/الفهارس/root) → query/Explorer/الخصوصية والإشراف المنفصلة
```
يجب إبقاء التسجيل، وقبول publisher، والنشر إلى registry، والتحقق من Telegram، والظهور العام أمورًا منفصلة. يجب أن تبقى إسقاطات PUBLIC / RESTRICTED / PRIVATE منفصلة. لا يثبت tombstone محو البيانات من blobs غير القابلة للتغيير أو التنزيلات أو caches.

## الخطوة التالية
إذا أصبح مسار Secrets آمن متاحًا في هذه البيئة، فاستخدم classic PAT قصير العمر مع نطاقات الحزم المطلوبة فقط، ثم اختبر حزمة معزولة وفريدة: push/pull حقيقي لـOCI وقبول media type؛ وpull مجهول الهوية؛ وسلوك retry للمحتوى ذي digest نفسه والمختلف؛ وتنافس publisherين على تحديث root؛ وlookup للشظية وحدها؛ وtombstone مقابل الحذف الفعلي للحزمة؛ وأحجام اصطناعية 100 KiB و1 MiB و10 MiB و100 MiB و1 GiB فقط إذا كان ذلك آمنًا وعمليًا. سجّل الطرق والأزمنة والإخفاقات. احذف الحزمة المنشأة لهذا الاختبار فقط. إذا ظل إدخال الاعتماد الآمن غير متاح، فتوقف قبل أي كتابة إلى registry وقدّم تقريرًا يقتصر على الاختبار المحلي.

## معايير الحكم النهائي
تنطبق خيارات A/B/C/D أدناه على صلاحية GHCR بوصفه طبقة تخزين فقط، ولا تصنّف صلاحية معمارية Community Registry نفسها.
- A — GHCR مناسب بوصفه طبقة التخزين العامة الأساسية.
- B — لا يصلح GHCR إلا طبقة تخزين محدودة/ثانوية.
- C — ينبغي رفض GHCR بوصفه طبقة تخزين.
- D — التجربة غير كافية؛ يلزم اختبار موجّه آخر.
تدعم الأدلة الحالية **D** فقط. يجب أن يميّز التقرير النهائي بين CONFIRMED وFEASIBLE وNEEDS EXPERIMENT وUNRESOLVED وREJECTED، وأن يقيّم كلًا على حدة: تخزين GHCR، وRegistry الكامل، وPages Explorer، وpublisher، وإسقاطات الظهور، ونموذج هوية registry.

## المراجع
- GHCR: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry
- GitHub Packages REST API/scopes: https://docs.github.com/en/rest/packages/packages?apiVersion=2022-11-28
- حدود GitHub Pages: https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits
- مرفق Task 5.1 الأصلي في مساحة العمل السابقة: `attached_assets/Pasted-Task-5-1-GHCR-Registry-Proof-of-Concept--1790964963253_1790964963260.txt`
