# Синхронізація та загальний доступ

У мові D найкращим способом створити багатопоточність є
покладання на `незмінні (immutable)` дані і синхронізування потоків,
використовуючи передачу повідомлень. Мова D має вбудовану підтримку
*синхронізації* примітивів, а також підтримку типу з ключовим
словом `загальнодоступні (shared)` для позначення доступних з
кількох потоків об'єктів.

Ідентифікатор типу `shared` дозволяє позначати змінні, які
використовуються різними потоками: 

    shared(int)* p = new int;
    int* t = p; // ПОМИЛКА

Наприклад, `std.concurrency.send` дозволяє відправляти `незмінні` або
`загальнодоступні` дані, а також копіювати повідомлення для відправки.
`Загальнодоступні` дані є транзитними, відповідно, якщо `класс` або
`структура` позначені як `shared`, всі їх члени будуть позначені
так само. Зверніть увагу, що змінні `static` не є `shared` за
замовчуванням і будуть реалізовані за допомогою
*локальної пам'яті потоку* (ЛПП), де кожен потік отримає власну змінну.

`synchronized` блоки дозволяють повідомити компілятору, що
дана секція є критичною і має бути доступна лише для одного
потоку в один проміжок часу.

    synchronized {
        importStuff();
    }

У межах `класу` ці блоки можуть бути обмежені різними об'єктами
*м'ютекс* із `synchronized(member1, member2)` для того, щоб обмежити
багатопоточність. Копмілятор D автоматично вставляє *критичні секції*.
Цілий клас також може бути позначений як `synchronized`, у той же час,
компілятор повинен переконатися, що тільки один потік має доступ до
певної копії.

Атомарні операції на змінних `shared` можуть бути реалізовані за
допомогою помічника `core.atomic.atomicOp`:

    shared int test = 5;
    test.atomicOp!"+="(4);

### Поглиблення

- [Паралелізм загальнодоступних даних у _Programming in D_](http://ddili.org/ders/d.en/concurrency_shared.html)
- [Кваліфікатор типу `shared`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=11)
- [Замкнена Синхронізація з `synchronized`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=13)
- [Взаємоблокування і функція `synchronized`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=15)
- [Специфікація `synchronized`](https://dlang.org/spec/statement.html#SynchronizedStatement)
- [Безмежні перетворення з даними типу `shared`](https://dlang.org/spec/const3.html#implicit_conversions)

## {SourceCode}

```d
import std.concurrency : receiveOnly, send,
    spawn, Tid, thisTid;
import core.atomic : atomicOp, atomicLoad;

/*
Черга може безпечно використовуватися
різними потоками. Весь доступ до копії
автоматично блокується ключовим словом
synchronized. 
*/
synchronized class SafeQueue(T)
{
    // Примітка: доступ має бути приватним
    // у синхронізованих класах, в іншому
    // випадку можна очікувати помилки.
    private T[] elements;

    void push(T value) {
        elements ~= value;
    }

    // Повертаємо T.init за відсутності
    // черги
    T pop() {
        import std.array : empty;
        T value;
        if (elements.empty)
            return value;
        value = elements[0];
        elements = elements[1 .. $];
        return value;
    }
}

/*
Безпечно виводимо повідомлення
незалежно від кількості паралельних
потоків. Зверніть увагу, що шаблонні
параметри використовуються для args,
які можуть виступати діапазоном
параметрів 0 .. N
*/
void safePrint(T...)(T args)
{
    // Доступ лише одним потоком одночасно
    synchronized {
        import std.stdio : writeln;
        writeln(args);
    }
}

void threadProducer(shared(SafeQueue!int) queue,
    shared(int)* queueCounter)
{
    import std.range : iota;
    // Вказуємо значення від 1 до 11
    foreach (i; iota(1,11)) {
        queue.push(i);
        safePrint("Pushed ", i);
        atomicOp!"+="(*queueCounter, 1);
    }
}

void threadConsumer(Tid owner,
    shared(SafeQueue!int) queue,
    shared(int)* queueCounter)
{
    int popped = 0;
    while (popped != 10) {
        auto i = queue.pop();
        if (i == int.init)
            continue;
        ++popped;

        // безпечно визначаємо поточне
        // значення queueCounter,
        // використовуючи atomicLoad
        safePrint("Popped ", i,
            " (Consumer pushed ",
            atomicLoad(*queueCounter), ")");
    }

    // Готово!
    owner.send(true);
}

void main()
{
    auto queue = new shared(SafeQueue!int);
    shared int counter = 0;
    spawn(&threadProducer, queue, &counter);
    auto consumer = spawn(&threadConsumer,
                    thisTid, queue, &counter);
    auto stopped = receiveOnly!bool;
    assert(stopped);
}
```
