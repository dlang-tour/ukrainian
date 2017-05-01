# Передача повідомлень

Замість того, щоб мати справу з потоками і синхронізацією,
програмування D дозволяє використовувати *передачу повідомлень* для
задіяння потужності кількох ядер. Потоки використовуються для
передачі *повідомлень*, тобто довільних значень, при розподіленні
роботи та синхронізації. Таким чином, немає навмисного розподілу
даних, що дозволяє уникнути типових проблем багатопоточності.

Усі функції, які реалізують передачу повідомлень у програмуванні D,
можуть бути знайдені у модулі [`std.concurrency`](https://dlang.org/phobos/std_concurrency.html).
`spawn` створює новий *потік* оснований на визначеній користувачем функції:

    auto threadId = spawn(&foo, thisTid);

`thisTid` є вбудованою функцією `std.concurrency`, яка посилається на
існуючий потік, необхідний для передачі повідомлень. `spawn` виконує
функцію першого параметра, а додаткові параметри цієї функції
виступають в якості аргументів.

    void foo(Tid parentTid) {
        receive(
          (int i) { writeln("Значення ", i, " було відправлене!"); }
        );
        
        send(parentTid, "Готово");
    }

Будучи подібною до функції `switch`-`case`, функція `receive` передає
значення, отримані з інших потоків до потрібних делегатів (`delegates`)
- в залежності від типу отриманого значення.

Для того щоб відправити повідомлення до певного потоку, необхідно
застосувати функцію `send` разом із її id:

    send(threadId, 42);

`receiveOnly` може бути використаний для досягнення певного виду:

    string text = receiveOnly!string();
    assert(text == "Готово");

Функції родини `receive` блокуються доти, доки певні дані не
відправляться до поштової скриньки потоку.

### Поглиблення

- [Обмін повідомленнями між потоками](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=5)
- [Паралелізм передачі повідомлень](http://ddili.org/ders/d.en/concurrency.html)
- [Шаблон, підходящий для отримання](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=6)
- [Багатопоточне копіювання файлів](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=7)
- [Завершення потоку](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=8)
- [Екстренна комунікація](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=9)
- [Переповнення поштової скриньки](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=10)

## {SourceCode}

```d
import std.stdio : writeln;
import std.concurrency : receive, receiveOnly,
    send, spawn, thisTid, Tid;

/*
Призначена для користувача структура
використовується у якості повідомлення
для армії маленьких потоків.
*/
struct NumberMessage {
    int number;
    this(int i) {
        this.number = i;
    }
}

/*
Повідомлення використовуються як
знак стоп для інших потоків
*/
struct CancelMessage {
}

/*
Підтвердження перед викликом
CancelMessage
*/
struct CancelAckMessage {
}

/*
Основна функція робочого потоку
із батьківським id передається
як аргумент.
*/
void worker(Tid parentId)
{
    bool canceled = false;
    writeln("Початок ", thisTid, "...");

    while (!canceled) {
      receive(
        (NumberMessage m) {
          writeln("Отримано int: ", m.number);
        },
        (string text) {
          writeln("Отримано string: ", text);
        },
        (CancelMessage m) {
          writeln("Завершення ", thisTid, "...");
          send(parentId, CancelAckMessage());
          canceled = true;
        }
      );
    }
}

void main()
{
    Tid threads[];
    //  Створюємо 10 маленьких робочих потоків
    for (size_t i = 0; i < 10; ++i) {
        threads ~= spawn(&worker, thisTid);
    }

    // Непарні потоки отримують номер,
    // парні потоки отримують рядок!
    foreach(int idx, ref tid; threads) {
        import std.string : format;
        if (idx  % 2)
            send(tid, NumberMessage(idx));
        else
            send(tid, format("T=%d", idx));
    }

    // Усі потоки отримують повідомлення
    // про відкликання!
    foreach(ref tid; threads) {
        send(tid, CancelMessage());
    }

    // Необхідно чекати доти, поки всі потоки 
    // підтвердять запит щодо зупинки
    foreach(ref tid; threads) {
        receiveOnly!CancelAckMessage;
        writeln("Отримано CancelAckMessage!");
    }
}
```
