# Foreach

{{#img-right}}dman-teacher-foreach.jpg{{/img-right}}

Цикл `foreach` у D дозволяє зменшити потенційні помилки та поліпшити
читабельність коду при переборі елементів.

### Перебір елементів

Для заданого масиву `arr` типу `int[]` є можливість перебрати всі його
елементи, використавши цикл `foreach`:

    foreach (int e; arr) {
        writeln(e);
    }

Перше поле у оголошенні `foreach` - це змінна, яка використовується у
тілі циклу. Її тип обчислюється автоматично:

    foreach (e; arr) {
        // typeof(e) поверне int
        writeln(e);
    }

Друге поле повинно бути масивом, або спеціальним підтримуючим перебір
об'єктом, який називається **range** (діапазон), про який буде
розказано у [наступному розділі](basics/ranges).

### Доступ за посиланням

Під час перебору елементи масиву або діапазону будуть скопійовані.
Це допустимо для основних типів, але може стати проблемою для великих
типів. Для запобігання копіювання або щоб мати можливість змінювати
оригінальні елементи, можна використовувати `ref`:

    foreach (ref e; arr) {
        e = 10; // перезаписуємо оригінальне значення
    }

### Зворотний перебір за допомогою `foreach_reverse`

Колекцію можна перебрати у зворотному порядку за допомогою
`foreach_reverse`:

    foreach_reverse (e; [1, 2, 3]) {
        writeln(e);
    }
    // 3 2 1

### Додатково

- [`foreach` у _Programming in D_](http://ddili.org/ders/d.en/foreach.html)
- [`foreach` з структурами та класами у _Programming in D_](http://ddili.org/ders/d.en/foreach_opapply.html)
- [`foreach` специфікація](https://dlang.org/spec/statement.html#ForeachStatement)

## {SourceCode}

```d
import std.stdio;

void main() {
    auto arr = [ [5, 15], // 20
          [2, 3, 2, 3], // 10
          [3, 6, 2, 9] ]; // 20

    // Перебір масиву в зворотному порядку
    import std.range: retro;
    foreach (row; retro(arr))
    {
        double accumulator = 0.0;
        foreach (c; row)
            accumulator += c;

        writeln("The average of ", row,
            " = ", accumulator / row.length);
    }
}
```
