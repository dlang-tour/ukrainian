# Керування потоком

Хід виконання програми можна контролювати за допомогою умовних
операторів `if` та `else`:

    if (a == 5) {
        writeln("Умова виконується");
    } else if (a > 10) {
        writeln("Інша умова виконується");
    } else {
        writeln("Нічого не виконується!");
    }

Якщо блок `if` чи `else` містить лише одну інструкцію, фігурні дужки
можуть бути опущені.

Для перевірки рівності змінних і їх порівняння D надає такі ж
оператори, як C/C++ і Java:

* `==` і `!=` для перевірки рівності і нерівності
* `<`, `<=`, `>` і `>=` для перевірки того, що значення менше
(- і рівне) та більше (- і рівне)

Умови можна комбінувати, використовуючи оператори `||` (Логічне *АБО*) та
`&&` (логічне *І*).

У D також є конструкція `switch`..`case`, яка вибирає одну гілку
виконання, в залежності від значення *однієї* змінної. `switch` працює
з усіма основними типами, а також з рядками!
Для цілих типів є можливість задавати діапазони, використавши
синтаксис `case START: .. case END:`. Не забудьте ознайомитись з
прикладом.

### Додатково

#### Основні посилання

- [Логічні вирази у _Programming in D_](http://ddili.org/ders/d.en/logical_expressions.html)
- [Умовний оператор if у _Programming in D_](http://ddili.org/ders/d.en/if.html)
- [Тернарний оператор у _Programming in D_](http://ddili.org/ders/d.en/ternary.html)
- [`switch` та `case` у _Programming in D_](http://ddili.org/ders/d.en/switch_case.html)

#### Додаткові посилання

- [Специфікація на вирази](https://dlang.org/spec/expression.html)
- [Специфікація на оператор if](https://dlang.org/spec/statement.html#if-statement)

## {SourceCode}

```d
import std.stdio;

void main()
{
    if (1 == 1)
        writeln("You can trust math in D");

    int c = 5;
    switch(c) {
        case 0: .. case 9:
            writeln(c, " is within 0-9");
            break; // необхідно!
        case 10:
            writeln("A Ten!");
            break;
        // якщо жодна умова не виконалась
        default:
            writeln("Nothing");
            break;
    }
}
```

