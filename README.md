
<h1>Технический отчёт</h1>
<h3>Айсберг</h3>

<div style="page-break-after: always;"></div>

## Содержание

- [О проекте](#о-проекте)
- [Фронтенд](#фронтенд)
    - [Архитектура](#архитектура)
    - [Cordova](#cordova)
    - [React](#react)
    - [PropTypes](#proptypes)
    - [Redux](#redux)
    - [Сборка](#Сборка)
    - [Линтеры](#линтеры)
    - [Тесты](#тесты)
    - [Тестирование на девайсах](#тестирование-на-девайсах)
    - [Разное](#разное)
- [Бэкенд](#бэкенд)
    - [Node.JS и Express](#nodejs-и-express)
    - [Heroku](#heroku)
    - [Apiary и Dredd](#apiary-и-dredd)
    - [Travis CI](#travis-ci)
- [Коммуникация и командная работа](#коммуникация-и-командная-работа)
    - [Взаимодействие в команде](#взаимодействие-в-команде)
    - [Коммуникация в команде разработчиков](#коммуникация-в-команде-разработчиков)
- [Ссылки](#ссылки)

<div style="page-break-after: always;"></div>

## О проекте

Яндекс.Айсберг – это сервис с подборками ссылок по различным темам, где можно удобно читать авторские подборки,а также составлять свои, делиться опытом с другими.

Нашей команде была поставлена задача разработать мобильное приложение (Android + iOS) со следующим функционалом:

- Лента созданных подборок, которая подстраивается под пользователя;
- Отображение детальной информации о созданной подборке и список ссылок в подборке;
- Поиск по подборкам;
- Возможность логиниться в приложении через соцсети;
- Возможность создавать собственные подборки и добавлять в них ссылки;
- Просмотр своего профиля с подробной информацией о добавленных подборках и ссылках, просмотр профилей других пользователей.

<div style="page-break-after: always;"></div>

## Фронтенд

### Архитектура

На старте проекта мы решили, что будем придерживаться правил методологии БЭМ в плане организации структуры папок и наименования классов, но со своими доработками:

- Дополнительная сущность `Pages` для крупных блоков-экранов;
- Отдельно висящие `Elements` как просто мелкие компоненты, не привязанные ни к каким блокам.

Этот подход с ростом проекта приводил к затруднениям, мы начали путаться, чем будет являться новый компонент, появлялась странная вложенность блоков. Мы обсудили ситуацию с кураторами и провели два крупных рефакторинга архитектуры, стараясь следовать каноническим правилам `БЭМ` без наших усложнений. Это себя оправдало и облегчило нам работу. Жить стало проще, появилось время по пятницам ходить в бар.

### Cordova

В качестве платформы для разработки мобильного приложения мы использовали `Cordova`. В первую очередь этот выбор обусловлен простотой разработки и большим количеством плагинов, предоставляющих нативный функционал. Вот несколько из них:

#### cordova-plugin-safariviewcontroller

Этот плагин позволяет просматривать веб-страницы внутри приложения. Он был выбран как альтернатива похожему по функционалу `cordova-plugin-inappbrowser` по ряду причин:

- Использует `Safari Webview` главного браузера системы на iOS, который быстрей, чем `UIWebView`;
- Это рекомендумый Apple способ для таких ситуаций;
- Поддерживает Android OS.

#### cordova-plugin-file

Мы столкнулись с проблемой отображения изображений при билде на реальные девайсы – они просто не загружались из локальной директории устройства. Долго думали и пытались понять, почему так происходит. Оказалось, что невозможно получить файл с помощью относительного пути в коде – нужно использовать абсолютный.

Чтобы решить эту проблему, мы подключили данный плагин, который позволяет работать с файловой системой устройства. С его помощью мы смогли получить путь к директории, где хранятся все файлы приложения, которые использовались в нашем коде.

#### cordova-plugin-camera

Плагин позволяет получить доступ к библиотеке изображений на телефоне. Мы используем его для загрузки фотографий в качестве обложки для карточки коллекций.

Изначально изображения отправлялись “сырыми” на сервер и там обрабатывались библиотекой: сжимались и обрезались до нужного размера. При таком подходе мы столкнулись с проблемой, что крупные картинки требовали большой объем памяти для этой обработки. Это превышало ограничения бесплатного тарифа сервиса `Heroku`, и операция завершалась ошибкой.

Мы стали искать решение этой проблемы и более детально изучили `cordova-plugin-camera`. Выяснили, что настройки плагина позволяют сжимать и изменять разрешение изображения на устройстве перед отправкой их на сервер. Причем происходит это очень быстро.

#### cordova-plugin-customurlscheme

Для реализации авторизации в наше приложение нам была необходима пользовательская `URL` схема, для её создания мы использовали данный плагин.

### React

Основой фронтенд-стека был выбран `React`. Причин у этого выбора несколько:

- Компонентный подход данной библиотеки гибок и позволяет удобно и эффективно решать поставленную задачу.
- Большая часть команды была уже хотя бы частично знакома с `React`, что позволило не тратить время на выбор компромисса и изучение новой технологии.
- Комьюнити `React` огромно, а это значит, что на большинство вопросов, возникающих в процессе разработки, можно найти ответ в статьях, туторилах или на специальных ресурсах.

Для навигации между экранами и логики переключения мы использовали `React Router v4`.

### PropTypes

Осознавая, что с ростом приложения будет сложно контролировать его изменения и вообще правильную работу всей системы, мы задумались о внедрении проверки типов в компонентах. Это можно было сделать несколькими способами, но мы остановились на `PropTypes`. Во-первых, библиотека являлась частью самого `React` (сейчас ставится отдельным пакетом). Во-вторых, проста в использовании, при этом отлично выполняет поставленную задачу. Ну и в целом такой подход приводит к самодокументируемости кода.

### Redux

При выборе модели данных мы обратили внимание на `Redux`. Этот выбор объясняется желанием иметь централизованное хранилище, с помощью которого удобно отслеживать потоки данных в приложении, а также желанием упростить взаимодействие с сервером. В качестве middleware для `Redux` мы используем `Thunk`.

При разработке и отлаживании проекта очень помог `Redux Devtools`. С помощью этого инструмента мы быстро отлаживали и тестировали модель данных, искали баги.

Для синхронизации роутинга со стором `Redux`, мы решили использовать `react-router-redux`. Но столкнулись с проблемой его работоспособности в нашем приложении. Разбираясь в причинах, мы выяснили, что 4-ая версия этой библиотеки не совместима с `react-router v4`. Обсудив ситуацию, мы ~~написали свой роутер~~ решили отказаться от `react-router-redux`, чтобы не даунгрейдить версию основного роутера.

### Сборка

Собирали проект с помощью `Webpack`. Конфиг писали сами, с нуля, учитывая особенности разработки гибридного приложения и предпочтения команды. Конфигурация разделена на два файла – production и development.

Для удобства и повышения скорости разработки в development режиме мы использовали `WebpackDevServer` и `HotModuleReplacementPlugin`. Сервер при старте билдит `index.html` (необходимо для того, чтобы Cordova собрала проект), а после следит за изменениями в папке `src` и в случае необходимости обновляет бандл `bundle.js`.

В loaders у нас находятся:

- `babel-loader` для транспиляции React и ES6+ синтаксиса;
- `style-loader`, `css-loader` и `sass-loader` для преобразования SASS в CSS и последующей сборки в `bundle.css`;
- `file-loader` для шрифтов, SVG-иконок и прочего;
- `eslint-loader` для линтинга.

Для удобства отладки в процессе разработки мы настроили `Source Map`.

### Линтеры

В работе над проектом мы использовали `ESLint` c пресетом от Airbnb. Часть правил конфигурации мы переписали в соответствии со своими предпочтениями.

Также в проекте использовался `Stylelint` с пресетом от HTML Academy.

### Тесты

В качестве инструмента для тестирования мы выбрали `Jest`. Из-за нехватки времени мы не смогли покрыть тестами весь код, а ограничились написанием тестов для редьюсеров.

### Тестирование на девайсах

В процессе разработки мы использовали эмуляторы устройств, а также тестировали приложение на различных реальных девайсах. Среди них были iPhone 5-7, устройства на Android с версиями 5-8.

### CI

Мы используем continuous integration (`Travis CI`) для автоматического запуска тестов перед каждым пулл реквестом.

### Сборка спрайта с svg иконками:

В дизайне нашего продукта используется значительное число `svg` иконок, соответственно, возник вопрос, как лучше загружать иконки в компонент. Ниже о вариантах, которые мы рассмотрели.

#### Простой инлайн `svg`-кода в нужный компонент

Этот вариант нас не устроил, так как загрязняется код компонента и отсутствует реиспользуемость кода.

#### Вставка иконки, как файл

Этот вариант помогает избежать загрязнения кода, но уже становится невозможным управлять цветом иконки, а хочется. Думаем дальше.

#### Создание компонента на каждую иконку

Да, неплохой вариант: не загрязняет код и можно управлять цветом. Но на данный момент у нас около 35 иконок, а это 35 компонентов… Для каждого еще `PropTypes` нужно указать. А с ростом проекта количество иконок будет увеличиваться. Может есть решение получше?

#### Собирать иконки в спрайт и создать один компонент для всех иконок

О, звучит здорово! Берём. В каждой иконке мы задаём цвет через `currentColor`, что позволит нам управлять цветом через компонент. Мы написали баш-скрипт для оптимизации иконок с помощью инструмента `svgo` и сборки всех иконок в единый спрайт с помощью `svgstore-cli`. Уже в процессе работы столкнулись с проблемой, что некоторые иконки не вписываются в заданный `viewBox`, что затрудняло задание размеров иконки. Решили это путём задания дефолтного значения размеров для каждой иконки, если же требуется размер, отличный от дефолтного - просто передаём нужные нам размеры в параметрах компонента.

### Авторизация

При интеграции авторизации в приложение по незнанию у нас возникли сложности: мы пытались встроить в `iframe` страницы авторизации `OAuth` провайдеров. Самое интересное, что Яндекс позволял это сделать, а остальные (VK и Facebook) отказывались. Оказалось, что необходимо было открывать страницы в отдельном окне, а по завершению, когда провайдер перейдет на страницу `callback` нашего сервера, с помощью `custom URL scheme` мы реализовали `deep linking` в наше приложение. Это позволило нам получать аутентификационные данные из системного браузера после успешной авторизации.

После авторизации мы возвращаем пользователя на экран, с которого он перешёл в авторизацию по записанному в `localStorage` руту.

Кажется, что данное решение не очень красивое. Возможно, лучше было реализовать с помощью `deep linking` на конкретный рут нашего приложения. Это можно отнести к вопросу, что мы хотим еще доработать.

### Разное

- В качестве препроцессора CSS мы используем `SASS`;

<div style="page-break-after: always;"></div>

## Бэкенд

На бэкенде мы используем `Node.JS`, `Express`, `MongoDB` + множество разных библиотек, которые существенно ускоряют разработку.

### Node.JS и Express

`Node.JS` – отличный выбор в ситуации, когда нужно быстро написать бэкенд силами фронтенд-разработчиков.

В качестве фреймворка был выбрал `Express` – минималистичный инструмент, с помощью которого можно легко создать как простой сервер, так и сложное API.

Код серверной части написан с ипользованием стандарта ES6. В процессе разработки использовлся линтер с конфигом от Airbnb.

Далее будут перечислены дополнительные модули/библиотеки, которые использовались в разработке:

- Защита:
    - `Cors` – по умолчанию *;
    - `Helmet` – заголовки для исключения базовых атак;
    - `Express-rate-limit` – ограничение количества запросов для одного IP.
- Обработка ошибок:
    - `Morgan`, `winston` – красивые логи в консоли;
    - `API-errors` – создание ошибок вида: статус, код ошибки, сообщение;
    - Своя обертка для классификации ошибок (добавление поля "тип ошибки").
- Расширение функционала `Express`:
    - `Body parser` – парсинг body запроса;
    - `Compression` – сжатие ответа сервера;
    - `Static` – размещение статических файлов;
- Аутентификация и регистрация:
    - `Passport` – используемые стратегии: `Basic`, `Local`, `BearerStrategy`, `ClientPasswordStrategy`, `Yandex`, `VK`, `Facebook` и `OAuth2`.
    - `Oauth2orize` – собственный сервер `OAuth2`, который прекрасно работает в связке с `passport`.
    - При первом заходе в приложение автоматически создаётся demo пользователь, у которого закрыты некоторые методы, например создание контента. После последующей авторизации через соцсети, вся информация накопленная за demo пользователем перейдет к полноценному авторизованному через соцсети пользователю, также доступны методы логаута со всех устройств.
- Статика:
    - `Firebase Storage` – модуль для хранения статики. Бесплатный тариф предоставляет 5GB места на диске + 1GB трафика в день, чего на нашем этапе вполне достаточно.
- Promise:
    - `BlueBird` – множество полезных надстроек над стандартными промисами (например, `Promise.each`).
- Управлениями процессами:
    - `PM2` – красивые логи, статистика, кластеризация.
- Валидация:
    - `Express-validator` – удобные middleware для валидации запросов, избавляют от дополнительных проверок в самих роутах.
- База данных:
    - `MongoDB` – мы выбрали ее, так как она очень проста в изучении + для наших задач хорошо подходи документоориентированая СУБД. Для удобной работы с базой данных используется библиотека mongoose, модуль `passport-local` (удобное создание пользователей с шифрованием паролей), а также модуль `findOrCreate`. Связи между коллекциями устанавливаются по `id mongo`, а при выдаче результата используется `aggregation pipeline` с `lookup`.
- Фичи, выполняющиеся на сервере:
    - `ColorThief` – определение среднего цвета изображения;
    - `Jimp` – изменение размера картинок;
    - `Cheerio`, `Find Favicon`, `Image Resolver` – парсинг страниц ссылок.

### Heroku

В качестве платформы для хостинга мы используем `Heroku`. Этот выбор обусловлен бесплатностью серсиса, простотой деплоя и интеграцией с `GitHub` (можно настроить автоматический деплой из ветки репозитория).

Однако, чем больше библиотек стал использовать проект, тем больше стало возникать технических трудностей. Некоторые из них:

- Трудности хранения файлов (файловая система на `Heroku` недоступна). Эта проблема была решена переносом статики на `Firebase`.
- Как следствие деплоя из открытого репозитория, необходимо было организовать хранение конфигов. Мы решили зашифровать конфиги с помощью `cryptoJSON`, а ключ передавать как переменную среды.

### Apiary и Dredd

Изначально для тестов планировалось использовать связку `mocha` + `chai` + `chai-http`, но потом выбор изменился в сторону `Apiary` (документация + тестирование с помощью `dredd` с небольшими `hook`). Конечно, это не комплексное тестирование API, но при желании можно дописать необходимое в `hook` файлах с использованием того же `chai`. В итоге мы имеем базовое покрытие и хорошую документацию.

### Travis CI

После написания тестов мы настроили CI, используя веб-сервис `Travis CI`. Как и в случае с Heroku, пришлось немного помучаться с настройкой конфига для запуска билда.

<div style="page-break-after: always;"></div>

## Коммуникация и командная работа

### Взаимодействие в команде

Первоначально оценив объем планируемой работы, мы распредились следующим образом: 4 человека на клиентскую часть и 1 на серверную. С развитием проекта эта расстановка работала успешно, задачи решались согласно планируемому графику.

Некоторые из нас до второго этапа мобилизации не имели практики командной работы над одним общим проектом. Мы стрались выслушивать друг друга и находить компромиссы в спорных мнениях. Не всегда это удавалось, но крупных конфликтов удалось избежать. Взаимное командное уважение осталось всегда на хорошем уровне.

Взаимодействие с менеджерской и дизайнерской стороной происходило без затруднений. Видение продукта и его формирование было единым - нам не ставили невыполнимых задач и всегда выслушивали мнение технической стороны.

Каждое утро менеджер проводил стендапы, на которых обсуждались выполненные задачи и ставились новые. Это помогало держать всю команду в курсе событий,выслушивать мнения каждого члена команды по важным вопросам.

Для онлайн коммуникации мы использовали `Slack` (важные сообщения, общение с кураторами, результаты стендапов) и `Telegram` (для более неформального общения внутри команды).

### Коммуникация в команде разработчиков

Мы создали на `GitHub` аккаунт организации, где объединили репозитории клиентской и серверной части. Это получилось весьма удобно - они всегда находились под общим наблюдением и при этом оставались полностью независимы, каждый со своей внутренней кухней.

Так как большую часть времени вся техническая команда работала оффлайн в одном помещении, мы решили не использовать сложные таск-менеджеры и в планировании задач ограничились `GitHub Issues`. Для удобства каждое issue помечалось тегом, который показывал категорию задачи (`bug`, `refactoring`, `feature` и т.д.).

В качестве модели разработки мы выбрали следующую схему:

- Ветка `master` защищена, при очередном релизе с ней сливается ветка `dev`;
- Ветка `dev` защищена, с ней сливаются рабочие ветки, для сливания необходим код ревью;
- Рабочие ветки называются в соответствии с номером issue, которое описывает изменения в этой ветки (новая фича, правка багов и пр.). Например, `issue-#100`.

При работе над клиентской частью мы практиковали `pull request code reviews`. Обязательные ревью перед слиянием ветки в `dev` улучшали качество кода, помогали находить «глупые» ошибки (опечатки) в реализации и повышали степень совместного владения кодом.

Из-за сильно ограниченных сроков мы решили не тратить время на написание подробной документации проекта. Стендапы и постоянное командное общение держали каждого разработчика в курсе происходящего.

Проектная коммуникация разработчиков с дизайнером строилась следующим образом: дизайнер отрисовывал макеты в `Sketch`, а после экспортировал их в `Zeplin`, в котором каждый разработчик мог удобно просматривать экраны и необходимые размеры, экспортировать графику и т.п.

<div style="page-break-after: always;"></div>

## Чему научились

### Максим:

- Немного ориентироваться в московском метро.
- Не ходить по мокрому полу, после того как прибралась хозяюшка в ШАДе.
- **ES6**. До этого не было практики работы с этим стандартом.
- **React + Redux**. До этого использовал только `Angular 1`. Перед началом второго этапа, не зная кто в какой команде будет, все обсуждали на чем собственно будем писать. И большинство было все-таки за `React`. Поэтому, чтобы не тратить время потом, я как и другие стал изучать эту библиотеку. И вышло вполне успешно - порог вхождения был преодолен к старту работы над проектом. Второй этап дал опыт уже более боевой. `Rudux`-подход позволил понять преимущества однонаправленного потока данных, когда ты лучше контролируешь логику своего приложения. Ну и бонусом опыт тестирования _react-кода_.
- Продумывание архитектуры приложения. Очень важный процесс, от которого зависит вся дальнейшая работа. Изначально не все проблемы смогли увидеть, но благополучно их решили по мере столкновения. Хорошо, что приложение не очень крупное, и рефакторинг был почти безболезненный.
- `Webpack, babel`. С `webpack` был небольшой опыт знакомства, теперь еще узнал как распределять конфиги для разработки и продакшена, что учитывать при использовании `babel`.
- `Eslinter, stylelinter`. До этого не следил за этим почти, и понял насколько это важно, когда у вас командная работа, когда все привыкли писать по-разному и это надо как-то сглаживать, чтобы сохранить удобочитаемость разрастающегося кода.
- Типизация. Очень полезная штука, помогала отлавливать баги, когда активно менялась структура.
- `Github`. Опыт работы с `git` и до этого был, здесь же получил больше практики именно с репозиториями на Github’е. Узнал что репозитории можно объединять под Организациями, как настраивать CI и запуск тестов на `pull request`.
- `Cordova`. Написание приложений на js под мобильные устройства вообще себе слабо представлял, а получилось весьма интересно. Работа с плагинами, решение возникающих проблем сложили общую картину, что приложение выходит хоть и явно не нативное, но сделать из этого что-то прикольное можно.
- **“Атмосфере стартапа”**. Не был до этого в таких условиях, когда у вас не так уж и много времени, когда сотворяете свой собственный продукт с нуля, когда он вам нравится, вы им болеете и готовы работать над ним овердофига. Вышло отлично, хоть и подустали все.
- И в целом, стал более актуальным во фронтенде, чем был до этого проекта.

## Планы на будущее

- Кеширование запросов к серверу;
- Документация для фронтенда, Storybook;
- Обработчик ошибок;

<div style="page-break-after: always;"></div>

## Ссылки

- [Организация на GitHub](https://github.com/Yamblz-Iceberg/)
- [Репозиторий приложения на GitHub](https://github.com/Yamblz-Iceberg/iceberg-app)
- [Репозиторий API на GitHub](https://github.com/Yamblz-Iceberg/iceberg-api)
- [Документация API на Apiary](http://docs.icebergproject.apiary.io/)
- [Схема базы данных](https://iceberg-project.herokuapp.com/db-diagram.html)
