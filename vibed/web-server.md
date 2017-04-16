# Веб-сервер

Vibe.d дозволяє створювати веб-сервери HTTP(S) за короткий проміжок часу: 

    auto settings = new HTTPServerSettings;
    settings.port = 8080;
    listenHTTP(settings, &foo);

Це дозволяє запустити веб-сервер на порті 8080, де всі запити
обробляються за допомогою функції `foo`:

    void foo(HTTPServerRequest req,
        HTTPServerResponse res) { ... }

Для того, щоб спростити стандартні моделі та конфігурації, існує
клас `URLRouter`, який реєструє `GET`, `POST` та інші
обробники з використанням функцій `.get("path", handler)` і
`.post("path", handler)`, або реєструє власний клас *веб-інтерфейсу*,
який реалізує шляхи веб-серверу, як функції:

    auto router = new URLRouter;
    router.registerWebInterface(new WebService);
    listenHTTP(settings, router);

Шляхи функцій веб-інтерфейсу `WebService` будуть автоматично виведені
за допомогою простої схеми:
* `index()` опрацює `/index`
* `getName()` опрацює `GET` запит до `/name`
* `postUsername()` опрацює `POST` запит до `/username`

Шлях користувача може бути встановлений за допомогою функції
`@path("/hello/world")`. Параметри для запитів `POST` будуть доступні
для функцій, що використовують імена змінних із префіксом `_`. Також
можна вказати параметри безпосередньо у самому шляху:

    @path("/my/api/:id")
    void foo(int _id)

Вам не потрібно передавати об'єкти `HTTPServerResponse` та
`HTTPServerRequest`, як це необхідно було робити із кожною функцією.
Vibe.d статично перевіряє, чи є котрийсь об'єкт у списку параметрів
функції та просто передає дані за необхідністю.

## {SourceCode:disabled}

```d
import vibe.d;

class WebService
{
    /*    
    При використанні змінних сесії, таких
    як ці, інформація, пов'язана з окремими
    користувачами, може зберігатися у всіх
    запитах на час існування сесії.
    */
    private SessionVar!(string, "username")
        username_;

    /*
    За замовчуванням, запити до кореневого
    шляху ("/") зв’язані з методом index.
    */
    void index(HTTPServerResponse res)
    {
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <form action="/username" method="POST">
        Your name:
        <input type="text" name="username">
        <input type="submit" value="Submit">
        </form>
        </body>
        </html>};

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }

    /*
    Атрибут @path може бути використаний для
    налаштування маршрутизації URL. Таким
    чином, запити до "/name" будуть
    співставлені з методом getName.
    */
    @path("/name")
    void getName(HTTPServerRequest req,
            HTTPServerResponse res)
    {
        import std.string : format;

        // Генеруємо теги з даними
        // заголовка <li> перевіряючи
        // властивості заголовка запиту.
        string[] headers;
        foreach(key, value; req.headers) {
            headers ~=
                "<li>%s: %s</li>"
                .format(key, value);
        }
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <h1>Your name: %s</h1>
        <h2>Headers</h2>
        <ul>
        %s
        </ul>
        </body>
        </html>}.format(username_,
                headers.join("\n"));

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }

    void postUsername(string username,
            HTTPServerResponse res)
    {
        username_ = username;
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <h1>Ваше ім'я: %s</h1>
        </body>
        </html>}.format(username_);

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }
}

void helloWorld(HTTPServerRequest req,
        HTTPServerResponse res)
{
    res.writeBody("Hello");
}

shared static this()
{
    auto router = new URLRouter;
    router.registerWebInterface(new WebService);
    router.get("/hello", &helloWorld);

    auto settings = new HTTPServerSettings;
    // Потрібно для використання SessionVar.
    settings.sessionStore =
        new MemorySessionStore;
    settings.port = 8080;
    listenHTTP(settings, router);
}
```
