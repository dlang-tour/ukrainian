# Підключення бази даних

Vibe.d полегшує доступ до баз даних у ваших серверних службах.
Підтримка **MongoDB** і ** **Redis** надходить безпосередньо з vibe.d,
у той час як інші адаптери баз даних можна знайти на
[code.dlang.org](https://code.dlang.org).

### MongoDB

Доступ до MongoDB моделюється за допомогою класу [`MongoClient`](http://vibed.org/api/vibe.db.mongo.client/MongoClient).
Реалізація не має зовнішніх залежностей і здійснюється за допомогою
асинхронних сокетів для vibe.d. Немає необхідності у блокуванні, якщо
з'єднання здійснюється із затримкою.

    auto client = connectMongoDB("127.0.0.1");
    auto users = client.getCollection("users");
    users.insert(Bson("peter"));

### Redis

Підтримка Redis так само реалізована за допомогою сокетів для vibe.d
за відсутності зовнішніх залежностей. Основним для реалізації є
[`RedisDatabase`](http://vibed.org/api/vibe.db.redis.redis/RedisDatabase)
клас, який дозволяє відправляти команди до сервера Redis. Також
доступна зручна обгортка, така як [`RedisList`](http://vibed.org/api/vibe.db.redis.types/RedisList).
Вона дозволяє прозоро отримати доступ до списку, що зберігається у Redis.

### MySQL

Підтримка MySQL без зовнішніх залежностей може бути додана до офіційної
бібліотеки MySQL за допомогою [mysql-native](http://code.dlang.org/packages/mysql-native)
проекту. Він також підтримує неблокуючі сокети для vibe.d.

### PostgreSQL

Повнофункціональний клієнт PostgreSQL реалізовано за допомогою
зовнішнього модулемя [dpq2](http://code.dlang.org/packages/dpq2), який
базується на офіційній бібліотеці *libpq*. Він використовує систему
подій vibe.d для реалізації асинхронної поведінки.

Альтернативою для PostgreSQL є [DDB](http://code.dlang.org/packages/ddb),
який реалізує клієнт PostgreSQL за допомогою сокета vibe.d за
відсутності зовнішніх залежностей.
