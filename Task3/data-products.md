# data-products.md — доменные Data Products: FinUnion + RetailBank

> Назначение документа: показать, как элементы Data Mesh могут применяться в целевой архитектуре данных объединённого банка, на примере нескольких доменных data-продуктов.

## Data Products

| ## | Data Product | Домен | BIAN-домен | Назначение | Основные данные | Владелец | Потребители | Источники | SLA | Качество | Безопасность |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 1 | **Customer 360** | Клиенты | Party Reference Data Directory | Единый профиль клиента (ФЛ, ЮЛ, ИП)<br><br>***главная цель после слияния*** | `party_id`<br>`inn`<br>`pasport_masked`<br>`ФИО/Наименование`<br>`контакты`<br>`согласия`<br>`segment` | Data Office (MDM Customer Hub) | CRM, BI, маркетинг, риск-менеджмент | MDM Customer Hub, FU CRM, RB CRM (сопоставление) | Near real-time (обновление < 5 мин) | Полнота ФИО, ИНН; уникальность party_id; дедупликация | Шифрование, RBAC, маскирование ПДн, аудит |
| 2 | **Loan Portfolio** | Кредиты | Consumer Loan | Кредитный портфель (выдачи, остатки, платежи)<br><br>***ключевой для отчётности и риск-менеджмента*** | `loan_id`<br>`party_id`<br>`amount`<br>`loan_rate`<br>`loan_term`<br>`balance`<br>`status`<br>`payment_schedule` | Кредитный отдел (FU Loans) | BI, риск-менеджмент, регулятор | FU Loans, MDM Customer Hub (party_id) | Ежедневно (batch, к 06:00) | Полнота loan_id; корректность графика платежей; уникальность | Шифрование, RBAC, маскирование сумм |
| 3 | **Payment History** | Платежи | Payment Order | История платежей (переводы, погашения)<br><br>***для антифрода (real-time) и клиентского сервиса*** | `payment_id`<br>`from_account_id`<br>`to_account_id`<br>`amount`<br>`currency`<br>`status`<br>`payment_date` | Платёжный отдел (FU Core) | BI, антифрод, клиентский сервис | FU Core, MDM Customer Hub | Near real-time (CDC, < 5 мин) | Полнота payment_id; корректность статусов; отсутствие дублей | Шифрование, RBAC, маскирование реквизитов |
| 4 | **Card Transactions** | Карты | Card Clearing | Карточные операции (покупки, снятия)<br><br>***для антифрода (real-time) и маркетинга (анализ покупок)*** | `transaction_id`<br>`card_id_masked`<br>`party_id`<br>`amount`<br>`merchant`<br>`status`<br>`transaction_date` | Отдел карт (FU Cards) | BI, антифрод, маркетинг | FU Cards, MDM Customer Hub | Near real-time (CDC + streaming, < 1 мин) | Полнота transaction_id; корректность merchant; отсутствие дублей | Токенизация карты, RBAC, маскирование |
| 5 | **Risk Scores** | Риски | Credit Risk Models | Скоринг клиентов (кредитный риск)<br><br>***для кредитования и регуляторной отчётности*** | `risk_id`<br>`party_id`<br>`score`<br>`model`<br>`calc_date`<br>`status` | Риск-менеджмент (FU Loans) | Кредитный отдел, BI, регулятор | FU Loans, MDM Customer Hub | Ежедневно (batch) | Полнота score; корректность модели; уникальность | Шифрование, RBAC, маскирование баллов |

## Связь с Data Mesh

| Принцип Data Mesh | Описание | Как реализовано |
| -- | -- | -- |
| **Domain ownership** | Домены владеют данными | 5 доменов: клиенты, кредиты, платежи, карты, риски |
| **Data as a Product** | Данные это продукт с владельцем, контрактом, SLA | 5 data products с владельцем, SLA, качеством и мерами безопасности |
| **Self-serve platform** | Общая платформа (Airflow, DataHub) | Airflow (оркестрация), DataHub (каталог), Trino (SQL) |
| **Federated governance** | Единые правила (идентификаторы, качество) | Единые `party_id`, справочники (MDM системы), качество (DQ) |

## Диаграмма положения Data Products в архитектуре системы

```mermaid
flowchart TB
    subgraph SOURCES["Источники"]
        FU_CRM["FU CRM"]
        FU_Core["FU Core"]
        FU_Cards["FU Cards"]
        FU_Loans["FU Loans"]
        MDM["MDM Customer Hub"]
    end

    subgraph PRODUCTS["Data Products (Data Mesh)"]
        Customer360["Customer 360\n(Data Office)"]
        LoanPortfolio["Loan Portfolio\n(Кредитный отдел)"]
        PaymentHistory["Payment History\n(Платёжный отдел)"]
        CardTransactions["Card Transactions\n(Отдел карт)"]
        RiskScores["Risk Scores\n(Риск-менеджмент)"]
    end

    subgraph CONSUMERS["Потребители"]
        BI["BI"]
        CRM["CRM"]
        Risk["Риск-менеджмент"]
        Marketing["Маркетинг"]
        Regulator["Регулятор"]
        Antifraud["Антифрод"]
    end

    FU_CRM --> Customer360
    MDM --> Customer360
    FU_Loans --> LoanPortfolio
    FU_Core --> PaymentHistory
    FU_Cards --> CardTransactions
    FU_Loans --> RiskScores

    Customer360 --> BI
    Customer360 --> CRM
    Customer360 --> Marketing
    Customer360 --> Risk

    LoanPortfolio --> BI
    LoanPortfolio --> Risk
    LoanPortfolio --> Regulator

    PaymentHistory --> BI
    PaymentHistory --> Antifraud
    PaymentHistory --> CRM

    CardTransactions --> BI
    CardTransactions --> Antifraud
    CardTransactions --> Marketing

    RiskScores --> BI
    RiskScores --> Regulator
```
