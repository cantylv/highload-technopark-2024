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
3) Подборка музыки по настроению, языку исполнения, характеру, типу занятия пользователя. 
4) Рекомендации для пользователя.
5) Добавление песни в Избранное, получение списка избранных песен.
6) Оформление подписки на артиста.
7) Получение истории прослушиваний.
8) Составление чартов песен по категориям.
9) Настроить подборку музыки на основе песни.
10) Донаты артистам на их цели.
11) Создание пользователем коллекций музыки.
12) Загрузка треков.

#### Ключевые продуктовые решения
1) Авторизация в сервисе будет через OAuth2 посредством Yandex ID. 
1) Полнотекстовый поиск.
2) Подборка музыки, рекомендации пользователю на основе LLMs.

## Расчёт нагрузки
### Продуктовые метрики
**MAU** - `25 млн` пользователей

**DAU** - `10 млн` пользователей

#### Средний размер хранилища пользователя
Поскольку авторизация в Yandex Music работает через OAuth2, то сервис не отвечает за хранение данных о пользователе. 

**Таблица хранилища пользователя**
| Хранимые данные                | Занимаемый размер    |
|--------------------------------|----------------------|
| Персональные данные, документы | 1 KB                 | 
| Аватарка                       | 300 KB               |
| Всего                          | 1.3 KB               |

Структура хранилища:
- <u>Персональные данные</u>: 
  - имя и фамилия
  - номер телефона
  - почта, запасная почта
  - username
  - публичные адреса пользователя (сайты, профиль)
  - пол
  - дата рождения
  - населенный пункт проживания
  - часовой пояс
- <u>Документы</u>: 
  - паспорт РФ
  - загран
  - свидетельство о рождении
  - водителельское удостоверение, СТС/СРТС
  - ОМС, ДМС
  - ИНН, СНИЛС

Это в среднем занимает `1 KB`, так как далеко не каждый пользователь заполняет о себе необязательную информацию.
- <u>Аватарка</u>

В среднем занимает `300 KB`.


## Состав участников по проектированию высоконагруженной системы
[Лобанов Иван (Я)](https://t.me/cantylv) — Проектировщик высоконагруженной системы. 

[Павел Шипилов](https://vk.com/ytmans) — Преподаватель.

## Источники
[^1]: [ЯНДЕКС объявляет финансовые результаты за II квартал 2024 года.](https://ir.yandex.ru/financial-releases?year=2024&report=q2)

[^2]: [Число подписчиков «Яндекс.Музыки» выросло в три раза за полтора года и достигло 3 млн.](https://vc.ru/media/96460-chislo-podpischikov-yandeksmuzyki-vyroslo-v-tri-raza-za-poltora-goda-i-dostiglo-3-mln)