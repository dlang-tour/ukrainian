# Алгоритми діапазонів

Стандартні модулі [std.range](http://dlang.org/phobos/std_range.html) і
[std.algorithm](http://dlang.org/phobos/std_algorithm.html) забезпечують
безліч чудових функцій, які можуть поєднуватись для реалізації складних
операцій у доволі читабельний вигляд, основуючись на *діапазонах* в
якості будівельних блоків.

Найбільша перевага цих алгоритмів у тому, що від вас вимагається лише
визначити свій власний діапазон. Потім зможете насолоджуватися усіма
перевагами стандартної бібліотеки.

### std.algorithm

`filter` - лямбда, подана в якості параметра шаблону, генерує новий
діапазон, здатний фільтрувати елементи:

    filter!"a > 20"(range);
    filter!(a => a > 20)(range);

`map` - генерує новий діапазон, використовуючи предикат, визначений як
параметр шаблону:

    [1, 2, 3].map!(x => to!string(x));

`each` - аналог функції `foreach` для перебору діапазонів:

    [1, 2, 3].each!(a => writeln(a));

### std.range

`take` - обмежує до *N* елементів:

    theBigBigRange.take(10);

`zip` - виконує паралельно два і більше діапазонів, повертаючи кортеж
з обох діапазонів під час ітерації:

    assert(zip([1,2], ["hello","world"]).front
      == tuple(1, "hello"));

`generate` - на основі функції створює діапазон,
який викликається під час кожної ітерації, наприклад:

    alias RandomRange = generate!(x => uniform(1, 1000));

`cycle` - повертає діапазон, який завжди повторюватиме заданий вхідний
діапазон.

    auto c = cycle([1]);
    // діапазон ніколи не буде порожнім!
    assert(!c.empty);

### Документація чекає вашого візиту!


### Поглиблення

- [Діапазони у _Programming in D_](http://ddili.org/ders/d.en/ranges.html)
- [Більше про діапазони у _Programming in D_](http://ddili.org/ders/d.en/ranges_more.html)

## {SourceCode}

```d
// Агов, давай, просто бери всю цю армію!
import std.algorithm : canFind, map,
  filter, sort, uniq, joiner, chunkBy, splitter;
import std.array : array, empty;
import std.range : zip;
import std.stdio : writeln;
import std.string : format;

void main()
{
    string text = q{This tour will give you an
overview of this powerful and expressive systems
programming language which compiles directly
to efficient, *native* machine code.};

    // розщеплення на предикат
    alias pred = c => canFind(" ,.\n", c);
    // як будь-який хороший алгоритм,
    // це працює ліниво без виділення
    // пам'яті!
    auto words = text.splitter!pred
      .filter!(a => !a.empty);

    auto wordCharCounts = words
      .map!"a.count";

	// Виводить кількість символів
    // за слово у хорошому способі
    // починаючи з найменшої
    // кількості символів!
    zip(wordCharCounts, words)
      // перетворюємо у масив для сортування
      .array()
      .sort()
      // нам не потрібно дублювання,
      // чи не так?
      .uniq()
      // поміщаємо все в один ряд, який
      // має таку ж кількість символів.
      // chunkBy допомагає тут нам
      // генеруючи діапазон діапазонів,
      // які фрагментовані по довжині
      .chunkBy!(a => a[0])
      // ці елементи будуть об'єднані
      // в одному рядку
      .map!(chunk => format("%d -> %s",
          chunk[0],
          // тільки слова
          chunk[1]
            .map!(a => a[1])
            .joiner(", ")))
      // з'єднуємо назад, але ліниво!
      .joiner("\n")
      // та виводимо у консоль
      .writeln();
}
```
