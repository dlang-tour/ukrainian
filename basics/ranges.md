# Дiапазони

Коли компiлятор зустрічає вираз `foreach`

    foreach(element; range) {

вiн на нижчому рівні перепиcує його в:

    for (; !range.empty; range.popFront()) {
        auto element = range.front;
        ...

Будь-який об'єкт, який імплементує вищезгаданий інтерфейс, називається
**діапазон** і може пiдтримувати ітерації:

    struct Range {
        @property empty() const;
        void popFront();
        T front();
    }

Функції у `std.range` і `std.algorithm` забезпечують будівельні блоки,
які використовують цей інтерфейс. Діапазони дозволяють створювати
складні алгоритми обробки об'єктiв, по яким може бути проведена
ітерація. Крім того, діапазони дозволяють створювати **ледачi**
об'єкти, які виконують обчислення тільки, коли це дійсно необхідно,
наприклад, коли наступний елемент діапазону є реально доступним.
Спецiальнi алгоритми для діапазонiв будуть описані пізніше у розділі
[Коштовності D](gems/range-algorithms).

### Вправа

Завершіть код та створіть дiапазон `FibonacciRange`, який повертає
[числа Фібоначчі](https://uk.wikipedia.org/wiki/%D0%9F%D0%BE%D1%81%D0%BB%D1%96%D0%B4%D0%BE%D0%B2%D0%BD%D1%96%D1%81%D1%82%D1%8C_%D0%A4%D1%96%D0%B1%D0%BE%D0%BD%D0%B0%D1%87%D1%87%D1%96).
Не вводьте себе в оману, видаляючи `assert`-и!

### Додатково

- [`std.algorithm`](http://dlang.org/phobos/std_algorithm.html)
- [`std.range`](http://dlang.org/phobos/std_range.html)

## {SourceCode:incomplete}

```d
import std.stdio;

struct FibonacciRange
{
    bool empty() const @property
    {
        // Так коли послiдовнiсть Фібоначчі
        // закiнчується?
    }

    void popFront()
    {
    }

    int front() @property
    {
    }
}

void main() {
    import std.range: take;
    import std.array: array;

    FibonacciRange fib;

    // `take` створює iнший дiапазон,
    // який повертає максимум N елементів.
    // Цей дiапазон є ледачим
    // та звертається до оригiнального
    // дiапазону, лише коли дiйсно потрiбно
    auto fib10 = take(fib, 10);

    // Але тут ми бажаємо отримати всі елементи
    // та перетворити дiапазон у масив цiлих
    // чисел.
    int[] the10Fibs = array(fib10);

    writeln("Перші 10 чисел Фібоначчі: ",
        the10Fibs);
    assert(the10Fibs ==
        [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]);
}
```
