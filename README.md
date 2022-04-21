Задача 2
# SpotBugs

# Домашнее задание к занятию «Выстраивание процесса непрерывной интеграции (CI): Github Actions. Покрытие кода с JaCoCo, статический анализ кода: CheckStyle, SpotBugs»

В качестве результата пришлите ссылки на ваши GitHub-проекты в личном кабинете студента на сайте [netology.ru](https://netology.ru).

Все задачи этого занятия нужно делать в разных репозиториях.

**Важно**: если у вас что-то не получилось, то оформляйте Issue [по установленным правилам](../report-requirements.md).

Вы можете делать все задачи этого занятия в одном репозитории (если делаете их последовательно).

Напоминалку по некоторым теоретическим моментам в джаве вы можете найти [здесь](../tips/tips.md).

## Как сдавать задачи

1. Инициализируйте на своём компьютере пустой Git-репозиторий
1. Добавьте в него готовый файл [.gitignore](../.gitignore)
1. Добавьте в этот же каталог необходимые файлы
1. Сделайте необходимые коммиты
1. Создайте публичный репозиторий на GitHub и свяжите свой локальный репозиторий с удалённым
1. Сделайте пуш (удостоверьтесь, что ваш код появился на GitHub)
1. Ссылку на ваш проект отправьте в личном кабинете на сайте [netology.ru](https://netology.ru)
1. Задачи, отмеченные, как необязательные, можно не сдавать, это не повлияет на получение зачета (в этом ДЗ все задачи являются обязательными)

## Задача №1 - Синдром 100%

### Легенда

Вы попали в команду максималистов, которые хотят, чтобы те авто-тесты, которые вы пишете, покрывали код на 100%.

Но вот незадача:
1. Непонятно, что такое 100%
2. Непонятно, как это сделать

Вспоминаем: покрытием кода у нас занимается JaCoCo, но он просто "сигнализирует" о том, что конкретно пошло не так.

Большинство подобных плагинов помимо целей отчётности (`report`) содержат ещё цель `check`, которая обрушает сборку, если не выполнены определённые проверки.

Что вам нужно:
1. Изучить [документацию на плагин](https://www.eclemma.org/jacoco/trunk/doc/maven.html) (а конкретно на цель `check`)
1. Внедрить эту цель в фазу `verify` (обратите внимание, что эта цель итак публикуется в эту фазу)
1. Настроить правила по покрытию на 100% (при этом нужно изучить разницу между счётчиками `INSTRUCTION`, `LINE`, `BRANCH`, `COMPLEXITY`)
1. Выбрать один из счётчиков и добиться 100% покрытия тестами

**Важно**: использовать можно только один из следующих: 
1. `INSTRUCTION`
1. `LINE`
1. `BRANCH`

По шагам:
1. Скачиваете [проект с лекции](https://github.com/netology-code/javaqa-code/tree/master/2.5_ci)
1. Мы просили плагин выполнить цель report для генерации отчёта, вам надо по аналогии с ним добавить ещё один execution, только не забыв заменить в нём report на check. Если не получается, см. как в подсказке ниже. 
1. Изучаете документацию, экспериментируете со счётчиками (`INSTRUCTION`, `LINE`, `BRANCH`)
1. Выбираете один из счётчиков (на ваше усмотрение из первых трёх: `INSTRUCTION`, `LINE`, `BRANCH`)
1. Запускаете, удостоверяетесь, что сборка падает, делаете push в GitHub (так, чтобы в логах осталось, что сборка падала в CI)
1. Анализируете отчёт JaCoCo, смотрите что осталось непокрытым 
1. Дописываете тесты, чтобы обеспечить 100% по выбранному счётчику, делаете push в GitHub (так, чтобы в логах осталось, что сборка зелёная в CI)

Итого: у вас должен быть репозиторий на GitHub, в котором расположен ваш Java-код.

<details>
  <summary>Подсказка №1</summary>
  
  Не всегда все плагины хорошо документированы. Достаточно часто плагин просто запускает какой-то инструмент. И именно в документации самого инструмента раскрываются значения параметров.
  
  Также и с JaCoCo. Вы можете найти описание счётчиков и их назначения [на странице документации самого инструмента JaCoCo](https://www.jacoco.org/jacoco/trunk/doc/) (не плагина).
  
</details>

<details>
  <summary>Подсказка №2 (пример настройки)</summary>
  
  ```
  <executions>
    ...
    <execution>
      <id>check</id>
      <goals>
        <goal>check</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <rules>
      <rule>
        <limits>
          <limit>
            <counter>ВАШ COUNTER</counter>
            <value>COVEREDRATIO</value>
            <minimum>100%</minimum>
          </limit>
        </limits>
      </rule>
    </rules>
  </configuration>
  ```
  
</details>

**Важно**: если вы используете Lombok, то покрытие сгенерированных им методов (например, getter/setter'ы тоже учитываются). Это не всегда бывает нужно.

Варианта тут два:
1. Вы читаете документацию на [`excludes`](https://www.jacoco.org/jacoco/trunk/doc/check-mojo.html#excludes), которая позволяет вам исключать целые классы из рассмотрения (не наш вариант)
1. Игнорируете сгенерированные Lombok'ом методы (для этого создайте файл `lombok.config` в корне проекта со следующим содержимым: `lombok.addLombokGeneratedAnnotation = true` и не забудьте сделать `mvn clean`)

## Задача №2 - "Пусть плагин ищет баги"*

**Важно**: это необязательная задача. Её (не)выполнение не влияет на получение зачёта по ДЗ.

### Легенда

[SpotBugs](https://spotbugs.github.io) и [Maven Plugin для него](https://spotbugs.readthedocs.io/en/latest/maven.html) предоставляют возможность производить статический анализ (анализ кода без его запуска) для выявления наиболее часто встречающихся багов.

Список багов, которые ищет SpotBugs: https://spotbugs.readthedocs.io/en/latest/bugDescriptions.html

Ваша задача:
1. Подключить плагин к вашему проекту (возьмите проект с первой задачи, либо создайте новый на базе кода с лекции)
1. Настроить goal `check` в фазу `verify`
1. Удостовериться, что код проходит проверки SpotBugs (если не проходит, то пофиксить)
1. Сделать push в GitHub и удостовериться, что сборка проходит

<details>
  <summary>Подсказка 😈</summary>
  
  Не обязательно каждый новый плагин должен "валить" сборку. С точки зрения некоторых плагинов всё может быть хорошо (если они не нашли ошибок, на которые настроены).
  
</details>

Итого: у вас должен быть репозиторий на GitHub, в котором расположен ваш Java-код.

## Задача №3 - "Внедряем стандарты кодирования"*

**Важно**: это необязательная задача. Её (не)выполнение не влияет на получение зачёта по ДЗ.

### Легенда

[CheckStyle](https://checkstyle.sourceforge.io/) и [Maven Plugin для него](https://maven.apache.org/plugins/maven-checkstyle-plugin/usage.html) предоставляют возможность производить проверки для выявления соответствия стиля написания кода заданным стандартам.

Стандарты в организации формируют ведущие программисты и от организации к организации стандарты могут отличаться.

Мы будем использовать [Google Java Style Guide](https://checkstyle.sourceforge.io/styleguides/google-java-style-20180523/javaguide.html)

Используйте приложенный файл [checkstyle.xml](assets/checkstyle.xml) в качестве набора правил (положите его в корень проекта и укажите в качестве настройки `configLocation`).

Ваша задача:
1. Подключить плагин к вашему проекту (возьмите проект с первой задачи, либо создайте новый на базе кода с лекции)
1. Настроить goal `check` в фазу `verify`
1. Удостовериться, что код не проходит проверки CheckStyle (фиксить не нужно)
1. Сделать push в GitHub и удостовериться, что сборка не проходит именно по причине наличия ошибок CheckStyle

Итого: у вас должен быть репозиторий на GitHub, в котором расположен ваш Java-код.
