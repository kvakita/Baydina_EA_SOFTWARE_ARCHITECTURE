# Диаграмма системного контекста
![image](https://uml.planttext.com/plantuml/png/ZLRRIXjH57tFLvpw547hYqzIIaigr8UY6Dz3I8PqQ9oKp2ZwbcIyqKXZBQMbfKIXVPLu2OunbryuvnVw9LrxdvDaPX9h1I_dp9wzrrvxxHLVk5wsx6rj5aLwr_MipLJ9yQmTRpvLsdnVjBDEpahdIcNRAIIUs4wkk9MtnBFKua9gCMCYdoSIdkqLBI7FPQ2ggYRRiYjxgYxlXEp8FlwagmuzMXAoAUzbIzNKdkoh2W9RGbKf38T0zkGrBlYjhXnG9DxigUeyFTLnyagOy07_H2wyU4K5rJuzkiUnpKKHcqYiMMMtv2JBjlikatMoHJGv9sRacSutZmezVECbpjEPeOTVSJDGFXS40bKJE5Oe7muLN0OJXUR4E9EyHRQAylbDnBVbBGxtwXYXr7fTK8A-l5GVL0D5UfI6ad6mEj97rNWwCpk4xthbh6SLTZFP_ARj4COJUGEE0eu4iuO9C292npRu3EGTD_KJD-O94nPgh8LZG6mJiDWuq1j6sPxC6X0yV6dP95sjaepBHoeObiFab1dJAL9ZZ2ZTKmsDfwyEkF-EyaLywX766UiUumwOz-4Khfdow-WCT1xGZdA2s-8NKAW_r2afhiM3qAtfnZBBEruoLwQEKkjlXBpac8vfu1lGZ9GCv9YQe09Xh2srFnT5oZUONr_pQtIyJrL3HSjMmSRSTwdEz-cA1tHkjgsdmv3EwIyqti5ldy85OccJReQwMeeIq2BC4G9myUVWD89PrMFe2gLjglOPaAfX351tKdx1_Pcya3_GvYyextoMeLtm-6zGu932QI16MRmNM3XLPs88HOzbHsIEuTCE7KoXoeLFUNR9SRNkU48q3N8VamZb1aCLQEtHtc7PzEQHD8XIdrLEH9cDh4xDXbnjNjP3YekXoMvvRoSN0B1zFf0c0kQsepM3sibqUbLpT89zgvb5kHrh2bhtHxjw7pOC31Axqvs28czx6WP1lI8m3887K3C2_iqfnmuK3k2gl4jiNms3ownC7qmTga-uxB46QcPjrgrYp6qdhU4nWrN-dBb1I9V5KGcPrZHFfMvURsoifO618CGTSnB4_njYtFd2jNInMFAX0auU6hbgB52HQPDcFrJ7k28wT5KQEBluwighrB31M4tjKOPQWdiKxYgoKn5tehJho-cD1zA6NMXwRhFt8yU1SPC7aruHNO-RwzqZ1OsnD9a67mjcgdoSQXxXnYZnoxML1_96FA8vLjlmWuS-648cqPBTV-SRxLTJMm4XeNLuy1CVQUaplyvQQ53ovMtByHOC5uaNbfF7_s5_0G00)
```PlantUML
@startuml SystemContext-ComplianceScoring
!include <C4/C4_Context>

title Системный контекст: Платформа скоринга комплаенс-рисков корпоративных клиентов

Person(risk_analyst, "Риск-аналитик", "Запускает проверки клиентов, анализирует результаты и объяснение решения.")
Person(strategy_admin, "Администратор стратегий", "Настраивает правила и скоринговые стратегии.")

System(scoring_system, "Система скоринга", "Выполняет оценку рисков клиентов и филиалов, агрегирует риски по головной компании.")

System_Ext(crm, "CRM банка", "Информация о клиентах, филиалах, договорах.")
System_Ext(registry, "Корпоративный реестр", "Структура владения: головная компания → филиалы.")
System_Ext(gov, "Госреестры (ЕГРЮЛ, ФНС)", "Юридические статусы, данные о владельцах.")
System_Ext(sanctions, "Санкционные списки", "Внешние и внутренние санкционные перечни.")
System_Ext(auth, "Система аутентификации (SSO)", "Авторизация и управление ролями.")
System_Ext(audit, "Сервис аудита", "Хранение аудита и регуляторной отчётности.")

Rel(risk_analyst, scoring_system, "Запускает проверку, просматривает результаты", "HTTPS")
Rel(strategy_admin, scoring_system, "Настраивает правила и стратегии", "HTTPS")

Rel(scoring_system, auth, "Аутентификация/авторизация пользователей", "OIDC")
Rel(scoring_system, crm, "Запрашивает данные клиента", "REST")
Rel(scoring_system, registry, "Запрашивает структуру группы компаний", "REST")
Rel(scoring_system, gov, "Получает юридические данные", "API")
Rel(scoring_system, sanctions, "Проверяет по спискам", "API")
Rel(scoring_system, audit, "Передаёт результаты и события", "Event/REST")

@enduml
```
# Диаграмма контейнеров с пояснениями по выбору базового архитектурного стиля / архитектуры уровня приложений
## Выбор архитектурного стиля  
**Выбранный стиль:** Service-Based Architecture (сервисная модульная архитектура)

**Обоснование:**  
* **Доменные границы системы чётко разделены.** Платформа скоринга комплаенс-рисков включает несколько независимых логических областей — скоринг, управление правилами, агрегацию данных, аудит и отчетность. Service-Based Architecture позволяет выделить каждую область в отдельный сервис без чрезмерной фрагментации, характерной для микросервисного подхода.  
* **Высокая интеграционная нагрузка.** Система взаимодействует с CRM, корпоративным реестром, госреестрами, санкционными списками и другими источниками. Модульные сервисы с четкими границами упрощают подключение таких интеграций через адаптеры.  
* **Независимое масштабирование.** Нагрузка на скоринговый сервис и сервис агрегации данных значительно выше, чем на управление правилами или UI. Архитектура позволяет масштабировать только те части, которые реально требуют ресурсов.  
* **Поддержка асинхронного обмена.** Бизнес-события вроде “ScoringStarted” и “ScoringCompleted” естественно передаются через брокер сообщений. Такой механизм повышает устойчивость и снижает связанность между компонентами системы.  
* **Эксплуатационная прозрачность.** Стиль остаётся достаточно простым для мониторинга, логирования и поддержки DevOps-командой, что критично для систем с высокими требованиями к SLA и регуляторному аудиту.

---

## Выбор архитектуры уровня приложений  
**Выбранная архитектура:** Layered Domain-Centric Architecture (многоуровневая архитектура с доменным ядром)

**Обоснование:**  
* **Сложность предметной области скоринга.** Логика оценки рисков, применение стратегий и правил, агрегация по структуре «головная компания → филиалы», формирование объяснений решений — всё это требует выделенного доменного слоя, независимого от инфраструктуры.  
* **Чёткое разделение ответственности.**  
  - Доменный слой реализует правила, агрегаторы, сценарии скоринга.  
  - Прикладной слой координирует выполнение процессов и взаимодействие сервисов.  
  - Инфраструктурный слой отвечает за внешние интеграции, базы данных, брокеры событий.  
  Такое разделение облегчает сопровождение и развитие платформы.  
* **Высокая тестируемость.** Поскольку доменная логика изолирована от внешних зависимостей, её можно тестировать автономно, что критично для верификации правил и стратегий скоринга.  
* **Гибкость изменения бизнес-логики.** Регуляторные требования и внутренние политики часто обновляются. Архитектура позволяет менять правила и стратегии без необходимости рефакторинга всей системы.  
* **Поддержка асинхронных сценариев.** Доменно-центричный подход упрощает интеграцию событийной модели: сервис может публиковать события о завершении скоринга или реагировать на обновления стратегий.

---

## Диаграмма контейнеров
![image](https://uml.planttext.com/plantuml/png/ZLPDRzj64BtpLsnrab6vkkGK54KisKdJrhKY6kdHM8bRbZL52YJbr0WAI5PJ5r40zHCDUd5pwBLW50kHPKl-2xl_g3Epf4IJ3Xe3ihZS-NZxPkQjxeKH3w9UnsLrtukuuuaWtApxdQxhSAyjpBOVE9vjVE1uRRTd2VPn_KwrVkTWPVo9OKHEv8grExCSRWUySvV9CtafOtMYVf1BrPTNCcPg8EUqcCc5V0ClS0axrsWfftAX1fkug-tan30Q0CZtoe4J7XvmZxl7OLHX9Vctjja4hmLyBc5v0a7dfOtK8um27Wdx-81R7ST3dv_bEQJ2pE0BiFe83j6mp78Ai6Ro0k8O9kGIdOEQt_Ci7XoNGrfZfFy_M6c3FJKC1dyhOifFH0kSlnAjJRJ2NAeF8QRoGWrAi6CsjbYvANWxgZxkYGry9L_c3LbAJ-7eo1UJCmRltd5EihZrh3U1ygu3kRSQ3za3eF09FsPLLhj_7m5zxaUkurK_uqUSVSZCBdO7g_b-H6Z_WCoODrPZHYbF0HOKUQAUGUvp6LSG_b8D2J4GLK6qamJTJBu5pny1zRW0ATIDY2pf9sQAuCXf2yoBQAeQ6S7uLOsWevP0pKADWRoJN7aewnjO4G70xL0RfOqt0vXfQTVR4F2rUgx6OBB0LeLOqmBEeEUA430qyRk0yKlV4Xzz6nB8Vx08L2C8VKsB2OIyHBvcnIe19WG0I0PmjYamEmFxM2ERK8SCuDMI8bAbTVHRC5eyujn6BhVXYMtPTY1i7Zc-bmNymASgPxkGE4ig10XPlRbB1PqYAeqN6QiWtPXyZgO4QG4LkQ9Z0NXDCfaMMwzdEJZWM_XTROgk7qItLBl8O0nLEjUFizkh1m1FyH4ybWJtHJxxTgjiuOZkTKL0V72NRTSmLyCF8s39VFH5IQlFLBw5m9Vg1SH39HZf6YvrAU1xNg4QqkOGdaOLHcTZ84kEMpXtHuGXjqMhXyspgnUi5lY78g2nuryVycgJjrfEjFk8CBoYVXpX9AdnNPQCXOboBYmOsEG5QRc0LwXetwz4RUUxgDmEKCoWY1WFjZDRq10E2EWn9lgBkBoc_p4DxeJgUN6ZzCKOjd-4xhyHsnbBDIhQXd3P8E6XFbcnUQRPrfaL0_1Uz0sAWxcNx1bDuUPk3Qs_yYL5McBng8ZG4kZpwVv-mmH7zClV7U_dIQvagWMqkMFk9m7fnLh3v9zmHEeN5FJyrP6uLkrcevxrJxK5sYy69LXYTw7tig0Me7fNcgKyXdI-RY3Giw5-GHsMauoR9YE5lSf_HXmCOPmI_LaaWFhf72HQbaEmYhHMe1MEMFsKLRHpwW6g_3k3j98nKC-q4AHQtozE9rslCBov43UcJfw1pskyJw7tXkcraHao3B9p9BmePMPr33r9ylSrQFnXWv6cf9nnxWmhn-oSlQBX649oL8n-ziUHLl9yGLTHaadxbte8HV2KheavvYzG2TTwtl-zfY-hPUiusSwY2GGMrpA8bpyrrPmkJA9Xx-4sThLnJtWM_Euq_WC0)
```PlantUML
@startuml Containers-ComplianceScoring
!include <C4/C4_Container>

title Container diagram: Платформа скоринга комплаенс-рисков

Person(risk_analyst, "Риск-аналитик")
Person(strategy_admin, "Администратор стратегий")

System_Boundary(system, "Платформа скоринга") {

    Container(web, "Web-интерфейс", "SPA (React/Vue)", "Интерфейс риск-аналитиков и администраторов стратегий")
    Container(api, "API Gateway / BFF", "Kotlin/Java + Spring Boot", "Единая точка входа, роутинг, авторизация")
    Container(scoring, "Scoring Service", "Java/Kotlin", "Выполняет скоринг, агрегирует риски, формирует объяснение")
    Container(rules, "Rules Service", "Java/Node.js", "Хранит правила и стратегии, обеспечивает версионирование")
    Container(dataagg, "Data Aggregation Service", "Go/Java", "Интеграция с CRM, реестрами, санкционными списками")
    Container(audit, "Audit/Reporting Service", "Java", "Хранит аудит и формирует отчетность")
    ContainerDb(db, "Operational DB", "PostgreSQL", "Результаты проверок, статусы, audit trail")
    Container(messagebus, "Message Broker", "Kafka/RabbitMQ", "События: ScoringStarted / ScoringCompleted")

}

System_Ext(crm, "CRM", "")
System_Ext(registry, "Корпоративный реестр", "")
System_Ext(gov, "Госреестры", "")
System_Ext(sanctions, "Санкционные списки", "")
System_Ext(auth, "SSO", "")

Rel(risk_analyst, web, "Использует", "HTTPS")
Rel(strategy_admin, web, "Использует", "HTTPS")

Rel(web, api, "REST")
Rel(api, scoring, "Запрос скоринга", "REST/gRPC")
Rel(api, rules, "Работа со стратегиями", "REST")
Rel(api, audit, "Запрос отчётов", "REST")

Rel(scoring, rules, "Получает наборы правил", "REST")
Rel(scoring, dataagg, "Запрашивает данные", "REST")
Rel(scoring, db, "Сохраняет результаты", "SQL")
Rel(scoring, messagebus, "Публикует события", "Event")

Rel(dataagg, crm, "Данные клиента", "REST")
Rel(dataagg, registry, "Структура компании", "REST")
Rel(dataagg, gov, "Юридические данные", "API")
Rel(dataagg, sanctions, "Санкционные статусы", "API")

Rel(api, auth, "Проверка токена", "OIDC")

@enduml
```
# Диаграмма компонентов Scoring Service
![image](https://uml.planttext.com/plantuml/png/ZLPDRzj64BtpLsnrQGB4q2MdeYXYoKuGD84TgNU3ahOC4OeKI2gDK1I8R3W9wW9kSWbGe2boA7gLxGXM_47_XTr_r3T3IboIicL17zpLSjxiUpDl-I2AxJ0UzZtH3Fg3m9T-7Asrkq7e-avRXY_ThhI-SVskD-n9yNtpVhrvVsUnyWVBYjtOa_czekVQJcZtlnDP334FOZK3FxPTNuOxZM3ez-nmT2TArzmJjTBgsbtngsK9l1QHxzW3rrYullsuXY453GFFao6Dr3_wjPgfItmkr4HDzQvGfxYumdIYn_hWdi3lhlgY9lez_iJqL9tgFNMk_y3qHE_gWzhT4cuGTf_BA0xjE0YD1BQCQSBU2FhOtM83bCqdn3z35cF4HLxwDP0ufsDzeAvKeiwHwfJJdGeznjeao_z4JIfvE64m7DWEiVM8BiMwuuJIAU4_2Hoto-X_gXFUtuc1p6aGq4nWdX98r37m4lLL43nekm9VbD6iaa0uzAJq7KY8y2qCn2QFbg9_qc-ohHrdIcGqWEGqASe1xoFUy0J_N8ALgM2n9hXhXkogBCXN0u-h222RTEtRiHlueZ5qlPvSpiHd1awmTN2Vg2Fzkpu42mRtu4c_IuVwiBfnEGWCvLiOHYve7p7Q-XQ3_CjfNpB8ihfADqPqJ_dth0B3nIITKTTpCIJ0R48qUOPW7lkO27dBQS6JY7fFlxsPv9ivxTcnlG3Tm61HMiJixS6fhy10fcD25NLE_6A6IDYlYlaItuDXXxIaIx4zx7Xkz3pLiG87GeyuujcydCSivm72fxhzPbdVYZL-2VM158QiY_wQ4Jj8vm9gf8kizYYAPNzdynNw2rp_Oe-016yHZzAHg8l6muSC_rU-6Gf6AXvpDn6XHXngaoZhaAZW1_CIoLk-9DRNibZ5oAHJQW0LaKoOlzb6o00fuLEkEKENWaW5XKfe9ClMGAOAs9xOQ53uxI2AWTj-zgIul1C6BxWhdyeeiXqf6Zn1j_ne__p21fCjkzDnuwVFSACXqsVOuI7InCvXHIYdNMwVI-wb_ToMafGv98eDpVLhIIyNBZqXr0TixHIVp7WeWzPc-wUwqzfk4ZJTmqjBpgu-yb6nnmQPLNHwShnX4nzNFHftkVxQ80owsFGyTZ5imJKNjWY0ej3bIDL0XlyRtZv1GDH-xkmWvnR1J1k50Wdu_v8P_tI9HwAvLiVCR9B2wVsqr27izBfpTdN0YY5c2b7Bl3DEbfAzhGvSCBphWXfzSXj2SojRRLILibdM3QWtFfld5dR1Sxc98JsQCXgXpDVSYQf6NI2iNZqqoZr3qKtISYUwZhs9-VHnWmKjGqeBYVr7_KS7QXgq3XEOqJ5zGqMbjaH1Sa4MphHIcFb3QvkUYMMlJYUL9t5LcvzG9DO3wVVmmFm_)
```PlantUML
@startuml Components-ScoringService
!include <C4/C4_Component>

title Component diagram: Scoring Service

Container_Boundary(scoring, "Scoring Service") {

    Component(api, "Scoring API", "Controller", "Принимает запросы, отдаёт результаты")
    Component(orchestrator, "Scoring Orchestrator", "Domain Service", "Оркестрирует выполнение скоринга")
    Component(groupagg, "Group Aggregator", "Domain Logic", "Агрегирует риски головной компании")
    Component(ruleengine, "Rule Engine", "Domain Logic", "Оценивает стратегии и бинарные правила")
    Component(explainer, "Explanation Builder", "Domain Logic", "Строит объяснение решения")
    Component(repo, "Scoring Repository", "DAO", "Хранит результаты проверок")
    Component(strategyclient, "Strategy Client", "Integration", "Получает стратегии и правила")
    Component(dataclient, "Data Aggregation Client", "Integration", "Получает данные о клиентах")
    Component(eventpub, "Event Publisher", "Integration", "Публикует события")
}

' ------ Внешние системы ------
System_Ext(gateway, "API Gateway / BFF", "Клиент сервиса")
System_Ext(rules, "Rules Service", "Хранение стратегий и правил")
System_Ext(dataagg, "Data Aggregation Service", "Нормализованные данные клиента")
System_Ext(db, "Operational DB", "PostgreSQL")
System_Ext(broker, "Message Broker", "Kafka / RabbitMQ")

' ------ Связи внутренних компонентов ------
Rel(gateway, api, "Вызывает", "REST/gRPC")

Rel(api, orchestrator, "Запускает процесс скоринга", "in-process")

Rel(orchestrator, ruleengine, "Оценка правил", "in-process")
Rel(orchestrator, groupagg, "Агрегация рисков", "in-process")
Rel(orchestrator, explainer, "Создание объяснения", "in-process")
Rel(orchestrator, repo, "Сохраняет результаты", "in-process")
Rel(orchestrator, strategyclient, "Запрашивает правила", "in-process")
Rel(orchestrator, dataclient, "Запрашивает данные", "in-process")
Rel(orchestrator, eventpub, "Публикует события", "in-process")

' ------ Связи с внешними системами ------
Rel(strategyclient, rules, "Получает правила/стратегии", "REST/gRPC")
Rel(dataclient, dataagg, "Запрашивает агрегированные данные", "REST/gRPC")
Rel(repo, db, "Читает/пишет результаты", "SQL")
Rel(eventpub, broker, "Публикует события ScoringStarted/Completed", "Event")

@enduml

```

# Диаграмма компонентов Data Aggregation Service
![image](https://uml.planttext.com/plantuml/png/bLVBRjjM4DtpAswzgGt4q2QheYZYo44Hm6rTgtk3bGY6W4GA96tKBGf8LjmQS1v8qw11YkQ50jq5h1mXj6HBlt3lB_HBUcRul2ALDjwOTyoPCuypvz8tWj3qmuDULpIzNjzpBJSCDhVDqDooRT-opT3nt9Rb7pfjo_Z8STlTWuubFcrURpIlx-SUdnb6w8HTguWXEevf-sRl4q6nH2cOoA8PJSyDJSUr_FsRte7RCVsZzGvUDctxcbXRvRQs8Num386V76lTx3lmAB-yjNjx3IP2yBrkr_Bf95_BkVfPJcKav5YToBbyIuzlvKmz57A1yqoDyC94NicOJ94yLmCSpt4SuCzZMAxabJfJ9-IM7zOsbX9gUvxVMKgfIJR7DKEFazdsUgXTx7Ysq-RadWDYAYTgg8xn140rmkbOgA7y6taFD4GXO_aELU0yKQUSBzd8cMk40S6GSJKrlxVLCVkXvLD_zhuGwObokEs65eP6wN9Alm5mGPMhOGqwd2lHKMqHVS-odI3qZ_ulH58wofxYzmXFPu860CC2x-105LtY475z5nLqspiiq3_t3jy7_48Vjvm8zO3wZZfZCgBjqwIhrRQVKxhCZ6nSTPqEJBTDS46HKoiplJUpb6iJ9aIqN7RCL2LQq7cgVe9jdYGeBycy81wGfxoivEHwVi_iEjzpDhnYN-QMMbR-GTG3UYHdL7lAoQ8dw9iS2tG2RSDvf1u8l4sRDK5xxrNfPxRlM1bsamu4qy9Ic3O_ocVglZhzeDtxqJ0-5flfZv2_54j1KnscQp6Hb-1GzfhHEWf2gxT_wpiCgktvZakYqqgUSgtXVBXoa2JPHPhAc3iSbPECU0uniYa5ngwbIvOOETvJHBgRTPMyI6dcDEKS1_QJQrJbdAW6E7exubfBqVrqlLZd_3uw65eYspc6UWMPUoZV4MKepYnfup-3nqWWPZYCLPtfOR54wVQaiHmh4EloYNmiNyetylU6_1EiUBN126-gAoClU96HBotni49V7IrV4cqxTe0RC80-JAfap3GxvhvV5KEkBiD4ewaEs_aMmR_gMydQcLsnVPFWThqWX6Atljv1nLCYOeCMYhhFKyY4DTv0E8rqfLPdzL8UoJZdj295tl4oZt9IxbdTWet9hPOhP8GknBm6l84hzsEvRXBYMwrl6lRURfEo9WmEdLrIIpTFoadoSzpDlk-rhI3875CVxH9PUKraq70k-MfNn8VeVtquJVFrSF-jvnIIFXMqY3MwNT5aZN0NHPXo74rhSTzCU5v0LuzOIf30e7PSMjifO3bI9g3oBzOqvdy3n8iItOVarWukcNZkYrUFCoRHw4-hNYlO7N6s2m9FUY9ewgKF4N0_rbINBYAOD9AN2PIePEb2plFA1J5LbdfcfqqgyweaZLgj5OQDglAaHotJAr765GjPnpPIfhluEw6TwLohagoiX6M-eV7fPs2Tn9MbjeoXZUy9d6U9D658WbhCVZ4B6hubJbZXw2iIhQRx27hrIFwwDBE4FIoTyZbQa6L2byYz9D0GcvS6I_DUdXYqbTAxORaT_DDX_0i0)
```PlantUML
@startuml Components-DataAggregationService
!include <C4/C4_Component>

title Component diagram: Data Aggregation Service

Container_Boundary(dataagg, "Data Aggregation Service") {

    Component(api, "Aggregation API", "Controller", "Точка входа для получения агрегированных данных")
    Component(coord, "Aggregation Coordinator", "Domain Logic", "Оркестрирует сбор данных из внешних источников")
    Component(crmAdapter, "CRM Adapter", "Integration", "Запрос данных из CRM")
    Component(regAdapter, "Registry Adapter", "Integration", "Запрос структуры группы компаний")
    Component(govAdapter, "Gov Adapter", "Integration", "Получение юридических данных из госреестров")
    Component(sanctionsAdapter, "Sanctions Adapter", "Integration", "Проверка компании по санкционным спискам")
    Component(normalizer, "Data Normalizer", "Domain Logic", "Нормализует данные в единую модель")
    Component(cache, "Data Cache", "Storage", "Кэш агрегированных данных")
}

' -------- Внешние системы --------
System_Ext(scoring, "Scoring Service", "Клиент сервиса агрегации")
System_Ext(crm, "CRM System", "Информация о клиентах, договорах")
System_Ext(registry, "Corporate Registry", "Связи голова–филиалы")
System_Ext(gov, "Gov Registries (ЕГРЮЛ/ФНС)", "Юридический статус компании")
System_Ext(sanctions, "Sanctions Lists", "Внешние и внутренние санкционные данные")
System_Ext(db, "Operational DB", "PostgreSQL (кэш/справочники)")

' -------- Связи компонентов внутри сервиса --------
Rel(scoring, api, "Запрашивает агрегированные данные", "REST/gRPC")

Rel(api, coord, "Оркестрация", "in-process")

Rel(coord, crmAdapter, "Запрос данных", "REST")
Rel(coord, regAdapter, "Запрос структуры группы", "REST")
Rel(coord, govAdapter, "Запрос юридических данных", "API")
Rel(coord, sanctionsAdapter, "Проверка санкций", "API")

Rel(coord, normalizer, "Передаёт сырые данные", "in-process")
Rel(normalizer, cache, "Чтение/запись", "in-process")

Rel(api, cache, "Читает кеш", "in-process")

' -------- Связи адаптеров с внешними системами --------
Rel(crmAdapter, crm, "Получает клиентские данные", "REST")
Rel(regAdapter, registry, "Читает структуру компании", "REST")
Rel(govAdapter, gov, "Запрашивает юридические факты", "API")
Rel(sanctionsAdapter, sanctions, "Проверяет санкционные статусы", "API")

' -------- Если кеш частично хранится в БД --------
Rel(cache, db, "Опционально сохраняет данные", "SQL")

@enduml

```
