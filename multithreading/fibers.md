# Нитки (Fibers)

**Нитки** - спосіб реалізації паралелізму шляхом *співробітництва*. Визначення
класу `Fiber` наведено у модулі [`core.thread`](https://dlang.org/phobos/core_thread.html).

Основна ідея полягає у тому, що коли нитка залишається без діла або
очікує на додаткові вхідні дані, вона *добровільно* передає можливість
виконувати інструкції іншим ниткам, викликаючи функцію `Fiber.yield()`.
Батьківський контекст знову отримує управління, але стан нитки,
а саме всі змінні на стеку, зберігається. Пізніше нитка може бути відновлена і продовжити
свою роботу з інструкції, яка слідує одразу ж за викликом `Fiber.yield()`.
Магія? Саме так!

    void foo() {
        writeln("Привіт");
        Fiber.yield();
        writeln("Світ");
    }
    // ...
    auto f = new Fiber(&foo);
    f.call(); // Друкує "Привіт"
    f.call(); // Друкує "Світ"

Ця можливість може бути використана для реалізації паралелізму, де безліч
ниток спільно використовують одне ядро. Перевага ниток у порівнянні
з потоками полягає у більш економному використанні ресурсів завдяки
відсутності зміни контексту.

Вдале використання цієї техніки можна побачити у фреймворку [vibe.d](http://vibed.org), 
в якому реалізовані неблокуючі (або асинхронні) операції вводу/виводу у
контексті ниток, що дозволяє створювати більш лаконічний код.

### Поглиблення

- [Нитки у _Programming in D_](http://ddili.org/ders/d.en/fibers.html)
- [Документація по core.thread.Fiber](https://dlang.org/library/core/thread/fiber.html)

## {SourceCode}

```d
import core.thread : Fiber;
import std.stdio : write;
import std.range : iota;

/**
Виконуємо ітерацію по `range` і
застосовуємо функцію `Fnc` для кожного
елементу x, щоб отримати `result`.
Нитка передає керування іншим
ниткам після кожного застосування.
*/
void fiberedRange(alias Fnc, R, T)(
    R range,
    ref T result)
{
    for(; !range.empty; range.popFront) {
        result = Fnc(range.front);
        Fiber.yield();
    }
}

void main()
{
    int squareResult, cubeResult;
    // Створюємо нитку, що починається
    // з делегату, який формує квадратний
    // діапазон.
    auto squareFiber = new Fiber({
        fiberedRange!(x => x*x)(
            iota(1,11), squareResult);
    });
    // ..  і тут створюємо куби!
    auto cubeFiber = new Fiber({
        fiberedRange!(x => x*x*x)(
            iota(1,9), cubeResult);
    });

    // стан TERM означає, що нитка закінчила
    // виконання, що пов'язане з її функцією.
    squareFiber.call();
    cubeFiber.call();
    while (squareFiber.state
        != Fiber.State.TERM && cubeFiber.state
        != Fiber.State.TERM)
    {
        write(squareResult, " ", cubeResult);
        squareFiber.call();
        cubeFiber.call();
        write("\n");
    }

    // squareFiber все ще може бути задіяна,
    // тому що її функціонування не було
    // завершене!
}
```
