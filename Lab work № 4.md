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
- Content-Type: application/json  

**Входные параметры (Body):**  
- target.type (string) — тип объекта (branch, counterparty)  
- target.id (string) — идентификатор объекта  
- target.name (string, optional) — наименование объекта  

**Пример запроса:**
```json
{
  "target": {
    "type": "branch",
    "id": "BR-001",
    "name": "Amsterdam Branch"
  }
}
```

**Коды ответа:**  
- 201 Created — скоринг успешно создан  
- 422 Unprocessable Entity — некорректные входные данные  

**Пример ответа:**
```json
{
  "runId": "sr_20260204_ab12cd",
  "status": "completed",
  "startedAt": "2026-02-04T10:15:12Z",
  "links": {
    "self": "/scoring-runs/sr_20260204_ab12cd",
    "result": "/scoring-runs/sr_20260204_ab12cd/result"
  }
}
```

**cURL:**
```
curl -X POST -H "Content-Type: application/json" \
-d '{"target":{"type":"branch","id":"BR-001","name":"Amsterdam Branch"}}' \
http://localhost:8000/scoring-runs
```

---

### GET /scoring-runs

**Описание:**  
Получение списка запусков скоринга с возможностью фильтрации.

**Входные параметры (Query):**  
- targetType (string, optional)  
- targetId (string, optional)  
- status (string, optional)  

**Пример запроса:**
```
/scoring-runs?targetType=branch&targetId=BR-001
```

**Пример ответа:**
```json
{
  "items": [
    {
      "runId": "sr_20260204_ab12cd",
      "status": "completed",
      "score": 0.73,
      "riskLevel": "high"
    }
  ],
  "paging": {
    "limit": 20,
    "offset": 0,
    "total": 1
  }
}
```

---

### GET /scoring-runs/{runId}

**Описание:**  
Получение метаданных конкретного запуска скоринга.

**Пример запроса:**
```
/scoring-runs/sr_20260204_ab12cd
```

**Пример ответа:**
```json
{
  "runId": "sr_20260204_ab12cd",
  "status": "completed",
  "score": 0.73,
  "riskLevel": "high"
}
```

---

### GET /scoring-runs/{runId}/result

**Описание:**  
Получение полного результата скоринга с детализацией.

**Пример ответа:**
```json
{
  "run": {
    "runId": "sr_20260204_ab12cd",
    "score": 0.73,
    "riskLevel": "high"
  },
  "rulesTriggered": [
    {
      "ruleId": "R-AML-010",
      "severity": "critical",
      "weight": 0.18
    }
  ],
  "servicesCalled": [
    {
      "service": "SanctionsScreening",
      "status": "success"
    }
  ]
}
```

---

### GET /scoring-runs/{runId}/rules

**Описание:**  
Получение списка сработавших правил скоринга.

**Пример ответа:**
```json
[
  {
    "ruleId": "R-AML-010",
    "severity": "critical",
    "weight": 0.18
  }
]
```

---

### GET /scoring-runs/{runId}/service-calls

**Описание:**  
Получение информации о вызванных сервисах.

**Пример ответа:**
```json
[
  {
    "service": "SanctionsScreening",
    "status": "success",
    "durationMs": 420
  }
]
```

---

### PUT /scoring-runs/{runId}

**Описание:**  
Обновление статуса запуска скоринга (soft-delete).

**Пример запроса:**
```json
{
  "status": "deleted"
}
```

---

### DELETE /scoring-runs/{runId}

**Описание:**  
Удаление запуска скоринга (идемпотентная операция).

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

Тест 1 — успешный запуск скоринга.  
POST-запрос с корректным телом запроса (тип объекта branch, идентификатор и наименование) завершился с кодом **201 Created**. В ответе был возвращён уникальный идентификатор запуска `runId`, статус выполнения и ссылки на связанные ресурсы.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 48 18" src="https://github.com/user-attachments/assets/8b1ee8e4-eb49-48af-bf73-9728dd46c59d" />


Тест 2 — запуск скоринга для другого типа объекта.  
POST-запрос с типом объекта counterparty также завершился успешно с кодом **201 Created**, что подтверждает универсальность метода.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 49 52" src="https://github.com/user-attachments/assets/d64c5a29-0741-420b-a199-54b0fa0f50ef" />

### GET /scoring-runs

Тест 1 — получение списка запусков.  
GET-запрос без параметров вернул код **200 OK** и список запусков скоринга в массиве `items`.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 51 56" src="https://github.com/user-attachments/assets/92371e37-31e6-4abd-9696-799a9b704e65" />

Тест 2 — фильтрация по типу и идентификатору.  
GET-запрос с параметрами `targetType=branch` и `targetId=BR-001` вернул только соответствующие записи, код ответа — **200 OK**.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 53 05" src="https://github.com/user-attachments/assets/335ea0ea-d0a4-4367-9858-f69c74d4f88c" />

### GET /scoring-runs/{runId}

Тест 1 — получение существующего запуска.  
GET-запрос с корректным `runId` завершился с кодом **200 OK** и вернул метаданные запуска.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 54 58" src="https://github.com/user-attachments/assets/e3fd7b91-304b-4916-8426-69f333d033b8" />

Тест 2 — запрос несуществующего запуска.  
GET-запрос с неверным `runId` вернул код **404 Not Found** и сообщение об ошибке с кодом `RUN_NOT_FOUND`.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 55 48" src="https://github.com/user-attachments/assets/786d6dac-2597-4635-894d-914dab0374be" />

### GET /scoring-runs/{runId}/result

Тест 1 — получение полного результата скоринга.  
GET-запрос с существующим `runId` вернул код **200 OK** и полный результат скоринга, включая правила и вызванные сервисы.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 56 26" src="https://github.com/user-attachments/assets/10fdf5ad-d065-4abd-af31-5da793856c40" />

Тест 2 — запрос результата для несуществующего запуска.  
GET-запрос с некорректным `runId` завершился с кодом **404 Not Found**.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 57 50" src="https://github.com/user-attachments/assets/e1a28a6e-de9d-4d76-9a17-355e7592d0a2" />

### GET /scoring-runs/{runId}/rules

Тест 1 — получение списка сработавших правил.  
GET-запрос с корректным `runId` вернул код **200 OK** и массив правил с указанием уровня критичности и веса.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 58 47" src="https://github.com/user-attachments/assets/732f630b-3637-4b4c-9add-7227357c62ba" />

Тест 2 — запрос правил для несуществующего запуска.  
GET-запрос завершился с кодом **404 Not Found**, что подтверждает корректную обработку ошибок.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 00 59 23" src="https://github.com/user-attachments/assets/c35a7368-61c8-469e-99d5-c11bc268fa13" />

### GET /scoring-runs/{runId}/service-calls

Тест 1 — получение списка вызванных сервисов.  
GET-запрос с корректным `runId` вернул код **200 OK** и список сервисов с информацией о статусе и времени выполнения.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 01 00 31" src="https://github.com/user-attachments/assets/070b6105-2b23-45aa-9e2a-6d587f5a5829" />

Тест 2 — запрос для несуществующего запуска.  
GET-запрос завершился с кодом **404 Not Found**.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 01 00 59" src="https://github.com/user-attachments/assets/bad6d49c-9a30-4d3f-b30c-7fcf722edd8b" />

### PUT /scoring-runs/{runId}

Тест 1 — изменение статуса запуска (soft-delete).  
PUT-запрос с телом `{ "status": "deleted" }` завершился с кодом **200 OK**, статус запуска был успешно обновлён.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 01 01 58" src="https://github.com/user-attachments/assets/da288f4b-c5b4-4eca-b969-ce94beacbf41" />

Тест 2 — попытка некорректного обновления.  
При попытке изменения данных завершённого запуска сервер вернул код **409 Conflict**.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 01 03 44" src="https://github.com/user-attachments/assets/ad097a79-fab5-48e0-b64e-9e560a679724" />

### DELETE /scoring-runs/{runId}

Тест 1 — удаление существующего запуска.  
DELETE-запрос с корректным `runId` вернул код **204 No Content**.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 01 04 38" src="https://github.com/user-attachments/assets/8ad2465e-6c01-4698-b71b-e698ee9b0f35" />

Тест 2 — повторное удаление (идемпотентность).  
Повторный DELETE-запрос для того же `runId` также завершился с кодом **204 No Content**, что подтверждает идемпотентность операции.
<img width="1680" height="1050" alt="Снимок экрана 2026-02-04 в 01 05 03" src="https://github.com/user-attachments/assets/14a75881-6ce9-471e-b608-b0f3977fc420" />


