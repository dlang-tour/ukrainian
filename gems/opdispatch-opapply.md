# opDispatch & opApply

Мова D дозволяє перевизначати такі оператори, як `+`, `-` або оператори
виклику `()` для [класів та структур](https://dlang.org/spec/operatoroverloading.html).
Ми трохи ближче познайомимось з двома спеціальними операторами
для перевантаження `opDispatch` і `opApply`.

### opDispatch

`opDispatch` може бути визначений як функція `структури` чи `класу`.
Будь-який виклик функції до цього типу передається до `opDispatch`
разом із ім'ям такої невідомої функції, як параметр шаблону `string`.
`opDispatch` є *всеохоплюючою* функцією, яка забезпечує ще один рівень
загального програмування, який повністю розраховується під **час компіляції**!

    struct C {
        void callA(int i, int j) { ... }
        void callB(string s) { ... }
    }
    struct CallLogger(C) {
        C content;
        void opDispatch(string name, T...)(T vals) {
            writeln("called ", name);
            mixin("content." ~ name)(vals);
        }
    }
    CallLogger!C l;
    l.callA(1, 2);
    l.callB("ABC");

### opApply

Альтернативний спосіб реалізації циклу `foreach` окрім визначеного
користувачем *діапазону*, є реалізація `opApply`. Виконання інтерації
з `foreach` над таким типом викликатиме перенавантажений оператор
`opApply` зі спеціальним делегатом, як параметр:

    class Tree {
        Tree lhs;
        Tree rhs;
        int opApply(int delegate(Tree) dg) {
            if (lhs && lhs.opApply(dg)) return 1;
            if (dg(this)) return 1;
            if (rhs && rhs.opApply(dg)) return 1;
            return 0;
        }
    }
    Tree tree = new Tree;
    foreach(node; tree) {
        ...
    }

Компілятор трансформує тіло `foreach` у спеціального делегата, який
передаватиметься об'єкту. Його один і єдиний параметр міститиме поточне
значення ітерації. Магічне повернення значення `int` повинно бути
інтерпретовано і якщо воно не дорівнює `0`, ітерацію потрібно зупинити.

### Поглиблення

- [Перевантаження операторів у _Programming in D_](http://ddili.org/ders/d.en/operator_overloading.html)
- [`opApply` у _Programming in D_](http://ddili.org/ders/d.en/foreach_opapply.html)
- [Перевантаження операторів у D](https://dlang.org/spec/operatoroverloading.html)

## {SourceCode}

```d
/*
Variant - це те, що може містити
будь-який інший тип:
https://dlang.org/phobos/std_variant.html
*/

import std.variant : Variant;

/*
Тип, у якому перевизначено opDispatch
для можливості роботи з будь-якою кількістю
членів. Так само як var у JavaScript.
*/
struct var {
    private Variant[string] values;

    @property
    Variant opDispatch(string name)() const {
        return values[name];
    }

    @property
    void opDispatch(string name, T)(T val) {
        values[name] = val;
    }
}

void main() {
    import std.stdio : writeln;

    var test;
    test.foo = "test";
    test.bar = 50;
    writeln("test.foo = ", test.foo);
    writeln("test.bar = ", test.bar);
    test.foobar = 3.1415;
    writeln("test.foobar = ", test.foobar);
    // ПОМИЛКА, тому що це вже не існує
    // writeln("test.notthere = ",
    //   test.notthere);
}
```

