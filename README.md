# Техническая защита

## О проекте

_(Описать о чем наш продукт, какая была постановка задачи)_

## Фронтенд

### Cordova

В качестве платформы для разработки мобильного приложения мы использовали **Cordova**. В первую очередь этот выбор обусловлен простотой разработки и большим количеством плагинов, предоставляющих нативный функционал. Вот несколько из них и описание задач, которые они помогали решать:

**cordova-plugin-safariviewcontroller**

Этот плагин позволяет просматривать веб-страницы внутри приложения. Он был выбран как альтернатива похожему по функционалу cordova-plugin-inappbrowser по ряду причин:
- использует Safari Webview главного браузера системы на iOS, который быстрей чем UIWebView;
- это рекомендумый Apple способ для таких ситуаций;
- поддерживает Android OS.

**cordova-plugin-file**

Плагин отвечает за работу с файловой системой устройства. Мы столкнулись с проблемой отображения изображений при билде на телефоны, так как некорректно указывался путь до нужного локального файла. Использование этого плагина позволило решить эту задачу для обоих платформ.

### React

Основой фронтенд-стека был выбран **React**. Причин у этого выбора несколько:

- Компонентный подход данной библиотеки гибок и позволяет удобно и эффективно решать поставленную задачу.
- Большая часть команды была уже хотя бы частично знакома с React, что позволило не тратить время на выбор компромисса и изучение новой технологии.
- Комьюнити React огромно, а это значит, что на большинство вопросов, возникающих в процессе разработки, можно найти ответ в статьях, туторилах или на специальных ресурсах.

Для навигации между экранами и логики переключения мы использовали **React Router v4**.

### Redux

При выборе модели данных мы обратили внимание на **Redux**. Этот выбор объясняется желанием иметь централизованное хранилище, с помощью которого удобно отслеживать потоки данных в приложении, а также желанием упростить взаимодействие с сервером. В качестве middleware для Redux мы используем **Thunk**.

При разработке и отлаживании проекта очень помог **Redux Devtools**. С помощью этого инструмента мы быстро отлаживали и тестировали модель данных, искали баги.

### Линтинг

В работе над проектом мы использовали **ESLint** c пресетом от Airbnb. Часть правил конфигурации мы переписали в соответствии со своими предпочтениями _(можно привести пример исправлений)_.

### Сборка

Собирали проект с помощью **Webpack**. Конфиг писали сами, с нуля, учитывая особенности разработки гибридного приложения и предпочтения команды. Конфигурация разделена на два файла – production и development.

Для удобства и повышения скорости разработки в development режиме мы использовали **WebpackDevServer** и **HotModuleReplacementPlugin**. Сервер при старте билдит `index.html` (необходимо для того, чтобы Cordova собрала проект), а после следит за изменениями в папке `src` и в случае необходимости обновляет бандл `bundle.js`.

В loaders у нас находятся:

- `babel-loader` для транспиляции React и ES6+ синтаксиса;
- `style-loader`, `css-loader` и `sass-loader` для преобразования SASS в CSS и последующей сборки в `bundle.css`;
- `file-loader` для шрифтов, SVG-иконок и прочего;
- `eslint-loader` для линтинга.

Для удобства отладки в процессе разработки мы настроили **Source Map**.

### UNIT-тесты

_(Написать что-то невероятно крутое про наши тесты)_

### Тестирование на девайсах

В процессе разработки мы использовали эмуляторы устройств, а также тестировали приложение на различных реальных девайсах. Среди них были iPhone 5-7, устройства на Android с версиями 5-8.

### Разное

_(Может как-то переименовать этот раздел?)_

В архитектуре файловой системы проекта и в наименовании классов мы старались придерживаться методологии **БЭМ**.

_(Написать про CI. Что мы не стали настраивать Continuous Integration, потому что на настройку и поддержку CI ушло бы много времени. Но CI – это круто!)_

## Бэкенд

Мы используем Node.JS + Express + MongoDB, и множетсво разных библиотек, которые существенно ускоряют разработку. Далее подробнее:

### Node.JS

Вполне очевидный выбор для JavaScript разработчика, тем более, что один из членов команды имеет опыт разработки на Node. Код написан с ипользованием стандарта ES6. Линтер конфиг от Airbnb с небольшими правками.

### Express

Быстрый и минималистичный фреймворк, с которым можно легко и быстро развернуть как простой сервер, так и создавать сложные API даже обычному фронтенд разработчику, конечно же с какими-то знаниями (переписать). Далее будут перечислены дополнительные модули/библиотеки, которые использовались в разработке:

- Защита
  - cors - по умолчанию *
  - helmet - заголовки для исключения базовых атак
  - express-rate-limit - ограничение количества запросов для одного ip
- Обработка ошибок
  - Morgan, winston - красивые логи в консоли
  - api-errors - создание ошибок вида: статус, код ошибки, сообщение
  - Своя обертка, для классификации ошибок - добавление поля тип ошибки
- Расширение функционала express
  - body parser - парсинг body запроса
  - compression - сжатие ответа сервера
  - static - размещение статических файлов
- Аутентификация и регистрация
  - Passport - используемые стратегии: Basic, Local, BearerStrategy, ClientPasswordStrategy, Yandex, Vk, Facebook и Oauth2.
  - oauth2orize - собственный  сервер OAuth2, который прекрасно работает в связке с passport.
  - При первом заходе в приложение автоматически создаётся demo пользователь, у которого закрыты некоторые методы, например создание контента. После последующей авторизации через соцсети, вся информация накопленная за demo пользователем перейдет к полноценному авторизованному через соцсети пользователю, также доступны методы логаута со всех устройств.
- Статика
  - Firebase Storage - модуль для хранения статики, бесплатных 5GB хватает за глаза, а ограничение 1GB трафика, хоть один раз и сработало, но для учебных целей - самое то
- Promise
  - BlueBird - куча полезных надстроек, например Promise.each, да и работает точно не медленнее стандартных промисов
- Управлениями процессами
  - PM2 - красивые логи, статистика, кластеризация
- Валидация
  - Express-validator - удобные middleware для валидации запросов, избавляют от дополнительных проверок в самих роутах
- База данных
  - MongoDB - Мы выбрали ее, т.к очень проста в изучении, да и для наших задач вполне подходит документоориентированая СУБД. Для удобной работы с базой данных используется библиотека mongoose, а также модули passport-local - удобнее создание пользователей с шифрованием паролей, а также модуль с говорящим сам за себя названием - findOrCreate. Связи между коллекциями устанавливаются по id mongo, а при выдаче результата используется aggregation pipeline с lookup.
- Фичи выполняющиеся на сервере
  - ColorThief - Определение среднего цвета
  - Jimp - Изменение размера картинок
  - Cheerio, Find Favicon, Image Resolver - Парсинг страниц

### Heroku

Задеплоить свой проект проще некуда, особенно радует интеграция с GitHub, когда после коммита например в Dev, автоматически запускается свежая установка сервера. Однако чем больше библиотек стал использовать проект, тем больше технических трудностей, одна из которых это хранение файлов (файловая система на heroku недоступна). Как следствие деплоя из открытого репозитория, необходимо было организовать хранение конфигов, библиотека canvas требует кастомный buildpack и cedar 14 версии. Проблема с файловой системой была разрешена с помощью FireBase. Конфиги были зашифрованны с помощью cryptoJSON, а ключ передается как переменная среды.

### Apiary и Dredd

Изначально для тестов планировалось использовать связку mocha, chai, chai-http, но потом была обнаружена apiary - просто пишешь документацию, и тестируешь с помощью dredd с небольшими hook'ами. Конечно это не комплексное тестирование api, но при желании можно дописать необходимое в hook файлах с использованием того же chai. В итоге мы имеем базовое покрытие и хорошую документацию

### Travis CI

Когда тесты написаны и настроены, можно с легкостью подключить и CI, тут добавить особо нечего, однако как в случае и heroku, пришлось немного помучаться с настройкой конфига для запуска билда.

## Коммуникация и командная работа

Для онлайн коммуникации мы использовали Slack (важные сообщения, общение с кураторами, результаты стендапов и пр.) и Telegram (для более неформального общения внутри команды).

Каждое утро менеджер проводил стендапы, на которых обсуждались выполненные задачи и ставились новые. Это помогало держать всю команду в курсе событий,выслушивать мнения каждого члена команды по важным вопросам.

Так как большую часть времени вся техническая команда работала оффлайн в одном помещении, мы решили не использовать сложные таск-менеджеры и в планировании задач ограничились GitHub Issues. Для удобства каждое issue помечалось тегом, который показывал категорию задачи (bug, refactoring, feature и т.д.).

В качестве модели разработки мы выбрали следующую схему:

- Ветка `master` защищена, при очередном релизе с ней сливается ветка `dev`;
- Ветка `dev` защищена, с ней сливаются рабочие ветки, для сливания необходим код ревью;
- Рабочие ветки называются в соответствии с номером issue, которое описывает изменения в этой ветки (новая фича, правка багов и пр.). Например, `issue-#100`.

При работе над клиентской частью мы практиковали **pull request code reviews**. Обязательные ревью перед слиянием ветки в `dev` улучшали качество кода, помогали находить «глупые» ошибки (опечатки) в реализации и повышали степень совместного владения кодом.

Проектная коммуникация разработчиков с дизайнером строилась следующим образом: дизайнер отрисовывал макеты в **Sketch**, а после экспортировал их в **Zeplin**, в котором каждый разработчик мог удобно просматривать экраны и необходимые размеры, экспортировать графику и т.п.

## Чему научились

### Команданая работа

Некторые из нас до второго этапа мобилизации не имели практики командной работы над одним общим проектом. Мы стрались выслушивать друг друга и находить компромисы в спорных мнениях. Не всегда это удавалось, но крупных конфликтов удалось избежать. Взаимное командное уважение осталось всегда на хорошем уровне.

Первоначально оценив объем планируемой работы, мы распредились 4 человека на клиентскую часть и 1 на серверную. И с развитием проекта эта расстановка работала успешно, задачи решались согласно планируемому графику.

Взаимодействие с менеджерской и дизайнерской стороной происходило без затруднений. Видение продукта и его формирование было единым - нам не ставили невыполнимых задач и всегда выслушивали мнение технической стороны.

### GitHub

На GitHub'е мы создали аккаунт организации, где объединили репозитории клиентской и серверной части. Это получилось весьма удобно - они всегда находились под общим наблюдением и при этом оставались полностью независимы, каждый со своей внутренней кухней.
