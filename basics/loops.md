# Цикли

D має чотири конструкції для організації циклів.

### 1) `while`

Цикл `while` виконує блок коду, поки задана умова вірна:

    while (умова) {
        foo();
    }

### 2) `do ... while`

Цикл `do .. while` виконує блок коду, поки задана умова вірна, але
на відміну від `while` _тіло циклу_ виконується ще до того, як
умова перевіриться вперше (_тіло циклу_ виконається мінімум один раз).

    do {
        foo();
    } while (умова);

### 3) Класичний цикл `for`

Класичний цикл `for` відомий з мов C/C++ та Java, який включає
_ініціалізатор_, _умову циклу_ та _оператор циклу_:

    for (int i = 0; i < arr.length; i++) {
        ...

### 4) `foreach`

[Цикл `foreach`](basics/foreach) ми більш детально розглянемо у
наступному розділі.

    foreach (el; arr) {
        ...
    }

#### Спеціальні ключові слова та мітки

Спеціальне ключове слово `break` негайно перериває поточний цикл.
У вкладених циклах можна використовувати _мітки_ для переривання
будь-якого зовнішнього циклу:

    outer: for (int i = 0; i < 10; ++i) {
        for (int j = 0; j < 5; ++j) {
            ...
            break outer;

Ключове слово `continue` запускає наступну ітерацію циклу.

### Додатково

- Цикли `for` у [_Programming in D_](http://ddili.org/ders/d.en/for.html), [специфікація](https://dlang.org/spec/statement.html#ForStatement)
- Цикли `while` у [_Programming in D_](http://ddili.org/ders/d.en/while.html), [специфікація](https://dlang.org/spec/statement.html#WhileStatement)
- Цикли `do-while` у [_Programming in D_](http://ddili.org/ders/d.en/do_while.html), [специфікація](https://dlang.org/spec/statement.html#do-statement)

## {SourceCode}

```d
import std.stdio : writeln;

/*
  Обчислює середнє число елементів масиву.
*/
double average(int[] array)
{
    immutable initialLength = array.length;
    double accumulator = 0.0;

    while (!array.empty)
    {
        // також це можна зробити за
        // допомогою .front, зробивши
        // import std.array : front;
        accumulator += array[0];
        array = array[1 .. $];
    }

    return accumulator / initialLength;
}

void main()
{
    auto testers = [ [5, 15], // 20
          [2, 3, 2, 3], // 10
          [3, 6, 2, 9] ]; // 20

    for (auto i = 0; i < testers.length; ++i)
    {
      writeln("The average of ", testers[i],
        " = ", average(testers[i]));
    }
}
```
