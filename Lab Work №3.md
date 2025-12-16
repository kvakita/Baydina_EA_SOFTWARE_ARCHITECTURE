# Лабораторная работа №3  
## Использование принципов проектирования на уровне методов и классов  

## 1. Цель работы

Получить практический опыт проектирования и реализации программных модулей с использованием принципов KISS, YAGNI, DRY, SOLID, а также проанализировать применимость дополнительных принципов разработки (BDUF, SoC, MVP, PoC) на примере дипломного проекта.

---

## 2. Выбранный вариант использования

### UC-01: Запуск комплаенс-скоринга филиала с учётом головной компании

**Описание варианта использования:**  
Риск-аналитик инициирует комплаенс-проверку филиала компании. Система определяет связанную головную компанию, агрегирует данные по всей группе юридических лиц, применяет правила и стратегию скоринга, формирует итоговый результат и объяснение решения, сохраняет результат во внутреннем хранилище, **записывает итоговый статус проверки в CRM**, после чего публикует событие о завершении скоринга.

---
## 3. Диаграмма компонентов Scoring Service
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

## 4. Диаграмма последовательностей (UML Sequence)
![image](https://uml.planttext.com/plantuml/png/RLJ1Jjj05BplLppb119HuSgX8d44YjG8Y1lrmcNZh18hndRjDHKt0bKhbGgEFVSF0ZGb119-OVUFEhkEqxeWPUDTpxot--RDUYULF97PSK2k-9Y9q1FLZhEvnQGl0q4TfxgvjKtgfvgfXJvJKpLNTwG_O_A8CSR_cFI8z8N-gYRwN2tK4waxF1unRi5Ug0SomUf9FOC_HzIKpEyUgsSrrIFzNLyHSa_KlRvGZrYUgx4Pf6x9QRSOuvwC4ggq_NH8jP07fwci44-BEOjHWU_vCGybLJw8BdMQ5kyqor3je4bxN8fF_DJYzTtTCi7reiGF-v0YEV4zOKckK-OqkEHKw_SJqUVIZy8IkL4hazjP8D8Ie-sM6IYGN1GXZYriwkrXXovF1LNsOv7OJNX0ZNg-GvsXJHd0baXJtXTKJwAXI2pOEcHiAGfLtw9-Gkz-6A7He96Q4IPdjZ4JF6Tgoa014KA0UR3llgStta1umvIPLq1S1u2ro5ARjMx2Gs_GxAqp1X2KHWsKlf1xIPJ5Hfr_AC2gHHRro_P_hY_L11xHP-ZsCmvq-Ry_7ehCRJVFR5LSMsTOhxB1D-UemHXb0iDDzRLrbQblZFgCFyT5dZp0a4UvJSc8yTVGQdmK8khEcX0h_SNcCt1ELBT78Knl0fxmBfSmFonAMQWR_KrVWpf7_Dp4hhIf6_duYLYgrnPf5aYK0c1LIZGiwCOCsvJ5S81mJ1zSoMMMhWJTe9xm_1I6sA04j-6TE171EZDXhsHh7M8PPzt0JmUqF80J7SU1aA9dEQlMlxn-BwrY7L6uoiXlQCPNhxiApCAAwFt2VXjkR7T-vof1kjTkFMCx8kpXWyJ-0000)
```PlantUML
@startuml Sequence-UC01
title UC-01: Скоринг филиала с учетом головной компании и записью результата в CRM

actor "Risk Analyst" as Analyst
participant "Web UI" as UI
participant "API Gateway" as BFF
participant "Scoring Service" as SC
participant "Data Aggregation Service" as DA
participant "Rules Service" as RS
participant "CRM System" as CRM
database "Operational DB" as DB
queue "Message Broker" as MQ

Analyst -> UI : Запуск проверки
UI -> BFF : POST /checks
BFF -> SC : startScoring(branchId)

SC -> DA : getGroupData(branchId)
DA --> SC : данные группы компаний

SC -> RS : getRules()
RS --> SC : стратегия и правила

SC -> SC : расчет скоринга\nагрегация рисков\nформирование объяснения

SC -> DB : saveScoringResult()
DB --> SC : ok

SC -> CRM : updateCheckStatus(branchId, decision, riskLevel)
CRM --> SC : ok

SC -> MQ : publish ScoringCompleted
SC --> BFF : результат проверки
BFF --> UI : статус проверки
UI --> Analyst : отображение результата

@enduml
```
---
## 5. Модель базы данных (UML Class Diagram)
![image](https://uml.planttext.com/plantuml/png/RPBFQW8n4CRlUOfXZmMblUx9gXI4zkAV1qWtWuPcipR95gGKQg4d-mXz2A4Lf6hx2iaRESfQTVMqCr---KrciZNhk75vgg1Plyi4AkQaKmZ-q__wRp_pY_0154pzog_29Bn36Fjv68StEbk62VWVCYxzdr-GPqQUSOKS98PNFwUYBpzbB57SMcXawP3h4Jmp02bYwFLQJGerJp66ZZDIzobreo6bXRTB2Nif0Tgek9EPte86o4MXj_RCUyDrCYZh_w1EacswjV4nH-lA5pfV342hShcDeIZhR5FI4uFSeRXsCJGfwCeKpxIokJhfPAHzkCKdL0JTXIGBJAIb0ObNH7lUndSZjI3c2IrNsA0tF5ocaVq-6YHJFSiKyYrfzn3HLGIL2aINrUf5LDukAorfPwRFqKpiUC9Zvi5j6QkX5lIdiJK0)
```PlantUML
@startuml DBModel
title Модель данных системы комплаенс-скоринга

class Company {
  id: UUID
  name: String
  type: CompanyType
}

class CompanyRelation {
  headCompanyId: UUID
  branchCompanyId: UUID
}

class ScoringRequest {
  id: UUID
  branchCompanyId: UUID
  status: RequestStatus
  createdAt: DateTime
}

class ScoringResult {
  id: UUID
  riskLevel: RiskLevel
  decision: Decision
}

class RiskFinding {
  id: UUID
  ruleCode: String
  triggered: Boolean
}

Company "1" -- "0..*" CompanyRelation
ScoringRequest "1" -- "1" ScoringResult
ScoringResult "1" -- "0..*" RiskFinding

@enduml
```
---
## 6. Реализация клиентского и серверного кода
# 6.1 Серверная часть (Scoring Service)
```Kotlin
data class RuleFinding(
    val ruleCode: String,
    val triggered: Boolean,
    val category: String
)

data class ScoringResult(
    val riskLevel: String,
    val decision: String
)

class RuleEngine {
    fun evaluate(data: Map<String, Any>, rules: List<Map<String, Any>>): List<RuleFinding> =
        rules.map {
            val triggered = data[it["field"]] == it["equals"]
            RuleFinding(it["code"].toString(), triggered, it["category"].toString())
        }
}

class GroupAggregator {
    fun aggregate(findings: List<RuleFinding>): ScoringResult {
        val score = findings.count { it.triggered }
        return when {
            score >= 3 -> ScoringResult("HIGH", "REJECTED")
            score == 2 -> ScoringResult("MEDIUM", "REVIEW")
            else -> ScoringResult("LOW", "APPROVED")
        }
    }
}

interface DataAggregationPort {
    fun getGroupData(branchId: String): Map<String, Any>
}

interface RulesPort {
    fun getRules(): List<Map<String, Any>>
}

interface CrmPort {
    fun updateStatus(branchId: String, decision: String, risk: String)
}

class ScoringOrchestrator(
    private val dataPort: DataAggregationPort,
    private val rulesPort: RulesPort,
    private val crmPort: CrmPort
) {
    private val ruleEngine = RuleEngine()
    private val aggregator = GroupAggregator()

    fun score(branchId: String): ScoringResult {
        val data = dataPort.getGroupData(branchId)
        val rules = rulesPort.getRules()
        val findings = ruleEngine.evaluate(data, rules)
        val result = aggregator.aggregate(findings)

        crmPort.updateStatus(branchId, result.decision, result.riskLevel)
        return result
    }
}
```
# 6.2 Клиентская часть

```Kotlin
fun main() {
    val orchestrator = ScoringOrchestrator(
        dataPort = object : DataAggregationPort {
            override fun getGroupData(branchId: String) =
                mapOf("status" to "ACTIVE", "sanction" to false)
        },
        rulesPort = object : RulesPort {
            override fun getRules() = listOf(
                mapOf("code" to "SANCTION", "field" to "sanction", "equals" to true, "category" to "REG"),
                mapOf("code" to "INACTIVE", "field" to "status", "equals" to "INACTIVE", "category" to "REG")
            )
        },
        crmPort = object : CrmPort {
            override fun updateStatus(branchId: String, decision: String, risk: String) {
                println("CRM updated: $branchId -> $decision ($risk)")
            }
        }
    )

    val result = orchestrator.score("BRANCH-001")
    println(result)
}
```
---
## 7. Применение принципов KISS, YAGNI, DRY, SOLID

В процессе проектирования и реализации модулей системы комплаенс-скоринга были применены базовые принципы разработки программного обеспечения, направленные на снижение сложности, повышение читаемости кода и упрощение сопровождения системы.

**KISS (Keep It Simple, Stupid).**  
Логика скоринга реализована на основе простых бинарных правил («выполнено / не выполнено») и подсчёта количества сработавших условий. Такой подход исключает использование сложных формул и специализированных языков описания правил, что делает алгоритм прозрачным, легко проверяемым и понятным для дальнейшего сопровождения.

**YAGNI (You Aren’t Gonna Need It).**  
В рамках лабораторной работы и минимального сценария использования сознательно не реализованы избыточные механизмы, такие как кэширование результатов, сложные DSL для описания правил, распределённые транзакции и оптимизации преждевременного характера. Реализована только функциональность, необходимая для поддержки выбранного варианта использования.

**DRY (Don’t Repeat Yourself).**  
Механизм оценки правил и агрегации риска реализован в виде единых компонентов, которые переиспользуются в рамках всего сценария скоринга. Это позволило избежать дублирования логики обработки правил и расчёта итогового уровня риска.

**SOLID.**  
### Принципы SOLID

**S — Single Responsibility Principle (принцип единственной ответственности).**  
Каждый класс в системе отвечает за одну чётко определённую задачу. Например, компонент оценки правил отвечает только за применение правил к данным, агрегатор рисков — только за расчёт итогового уровня риска, а оркестратор — за координацию всего процесса скоринга. Такое разделение упрощает сопровождение кода и снижает риск побочных изменений.

**O — Open/Closed Principle (принцип открытости/закрытости). ** 
Компоненты бизнес-логики спроектированы таким образом, чтобы их можно было расширять без изменения существующего кода. Добавление новых правил скоринга или категорий рисков возможно за счёт изменения конфигурации или добавления новых реализаций, не затрагивая уже работающие модули.

**L — Liskov Substitution Principle (принцип подстановки Барбары Лисков).**  
Интеграционные компоненты и клиенты внешних сервисов реализуют общие интерфейсы. Любая конкретная реализация (например, тестовая заглушка или реальный клиент CRM) может быть подставлена вместо другой без изменения корректности работы системы.

**I — Interface Segregation Principle (принцип разделения интерфейсов).**
Интерфейсы определены узкими и специализированными. Каждый интерфейс содержит только те методы, которые действительно необходимы конкретному компоненту. Это предотвращает зависимость компонентов от методов, которые они не используют, и упрощает реализацию альтернативных адаптеров.

**D — Dependency Inversion Principle (принцип инверсии зависимостей).** 
Высокоуровневые компоненты (например, оркестратор скоринга) не зависят от конкретных реализаций интеграционных сервисов и хранилищ данных. Все зависимости задаются через абстракции (интерфейсы), а конкретные реализации внедряются извне, что повышает гибкость архитектуры и упрощает тестирование.


---

## 8. Дополнительные принципы разработки

Помимо базовых принципов, были рассмотрены дополнительные подходы к проектированию и разработке программных систем.

**BDUF (Big Design Up Front).**  
Не применен

**SoC (Separation of Concerns).**  
Применён. В архитектуре чётко разделены зоны ответственности: пользовательский интерфейс, оркестрация сценариев, доменная логика скоринга и интеграционные компоненты. Это повышает читаемость архитектуры и упрощает сопровождение системы.

**MVP (Minimum Viable Product).**  
Применён. В рамках лабораторной работы реализован минимально жизнеспособный функционал — скоринг одного филиала с учётом головной компании и сохранением результата, что позволяет продемонстрировать ценность выбранного архитектурного подхода.

**PoC (Proof of Concept).**  
Применён. Реализация ключевого сценария скоринга подтверждает работоспособность концепции агрегации рисков на уровне группы компаний и целесообразность выбранных архитектурных решений.



