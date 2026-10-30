# reference-model.md — референсная модель BIAN: FinUnion + RetailBank

> Назначение документа: сопоставить ключевые сущности FinUnion + RetailBank с сервисными доменами BIAN (Banking Industry Architecture Network). Это позволяет унифицировать, определить границы и пересечения между системами FinUnion и RetailBank. А также поможет для будушей подготовки Data Products.

## Маппинг сущностей на BIAN Service Domains

| ## | Сущность | BIAN Service Domain | BIAN Business Area | Зачем это нужно |
| -- | -- | -- | -- | -- |
| 01 | **Клиент (Party)** | **Party Reference Data Directory** | Customers | Единый подход к клиентским данным (ФЛ/ЮЛ/ИП) |
| 02 | **Договор (Contract)** | **Customer Agreement** | Sales / Products | Унификация юридических условий после слияния |
| 03 | **Счет (Account)** | **Current Account** | Consumer Banking / Products | Унификация модели счета (FU Core vs RB Core) |
| 04 | **Карта (Card)** | **Card Clearing** | Cards | Унификация статусов карт и клиринга |
| 05 | **Кредит (Loan)** | **Consumer Loan** | Loans and Deposits | Унификация правил кредитования |
| 06 | **Транзакция (Transaction)** | **Position Keeping** | Operations / Accounting | Унификация учета операций |
| 07 | **Платеж (Payment)** | **Payment Order** | Payments | Унификация платежных поручений |
| 08 | **Продукт (Product)** | **Product Directory** | Products | Унификация каталога продуктов |
| 09 | **Обращение (Request)** | **Customer Care** | Customer Care | Унификация обработки обращений |
| 10 | **Справочник (Reference)** | **Public Reference Data Management** | Reference Data | Унификация справочников (валюты, страны) |
| 11 | **Согласие (Consent)** | — | — | |
| 12 | **Скоринг (Risk Score)** | **Credit Risk Models** | Finance and Risk Management | Унификация моделей скоринга |
| 13 | **Документ (Document)** | **Document Library** | Operations / Document Services | Унификация управления документами |
| 14 | **Офис (Branch)** | **Location Data Management** | Operations | Унификация справочника офисов |
| 15 | **Тариф (Tariff)** | **Product Directory** (часть) | Products | Унификация тарифов (часть Product Directory) |

## Что дает BIAN

BIAN Service Landscape — это стандартизированная, vendor-neutral структура для банковской бизнес-архитектуры. Она декомпозирует банковские операции на дискретные, переиспользуемые функциональные блоки — Service Domains.

Ключевые понятия в BIAN:

- **Service Domain** — "назначаемая зона ответственности", представляющая дискретную бизнес-функцию. Каждый домен имеет единственного владельца.
- **Business Area** — высокоуровневая группировка (например, "Products", "Operations").
- **Business Domain** — крупная бизнес-способность внутри Business Area (например, "Loans and Deposits").

### Унификация терминологии по BIAN

Если FinUnion и RetailBank называют одни и те же сущности по-разному (например, **Account** и **Product Account**), то при слиянии можно перейти на единую модель именования (в данном случае **Current Account**). Либо по крайней мере, поставить в соотвествие сущности одной системы сущностям другой.

### Определение границ владения и пересечений между системами

При использовании общей модели BIAN становится лучше видно, какие системы дублируют функции и что можно объединять при слиянии. BIAN помогает провести MECE-разбиение и выделить единого владельца для каждой функции, из-за этого бслияние систем должно происхходить с меньшим количеством конфликтов.

| Система FinUnion | Система RetailBank | Общий домен BIAN |
| -- | -- | -- |
| FU CRM | RB CRM | **Party Reference Data Directory** |
| FU Core | RB Core | **Current Account** |
| FU Loans | RB Loans | **Consumer Loan** |
| FU Cards | RB Cards | **Card Clearing** |
| FU DWH | RB DWH | **Position Keeping** |

### Подготовка целевых доменов данных

BIAN является некой стандартизированной бизнес-архитектурой, референсом в банковской сфере, что дает основу для выявления доменов данных и продуктов данных, после сопоставления внутренних сощностей доменам из BIAN. Например:

| ## | Сущность | BIAN Service Domain | Целевой домен данных (Data Mesh) | Data Product (пример названия) |
| -- | -- | -- | -- | -- |
| 01 | Клиент (Party) | Party Reference Data Directory | Клиенты (Customers) | Customers (единый профиль) |
| 02 | Договор (Contract) | Customer Agreement | Договоры (Agreements) | Contract Registry (реестр договоров) |
| 03 | Счет (Account) | Current Account | Счета (Accounts) | Account Balance (балансы и остатки) |
| 04 | Карта (Card) | Card Clearing | Карты (Cards) | Card Transactions (карточные операции) |
| 05 | Кредит (Loan) | Consumer Loan  | Кредиты (Loans) | Loan Portfolio (кредитный портфель) |
| 06 | Транзакция (Transaction) | Position Keeping  | Операции (Transactions) | Transaction Ledger (журнал операций) |
| 07 | Платеж (Payment) | Payment Order  | Платежи (Payments) | Payment History (история платежей) |
| 08 | Продукт (Product) | Product Directory | Продукты (Products) | Product Catalog (каталог продуктов) |
| 09 | Обращение (Request) | Customer Care | Обслуживание (Servicing) | Customer Requests (обращения) |
| 10 | Справочник (Reference) | Public Reference Data Management | Справочники (Reference Data) | Reference Data Hub (справочники) |
| 11 | Согласие (Consent) | — | Комплаенс (Compliance) | Consent Registry (реестр согласий) |
| 12 | Скоринг (Risk Score) | Credit Risk Models | Риски (Risk) | Risk Scores (скоринги) |
| 13 | Документ (Document) | Document Library | Документы (Documents) | Document Vault (хранилище документов) |
| 14 | Офис (Branch) | Location Data Management | Справочники (Reference Data) | Branch Directory (справочник офисов) |
| 15 | Тариф (Tariff) | Product Directory | Продукты (Products) | Tariff Catalog (тарифы) |
