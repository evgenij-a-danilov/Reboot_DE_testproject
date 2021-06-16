# Reboot_DE_testproject

## Лаба 3. Работа с графом социальных связей

## Дедлайн

Четверг, 8 июля, 23:59.

## Задача

* По имеющимся данным о друзьях в соцсетях (estkontakt: 20 000 пользователей) расчитать среднее значение числа Данбара внутри группы людей одной профессии.
* (Доп.задача) Проверить или опровергнуть теорию шести рукопожатий на датасете : для всех возможных пар (`userid1` , `userid2`) построить кратчайшую цепочку связей и убедиться, что нет ни одной цепочки с длиной более 6.

## Описание данных

Имеются следующие входные данные:

* Таблица `users_x_friends` со связями user_id -> friend_id ( пары ([user_id]; [user_id]). Данные выгружены в файл.
   Архив с датасетом будет находиться в HDFS (/data/lab3/var/social_graph.zip). Файл u.users_x_friends содержит все связи, файл u.users — базовые данные по всем пользователем,      u.users_x_messages - краткая история полученных сообщений, файл u.sample - ограниченный набор пар (userid; userid) для использования в дополнительном задании.
* Справочник lab_data.global_dictionary находится в Oracle (используйте данные для входа из вашей домашней работы). 

`!hdfs dfs -ls /data/lab3/var`
* `var` номер варианта проекта

## Результат

Выходной формат файла — json. Пример решения:

```json
{
   "professionid" : 10,
   "profession" : "doctor",
   "danbar_value" : 133.67,
   "dispersion" : 50.28
},

{
   "professionid" : 11,
   "profession" : "teacher",
   "danbar_value" : 284.52,
   "dispersion" : 70.49
},

{
   "professionid" : 12,
   "profession" : "driver",
   "danbar_value" : 90.15,
   "dispersion" : 34.83
}
```

В поле `“profession”` нужно указать для заданного `professionid` нужно указать расшифровку. Расшифровку можно взять из справочника в Oracle. Таблица lab_data.global_dictionary.

В поле `“danbar_value”` нужно указать среднее арифметическое числа Данбара, расчитанное для всех пользователей той же профессии. Алгоритм расчета описан ниже.

В поле `“dispersion”` нужно указать значение дисперсии числа Данбара для данной группы. Формула расчета дисперсии описана ниже.

## Алгоритм расчета числа Данбара

Число Данбара - ограничение на количество постоянных социальных связей, которые человек может поддерживать. 
https://ru.wikipedia.org/wiki/%D0%A7%D0%B8%D1%81%D0%BB%D0%BE_%D0%94%D0%B0%D0%BD%D0%B1%D0%B0%D1%80%D0%B0

В файле u.users_x_messages указана дата первого и последнего сообщения. Структура json-файла users_x_messages

```json
{
   "userid" : 1234,
   "friendid" : 5678,
   "start_date" : "2012-05-12",
   "last_date" : "2016-03-17"
},

{
   "userid" : 1234,
   "friendid" : 4567,
   "start_date" : "2010-11-22",
   "last_date" : "2021-06-10"
},

{
   "userid" : 1235,
   "friendid" : 4567,
   "start_date" : "2008-01-12",
   "last_date" : "2021-06-10"
}
```
Предполагаем, что все друзья, с которыми поддерживается активный диалог, должны учитываться при расчете числа Данбара.
Нужно учитывать всех друзей пользователя, с которыми происходил диалог в течение последних 5 лет и условное время поддержания диалога составляет больше года.
Иными словами должны выполняться условия:

* datediff(last_date, current_date())/365 < 5
* datediff(last_date,start_date)/365 > 1

Далее просто берем среднее арифметическое числа Данбара для всего графа и внутри каждой подгруппы (профессии).

Для вычисления дисперсии используем формулу:

![image](https://user-images.githubusercontent.com/49373421/121674532-d9b24980-caba-11eb-9cdf-3ec373896d7e.png)

https://ru.wikipedia.org/wiki/%D0%94%D0%B8%D1%81%D0%BF%D0%B5%D1%80%D1%81%D0%B8%D1%8F_%D1%81%D0%BB%D1%83%D1%87%D0%B0%D0%B9%D0%BD%D0%BE%D0%B9_%D0%B2%D0%B5%D0%BB%D0%B8%D1%87%D0%B8%D0%BD%D1%8B

## Проверка теории 6 рукопожатий

В качестве дополнительного задания предлагается вариация задачи обхода графа.
Предлагается проверить или опровергнуть теорию о шести рукопожатиях на ограниченном наборе пар ([userX]; [userY]).

Для обхода графа предлагается использовать алгоритм обхода в ширину:
https://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D0%B8%D1%81%D0%BA_%D0%B2_%D1%88%D0%B8%D1%80%D0%B8%D0%BD%D1%83

Для проверки теории используем сэмпл рандомных пар (файл u.sample). 
Находим длину пути в нашем невзвешенном графе для всех пар в сэмпле и сохраняем в виде агрегата.
{[длина пути в графе], [количество пар], [отношение к общему количеству пар]}

```json
{
   "length" : 1,
   "amount_of_pairs" : 0,
   "percent" : 0
},

{
   "length" : 2,
   "amount_of_pairs" : 380,
   "percent" : 0.1
},

{
   "userid" : 1235,
   "amount_of_pairs" : 24280,
   "percent" : 1.5
}
```

## Проверка

Файл с результатами расчета необходимо положить в свою домашнюю директорию на кластере под названием: `project3.json`.
