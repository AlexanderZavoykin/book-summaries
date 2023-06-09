# Глава 6. Партиционирование.

Цель партиционирования (секционирования) заключается в распределении нагрузки по данным и запросам равномерно по нескольким машинам, 
а также в том, чтобы избежать непропорционально высокой нагрузки на отдельных узлах (*hot spot*).

## Партиционирование данных типа "ключ-значение".
Способы распределения данных по узлам:
- случайное. Распределение равномерное, но при чтении необходимо опрашивать все узлы по порядку на предмет наличия в них искомого ключа.
- по диапазону ключей. Каждому узлу назначается диапазон ключей. Ключи можно хранить в отсортированном виде, а значит делать запросы 
на чтение диапазонов данных. Невдумчивый выбор ключа партиционирования может привести к *hot spots*.
- по хэшу ключа. Распределение равномерное, но нельзя делать запросы на чтение диапазонов данных.
- по составному ключу. Одна часть ключа используется для партиционирования, а другая - для сортировки (становится возможным делать запросы 
на чтение по диапазону по второй части ключа).

## Партиционирование вторичных индексов.
Способы:
- по документам (*by document*). Каждая партиция строит индекс (*local index*) по хранящимся в ней значениям и ничего не знает о данных, 
хранимых на других партициях. Запросы на чтение отправляются на все партиции, а результаты объединяются (поход носит название *scatter/gather*).
- по термам (*by term*). Вторичные индексы являются глобальными (один индекс на все партиции), но аналогично самим данным распределяются по партициям. 
В данном случае чтение более эффективное (клиенту нужно сделать запрос в нужную партицию), но запись медленнее и сложнее (так как запись в один документ 
может затронутиь сразу несколько партиций индекса). При партиционировании по термам обычно применяется асинхронное обновление глобальных вторичных
индексов, поэтому результаты записи могут быть не сразу отражены при чтении.

## Перебалансировка партиций.
При добавлении или удалении узлов требуется перебалансировка - перераспределение данных по партициям. Минимальные требования при перебалансировке:
- после перебалансировки нагрузка (место для хранения, а также запросы на запись и чтение) должны быть распределены равномерно между партициями
- во время перебалансировки база данных должна продолжать принимать запросы на запись и чтение
- для сокращения времени перебалансировки, сетевой и дисковой нагрузки количество перемещаемых данных должно быть минимальным.

Стратегии перебалансировки:
- хэширование по остатку деления на количество партиций (*hash mod N*). Плохой вариант, так как требует слишком много перемещений данных при добавлении
или удалении новых узлов.
- фиксированное количество партиций. На каждом узле создается сразу фиксированное большое количество партиций. При перебалансировке с узла на узел перемещаются
некоторые партиции целиком. Выбранное количество партиций на один узел не должно быть слишком большим (накладные потери на управление).
- динамическое партиционирование. Разделение партиции на несколько новых при превышении размера и слияние партиций в одну большую. Преимущества подхода:
количество партиций всегда соответствует количеству данных.
- партиционирование по количеству узлов. На каждом узле фиксированное количество партиций. Размер партиций растёт с увеличением количества данных, а количество 
узлов остаётся неизменным. При добавлении нового узла размер партиций уменьшается. 

## Маршрутизация запросов.
Существуют следующие подходы по маршрутизации запросов:
- клиент делает запрос к узлу. Если узел не содержит партицию с искомыми данными, он перенаправляет запрос к узлу, обладающему данными, получает от него ответ и 
перенаправляет ответ клиенту.
- все запросы отправляются на звено маршрутизации, которое служит балансировщиком нагрузки и перенаправляет запросы узлам.
- потребовать, чтобы клиенты учитывали партиционирование и привязку партиций к узлам и самостоятельно делали прямые запросы сразу к нужным узлам.


