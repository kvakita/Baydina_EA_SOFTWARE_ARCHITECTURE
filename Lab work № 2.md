# Диаграмма системного контекста
![image](https://uml.planttext.com/plantuml/png/ZLPDRzD04BtxLomzfL9ABvmGGa2e10VKgkazYXghMAHE5RiL4K9AcduWWaW18H0VAfNW20fj8zmqJl_2xb_Wb_1cTftOZWjKQkjTpywyUVDcrhSylrptDzUhejZmV7kzK7Dz-x4_LwYjRrITihjg5rThTSTTiwuuxcfrisAB6uLhyuLh9MDvqx9ynw_QGXx9GBNKjkpBeGnLHvu9EP0ZFDBMW5vT5_9GdikUsbOxSgIQCEm9rI8pB08PoXDiyAcXl21Bd0nLQuxUQZjvB8pv1V-5Bnmyfe1gbrwTOzddeB2rh2MxxjNSNDtn7fNARhcA9FDYHdxH_kOG8SGl3h6VmGozV8UT2zNcW42WjWMMJVA7HHERmLIWl9Xuah_WhQdQV1BsVVaBYtFr0gQKUaUGWv7ygPwhBeA4v8QSiR5wffUgUtLcDeBl-VMoRwyrIkNAkkCInWDv2eu2jWIpXWam8cBBFlWCv1adzHaxvWqJ5akiXsL0R1EmL3cG6qhPd_OQnE1LxAtQXeSeT-sjnGrFp0lvdIfEDKJgFKvyZt9YA9z85Smuno6MMdXiyKOtug8raPFgv0NyH5ZFOSJWjRnsO7-2SZMZ84GCeREqf4jtQfjkfLnlvBm6WVmF3SxCYYUMm8_saFDq3vGyNfAFkDWppfF9Qn07aI9O604U0I1LTw3Q8gtfX9wu4ZjSbO1L5MdiX0LrabIOzWDH8PpWel4111f5NKZzr6DgcPgdOt8M7lkvrJhbL5X-8EHFjXgO5DuplOObJQwW476qFRMRJsBb7UXpfFLZ-dINuiQ2rkqr1xfkK9mFsHqDy9nkNwkF8HtH4r8Vu3_NN90Y4FSqwflhIGfwX3b10JP-xxrEO5QT5BgrsXP5UyEwZ650z-RaM-n_aJ_aHwJv5PrrDClGVh00JX5Wdyof9APpM6uOA2Ha612AaDkAo9p0fncnbq6LXpdiEpNNqyhZ2b4rk8z2qv_8pkX0gu_c2eQ9xYOI1r7QvYuceiPTbkKDlleyZFR9BeMclEa_d6u1m6xpWZGHCBS3hHd4pXMBYvgZ0ymJq_UOasDD1Od-ZW_44RVxC0D5nV4r389wJ60O11TWsmZu6xkSJDYu0KVbRkBnt3MuJDECmDI-UeNDqCmcFLEMxMhgDfaU3l-wG5GxRtPWCcHnD6D3EUlEe6pkhQmi5O618AHlX6aG_p_mqvCz7ai7IpcF571mMSbLTvw2P5MQ9w9wWIsYGqUbWlCjjNZ_RY5AC1LJpoX3BS5z5iygqLE9wKHkbnUAAvUuZK-XRD-cxySJ1wERfZ3f5TQTz7Wz-qT0CrWEcOOs6in4UPavFEA9aUFRI_SlyPkO4OSJjKLN3rqDiI7HayE_-nltrw4U1OI4sk7IBngwm_5Dqs4jy4M_i6Mx_hpXmhfbknLyPrf_0000)
'''
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

'''
