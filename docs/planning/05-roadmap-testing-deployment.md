# Әзірлеу кезеңдері, тестілеу және орналастыру

## 1. Жұмыс тәсілі

Әр кезең: бизнес ережесі → шағын implementation → тиісті тест → телефонда көрсету → нәтижені review. Төмендегі кезеңдер қазір іске қосылған жоқ. Уақыт бағасы қызметкерлер саны, ережелер және hosting шешімі анықталғаннан кейін беріледі.

Қазіргі кезең — бекітілген техникалық шешімдерді құжаттарда нақтылау. Git репозиторийі бар; бұл құжаттарды тексеру/commit/push application implementation не deployment болып саналмайды. Нақты адамдардың міндеттері, комиссия формуласы және closure exception саясаты әлі [A02/A04/A14/A29/A30](06-business-assumptions.md) шешімдерін күтеді.

| Кезең | Жұмыс | Аяқталу шарты |
| --- | --- | --- |
| 0. Құжаттарды нақтылау | Pending commission, shared visit, уақыт, account links, verification және data quality | Техникалық шешімдер мен операциялық ұсыныстар ажыратылған; AC-01–AC-16 және P1 шешімдері өзара сәйкес |
| 1. Негіз | Git, Next.js/TS, styling, екі тіл, тек жасанды дерек, dev tooling | Build/typecheck; kk/ru login және mobile shell; env үлгілері secrets-із |
| 2. Identity және DB шекарасы | Supabase Auth, profiles/membership, Owner-only employee_account_links, миграциялар, RLS | Үш рөл, reassignment, disabled identity және бөтен scope тесттері өтеді; admin key frontend-те жоқ |
| 3. Анықтамалық және қызмет | Shared visits/participants, өз қызметін тіркеу, pending/confirmed commission, verification және негізгі quality issues | AC-01/02/07–11; комиссиясыз capture және екі маманның бір visit-ке жазуы тексерілген |
| 4. Қаржылық операциялар | Split receipts, debt, refunds, adjustments, expenses, cash movements, audit, idempotency | Транзакция, concurrency, reversal/refund лимиттері мен correction тесттері өтеді |
| 5. Күнді жабу | Cash openings, employee verification күйі, effective-day reconciliation, immutable snapshot/cutoff, exception review | AC-04–06/10/12/14; өткен күнді түзету бүгінгі cash-ты жасанды өзгертпейді; unresolved issue жасырылмайды |
| 6. Есеп және экспорт | Рөлдік dashboard, CSV/XLSX, data quality | Қолмен тексерілген fixture сомаларымен дәл сәйкестік; PII/commission ағып шықпайды |
| 7. Пилотқа дайындық | Қауіпсіздік review, restore rehearsal, monitoring, қызметкер оқыту | Hosting/data decision бекітілген; production checklist орындалған |
| 8. Шектеулі пилот | 1–2 аптаға ұсынылатын бақыланатын нақты пайдалану | Күн сайын салыстыру; маңызды ақаулар жоқ; енгізу уақыты мен толықтық өлшенген |
| 9. Тұрақты пайдалану | Бақылау, patching, сапа review | Owner жүйені негізгі журнал ретінде қабылдаған; жауапты қолдау адамы бар |
| 10. Кейінгі аналитика | SQL cohorts, Python, Power BI | Жеткілікті таза тарих, анықталған метрика және рұқсат етілген иесіздендірілген есеп қабаты |

Пилотқа дейін 1–7 кезеңдер бірге MVP-ді құрайды. Audit/RLS/idempotency әр feature-мен бірге жасалады; кезең 4 оны алғаш қосу кезеңі емес. Verification мен reconciliation пилоттың міндетті бөлігі. Уақытша қағаз journal source_reference-імен, reason-мен және duplicate review-мен енгізіледі; нақты reference пен деректер Git-ке кірмейді.

## 2. Тестілеу стратегиясы

| Қабат | Ұсынылатын құрал | Нені дәлелдейді |
| --- | --- | --- |
| Domain unit | Vitest | Money arithmetic, bps rounding, қалдық, даталар, input schema, locale mapping |
| Компонент | Testing Library + accessibility checks | Field error, keyboard, loading/double submit, тіл ауыстыру |
| Database integration | Local Supabase/PostgreSQL + pgTAP не SQL assertions | Constraints, RLS, функция grant-тары, атомарлық, immutable фактілер |
| Server integration | Session-мен RPC/HTTP | Authorization, malformed payload, idempotency және минимал DTO |
| End-to-end | Playwright | Login → walk-in → multiple service → split payment → close → export |
| Қолмен | Нақты Android/iPhone және салон қызметкерлері | Түсініктілік, кіші экран, әлсіз желі, бір қолмен жұмыс |

Құралдар — ұсыныс; әлі орнатылған жоқ. CI нақты қауіптерді тексереді; implementation-ды сол күйі қайталайтын тесттерден гөрі бизнес нәтижесін тексереміз.

### Міндетті сценарийлер

1. 10 000 + 5 000 ₸ екі қызмет, 5 000 cash + 10 000 card: service total = 15 000, net payment = 15 000, balance = 0. Join арқылы сомалар көбеймейді.
2. 15 000 ₸ қызметке 10 000 ₸ төлем: debt = 5 000; келесі күні 5 000 шын түскенде balance = 0, effective күн бойынша ақша дұрыс есептеледі.
3. 15 000 ₸ толық төленген қызметке −3 000 adjustment: credit = 3 000. Refund 3 000 қосылса balance = 0 және касса 3 000 азаяды.
4. Қате 10 000 card receipt → reversal + cash replacement: жалпы net payment өзгермейді, үш жазба сақталады; original command өзгермейді. Жабылған кешегі оқиға бүгін түзетілсе, бүгін inflow өзгермейді (AC-04).
5. Бір receipt-ке қатар келген екі refund жиынтығы лимиттен аспайды; concurrent overpayment де қабылданбайды.
6. Бір idempotency key-мен екі сұрау бір нәтиже береді; сол кілтпен өзгерген payload конфликт береді. Транзакция ортасында failure болса ешқандай жартылай жазба қалмайды.
7. Payment/historical correction мен close-day қатар жүрсе, command sequence және cutoff commit ретімен сәйкес; live write жабық effective күнге өтпейді, authorized historical append бұрынғы snapshot-ты өзгертпейді.
8. Employee өз shared visit summary-ін және өз қызметін оқи алады; басқа employee сомаларын/payment/customer contacts/audit/earning дерегін тікелей REST/RPC арқылы да алмайды. Бөтен scope/performer, revoked participant және тек visit_number білу access бермейді.
9. Membership өшірілген соң ескі сессия business дерекке қол жеткізбейді. Соңғы Owner-ды қатар екі өзгеріспен жою мүмкін емес.
10. Каталог бағасы/комиссия өзгерсе, бұрынғы snapshot өзгермейді; архивтелген қызметкер тарихта қалады.
11. Түнгі шекарада `Asia/Almaty` effective_business_date/recorded_business_date дұрыс; client clock ауыстыру authorisation-ды айналып өтпейді. Кеш енгізу нақты effective уақытын сақтайды және reason/approval қажет.
12. Cash opening 20 000 + receipt 15 000 − refund 2 000 − expense 3 000 + float 5 000 − withdrawal 10 000 = expected 25 000 ₸. Actual 24 500 болса айырма −500; exception бекітілген A14 рәсіміне сай; келесі opening = 24 500. Actual null 0 емес.
13. CSV/XLSX кирилл, телефондағы `+`, `=`, `@` және formula injection payload-тарын қауіпсіз сақтайды; numeric amount мәтінге айналмайды; рұқсатсыз бағандар жоқ.
14. Backup restore нәтижесінде операция саны, бақылау сомалары, RLS, functions, indexes және Auth байланысы тексеріледі.
15. Missing commission қызмет/төлемді тоқтатпайды; pending/null, issue және audit сақталады. Кейін Owner effective rule-мен есептейді; бұрынғы adjustments ескеріледі; бекітілген none не расталған ереженің нөлдік нәтижесі ғана нақты 0 береді.
16. Екі маман бір visit-ке қатар қызмет қосады; өз performer-і ғана; әрқайсысы өз күндік тізімін растайды. Кейінгі түзету version-ды өсіріп, ескі verification-ды stale етеді; Admin басқа маманның атынан confirmation жасай алмайды.
17. Owner account reassign кезінде active link uniqueness, old-link revoke және audit бірге сақталады; Admin жалпы employee update арқылы binding өзгерте алмайды; original financial actors өзгермейді.
18. Repeated data quality check duplicate issue жасамайды; duplicate candidate автоматты жойылмайды; anonymous client қате саналмайды. Pending commission жай dismiss емес; closure мәселелерді автоматты resolved қылмайды.
19. Кеше cash кеш тіркелді, бүгін шын refund жасалды: late receipt бүгінгі cash-қа қосылмайды, actual refund бүгінгі cash-тан алынады. Closed-day correction Owner approval-сыз өтпейді.
20. Audit insertion failure бүкіл command-ты rollback етеді; export-та pending count/null, effective кезең және as_of_sequence сақталады; PII/құпия commission бағандары рөл бойынша шектеледі.
21. Outage кезінде тарихи күн ашылмаған және actual count жоқ: Owner unreconciled metadata жасайды; late entry reason/authorization-мен түседі. Жаңа күннің counted gap opening-і approved/reason/issue арқылы белгіленеді; жалған өткен closure немесе жетіспейтін actual пайда болмайды.

### Қабылдау критерийлерінің тесттермен байланысы

Негізгі анықтамалар: [AC-01–AC-16](02-mvp-specification.md#8-қабылдау-критерийлері).

| Критерийлер | Қамту |
| --- | --- |
| AC-01, AC-02 | DB + server integration + UI: pending capture, confirmation, null/zero айырмасы |
| AC-03 | Money unit + DB concurrency + e2e split payment |
| AC-04, AC-05, AC-06, AC-14 | Effective-time fixture, opening continuity, close/correction race, snapshot/restated comparison |
| AC-07, AC-08, AC-09, AC-15 | Үш role және екі employee-мен тікелей RPC/REST теріс тесттері, shared visit, reassignment |
| AC-10 | Employee/day version және immutable verification integration |
| AC-11, AC-12 | Quality issue deduplication, scoped visibility, resolution/exception audit |
| AC-13 | Fault injection: audit failure rollback; original/correction/output FK және actor тізбегі |
| AC-16 | CSV/XLSX content, role filtering, pending/null және as-of метадерегі |

Бұл тесттер болашақ implementation үшін; осы documentation-only кезеңінде орындалған тесттер деп көрсетілмейді.

## 3. Git және CI ұсынысы

Кейінгі әзірлеу үшін ұсыныс: қысқа feature branch және шағын pull request; `main` қорғауын баптау. Бұл құжат branch protection немесе CI қазір қосылған дегенді білдірмейді. Болашақ CI: formatting/lint, TypeScript, unit, DB/RLS integration, build және маңызды e2e. DB өзгерісі миграциямен беріледі; production dashboard-та қолмен schema өзгерту қалыпты workflow болмайды.

Documentation-only тексеру: `git diff --check`, Markdown файл/anchor сілтемелері, UTF-8, fenced block/table құрылымы, AC ID бірегейлігі мен test mapping, ескі қайшы ережелерге іздеу, өзгерген файлдардың тек `docs/planning/` ішінде болуы. Репозиторийде дайын lint/test configuration болмаса, жаңа dependency орнатпай жергілікті тексеру жасалады. Semantic review міндетті; мәтін іздеу оның орнын баспайды.

Pull request ішінде өзгерген мінез-құлық, migration тәуекелі және нақты validation нәтижесі көрсетіледі. Dependencies lockfile-мен бекітіледі, security жаңартулары жоспарлы қаралады. GitHub/Vercel/Supabase ұйымдық аккаунттарының иесі Dili болады; қолжетімділік жеке аккаунтпен беріледі.

## 4. Орта және deployment ұсынысы

| Орта | Дерек | Мақсаты |
| --- | --- | --- |
| Local | Жасанды seed | Күнделікті әзірлеу және migration/RLS тесті |
| Preview/staging | Жеке оқшауланған жасанды dataset | PR және қабылдау тексеруі |
| Production | Тек рұқсат етілген нақты дерек | Пилоттан бастап операциялық жұмыс |

Preview production DB/secret-терін алмайды. Auth redirect allowlist, custom domain, HTTPS, monitoring, email жеткізу және env validation бөлек тексеріледі. Paid plan бағалары қазір болжанбайды; backup, restore, limits және деректер аймағы талапқа сай нақты ұсыныспен кейін бағаланады.

Орналастыру тізбегі: staging migration → тест → backup тексеру → backward-compatible production migration → application release → smoke test. Destructive migration expand/contract тәсілімен бірнеше release-ке бөлінеді. App rollback DB өзгерісін автоматты кері қайтармайды; дерек жоғалтпайтын forward-fix және restore жоспары бөлек.

## 5. Қалпына келтіру және бақылау

Бастапқы талқылау мақсаты: **RPO ≤ 15 минут**, **RTO ≤ 4 сағат**. Бұл қолданыстағы кепілдік емес: таңдалған hosting, backup/PITR мүмкіндігі және budget-пен расталып, restore rehearsal арқылы өлшенеді. Егер қамтамасыз етілмесе, бизнес қабылдайтын басқа межені іске қосуға дейін бекіту қажет.

Backup сәттілігі, restore уақыты, application error rate, failed financial commands, auth failures, DB latency және жабылмаған күндер бақыланады. Log-қа клиент PII немесе token түспейді. Техникалық monitoring деректі толық енгізудің дәлелі емес; күн сайын operational reconciliation қажет.

Ұсынылған outage рәсімі (A24/A31 бойынша әлі расталады): уақытша нөмірленген журнал; қалпына келгенде effective_at, source_reference және late_entry_reason арқылы authorized entry, жабық effective кезеңге Owner approval, duplicate review. Түпкі журналдың сақталуы/жойылуы және алмастырушы нақтыланады. Онлайн жүйе network error кезінде жалған «Сақталды» көрсетпейді.

## 6. Production-ға өту шарттары

- Рөлдер мен комиссия/қайтарым ережелері бекітілген.
- Ортақ visit қатысушыларын тағайындаушы/алмастырушы, employee-day cohort, verification және closure exception рәсімдері бекітілген; unresolved issues көрінеді.
- Дербес деректер орны, құқықтық негізі, retention және жауапты адам анықталған.
- RLS/negative authorization және қаржы concurrency тесттері өтеді.
- Сақтық көшірмеден нақты қалпына келтіру көрсетілген.
- Owner/Administrator жеке аккаунттары, қажетті MFA және recovery рәсімі бар.
- Қызметкерлер екі тілдегі негізгі сценарийді орындайды.
- Қауіпті ашық ақау жоқ; pilot support, incident response және тоқтату критерийі анық.
- Өнімді іске қосу жеке келесі қадам ретінде келісілген. Бұл жоспарды дайындау deployment әрекеті емес.
