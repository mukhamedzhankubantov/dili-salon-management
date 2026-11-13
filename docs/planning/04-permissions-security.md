# Рөлдер, қауіпсіздік және деректерді қорғау

## 1. Рөлдер матрицасы

Бекітілген техникалық шекаралар: Employee өзінің қызметін тіркейді/растайды; ортақ visit қатысуы басқа маманның құпия дерегін ашпайды; employee-account link/reassign тек Owner-ға тиесілі; сезімтал historical correction Owner approval талап етеді. Төмендегі өзге өкілеттер — least-privilege операциялық ұсыныс, салонның қазіргі рәсімі емес; нақты адамдар, алмастырушы және күн жабу шектері [бизнес болжамдары](06-business-assumptions.md) арқылы расталады. Owner — қолданбадағы рөл, PostgreSQL superuser емес.

| Әрекет | Owner | Administrator | Employee |
| --- | --- | --- | --- |
| Қызмет каталогын оқу | Барлығы | Барлығы | Белсенді каталог |
| Каталог/баға өзгерту, архивтеу | Иә | Иә, аудитпен | Жоқ |
| Қызметкер карточкасын басқару | Иә | Иә, рөл мен комиссиясыз | Өз карточкасын оқу |
| Employee-account link/reassign | Иә, арнайы reason/audit командасымен | Жоқ | Жоқ |
| Аккаунт шақыру, рөл беру/бұғаттау | Иә | Жоқ | Жоқ |
| Комиссия ережелерін өзгерту | Иә | Жоқ | Жоқ |
| Қызметтерді көру | Салон бойынша | Салон бойынша | Өзі орындаған жолдар |
| Қызмет енгізу | Кез келген орындаушы | Кез келген орындаушы | Өзі ашқан не active participant болған келуге тек өзі үшін; pending комиссиямен де |
| Ортақ келуге маман қосу/алып тастау | Иә | Иә, операциялық ұсыныс | Өз жаңа келуіне автоматты қатысу; басқа маманға grant бермейді |
| Күндік жеке қызметтерді растау | Өз employee байланысы болса өзі үшін | Өз employee байланысы болса өзі үшін | Тек өз тізімінің ағымдағы нұсқасы |
| Жеңілдік/ерекше баға | Иә | Ашық күні, себеппен | Жоқ |
| Клиент карточкасы және телефон | Иә | Иә | Жоқ; өз қызметіне қажет masked summary ғана |
| Клиентті келуге байланыстыру | Иә | Иә | Жоқ; аноним келу тіркей алады |
| Төлем/бөлінген төлем енгізу | Иә | Ашық күні | Жоқ |
| Нақты refund | Иә | Сұрау ғана | Жоқ |
| Қате payment reversal + replacement | Иә | Өзгеріс ашық ағымдағы күндегі бастапқы жазбаға қатысты болса, себеппен | Жоқ |
| Service correction/void | Иә | Ашық ағымдағы күнге, себеппен | Өз жолына сұрау ғана |
| Жабылған кезеңге қатысты түзету | Бастапқы/дұрыс effective кезеңге, бүгін recorded/себеппен | Сұрау ғана | Өз жолына сұрау ғана |
| Late entry | Рөліне сай; жабық effective кезеңге approval | Ашық effective күндегі рұқсатты операция, себеппен | Ашық effective күнде өзінің қызметі, себеппен; басқа жағдайда сұрау |
| Операциялық шығын | Иә | Ағымдағы күнде тіркеу/қарау | Жоқ |
| Cash float/инкассация | Иә | Сұрау ғана | Жоқ |
| Күнді жабу | Бекітілетін рәсім бойынша | Дайындау/өкілет берілгенде жабу | Жоқ |
| Ақша айырмасы/тексеру exception-ын бекіту | Ұсынылған жауапты | Дайындау; нақты шек A14 арқылы | Жоқ |
| Есептелген табысты көру | Барлығы | Жоқ | Тек өзінікі |
| Pending комиссияны шешу | Иә, confirm_commission | Тек операциялық күй/сан, сома/ереже жоқ | Өз pending күйін көру, ережені бекітпейді |
| Дерек сапасы мәселелерін қарау | Салон бойынша | Операциялық мәселелер; сезімтал correction-ға сұрау | Өз мәселелері, басқа visit/employee дерегі жоқ |
| Dashboard | Барлық метрика | Күнделікті қызмет/төлем/шығын, комиссиясыз | Өз қызметі/табысы |
| CSV/XLSX | Рұқсат етілген барлық есеп | Операциялық есеп, клиент PII/комиссиясыз | Жоқ |
| Аудит | Салон бойынша | Операциялық нысандар, рөл/комиссия/PII-сіз | Өз correction сұрауының тарихы |
| Audit/қаржы тарихын жою | Жоқ | Жоқ | Жоқ |

Employee cashier міндетін де атқарса, бастапқыда кең Administrator рөлін ойланбай бермейміз. Оны іске асыру алдында жеке capability ретінде жобалау қажет. MVP-де үш рөл және жоғарыдағы шекаралар қолданылады.

Қызметкер карточкасының Administrator өзгерте алатын бастапқы өрістері: display_name, started_on, ended_on және операциялық is_active (A28 бойынша расталады). Auth identity, role, membership, account links, rate/earnings бұл командаға кірмейді. Employee-ді inactive ету оның performer ретінде жаңа қызмет енгізуін тоқтатады; membership арқылы басқа рөлдегі қолжетімділік автоматты жойылды деп саналмайды. Аккаунтты толық тоқтатуды Owner бөлек не атомарлы offboarding командасымен орындайды.

Owner account reassignment кезінде салонды lock-пен қорғайды, ескі link.valid_to-ны бекітеді, жаңа link ашады және actor/reason/audit жазады. Бір уақытта бір employee бір active membership-ке, бір membership бір active employee-ге ғана сәйкес. Ownership RLS ағымдағы link-ке қарайды: ескі аккаунттың сол employee атынан жаңа қызмет жазу/табысын оқу құқығы жоғалады; оның бастапқы audit actor ID-лері өзгермейді. Profile.disabled_at және membership.is_active әр қорғалған операцияда тексеріледі.

## 2. Authentication және сессия

Жеке шақырылған аккаунттар; ортақ «касса» логині жоқ. Бастапқы ұсыныс — email/password, email-і жоқ қызметкерлер үшін балама тәсіл бірінші кезеңге дейін анықталады. Owner үшін MFA production шарты, Administrator үшін де ұсынылады. Password reset, invite redirect URL және session timeout нақты конфигурацияланады.

Supabase SSR cookie client қолданылады. Сервер identity-ді тексереді; `getSession()` нәтижесіндегі user объектісіне жалғыз өзі сенбейді. Ағымдағы ресми нұсқаулық JWT үшін `getClaims()`, жаңа Auth user күйі керек болса `getUser()` қолдануын түсіндіреді. Құқық үшін әр сұрауда DB-дегі белсенді membership тексеріледі; бұрын берілген JWT-дегі role өздігінен жеткіліксіз. [Supabase SSR](https://supabase.com/docs/guides/auth/server-side/creating-a-client?queryGroups=framework&framework=nextjs).

Shared device-тен шығу, белсенді емес аккаунтты дереу тоқтату және жоғалған құрылғы сессиясын қайтарып алу тексеріледі. SSR SDK cookie талабына сай Secure/SameSite баптаулары қолданылады; browser SDK-ға қажет cookie-ге үйлеспейтін HttpOnly талабын соқыр түрде қоспаймыз. Auth response және жеке беттер public cache-ке түспейді.

## 3. RLS және DB құқықтары

Барлық exposed business кестелерде RLS, әдепкі тыйым және ең аз GRANT. `anon` ешбір салон дерегін оқымайды. `authenticated` бүкіл кестеге автоматты full access алмайды. Scope: тексерілген `auth.uid()` → profile → active membership → salon → role/employee ownership.

| Деректер тобы | Оқу саясаты | Жазу саясаты |
| --- | --- | --- |
| Membership/profile | Өз профиль; Owner өз салон мүшелігін басқарады | Role/status тек Owner командасымен; өзін Owner етуге жол жоқ |
| Каталог | Active membership; Employee үшін қажетті каталог өрістері | Owner/Administrator scoped command |
| Visits/services/participants | Owner/Administrator salon scope; Employee active participant visit summary және өз тарихи service жолдары | Performer active account link-тен, жаңа append үшін active participant міндетті; visit number access бермейді |
| Customers/consents | Owner/Administrator | Бақыланатын команда; Employee raw access жоқ |
| Rates/earnings | Owner немесе нақты employee ownership | Тек Owner ережесі және trusted есептеу |
| Payments/expenses/closure | Owner/Administrator рөл шегінде | Тек транзакциялық command |
| Audit | Role-specific safe projection | Тек DB trigger/trusted command; update/delete жоқ |
| Account links | Өзіне қажетті current binding; Owner салондағы тарихты оқиды | Owner-only link/reassign; Admin generic employee edit арқылы жаза алмайды |
| Employee day checks/verifications | Өз employee тізімі; Owner/Admin операциялық current/stale күйі | Employee өзінің data_version-ын растайды; Admin оның атынан растау жасамайды |
| Data-quality issues | Өз scope; Admin операциялық summary; Owner сезімтал дерек | Issue шешімі domain permission-ға бағынады; pending_commission жай dismiss етілмейді |

RLS жолдарды шектейді, бағандарды өздігінен жасырмайды. Employee-ге raw visit/customer жолын бермеу үшін бағандық GRANT және қауіпсіз view/RPC қолданылады. Earnings бөлек кестеде. Safe summary басқа орындаушының қызметін, visit толық сомасын немесе телефонды жарияламайды. Аудит JSON-ы да рөлге сай сүзгіленеді.

Shared visit summary тек visit ID/нөмірі, open/completed күйі және өзіне қатысты рұқсаттарды қайтарады. Басқа participant-тың rate, service amount, earnings, payment не customer contact өрістері берілмейді. Participant grant-ін жою бұрынғы өз қызметтерін оқуды автоматты жоймайды, бірақ жаңа append-ті тоқтатады; profile/membership/account link бөлек жарамды болуы тиіс. Pending комиссияны шешу үшін құпия rate-ті Administrator UI-ына беру қажет емес.

Authorization `recorded_business_date` ашық болуын ғана тексермейді: effective күн closed/unreconciled болса, late entry/correction-ға Owner approval және reason керек. Unreconciled historical day metadata және gap opening baseline-ды тек Owner жасайды; ақша санау нәтижесі ойдан толтырылмайды. Approval ID-ін клиент payload-ынан сенімді деп қабылдамаймыз. Payment correction output-ы түпкі payment-пен бір салонда/visit ішінде болуы, original FK және уақыт мағынасы DB командасында тексеріледі.

Оқу view-ларында қолдайтын PostgreSQL нұсқасында `security_invoker = true`, тиісті underlying GRANT және RLS бірге қолданылады. Құпия projection үшін тексерілген функция немесе exposed емес schema таңдалады. Policy helper membership кестесін рекурсивті оқып шексіз RLS тізбегін жасамауы тиіс. [Supabase RLS](https://supabase.com/docs/guides/database/postgres/row-level-security).

Қаржы кестелеріне end-user INSERT/UPDATE/DELETE берілмейді. Сондықтан жазу RPC-лерінің аз бөлігі шектеулі `SECURITY DEFINER` болады: арнайы ең аз құқықты owner role, бекітілген бос `search_path`, толық schema-qualified атаулар, `PUBLIC/anon` EXECUTE-ін қайтарып алу. Функция ішінде identity, membership, объект scope және рөл міндетті тексеріледі. Definer өздігінен RLS кепілдігі емес; оны жеке қауіпсіздік шекарасы ретінде тестілейміз. Қалған функциялар invoker болады. [Supabase Database Functions](https://supabase.com/docs/guides/database/functions).

Server Action айналып өтіліп, RPC тікелей шақырылса да дәл сол ереже орындалуы тиіс. Рөлді немесе `salon_id`-ді form field арқылы ауыстыру мүмкін емес. Соңғы Owner-ды жоюға/төмендетуге қарсы тексеру бір салонның lock-ымен жасалады.

## 4. Құпиялар мен қосымша қорғаныс

- Frontend-те тек жариялауға арналған Supabase URL/publishable key. `service_role`, secret key, DB password, signing secret және GitHub token тек серверлік secret storage ішінде.
- Қалыпты бизнес сұраулар user session-мен орындалады. Admin key күнделікті CRUD үшін пайдаланылмайды; қажет болса invite секілді шектеулі identity management функциясына ғана server-only бөлік беріледі.
- `.env.local` және дамптар Git-ке кірмейді. `.env.example` бос үлгі. Secret scanning, dependency review және leak болса key rotation тәртібі болады.
- Барлық сыртқы байланыс HTTPS/TLS. Origin/CSRF тексеру, login/reset/export rate limit, қауіпсіз redirect allowlist, CSP және security headers іске асырылады.
- Қате хабарлары SQL, token, email тізімі немесе клиент телефонын ашпайды. Логтарда request ID, actor reference және операцияның техникалық күйі жеткілікті.
- CSV-де қауіпті формула басталатын мәтіндер қорғалады; XLSX мәтін ұяшықтары formula ретінде жазылмайды. Export URL public bucket-ке қойылмайды; default жауап streaming/no-store.
- Жеке беттерге analytics session replay қосылмайды. Клиент деректерін bug report, AI prompt немесе screenshot-қа көшіру default workflow емес.
- Файл тіркемелері MVP-ге кірмейді; кейін енгізілсе private storage, size/type тексеруі және қысқа мерзімді signed URL қажет.
- Нақты customer/employee identity, Auth credential, payment reference және қаржылық операциялар Git commit, тест fixture немесе жоспар мысалына кірмейді. Жоспардағы барлық сандар жасанды.

Қауіпсіздікті қабылдау критерийлері: [AC-07–AC-15](02-mvp-specification.md#8-қабылдау-критерийлері). Әр тыйым UI-да ғана емес, тікелей RPC/REST және DB integration деңгейінде тексеріледі. Actor мен reason аудитте бар болуы, grant/revoke/reassignment history сақталуы және audit жазылмаса command rollback болуы міндетті.

## Дербес деректер және орналастыру

«Дербес деректер және оларды қорғау туралы» заңның 12-бабында Қазақстан аумағындағы сақтау талабы, 16-бабында трансшекаралық беру шарттары бар. Бұл архитектураға әсер ететін тексеру нүктесі. Нақты құқықтық негіз, қолданыстағы редакция және Dili-ге қолданылуы жергілікті заңгермен расталады. Зерттеу кезінде негізгі бет JavaScript талап етті; баптардың мәтіні ресми Әділеттің индекстелген нәтижелерінен тексерілді, толық құқықтық сараптама жасалған жоқ. [Әділет, заң №94-V](https://old.adilet.zan.kz/rus/docs/Z1300000094).

Тексерілген Supabase Cloud region тізімінде Қазақстан жоқ. Сондықтан географиялық жақын серверді таңдау compliance мәселесін автоматты шешпейді. [Supabase regions](https://supabase.com/docs/guides/platform/regions).

Таңдауға ұсынылатын нұсқалар:

1. Заңды негізі мен деректер ағыны расталған жағдайда сұралған managed Supabase + Vercel конфигурациясы.
2. Қазақстандағы инфрақұрылымда Supabase self-hosting және қажет болса Next.js орналастыру. Бұл backup, patching, мониторинг және incident response жауапкершілігін арттырады; бөлек бағалау қажет.

Vercel-ді тек UI үшін қалдыру да server rendering, Auth cookies, IP, логтар және қате есептері арқылы деректер шетелге өтуі мүмкін екенін ескеруді талап етеді. Қазақстандағы DB-нің болуы қалған деректер ағынын автоматты заңдастырмайды. Бекітілгенге дейін тек жасанды dev/test деректері қолданылады.

Минимизация: клиент аты/телефоны optional; ЖСН, карта PAN/CVV, жеке куәлік, денсаулық туралы еркін жазбалар жиналмайды. Қызметкер аты мен Auth email-і де дербес дерек екенін ескереміз. Marketing consent негізгі операциялық өңдеуден бөлек.

## 6. Сақтау мерзімі, аудит және қалпына келтіру

Owner мен заңгер retention кестесін бекітеді: клиент контактілері, consent дәлелдері, операциялар, аудит, export және backup үшін мақсат/мерзім/жою тәсілі бөлек көрсетіледі. Қаржы мен клиент байланысын сақтау қажеттілігі аяқталса, уәкілетті anonymization процесі қолданылады; жай UI delete емес.

Аудитті кәдімгі қолданба рөлі өзгерте алмайды, бірақ DB әкімшісі техникалық түрде өзгерте алады. Сондықтан «өзгерту мүлде мүмкін емес» деп уәде бермейміз. Production әкімшілік қолжетімділік шектеледі, admin әрекеттері бөлек бақыланады және backup/қажет болса сыртқы append-only көшірме тексеріледі.

Backup-та да дербес дерек бар. Шифрлау, сақталатын аймақ, retention және қолжетімділік тексеріледі. Қалпына келтіргеннен кейін бұрын орындалған anonymization/withdrawal әрекеттерін қайта қолдану тәртібі қажет. Дерекқор backup-ы Storage файлдарын қамтымайды; тіркемелер кейін қосылса олардың backup-ы бөлек жоспарланады. [Supabase backups](https://supabase.com/docs/guides/platform/backups).
