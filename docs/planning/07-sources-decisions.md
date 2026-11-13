# Дереккөздер және шешімдер тізілімі

Нақтылану күні: **2026-11-13**. Төмендегі ресми сілтемелер бастапқы зерттеу мен архитектура шолуында қолданылды; осы documentation-only кезеңі жаңа құқықтық/платформалық тексеру жүргізілді дегенді білдірмейді. Техникалық шешімдердің бір бөлігі пайдаланушы тарапынан бекітілді; нақты салон рәсімдері мен адамдардың өкілеті [бизнес болжамдарында](06-business-assumptions.md) бөлек ашық қалады.

## 1. Ресми дереккөздер

| Дереккөз | Жобадағы қолданылуы |
| --- | --- |
| [Next.js Data Security](https://nextjs.org/docs/app/guides/data-security) | Серверлік DAL, минимал DTO, әр mutation-дағы authorization |
| [Next.js Authentication](https://nextjs.org/docs/app/guides/authentication) | UI/route қорғауы мен дерек операциясын қорғаудың айырмасы |
| [Supabase SSR client](https://supabase.com/docs/guides/auth/server-side/creating-a-client?queryGroups=framework&framework=nextjs) | Cookie client, тексерілген identity, сессия және cache шекарасы |
| [Supabase Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security) | RLS, view қауіпсіздігі, auth scope |
| [Supabase Database Functions](https://supabase.com/docs/guides/database/functions) | Invoker/definer шекарасы, search_path және execute grants |
| [Supabase Regions](https://supabase.com/docs/guides/platform/regions) | Hosting аймақтарын тексеру; Қазақстан тізімде көрсетілмеген |
| [Supabase Backups](https://supabase.com/docs/guides/platform/backups) | Backup/PITR мүмкіндігін plan-мен тексеру, Storage объектілерінің бөлек болуы |
| [Әділет: дербес деректер туралы заң](https://old.adilet.zan.kz/rus/docs/Z1300000094) | Сақтау және трансшекаралық беру талаптарын заңгермен нақтылау қажеттілігі |

Құжаттардың нұсқасы және тариф/region мүмкіндіктері implementation/deployment кезінде қайта тексеріледі. Заң мәтініне тікелей қол жеткізу шектелгені және индекстелген ресми үзінділер қолданылғаны [қауіпсіздік құжатында](04-permissions-security.md#дербес-деректер-және-орналастыру) көрсетілген. Заңгердің тексеруі аяқталды деген тұжырым жасалмайды.

## 2. Архитектуралық шешімдер

`accepted` — бекітілген техникалық талап; `proposed` — іске асыру/операциялық нақтылауы бар ұсыныс. Accepted мәртебесі код жазылды немесе салон рәсімі расталды дегенді білдірмейді.

| ID | Мәртебе | Шешім | Негіздеме және шектеу |
| --- | --- | --- | --- |
| ADR-001 | accepted | Бір салон, KZT, модульдік Next.js монолит | Бекітілген бастапқы шекара; көпфилиалды өнім MVP емес |
| ADR-002 | accepted | PostgreSQL — негізгі факт көзі | Relational integrity; нақты деректер Git-ке кірмейді |
| ADR-003 | accepted | Қызмет пен payment бөлек | Бір visit-ке бірнеше service және split receipts; allocations тек бизнес қажет болса |
| ADR-004 | accepted, revised | Қызмет capture-ы комиссияға тәуелсіз | Missing rule → pending/null; confirmed snapshot өзгермейді; формула A04/A12 әлі ашық |
| ADR-005 | accepted | Append-only ақша және correction тарихы | Original command өзгермейді; жаңа correction command, reversal FK және audit тізбегі |
| ADR-006 | accepted | DB atomic command және idempotency | Retry дубль жасамайды; адам қайта енгізген candidate үшін бөлек quality check |
| ADR-007 | accepted | Auth, active identity/membership/link, RLS | Shared visit grants құпия сомаларды ашпайды; elevated RPC бөлек тексеріледі |
| ADR-008 | proposed | Online-first MVP | Offline sync кейін; уақытша журнал/алмастырушы A24/A31 арқылы расталады |
| ADR-009 | accepted, revised | Effective/recorded/correction уақыттарын бөлу | Тарихи түзету бүгінгі жалған cash емес; snapshot immutable, restated бөлек; күн жабу өкілеті A14 |
| ADR-010 | accepted | Anonymous walk-in | Клиентті байланыстыру міндетті емес; retention қамтуы бөлек |
| ADR-011 | proposed | Hosting дерек талабына тәуелді | Нақты орналасу, Auth/logs/backups және құқықтық негіз A20/A21 арқылы шешіледі |
| ADR-012 | accepted | Ескі Excel автоматты импортталмайды | Кейін staging/provenance/verification; нақты import осы кезеңде жоқ |
| ADR-013 | accepted | Қазақша/орысша | Негізгі жоба талабы; default тіл A24 арқылы |
| ADR-014 | accepted | Analytics сенімді дерекке сүйенеді | Pending/incomplete және бақыланған тарих шектеуі есепте көрсетіледі |
| ADR-015 | accepted | Controlled shared visit contributions | Active participant өз service-ін қосады; assignment operator/алмастырушы A29 |
| ADR-016 | accepted | Owner-only employee-account links | Уақыттық link history, атомарлы reassignment, current ownership RLS, immutable actors |
| ADR-017 | accepted | Employee-day verification + ақша reconciliation | Екі бөлек бақылау; service өзгерсе verification stale; cohort/мерзім A30 |
| ADR-018 | accepted | Late entry және unresolved quality worklist | Reason/authorization, effective кезең, candidate review; SLA/оператор A31/A32 |
| ADR-019 | proposed | Cash opening chain және transaction-ordered sequence | P1 шешімінің нақты моделі: opening FK, snapshot cutoff; бір салон write lock-ы, нақты DDL кейін |

## 3. P1 шешімдерінің қадағалануы

| Шолу мәселесі | Құжатталған шешім | Қабылдау критерийлері |
| --- | --- | --- |
| Комиссия жоқта қызмет жоғалады | Pending projection, immutable confirmed earnings және quality issue | AC-01, AC-02, AC-16 |
| Кешегі correction бүгін cash жасайды | Effective күн бойынша есеп, бөлек recorded/corrected уақыт, immutable snapshot/opening chain | AC-04, AC-05, AC-06, AC-14 |
| Бір тіркеушіге тәуелділік және shared visit құпиялығы | Visit participants, өз performer-і, минимал DTO; оператор/алмастырушы ашық болжам | AC-07, AC-08, AC-10, AC-11 |
| Admin employee binding-ті өзгерте алады | Generic edit-тен binding алып тастау, Owner-only links/reassignment | AC-09, AC-15 |
| Тіркелмеген/толық емес операциялар байқалмайды | Employee verification, issue worklist және тәуелсіз reconciliation; толықтық кепілдігі деп көрсетілмейді | AC-10, AC-11, AC-12 |
| Original/correction history байланысы қайшы | Original command сақталады, жаңа correction FK/command, атомарлы audit | AC-13, AC-14 |

AC анықтамалары: [MVP қабылдау критерийлері](02-mvp-specification.md#8-қабылдау-критерийлері). Тесттермен байланысы: [жол картасы](05-roadmap-testing-deployment.md#қабылдау-критерийлерінің-тесттермен-байланысы).

## 4. Осы кезеңнің шекарасы

Бар Git репозиторийінде сегіз planning Markdown файлын нақтылау, documentation checks, commit және рұқсатты push — осы кезеңнің жұмысы. Жаңа репозиторий/сыртқы ресурс, қолданба scaffold-ы, пакет орнату, SQL migration, deployment немесе нақты клиент/қаржы дерегін импорттау жасалмайды. Commit/push нақты нәтижесі task есебінде көрсетіледі; болашақ application acceptance tests орындалды деп мәлімделмейді.
