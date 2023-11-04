# Сбор, мониторинг и анализ аудит логов во внешний SIEM ArcSight
![Дашборд](https://user-images.githubusercontent.com/85429798/128209194-bc4eb274-1b97-4271-a712-e00a5f3f9b84.png)
![Сценарии](https://user-images.githubusercontent.com/85429798/128209212-a705f950-4eea-4305-8f21-decfc2ab7af0.png)

## Содержание

- [Сбор, мониторинг и анализ аудит логов во внешний SIEM ArcSight](#)
  * [Описание решения](#описание-решения)
  * [Два сценария отгрузки логов](#два-сценария-отгрузки-логов)
  * [Схема решения](#схема-решения)
  * [Security Content](#security-content)
  * [Долгосрочное хранение логов в S3](#долгосрочное-хранение-логов-в-s3)
  * [Инструкция для сценариев](#инструкция-для-сценариев)
      - [Пререквизиты для сценариев:](#пререквизиты-для-сценариев)
      - [Сценарий №1 - Загрузка лог файлов в  ArcSight с сервера, который находится внутри инфраструктуры удаленной площадки Заказчика](#пререквизиты-для-сценариев)
      - [Сценарий №2 - Загрузка лог файлов в  ArcSight с помощью ВМ, которая находится в Yandex Cloud "](#пререквизиты-для-сценариев)
  * [Поддержка/Консалтинговые услуги](#поддержкаконсалтинговые-услуги)


## Описание решения
Актуальная версия Security Content находится [здесь](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/auditlogs/export-auditlogs-to-ArcSight/arcsight_content) сервис партнёр по поддержке ООО «АТБ»
Решение позволяет собирать, мониторить и анализировать аудит логи в Yandex.Cloud со следующих источников:

- [Yandex Audit Trails](https://cloud.yandex.ru/docs/audit-trails/)


## Два сценария отгрузки логов
- [x] Загрузка лог файлов в ArcSight с сервера, который находится внутри инфраструктуры удаленной площадки Заказчика

- [x] Загрузка лог файлов в ArcSight с помощью ВМ, которая находится в Yandex.Cloud 


## Схема решения
#### Сценарий №1 - Загрузка лог файлов в ArcSight с сервера, который находится внутри инфраструктуры удаленной площадки Заказчика
Описание: 
- JSON файлы с логами хранятся в S3
- На сервер в инфраструктуре заказчика устанавливается утилита s3fs, которая позволяет монтировать S3 bucket, как локальную папку в ОС
- На сервер в инфраструктуре заказчика устанавливается стандартный ArcSight Connector
- Загруается security content из текущего репозитория
- ArcSight Connector с помощью security content вычитывает файлы, парсит и отправляет на сервер ArcSight 

![Схема](https://user-images.githubusercontent.com/85429798/128553857-a6837742-8e63-4d8c-967a-be92454a0cb0.png)


#### Сценарий №2 - Загрузка лог файлов в ArcSight с помощью ВМ, которая находится в Yandex Cloud
 
![Схема](https://user-images.githubusercontent.com/85429798/128553811-2d25dcc7-0500-446b-96ea-35a8fe8959ba.png)


## Security Content
Security Content - объекты ArcSight, которые загружаются по инструкции. Весь контент разработан совместно с командой партнером ООО «АТБ» с учетом многолетнего опыта Security команды Yandex.Cloud и на основе опыта Клиентов облака.

Актуальная версия Security Content находится [здесь](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/auditlogs/export-auditlogs-to-ArcSight/arcsight_content) 

Содержит следующий Security Content:
- Parsing file (+map file)
- Dashboard, на котором отражена полезная статистика
- Набор Filters, Active channels, Active lists
- Набор Правил корреляции (Rules). [Подробное описание списка правил корреляции](./Use-cases.docx) (Клиенту самостоятельно необходимо указать назначение уведомлений)
- Все интересные поля событий преобразованы в формат [Common Event Format](https://community.microfocus.com/cyberres/productdocs/w/connector-documentation/38809/arcsight-common-event-format-cef-implementation-standard)

Подробное описание мапинга полей в файле [Поля ArcSight_JSON.docx](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/auditlogs/export-auditlogs-to-ArcSight/arcsight_content)


## Долгосрочное хранение логов в S3
По умолчанию данная инструция предлагает удалять файлы после вычитывания, но вы можете одновременно хранить аудит логи Audit Trails в S3 на долгосрочной основе и отсылать в ArcSight.
Для этого необходимо создать два Audit Trails в разных S3 бакетах:
- Первый бакет будет использоваться только для хранения 
- Второй бакет будет использоваться для интеграции с ArcSight 


## Инструкция для сценариев
#### Пререквизиты для сценариев
- :white_check_mark: Object Storage Bucket для Audit Trails ([инструкция](https://cloud.yandex.ru/docs/storage/quickstart))
- :white_check_mark: Включенный сервис Audit Trails в UI ([инструкция](https://cloud.yandex.ru/docs/audit-trails/quickstart))


#### Сценарий № 1 - Загрузка лог файлов в ArcSight с сервера, который находится внутри инфраструктуры удаленной площадки Заказчика
1) Установите на сервер внутри инфраструктуры удаленной площадки и подготовьте к работе утилиту s3fs [согласно инструкции](https://cloud.yandex.ru/docs/storage/tools/s3fs). Результат: смонтированный в качестве папки Object Storage бакет, в котором находятся json файлы Audit Trails. Например: `/var/trails/`

2) Установите на ваш сервер ПО ArcSight SmartConnector (FlexAgent - JSON Folder follower) [согласно официальной инструкции](https://www.microfocus.com/documentation/arcsight/arcsight-smartconnectors/AS_smartconn_install/)

3) При установке выбирете *ArcSight FlexConnector JSON Folder Follower* и укажите примонтированную папку ранее `/var/trails/`

4) Укажите JSON configuration filename prefix - `yc`

5) Завершите установку connector 

6) Скачайте все файлы Security Content [здесь](https://github.com/yandex-cloud/yc-solution-library-for-security/tree/master/auditlogs/export-auditlogs-to-ArcSight/arcsight_content)

7) Скопируйте файл `yc.jsonparser.properties` в `<папку установки агента>/current/user/agent/flexagent`

8) Скопируйте файл `map.0.properties` в `<папку установки агента>/current/user/agent/map`

9) отредактируйте файл `<папку установки агента>/current/user/agent/agent.properties` следующим образом:
- `agents[0].mode=DeleteFile`
- `agents[0].proccessfoldersrecursively=true` 

10) Запустите коннектор и убедитесь, что события поступают
![События](https://user-images.githubusercontent.com/85429798/128209247-c1582fc9-ea2a-4908-9c95-618ac1a097ee.png)


#### Сценарий №2 - Загрузка лог файлов в ArcSight с помощью ВМ, которая находится в Yandex.Cloud

- ручное 
- пререквизиты, что должен быть впн или интерконнект
- через терраформ пример с установкой VPN соединения


## Поддержка/Консалтинговые услуги
Компания сервис партнёр по поддержке – ООО «АТБ» готова оказывать следующие услуги на платной основе:
- Установка и настройка коннектора
- Подключение новых источников данных о событиях безопасности
- Разработка новых правил корреляции и средств визуализации
- Разработка механизмов реагирования на возникающие инциденты

Контактные данные партнёра:
- +7 (499) 648-75-48
- info@ast-security.ru

![image](https://user-images.githubusercontent.com/85429798/128419821-aa2a4c85-7c67-4173-b21b-f0ec6b96e9e3.png)
