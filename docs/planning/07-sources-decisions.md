# Дереккөздер және шешімдер тізілімі

Тексеру күні: **2026-11-13**. Төмендегі құжаттар технологияның ресми мінез-құлқын тексеру үшін қолданылды. Dili-дің рөлдері, бизнес workflow-ы, кестелері және кезеңдері — осы жобаға ұсынылған инженерлік шешімдер; олар платформалардың дайын талабы емес.

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

Барлық жолдардың мәртебесі: **ұсынылған**. Review нәтижесінде accepted/revised күйіне көшеді.

| ID | Шешім | Себебі және салдары |
| --- | --- | --- |
| ADR-001 | Модульдік Next.js монолит | Бір салонға жеткілікті, deployment және транзакцияларды түсіну оңай; кейін модуль бөлуге болады |
| ADR-002 | PostgreSQL — негізгі факт көзі | Relational integrity мен атомарлыққа сүйенеміз; Excel — тек кейінгі verified import/export көзі |
| ADR-003 | Қызмет пен payment бөлек | Partial/split payment, debt және refund қызмет фактісін бұзбайды |
| ADR-004 | Орындалған қызмет және earnings snapshot | Каталог/комиссия өзгерісі өткен тарихты қайта есептемейді |
| ADR-005 | Append-only ақша, explicit corrections | Actor/reason/original reference арқылы қадағалау; кесте саны қарапайым CRUD-тан көбірек |
| ADR-006 | DB транзакциясы + idempotency | Double tap, network retry, concurrent refund және day-close қауіптерін шешеді |
| ADR-007 | Auth + active membership + RLS | UI-ды айналып өтсе де деректер шекарасы бар; elevated RPC міндетті жеке тексеріледі |
| ADR-008 | Online-first MVP | Тұрақты offline sync қақтығыстарын бастапқы өнімге қоспау; уақытша журнал тәртібі қажет |
| ADR-009 | Күн жабу snapshot-ы, кейінгі күнге correction | Жабылған есеп сақталады; restated report бөлек көрсетіледі |
| ADR-010 | Anonymous walk-in рұқсат | Дерек енгізуді жеңілдетеді; retention аналитикасының қамтуы бөлек өлшенеді |
| ADR-011 | Hosting шешімі деректер талабына тәуелді | Supabase/Vercel стек ниеті сақталады, бірақ нақты deployment архитектурасы тексеруден өтеді |
| ADR-012 | Ескі Excel автоматты импортталмайды | Белгісіз толықтық/дәлдік жаңа сенімді dataset-ті бұзбауы тиіс |
| ADR-013 | Екі тіл ең басынан | Формалар мен қателерді кейін толық қайта жазуды болдырмайды |
| ADR-014 | Analytics кейін біртіндеп кеңейеді | Алдымен операция мен анық метрика; production admin credential BI-ге берілмейді |

## 3. Осы кезеңде жасалмаған әрекеттер

Қолданба scaffold-ы, пакет орнату, DB migration орындау, GitHub репозиторийін ашу, Supabase/Vercel ресурс жасау, төлем, deployment және нақты клиент дерегін импорттау орындалған жоқ. Құжаттар — implementation бастауға арналған review материалы.
