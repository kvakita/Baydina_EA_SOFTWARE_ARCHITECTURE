# Лабораторная работа №4 REST API

## Проектирование REST API для получения результатов скоринга

---

## Цель работы

Целью лабораторной работы является получение практического опыта проектирования, реализации и тестирования REST API. Разрабатываемый программный интерфейс предназначен для получения результатов скоринга с детализацией по сработавшим правилам и вызванным сервисам.

---

## Оглавление

- Документация по API  
  - Работа с запуском скоринга  
    - POST /scoring-runs  
    - GET /scoring-runs  
    - GET /scoring-runs/{runId}  
    - GET /scoring-runs/{runId}/result  
    - GET /scoring-runs/{runId}/rules  
    - GET /scoring-runs/{runId}/service-calls  
    - PUT /scoring-runs/{runId}  
    - DELETE /scoring-runs/{runId}  
- Реализация API  
- Тестирование API  

---

## Документация по API

### Работа с запуском скоринга

---


### POST /scoring-runs

**Описание:**  
Создание (запуск) нового скоринга для указанного объекта (филиала или контрагента).

**Заголовки:**  
- `Content-Type: application/json`

**Входные параметры (Body):**

| Параметр | Тип | Обязательность | Пояснение | Допустимые значения / пример |
|---|---|---:|---|---|
| `target` | object | да | Описывает объект, для которого запускается скоринг | `{...}` |
| `target.type` | string | да | Тип оцениваемого объекта | `branch`, `counterparty` |
| `target.id` | string | да | Идентификатор объекта в системе-источнике | `"BR-001"`, `"CP-7781"` |
| `target.name` | string | нет | Человекочитаемое наименование объекта | `"Amsterdam Branch"` |

**Выходные параметры (Response 201):**

| Параметр | Тип | Пояснение | Пример |
|---|---|---|---|
| `runId` | string | Уникальный идентификатор запуска скоринга | `"sr_20260204_ab12cd"` |
| `status` | string | Текущий статус запуска | `"completed"` |
| `startedAt` | string (date-time, ISO 8601) | Дата и время начала выполнения скоринга | `"2026-02-04T10:15:12Z"` |
| `links` | object | Набор ссылок на связанные ресурсы API | `{...}` |
| `links.self` | string | Относительный путь к ресурсу запуска | `"/scoring-runs/sr_20260204_ab12cd"` |
| `links.result` | string | Относительный путь к ресурсу результата | `"/scoring-runs/sr_20260204_ab12cd/result"` |

**Коды ответа:**  
- `201 Created` — скоринг успешно создан  
- `422 Unprocessable Entity` — некорректные входные данные

---

### GET /scoring-runs

**Описание:**  
Получение списка запусков скоринга с возможностью фильтрации.

**Входные параметры (Query):**

| Параметр | Тип | Обязательность | Пояснение | Пример |
|---|---|---:|---|---|
| `targetType` | string | нет | Фильтр по типу объекта | `branch` |
| `targetId` | string | нет | Фильтр по идентификатору объекта | `BR-001` |
| `status` | string | нет | Фильтр по статусу запуска | `completed`, `deleted` |

**Выходные параметры (Response 200):**

| Параметр | Тип | Пояснение |
|---|---|---|
| `items` | array<object> | Массив запусков скоринга, удовлетворяющих фильтрам |
| `items[].runId` | string | Идентификатор запуска |
| `items[].status` | string | Статус запуска |
| `items[].score` | number | Итоговый скоринговый балл (0..1) |
| `items[].riskLevel` | string | Уровень риска, полученный по результату скоринга |
| `paging` | object | Служебная информация для пагинации |
| `paging.limit` | integer | Размер страницы |
| `paging.offset` | integer | Смещение (номер первого элемента) |
| `paging.total` | integer | Общее число записей, подходящих под фильтр |

---

### GET /scoring-runs/{runId}

**Описание:**  
Получение метаданных конкретного запуска скоринга.

**Входные параметры (Path):**

| Параметр | Тип | Обязательность | Пояснение | Пример |
|---|---|---:|---|---|
| `runId` | string | да | Идентификатор запуска скоринга | `sr_20260204_ab12cd` |

**Выходные параметры (Response 200):**

| Параметр | Тип | Пояснение | Пример |
|---|---|---|---|
| `runId` | string | Идентификатор запуска | `"sr_20260204_ab12cd"` |
| `status` | string | Статус запуска | `"completed"` |
| `score` | number | Итоговый скоринговый балл (0..1) | `0.73` |
| `riskLevel` | string | Уровень риска | `"high"` |

**Коды ответа:**  
- `200 OK` — запуск найден  
- `404 Not Found` — запуск с указанным `runId` не найден

---

### GET /scoring-runs/{runId}/result

**Описание:**  
Получение полного результата скоринга с детализацией по правилам и сервисам.

**Входные параметры (Path):**

| Параметр | Тип | Обязательность | Пояснение | Пример |
|---|---|---:|---|---|
| `runId` | string | да | Идентификатор запуска скоринга | `sr_20260204_ab12cd` |

**Выходные параметры (Response 200):**

| Параметр | Тип | Пояснение |
|---|---|---|
| `run` | object | Итоговые данные по запуску |
| `run.runId` | string | Идентификатор запуска |
| `run.score` | number | Итоговый скоринговый балл (0..1) |
| `run.riskLevel` | string | Уровень риска |
| `rulesTriggered` | array<object> | Список сработавших правил скоринга |
| `rulesTriggered[].ruleId` | string | Идентификатор правила |
| `rulesTriggered[].severity` | string | Критичность срабатывания правила | 
| `rulesTriggered[].weight` | number | Вес правила в суммарной оценке |
| `servicesCalled` | array<object> | Список вызванных сервисов |
| `servicesCalled[].service` | string | Наименование сервиса |
| `servicesCalled[].status` | string | Статус вызова сервиса (`success`/`failed`) |

**Коды ответа:**  
- `200 OK` — результат найден  
- `404 Not Found` — запуск/результат не найден

---

### GET /scoring-runs/{runId}/rules

**Описание:**  
Получение списка сработавших правил скоринга (без итогового объекта и сервисов).

**Входные параметры (Path):**

| Параметр | Тип | Обязательность | Пояснение |
|---|---|---:|---|
| `runId` | string | да | Идентификатор запуска скоринга |

**Выходные параметры (Response 200):**

| Параметр | Тип | Пояснение |
|---|---|---|
| `[]` | array<object> | Массив сработавших правил |
| `[].ruleId` | string | Идентификатор правила |
| `[].severity` | string | Критичность правила |
| `[].weight` | number | Вес правила в суммарной оценке |

**Коды ответа:**  
- `200 OK`  
- `404 Not Found`

---

### GET /scoring-runs/{runId}/service-calls

**Описание:**  
Получение информации о вызванных сервисах в рамках скоринга.

**Входные параметры (Path):**

| Параметр | Тип | Обязательность | Пояснение |
|---|---|---:|---|
| `runId` | string | да | Идентификатор запуска скоринга |

**Выходные параметры (Response 200):**

| Параметр | Тип | Пояснение |
|---|---|---|
| `[]` | array<object> | Массив вызовов сервисов |
| `[].service` | string | Наименование сервиса |
| `[].status` | string | Статус вызова (`success`/`failed`) |
| `[].durationMs` | integer | Время выполнения вызова в миллисекундах |

**Коды ответа:**  
- `200 OK`  
- `404 Not Found`

---

### PUT /scoring-runs/{runId}

**Описание:**  
Обновление статуса запуска скоринга (soft-delete).

**Входные параметры (Path):**

| Параметр | Тип | Обязательность | Пояснение |
|---|---|---:|---|
| `runId` | string | да | Идентификатор запуска скоринга |

**Входные параметры (Body):**

| Параметр | Тип | Обязательность | Пояснение | Пример |
|---|---|---:|---|---|
| `status` | string | да | Новый статус запуска | `"deleted"` |

**Выходные параметры (Response 200):**

| Параметр | Тип | Пояснение |
|---|---|---|
| `runId` | string | Идентификатор запуска |
| `status` | string | Установленный статус |

**Коды ответа:**  
- `200 OK`  
- `404 Not Found`

---

### DELETE /scoring-runs/{runId}

**Описание:**  
Удаление запуска скоринга (идемпотентная операция).

**Входные параметры (Path):**

| Параметр | Тип | Обязательность | Пояснение |
|---|---|---:|---|
| `runId` | string | да | Идентификатор запуска скоринга |

**Выходные параметры:**  
Тело ответа отсутствует.

**Коды ответа:**  
- `204 No Content` — удаление выполнено (в т.ч. повторно)  
- `404 Not Found` — (опционально, в зависимости от реализации) если принято возвращать ошибку на несуществующий ресурс

---

## Реализация API

API реализован с использованием Python и фреймворка FastAPI. Реализация обеспечивает обработку HTTP-запросов, валидацию входных данных и формирование ответов в формате JSON.

```python
from __future__ import annotations

from datetime import datetime, timezone
from typing import Dict, List, Literal, Optional
from uuid import uuid4

from fastapi import FastAPI, HTTPException, Query
from pydantic import BaseModel, Field, ConfigDict


app = FastAPI(
    title="Scoring Results API",
    version="1.0.0",
    description="API for scoring runs with triggered rules and called services."
)

# ----------------------------
# Models
# ----------------------------

TargetType = Literal["branch", "counterparty"]
RunStatus = Literal["queued", "running", "completed", "failed", "deleted"]
RiskLevel = Literal["low", "medium", "high"]
Severity = Literal["minor", "major", "critical"]
ServiceCallStatus = Literal["success", "failed", "skipped"]


def now_iso() -> str:
    return datetime.now(timezone.utc).replace(microsecond=0).isoformat().replace("+00:00", "Z")


class Target(BaseModel):
    type: TargetType
    id: str
    name: Optional[str] = None


class ScoringOptions(BaseModel):
    includeRules: bool = True
    includeServices: bool = True
    explain: bool = True


class CreateRunRequest(BaseModel):
    target: Target
    options: ScoringOptions = Field(default_factory=ScoringOptions)


class EvidenceItem(BaseModel):
    field: str
    value: object
    reason: str


class TriggeredRule(BaseModel):
    ruleId: str
    name: str
    category: str
    severity: Severity
    triggered: bool = True
    weight: float = Field(ge=0.0, le=1.0)
    evidence: List[EvidenceItem] = Field(default_factory=list)
    firedAt: str = Field(default_factory=now_iso)


class ServiceCall(BaseModel):
    serviceCallId: str
    service: str
    endpoint: str
    status: ServiceCallStatus
    startedAt: str
    durationMs: int = Field(ge=0)
    requestRef: str
    responseSummary: dict = Field(default_factory=dict)
    error: Optional[dict] = None


class WarningItem(BaseModel):
    code: str
    message: str


class RunSummary(BaseModel):
    triggeredRulesCount: int
    servicesCalledCount: int
    warningsCount: int


class ScoringRun(BaseModel):
    model_config = ConfigDict(extra="forbid")

    runId: str
    target: Target
    status: RunStatus
    startedAt: str
    finishedAt: Optional[str] = None
    durationMs: Optional[int] = None
    score: Optional[float] = Field(default=None, ge=0.0, le=1.0)
    riskLevel: Optional[RiskLevel] = None
    rulesVersion: str = "ruleset-1.0.0"
    modelVersion: str = "score-model-1.0.0"
    traceId: str
    summary: Optional[RunSummary] = None


class RunResult(BaseModel):
    run: ScoringRun
    rulesTriggered: List[TriggeredRule] = Field(default_factory=list)
    servicesCalled: List[ServiceCall] = Field(default_factory=list)
    warnings: List[WarningItem] = Field(default_factory=list)


class CreateRunResponse(BaseModel):
    runId: str
    status: RunStatus
    startedAt: str
    links: dict


class ListRunsResponse(BaseModel):
    items: List[ScoringRun]
    paging: dict


class UpdateRunRequest(BaseModel):
    status: Optional[RunStatus] = None
    finishedAt: Optional[str] = None
    score: Optional[float] = Field(default=None, ge=0.0, le=1.0)
    riskLevel: Optional[RiskLevel] = None


class ApiError(BaseModel):
    error: dict


# ----------------------------
# In-memory storage
# ----------------------------

RUNS: Dict[str, ScoringRun] = {}
RESULTS: Dict[str, RunResult] = {}


def api_error(code: str, message: str, trace_id: Optional[str] = None):
    return {
        "error": {
            "code": code,
            "message": message,
            "traceId": trace_id or str(uuid4())
        }
    }


def generate_mock_result(run: ScoringRun) -> RunResult:
    """
    Мок-логика: генерируем итог скоринга + сработавшие правила + вызванные сервисы.
    Для лабы этого более чем достаточно (и красиво выглядит в Postman).
    """
    started = run.startedAt

    rules = [
        TriggeredRule(
            ruleId="R-AML-010",
            name="High-risk jurisdiction",
            category="aml",
            severity="critical",
            weight=0.18,
            evidence=[EvidenceItem(field="country", value="X", reason="Listed as high-risk")],
            firedAt=started,
        ),
        TriggeredRule(
            ruleId="R-FRAUD-002",
            name="Abnormal transaction pattern",
            category="fraud",
            severity="major",
            weight=0.12,
            evidence=[EvidenceItem(field="txVelocity", value=12, reason="Above threshold 8")],
            firedAt=started,
        ),
        TriggeredRule(
            ruleId="R-KYC-003",
            name="KYC data incomplete",
            category="kyc",
            severity="minor",
            weight=0.05,
            evidence=[EvidenceItem(field="kycProfile", value="partial", reason="Missing documents")],
            firedAt=started,
        ),
    ]

    service_calls = [
        ServiceCall(
            serviceCallId=f"sc_{uuid4().hex[:8]}",
            service="SanctionsScreening",
            endpoint="/screen",
            status="success",
            startedAt=started,
            durationMs=420,
            requestRef=f"req_{uuid4().hex[:10]}",
            responseSummary={"matches": 1, "matchLevel": "possible"},
            error=None,
        ),
        ServiceCall(
            serviceCallId=f"sc_{uuid4().hex[:8]}",
            service="PEPCheck",
            endpoint="/pep",
            status="success",
            startedAt=started,
            durationMs=210,
            requestRef=f"req_{uuid4().hex[:10]}",
            responseSummary={"pepFound": False},
            error=None,
        ),
    ]

    warnings = [WarningItem(code="DATA_INCOMPLETE", message="KYC profile is partially missing; fallback weights applied.")]

    # Простая формула score для демонстрации
    total_weight = sum(r.weight for r in rules if r.triggered)
    score = min(1.0, 0.4 + total_weight)  # базовый 0.4 + веса
    risk: RiskLevel = "high" if score >= 0.7 else ("medium" if score >= 0.5 else "low")

    finished_at = now_iso()
    # durationMs - фиктивно
    duration_ms = 1240

    run_completed = run.model_copy(update={
        "status": "completed",
        "finishedAt": finished_at,
        "durationMs": duration_ms,
        "score": round(score, 2),
        "riskLevel": risk,
        "summary": RunSummary(
            triggeredRulesCount=len(rules),
            servicesCalledCount=len(service_calls),
            warningsCount=len(warnings),
        )
    })

    return RunResult(
        run=run_completed,
        rulesTriggered=rules,
        servicesCalled=service_calls,
        warnings=warnings,
    )


def get_run_or_404(run_id: str) -> ScoringRun:
    run = RUNS.get(run_id)
    if not run or run.status == "deleted":
        raise HTTPException(status_code=404, detail=api_error("RUN_NOT_FOUND", f"Scoring run '{run_id}' not found")["error"])
    return run


# ----------------------------
# Endpoints
# ----------------------------

@app.post("/scoring-runs", response_model=CreateRunResponse, status_code=201)
def create_scoring_run(req: CreateRunRequest):
    run_id = f"sr_{datetime.now().strftime('%Y%m%d')}_{uuid4().hex[:6]}"
    trace_id = str(uuid4())

    run = ScoringRun(
        runId=run_id,
        target=req.target,
        status="running",
        startedAt=now_iso(),
        traceId=trace_id,
        rulesVersion="ruleset-1.4.2",
        modelVersion="score-model-2.1.0",
        summary=None,
    )

    # Для лабы делаем "как будто быстро посчитали" и сразу completed
    result = generate_mock_result(run)

    # Сохраняем
    RUNS[run_id] = result.run
    RESULTS[run_id] = result

    return CreateRunResponse(
        runId=run_id,
        status=result.run.status,
        startedAt=result.run.startedAt,
        links={
            "self": f"/scoring-runs/{run_id}",
            "result": f"/scoring-runs/{run_id}/result",
            "rules": f"/scoring-runs/{run_id}/rules",
            "serviceCalls": f"/scoring-runs/{run_id}/service-calls",
        }
    )


@app.get("/scoring-runs", response_model=ListRunsResponse)
def list_scoring_runs(
    targetType: Optional[TargetType] = None,
    targetId: Optional[str] = None,
    status: Optional[RunStatus] = None,
    limit: int = Query(default=20, ge=1, le=100),
    offset: int = Query(default=0, ge=0),
):
    items = list(RUNS.values())

    if targetType:
        items = [r for r in items if r.target.type == targetType]
    if targetId:
        items = [r for r in items if r.target.id == targetId]
    if status:
        items = [r for r in items if r.status == status]

    total = len(items)
    sliced = items[offset: offset + limit]

    return ListRunsResponse(
        items=sliced,
        paging={"limit": limit, "offset": offset, "total": total}
    )


@app.get("/scoring-runs/{runId}", response_model=ScoringRun)
def get_scoring_run(runId: str):
    return get_run_or_404(runId)


@app.get("/scoring-runs/{runId}/result", response_model=RunResult)
def get_scoring_result(
    runId: str,
    includeRules: bool = True,
    includeServices: bool = True,
):
    _ = get_run_or_404(runId)
    result = RESULTS.get(runId)
    if not result:
        raise HTTPException(status_code=404, detail=api_error("RESULT_NOT_FOUND", f"Result for run '{runId}' not found")["error"])

    # по флагам можно "обрезать"
    filtered = RunResult(
        run=result.run,
        rulesTriggered=result.rulesTriggered if includeRules else [],
        servicesCalled=result.servicesCalled if includeServices else [],
        warnings=result.warnings,
    )
    return filtered


@app.get("/scoring-runs/{runId}/rules", response_model=List[TriggeredRule])
def get_triggered_rules(runId: str):
    _ = get_run_or_404(runId)
    result = RESULTS.get(runId)
    if not result:
        raise HTTPException(status_code=404, detail=api_error("RESULT_NOT_FOUND", f"Result for run '{runId}' not found")["error"])
    return result.rulesTriggered


@app.get("/scoring-runs/{runId}/service-calls", response_model=List[ServiceCall])
def get_service_calls(runId: str):
    _ = get_run_or_404(runId)
    result = RESULTS.get(runId)
    if not result:
        raise HTTPException(status_code=404, detail=api_error("RESULT_NOT_FOUND", f"Result for run '{runId}' not found")["error"])
    return result.servicesCalled


@app.put("/scoring-runs/{runId}", response_model=ScoringRun)
def update_scoring_run(runId: str, req: UpdateRunRequest):
    run = get_run_or_404(runId)

    if run.status in ("deleted",):
        raise HTTPException(status_code=409, detail=api_error("RUN_DELETED", "Cannot update a deleted run", run.traceId)["error"])

    # Пример бизнес-правила: completed можно обновлять только статусом deleted (иначе 409)
    if run.status == "completed" and req.status and req.status not in ("deleted",):
        raise HTTPException(status_code=409, detail=api_error("RUN_IMMUTABLE", "Completed run cannot be modified (except deletion)", run.traceId)["error"])

    updated = run.model_copy(update={
        "status": req.status or run.status,
        "finishedAt": req.finishedAt or run.finishedAt,
        "score": req.score if req.score is not None else run.score,
        "riskLevel": req.riskLevel or run.riskLevel,
    })

    RUNS[runId] = updated

    # если есть result — тоже обновим run внутри result
    if runId in RESULTS:
        RESULTS[runId] = RESULTS[runId].model_copy(update={"run": updated})

    return updated


@app.delete("/scoring-runs/{runId}", status_code=204)
def delete_scoring_run(runId: str):
    run = RUNS.get(runId)
    if not run or run.status == "deleted":
        # DELETE идемпотентный: можно вернуть 204 даже если уже удалено
        return

    deleted = run.model_copy(update={"status": "deleted"})
    RUNS[runId] = deleted
    if runId in RESULTS:
        RESULTS[runId] = RESULTS[runId].model_copy(update={"run": deleted})
    return

```
---
## Тестирование API в Postman

Тестирование разработанного REST API выполнялось с использованием инструмента **Postman**. Для каждого реализованного endpoint были выполнены как минимум два тестовых запроса, включая корректные (позитивные) сценарии и сценарии обработки ошибок. В процессе тестирования анализировались передаваемые параметры запроса, HTTP-заголовки, тело запроса и ответа, а также возвращаемые коды состояния.

### POST /scoring-runs
 
POST-запрос с корректным телом запроса (тип объекта branch, идентификатор и наименование) завершился с кодом **201 Created**. В ответе был возвращён уникальный идентификатор запуска `runId`, статус выполнения и ссылки на связанные ресурсы.
Автотест
```
// Basic response checks
pm.test("Status code is 201", function () {
    pm.response.to.have.status(201);
});

pm.test("Response is valid JSON", function () {
    pm.response.to.be.json;
});

let json;
try {
    json = pm.response.json();
} catch (e) {
    // If parsing fails, subsequent tests should fail as well
    json = null;
}

// Top-level shape
pm.test("Response has required top-level fields", function () {
    pm.expect(json).to.be.an("object");
    pm.expect(json).to.have.property("runId");
    pm.expect(json).to.have.property("status");
    pm.expect(json).to.have.property("startedAt");
    pm.expect(json).to.have.property("links");
});

// runId checks
pm.test("runId is a non-empty string and starts with 'sr_'", function () {
    pm.expect(json.runId).to.be.a("string").and.to.not.be.empty;
    pm.expect(json.runId).to.match(/^sr_/);
});

// status checks (allowing common states)
pm.test("status is one of expected values", function () {
    const allowed = ["pending", "running", "completed", "failed", "cancelled"];
    pm.expect(json.status).to.be.a("string");
    pm.expect(allowed).to.include(json.status);
});

// startedAt is ISO-8601
pm.test("startedAt is a valid ISO-8601 timestamp", function () {
    pm.expect(json.startedAt).to.be.a("string");
    const t = Date.parse(json.startedAt);
    pm.expect(Number.isFinite(t)).to.be.true;
});

// links object
pm.test("links contains self and result URLs", function () {
    pm.expect(json.links).to.be.an("object");
    pm.expect(json.links).to.have.property("self").that.is.a("string");
    pm.expect(json.links).to.have.property("result").that.is.a("string");
});

// Response time
pm.test("Response time is less than 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});

// Save useful values to environment variables
if (json && json.runId) {
    pm.environment.set("runId", json.runId);
}
if (json && json.links && json.links.result) {
    pm.environment.set("resultLink", json.links.result);
}
```
<img width="1680" height="1050" alt="Снимок экрана 2026-02-05 в 23 11 47" src="https://github.com/user-attachments/assets/03a32fa6-0e28-405e-936a-ac20579c1a04" />


### GET /scoring-runs

Тест 1 — получение списка запусков.  
GET-запрос без параметров вернул код **200 OK** и список запусков скоринга в массиве `items`.
Автотесты
```
// 1) Assert status code is 200
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 2) Assert response is valid JSON and contains 'items' array and 'paging' object
pm.test("Response is valid JSON with 'items' array and 'paging' object", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an("object");
    pm.expect(jsonData).to.have.property("items").that.is.an("array");
    pm.expect(jsonData).to.have.property("paging").that.is.an("object");
});

// 3) Validate each item in items array
pm.test("Each item has valid structure and field types", function () {
    const jsonData = pm.response.json();
    const isoTimestampRegex = /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d+)?Z$/;
    
    jsonData.items.forEach((item, index) => {
        // runId is string
        pm.expect(item.runId, `items[${index}].runId`).to.be.a("string");
        
        // target.type and target.id are strings
        pm.expect(item.target, `items[${index}].target`).to.be.an("object");
        pm.expect(item.target.type, `items[${index}].target.type`).to.be.a("string");
        pm.expect(item.target.id, `items[${index}].target.id`).to.be.a("string");
        
        // status is string
        pm.expect(item.status, `items[${index}].status`).to.be.a("string");
        
        // startedAt and finishedAt are valid ISO timestamps
        pm.expect(item.startedAt, `items[${index}].startedAt`).to.be.a("string").and.match(isoTimestampRegex);
        pm.expect(item.finishedAt, `items[${index}].finishedAt`).to.be.a("string").and.match(isoTimestampRegex);
        
        // durationMs is number
        pm.expect(item.durationMs, `items[${index}].durationMs`).to.be.a("number");
        
        // score is number between 0 and 1
        pm.expect(item.score, `items[${index}].score`).to.be.a("number").and.to.be.at.least(0).and.to.be.at.most(1);
        
        // riskLevel is string
        pm.expect(item.riskLevel, `items[${index}].riskLevel`).to.be.a("string");
        
        // rulesVersion, modelVersion, traceId are strings
        pm.expect(item.rulesVersion, `items[${index}].rulesVersion`).to.be.a("string");
        pm.expect(item.modelVersion, `items[${index}].modelVersion`).to.be.a("string");
        pm.expect(item.traceId, `items[${index}].traceId`).to.be.a("string");
        
        // summary contains triggeredRulesCount, servicesCalledCount, warningsCount as numbers
        pm.expect(item.summary, `items[${index}].summary`).to.be.an("object");
        pm.expect(item.summary.triggeredRulesCount, `items[${index}].summary.triggeredRulesCount`).to.be.a("number");
        pm.expect(item.summary.servicesCalledCount, `items[${index}].summary.servicesCalledCount`).to.be.a("number");
        pm.expect(item.summary.warningsCount, `items[${index}].summary.warningsCount`).to.be.a("number");
    });
});

// 4) Assert paging contains limit/offset/total as numbers and total equals items.length
pm.test("Paging has valid structure and total equals items count", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.paging.limit, "paging.limit").to.be.a("number");
    pm.expect(jsonData.paging.offset, "paging.offset").to.be.a("number");
    pm.expect(jsonData.paging.total, "paging.total").to.be.a("number");
    pm.expect(jsonData.paging.total, "paging.total equals items.length").to.equal(jsonData.items.length);
});

// 5) Set collection variable lastRunIds as JSON string array of all runIds
const responseData = pm.response.json();
const runIds = responseData.items.map(item => item.runId);
pm.collectionVariables.set("lastRunIds", JSON.stringify(runIds));

// 6) Check at least one item has riskLevel 'high'
pm.test("At least one item has riskLevel 'high'", function () {
    const jsonData = pm.response.json();
    const hasHighRisk = jsonData.items.some(item => item.riskLevel === "high");
    pm.expect(hasHighRisk, "At least one item with riskLevel 'high'").to.be.true;
});
```
<img width="1680" height="1050" alt="Снимок экрана 2026-02-05 в 23 13 30" src="https://github.com/user-attachments/assets/5a2c5371-4d35-41b9-bc38-729ec91d9aad" />

### GET /scoring-runs/{runId}

Тест 1 — получение существующего запуска.  
GET-запрос с корректным `runId` завершился с кодом **200 OK** и вернул метаданные запуска.
```
// Basic status and JSON parsing
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

let json;
pm.test("Response is valid JSON", function () {
    try {
        json = pm.response.json();
        pm.expect(json).to.be.an("object");
    } catch (e) {
        pm.expect.fail("Response is not valid JSON: " + e.message);
    }
});

// Helper
function isISODateString(s) {
    // basic ISO 8601 check (YYYY-MM-DDTHH:MM:SSZ or with offset)
    return typeof s === "string" && /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(?:\.\d+)?(?:Z|[+-]\d{2}:\d{2})$/.test(s);
}

pm.test("Top-level fields exist and types are correct", function () {
    pm.expect(json).to.have.property("runId").that.is.a("string");
    pm.expect(json).to.have.property("target").that.is.an("object");
    pm.expect(json.target).to.have.property("type").that.is.a("string");
    pm.expect(json.target).to.have.property("id").that.is.a("string");
    pm.expect(json.target).to.have.property("name").that.is.a("string");
    pm.expect(json).to.have.property("status").that.is.a("string");
    pm.expect(json).to.have.property("startedAt").that.is.a("string");
    pm.expect(json).to.have.property("finishedAt").that.is.a("string");
    pm.expect(json).to.have.property("durationMs").that.is.a("number");
    pm.expect(json).to.have.property("score").that.is.a("number");
    pm.expect(json).to.have.property("riskLevel").that.is.a("string");
    pm.expect(json).to.have.property("rulesVersion").that.is.a("string");
    pm.expect(json).to.have.property("modelVersion").that.is.a("string");
    pm.expect(json).to.have.property("traceId").that.is.a("string");
    pm.expect(json).to.have.property("summary").that.is.an("object");
    pm.expect(json.summary).to.have.property("triggeredRulesCount").that.is.a("number");
    pm.expect(json.summary).to.have.property("servicesCalledCount").that.is.a("number");
    pm.expect(json.summary).to.have.property("warningsCount").that.is.a("number");
});

pm.test("Timestamps are valid ISO strings and startedAt <= finishedAt", function () {
    pm.expect(isISODateString(json.startedAt), "startedAt is not valid ISO timestamp").to.be.true;
    pm.expect(isISODateString(json.finishedAt), "finishedAt is not valid ISO timestamp").to.be.true;

    const started = Date.parse(json.startedAt);
    const finished = Date.parse(json.finishedAt);
    pm.expect(isNaN(started), "startedAt could not be parsed").to.be.false;
    pm.expect(isNaN(finished), "finishedAt could not be parsed").to.be.false;

    pm.expect(started <= finished, "startedAt is after finishedAt").to.be.true;
});

pm.test("Numeric fields in expected ranges", function () {
    pm.expect(json.durationMs, "durationMs should be non-negative").to.be.at.least(0);
    pm.expect(json.score, "score should be between 0 and 1").to.be.within(0, 1);
    pm.expect(json.summary.triggeredRulesCount, "triggeredRulesCount should be >= 0").to.be.at.least(0);
    pm.expect(json.summary.servicesCalledCount, "servicesCalledCount should be >= 0").to.be.at.least(0);
    pm.expect(json.summary.warningsCount, "warningsCount should be >= 0").to.be.at.least(0);
});

// Save useful value for later runs
if (json && json.runId) {
    pm.collectionVariables.set("lastSuccessfulRunId", json.runId);
    pm.test("Saved runId to collection variable lastSuccessfulRunId", function () {
        pm.expect(pm.collectionVariables.get("lastSuccessfulRunId")).to.eql(json.runId);
    });
}
```
<img width="1680" height="1050" alt="Снимок экрана 2026-02-05 в 23 14 38" src="https://github.com/user-attachments/assets/87825e32-dab3-491a-ad3f-0baa5d678b7b" />

### GET /scoring-runs/{runId}/result

Тест 1 — получение полного результата скоринга.  
GET-запрос с существующим `runId` вернул код **200 OK** и полный результат скоринга, включая правила и вызванные сервисы.
```
// 1) Assert status is 200
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 2) Assert response is valid JSON
pm.test("Response is valid JSON", function () {
    pm.response.to.be.json;
});

// 3) Assert run.runId matches pattern or at least non-empty string starting with 'sr_'
pm.test("run.runId matches expected pattern", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.run).to.be.an('object');
    pm.expect(jsonData.run.runId).to.be.a('string').and.not.empty;
    pm.expect(jsonData.run.runId.startsWith('sr_')).to.be.true;
    // Check full pattern: sr_YYYYMMDD_6hexchars
    const runIdPattern = /^sr_\d{8}_[a-f0-9]{6}$/;
    pm.expect(jsonData.run.runId).to.match(runIdPattern);
});

// 4) Assert score is a number between 0 and 1 and riskLevel is one of ['low','medium','high']
pm.test("score is a number between 0 and 1", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.run.score).to.be.a('number');
    pm.expect(jsonData.run.score).to.be.at.least(0);
    pm.expect(jsonData.run.score).to.be.at.most(1);
});

pm.test("riskLevel is one of ['low','medium','high']", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.run.riskLevel).to.be.oneOf(['low', 'medium', 'high']);
});

// 5) Assert rulesTriggered is an array and each item has ruleId, name, severity, triggered (boolean)
pm.test("rulesTriggered is an array with valid structure", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.rulesTriggered).to.be.an('array');
    jsonData.rulesTriggered.forEach(function (rule, index) {
        pm.expect(rule.ruleId, `rulesTriggered[${index}].ruleId`).to.be.a('string').and.not.empty;
        pm.expect(rule.name, `rulesTriggered[${index}].name`).to.be.a('string').and.not.empty;
        pm.expect(rule.severity, `rulesTriggered[${index}].severity`).to.be.a('string').and.not.empty;
        pm.expect(rule.triggered, `rulesTriggered[${index}].triggered`).to.be.a('boolean');
    });
});

// 6) Assert servicesCalled is an array and each service has serviceCallId, service, status
pm.test("servicesCalled is an array with valid structure", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.servicesCalled).to.be.an('array');
    jsonData.servicesCalled.forEach(function (svc, index) {
        pm.expect(svc.serviceCallId, `servicesCalled[${index}].serviceCallId`).to.be.a('string').and.not.empty;
        pm.expect(svc.service, `servicesCalled[${index}].service`).to.be.a('string').and.not.empty;
        pm.expect(svc.status, `servicesCalled[${index}].status`).to.be.a('string').and.not.empty;
    });
});

// 7) Extract traceId and set as environment variable 'lastTraceId'
pm.test("Extract traceId and set environment variable", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.run.traceId).to.be.a('string').and.not.empty;
    pm.environment.set("lastTraceId", jsonData.run.traceId);
});

// 8) Assert response time is under 2000ms
pm.test("Response time is under 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```
<img width="1680" height="1050" alt="Снимок экрана 2026-02-05 в 23 15 28" src="https://github.com/user-attachments/assets/6d5a999d-ea6d-4508-bf01-ff9ae0cad8c4" />


### GET /scoring-runs/{runId}/rules

Тест 1 — получение списка сработавших правил.  
GET-запрос с корректным `runId` вернул код **200 OK** и массив правил с указанием уровня критичности и веса.
```
// 1) Assert status is 200
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 2) Assert response is JSON and is an array
pm.test("Response is JSON and is an array", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an('array');
});

// 3) Assert array length is >= 1
pm.test("Array contains at least one item", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.length).to.be.at.least(1);
});


// 4b) Validate that triggered is true for all items
pm.test("All rules have triggered set to true", function () {
    const jsonData = pm.response.json();
    jsonData.forEach((rule, index) => {
        pm.expect(rule.triggered, `Rule at index ${index} should have triggered=true`).to.be.true;
    });
});

// 5) Assert at least one rule has severity 'critical' or 'major'
pm.test("At least one rule has severity 'critical' or 'major'", function () {
    const jsonData = pm.response.json();
    const hasCriticalOrMajor = jsonData.some(rule => 
        rule.severity === 'critical' || rule.severity === 'major'
    );
    pm.expect(hasCriticalOrMajor, "Expected at least one rule with severity 'critical' or 'major'").to.be.true;
});

// 6) Calculate total weight and set as environment variable
pm.test("Calculate and store total weight", function () {
    const jsonData = pm.response.json();
    const totalWeight = jsonData.reduce((sum, rule) => sum + rule.weight, 0);
    pm.environment.set('rules_total_weight', totalWeight);
    console.log("Total weight calculated: " + totalWeight);
});

// 7) Check each evidence item has field and reason properties
pm.test("Each evidence item has 'field' and 'reason' properties", function () {
    const jsonData = pm.response.json();
    jsonData.forEach((rule, ruleIndex) => {
        rule.evidence.forEach((ev, evIndex) => {
            pm.expect(ev, `Evidence at rule[${ruleIndex}].evidence[${evIndex}]`).to.have.property('field');
            pm.expect(ev, `Evidence at rule[${ruleIndex}].evidence[${evIndex}]`).to.have.property('reason');
        });
    });
});
```
<img width="1680" height="1050" alt="Снимок экрана 2026-02-05 в 23 17 22" src="https://github.com/user-attachments/assets/2c2253f3-f16b-4f38-8c94-fc20bd5b3c27" />

### GET /scoring-runs/{runId}/service-calls

Тест 1 — получение списка вызванных сервисов.  
GET-запрос с корректным `runId` вернул код **200 OK** и список сервисов с информацией о статусе и времени выполнения.
```
// 1) Assert HTTP status is 200
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 2) Assert response is valid JSON and an array
pm.test("Response is valid JSON and an array", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an('array');
});

// 3) Assert each array item has required keys
pm.test("Each item has required keys", function () {
    const jsonData = pm.response.json();
    const requiredKeys = ['serviceCallId', 'service', 'endpoint', 'status', 'startedAt', 'durationMs', 'requestRef', 'responseSummary', 'error'];
    
    jsonData.forEach((item, index) => {
        requiredKeys.forEach(key => {
            pm.expect(item).to.have.property(key);
        });
    });
});

// 4) Assert durationMs is a non-negative number for each item
pm.test("durationMs is a non-negative number for each item", function () {
    const jsonData = pm.response.json();
    
    jsonData.forEach((item, index) => {
        pm.expect(item.durationMs).to.be.a('number');
        pm.expect(item.durationMs).to.be.at.least(0);
    });
});

// 5) Assert startedAt is a valid ISO-8601 timestamp for each item
pm.test("startedAt is a valid ISO-8601 timestamp for each item", function () {
    const jsonData = pm.response.json();
    const iso8601Regex = /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d+)?(Z|[+-]\d{2}:\d{2})?$/;
    
    jsonData.forEach((item, index) => {
        pm.expect(item.startedAt).to.match(iso8601Regex);
        pm.expect(new Date(item.startedAt).toString()).to.not.equal('Invalid Date');
    });
});

// 6) Assert responseSummary is an object and contains at least one key
pm.test("responseSummary is an object with at least one key", function () {
    const jsonData = pm.response.json();
    
    jsonData.forEach((item, index) => {
        pm.expect(item.responseSummary).to.be.an('object');
        pm.expect(Object.keys(item.responseSummary).length).to.be.at.least(1);
    });
});

// 7) If status is 'success' then error is null
pm.test("If status is 'success', error is null", function () {
    const jsonData = pm.response.json();
    
    jsonData.forEach((item, index) => {
        if (item.status === 'success') {
            pm.expect(item.error).to.be.null;
        }
    });
});

// 8) Check there is at least one service call where service equals 'SanctionsScreening'
pm.test("At least one service call has service 'SanctionsScreening'", function () {
    const jsonData = pm.response.json();
    const hasSanctionsScreening = jsonData.some(item => item.service === 'SanctionsScreening');
    pm.expect(hasSanctionsScreening).to.be.true;
});

// 9) Set environment variables
const jsonData = pm.response.json();
pm.environment.set('lastServiceCallCount', jsonData.length);
pm.environment.set('lastServiceCallIds', JSON.stringify(jsonData.map(item => item.serviceCallId)));
```
<img width="1680" height="1050" alt="Снимок экрана 2026-02-05 в 23 18 33" src="https://github.com/user-attachments/assets/2049e486-2c95-43bb-83cc-f81f254534b4" />

### PUT /scoring-runs/{runId}

Тест 1 — изменение статуса запуска (soft-delete).  
PUT-запрос с телом `{ "status": "deleted" }` завершился с кодом **200 OK**, статус запуска был успешно обновлён.
```
// Test 1: Assert status code is 200
pm.test("Status code is 200", function () {
    pm.expect(pm.response.code).to.equal(200);
});

// Test 2: Assert runId matches the request path segment
pm.test("Response runId matches request path segment", function () {
    const jsonData = pm.response.json();
    const urlPath = pm.request.url.path;
    // Extract the runId from the URL path (e.g., "sr_20260204_0c5c5d")
    const expectedRunId = urlPath.find(segment => segment.startsWith("sr_"));
    pm.expect(jsonData.runId).to.equal(expectedRunId);
});

// Test 3: Assert status field equals 'deleted'
pm.test("Status field equals 'deleted'", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.status).to.equal("deleted");
});

// Test 4: Assert startedAt and finishedAt are valid ISO8601 timestamps and finishedAt >= startedAt
pm.test("startedAt and finishedAt are valid ISO8601 timestamps and finishedAt >= startedAt", function () {
    const jsonData = pm.response.json();
    
    // Validate ISO8601 format using regex
    const iso8601Regex = /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d+)?(Z|[+-]\d{2}:\d{2})?$/;
    
    pm.expect(jsonData.startedAt).to.match(iso8601Regex, "startedAt should be valid ISO8601");
    pm.expect(jsonData.finishedAt).to.match(iso8601Regex, "finishedAt should be valid ISO8601");
    
    // Parse and compare timestamps
    const startedAt = new Date(jsonData.startedAt);
    const finishedAt = new Date(jsonData.finishedAt);
    
    pm.expect(startedAt.toString()).to.not.equal("Invalid Date", "startedAt should be a valid date");
    pm.expect(finishedAt.toString()).to.not.equal("Invalid Date", "finishedAt should be a valid date");
    pm.expect(finishedAt.getTime()).to.be.at.least(startedAt.getTime(), "finishedAt should be >= startedAt");
});

// Test 5: Assert score is a number between 0 and 1 inclusive
pm.test("Score is a number between 0 and 1 inclusive", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.score).to.be.a("number");
    pm.expect(jsonData.score).to.be.at.least(0);
    pm.expect(jsonData.score).to.be.at.most(1);
});

// Test 6: Assert summary.triggeredRulesCount is a non-negative integer
pm.test("summary.triggeredRulesCount is a non-negative integer", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.summary).to.be.an("object");
    pm.expect(jsonData.summary.triggeredRulesCount).to.be.a("number");
    pm.expect(Number.isInteger(jsonData.summary.triggeredRulesCount)).to.be.true;
    pm.expect(jsonData.summary.triggeredRulesCount).to.be.at.least(0);
});

// Test 7: Set collection variable lastDeletedRunId to the runId
pm.test("Set collection variable lastDeletedRunId", function () {
    const jsonData = pm.response.json();
    pm.collectionVariables.set("lastDeletedRunId", jsonData.runId);
    pm.expect(pm.collectionVariables.get("lastDeletedRunId")).to.equal(jsonData.runId);
});
```
<img width="1680" height="1050" alt="Снимок экрана 2026-02-05 в 23 19 51" src="https://github.com/user-attachments/assets/28fb612a-60bb-49f8-a727-2ff596ea94ef" />

### DELETE /scoring-runs/{runId}

Тест 1 — удаление существующего запуска.  
DELETE-запрос с корректным `runId` вернул код **204 No Content**.
```
// Test 1: Response status is 200 or 204 (successful delete) or 404 (not found)
pm.test("Status code is 200, 204, or 404", function () {
    pm.expect(pm.response.code).to.be.oneOf([200, 204, 404]);
});

// Test 2: Response time is under 1000ms
pm.test("Response time is under 1000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});

// Test 3: If response has JSON body with successful delete, validate properties
if (pm.response.code === 200 || pm.response.code === 204) {
    // Try to parse JSON body gracefully
    let jsonData = null;
    try {
        jsonData = pm.response.json();
    } catch (e) {
        // Response body is not JSON or is empty - this is acceptable for 204
    }

    if (jsonData) {
        pm.test("Response contains 'deleted': true", function () {
            pm.expect(jsonData).to.have.property("deleted", true);
        });

        pm.test("Response contains correct 'runId'", function () {
            pm.expect(jsonData).to.have.property("runId", "sr_20260204_1adbf5");
        });
    }
}

// Test 4: If 404 returned, assert message includes "not found" (case-insensitive)
if (pm.response.code === 404) {
    pm.test("404 response message includes 'not found'", function () {
        let responseText = "";
        try {
            const jsonData = pm.response.json();
            responseText = JSON.stringify(jsonData).toLowerCase();
        } catch (e) {
            responseText = pm.response.text().toLowerCase();
        }
        pm.expect(responseText).to.include("not found");
    });
}
```
<img width="1680" height="1050" alt="Снимок экрана 2026-02-05 в 23 20 28" src="https://github.com/user-attachments/assets/1262e0c7-e86c-4cfd-8f30-c95de7c20b9b" />

