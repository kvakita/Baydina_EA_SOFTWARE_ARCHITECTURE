# Диаграмма системного контекста
![image](https://uml.planttext.com/plantuml/png/ZLPDRzD04BtxLomzfL9ABvmGGa2e10VKgkazYXghMAHE5RiL4K9AcduWWaW18H0VAfNW20fj8zmqJl_2xb_Wb_1cTftOZWjKQkjTpywyUVDcrhSylrptDzUhejZmV7kzK7Dz-x4_LwYjRrITihjg5rThTSTTiwuuxcfrisAB6uLhyuLh9MDvqx9ynw_QGXx9GBNKjkpBeGnLHvu9EP0ZFDBMW5vT5_9GdikUsbOxSgIQCEm9rI8pB08PoXDiyAcXl21Bd0nLQuxUQZjvB8pv1V-5Bnmyfe1gbrwTOzddeB2rh2MxxjNSNDtn7fNARhcA9FDYHdxH_kOG8SGl3h6VmGozV8UT2zNcW42WjWMMJVA7HHERmLIWl9Xuah_WhQdQV1BsVVaBYtFr0gQKUaUGWv7ygPwhBeA4v8QSiR5wffUgUtLcDeBl-VMoRwyrIkNAkkCInWDv2eu2jWIpXWam8cBBFlWCv1adzHaxvWqJ5akiXsL0R1EmL3cG6qhPd_OQnE1LxAtQXeSeT-sjnGrFp0lvdIfEDKJgFKvyZt9YA9z85Smuno6MMdXiyKOtug8raPFgv0NyH5ZFOSJWjRnsO7-2SZMZ84GCeREqf4jtQfjkfLnlvBm6WVmF3SxCYYUMm8_saFDq3vGyNfAFkDWppfF9Qn07aI9O604U0I1LTw3Q8gtfX9wu4ZjSbO1L5MdiX0LrabIOzWDH8PpWel4111f5NKZzr6DgcPgdOt8M7lkvrJhbL5X-8EHFjXgO5DuplOObJQwW476qFRMRJsBb7UXpfFLZ-dINuiQ2rkqr1xfkK9mFsHqDy9nkNwkF8HtH4r8Vu3_NN90Y4FSqwflhIGfwX3b10JP-xxrEO5QT5BgrsXP5UyEwZ650z-RaM-n_aJ_aHwJv5PrrDClGVh00JX5Wdyof9APpM6uOA2Ha612AaDkAo9p0fncnbq6LXpdiEpNNqyhZ2b4rk8z2qv_8pkX0gu_c2eQ9xYOI1r7QvYuceiPTbkKDlleyZFR9BeMclEa_d6u1m6xpWZGHCBS3hHd4pXMBYvgZ0ymJq_UOasDD1Od-ZW_44RVxC0D5nV4r389wJ60O11TWsmZu6xkSJDYu0KVbRkBnt3MuJDECmDI-UeNDqCmcFLEMxMhgDfaU3l-wG5GxRtPWCcHnD6D3EUlEe6pkhQmi5O618AHlX6aG_p_mqvCz7ai7IpcF571mMSbLTvw2P5MQ9w9wWIsYGqUbWlCjjNZ_RY5AC1LJpoX3BS5z5iygqLE9wKHkbnUAAvUuZK-XRD-cxySJ1wERfZ3f5TQTz7Wz-qT0CrWEcOOs6in4UPavFEA9aUFRI_SlyPkO4OSJjKLN3rqDiI7HayE_-nltrw4U1OI4sk7IBngwm_5Dqs4jy4M_i6Mx_hpXmhfbknLyPrf_0000)
```PlantUML
@startuml SystemContext-ComplianceScoring
!include <C4/C4_Context>

title Системный контекст: Платформа скоринга комплаенс-рисков корпоративных клиентов

Person(risk_analyst, "Риск-аналитик", "Запускает проверки клиентов, анализирует результаты и объяснение решения.")
Person(strategy_admin, "Администратор стратегий", "Настраивает правила и скоринговые стратегии.")
Person(devops, "DevOps / Эксплуатация", "Мониторит состояние системы, управляет конфигурациями.")

System_Boundary(sys, "Платформа скоринга") {
    System(scoring_system, "Система скоринга", "Выполняет оценку рисков клиентов и филиалов, агрегирует риски по головной компании.")
}

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
![image](https://uml.planttext.com/plantuml/png/ZLLTInj157tVNp7DauAbBpwLKjGeHIWe-WDaJOPrOBCJ9hjIAGKJqPQcOFlacyM7lg_QOAfrynTc_b7FdLtNJMOhXdpCt9svvznppqoMUh6lugGJigRiT6N4exZtShibbOY2RQxsHOjxBqJK2fCsPs-QYw-QYplbohUU5uiuv8zxMLluWV8xhrcHWpqayPeoYdqHSRMp8fEexQk3kLw-Pe4rPbOtvjb7psDubPdd_AweB5pUN6zWIacL34EkQAP_caCzrlVu_DMf7fa1qpSOJ13EJDyC5nX-1_gtJiqFF4Huf6_CaRuptn1EpS0C6_Cpk5ArzdWlLduiLON0HYLCsAkoW-hA0eZDEVBVWaKVUS7B70B9SheoGptHcRu3rP6bEsAcZxLPmVzQfmwFGCcawmUarZiQikKWK3oOmNyl0z6owD_rjQtlke9iQH3Gc25E139z1RnC_s443zacq8iOZHq2AWavZmAq4EctC65hTbQBVcvEYjAkYauKCa3ad1HnmFlI5fpYoJrK6J7Rh1Ixnc3dgi0_T4Fh8e2iqJZoOo4ZjfA8iCthbRYmm1bAX_QPlZHVpIbKg6WFdSoNV6fEtS9vLrOwluLfJq3s0uksl651VbdQznQapbTvOIJtoFxUEZ3MJ3muQ8K2Km8i0gnf8mIp7Y6WRCcvuQc9viXyVbhafpLj-x5VWgvYKbgBb7q-E9qhA53qCID7tP6-Y908nsupz_7TJNQfbpHacybkA7fxUHyTE0XwQJFUFjYvRzivHFgyRvyyRuk7-LKnSsxrcRq8Zgpz2xxvlN1YZsY_xWHIikayKuUWT3aoJebSkxLoW2k7CqMAdBoxesfsKhleRbOQrtNpVp8L5jONI7ASEzMSFbm9TNUWcyAnfzl_MPkvIQQizgI1wWLvDCnpxE0jyQYD_pRl7m00)
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

Rel(api, orchestrator, "Запускает процесс скоринга")
Rel(orchestrator, ruleengine, "Оценка правил")
Rel(orchestrator, groupagg, "Агрегация")
Rel(orchestrator, explainer, "Построение объяснения")
Rel(orchestrator, repo, "Сохраняет результаты")
Rel(orchestrator, strategyclient, "Получает правила")
Rel(orchestrator, dataclient, "Получает данные")
Rel(orchestrator, eventpub, "Публикует события")

@enduml

```

# Диаграмма компонентов Data Aggregation Service
![image](https://uml.planttext.com/plantuml/png/ZLJDQXin4BxhALIV4cZ99QTIIjmDb41RYlq0GUmAHR0h6QriI4B1SKWRQDhS2-sXbrpTn4jDNAUlC7gZZjPUhtVjjjwJPlHzy_FDQFTIoupj9Z4DTDBHYYkRFjbdbZM5C5mmAxLgSzEJ8IUFf0hZRiJfis1t9zWzN32U4sAbZNaPWqQI2SEIfzJ7eal1Q16D15fP9XKtXozqLqNC76z526P2FAQDlz4Qs_IK48hV8jSMwqXaB8ERRmyQwF8PZ8vZRhm5t-71VO1x65AuTHVm02D_7C74NLFuZVR4dICWWocClMi8TwwFvXsQVRo-HSyKfkwZk_2qXT7OhXKKQcsYMac1zqd5hCwBsTS9zavVQI73lBXlcE8UCdVc1dZ2XEuShG5rP_03w-x_9wD9cX7hM6vyswtNT6xvq0VASfo5hsA5X-MLl1ONChNcU5EoqBsI_5BtDkMbJ8N-8YtPxSArQGobJS9YUPApysLvi_2irVUh5n5-eOOJ6CFFkRgbg93XLb2SzWYlfxW8docY_Ovak1TNgugpy8WNkGDl-3Hj72yJ-H6-k4_kSdcBXkuznXvZg7U4j7WyMznYMMeRa8FTDO8zSWwgZxbwktQOLSZAoFoBgqB-DPqgiZA3ki2r1fQnXNHmumQprmRPZcVW8nnZ-rTppaoUDM2QwtG9GmIIFQuY_7-HFm00)
```PlantUML
@startuml Components-DataAggregationService
!include <C4/C4_Component>

title Component diagram: Data Aggregation Service

Container_Boundary(dataagg, "Data Aggregation Service") {

    Component(api, "Aggregation API", "Controller", "Точка входа для получения агрегированных данных")
    Component(coord, "Aggregation Coordinator", "Domain Logic", "Оркестрирует сбор данных")
    Component(crmAdapter, "CRM Adapter", "Integration")
    Component(regAdapter, "Registry Adapter", "Integration")
    Component(govAdapter, "Gov Adapter", "Integration")
    Component(sanctionsAdapter, "Sanctions Adapter", "Integration")
    Component(normalizer, "Data Normalizer", "Domain Logic", "Нормализует данные в единую модель")
    Component(cache, "Data Cache", "Storage", "Кэш агрегации")
}

Rel(api, coord, "Оркестрация")
Rel(coord, crmAdapter, "CRM")
Rel(coord, regAdapter, "Registry")
Rel(coord, govAdapter, "Gov data")
Rel(coord, sanctionsAdapter, "Sanctions")
Rel(coord, normalizer, "Нормализация")
Rel(normalizer, cache, "Чтение/запись")
Rel(api, cache, "Чтение кеша")

@enduml

```
