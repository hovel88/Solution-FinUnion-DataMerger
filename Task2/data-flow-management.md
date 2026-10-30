# data-flow-management.md — управление потоками данных: FinUnion + RetailBank

> Назначение документа: описать управление потоками данных для промежуточного состояния (3 месяца) после слияния FinUnion и RetailBank. Это нужно не столько для проектирования, но больше для администрирования потоков данных, в рамках которого определяются правила маршрутизации, форматы сообщений, SLA доставки, типы операций над объектами и метрики качества на всех участках интеграционного слоя.

## Таблица управления потоками

### 1. Потоки от клиентов (FinUnion)

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 1 | Клиент FU | FU CRM | Клиент, Обращение | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 200 мс | FU CRM Team | Полнота ФИО, ИНН; валидация формата | TLS 1.3, RBAC, маскирование ПДн | latency, error rate |
| 2 | Сотрудник FU | FU CRM | Клиент | 🔵 [Read]/🟡 [Update] | JSON | Real-time | Sync (REST) | < 200 мс | FU CRM Team | Полнота данных; корректность запроса | TLS 1.3, RBAC, аудит | latency, error rate |

### 2. Потоки внутри FinUnion

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 3 | FU CRM | FU Core | Клиент, Договор | 🟢 [Create]/🟡 [Update] | JSON | Real-time | Sync (REST) | < 500 мс | FU Core Team | Полнота party_id, product_id; уникальность contract_id | TLS 1.3, RBAC, аудит | latency, error rate |
| 4 | FU CRM | FU Loans | Заявка на кредит | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | FU Loans Team | Полнота заявки; корректность суммы | TLS 1.3, RBAC, аудит | latency, error rate |
| 5 | FU Core | FU Cards | Счет, карта | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | FU Cards Team | Полнота account_id; уникальность card_id | TLS 1.3, RBAC, токенизация карты | latency, error rate |
| 6 | FU Core | FU Loans | Счет, платеж | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | FU Loans Team | Полнота payment_id; корректность суммы | TLS 1.3, RBAC, аудит | latency, error rate |

### 3. Потоки от клиентов (RetailBank, старые)

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 7 | Клиент RB | RB CRM | Клиент, обращение | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 200 мс | RB CRM Team | Полнота ФИО, ИНН; валидация формата | TLS 1.3, RBAC, маскирование ПДн | latency, error rate |
| 8 | Сотрудник RB | RB CRM | Клиент | 🔵 [Read]/🟡 [Update] | JSON | Real-time | Sync (REST) | < 200 мс | RB CRM Team | Полнота данных; корректность запроса | TLS 1.3, RBAC, аудит | latency, error rate |

### 4. Потоки внутри RetailBank (старые продукты)

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 9 | RB CRM | RB Core | Клиент (старый) | 🟢 [Create]/🟡 [Update] | JSON | Real-time | Sync (REST) | < 500 мс | RB Core Team | Полнота party_id; уникальность contract_id | TLS 1.3, RBAC, аудит | latency, error rate |
| 10 | RB CRM | RB Loans | Заявка (старая) | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | RB Loans Team | Полнота заявки; корректность суммы | TLS 1.3, RBAC, аудит | latency, error rate |
| 11 | RB Core | RB Cards | Счет (старый) | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | RB Cards Team | Полнота account_id; уникальность card_id | TLS 1.3, RBAC, токенизация карты | latency, error rate |
| 12 | RB Core | RB Loans | Счет (старый) | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | RB Loans Team | Полнота payment_id; корректность суммы | TLS 1.3, RBAC, аудит | latency, error rate |

### 5. Потоки, когда старый клиент RetailBank хочет новые продукты

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 13 | RB CRM | Integration Layer | Клиент | 🟢 [Create]/🟡 [Update] + 🟠 [Validate] | JSON | Near real-time | Async (Kafka) | < 5 мин | Integration Team | Полнота ФИО, ИНН, паспорта; валидация | TLS 1.3, RBAC, маскирование ПДн | Kafka Lag, DLQ Size |
| 14 | Integration Layer | FU CRM | party_id | 🟡 [Update] | JSON | Near real-time | Async (Kafka) | < 5 мин | Integration Team | Уникальность party_id; дедупликация | TLS 1.3, RBAC, аудит | Kafka Lag, DLQ Size |
| 15 | FU CRM | FU Loans | Новый кредит | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | FU Loans Team | Полнота заявки; корректность суммы | TLS 1.3, RBAC, аудит | latency, error rate |
| 16 | FU CRM | FU Cards | Новая карта | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | FU Cards Team | Полнота account_id; уникальность card_id | TLS 1.3, RBAC, токенизация карты | latency, error rate |
| 17 | FU CRM | FU Core | Новый счет | 🟢 [Create] | JSON | Real-time | Sync (REST) | < 500 мс | FU Core Team | Полнота party_id; уникальность account_id | TLS 1.3, RBAC, аудит | latency, error rate |

### 6. Сопоставление продуктов (Integration Layer)

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 18 | FU Loans | Integration Layer | Новые кредиты | 🔵 [Read] | JSON | Near real-time | Async (Kafka) | < 5 мин | Integration Team | Полнота party_id; уникальность loan_id | TLS 1.3, RBAC, аудит | Kafka Lag, DLQ Size |
| 19 | RB Loans | Integration Layer | Старые кредиты | 🔵 [Read] | JSON | Near real-time | Async (Kafka) | < 5 мин | Integration Team | Полнота party_id; уникальность loan_id | TLS 1.3, RBAC, аудит | Kafka Lag, DLQ Size |
| 20 | FU Cards | Integration Layer | Новые карты | 🔵 [Read] | JSON | Near real-time | Async (Kafka) | < 5 мин | Integration Team | Полнота party_id; уникальность card_id | TLS 1.3, RBAC, токенизация карты | Kafka Lag, DLQ Size |
| 21 | RB Cards | Integration Layer | Старые карты | 🔵 [Read] | JSON | Near real-time | Async (Kafka) | < 5 мин | Integration Team | Полнота party_id; уникальность card_id | TLS 1.3, RBAC, токенизация карты | Kafka Lag, DLQ Size |
| 22 | FU Core | Integration Layer | Новые счета | 🔵 [Read] | JSON | Near real-time | Async (Kafka) | < 5 мин | Integration Team | Полнота party_id; уникальность account_id | TLS 1.3, RBAC, аудит | Kafka Lag, DLQ Size |
| 23 | RB Core | Integration Layer | Старые счета | 🔵 [Read] | JSON | Near real-time | Async (Kafka) | < 5 мин | Integration Team | Полнота party_id; уникальность account_id | TLS 1.3, RBAC, аудит | Kafka Lag, DLQ Size |

### 7. Потоки в DWH (batch)

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 24 | FU Core | DWH | Новые счета | 🔵 [Read]+🟢 [Create] | Parquet | Batch (ночь) | Batch (ETL) | К 06:00 | DWH Team | Полнота account_id; уникальность | TLS 1.3, RBAC, маскирование ПДн | ETL-мониторинг (успех/ошибка) |
| 25 | FU Cards | DWH | Новые карты | 🔵 [Read]+🟢 [Create] | Parquet | Batch (ночь) | Batch (ETL) | К 06:00 | DWH Team | Полнота card_id; уникальность | TLS 1.3, RBAC, токенизация | ETL-мониторинг |
| 26 | FU Loans | DWH | Новые кредиты | 🔵 [Read]+🟢 [Create] | Parquet | Batch (ночь) | Batch (ETL) | К 06:00 | DWH Team | Полнота loan_id; уникальность | TLS 1.3, RBAC, маскирование | ETL-мониторинг |
| 27 | RB Core | DWH | Старые счета | 🔵 [Read]+🟢 [Create] | Parquet | Batch (ночь) | Batch (ETL) | К 06:00 | DWH Team | Полнота account_id; уникальность | TLS 1.3, RBAC, маскирование | ETL-мониторинг |
| 28 | RB Cards | DWH | Старые карты | 🔵 [Read]+🟢 [Create] | Parquet | Batch (ночь) | Batch (ETL) | К 06:00 | DWH Team | Полнота card_id; уникальность | TLS 1.3, RBAC, токенизация | ETL-мониторинг |
| 29 | RB Loans | DWH | Старые кредиты | 🔵 [Read]+🟢 [Create] | Parquet | Batch (ночь) | Batch (ETL) | К 06:00 | DWH Team | Полнота loan_id; уникальность | TLS 1.3, RBAC, маскирование | ETL-мониторинг |
| 30 | Integration Layer | DWH | Сопоставленные данные | 🟢 [Create] | Parquet | Batch (ночь) | Batch (ETL) | К 06:00 | DWH Team | Полнота party_id; консистентность | TLS 1.3, RBAC, маскирование | ETL-мониторинг |

### 8. Потоки к потребителям

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 31 | DWH | BI | Витрины | 🔵 [Read]+🟣 [Aggregate] | SQL | Real-time | Sync (SQL) | < 5 сек | DWH Team | Полнота витрин; корректность агрегаций | TLS 1.3, RBAC, маскирование ПДн | BI-мониторинг (latency) |
| 32 | DWH | Регулятор | Отчетность | 🔵 [Read] | XML | Batch (месяц) | Batch | По регламенту | DWH Team | Полнота отчетности; корректность форм | TLS 1.3, RBAC, подпись | Мониторинг отправки |

### 9. Потоки в DataHub (каталог)

| ## | Источник | Приемник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец | Контроль качества | Безопасность | Мониторинг |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| 33 | FU CRM | DataHub | Метаданные | 🟢 [Create] | JSON | Near real-time | Async | < 15 мин | Data Office | Полнота метаданных; lineage | TLS 1.3, RBAC | DataHub-мониторинг |
| 34 | RB CRM | DataHub | Метаданные | 🟢 [Create] | JSON | Near real-time | Async | < 15 мин | Data Office | Полнота метаданных; lineage | TLS 1.3, RBAC | DataHub-мониторинг |
| 35 | Integration Layer | DataHub | Lineage | 🟢 [Create] | JSON | Near real-time | Async | < 15 мин | Data Office | Полнота lineage | TLS 1.3, RBAC | DataHub-мониторинг |
| 36 | DWH | DataHub | Метаданные | 🟢 [Create] | JSON | Near real-time | Async | < 15 мин | Data Office | Полнота метаданных; lineage | TLS 1.3, RBAC | DataHub-мониторинг |

## Требования к безопасности

### Data in Transit

| Мера | Применяется |
| -- | -- |
| **TLS 1.3** | Все потоки (sync и async) |
| **Подпись сообщений** | Критичные потоки (платежи, регулятор) |
| **Контроль схем** | Все потоки (JSON Schema) |
| **Идемпотентность** | Async-потоки (Kafka) |
| **Очереди** | Async-потоки (Kafka) |
| **Retry** | Async-потоки (Kafka) |
| **Мониторинг** | Все потоки |

### Data at Rest

| Мера | Применяется |
| -- | -- |
| **Шифрование** | DWH, DataHub, FU/RB БД |
| **RBAC** | Все системы |
| **Маскирование** | BI, UI |
| **Аудит** | Все системы |
| **Токенизация** | Карты, ПДн в DWH |

## Метрики

| Поток | Метрика | Измеряет | Целевое значение | Алерт |
| -- | -- | -- | -- | -- |
| CRM —> Integration Layer | **Consumer Lag** | Отставание консьюмера | < 1000 сообщений | > 5000 сообщений |
| Integration Layer —> CRM | **Consumer Lag** | Отставание консьюмера | < 1000 сообщений | > 5000 сообщений |
| Все async-потоки | **DLQ Size** | Размер очереди недоставленных | < 100 сообщений | > 500 сообщений |
| Все async-потоки | **End-to-End Latency** | Сквозная задержка | < 5 мин | > 20 мин |
| Все sync-потоки | **End-to-End Latency** | Сквозная задержка | < 500 мс | > 1 сек |
| Все потоки | **Message Loss Rate** | Процент потерянных сообщений | < 0.01% | > 0.1% |
| Все потоки | **Throughput / TPS** | Пропускная способность | — | — |
| Batch-потоки | **ETL-мониторинг** | Выгрузка в DWH | SLA к 06:00 | — |
