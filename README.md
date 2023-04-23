## Задача №1: скоро дедлайн

Случилось то, что обычно случается ближе к дедлайну: никто ничего не успевает и винит во всём остальных.

Разработчикам особо не до вас, им ведь нужно пилить новые фичи, поэтому они подготовили сборку, работающую с СУБД, и даже приложили схему базы данных (см. файл [schema.sql](schema.sql)). Но при этом сказали: «Остальное вам нужно сделать самим, там несложно» 😈

Что вам нужно сделать:
1. Внимательно изучить схему.
1. Создать Docker container на базе MySQL 8, прописать создание базы данных, пользователя, пароля.
1. Запустить SUT ([app-deadline.jar](app-deadline.jar)). Для указания параметров подключения к базе данных можно использовать:
- либо переменные окружения `DB_URL`, `DB_USER`, `DB_PASS`;
- либо указать их через флаги командной строки при запуске: `-P:jdbc.url=...`, `-P:jdbc.user=...`, `-P:jdbc.password=...`. Внимание: при запуске флаги не нужно указывать через запятую. Приложение не использует файл `application.properties` в качестве конфигурации, конфигурационный файл находится внутри JAR-архива;
- либо можете схитрить и попробовать подобрать значения, зашитые в саму SUT.

А дальше выясняется куча забавных вещей 😈 Рекомендуем вам попробовать разобраться самим, но если будет сложно, загляните в подсказку.

### Проблема первая: SUT не стартует

<details>
   <summary>Подсказка</summary>

Проблема: SUT не создаёт самостоятельно таблицы в базе данных.

Поэтому вам нужно сходить на сайт-описание Docker image MySQL и посмотреть, как при инициализации скармливать схему. Будет использоваться технология volumes.
</details>

### Проблема вторая: SUT валится при повторном перезапуске

<details>
   <summary>Подсказка</summary>

Проблема: SUT вставляет в базу данных демо-данные, а поскольку там есть ограничение уникальности, это приводит к ошибкам.

Поэтому вам нужно где-то настроить вычистку данных за SUT.
</details>

### Проблема третья (опционально): пароли

Если вы решите вдруг генерировать пользователей, чтобы под ними тестировать вход в приложение, то не должны удивляться тому, что в базе данных пароль пользователя хранится в зашифрованном виде.

Попытка его записать туда в открытом виде ни к чему хорошему не приведёт.

Настойчивые требования к разработчикам раскрыть алгоритм генерации пароля тоже ни к чему не привели.

Что же делать?

<details>
   <summary>Подсказка</summary>

Если вы внимательно присмотритесь к демо-данным, то они очень, прямо подозрительно похожи на те, что были в одной из предыдущих задач.

Значит, вы можете попробовать использовать уже готовые зашифрованные пароли, зная то, какие они были в незашифрованном виде.
</details>

Если вы добрались до этого шага и всё-таки успешно запустили SUT, вы уже герой.

Но теперь выяснилась забавная информация: разработчики фронтенда поругались с разработчиками бэкенда, и вы можете протестировать только вход в систему.

Внимательно посмотрите, как и куда сохраняются коды генерации в СУБД, и напишите тест, который, взяв информацию из БД о сгенерированном коде, позволит вам протестировать вход в систему через веб-интерфейс.

P.S. Неплохо бы ещё проверить, что при трёхкратном неверном вводе пароля система блокируется.

Итого в результате у вас должно получиться:
* docker-compose.yml*,
* app-deadline.jar,
* schema.sql,
* код ваших автотестов.

Если ваша система не поддерживает Docker, то вам, к сожалению, придётся вручную установить MySQL на свой компьютер и отрабатывать тесты уже на ней. В этом случае положите в репозиторий файлик `README.md`, в котором опишите последовательность действий со скриншотами для установки сервера MySQL и загрузки в него файла `schema.sql`.
