# Задача 1
*Написать SQL запрос, который должен отобрать сделки с бизнес датой события больше, чем 01.04.2021 в статусе VERIFIED и плановые платежи по этим сделкам в валюте RUB, актуальные на эту дату события.*

Начала решение задания с анализа таблиц и их взаимосвязи, с определения ключевых полей и внешних полей.
Определила тип данных для каждого поля. Поле ID в каждой из таблиц не уникально из-за разных значений версий. Поэтому поставила в таблице составной ключ UNIQUE (ID, Version).
Нарисовала модели данных, используя платформу SqlDBM.

*Концептуальная модель данных*

![концептуальная](https://user-images.githubusercontent.com/108063450/192090473-df605d16-f7d4-4dbc-8a47-77fabeb761e8.png)

![логическая](https://user-images.githubusercontent.com/108063450/192090480-48f159ed-e499-4836-b47c-2e650d252976.png)

*Физическая модель данных*

![физическая](https://user-images.githubusercontent.com/108063450/192090499-e91db009-4922-48a6-9f56-4f3d19721e75.png)

## Скрипт на создание таблиц

Создала таблицы в приложении DBeaver (MySQL).

``` js
CREATE TABLE `Trade`
(
 `Trade_ID`       int primary key NOT NULL ,
 `Number`         int NOT NULL ,
 `Status`         varchar(10) NOT NULL ,
 `MurexID`        int NOT NULL ,
 `DealDate`       datetime NOT NULL ,
 `CounterPartyID` int NOT NULL ,
 `ActualDate`     datetime NOT NULL ,
 `FixDate`        datetime NOT NULL ,
 `PremiumType`    varchar(10) NOT NULL ,
 `SettlementType` varchar(10) NOT NULL ,
 `Version`        int NOT NULL ,
 `Product_fk`     int NOT null,
 CONSTRAINT PK_Trade_ID unique (`Trade_ID`,`Version`)
 );
 ```
 
 Результат
 
 ![table trade](https://user-images.githubusercontent.com/108063450/192090675-78d541c2-1a9b-44fd-b448-231d1ad8c935.png)


``` js
CREATE TABLE `Fee`
(
 `Fee_ID`         int  primary key NOT NULL ,
 `Notional`       decimal(8,2) NOT NULL ,
 `Trade_fk`       int NOT NULL ,
 `Product_fk`     int NOT NULL ,
 `Anount`         decimal(8,2) NOT NULL ,
 `ValueDate`      datetime NOT NULL ,
 `FeeType`        varchar(15) NOT NULL ,
 `CounterPartyID` int NOT NULL ,
 `Currency`       varchar(3) NOT NULL ,
 `Input`          decimal(8,2) NOT NULL ,
 `Version`        int NOT NULL ,
 CONSTRAINT PK_Fee_ID unique (`Fee_ID`,`Version`),
 foreign key (`Trade_fk`) references Trade (Trade_ID)
 );
 ```
 
Результат
 ![table fee](https://user-images.githubusercontent.com/108063450/192090691-83bce71e-65e0-4309-9034-e917e0920dab.png)


``` js
CREATE TABLE `Cashflow`
(
 `Cashflow_ID`    int primary key NOT NULL ,
 `Notional`       decimal(8,2) NOT NULL ,
 `Trade_ID`       int NOT NULL ,
 `Product_fk`     int NOT NULL ,
 `Anount`         decimal(8,2) NOT NULL ,
 `ValueDate`      datetime NOT NULL ,
 `TransferType`   varchar(9) NOT NULL ,
 `CounterPartyID` int NOT NULL ,
 `Currency`       varchar(3) NOT NULL ,
 `Version`        int NOT NULL ,
 `Rate`           decimal(5,2) NOT NULL ,
 CONSTRAINT PK_Cashflow_ID unique (`Cashflow_ID`,`Version`),
 foreign key (`Trade_ID`) references Trade (Trade_ID)
 );
 ```
 
 Результат
 
 ![table cashflow](https://user-images.githubusercontent.com/108063450/192090712-de4b33c8-cb58-49b6-8e57-7bb5b3180be3.png)


## SQL-запрос на решение задачи 1

``` js
SELECT DISTINCT Number AS 'Номер_сделки', DealDate AS 'Дата_заключения_сделки', 
ActualDate AS 'Бизнес_дата_события', Anount AS 'Сумма планового платежа', Currency AS 'Валюта_планового_платежа', ValueDate AS 'Плановая_дата_исполнения_платежа'
FROM Trade T INNER JOIN Cashflow C 
ON T.Trade_ID=C.Trade_ID
WHERE ActualDate>'2021-04-01 00:00:00' AND Status='VERIFIED' AND Currency='RUB'
ORDER BY Number ASC, ValueDate ASC;
```

Результат
![Запрос 1](https://user-images.githubusercontent.com/108063450/192090798-c8e9b6a1-69d4-4324-8e49-25496353f2ac.png)



# Задача 2

*Определить статус контрагентов на 03 февраля 2021г.*

Создала таблицу Counterparty и заполнила её. Максимальное количество символов в столбце Counterparty_id нашла через функцию =ДЛСТР().

``` js
create table Counterparty (
Counterparty_id varchar(18),
status varchar(9),
valid_from datetime,
valid_to datetime
);
``` 

``` js
INSERT INTO Counterparty (Counterparty_id, status, valid_from, valid_to) VALUES ('2356aa563bb5874cc','active','2021-02-01 00:00:00','2021-02-03 00:00:00');
INSERT INTO Counterparty (Counterparty_id, status, valid_from, valid_to) VALUES ('2356aa563bb5874cc','defaulter','2021-02-03 00:00:00','2021-02-06 00:00:00');
INSERT INTO Counterparty (Counterparty_id, status, valid_from, valid_to) VALUES ('2356aa563bb5874cc','removed','2021-02-06 00:00:00',null);
...
``` 

Результат

![table counterparty](https://user-images.githubusercontent.com/108063450/192091276-9c2d223d-98a0-4938-a337-19abea5151a4.png)

## SQL-запрос на решение задачи 2

``` js
SELECT Counterparty_id, status
FROM Counterparty
WHERE valid_from<='2021-02-03 00:00:00' AND coalesce(valid_to, '2100-01-01')>'2021-02-03 00:00:00';
```

Результат

![2](https://user-images.githubusercontent.com/108063450/192117894-20df5338-f018-48a9-b2da-e905d27068cb.png)



# Задача 3

*Выбрать самую актуальную запись для каждого экземпляра таблицы*

Создадим таблицу Counterparty_new и заполним её.

Результат

![3](https://user-images.githubusercontent.com/108063450/192118491-4a1c257f-5fb2-4139-8450-d84d0de76ee6.png)

## SQL-запрос на решение задачи 3

``` js
SELECT Counterparty_id, name, address, version
FROM Counterparty_new
WHERE version IN 
    (SELECT MAX(version)
    FROM Counterparty_new
    GROUP BY Counterparty_id);
```

Результат

![3 1](https://user-images.githubusercontent.com/108063450/192141358-eb5575c6-d895-4b88-9802-6dafd16efe82.png)










 
