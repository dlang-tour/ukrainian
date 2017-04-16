# JSON REST Інтерфейс

Vibe.d дозволяє швидко реалізувати веб-сервіс JSON. Якщо ми хочемо 
реалізувати наступний формат JSON для запиту HTTP, треба звернутися
до `/api/v1/chapters`: 

    [
      {
        "title": "Hello",
        "id": 1,
        "sections": [
          {
            "title": "World",
            "id": 1
          }
        ]
      },
      {
        "title": "Advanced",
        "id": 2,
        "sections": []
      }
    ]

Спочатку слід визначити інтерфейс, який реалізує відповідні функції і D
`структури`, які серіалізовані за принципом **1:1**: 

    interface IRest
    {
        struct Section {
            string title;
            int id;
        }
        struct Chapter {
            string title;
            int id;
            Section[] sections;
        }
        @path("/api/v1/chapters")
        Chapter[] getChapters();
    }

Для фактичного заповнення структур даних, необхідно слідувати
бізнес-логіці, запровадженій інтерфейсом:

    class Rest: IRest {
        Chapter[] getChapters() {
          // заповнити
        }
    }

Надсилаючи копію до `URLRouter`, ми реєструємо копію класу `Rest` і
готово!

    auto router = new URLRouter;
    router.registerRestInterface(new Rest);

Vibe.d *генератор інтерфейсу REST* також підтримує запит POST, де
дочірні модулі опублікованого об'єкту JSON зіставляються з параметрами
функції.

REST інтерфейс може бути використаний для створення копії клієнта REST,
який прозоро відправляє запити JSON до відповідного сервера,
використовуючи ті ж самі функції, що і на стороні серверної.
Це вигідно тоді, коли код розподілений між клієнтом і сервером.

    auto api = new RestInterfaceClient!IRest("http://127.0.0.1:8080/");
    // відправляє GET /api/v1/chapters і десеріалізує
    // відповідь у IRest.Chapter[]
    auto chapters = api.getChapters();

## {SourceCode:disabled}

```d
import vibe.d;

interface IRest
{
    struct Section
    {
        string title;
        int id;
    }

    struct Chapter
    {
        string title;
        int id;
        Section[] sections;
    }

    @path("/api/v1/chapters")
    Chapter[] getChapters();

    /*
    Відправляє дані як:
        { "title": "D Language" }
    */
    @path("/api/v1/add-chapter")
    @method(HTTPMethod.POST)
    int addChapter(string title);
}

class Rest: IRest
{
    private Chapter[] chapters_;

    this()
    {
        chapters_ = [ Chapter("Hello", 1,
                [ Section("World", 1) ] ),
                 Chapter("Advanced", 2) ];
    }

    Chapter[] getChapters()
    {
        return chapters_;
    }

    int addChapter(string title)
    {
        import std.algorithm : map, max, reduce;
        // Сформувати наступний ідентифікатор
        auto newId = chapters_.map!(x => x.id)
                            .reduce!max + 1;
        chapters_ ~= Chapter(title, newId);
        return newId;
    }
}

shared static this()
{
    auto router = new URLRouter;
    router.registerRestInterface(new Rest);

    auto settings = new HTTPServerSettings;
    settings.port = 8080;
    listenHTTP(settings, router);
}
```
