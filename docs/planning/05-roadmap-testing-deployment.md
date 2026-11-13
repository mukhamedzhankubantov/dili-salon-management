# Әзірлеу кезеңдері, тестілеу және орналастыру

## 1. Жұмыс тәсілі

Әр кезең: бизнес ережесі → шағын implementation → тиісті тест → телефонда көрсету → нәтижені review. Төмендегі кезеңдер қазір іске қосылған жоқ. Уақыт бағасы қызметкерлер саны, ережелер және hosting шешімі анықталғаннан кейін беріледі.

| Кезең | Жұмыс | Аяқталу шарты |
| --- | --- | --- |
| 0. Осы жоспарды қарау | Дерек ағыны, жауаптылар, комиссия, төлем, күн жабу, сақтау орны | Блоктаушы болжамдарға жауап; архитектура келісілген |
| 1. Негіз | Git, Next.js/TS, styling, екі тіл, тек жасанды дерек, dev tooling | Build/typecheck; kk/ru login және mobile shell; env үлгілері secrets-із |
| 2. Identity және DB шекарасы | Supabase Auth, profiles/membership, миграциялар, базалық RLS, Owner bootstrap | Үш рөл және бөтен салонға қолжетімділікті терістейтін тесттер өтеді; admin key frontend-те жоқ |
| 3. Анықтамалық және қызмет | Employees, services, rates, walk-in visit, multiple services, earnings snapshot | Екі нақты қызметкермен сценарийлік review; орындалған қызмет тарихы сақталады |
| 4. Қаржылық операциялар | Split receipts, debt, refunds, adjustments, expenses, cash movements, audit, idempotency | Транзакция, concurrency, reversal/refund лимиттері мен correction тесттері өтеді |
| 5. Күнді жабу | Cash/electronic салыстыру, snapshot, closed-day lock, exception review | Қалыпты және айырмашылығы бар күн дұрыс жабылады; кеш түзету бастапқы есепті бұзбайды |
| 6. Есеп және экспорт | Рөлдік dashboard, CSV/XLSX, data quality | Қолмен тексерілген fixture сомаларымен дәл сәйкестік; PII/commission ағып шықпайды |
| 7. Пилотқа дайындық | Қауіпсіздік review, restore rehearsal, monitoring, қызметкер оқыту | Hosting/data decision бекітілген; production checklist орындалған |
| 8. Шектеулі пилот | 1–2 аптаға ұсынылатын бақыланатын нақты пайдалану | Күн сайын салыстыру; маңызды ақаулар жоқ; енгізу уақыты мен толықтық өлшенген |
| 9. Тұрақты пайдалану | Бақылау, patching, сапа review | Owner жүйені негізгі журнал ретінде қабылдаған; жауапты қолдау адамы бар |
| 10. Кейінгі аналитика | SQL cohorts, Python, Power BI | Жеткілікті таза тарих, анықталған метрика және рұқсат етілген иесіздендірілген есеп қабаты |

Пилотқа дейін 1–7 кезеңдер бірге MVP-ді құрайды. Audit, RLS немесе reconciliation «кейін қосылатын» қосымша емес. Қағазбен қысқа салыстыру кезеңінде әр нақты оқиғаға бір source identifier беріліп, екінші рет жүйеге кіргізілмейді.

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
2. 15 000 ₸ қызметке 10 000 ₸ төлем: debt = 5 000; келесі күні 5 000 түскенде balance = 0, күндік ақша дұрыс күнге түседі.
3. 15 000 ₸ толық төленген қызметке −3 000 adjustment: credit = 3 000. Refund 3 000 қосылса balance = 0 және касса 3 000 азаяды.
4. Қате 10 000 card receipt → reversal + cash replacement: жалпы net payment өзгермейді, шоттар түзеледі, үш жазба сақталады.
5. Бір receipt-ке қатар келген екі refund жиынтығы лимиттен аспайды; concurrent overpayment де қабылданбайды.
6. Бір idempotency key-мен екі сұрау бір нәтиже береді; сол кілтпен өзгерген payload конфликт береді. Транзакция ортасында failure болса ешқандай жартылай жазба қалмайды.
7. Payment жазу мен close-day қатар жүрсе, біреуі алдымен аяқталады; жабылған күнге жасырын жазба өтпейді.
8. Employee бөтен employee/service/payment/customer/audit/earning дерегін тікелей REST/RPC арқылы да ала алмайды; бөтен salon FK және spoofed role қабылданбайды.
9. Membership өшірілген соң ескі сессия business дерекке қол жеткізбейді. Соңғы Owner-ды қатар екі өзгеріспен жою мүмкін емес.
10. Каталог бағасы/комиссия өзгерсе, бұрынғы snapshot өзгермейді; архивтелген қызметкер тарихта қалады.
11. Түнгі шекарада `Asia/Almaty` business_date дұрыс; client clock ауыстыру posting day-ды айналып өтпейді.
12. Cash opening 20 000 + receipt 15 000 − refund 2 000 − expense 3 000 + float 5 000 − withdrawal 10 000 = expected 25 000 ₸. Actual 24 500 болса айырма −500; Owner себебімен бекітеді; келесі opening = 24 500.
13. CSV/XLSX кирилл, телефондағы `+`, `=`, `@` және formula injection payload-тарын қауіпсіз сақтайды; numeric amount мәтінге айналмайды; рұқсатсыз бағандар жоқ.
14. Backup restore нәтижесінде операция саны, бақылау сомалары, RLS, functions, indexes және Auth байланысы тексеріледі.

## 3. Git және CI ұсынысы

Қысқа feature branch және шағын pull request. `main` қорғалады; CI: formatting/lint, TypeScript, unit, DB/RLS integration, build және маңызды e2e сценарийлері. DB өзгерісі миграциямен беріледі; production dashboard-та қолмен schema өзгерту қалыпты workflow болмайды.

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

Интернет/сервис тоқтаса: салон келісілген уақытша нөмірленген қағаз журналын қолданады; қалпына келген соң бір жауапты адам `occurred_at`, нақты posting day және late-entry reason арқылы енгізеді; екінші адам қайталанбағанын салыстырады. Төтенше журналдың сақталуы мен жойылу тәртібі де белгіленеді.

## 6. Production-ға өту шарттары

- Рөлдер мен комиссия/қайтарым ережелері бекітілген.
- Дербес деректер орны, құқықтық негізі, retention және жауапты адам анықталған.
- RLS/negative authorization және қаржы concurrency тесттері өтеді.
- Сақтық көшірмеден нақты қалпына келтіру көрсетілген.
- Owner/Administrator жеке аккаунттары, қажетті MFA және recovery рәсімі бар.
- Қызметкерлер екі тілдегі негізгі сценарийді орындайды.
- Қауіпті ашық ақау жоқ; pilot support, incident response және тоқтату критерийі анық.
- Өнімді іске қосу жеке келесі қадам ретінде келісілген. Бұл жоспарды дайындау deployment әрекеті емес.
