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


Создала и заполнила таблицы Trade, Fee и Cashflow в приложении DBeaver (MySQL).

![trade](https://user-images.githubusercontent.com/108063450/192942722-28e47034-b84b-4ee2-88d1-bdb69c025d52.png)

![Fee](https://user-images.githubusercontent.com/108063450/192942747-cf584abc-36cd-4743-b7a2-3572552a195e.png)

![cashflow](https://user-images.githubusercontent.com/108063450/192942770-978b872d-e24d-45f8-8ed3-33affe1d7169.png)



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

### БЫЛО

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

### ИСПРАВЛЕНО

``` js
select Counterparty_id, name, address, MAX(version)
from Counterparty_new
group by Counterparty_id;
```

Результат

![3 2](https://user-images.githubusercontent.com/108063450/192512661-18429544-7f43-41cd-a51f-1f1acdfcac15.png)



 
