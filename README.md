Проект # 3
Моделирование изменения балансов студентов.

Что нужно было сделать:

Задача 1. Выберите топ-1000 строк из CTE balances с сортировкой по user_id и dt. Посмотрите на изменения балансов студентов
Задача 2. Создайте визуализацию (линейную диаграмму) итогового результата.
Как решала: было создано несколько запросов CTE и из разных таблиц было найдено отношение транзакций к потраченным урокам, визуализация была сделана в Metabase.

Как решала:
1. Узнала, когда была первая транзакция для каждого студента. 
Создадим CTE `first_payments` с двумя полями: `user_id` и `first_payment_date` (дата первой успешной транзакции). 
2. Собрала таблицу с датами за каждый календарный день 2016 года. Выбрала все даты из таблицы `classes`, создала CTE `all_dates` с полем `dt`, где будут храниться уникальные даты (без времени) уроков. 
3. Узнала, за какие даты имеет смысл собирать баланс для каждого студента. Для этого объединила таблицы и создала CTE `all_dates_by_user`, где будут храниться все даты жизни студента после того, как произошла его первая транзакция. 
4. Нашла все изменения балансов, связанные с успешными транзакциями. Выбрала все транзакции из таблицы `payments`, сгруппировала их по `user_id` и дате транзакции (без времени) и найшла сумму по полю `classes`. В результате получила CTE `payments_by_dates` с полями: `user_id`, `payment_date`, `transaction_balance_change` (сколько уроков было начислено или списано в этот день). 
 5. Нашла баланс студентов, который сформирован только транзакциями. Для этого объединила `all_dates_by_user` и `payments_by_dates` так, чтобы совпадали даты и `user_id`. Использовала оконные выражения (функцию `sum`), чтобы найти кумулятивную сумму по полю `transaction_balance_change` для всех строк до текущей включительно с разбивкой по `user_id` и сортировкой по `dt`. В результате получила CTE `payments_by_dates_cumsum` с полями: `user_id`, `dt`, `transaction_balance_change` — `transaction_balance_change_cs` (кумулятивная сумма по `transaction_balance_change`).
 6. Нашла изменения балансов из-за прохождения уроков. Создала CTE `classes_by_dates`, посчитав в таблице `classes` количество уроков за каждый день для каждого ученика. Получила результат с такими полями: `user_id`, `class_date`, `classes` (количество пройденных в этот день уроков). Причем `classes` умножила на `-1`, чтобы отразить, что `-` — это списания с баланса.
7.По аналогии с уже проделанным шагом для оплат создала CTE для хранения кумулятивной суммы количества пройденных уроков. 
Для этого объединила таблицы `all_dates_by_user` и `classes_by_dates` так, чтобы совпадали даты и `user_id`. В результате получила CTE `classes_by_dates_dates_cumsum`с полями: `user_id`, `dt`, `classes` — `classes_cs`(кумулятивная сумма по `classes`).
8. Создала CTE `balances` с вычисленными балансами каждого студента. Для этого объединила таблицы `payments_by_dates_cumsum` и `classes_by_dates_dates_cumsum` так, чтобы совпадали даты и `user_id`.
9. Посмотрела, как менялось общее количество уроков на балансах студентов. Для этого просуммировала поля `transaction_balance_change`, `transaction_balance_change_cs`, `classes`, `classes_cs`, `balance` из CTE `balances` с группировкой и сортировкой по `dt`.
10. Сбрала код в единый цельный запрос.
11. Сделала визуализацию.

> <a href="https://metabase.sky.pro/question/79661"> Ссылка на проект

Выводы (итоги):

В результате получился запрос, который собирает данные о балансах студентов за каждый "прожитый" ими день. 
На основе полученных данных было выявлено что не везде значения стремятся к 0 (транзакция минус урок) были заданы вопросы к дата инженерам по отрицательным и пустым значениям в таблицах.
На основании построенного графика было видно, что в общем за год идет прирост в количествахвах транзакций и соответственно количество проведенных уроков, но из-за не точных данных линия итого баланса находиться выше оси Х.
