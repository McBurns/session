# express-session

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Build Status][travis-image]][travis-url]
[![Test Coverage][coveralls-image]][coveralls-url]

## Installation

Этот модуль [Node.js](https://nodejs.org/en/)  доступен для установки через
[npm registry](https://www.npmjs.com/). Установка производится 
[командой `npm install`](https://docs.npmjs.com/getting-started/installing-npm-packages-locally):

```sh
$ npm install express-session
```

## API

```js
var session = require('express-session')
```

### session(options)

Создайте сеанс middleware с параметрами `options`.

**Примечание:** данные сессии _не_ _сохраняются_ в cookies, в них хранится только 
идентификатор (ID) сессии. 
Сами данные хранятся на сервере. 

**Примечание:** начиная с версии 1.5.0 необходимости в использовании 
[`cookie-parser` middleware](https://www.npmjs.com/package/cookie-parser)
больше нет. Модуль сам читает и записывает файлы cookies в `req`/`res`. 
Использование `cookie-parser` может вызвать конфликты в случае несовпадения параметра
 `secret` в этих двух модулях.

**Предупреждение:** используемое по умолчанию на стороне сервера хранилище `MemoryStore`
_намеренно_ не предназначено для использования в рабочих проектах. 
Его использование может привести к утечкам памяти и невозможности масштабирования процессов.
Хранилище предназначено для разработки и отладки.

Список возможных хранилищ смотри в [разделе совместимых хранилищ](#compatible-session-stores).

#### Options

Свойства, используемые `express-session` в объекте options.

##### cookie

Настройки идентификации сессии. Параметры по умолчанию:
`{ path: '/', httpOnly: true, secure: false, maxAge: null }`.

Ниже перечислены параметры, которые можно задать в этом объекте.

##### cookie.domain

Задаёт значение `Domain` аттрибута `Set-Cookie`. По умолчанию домен не задан, 
большинство клиентов будут использовать cookies применительно к текущему домену.

##### cookie.expires

Использует объект `Date` для задания значения `Expires` аттрибута `Set-Cookie`.
По умолчанию значение не задано, и большинство клиентов будет считать его непостоянным
или "сессионным (сеансовым) cookie" и удалит при завершении web-приложения.

**Примечание:** если задан параметр `expires` вместе с `maxAge`, будет использоваться 
параметр, заданный последним.

**Примечание:** Не рекомендуется устанавливать напрямую значение `expires`, вместо этого
лучше использовать параметр `maxAge`.

##### cookie.httpOnly

Устанавливает `булево` значение `HttpOnly` аттрибута `Set-Cookie`. 
Если используется `true`, значение `HttpOnly` установлено, в противном случае нет. 
По умолчанию `HttpOnly` значение установлено.

**Примечание:** рекомендуется устанавливать значение `true`. В противном случае
JavaScript на стороне клиента не будет видеть cookie приложения в свойстве `document.cookie`.

##### cookie.maxAge

Устанавливает значение `number` (в миллисекундах) для вычисления значения `Expires`
аттрибута `Set-Cookie`. Вычисляется добавлением значения
`maxAge` к тетущему времени сервера в миллисекундах для вычисления времени `Expires`.
По умолчанию не установлено.

**Примечание** Если установлены `expires` и `maxAge` вместе, используется значение, 
заданное последним.

##### cookie.path

Устанавливает значение `Path` атрибута `Set-Cookie`. По умолчанию установлено `'/'`, 
то есть, корневой каталог домена.

##### cookie.sameSite

Устанавливает `булево` или `строковое` значение для `SameSite` аттрибута `Set-Cookie`.

  - `true` устанавливает аттрибут `SameSite` в значение `Strict` для строгого соблюдения сайта.
  - `false` отключает аттрибут `SameSite`.
  - `'lax'` устанавливает аттрибут `SameSite` в значение `Lax` для слабого контроля сайта.
  - `'strict'` устанавливает аттрибут `SameSite` в значение `Strict` для строгого соблюдения сайта.

Подробную информацию можно найти на странице описания спецификации:
https://tools.ietf.org/html/draft-west-first-party-cookies-07#section-4.1.1

**Примечание:** параметр не полностью стандатизирован и может измениться в будущем.
Кроме того, следует учитывать, что некоторые клиенты могут его игнорировать или неверно
понимать.

##### cookie.secure

Устанавливает `булево` значение для  `Secure` аттрибута `Set-Cookie`. 
В значении `true` параметр `Secure` установлен, иначе нет. По умолчанию параметр `Secure`
не установлен.

**Примечание:** если параметр установлен в значение `true`, клиенты, совместимые с этим
 параметром не буду отправлять на сервер файл cookie, если не установлено HTTPS соединение.

Пожалуйста, обратите внимание, что `secure: true` является **рекомендованным** значением. 
Однако для безопасного получения файлов cookie необходимо, чтобы сайт поддерживал 
соединение https. Если параметр `secure` установлен, но соединение проходит через HTTP, 
cookie не будет установлен. Если вы используете node.js через прокси-сервер, и 
установлено `secure: true`, необходимо установить "доверенный proxy" в express:

```js
var app = express()
app.set('trust proxy', 1) // trust first proxy
app.use(session({
  secret: 'keyboard cat',
  resave: false,
  saveUninitialized: true,
  cookie: { secure: true }
}))
```

Если вы используете в готовом проекте безопасные cookie, но необходимо тестирование в 
процессе разработки, вы можете использовать в express настройки на основе `NODE_ENV`:

```js
var app = express()
var sess = {
  secret: 'keyboard cat',
  cookie: {}
}

if (app.get('env') === 'production') {
  app.set('trust proxy', 1) // trust first proxy
  sess.cookie.secure = true // serve secure cookies
}

app.use(session(sess))
```

Парметр `cookie.secure` может быть установлен в специальное значение `'auto'`,
чтобы использовать любое доступное соединение.
Необходимо осторожно использовать такое значение, если сайт доступен как по HTTP, так и
по HTTPS, так как после установки значения cookie для HTTPS, он больше не будет
доступен в HTTP. Значение используется при верно настроенном в Express `"trust proxy"`
для упрощения перехода от разработки к готовому проекту.

##### genid

Метод, генерирующий новый идентификатор ID сессии. Возвращает функцию, значение которой
строка, которая будет использоваться в качестве нового идентификатора сессии. 
Первый аргумент полученной функции `req`, вы можете использовать некое значение,
 привязанное к `req` для генерации ID.

По умолчанию это функция, использующая библиотеку `uid-safe`для создания ID.

**Примечание:** будьте внимательны при создании нового идентификатора, чтобы не получить 
конфликт сессий.

```js
app.use(session({
  genid: function(req) {
    return genuuid() // use UUIDs for session IDs
  },
  secret: 'keyboard cat'
}))
```

##### name

Имя идентификатора ID сессии cookie для устновки в ответе (и чтения в запросе).

По умолчанию значение равно `'connect.sid'`.

**Примечание:** если у вас несколько приложений работает на одном и том же хосте
(при использовании `localhost` или `127.0.0.1` разные порты не обеспечивают разные
имена хоста), надо отделить файлы сессий cookie друг от друга.
Самый простой способ, дать разные значения `name` для каждого приложения.

##### proxy

Установка доверенного прокси сервера при настройке безопасных cookies 
(через заголовок "X-Forwarded-Proto").

Значение по умолчанию `undefined`.

  - `true` Заголовок "X-Forwarded-Proto" используется.
  - `false` Все заголовки игнорируются, соединение считается доверенным
   только при наличии прямого соединения TLS/SSL.
  - `undefined` Использует параметр "trust proxy" из express

##### resave

Принудительно сохраняет сессию в хранилище, даже если никакие значения не
изменялись при запросах. Может быть необходимо в зависимости от используемого
хранилища, однако может привести к тому, что при двух параллельных запросах от
клиента изменения, внесённые при работе одного запроса, перезапишутся, когда
закончится работа другого запроса, даже если он не внёс изменний (это зависит 
от типа используемого хранилища).

Значение по умолчанию `true`, но в будущем оно изменится.
Пожалуйста, будьте внимательны при выборе значения для своего случая.
Скорее всего, вам понадобится значение `false`.

Как узнать, какое значение требуется для моего хранилища? 
Лучший всего узнать, реализован ли в хранилище метод `touch`. 
Если реализован, вы можете уверенно устанавливать `resave: false`. 
Если метод `touch` не реализован в вашем хранилище, и хранилище устанавливает
время истечениясрока действия сохранённых сессий, вам может понадобиться `resave: true`.

##### rolling

Настраивает принудительное обновление cookie после каждого обращения. 
Срок истечения действия cookie при каждом обращении сбрасывается до 
[`maxAge`](#cookiemaxage), сбрасывая отсчёт срока действия.

Значение по умолчанию `false`.

**Примечание:** если этот параметр установлен в `true`, а параметр 
`saveUninitialized` установлен в `false`, cookie не будет установлен для
неинициализированной сессии.

##### saveUninitialized

Настраивает принудительное сохранение неинициализированной сессии в хранилище.
Сессия не инициализирована, когда она была создана, но данные не изменены.
Выбор `false` нужен для реализации аутентификации пользователя сессии, 
сокращения обращений к хранилищу сервера или выполнения правил, требующих
получения разрешения перед записью cookie. Кроме того, значение `false` 
позволяет неавторизованному пользователю делать несколько параллельных запросов
без создания сессии.

Значение по умолчанию `true`, но это значение в будущем изменится.
Пожалуйста, будьте внимательны при выборе значения параметра для вашего случая.

**Примечание:** при использовании Session вместе с PassportJS, Passport
добавляет пустой объект Passport в сессию для использования после аутентификации
пользователя. Это будет рассматриваться как изменение сессии, что и будет сохранено. 
*Это поведение было исправлено в PassportJS 0.3.0*

##### secret

**Требуемый параметр**

Секретное значение, используемое для подписи cookie сессии. 
Может содержать одиночную строку или массив строк. При массиве строк для подписи
файла cookie идентификатора сессии будет использован только первый элемент массива.
При проверке подписи в запросах будут учитываться все элементы массива

##### store

Хранилище сессии, по умолчанию новый экземпляр `MemoryStore`.

##### unset

Управляет результатом сброса `req.session` (использование `delete`, установка в `null` 
и т.д.).

Значение по умолчанию `'keep'`.

  - `'destroy'` Сессия будет уничтожена после завершения запроса.
  - `'keep'` Сессия будет сохранена в хранилище, но изменения, сделанные во время
  запроса, игнорируются и не сохранятся.

### req.session

Чтобы сохранить или получить доступ к данным сессии, просто используйте `req.session`,
который в большинстве случаев сериализуется хранилищем в виде JSON для сохранения
точности полученных данных.
Для примера ниже представлен пользовательский счётчик просмотров:

```js
// Использование middleware сессии
app.use(session({ secret: 'keyboard cat', cookie: { maxAge: 60000 }}))

// Доступ к данным сессии через req.session
app.get('/', function(req, res, next) {
  if (req.session.views) {
    req.session.views++
    res.setHeader('Content-Type', 'text/html')
    res.write('<p>Просмотров: ' + req.session.views + '</p>')
    res.write('<p>Срок действия: ' + (req.session.cookie.maxAge / 1000) + 's</p>')
    res.end()
  } else {
    req.session.views = 1
    res.end('Добро пожаловать в демонстрацию сессий. Обновите страницу!')
  }
})
```

#### Session.regenerate(callback)

Для генерации сессии просто вызовите этот метод. После его завершения новый SID и 
экземпляр `Session` будут инициализированы в `req.session`, и вызовется `callback`.

```js
req.session.regenerate(function(err) {
  // Здесь будет новая сессия
})
```

#### Session.destroy(callback)

Уничтожает сессию и отключает свойство `req.session`.
После завершения будет вызван `callback`.

```js
req.session.destroy(function(err) {
  // Доступа к сессии здесь уже нет
})
```

#### Session.reload(callback)

Перезагружает данные сессии из хранилижа, заново заполняет объект`req.session`. 
После завершения вызывает `callback`.

```js
req.session.reload(function(err) {
  // Обновление сессии
})
```

#### Session.save(callback)

Сохраняет сессию в хранилище, заменяя содержимое хранилища хранящимися в памяти 
данными (некоторые хранилища могут изменять это поведение, обратитесь к документации
хранилища для получения точной информации).

Метод автоматически вызывается после ответа HTTP, если данные сессии были изменены
(некоторые хранилища могут изменять это поведение с помощью различных опций middleware),
поэтому обычно метод специально не вызывается.

В некоторых случаях имеет смысл вызвать метод специально. Например, при перенаправлении,
длительных по времени запросах или использовании WebSockets.

```js
req.session.save(function(err) {
  // Сессия сохранена
})
```

#### Session.touch()

Обновляет свойство `.maxAge`. Как правило, делать это необязательно, поскольку 
middleware делает это за вас.

### req.session.id

Содержит уникальный идентификатор сессии. Это свойство - псевдоним 
[`req.sessionID`](#reqsessionid-1) и не может быть изменено.
свойство было добавлено, чтобы сделать ID сессии доступным из объекта `session`.

### req.session.cookie

Содержит уникальный объект cookie сессии. Метод позволяет изменять cookie сессии для
каждого пользователя. Например, можно установить `req.session.cookie.expires` в `false`,
чтобы cookie оставался только на время действия сессии пользователя.

#### Cookie.maxAge

Псевдоним `req.session.cookie.maxAge`. Возвращает осташееся время в миллисекундах,
позволяет назначить новое значение для свойства `.expires`. 
По сути, следующие два вызова идентичны:

```js
var hour = 3600000
req.session.cookie.expires = new Date(Date.now() + hour)
req.session.cookie.maxAge = hour
```

К примеру, если `maxAge` установлено в значение `60000` (одна минута), и прошло
30 секунд, свойство будет возвращать `30000`, пока текущий запрос не завершится, и
всё это время будет вызываться `req.session.touch()` для сброса `req.session.maxAge`
в его первоначальное значение.

```js
req.session.cookie.maxAge // => 30000
```

#### Cookie.originalMaxAge

Свойство `req.session.cookie.originalMaxAge` возвращает
`maxAge` (время жизни) вайла cookie сессии в миллисекундах.

### req.sessionID

Для получения идентификатора сессии, обратитесь в запросе к свойству `req.sessionID`. 
Это простое значение, доступное только для чтения во время создания/исполнения запроса.

## Session Store Implementation
_Реализация_ _хранилища_ _сеансов_

Каждое хранилище _должно_ _быть_ `EventEmitter` и реализовывать специальные методы. 
Следующие методы представляют собой список **обязательных**, **рекомендованных**
и **необязательных**.

  * Обязательные методы должны быть обязательно реализованы в хранилище.
  * Рекомендованные методы будут вызываться только в том случае, если они реализованы.
  * Необязательные методы хранилище не вызывает при работе, но может использовать для
  совместимости с другими хранилищами или для удобства пользователя.

В качестве примера рассмотрим реализацию хранилища [connect-redis](http://github.com/visionmedia/connect-redis).

### store.all(callback)

**Необязательный**

Этот необязательный метод используется для получения массива всех сессий в хранилище. 
Свойство `callback` вызывается в виде `callback(error, sessions)`.

### store.destroy(sid, callback)

**Обязательный**

Этот необходимый метод используется для уничтожения/удаления сессии из хранилища 
с учетом идентификатора сеанса (`sid`). 
Свойство `callback` вызывается в виде `callback(error, sessions)` после завершения 
сессии.

### store.clear(callback)

**Необязательный**

Этот необязательный метод используется для удаления всех сеансов из хранилища. 
Свойство `callback` вызывается в виде `callback(error, sessions)` после очистки
хранилища.

### store.length(callback)

**Необязательный**

Этот необязательный метод используется для подсчета всех сессий в хранилище.
Свойство `callback` вызывается в виде `callback(error, len)`.

### store.get(sid, callback)

**Обязательный**

Обязательный метод. Используется для получения из хранилища данных сессии с
переданным ID (`sid`). 
Свойство `callback` вызывается в виде `callback(error, session)`.

Аргумент `session` должен быть сессией, если она есть, иначе `null` или
`undefined`, если сессия не найдена (и если не было ошибки). Специальный
 случай возникает при `error.code === 'ENOENT'` и действует как `callback(null, null)`.

### store.set(sid, session, callback)

**Обязательный**

Обязательный метод. Используется для воссоздания в хранилище с идентификатором 
ID (`sid`) и объектом сессии (`session`). 
Свойство `callback` вызывается в виде `callback(error)` после того, как сессия
была записана.

### store.touch(sid, session, callback)

**Рекомендованный**

Рекомендованный метод. Используется для обращения ("touch") к данным сеанса с
указанным ID (`sid`) и объектом (`session`). 
Свойство `callback` вызывается в виде `callback(error)` после того, как обращение 
было выполнено.

В первую очередь исользуется для того, чтобы из хранилища не удалялись сессии
при отсутствии других обращений к хранилищу сессии, сбрасывает таймер простоя.

## Compatible Session Stores
_Совместимые_ _хранилища_ _сессии_

Нижеследующие модули реализуют хранилища сессии, совместимые с этим модулем.
Пожалуйста, сделайте PR для добавления других модулей. :)

[![★][aerospike-session-store-image] aerospike-session-store][aerospike-session-store-url] A session store using [Aerospike](http://www.aerospike.com/).

[aerospike-session-store-url]: https://www.npmjs.com/package/aerospike-session-store
[aerospike-session-store-image]: https://img.shields.io/github/stars/aerospike/aerospike-session-store-expressjs.svg?label=%E2%98%85

[![★][cassandra-store-image] cassandra-store][cassandra-store-url] An Apache Cassandra-based session store.

[cassandra-store-url]: https://www.npmjs.com/package/cassandra-store
[cassandra-store-image]: https://img.shields.io/github/stars/webcc/cassandra-store.svg?label=%E2%98%85

[![★][cluster-store-image] cluster-store][cluster-store-url] A wrapper for using in-process / embedded
stores - such as SQLite (via knex), leveldb, files, or memory - with node cluster (desirable for Raspberry Pi 2
and other multi-core embedded devices).

[cluster-store-url]: https://www.npmjs.com/package/cluster-store
[cluster-store-image]: https://img.shields.io/github/stars/coolaj86/cluster-store.svg?label=%E2%98%85

[![★][connect-azuretables-image] connect-azuretables][connect-azuretables-url] An [Azure Table Storage](https://azure.microsoft.com/en-gb/services/storage/tables/)-based session store.

[connect-azuretables-url]: https://www.npmjs.com/package/connect-azuretables
[connect-azuretables-image]: https://img.shields.io/github/stars/mike-goodwin/connect-azuretables.svg?label=%E2%98%85

[![★][connect-cloudant-store-image] connect-cloudant-store][connect-cloudant-store-url] An [IBM Cloudant](https://cloudant.com/)-based session store.

[connect-cloudant-store-url]: https://www.npmjs.com/package/connect-cloudant-store
[connect-cloudant-store-image]: https://img.shields.io/github/stars/adriantanasa/connect-cloudant-store.svg?label=%E2%98%85

[![★][connect-couchbase-image] connect-couchbase][connect-couchbase-url] A [couchbase](http://www.couchbase.com/)-based session store.

[connect-couchbase-url]: https://www.npmjs.com/package/connect-couchbase
[connect-couchbase-image]: https://img.shields.io/github/stars/christophermina/connect-couchbase.svg?label=%E2%98%85

[![★][connect-datacache-image] connect-datacache][connect-datacache-url] An [IBM Bluemix Data Cache](http://www.ibm.com/cloud-computing/bluemix/)-based session store.

[connect-datacache-url]: https://www.npmjs.com/package/connect-datacache
[connect-datacache-image]: https://img.shields.io/github/stars/adriantanasa/connect-datacache.svg?label=%E2%98%85

[![★][@google-cloud/connect-datastore-image] @google-cloud/connect-datastore][@google-cloud/connect-datastore-url] A [Google Cloud Datastore](https://cloud.google.com/datastore/docs/concepts/overview)-based session store.

[@google-cloud/connect-datastore-url]: https://www.npmjs.com/package/@google-cloud/connect-datastore
[@google-cloud/connect-datastore-image]: https://img.shields.io/github/stars/GoogleCloudPlatform/cloud-datastore-session-node.svg?label=%E2%98%85

[![★][connect-db2-image] connect-db2][connect-db2-url] An IBM DB2-based session store built using [ibm_db](https://www.npmjs.com/package/ibm_db) module.

[connect-db2-url]: https://www.npmjs.com/package/connect-db2
[connect-db2-image]: https://img.shields.io/github/stars/wallali/connect-db2.svg?label=%E2%98%85

[![★][connect-dynamodb-image] connect-dynamodb][connect-dynamodb-url] A DynamoDB-based session store.

[connect-dynamodb-url]: https://github.com/ca98am79/connect-dynamodb
[connect-dynamodb-image]: https://img.shields.io/github/stars/ca98am79/connect-dynamodb.svg?label=%E2%98%85

[![★][connect-loki-image] connect-loki][connect-loki-url] A Loki.js-based session store.

[connect-loki-url]: https://www.npmjs.com/package/connect-loki
[connect-loki-image]: https://img.shields.io/github/stars/Requarks/connect-loki.svg?label=%E2%98%85

[![★][connect-ml-image] connect-ml][connect-ml-url] A MarkLogic Server-based session store.

[connect-ml-url]: https://www.npmjs.com/package/connect-ml
[connect-ml-image]: https://img.shields.io/github/stars/bluetorch/connect-ml.svg?label=%E2%98%85

[![★][connect-mssql-image] connect-mssql][connect-mssql-url] A SQL Server-based session store.

[connect-mssql-url]: https://www.npmjs.com/package/connect-mssql
[connect-mssql-image]: https://img.shields.io/github/stars/patriksimek/connect-mssql.svg?label=%E2%98%85

[![★][connect-monetdb-image] connect-monetdb][connect-monetdb-url] A MonetDB-based session store.

[connect-monetdb-url]: https://www.npmjs.com/package/connect-monetdb
[connect-monetdb-image]: https://img.shields.io/github/stars/MonetDB/npm-connect-monetdb.svg?label=%E2%98%85

[![★][connect-mongo-image] connect-mongo][connect-mongo-url] A MongoDB-based session store.

[connect-mongo-url]: https://www.npmjs.com/package/connect-mongo
[connect-mongo-image]: https://img.shields.io/github/stars/kcbanner/connect-mongo.svg?label=%E2%98%85

[![★][connect-mongodb-session-image] connect-mongodb-session][connect-mongodb-session-url] Lightweight MongoDB-based session store built and maintained by MongoDB.

[connect-mongodb-session-url]: https://www.npmjs.com/package/connect-mongodb-session
[connect-mongodb-session-image]: https://img.shields.io/github/stars/mongodb-js/connect-mongodb-session.svg?label=%E2%98%85

[![★][connect-pg-simple-image] connect-pg-simple][connect-pg-simple-url] A PostgreSQL-based session store.

[connect-pg-simple-url]: https://www.npmjs.com/package/connect-pg-simple
[connect-pg-simple-image]: https://img.shields.io/github/stars/voxpelli/node-connect-pg-simple.svg?label=%E2%98%85

[![★][connect-redis-image] connect-redis][connect-redis-url] A Redis-based session store.

[connect-redis-url]: https://www.npmjs.com/package/connect-redis
[connect-redis-image]: https://img.shields.io/github/stars/tj/connect-redis.svg?label=%E2%98%85

[![★][connect-memcached-image] connect-memcached][connect-memcached-url] A memcached-based session store.

[connect-memcached-url]: https://www.npmjs.com/package/connect-memcached
[connect-memcached-image]: https://img.shields.io/github/stars/balor/connect-memcached.svg?label=%E2%98%85

[![★][connect-memjs-image] connect-memjs][connect-memjs-url] A memcached-based session store using
[memjs](https://www.npmjs.com/package/memjs) as the memcached client.

[connect-memjs-url]: https://www.npmjs.com/package/connect-memjs
[connect-memjs-image]: https://img.shields.io/github/stars/liamdon/connect-memjs.svg?label=%E2%98%85

[![★][connect-session-knex-image] connect-session-knex][connect-session-knex-url] A session store using
[Knex.js](http://knexjs.org/), which is a SQL query builder for PostgreSQL, MySQL, MariaDB, SQLite3, and Oracle.

[connect-session-knex-url]: https://www.npmjs.com/package/connect-session-knex
[connect-session-knex-image]: https://img.shields.io/github/stars/llambda/connect-session-knex.svg?label=%E2%98%85

[![★][connect-session-sequelize-image] connect-session-sequelize][connect-session-sequelize-url] A session store using
[Sequelize.js](http://sequelizejs.com/), which is a Node.js / io.js ORM for PostgreSQL, MySQL, SQLite and MSSQL.

[connect-session-sequelize-url]: https://www.npmjs.com/package/connect-session-sequelize
[connect-session-sequelize-image]: https://img.shields.io/github/stars/mweibel/connect-session-sequelize.svg?label=%E2%98%85

[![★][couchdb-expression-image] couchdb-expression][couchdb-expression-url] A [CouchDB](https://couchdb.apache.org/)-based session store.

[couchdb-expression-url]: https://www.npmjs.com/package/couchdb-expression
[couchdb-expression-image]: https://img.shields.io/github/stars/tkshnwesper/couchdb-expression.svg?label=%E2%98%85

[![★][dynamodb-store-image] dynamodb-store][dynamodb-store-url] A DynamoDB-based session store.

[dynamodb-store-url]: https://www.npmjs.com/package/dynamodb-store
[dynamodb-store-image]: https://img.shields.io/github/stars/rafaelrpinto/dynamodb-store.svg?label=%E2%98%85

[![★][express-mysql-session-image] express-mysql-session][express-mysql-session-url] A session store using native
[MySQL](https://www.mysql.com/) via the [node-mysql](https://github.com/felixge/node-mysql) module.

[express-mysql-session-url]: https://www.npmjs.com/package/express-mysql-session
[express-mysql-session-image]: https://img.shields.io/github/stars/chill117/express-mysql-session.svg?label=%E2%98%85

[![★][express-oracle-session-image] express-oracle-session][express-oracle-session-url] A session store using native
[oracle](https://www.oracle.com/) via the [node-oracledb](https://www.npmjs.com/package/oracledb) module.

[express-oracle-session-url]: https://www.npmjs.com/package/express-oracle-session
[express-oracle-session-image]: https://img.shields.io/github/stars/slumber86/express-oracle-session.svg?label=%E2%98%85

[![★][express-sessions-image] express-sessions][express-sessions-url]: A session store supporting both MongoDB and Redis.

[express-sessions-url]: https://www.npmjs.com/package/express-sessions
[express-sessions-image]: https://img.shields.io/github/stars/konteck/express-sessions.svg?label=%E2%98%85

[![★][connect-sqlite3-image] connect-sqlite3][connect-sqlite3-url] A [SQLite3](https://github.com/mapbox/node-sqlite3) session store modeled after the TJ's `connect-redis` store.

[connect-sqlite3-url]: https://www.npmjs.com/package/connect-sqlite3
[connect-sqlite3-image]: https://img.shields.io/github/stars/rawberg/connect-sqlite3.svg?label=%E2%98%85

[![★][documentdb-session-image] documentdb-session][documentdb-session-url] A session store for Microsoft Azure's [DocumentDB](https://azure.microsoft.com/en-us/services/documentdb/) NoSQL database service.

[documentdb-session-url]: https://www.npmjs.com/package/documentdb-session
[documentdb-session-image]: https://img.shields.io/github/stars/dwhieb/documentdb-session.svg?label=%E2%98%85

[![★][express-nedb-session-image] express-nedb-session][express-nedb-session-url] A NeDB-based session store.

[express-nedb-session-url]: https://www.npmjs.com/package/express-nedb-session
[express-nedb-session-image]: https://img.shields.io/github/stars/louischatriot/express-nedb-session.svg?label=%E2%98%85

[![★][express-session-cache-manager-image] express-session-cache-manager][express-session-cache-manager-url]
A store that implements [cache-manager](https://www.npmjs.com/package/cache-manager), which supports
a [variety of storage types](https://www.npmjs.com/package/cache-manager#store-engines).

[express-session-cache-manager-url]: https://www.npmjs.com/package/express-session-cache-manager
[express-session-cache-manager-image]: https://img.shields.io/github/stars/theogravity/express-session-cache-manager.svg?label=%E2%98%85

[![★][express-session-level-image] express-session-level][express-session-level-url] A [LevelDB](https://github.com/Level/levelup) based session store.

[express-session-level-url]: https://www.npmjs.com/package/express-session-level
[express-session-level-image]: https://img.shields.io/github/stars/tgohn/express-session-level.svg?label=%E2%98%85

[![★][express-etcd-image] express-etcd][express-etcd-url] An [etcd](https://github.com/stianeikeland/node-etcd) based session store.

[express-etcd-url]: https://www.npmjs.com/package/express-etcd
[express-etcd-image]: https://img.shields.io/github/stars/gildean/express-etcd.svg?label=%E2%98%85

[![★][express-session-etcd3-image] express-session-etcd3][express-session-etcd3-url] An [etcd3](https://github.com/mixer/etcd3) based session store.

[express-session-etcd3-url]: https://www.npmjs.com/package/express-session-etcd3
[express-session-etcd3-image]: https://img.shields.io/github/stars/willgm/express-session-etcd3.svg?label=%E2%98%85

[![★][firestore-store-image] firestore-store][firestore-store-url] A [Firestore](https://github.com/hendrysadrak/firestore-store)-based session store.

[firestore-store-url]: https://github.com/hendrysadrak/firestore-store
[firestore-store-image]: https://img.shields.io/github/stars/hendrysadrak/firestore-store.svg?label=%E2%98%85

[![★][fortune-session-image] fortune-session][fortune-session-url] A [Fortune.js](https://github.com/fortunejs/fortune)
based session store. Supports all backends supported by Fortune (MongoDB, Redis, Postgres, NeDB).

[fortune-session-url]: https://www.npmjs.com/package/fortune-session
[fortune-session-image]: https://img.shields.io/github/stars/aliceklipper/fortune-session.svg?label=%E2%98%85

[![★][hazelcast-store-image] hazelcast-store][hazelcast-store-url] A Hazelcast-based session store built on the [Hazelcast Node Client](https://www.npmjs.com/package/hazelcast-client).

[hazelcast-store-url]: https://www.npmjs.com/package/hazelcast-store
[hazelcast-store-image]: https://img.shields.io/github/stars/jackspaniel/hazelcast-store.svg?label=%E2%98%85

[![★][level-session-store-image] level-session-store][level-session-store-url] A LevelDB-based session store.

[level-session-store-url]: https://www.npmjs.com/package/level-session-store
[level-session-store-image]: https://img.shields.io/github/stars/scriptollc/level-session-store.svg?label=%E2%98%85

[![★][medea-session-store-image] medea-session-store][medea-session-store-url] A Medea-based session store.

[medea-session-store-url]: https://www.npmjs.com/package/medea-session-store
[medea-session-store-image]: https://img.shields.io/github/stars/BenjaminVadant/medea-session-store.svg?label=%E2%98%85

[![★][memorystore-image] memorystore][memorystore-url] A memory session store made for production.

[memorystore-url]: https://www.npmjs.com/package/memorystore
[memorystore-image]: https://img.shields.io/github/stars/roccomuso/memorystore.svg?label=%E2%98%85

[![★][mssql-session-store-image] mssql-session-store][mssql-session-store-url] A SQL Server-based session store.

[mssql-session-store-url]: https://www.npmjs.com/package/mssql-session-store
[mssql-session-store-image]: https://img.shields.io/github/stars/jwathen/mssql-session-store.svg?label=%E2%98%85

[![★][nedb-session-store-image] nedb-session-store][nedb-session-store-url] An alternate NeDB-based (either in-memory or file-persisted) session store.

[nedb-session-store-url]: https://www.npmjs.com/package/nedb-session-store
[nedb-session-store-image]: https://img.shields.io/github/stars/JamesMGreene/nedb-session-store.svg?label=%E2%98%85

[![★][restsession-image] restsession][restsession-url] Store sessions utilizing a RESTful API

[restsession-url]: https://www.npmjs.com/package/restsession
[restsession-image]: https://img.shields.io/github/stars/jankal/restsession.svg?label=%E2%98%85

[![★][sequelstore-connect-image] sequelstore-connect][sequelstore-connect-url] A session store using [Sequelize.js](http://sequelizejs.com/).

[sequelstore-connect-url]: https://www.npmjs.com/package/sequelstore-connect
[sequelstore-connect-image]: https://img.shields.io/github/stars/MattMcFarland/sequelstore-connect.svg?label=%E2%98%85

[![★][session-file-store-image] session-file-store][session-file-store-url] A file system-based session store.

[session-file-store-url]: https://www.npmjs.com/package/session-file-store
[session-file-store-image]: https://img.shields.io/github/stars/valery-barysok/session-file-store.svg?label=%E2%98%85

[![★][session-pouchdb-store-image] session-pouchdb-store][session-pouchdb-store-url] Session store for PouchDB / CouchDB. Accepts embedded, custom, or remote PouchDB instance and realtime synchronization.

[session-pouchdb-store-url]: https://www.npmjs.com/package/session-pouchdb-store
[session-pouchdb-store-image]: https://img.shields.io/github/stars/solzimer/session-pouchdb-store.svg?label=%E2%98%85

[![★][session-rethinkdb-image] session-rethinkdb][session-rethinkdb-url] A [RethinkDB](http://rethinkdb.com/)-based session store.

[session-rethinkdb-url]: https://www.npmjs.com/package/session-rethinkdb
[session-rethinkdb-image]: https://img.shields.io/github/stars/llambda/session-rethinkdb.svg?label=%E2%98%85

[![★][sessionstore-image] sessionstore][sessionstore-url] A session store that works with various databases.

[sessionstore-url]: https://www.npmjs.com/package/sessionstore
[sessionstore-image]: https://img.shields.io/github/stars/adrai/sessionstore.svg?label=%E2%98%85

## Example

Простой пример использования `express-session` для сохранения просмотров страницы 
для пользователя.

```js
var express = require('express')
var parseurl = require('parseurl')
var session = require('express-session')

var app = express()

app.use(session({
  secret: 'keyboard cat',
  resave: false,
  saveUninitialized: true
}))

app.use(function (req, res, next) {
  if (!req.session.views) {
    req.session.views = {}
  }

  // get the url pathname
  var pathname = parseurl(req).pathname

  // count the views
  req.session.views[pathname] = (req.session.views[pathname] || 0) + 1

  next()
})

app.get('/foo', function (req, res, next) {
  res.send('you viewed this page ' + req.session.views['/foo'] + ' times')
})

app.get('/bar', function (req, res, next) {
  res.send('you viewed this page ' + req.session.views['/bar'] + ' times')
})
```

## License

[MIT](LICENSE)

[npm-image]: https://img.shields.io/npm/v/express-session.svg
[npm-url]: https://npmjs.org/package/express-session
[travis-image]: https://img.shields.io/travis/expressjs/session/master.svg
[travis-url]: https://travis-ci.org/expressjs/session
[coveralls-image]: https://img.shields.io/coveralls/expressjs/session/master.svg
[coveralls-url]: https://coveralls.io/r/expressjs/session?branch=master
[downloads-image]: https://img.shields.io/npm/dm/express-session.svg
[downloads-url]: https://npmjs.org/package/express-session
