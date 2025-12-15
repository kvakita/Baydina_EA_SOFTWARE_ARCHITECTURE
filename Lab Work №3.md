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
## 3. Диаграмма последовательностей (UML Sequence)
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
## 4. Модель базы данных (UML Class Diagram)
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
## 5. Реализация клиентского и серверного кода
# 5.1 Серверная часть (Scoring Service)
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
# 5.2 Клиентская часть

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
Принципы SOLID применены частично, в объёме, достаточном для лабораторной работы:
- **SRP (Single Responsibility Principle):** каждый класс выполняет одну чётко определённую задачу (оценка правил, агрегация риска, оркестрация процесса, интеграция с внешними системами).
- **DIP (Dependency Inversion Principle):** зависимости между компонентами заданы через интерфейсы, что позволяет подменять реализации (например, интеграционные клиенты) без изменения бизнес-логики.

---

## 8. Дополнительные принципы разработки

Помимо базовых принципов, были рассмотрены дополнительные подходы к проектированию и разработке программных систем.

**BDUF (Big Design Up Front).**  
Применён частично. Полное детальное проектирование системы на раннем этапе было сознательно отклонено, так как требования к правилам скоринга и регуляторным ограничениям подвержены изменениям. Архитектура уточняется итеративно по мере развития системы.

**SoC (Separation of Concerns).**  
Применён. В архитектуре чётко разделены зоны ответственности: пользовательский интерфейс, оркестрация сценариев, доменная логика скоринга и интеграционные компоненты. Это повышает читаемость архитектуры и упрощает сопровождение системы.

**MVP (Minimum Viable Product).**  
Применён. В рамках лабораторной работы реализован минимально жизнеспособный функционал — скоринг одного филиала с учётом головной компании и сохранением результата, что позволяет продемонстрировать ценность выбранного архитектурного подхода.

**PoC (Proof of Concept).**  
Применён. Реализация ключевого сценария скоринга подтверждает работоспособность концепции агрегации рисков на уровне группы компаний и целесообразность выбранных архитектурных решений.



