# Проектирование высоконагруженной системы Yandex Music
Курсовая работа в рамках 3-го семестра программы по WEB-разработке ОЦ VK x МГТУ им. Н.Э. Баумана по дисциплине "Проектирование высоконагруженных систем".

Работа выполнена в соответствии с [**заданием**](https://github.com/init/highload/blob/main/homework_architecture.md).

## Содержание
- [Тема, целевая аудитория и функционал](#тема-целевая-аудитория-и-функционал)
- [Расчёт нагрузки](#расчёт-нагрузки)
- [Состав участников](#состав-участников-по-проектированию-высоконагруженной-системы)
- [Источники](#источники)

## Тема, целевая аудитория и функционал
В качестве высоконагруженной системы был выбран Yandex Music. 

Это ведущий в России музыкальный стриминговый сервис для поиска и прослушивания музыки с рекомендациями для каждого пользователя. Входит в состав Yandex Plus - единой подписки, открывающей мир множества фильмов и сериалов, миллионов песен и выгодного кэшбэка при пользовании различными сервисами компании.

### Аудитория 
+ В 2019 году количество слушателей Yandex Music составило `3 млн`.[^1] 
  Учитывая события последних 5 лет, оно кратно возросло с уходом зарубежных площадок, прогназируемое DAU - `от 6 до 10 млн`. 
+ MAU порядка `20-25 млн`. 
+ К концу 2022 года время прослушивания выросло `до 29ч/мес`.[^2] 
  Рост был связан с появлением рекомендательной системы "Моя волна". Будем считать время прослушивания неизменным, поскольку времени у людей больше не стало.
+ Время прослушивания в день: `29ч / 30д = 58 мин/д`.
+ Сервисом пользуются люди разных возрастов: от подростков и до взрослых людей (45+).
+ Школьники и студенты чаще заходят на сервис во второй половине дня, когда заканчиваются занятия. Пользователи старше 24 лет, похоже, больше всего слушают музыку на работе: те, кто моложе, активнее всего утром, а их старшие коллеги — в послеобеденные и вечерние часы.[^3]

#### Страны, которым доступна бесплатная версия приложения
+ Россия
+ Беларусь
+ Казахстан
+ Азербайджан
+ Армения
+ Киргизия
+ Молдавия
+ Таджикистан
+ Туркменистан
+ Узбекистан
+ Грузия
+ Израиль

Остальным странам мира, за исключением некоторых, доступна только платная версия.

#### География распространения сервиса
![География распространения сервиса](src/service_geography.png)

### Функционал
#### Ключевой функционал сервиса
1) Авторизация.
2) Поиск треков, альбомов, исполнителей, плейлистов, подкастов, аудиокниг по названию.
3) Рекомендации для пользователя.
4) Загрузка исполнителями треков.
5) Монетизация.

#### Ключевые продуктовые решения
1) Создание пользователем коллекций музыки.
2) Составление чартов песен по категориям.
3) Авторизация в сервисе будет через OAuth2 посредством Yandex ID. 
4) Получение истории прослушиваний. 
5) Добавление песни в Избранное, получение списка избранных песен.
6) Полнотекстовый поиск.
7) Подборка музыки по настроению, языку исполнения, характеру, типу занятия пользователя. 
8) Подборка музыки, рекомендации пользователю на основе LLMs.

## Расчёт нагрузки
### Продуктовые метрики
**MAU** - `25 млн` пользователей

**DAU** - `10 млн` пользователей

#### Средний размер хранилища пользователя
| Хранимые данные                | Занимаемый размер    |
|--------------------------------|----------------------|
| Персональные данные, документы | 1 KB                 |
| Аватарка                       | 300 KB               |
| Связанные данные*              | 3 KB                 |
| Всего                          | 4.3 KB               | 

**Пояснение:** Связанные данные включают в себя историю прослушиваний, лайки, дизлайки, подписки и прочее.

Всего пользователей в сервисе Yandex Music - `25 млн`, тогда занимаемую память под пользователей можно вычислить по следующей формуле: 

$$
  memoryUsage = userCount \times memoryPerUser
$$

#### Средний размер хранилища медиа
| Хранимые данные                | Занимаемый размер    |
|--------------------------------|----------------------|
| Ссылка на медиа                | 0.1 KB               |
| Связанные данные               | 0.5 KB               |
| Всего                          | 0.6 KB               | 

Медиа касается как музыкальных треков, так и подкастов, аудиокниг, обложек, клипов и т.д. 
Yandex Music работает с большим количеством таких данных, поэтому мы будем работать с системами хранения, которые нацелены на работу с такими форматами данных, а в БД будем хранить только ссылки на эти ресурсы, которые и будем возвращать клиенту. 

#### Типы медиа и их размер
| Тип                            | Занимаемый размер    |
|--------------------------------|----------------------|
| Музыкальный трек               | 6.86 MB              |
| Подкаст                        | 1029 MB              |
| Аудиокнига                     | 1372 MB              | 
| Клипы                          | 5.96 MB              | 
| Фото                           | 0.6 KB               | 

**Расчеты:** 
1) Музыкальный трек  

Все треки высокого качества. Предположим, что битрейт был постоянный и равный `320 kb/s`. Средняя продолжительность трека - `3 минуты`, следовательно размер аудиофайла примерно равен:

 `(3 * 60 * 320 * 1000) / (8 * 1024 * 1024) = 6.86 (MB)`. Получается, что средний размер трека - `6.86 MB`.

На сегодняшний день в Yandex Music доступно 74 миллиона музыкальных треков[^4]. 
Их суммарная занимаемая память равна `(74 * 1000 * 1000 * 6.86) / (1024 * 1024) = 484.12 (TB)`.

2) Подкаст (аудио)

Аналогичный расчет, средняя продолжительность эпизода подкаста равняется `30 минутам`[^3]. Проводим рассчеты:

`(30 * 60 * 320 * 1000) / (8 * 1024 * 1024) = 68.6 (MB)`. Получается, что средний размер эпизода подкаста - `68.6 MB`.

В среднем подкаст имеет 15 эпизодов, что оценивается размером в `68.6 * 15 = 1029 (MB)`. В 2021 в каталоге Yandex Music было 11,5 тысячи подкастов[^5]. Количество подкастов тогда росло быстро (по 500 новых проектов ежемесячно), но сейчас рост явно уменьшился, поэтому предположим, что в 2022-2024 прирост подкастов в среднем был по 150 новых проектов. Тогда итоговое количество подкастов `11500 + 3 * 12 * 150 = 16900`. Тогда суммарный объем памяти, занимаемый подкастами, равен `16900 * 1029 / (1024 * 1024) = 16.6 (TB)`.

3) Аудиокнига

Продолжительность главы аудиокниги должна быть от `10 минут` (это общая рекомендация для издателей), возьмем среднее значение в `15 минут`. Получается, что размер главы аудиокниги равен:

`(15 * 60 * 320 * 1000) / (8 * 1024 * 1024) = 34.3 (MB)`. Получается, что средний размер главы аудиокниги - `34.3 MB`.

Сложно сказать, сколько в среднем глав имеет книга, но пусть это будет 40 глав (на самом деле это не только главы, но и подтемы, которые также транслируются в отдельном плеере). Таким образом, размер книги равен `40 * 34.3 = 1372 (MB)`.

В пресс-службе компании заявиили, что количество аудиокниг в Yandex Music выросло в два раза, до `15 тыс`, на момент 2022 года. Пусть количество аудиокниг за 2 года выросло в 1.5 раза, что составило `22.5 тыс`. Тогда объем памяти составит `22500 * 1372 / (1024 * 1024) = 29.4 (TB)`.

4) Клип

Клип - яркий и короткий видеоролик, который крутится в плеере при прослушивании песни популярного исполнителя и не только. Средняя продолжительность такого клипа примерно `10 секунд`. Рассчитаем размер, предположив, что битрейт записи видео был `5 mb/s`. Итоговый размер видео:

 `(5 * 1000 * 1000 * 10) / (8 * 1024 * 1024) = 5.96 (MB)`. Итого, клип на треке в среднем занимает `5.96 MB`.

Количество клипов очень мало, поэтому мы пренебрегаем занимаемой памятью.

5) Фото

Имеет средний размер в `300 KB`.

Пусть каждый третий пользователь создает собственный плейлист и ставит фотографию на него, тогда количество плейлистов будет `8.3 млн`.
Фото есть у пользователей, треков, плейлистов, подкастов, аудиокниг. Посчитаем суммарное количество объектов: `25000000 + 79000000 + 8300000 + 16900 + 22500 = 112339400`. Тогда занимаемая память равна `112339400 * 300 * 1000/ (1024 * 1024 * 1024 * 1024) = 30.65 (TB)`.

В системе примерно `112339400` медиа файлов, тогда средний размер хранилища медиа будет `112339400 * 0.6 * 1000 / (1024 * 1024 * 1024) = 62.77 (GB)`.


#### Среднее количество действий пользователя по типам в день:

| Действие                                                                             | Запросы/день на одного пользователя               |
|--------------------------------------------------------------------------------------|---------------------------------------------------|
| Авторизация                                                                          | 0,032 (1 в месяц)                                 |
| Поиск треков, альбомов, исполнителей, плейлистов, подкастов, аудиокниг по названию.  | 10                                                |
| Открытие рекомендации для пользователя                                               | 1                                                 |

## Состав участников по проектированию высоконагруженной системы
[Лобанов Иван (Я)](https://t.me/cantylv) — Проектировщик высоконагруженной системы. 

[Павел Шипилов](https://vk.com/ytmans) — Преподаватель.

## Источники
[^1]: [Число подписчиков «Яндекс.Музыки» выросло в три раза за полтора года и достигло 3 млн.](https://vc.ru/media/96460-chislo-podpischikov-yandeksmuzyki-vyroslo-v-tri-raza-za-poltora-goda-i-dostiglo-3-mln)

[^2]: [Яндекс.Музыка представил статистический отчет о предпочтениях пользователей в 2022 году.](https://www.rma.ru/news/64709/)

[^3]: [У каждого поколения — своя музыка. Или нет?](https://yandex.ru/company/researches/2016/ya_music_and_age)

[^3]: [«Яндекс» опубликовал исследование популярности подкастов на «Яндекс.Музыке».](https://habr.com/ru/news/591545/)

[^4]: [Transparency Report | Yandex](https://yandex.ru/support/music/ru/rules/transparencyreport)

[^5]: [Подкасты России.](https://yandex.ru/company/researches/2021/podcasts#:~:text=%D0%92%20%D0%BA%D0%B0%D1%82%D0%B0%D0%BB%D0%BE%D0%B3%D0%B5%20%D0%AF%D0%BD%D0%B4%D0%B5%D0%BA%D1%81.,%D0%9C%D1%83%D0%B7%D1%8B%D0%BA%D0%B8%2011%2C5%20%D1%82%D1%8B%D1%81%D1%8F%D1%87%D0%B8%20%D0%BF%D0%BE%D0%B4%D0%BA%D0%B0%D1%81%D1%82%D0%BE%D0%B2.)