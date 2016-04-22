# school.js
## Для чего?
Библиотека предосталяет простой программный интерфейс для работы с процессом обучения ШРИ.<br>
Для хранения используется IndexedDB, но вам совсем необязательно знать ее особенности.
## Что умеет
* ##### Инициализировать курс

Для работы бибилиотеки необходимо сначало подключить скрипт Dexie.js<br>
`<script src="https://npmcdn.com/dexie/dist/dexie.js"></script>`

```javascript
const shri = new School('Shri-2016-Moscow');
```

* ##### Получать все элементы колекции (students, mentors, tasks)
```javascript
shri
    .getCollection('students')
    .then((result) => {
        // your code
    });
```
* ##### Получать элементы по условию
```javascript
shri
    .getCollectionWhere('students', 'team', 'SuperStars')
    .then((result) => {
        // your code
    });
```

* ##### Создавать студентов, менторов, задания
```javascript
shri
    .getCollection('tasks')
    .then((result) => {
        result.add({name: 'CSS project', type: 'team'});
    });
```

* ##### Обновлять (например выставление оценки группе)
```javascript
const taskId = 1;
shri
    .getCollectionWhere('students', 'team', 'SupreStars')
    .then((result) => {
        result.update({taskId: 5}); // five for all team
    });
```
* ##### Удалять и поддерживать целостность БД
```javascript
shri
    .getCollectionWhere('students', 'id', 2)
    .then((result) => {
        result.remove();
    });
```

* ##### Решать задачу распределения студентов по менторам *(используется алгоритм устойчивых паросочетаний)*

```javascript
shri.transaction(() => {
    shri
        .getCollection('students')
        .then((result) => {
            result.add([[
                {name: 'student1', prefList: [2, 1]}, // Список с предпочтениями,
                {name: 'student2', prefList: [1, 2]}, // где ид с 0 индексом - самй желаемый ментор
                {name: 'student3', prefList: [1, 2]},
                {name: 'student4', prefList: [1, 2]},
                {name: 'student5', prefList: [1, 2]}
            ]]);
        });

    shri
    .getCollection('mentors')
    .then((result) => {
        result.add([[
            {name: 'mentor1', capacity: 1, prefList: [1, 2, 3, 4, 5]},
            {name: 'mentor2', capacity: 2, prefList: [1, 2, 3, 4, 5]}
        ]]);
    });
}).then(() => {
    const resultLists = shri.calcPrefLists(); // К ментору1 попадет студент2,
                                              // а к метору2 попадут студент1 и студент3
});

```
## Документация
https://vchagaev.github.io/shri-task2-school.js/docs/index.html

# Комментарий к заданию
## Проектирование
#### Что же нужно будет пользователю моей библиотеки?
Начнем с самого главного - цель библиотеки. Она должна облегчить разработчику приложения для ШРИ работу с данными. Что бы он не задумывался о целостности данных и
особенностях работы с IndexedDB, а сконцентрировался на логике. Соответственно API должен быть гибким и простым. Я примерно спроектировал имена классов и как бы я хотел с ними работать
будь я разработчиком приложения. Изначально я спроектировал с расширением промисов или стэком операций, но понял, что имплеменитировать качественно я такое не смогу, поэтому решил реализовать используя промисы.
#### Выбор места хранения
Из цели понятно, что придется делать CRUD в браузере. Для такой задачи необходимо хранилище, а так как бэкэнд у нас не подразумевается, то
такое хранилище должно быть в браузере. Из выбора у нас LocalStorage, IndexDB, WebSQL и кэш. LocalStorage для такой задачи совершенно не подходит как и кэш.
WebSQL больше не поддерживается и будущее его не понятно. Из плючов - это SQL а я с ним работал :) IndexedDB NO SQL БД. Исходя из специфики задания, а нам не нужно
поддерживать какую-то сумашедшую целостность данных я выбрал IndexedDB. Примерно спроектировал как у меня все будет храниться. Самая частая операция - выставление оценок
и именно под нее проектировал данные.
#### Выбор инструмента для работы с IndexedDB
Посмотрев стандартный API IndexedDB и сравнив его с API врапперов и их фичами, то решил использовать wrapper. Мне нужен был
простой - для запросов и CRUD действий и простым интерфейсом. Выбор пал на Dexie.js https://github.com/dfahlander/Dexie.js.
Легкая, документируемая, без вербозного API. Как раз то что нужно.
## Тестирование
Для тестирования решил применить Jasmine и Karma. Jasmine - хороший фреймворк, которым я пользовался. Тестировать нужно в боевой среде - браузере. Поэтому Karma - для прогонки тестов в браузере и анализ покрытия кода.
 Вместе хорошо работают + не вижу причин не использовать их или использовать что-то другое. Покрыл бибилиотеку тестами на все основные кейсы.
## Реализация
 При реализации использовал промисы для работы с асинхронностью. В целом старался придерживаться стандарту ECMA 6. Для код-стайла JSCS, а линтер - ESLint.
  Старался придерживаться код стайлу https://github.com/ymaps/codestyle. При реализации решил расширить массив для работы с коллекциями.
  ## Экспорт и импорт
  Экспортировать и импортировать решил использую JSON. В принице с ними ничего сложного, так как IndexedDB в себе хранит объекты.
  ## Документация
Сгенерировал докуменацию на основе JSDoc комментов. Использовал описание классов.
## Веб-интерфейс
Для реализации веб-интерфейса планировал использваоть какой-нибудь MVC-фреймворк (Angular или поразбирался бы с BEM MVC).
По возможности использовал бы Drag & Drop (ранжирование списков предпочтений или распределение студентов по командам)