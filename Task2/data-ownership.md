# data-ownership.md — владение данными: FinUnion + RetailBank

> Назначение документа: определить роли и ответственность за данные для ключевых сущностей после слияния FinUnion и RetailBank.

## Матрица владения данными

| ## | Сущность | Data Owner | Data Steward | Producer | Consumer | Правила изменения | Правила качества |
| -- | -- | -- | -- | -- | -- | -- | -- |
| 1 | **Клиент (Party)** | Директор по работе с клиентами | **Data Office** (FinUnion) | FU CRM, RB CRM | FU Core, RB Core, FU Loans, RB Loans, DWH, BI, MDM | Только через MDM Customer Hub (целевое); на 3 мес — через FU CRM с правилами сопоставления | Полнота ФИО, ИНН, паспорта<br>уникальность party_id<br>дедупликация |
| 2 | **Договор (Contract)** | Директор по продуктам | Юридический отдел | FU Core, RB Core | FU DWH, RB DWH, BI, MDM | Только через Core Banking (FU Core) | Полнота party_id, product_id<br>корректность дат<br>уникальность contract_id |
| 3 | **Счет (Account)** | Директор по операциям | Операционный отдел | FU Core, RB Core | FU DWH, RB DWH, BI, FU Cards, FU Loans | Только через Core Banking (FU Core) | Полнота account_number, balance, currency<br>уникальность account_id<br>корректность статусов |
| 4 | **Карта (Card)** | Директор по карточным продуктам | Отдел банковских карт | FU Cards, RB Cards | FU DWH, RB DWH, BI, FU Core | Только через Card Processing (FU Cards) | Полнота card_number, expiry_date<br>уникальность card_id<br>корректность статусов |
| 5 | **Кредит (Loan)** | Директор по кредитованию | Кредитный отдел | FU Loans, RB Loans | FU DWH, RB DWH, BI, FU Core | Только через Loan System (FU Loans) | Полнота amount, rate, term<br>уникальность loan_id<br>корректность графика платежей |
| 6 | **Транзакция (Transaction)** | Директор по операциям | Операционный отдел | FU Core, RB Core | FU DWH, RB DWH, BI, FU DataLake | Только через Core Banking (FU Core) | Полнота amount, datetime, type<br>уникальность transaction_id<br>отсутствие дублей |
| 7 | **Платеж (Payment)** | Директор по платежам | Платежный отдел | FU Core, RB Core | FU DWH, RB DWH, BI, FU Loans | Только через Core Banking (FU Core) | Полнота from_account_id, to_account_id, amount<br>уникальность payment_id<br>корректность статусов |
| 8 | **Продукт (Product)** | Директор по продуктам | Отдел банковских продуктов | FU CRM, RB CRM | FU Core, RB Core, FU DWH, BI, MDM | Только через MDM Product Catalog | Полнота product_name, type, category<br> уникальность product_id<br>корректность статусов |
| 9 | **Обращение (Request)** | Директор контакт-центра | Контакт-центр | FU CRM, RB CRM | FU DWH, RB DWH, BI, SearchEngine | Только через CRM (FU CRM) | Полнота party_id, type, channel<br>уникальность request_id<br>корректность статусов |
| 10 | **Справочник (Reference)** | Директор по данным (Data Office) | Data Office | FU Core, RB Core | Все системы | Только через MDM RDM | Полнота code, value<br>уникальность reference_id<br>корректность типов |
| 11 | **Согласие (Consent)** | Директор по комплаенсу | Комплаенс-департамент | FU CRM, RB CRM | FU Core, RB Core, FU DWH, BI, MDM | Только через CRM (FU CRM) | Полнота consent_type, issue_date<br>уникальность consent_id<br>корректность статусов (выдано/отозвано) |
| 12 | **Скоринг (Risk Score)** | Директор по рискам | Риск-менеджмент | FU Loans, RB Loans | FU DWH, RB DWH, BI, FU Loans | Только через Loan System (FU Loans) | Полнота score, model, calc_date<br>уникальность risk_id<br>корректность баллов |
| 13 | **Документ (Document)** | Директор по комплаенсу | Комплаенс-департамент | FU CRM, RB CRM | FU Core, RB Core, FU DWH, BI, Object Storage, SearchEngine | Только через CRM (FU CRM) + Object Storage | Полнота doc_type, doc_number<br>уникальность document_id<br>корректность статусов<br>шифрование |
| 14 | **Офис (Branch)** | Директор сетей офисов | Операционный отдел | FU Core, RB Core | Все системы | Только через MDM RDM | Полнота branch_name, address<br>уникальность branch_id<br>корректность статусов |
| 15 | **Тариф (Tariff)** | Директор по продуктам | Отдел банковских продуктов | FU Core, RB Core | FU Core, RB Core, FU DWH, BI, MDM | Только через MDM Product Catalog | Полнота tariff_name, rate<br>уникальность tariff_id<br>корректность статусов |

## Правила изменения данных

- Единая точка входа — изменение данных только через мастер-систему
- Запрет прямых изменений — нельзя менять данные в самих потребителях (DWH, BI, Lakehouse)
- Аудит — все изменения логируются (кто, когда, что)
- Согласование — критичные изменения (ПДн, финансовые и т.д.) проходят через Data Owner

На промежуточном этапе (3 месяца):

- Старые продукты клиентов RB — в **RB-системах** (RB Loans, RB Cards, RB Core)
- Новые продукты клиентов RB — в **FU-системах** (FU Loans, FU Cards, FU Core)
- Новые клиенты — в **FU CRM**

| Продукт | Где создается новый | Где остается старый | Как изменяется |
| -- | -- | -- | -- |
| Клиент | FU CRM | RB CRM (для старых клиентов RB) | Только через мастер-систему (FU CRM или RB CRM) |
| Кредит | FU Loans | RB Loans | Только через мастер-систему (FU Loans или RB Loans) |
| Карта | FU Cards | RB Cards | Только через мастер-систему (FU Cards или RB Cards) |
| Счет | FU Core | RB Core | Только через мастер-систему (FU Core или RB Core) |
| Договор | FU Core | RB Core | Только через мастер-систему (FU Core или RB Core) |
| Транзакция | FU Core | RB Core | Только через мастер-систему (FU Core или RB Core) |
| Платеж | FU Core | RB Core | Только через мастер-систему (FU Core или RB Core) |

## Правила качества данных

- Сопоставление клиентов — все клиенты RB имеют party_id в FU CRM
- Сопоставление продуктов — все продукты привязаны к party_id
- Статусы продуктов в FU и RB согласованы
- Нет клиентов без party_id
- Нет дублей party_id

| Сущность | Полнота | Уникальность | Консистентность | Точность | Своевременность |
| -- | -- | -- | -- | -- | -- |
| Клиент | ФИО, ИНН, паспорт | party_id | Статус в CRM = статус в Core | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Договор | party_id, product_id | contract_id | Даты корректны | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Счет | account_number, balance, currency | account_id | Баланс в Core = баланс в DWH | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Карта | card_number (маскированный), expiry_date | card_id | Статус в Cards = статус в Core | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Кредит | amount, rate, term | loan_id | График платежей корректен | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Транзакция | amount, datetime, type | transaction_id | Нет дублей | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Платеж | from_account_id, to_account_id, amount | payment_id | Статус корректен | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Продукт | product_name, type, category | product_id | Каталог в CRM = каталог в Core | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Обращение | party_id, type, channel | request_id | Статус корректен | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Справочник | code, value | reference_id | Коды совпадают во всех системах | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Согласие | consent_type, issue_date | consent_id | Статус (выдано/отозвано) | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Скоринг | score, model, calc_date | risk_id | Баллы корректны | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Документ | doc_type, doc_number | document_id | Статус корректен | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Офис | branch_name, address | branch_id | Адрес корректен | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
| Тариф | tariff_name, rate, fee | tariff_id | Тариф в Core = тариф в CRM | Соответствуют ли значения реальности | Задержка между событием и загрузкой (по метрикам) |
