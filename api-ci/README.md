# Домашнее задание к занятию «1.2. Тестирование API, CI»

В качестве результата пришлите ссылку на ваш GitHub-проект в личном кабинете студента на сайте [netology.ru](https://netology.ru).

Первые две задачи этого занятия нужно делать в одном репозитории.

**Важно**: если у вас что-то не получилось, то оформляйте Issue [по установленным правилам](../report-requirements.md).

**Важно**: не делайте ДЗ всех занятий в одном репозитории! Иначе вам потом придётся достаточно сложно подключать системы Continuous Integration.

## Как сдавать задачи

1. Инициализируйте на своём компьютере пустой Git-репозиторий
1. Добавьте в него готовый файл [.gitignore](../.gitignore)
1. Добавьте в этот же каталог код вашего приложения
1. Сделайте необходимые коммиты
1. Создайте публичный репозиторий на GitHub и свяжите свой локальный репозиторий с удалённым
1. Сделайте пуш (удостоверьтесь, что ваш код появился на GitHub)
1. Ссылку на ваш проект отправьте в личном кабинете на сайте [netology.ru](https://netology.ru)
1. Задачи, отмеченные, как необязательные, можно не сдавать, это не повлияет на получение зачета

В качестве примера можете посмотреть на этот проект (ваш по структуре должен выглядеть так же): https://github.com/netology-code-samples/aqa-ci-demo

## Информация по учебным JAR

Важно: если вы работаете на Windows с Турецкой локалью и при запуске учебных приложений получаете Exception вида:
```
Caused by: java.lang.NoSuchFieldException: wrıteHandlerReference (тут не английская i, а именно ı или ?)
  at java.lang.Class.getDeclaredField(Unknown Source)
  at java.util.concurrent.atomic.AtomicReferenceFieldUpdater$AtomicReferenceFieldUpdaterImpl$1.run(Unknown Source)
  at java.util.concurrent.atomic.AtomicReferenceFieldUpdater$AtomicReferenceFieldUpdaterImpl$1.run(Unknown Source)
  at java.security.AccessController.doPrivileged(Native Method)
  ... 21 more
```

Тогда вам все JAR'ники нужно будет запускать командой:
```
java -Duser.language=en -Duser.country=US -jar app-mbank.jar
```

Часть `-jar app-mbank.jar` может меняться, но первая часть для вас во всех ДЗ (при запуске на вашем ПК будет именно такой).

## Задача №1 - Настройка CI

Напоминаем, CI - это чаще всего отдельная система (сервер, набор серверов, облако), в котором ваш код и ваши авто-тесты собираются в автоматическом режиме (без вашего непосредственного участия). Вы лишь настраиваете CI для того, чтобы при возникновении определённых событий (например, push в репозиторий) стартовал процесс сборки и прогона тестов.

Детальнее про CI вы можете узнать погуглив "Continuous Integration", "Jenkins", "GitLab CI", "Appveyor", "Travis", "Circle CI", "GitHub Actions".

Что надо сделать: берёте проект [с лекции](https://github.com/netology-code/aqa-code/tree/master/api-ci/rest), настраиваете для него CI (см. инструкцию ниже). Удостоверяетесь, что CI показывает, что сборка падает (в процессе сборки будут автоматически прогоняться все авто-тесты).

**Важно**: иногда можно настроить CI так, что там **всегда будет SUCCESS** 😈! Не забывайте убедиться, что в CI сборка действительно падает, если вы запушите в GitHub падающий тест.

<details>
  <summary>Подсказка</summary>
  
  Возможно, это как-то связано с файлом gradlew и правами доступа на него. Для добавления прав на запуск файла gradlew, добавьте в CI исполнение команды `chmod +x gradlew` перед тем как использовать этот файл как команду для работы с гредлом. 
</details>

Общая схема работы выглядит следующим образом: CI должен запустить целевой сервис в фоновом режиме (который вы и тестируете) и ваши авто-тесты. Для этого мы будем на этот раз использовать возможности Bash.

Для того, чтобы запустить целевой сервис есть несколько вариантов, самый простой из которых - положить jar-файл прямо в ваш репозиторий. Когда AppVeyor будет выкачивать исходники авто-тестов, он выкачает и ваш сервис.

Конечно, вы должны понимать, что в реальной жизни артефакты (собранный целевой сервис) хранятся в специальных системах и процесс выкачивания будет зависеть от того, где и как хранится артефакт.

Ваш целевой сервис (SUT - System under test), расположен в файле [app-mbank.jar](app-mbank.jar) (этот же файл используется в примерах на лекции). Вам нужно его положить в каталог `artifacts` вашего проекта (создайте его).

Поскольку файлы с расширением `.jar` находятся в списках `.gitignore`б вам нужно принудительно заставить git следить за ними: `git add -f artifacts/app-mbank.jar`.

После чего сделать `git push`. Обязательно удостоверьтесь, что файл попал в репозиторий.

### AppVeyor

[AppVeyor](https://www.appveyor.com) - одна из платформ, предоставляющих функциональность Continuous Integration. В базовом варианте - бесплатна.

#### Шаг 0. Конфигурация как код

Поскольку вручную настраивать каждый проект в системе Continuous Integration - лишняя трата времени, мы будем хранить всю конфигурацию для AppVeyor в специальном файле с названием `.appveyor.yml`.

Важно: внимательно посмотрите на структуру демо-репозитория из ваших лекций. Большинство инструментов используют подход "Configuration by exception" - т.е. конфигурируется только то, что не соответствует настройкам по умолчанию. Поэтому у вас всего два пути - либо использовать настройки по умолчанию и писать как можно меньше конфигурации, либо "идти против системы" и писать много конфигурации (а потом ещё и отлаживать её).

Файл этот должен храниться в самом репозитории на GitHub, тогда AppVeyor будет автоматически подхватывать настройки из него:

![](https://i.imgur.com/Gg7B961.png)

Yaml - формат данных, используемый многими системами для хранения конфигурации.

Ссылки:
* [Wikipedia](https://en.wikipedia.org/wiki/YAML)
* [Спецификация](https://yaml.org/spec/1.2/spec.html)

Странички на Wikipedia достаточно для понимания базовых конструкций языка.

AppVeyor предлагает вам два вида серверов, на которых можно проводить сборку вашего приложения: под управлением Windows или под управлением Linux. Можно организовать сборку под несколькими сразу, но для упрощения мы пока остановимся только на одной ОС для каждого вашего проекта.

##### Linux Config

```yaml
image: Ubuntu  # образ для сборки

stack: jdk 11  # версия JDK

branches:
  only:
    - master  # ветка git

build: off  # будем использовать свой скрипт сборки

install:
  # запускаем SUT (& означает, что в фоновом режиме - не блокируем терминал для запуска тестов)
  - java -jar ./artifacts/app-mbank.jar &

build_script:
  - ./gradlew test --info  # запускаем тест, флаг --info позволяет выводить больше информации
```

Естественно, у вас должен возникнуть вопрос, а что будет, если SUT не успеет стартовать к моменту запуска авто-тестов?

Тогда ваши тесты упадут. Что с этим делать и как классифицировать подобные случаи, мы поговорим на следующих лекциях.

Напоминаем, ваш `build.gradle` должен выглядеть вот так:
```groovy
plugins {
    id 'java'
}

group 'ru.netology'
version '1.0-SNAPSHOT'

sourceCompatibility = 11
compileJava.options.encoding = 'UTF-8'
compileTestJava.options.encoding = 'UTF-8'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'io.rest-assured:rest-assured:4.3.0'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.6.1'
    testImplementation 'io.rest-assured:json-schema-validator:4.3.1'
}

test {
    useJUnitPlatform()
}
```

#### Шаг 1. Регистрация

![](https://i.imgur.com/Rugmz7D.png)

#### Шаг 2. Регистрация через GitHub

AppVeyor предоставляет бесплатный тарифный план для публичных репозиториев GitHub (авторизация - также через GitHub):

![](https://i.imgur.com/jXvftMb.png)

#### Шаг 3. Разрешение доступа

При подключении необходимо разрешить AppVeyor получать уведомления:

![](https://i.imgur.com/2Fvcj96.png)

#### Шаг 4. Создание проекта

После авторизации станет доступной панель управления, где можно создать новый проект:

![](https://i.imgur.com/wUBKbYY.png)


Авторизуйте AppVeyor в качестве OAuth App:

![](https://i.imgur.com/oQadLLj.png)

Это даст возможность приложению получать уведомления о ваших `push` в репозиторий, модификации и т.д.

![](https://i.imgur.com/2jwH6Sa.png)

Детальнее об OAuth вы можете прочитать на:
* https://oauth.net/2/
* https://auth0.com/docs/protocols/oauth2

#### Шаг 5. Выбор репозитория

После авторизации достаточно будет нажать кнопку `ADD` напротив необходимого репозитория:

![](https://i.imgur.com/4VQME6j.png)


После настройки всего процесса каждый `push` в ветку `master` GitHub-репозитория будет приводить к запуску сборки на AppVeyor.

#### Шаг 6. Status Badge

На странице `Settings` - `Badges` AppVeyor предлагает код для "бейджика" статуса вашего проекта:

![](https://i.imgur.com/DECtZjg.png)


Этот badge необходимо разместить в файле `README.md` для отображения текущего статуса вашего проекта:

![](https://i.imgur.com/V9cOeJO.png)

**Важно: убедитесь, что вы не скопировали бейджик с другого проекта! За такую "хитрость" ДЗ будет отправляться на доработку!**

## Задача №2 - JSON Schema

JSON Schema предлагает нам инструмент валидации JSON-документов. С описанием вы можете познакомиться по этому адресу: https://json-schema.org/understanding-json-schema/index.html

Как строится схема: 
```js
{
  "$schema": "http://json-schema.org/draft-07/schema", // версия схемы: https://json-schema.org/understanding-json-schema/reference/schema.html
  "type": "array", // тип корневого элемента: https://json-schema.org/understanding-json-schema/reference/type.html
  "items": { // какие элементы допустимы внутри массива: https://json-schema.org/understanding-json-schema/reference/array.html#items
    "type": "object", // должны быть объектами: https://json-schema.org/understanding-json-schema/reference/object.html
    "required": [ // должны содержать следующие поля: https://json-schema.org/understanding-json-schema/reference/object.html#required-properties
      "id",
      "name",
      "number",
      "balance",
      "currency"
    ],
    "additionalProperties": false, // дополнительных полей быть не должно 
    "properties": { // описание полей: https://json-schema.org/understanding-json-schema/reference/object.html#properties
      "id": {
        "type": "integer" // целое число: https://json-schema.org/understanding-json-schema/reference/numeric.html#integer
      },
      "name": {
        "type": "string", // строка: https://json-schema.org/understanding-json-schema/reference/string.html
        "minLength": 1 // минимальная длина - 1: https://json-schema.org/understanding-json-schema/reference/string.html#length
      },
      "number": {
        "type": "string", // строка: https://json-schema.org/understanding-json-schema/reference/string.html
        "pattern": "^•• \\d{4}$" // соответствует регулярному выражению: https://json-schema.org/understanding-json-schema/reference/string.html#regular-expressions
      },
      "balance": {
        "type": "integer" // целое число: https://json-schema.org/understanding-json-schema/reference/numeric.html#integer
      },
      "currency": {
        "type": "string" // строка: https://json-schema.org/understanding-json-schema/reference/string.html
      }
    }
  }
}
```

Что нужно сделать:

#### Шаг 1. Добавить зависимость

```groovy
dependencies {
    testImplementation 'io.rest-assured:rest-assured:4.3.0'
    testImplementation 'io.rest-assured:json-schema-validator:4.3.0'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.6.1'
}
```

#### Шаг 2. Сохраните схему в ресурсах

Создайте каталог `resources` в `src/test` и поместите туда схему (не забудьте удалить комментарии):

![](pic/schema.png)

#### Шаг 3. Включить проверку схемы

Модифицируйте существующий тест так, чтобы он проверял соответствие схеме. Для этого:

```java
      // код теста
      .then()
          .statusCode(200)
          // static import для JsonSchemaValidator.matchesJsonSchemaInClasspath
          .body(matchesJsonSchemaInClasspath("accounts.schema.json"))
      ;
```

Удостоверьтесь, что тесты проходят при соответствии ответа схеме и падают, если вы поменяете что-то в схеме (например, тип для `id`)

#### Шаг 4. Доработать схему

Изучите документацию на тип [`object`](https://json-schema.org/understanding-json-schema/reference/object.html) и найдите способ валидации значения поля на два из возможных значения: "RUB" или "USD".

Доработайте схему соответствующим образом, удостоверьтесь, что тесты проходят (в том числе в CI).

Поменяйте "RUB" на "RUR" и удостоверьтесь, что тесты падают (в том числе в CI).

Пришлите на проверку ссылку на ваш репозиторий (удостоверьтесь, что в истории сборки были как Success, так и Fail, иначе будет не видно, как вы проверяли, что сборка падает в CI).

## Задача №3 - Postman Echo

**Важно**: эту задачу нужно выполнять в отдельном репозитории

В этой задаче мы сэмулируем ситуацию, в которой SUT уже запущен, а мы из теста просто обращаемся к нему.

Есть специальный сервис, предназначенный для тестирования HTTP-запросов. Называется он [Postman Echo](https://docs.postman-echo.com) (никогда не тестируйте автоматизированными средствами веб-сервисы, если у вас нет на этого письменного разрешения либо веб-сервисы специально не предназначены для этого).

Мы можем отправлять туда запросы и получать ответы.

С GET-запросами мы немного потренировались, теперь нас будут интересовать POST-запросы, а именно отправка тела запроса:

```java
// Given - When - Then
// Предусловия
given()
  .baseUri("https://postman-echo.com")
  .body("some data") // отправляемые данные (заголовки и query можно выставлять аналогично)
// Выполняемые действия
.when()
  .post("/post")
// Проверки
.then()
  .statusCode(200)
  .body(/* --> ваша проверка здесь <-- */)
;
```

Что нужно сделать:
1. Создайте новый проект на базе Gradle
2. Добавьте необходимые зависимости (если вы не пишите схему, то только rest-assured)
3. Напишите тест, взяв сам запрос из кода выше
4. Изучите ответ и напишите JsonPath-выражение вместо строк `/* --> ваша проверка здесь <--*/`, которое проверит, что в нужном поле хранятся отправленные вами данные (обратите внимание, теперь у вас не массив, а объект).

Можете воспользоваться сервисом https://rapathevaluate.herokuapp.com для быстрой проверки своих JsonPath-выражений.

Удостоверьтесь, что если вы будете использовать неверное выражение, то тесты упадут (в том числе и в CI).

Обратите внимание: если вам приходит вот такой объект:
```json
{
    "data": "some value"
}
```

То обратиться к нему с помощью JsonPath можно вот так: `data`, например: `.body("data", equalTo("some value"))`. Т.е. обращение к полю верхнеуровневого объекта (он называется безымянный) идёт без точки (в примере на лекциях у нас был массив и мы сразу обращались `[0]` - то же самое).

Если соберётесь отправлять текст не на латинице, то вам нужно будет выставлять кодировку (например, UTF-8):
```java
given()
  .baseUri("https://postman-echo.com")
  .contentType("text/plain; charset=UTF-8")
  .body("some data")
.when()
  .post("/post")
.then()
  .statusCode(200)
  .body(/* --> ваша проверка здесь <-- */)
;
```

Пришлите на проверку ссылку на ваш репозиторий (удостоверьтесь, что в истории сборки были как Success, так и Fail, иначе будет не видно, как вы проверяли, что сборка падает в CI).
