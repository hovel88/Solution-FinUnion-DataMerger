# migration-target-state.md — образ целевого результата: FinUnion + RetailBank

> Назначение документа: зафиксировать образ целевого результата через 12 месяцев после слияния FinUnion и RetailBank. Показать какие системы остаются, какие системы выводятся из эксплуатации, какие данные мигрируют, какие критерии завершения перехода.

## Системы, который остаются

| Система | Назначение | Тип | Причина оставления |
| -- | -- | -- | -- |
| **FU CRM** | Обращения, Клиенты | OLTP | Целевая CRM FinUnion |
| **FU Core** | Счета, Платежи | OLTP | Целевая Core Banking |
| **FU Cards** | Карты | OLTP | Целевой Card Processing |
| **FU Loans** | Кредиты | OLTP | Целевая Loan System |
| **MDM Customer Hub** | Золотая запись Клиента | MDM | Целевая мастер-система клиентов |
| **MDM Product Catalog** | Продукты, Тарифы | MDM | Целевая мастер-система продуктов |
| **MDM Reference Data Management** | Справочники, Офисы | MDM | Целевая мастер-система справочников |
| **API Gateway** | Sync REST | Integration | Единая точка для MDM. применение политик и безопасности |
| **Kafka** | Async события | Integration | Event streaming |
| **Debezium** | CDC | Integration / OLAP | Захват изменений из баз целевых операционных систем и передача в Kafka |
| **Read Replica** | Реплика для batch | OLAP | Чтобы нагружать операционные БД сканами для аналитического контура |
| **DWH** | Регламентированная отчётность | OLAP | Целевое хранилище для BI |
| **Lakehouse** | История, события, ML | OLAP | Целевое хранилище для big data |
| **Airflow** | Оркестрация batch | Integration / OLAP | Управление пайплайнами |
| **BI** | Дашборды | OLAP | Визуализация бизнес-показателей |
| **DataHub** | Каталог, lineage | Каталог | Обеспечивает трассируемость данных |

## Системы, которые выводятся

| Система | Причина вывода | Когда выводится |
| -- | -- | -- |
| **RB CRM** | Клиенты мигрируют в MDM Customer Hub | После миграции клиентов |
| **RB Core** | Счета/Платежи мигрируют в FU Core | После миграции счетов |
| **RB Cards** | Карты мигрируют в FU Cards | После миграции карт |
| **RB Loans** | Кредиты мигрируют в FU Loans | После миграции кредитов |
| **RB DWH** | Отчётность мигрирует в DWH | После миграции отчётности |
| **RB BI** | Дашборды мигрируют в BI | После миграции дашбордов |
| **Point-to-point интеграции** | Унификация через Integration Layer | После миграции всех систем |

## Данные, которые мигрируют

| Данные | Откуда | Куда |
| -- | -- | -- |
| **Клиенты RB** | RB CRM | MDM Customer Hub |
| **Счета RB** | RB Core | FU Core |
| **Карты RB** | RB Cards | FU Cards |
| **Кредиты RB** | RB Loans | FU Loans |
| **Транзакции RB** | RB Core | FU Core + Lakehouse (история) |
| **Отчётность RB** | RB DWH | DWH |
| **Справочники RB** | RB Core | MDM Reference Data Management |
| **Продукты RB** | RB CRM | MDM Product Catalog |

## Потоки, которые переключаются

| Поток | Откуда | Куда |
| -- | -- | -- |
| **Клиенты** | RB CRM | FU CRM (через Integration Layer в MDM) |
| **Счета** | RB Core | FU Core (через Integration Layer) |
| **Карты** | RB Cards | FU Cards (через Integration Layer) |
| **Кредиты** | RB Loans | FU Loans (через Integration Layer) |
| **Отчётность** | RB DWH | DWH (через Airflow) |
| **Справочники** | RB Core | MDM Reference Data Management (через API Gateway) |
| **Продукты** | RB CRM | MDM Product Catalog (через API Gateway) |

## Критерии завершения перехода

| ## | Критерий | Целевое значение | Как проверить |
| -- | -- | -- | -- |
| 1 | **Все клиенты RB сопоставлены** | 100% имеют `party_id` в MDM | SQL-запрос: `SELECT COUNT(*) FROM customers WHERE party_id IS NULL` = 0 |
| 2 | **Все счета RB мигрированы** | 100% в FU Core | SQL-запрос: `SELECT COUNT(*) FROM accounts WHERE source = 'RB'` = 0 |
| 3 | **Все карты RB мигрированы** | 100% в FU Cards | SQL-запрос: `SELECT COUNT(*) FROM cards WHERE source = 'RB'` = 0 |
| 4 | **Все кредиты RB мигрированы** | 100% в FU Loans | SQL-запрос: `SELECT COUNT(*) FROM loans WHERE source = 'RB'` = 0 |
| 5 | **RB-системы выведены** | 0 активных запросов к RB | Мониторинг: `RB_CRM_requests = 0`, `RB_Core_requests = 0`, `RB_Cards_requests = 0`, `RB_Loans_requests = 0` |
| 6 | **DataHub содержит lineage** | 100% данных имеют lineage | DataHub UI: все таблицы имеют lineage |
| 7 | **DWH объединяет FU + RB** | 100% отчётности из DWH | BI-отчёты: все работают с DWH, данные по RB не потеряны, нет разрывов |
| 8 | **Lakehouse содержит историю** | 100% транзакций (CDC) | Trino: `SELECT COUNT(*) FROM lakehouse.transactions` = полная история |
| 9 | **BI работает с новыми витринами** | 0 отчётов на RB DWH | BI-аудит: все отчёты берут данные из DWH |
| 10 | **Регулятор получает отчётность** | Все формы из DWH | Отчетность: повторная генерация прошлых отчетов дает те же результаты. Регулятор: отчётность принята |
