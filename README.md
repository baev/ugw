# ПОЯСНИТЕЛЬНАЯ ЗАПИСКА

## Введение

Основная цель работы - ???

Во всех реализация xUnit предоставляется базовый набор функций, которые позволяют решать следующие задачи:

* описывать тест как тестовый метод (Test Method);
* описывать ожидаемые результаты внутри тестового метода в форме вызовов методов с утверждением (Assertion Method);
* аггрегировать тесты в наборы, которые могут запускаться с помощью одной команды;
* запускать один или несколько тестов для получения отчета о результатах запуска.

Этого достаточно, если речь идет о модульном тестировании (Unit testing). Но для более высокоуровневых тестов, например, проудктовых (Product testing), этого недостаточно. Такие тесты могут выполняться часы, в них проверяется много утверждений. В рамках стандартной модели xUnit анализировать результаты таких тестов достаточно проблематично. Например, тест, проверяющий наличие логотипа на заглавной странице сайта, со следующим сценарием:

* открываем браузер;
* переходим страницу сайта;
* находим элемент, содержащий логотип;
* сравниваем найденный логотип с образцом;
* закрываем браузер

может не пройти из-за следующих причин:

* не получилось открыть браузер;
* нет подключения к интернету;
* не получилось перейти на страницу сайта;
* нету элемента, содержащего логотип;
* логотип не совпадает с образцом;
* не получилось закрыть браузер.

В рамках модели xUnit мы получим сообщение об ошибке, по которому будет достаточно сложно понять, что произошло. 
А если разбить тест на шаги (Steps) и добавить к результатам теста скришот сайта, то анализ ошибки не займет много времени.

Отсюда возникает следующая задача: разработать систему, позволяющую агрегировать дополнительную информацию о тестах и отображать ее в отчете о результатах тестов. Система не должна контролировать процесс выполнения самих тестов, а должна предоставлять API адаптации уже написанных тестов.

В данной работе будет описан процесс разработки такой системы. 

Во второй главе, на основании анализа различных систем построения отчетов автотестов, а также опыта написания тестирования, сформулированы основные принципы для организации системы. 

В третьей главе приведена подробная архитектура Allure, позволяющая легко интегрироваться с любыми существующими
тестовыми фремворками и расширять имующийся функционал. Подробно описана интеграция новых фремворков, и новых систем сборки.

В заключении дано описание текущего состояния разработки и перспективы
ее развития.

## Глава 1. Постановка задачи

В настоящее время существует не сильно много систем построения отчетов автотестов, и все они заточенны на решение какой-то конкретной задачи. Единого кроссязычного (имеются в ввиду языки программирования) решения нету.

### 1.1. Термины и понятия

В данном разделе описаны термины, используемые других частях представленной работы. При этом смысл многих 
терминов сужен, по сравнению, с их обычным смыслом. Это связано, с тем, что данная работа ориентирована в
первую очередь, разработку системы построения отчетов автотестов. В дальнейшем приведенные термины будут использоваться в указанных значениях, если не оговорено обратное.

#### 1.1.1. Тестирование

##### xUnit 
Под термином "xUnit" подразумевается любой член семейства инфраструктур автоматизации тестов (Test Automation Framework), применяемых для автоматизации созданных вручную сценариев тестов. Для большинства современных языков программирования существует как минимум одна реализация xUnit. Обычно для автоматизации применяется тот же язык, который использовался для написания тестируемой ситстемы. Хотя это не всегда так, использовать подобную стратегию проще, поскольку тесты легко получают доступ к программному интерфейсу тестируемой системы.

Большинство членов xUnit реализованы с использованием объектно-ориентированной парадигмы.

##### Тест 

Метод изучения глубинных процессов деятельности системы, посредством помещения системы в разные ситуации и отслеживание доступных наблюдению изменений в ней

##### Тестирование программного обеспечения

процесс исследования, испытания программного продукта, имеющий две различные цели:

* продемонстрировать разработчикам и заказчикам, что программа соответствует требованиям;
* выявить ситуации, в которых поведение программы является неправильным, нежелательным или не соответствующим спецификации.

##### Автоматическое тестирование программного обеспечения

использует программные средства для выполнения тестов и проверки результатов выполнения, что помогает сократить время тестирования и упростить его процесс.

##### Автотесты

автоматические проверки (тесты) использующиеся для автоматического тестирования КЭП

##### Модульное тестирование (Unit testing)

процесс в программировании, позволяющий проверить на корректность отдельные модули исходного кода программы.

##### Продуктовые тесты

процесс в программировании, позволяющий проверить на корректность продукт ???

##### Шаги (steps, степы)

некоторая часть теста. Является логически законченной единицей, но не может быть выделена в отдельный тест. Часто продуктовые тесты разбиваются на шаги для лучшего понимания происходящего

##### Аттачменты

Допольнительные артифакты, возникающие в процессе выполнения тестов, напрмиер, логи, скриншоты, базы данных etc

##### Аспектно ориентированное программирование

##### Аспекты

##### Тест суит

##### Тест кейс

#### 1.1.1. Сокращения

##### XSLT

##### JVM

#### 1.2. Обоснование актуальности

В последнее время начало появляться большое количество продуктовых тестов. Только в нашем отделе тестирования в Яндексе ежедневно запускается несколько миллионов таких тестов. Анализировать результаты такого количества тестов очень затруднительно. 

#### 1.3. Обзор существующих систем

* surefire. Содержит простую таблицу с именем теста и его статусом. В случае наличия ошибки есть стектрейс. Для больших запусков (20-30 тысяч тесткейсов) даже открыть отчет затруднительно.

* Thucydides - инструмент для быстрого написания тестов с использованием WebDriver. По стути - набстройка над jUnit. Управляет ходом выполнения тестов, по сути является тестовым фремворком, а не фремворком для построения отчетов. Однако, строит отчет. Минусы - слишком много ограничений: только jUnit, только такой синтаксис, только такая логика поведения тестов, слишком много проблем: постоянные проблемы, возникающие из-за того, что это очень большая сложная система. Отчет содержит допольнительную информацию, в отличие от surefire, и есть возможность разбивать тесты на шаги. Но нет возможности добавлять к тесту произвольную информацию. Также не совсем удобное отображение тестов в отчете.

#### 1.4. Технический и организационный контекст

#### 1.5. Уточненные требования к работе

### Глава 2. Теоритические результаты

#### 2.1. Анализ предыдущих разработок

### Глава 3. Проектирование программного продукта

### Заключение

В даннай работе создан фреймворк для генерации отчетов автотестов. Фремворк легко адапатируется к другим системам.

### Список литературы

### Приложения
