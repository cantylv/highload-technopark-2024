# Проектирование высоконагруженной системы
Курсовая работа в рамках 3-го семестра программы по WEB-разработке ОЦ VK x МГТУ им. Н.Э. Баумана по дисциплине "Проектирование высоконагруженных систем".

Работа выполнена в соответствии с [**заданием**](https://github.com/init/highload/blob/main/homework_architecture.md).

## Содержание
- [Тема, целевая аудитория и функционал](#тема-целевая-аудитория-и-функционал)
- [Расчёт нагрузки](#расчет-нагрузки)
- [Состав участников](#состав-участников-по-проектированию-высоконагруженной-системы)
- [Источники](#источники)

## Тема, целевая аудитория и функционал
В качестве высоконагруженной системы был выбран Yandex Music. 
Это ведущий в России музыкальный стриминговый сервис для поиска и прослушивания музыки с рекомендациями для каждого пользователя. Входит в состав Yandex Plus - единой подписки, открывающей мир множества фильмов и сериалов, миллионов песен и выгодного кэшбэка при пользовании различными сервисами компании.

### Аудитория 
Число подписчиков Яндекс Плюса во втором квартале 2024 года достигло `33,7 миллиона`.[^1] В 2019 году количество слушателей Yandex Music составило `3 млн`.[^2] Учитывая события последних 5 лет, оно кратно возросло с уходом зарубежных площадок, прогназируемое количество DAU - `от 6 до 10 млн` в СНГ и не только. Число MAU порядка `20-25 млн`. 

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
4) Монетизация.
5) Загрузка исполнителями треков.

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
| Связанные данные*              | 1 KB                 |
| Всего                          | 1.3 KB               | 

**Пояснение:** Связанные данные включают в себя историю прослушиваний треков, лайки, дизлайки и другое.

#### Средний размер хранилища медиа
| Хранимые данные                | Занимаемый размер    |
|--------------------------------|----------------------|
| Ссылка на медиа                | 0.1 KB               |
| Связанные данные               | 0.5 KB               |
| Всего                          | 0.6 KB               | 

Медиа касается как музыкальных треков, так и подкастов, аудиокниг, обложек, клипов и т.д. 
Yandex Music работает с большим количеством таких данных, поэтому мы будем работать с системами хранения, которые нацелены на работу с такими форматами данных, а в БД будем хранить только ссылки на эти ресурсы, которые и будем возвращать клиенту. 

**Типы медиа и их размер**
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

2) Подкаст (аудио)

Аналогичный расчет, средняя продолжительность эпизода подкаста равняется `30 минутам`[^3]. Проводим рассчеты:

`(30 * 60 * 320 * 1000) / (8 * 1024 * 1024) = 68.6 (MB)`. Получается, что средний размер эпизода подкаста - `68.6 MB`.

В среднем подкаст имеет 15 эпизодов, что оценивается размером в `68.6 * 15 = 1029 (MB)`. 

3) Аудиокнига

Продолжительность главы аудиокниги должна быть от `10 минут` (это общая рекомендация для издателей), возьмем среднее значение в `15 минут`. Получается, что размер главы аудиокниги равен:

`(15 * 60 * 320 * 1000) / (8 * 1024 * 1024) = 34.3 (MB)`. Получается, что средний размер главы аудиокниги - `34.3 MB`.

Сложно сказать, сколько в среднем глав имеет книга, но пусть это будет 40 глав (на самом деле это не только главы, но и подтемы, которые также транслируются в отдельном плеере). Таким образом, размер книги равен `40 * 34.3 = 1372 (MB)`.

4) Клип

Клип - яркий и короткий видеоролик, который крутится в плеере при прослушивании песни популярного исполнителя и не только. Средняя продолжительность такого клипа примерно `10 секунд`. Рассчитаем размер, предположив, что битрейт записи видео был `5 mb/s`. Итоговый размер видео:

 `(5 * 1000 * 1000 * 10) / (8 * 1024 * 1024) = 5.96 (MB)`. Итого, клип на треке в среднем занимает `5.96 MB`.

5) Фото

Имеет средний размер в `300 KB`.

## Состав участников по проектированию высоконагруженной системы
[Лобанов Иван (Я)](https://t.me/cantylv) — Проектировщик высоконагруженной системы. 

[Павел Шипилов](https://vk.com/ytmans) — Преподаватель.

## Источники
[^1]: [ЯНДЕКС объявляет финансовые результаты за II квартал 2024 года.](https://ir.yandex.ru/financial-releases?year=2024&report=q2)

[^2]: [Число подписчиков «Яндекс.Музыки» выросло в три раза за полтора года и достигло 3 млн.](https://vc.ru/media/96460-chislo-podpischikov-yandeksmuzyki-vyroslo-v-tri-raza-za-poltora-goda-i-dostiglo-3-mln)

[^3]: [«Яндекс» опубликовал исследование популярности подкастов на «Яндекс.Музыке»](https://habr.com/ru/news/591545/)