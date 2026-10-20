---
description: "فهرست وظایف پیاده‌سازی وبلاگ با ایجنت‌های نویسنده و بازبین"
---

# Tasks: وبلاگ با ایجنت‌های نویسنده و بازبین

**Input**: اسناد طراحی در `specs/001-ai-blog-agents/`

**Prerequisites**: `plan.md` و `spec.md`؛ همچنین `research.md`، `data-model.md`، `quickstart.md` و `contracts/openapi.yaml`

**Tests**: آزمون‌های پذیرش در مشخصات محصول الزامی‌اند؛ آزمون‌ها در وظایف زیر نوشته می‌شوند، اما در این مرحله اجرا نمی‌شوند.

**Organization**: وظایف بر اساس داستان کاربری و اولویت P1 سپس P2 مرتب شده‌اند.

**Path conventions**: کد مطابق ساختار `plan.md` در `src/main/java/<base-package>/` و آزمون‌ها در `src/test/java/<base-package>/` قرار می‌گیرند.

## قالب وظایف

هر وظیفه چک‌باکس، شناسه ترتیبی، نشانگر اختیاری موازی‌سازی و برای وظایف داستانی برچسب داستان دارد. هر شرح، مسیر فایل هدف را مشخص می‌کند.

## فاز ۱: آماده‌سازی پروژه

**هدف**: ساخت اسکلت Maven و پیکربندی اجرای محلی Java 17 و Spring Boot 3.

- [ ] T001 ایجاد `pom.xml` با Spring Web، Spring Data JPA، Spring Security، H2، springdoc-openapi و وابستگی‌های آزمون JUnit 5، Spring Boot Test، MockMvc و ArchUnit.
- [ ] T002 ایجاد نقطه ورود برنامه و ساختار بسته‌ها مطابق `plan.md` در `src/main/java/<base-package>/BlogApplication.java` و پوشه‌های `article/`، `agent/`، `security/` و `shared/web/`.
- [ ] T003 [P] پیکربندی H2 محلی، حالت پیش‌فرض `AGENT_MODE=MOCK` و پروفایل اختیاری PostgreSQL در `src/main/resources/application.yml` و `src/main/resources/application-postgres.yml`.

## فاز ۲: زیرساخت پایه

**هدف**: فراهم‌کردن مدل ماندگار، قراردادهای ایجنت و زیرساخت مشترکی که همه داستان‌ها به آن وابسته‌اند.

- [ ] T004 [P] تعریف enum وضعیت مقاله دقیقاً با مقادیر `DRAFT`، `IN_REVIEW`، `NEEDS_CHANGES`، `APPROVED` و `PUBLISHED` و enumهای تغییر نسخه و نتیجه اجرا در `src/main/java/<base-package>/article/domain/` و `src/main/java/<base-package>/agent/contract/`.
- [ ] T005 [P] پیاده‌سازی موجودیت `Article` با `id` یکتا، `slug` یکتا، `title` و `excerpt` غیرخالی، `bodyMarkdown`، `tags`، `status`، `currentVersion` عدد صحیح مثبت و زمان‌های ایجاد/ویرایش/انتشار در `src/main/java/<base-package>/article/domain/Article.java`.
- [ ] T006 [P] پیاده‌سازی snapshot تغییرناپذیر `ArticleRevision` با `version` مثبت و یکتا برای هر مقاله، محتوای کامل نسخه، زمان ایجاد و `changeSource` در `src/main/java/<base-package>/article/domain/ArticleRevision.java`.
- [ ] T007 [P] پیاده‌سازی `ReviewReport` و `Finding` با verdictهای `APPROVED`/`NEEDS_CHANGES`، امتیاز صحیح ۰ تا ۱۰۰، شدت `HIGH`/`MEDIUM`/`LOW` و فیلدهای الزامی location، issue و suggestion در `src/main/java/<base-package>/article/domain/ReviewReport.java` و `Finding.java`.
- [ ] T008 [P] پیاده‌سازی `AgentRun` با نوع `WRITER`/`REVIEWER`، مقاله و نسخه ورودی تهی‌پذیر، نام مدل بدون راز، نتیجه `SUCCESS`/`FAILURE`، مدت نامنفی و زمان اجرا در `src/main/java/<base-package>/agent/contract/AgentRun.java`.
- [ ] T009 تعریف repositoryهای JPA برای مقاله، نسخه، گزارش و اجرای ایجنت، همراه قید یکتایی slug و شماره نسخه در هر مقاله در `src/main/java/<base-package>/article/persistence/` و `src/main/java/<base-package>/agent/persistence/`.
- [ ] T010 [P] تعریف قراردادهای مستقل ورودی/خروجی برای نویسنده و بازبین؛ بازبین فقط snapshot صریح نسخه را دریافت کند و هیچ وابستگی به repository مقاله نداشته باشد در `src/main/java/<base-package>/agent/contract/`.
- [ ] T011 ایجاد قالب پاسخ خطا و نگاشت خطاهای ورودی، مورد ناموجود، تعارض وضعیت و خطای سرویس مدل به پاسخ‌های کنترل‌شده 400، 404، 409 و 502 در `src/main/java/<base-package>/shared/web/`.
- [ ] T012 پیاده‌سازی سیاست گذار وضعیت بر اساس جدول `FR-014`؛ هر گذار تعریف‌نشده 409 بدهد و خطای مدل، محتوا و وضعیت معتبر پیشین را حفظ کند در `src/main/java/<base-package>/article/domain/ArticleStatusPolicy.java`.
- [ ] T013 [P] پیکربندی springdoc و مستندات API از قرارداد OpenAPI در `src/main/java/<base-package>/shared/web/OpenApiConfiguration.java`.

## فاز ۳: داستان ۱ - ساخت پیش‌نویس و مدیریت مقاله (P1، MVP)

**هدف**: مدیر با موضوع معتبر پیش‌نویس DRAFT نسخه ۱ تولید کند و مقاله را به‌شکل نسخه‌دار ویرایش کند.

**آزمون مستقل**: در MOCK پیش‌نویس کامل بگیرید، ورودی خالی و پاسخ ناقص نویسنده را رد کنید و پس از ویرایش، نسخه جدید و snapshot قبلی را مشاهده کنید.

### آزمون‌های داستان ۱

- [ ] T014 [P] [US1] آزمون پذیرش تولید MOCK پیش‌نویس کامل در DRAFT نسخه ۱ و نشانگر `mock` در `src/test/java/<base-package>/article/GenerateArticleAcceptanceTest.java`.
- [ ] T015 [P] [US1] آزمون پذیرش رد موضوع خالی/فاصله‌ای و عدم ذخیره مقاله در `src/test/java/<base-package>/article/GenerateArticleValidationTest.java`.
- [ ] T016 [P] [US1] آزمون پذیرش رد خروجی ناقص نویسنده و عدم ایجاد مقاله یا نسخه در `src/test/java/<base-package>/article/InvalidWriterOutputTest.java`.
- [ ] T017 [P] [US1] آزمون پذیرش ویرایش محتوایی، افزایش شماره نسخه و حفظ snapshot نسخه قبل در `src/test/java/<base-package>/article/ArticleRevisionAcceptanceTest.java`.

### پیاده‌سازی داستان ۱

- [ ] T018 [P] [US1] پیاده‌سازی `WritingAgent` شبیه‌سازی‌شده با خروجی قطعی دارای عنوان، خلاصه، Markdown و برچسب‌ها در `src/main/java/<base-package>/agent/writing/MockWritingAgent.java`.
- [ ] T019 [US1] پیاده‌سازی اعتبارسنجی خروجی نویسنده و کاربرد تولید اتمیک مقاله و `ArticleRevision` نسخه ۱ با مقداردهی تمام فیلدهای الزامی در `src/main/java/<base-package>/article/application/GenerateArticleService.java`.
- [ ] T020 [US1] پیاده‌سازی DTOهای درخواست/پاسخ و endpointهای `POST /api/admin/articles/generate`، `GET /api/admin/articles` و `GET /api/admin/articles/{id}` در `src/main/java/<base-package>/article/web/AdminArticleController.java`.
- [ ] T021 [US1] پیاده‌سازی ویرایش فقط برای مقاله غیرمنتشرشده؛ هر تغییر محتوایی دقیقاً یک snapshot کامل تازه بسازد و ویرایش مقاله PUBLISHED را با 409 رد کند در `src/main/java/<base-package>/article/application/EditArticleService.java`.
- [ ] T022 [US1] پیاده‌سازی endpointهای `PUT /api/admin/articles/{id}` و `GET /api/admin/articles/{id}/revisions` با اعتبارسنجی `title`، `excerpt` و `bodyMarkdown` غیرخالی و برچسب‌های ساختاریافته در `src/main/java/<base-package>/article/web/AdminArticleController.java`.

## فاز ۴: داستان ۲ - بازبینی و اصلاح با اختیار مدیر (P1)

**هدف**: بازبینی فقط با درخواست مدیر روی snapshot مشخص اجرا شود؛ اصلاح نویسنده نیز فقط با درخواست صریح مدیر رخ دهد.

**آزمون مستقل**: گزارش به نسخه ورودی متصل باشد و محتوا را تغییر ندهد؛ NEEDS_CHANGES اجرای خودکار نویسنده نداشته باشد؛ درخواست اصلاح مدیر نسخه تازه بسازد.

### آزمون‌های داستان ۲

- [ ] T023 [P] [US2] آزمون پذیرش گزارش بازبینی ساختاریافته و اتصال آن به شماره نسخه ورودی در `src/test/java/<base-package>/agent/ReviewArticleAcceptanceTest.java`.
- [ ] T024 [P] [US2] آزمون پذیرش عدم تغییر عنوان، خلاصه، متن، برچسب‌ها و وضعیت مقاله در اثر بازبینی در `src/test/java/<base-package>/agent/ReviewIsReadOnlyTest.java`.
- [ ] T025 [P] [US2] آزمون پذیرش عدم اجرای نویسنده پس از NEEDS_CHANGES تا درخواست صریح مدیر در `src/test/java/<base-package>/agent/NoAutomaticRevisionTest.java`.
- [ ] T026 [P] [US2] آزمون پذیرش اصلاح صریح با بازخورد مدیر، ایجاد نسخه تازه و نامعتبرشدن تأیید نسخه قبلی در `src/test/java/<base-package>/agent/ExplicitRevisionAcceptanceTest.java`.
- [ ] T027 [P] [US2] آزمون قطعی بازبین MOCK که `[NEEDS_FIX]` را NEEDS_CHANGES و سایر متن‌ها را APPROVED می‌کند در `src/test/java/<base-package>/agent/MockReviewAgentTest.java`.

### پیاده‌سازی داستان ۲

- [ ] T028 [P] [US2] پیاده‌سازی `ReviewAgent` شبیه‌سازی‌شده با verdict قطعی، امتیاز و داده‌های ساختاریافته معتبر در `src/main/java/<base-package>/agent/review/MockReviewAgent.java`.
- [ ] T029 [US2] پیاده‌سازی سرویس بازبینی که فقط snapshot نسخه فعلی را به قرارداد بازبین می‌دهد، گزارش معتبر را ذخیره می‌کند و گزارش را به همان نسخه متصل می‌سازد در `src/main/java/<base-package>/article/application/ReviewArticleService.java`.
- [ ] T030 [US2] پیاده‌سازی اعتبارسنجی کامل خروجی بازبین و نگاشت verdict به وضعیت APPROVED یا NEEDS_CHANGES؛ خطای مدل گزارش موفق نسازد و متن را تغییر ندهد در `src/main/java/<base-package>/agent/review/ReviewResultValidator.java`.
- [ ] T031 [US2] پیاده‌سازی اصلاح درخواستی مدیر با بازخورد صریح؛ ورودی شامل snapshot جاری و بازخورد باشد و خروجی معتبر یک نسخه تازه DRAFT بسازد در `src/main/java/<base-package>/article/application/ReviseArticleService.java`.
- [ ] T032 [US2] پیاده‌سازی endpointهای `POST /api/admin/articles/{id}/review`، `GET /api/admin/articles/{id}/reviews` و `POST /api/admin/articles/{id}/revise` با پاسخ‌های قرارداد OpenAPI در `src/main/java/<base-package>/article/web/AdminArticleController.java`.

## فاز ۵: داستان ۳ - انتشار امن و مشاهده عمومی (P1)

**هدف**: انتشار فقط با اقدام دستی مدیر و گزارش APPROVED معتبر برای نسخه فعلی ممکن باشد؛ لغو انتشار دسترسی عمومی را فوراً ببندد.

**آزمون مستقل**: انتشار بدون تأیید یا پس از تغییر نسخه رد شود؛ نسخه تأییدشده فعلی پس از اقدام مدیر عمومی شود؛ لغو انتشار آن را از فهرست و جزئیات حذف کند.

### آزمون‌های داستان ۳

- [ ] T033 [P] [US3] آزمون پذیرش رد انتشار بدون گزارش APPROVED برای نسخه فعلی و حفظ عدم‌انتشار در `src/test/java/<base-package>/article/PublishApprovalGuardTest.java`.
- [ ] T034 [P] [US3] آزمون پذیرش نامعتبرشدن تأیید نسخه ۱ پس از ویرایش به نسخه ۲ و رد انتشار نسخه ۲ در `src/test/java/<base-package>/article/StaleApprovalPublishTest.java`.
- [ ] T035 [P] [US3] آزمون پذیرش انتشار دستی نسخه تأییدشده و حذف فوری آن از API عمومی پس از لغو انتشار در `src/test/java/<base-package>/api/PublishUnpublishAcceptanceTest.java`.
- [ ] T036 [P] [US3] آزمون پذیرش عدم افشای وجود مقاله منتشرنشده در endpoint جزئیات عمومی در `src/test/java/<base-package>/api/UnpublishedArticlePrivacyTest.java`.
- [ ] T037 [P] [US3] آزمون پذیرش پاک‌سازی HTML یا اسکریپت ناامن از محتوای Markdown عمومی در `src/test/java/<base-package>/api/PublicMarkdownSanitizationTest.java`.

### پیاده‌سازی داستان ۳

- [ ] T038 [US3] پیاده‌سازی انتشار و لغو انتشار با کنترل وضعیت و الزام آخرین گزارش APPROVED مربوط به `currentVersion` در `src/main/java/<base-package>/article/application/PublishArticleService.java`.
- [ ] T039 [US3] پیاده‌سازی خواندن عمومی فقط از مقاله‌های PUBLISHED و پاسخ یکسان 404 برای slug مقاله منتشرنشده یا ناموجود در `src/main/java/<base-package>/article/application/PublicArticleQueryService.java`.
- [ ] T040 [US3] پیاده‌سازی پاک‌سازی HTML ناامن متن Markdown در مرز پاسخ عمومی در `src/main/java/<base-package>/article/web/PublicMarkdownSanitizer.java`.
- [ ] T041 [US3] پیاده‌سازی endpointهای `POST /api/admin/articles/{id}/publish`، `POST /api/admin/articles/{id}/unpublish`، `GET /api/public/articles` و `GET /api/public/articles/{slug}` در `src/main/java/<base-package>/article/web/`.

## فاز ۶: داستان ۴ - کنترل مدیریتی و پایداری گردش کار (P2)

**هدف**: عملیات مدیریتی محافظت‌شده باشد، تاریخچه اجرای ایجنت قابل مشاهده باشد و خطای مدل هیچ داده یا مجوز معتبری را خراب نکند.

**آزمون مستقل**: توکن غایب/نامعتبر رد شود؛ توکن معتبر گزارش‌ها و اجرای ایجنت را ببیند؛ خطاها بدون افشای راز ثبت شوند و متن و وضعیت تأییدنشده حفظ شود؛ تنظیمات دو ایجنت مستقل بماند.

### آزمون‌های داستان ۴

- [ ] T042 [P] [US4] آزمون پذیرش پاسخ 401 برای درخواست‌های مدیریتی بدون توکن یا با توکن نادرست و موفقیت توکن معتبر در `src/test/java/<base-package>/api/AdminAuthenticationTest.java`.
- [ ] T043 [P] [US4] آزمون پذیرش خطای قطع ارتباط/timeout/محدودیت نرخ/پاسخ خالی یا نامعتبر با حفظ محتوا، وضعیت قبلی و عدم APPROVED/PUBLISHED در `src/test/java/<base-package>/agent/AgentFailureSafetyTest.java`.
- [ ] T044 [P] [US4] آزمون پذیرش ثبت تاریخچه WRITER و REVIEWER با مقاله، نسخه ورودی، مدل، نتیجه، مدت و زمان و بدون راز در `src/test/java/<base-package>/agent/AgentRunHistoryTest.java`.
- [ ] T045 [P] [US4] آزمون پذیرش جدایی تنظیمات و دستورالعمل‌های نویسنده و بازبین، اجرای MOCK بدون کلید و خطای روشن REAL با پیکربندی ناقص در `src/test/java/<base-package>/agent/AgentConfigurationIsolationTest.java`.
- [ ] T046 [P] [US4] آزمون معماری ArchUnit برای ممنوعیت وابستگی مستقیم agent.writing به agent.review و دسترسی ReviewAgent به persistence مقاله در `src/test/java/<base-package>/architecture/AgentBoundaryTest.java`.

### پیاده‌سازی داستان ۴

- [ ] T047 [US4] پیاده‌سازی Bearer-token filter بر پایه `ADMIN_API_TOKEN` برای تمام مسیرهای `/api/admin/**` و آزادگذاشتن مسیرهای عمومی در `src/main/java/<base-package>/security/`.
- [ ] T048 [US4] تعریف تنظیمات جداگانه Writer و Reviewer شامل endpoint، model، API key، timeout پیش‌فرض ۳۰ ثانیه و system prompt/version مستقل در `src/main/java/<base-package>/agent/writing/`، `src/main/java/<base-package>/agent/review/` و `src/main/resources/application.yml`.
- [ ] T049 [P] [US4] پیاده‌سازی adapter واقعی نویسنده برای API سازگار با OpenAI با timeout، خطای تنظیمات ناقص و حذف کلید از پیام‌ها در `src/main/java/<base-package>/agent/writing/OpenAiWritingAgent.java`.
- [ ] T050 [P] [US4] پیاده‌سازی adapter واقعی بازبین به‌صورت مستقل با قرارداد خروجی ساختاریافته، timeout و حذف کلید از پیام‌ها در `src/main/java/<base-package>/agent/review/OpenAiReviewAgent.java`.
- [ ] T051 [US4] پیاده‌سازی ثبت موفقیت/شکست، نوع ایجنت، شناسه مقاله، نسخه ورودی، نام مدل، مدت و زمان؛ جلوگیری از ثبت کلید، توکن، prompt حساس و stack trace در `src/main/java/<base-package>/agent/orchestration/AgentRunRecorder.java`.
- [ ] T052 [US4] پیاده‌سازی endpoint مدیریتی تاریخچه اجرای مقاله `GET /api/admin/articles/{id}/agent-runs` و DTO امن آن مطابق فیلدهای `AgentRun` در `src/main/java/<base-package>/agent/web/AdminAgentRunController.java` و `specs/001-ai-blog-agents/contracts/openapi.yaml`.
- [ ] T053 [US4] یکپارچه‌سازی خطاهای اتصال، timeout، محدودیت نرخ، پاسخ خالی و نامعتبر با پاسخ 502 قابل‌فهم و تراکنش‌هایی که محتوای ذخیره‌شده یا مجوز انتشار را تغییر ندهند در `src/main/java/<base-package>/agent/orchestration/` و `src/main/java/<base-package>/shared/web/`.

## فاز ۷: داستان ۵ - دسترسی خواندنی بازدیدکننده (P2)

**هدف**: خواننده بدون ورود فهرست و جزئیات مقاله‌های منتشرشده را با slug یکتا دریافت کند.

**آزمون مستقل**: فهرست تنها PUBLISHEDها را برگرداند، مقاله‌های هم‌عنوان slug یکتا بگیرند و جزئیات منتشرنشده وجود مقاله را فاش نکند.

### آزمون‌های داستان ۵

- [ ] T054 [P] [US5] آزمون پذیرش یکتایی slug برای عنوان تکراری با پسوندهای `-2`، `-3` و مانند آن در `src/test/java/<base-package>/article/UniqueSlugAcceptanceTest.java`.
- [ ] T055 [P] [US5] آزمون پذیرش slug شامل حروف کوچک، خط تیره برای فاصله، حذف نویسه غیرمجاز و حفظ حروف فارسی در `src/test/java/<base-package>/article/SlugNormalizationTest.java`.
- [ ] T056 [P] [US5] آزمون پذیرش فهرست عمومی فقط شامل PUBLISHED و جزئیات مقاله منتشرشده شامل عنوان، خلاصه، Markdown پاک‌سازی‌شده، برچسب‌ها و slug در `src/test/java/<base-package>/api/PublicArticleReadAcceptanceTest.java`.

### پیاده‌سازی داستان ۵

- [ ] T057 [US5] پیاده‌سازی ساخت slug مطابق حروف کوچک، تبدیل فاصله به خط تیره، حذف نویسه‌های غیرمجاز با حفظ حروف فارسی و تخصیص suffix عددی هنگام برخورد در `src/main/java/<base-package>/article/domain/SlugGenerator.java`.
- [ ] T058 [US5] افزودن قید و بازیابی برخورد یکتایی slug در لایه persistence برای جلوگیری از slug تکراری حتی هنگام ساخت هم‌زمان مقاله در `src/main/java/<base-package>/article/persistence/`.
- [ ] T059 [US5] تکمیل DTO عمومی فهرست/جزئیات و مرتب‌سازی یا فیلتر خروجی به‌گونه‌ای که فقط PUBLISHEDها از query عمومی عبور کنند در `src/main/java/<base-package>/article/application/PublicArticleQueryService.java` و `src/main/java/<base-package>/article/web/PublicArticleController.java`.

## فاز نهایی: پرداخت نهایی و مستندسازی

**هدف**: راه‌اندازی MOCK، عملیات API، پیکربندی امن و قرارداد مستند برای استفاده و تحویل قابل مشاهده باشند.

- [ ] T060 [P] به‌روزرسانی `specs/001-ai-blog-agents/contracts/openapi.yaml` برای endpoint تاریخچه اجرای ایجنت، طرح پاسخ امن و پاسخ‌های خطای مرتبط با پیاده‌سازی.
- [ ] T061 [P] تکمیل `README.md` با پیش‌نیاز Java 17، اجرای محلی بدون Docker در حالت MOCK، تنظیم متغیرهای نمونه بدون راز واقعی و راهنمای حالت REAL.
- [ ] T062 [P] مستندسازی تمام endpointهای عمومی و مدیریتی، قراردادهای ورودی/خروجی، Bearer token و پاسخ‌های 400/401/404/409/502 در `README.md` و پیکربندی springdoc.
- [ ] T063 بازبینی `src/main/resources/application.yml`، مستندات نمونه و logging برای اطمینان از نبود توکن یا API key واقعی و ثبت‌نشدن رازها در پاسخ، خطا و تاریخچه.
- [ ] T064 تکمیل پوشش ۱۵ آزمون پذیرش ضروری PRD در مجموعه آزمون‌های `src/test/java/<base-package>/` برای اجرای آفلاین و MOCK؛ اجرای آن‌ها پس از پیاده‌سازی طبق دستور `mvn test` در `specs/001-ai-blog-agents/quickstart.md`.

## وابستگی‌ها و ترتیب اجرا

### وابستگی فازها

- آماده‌سازی (فاز ۱) پیش‌نیاز زیرساخت پایه است.
- زیرساخت پایه (فاز ۲) پیش‌نیاز شروع همه داستان‌ها است.
- داستان‌های P1 به ترتیب قابل تحویل‌اند: US1، سپس US2 و US3؛ US2 و US3 برای گردش کار کامل به مقاله نسخه‌دار US1 نیاز دارند.
- داستان US4 به قراردادهای agent و مقاله/گزارش ایجادشده در US1 و US2 برای آزمون تاریخچه و خطا متکی است.
- داستان US5 از API انتشار US3 استفاده می‌کند و پس از آن تکمیل می‌شود.
- پرداخت نهایی به داستان‌هایی وابسته است که قرار است در تحویل گنجانده شوند.

### نمودار وابستگی داستان‌ها

```text
Setup → Foundational → US1 → US2 → US3 → US5
                            └────────→ US4
```

### فرصت‌های موازی‌سازی

- پس از `T001` و `T002`، پیکربندی `T003` می‌تواند موازی انجام شود.
- وظایف مدل‌های مستقل `T004` تا `T008` و آزمون‌های مستقل هر داستان با نشان `[P]` قابل موازی‌سازی‌اند.
- پس از آماده‌شدن زیرساخت پایه، آزمون‌های هر فاز داستانی مستقل از هم نوشته می‌شوند؛ پیاده‌سازی آزمون‌ها را می‌توان موازی کرد، ولی تکمیل داستان‌ها طبق وابستگی‌های بالا یکپارچه می‌شود.
- نمونه اجرای موازی US1: `T014`، `T015`، `T016` و `T017` هرکدام فایل آزمون جدا دارند؛ پس از آن پیاده‌سازی mock در `T018` از سرویس تولید `T019` مستقل است.
- نمونه اجرای موازی US4: adapter نویسنده `T049` و adapter بازبین `T050` فایل‌ها و تنظیمات مستقل دارند.

## راهبرد پیاده‌سازی

### MVP

1. تکمیل فاز آماده‌سازی و زیرساخت پایه.
2. تکمیل US1: تولید MOCK، اعتبارسنجی، ویرایش، نسخه‌ها و آزمون‌های پذیرش آن.
3. بررسی مستقل گردش تولید و ویرایش؛ محدوده MVP اولیه همین داستان است.

### تحویل افزایشی

1. US1 پیش‌نویس نسخه‌دار را فراهم می‌کند.
2. US2 بازبینی متصل به نسخه و اصلاح صریح مدیر را اضافه می‌کند.
3. US3 انتشار امن دستی و خواندن عمومی اولیه را فراهم می‌کند.
4. US4 احراز هویت، تاریخچه اجرا، تنظیمات مستقل و مقاومت در برابر خطا را کامل می‌کند.
5. US5 یکتایی slug و تجربه خواندن عمومی را تکمیل می‌کند.
6. در پایان، مستندات و پوشش کامل ۱۵ آزمون پذیرش تکمیل می‌شوند.

## معیارهای مستقل داستان‌ها

- **US1**: تولید MOCK پیش‌نویس معتبر، رد ورودی/خروجی نامعتبر، ویرایش و حفظ snapshot قبلی.
- **US2**: گزارش معتبر برای snapshot مشخص، عدم تغییر مقاله توسط بازبین، عدم اصلاح خودکار و ایجاد نسخه با اصلاح صریح.
- **US3**: رد انتشار بدون تأیید نسخه جاری، انتشار دستی پس از تأیید، لغو انتشار و عدم افشای مقاله پنهان یا اجرای HTML ناامن.
- **US4**: رد دسترسی بدون Bearer معتبر، مشاهده اجرای ایجنت بدون راز و حفظ محتوا/وضعیت هنگام خطای سرویس.
- **US5**: فهرست فقط شامل منتشرشده‌ها، slug نرمال و یکتا و پاسخ عمومی کامل برای مقاله منتشرشده.

## اعتبارسنجی قالب

شناسه‌ها از T001 تا T064 پیوسته‌اند. تمام وظایف با `- [ ] Tnnn` آغاز می‌شوند؛ وظایف داستانی برچسب `[USn]` و تمام شرح‌ها مسیر فایل دارند. نشانگر `[P]` فقط برای فایل‌ها/کارهای مستقل درج شده است.
