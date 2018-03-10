# Контрактне програмування

Контрактне програмування D включає у себе набір мовних конструкцій,
які дозволяють підвищити якість коду шляхом впровадження перевірок
програми на відсутність очевидних помилок, що дає змогу переконатися,
що код поводить себе, як передбачалося. Контракти доступні тільки у
режимах **відлагодження** чи **тестування** та не будуть працювати у
режимі релізу. Саме тому вони не повинні використовуватися для
перевірки введених користувачем даних або як альтернативу обробки винятків.

### `assert`

Найпростіша форма контрактного програмування у D - це вираз `assert(...)`,
який перевіряє, чи певна умова виконана, а також перериває програму за
допомогою `AssertionError`, якщо умова не вірна.

    assert(sqrt(4) == 2);
    // опціонально можна вказати власну помилку для користувача
    assert(sqrt(16) == 4, "sqrt не працює!");

### Функціональні контракти

`in` та `out` дозволяють оформити контракти для перевірки вхідних
параметрів і результату функцій.

    long square_root(long x)
    in {
        assert(x >= 0);
    } out (result) {
        assert((result * result) <= x
            && (result+1) * (result+1) > x);
    } body {
        return cast(long)std.math.sqrt(cast(real)x);
    }

Зміст блоку `in` також можна перенести у тіло функції, але втрачається
очевидність. У блоці `out` повернення значення функції можна перехопити
за допомогою `out(result)` і перевірити коректність результату.

### Інваріантна перевірка

`invariant()` - це спеціальна функція `struct` та `class`, що дозволяє
контролювати стан об'єкта напротязі всього часу його життя:

* Вона викликається після виклику конструктора і перед викликом деструктора.
* Вона викликається перед виконанням функції
* `invariant()` викликається після завершення функції-члена.

### Перевірка даних, введених користувачем

Оскільки усі контракти будуть видалені у релізній збірці, дані, введені
користувачем, не повинні перевірятися за допомогою контрактів. При цьому,
`assert`-и можуть бути використані у `nothrow`-функціях, тому що
вони породжують фатальні помилки (`Error`).
Аналогом часу виконання для `assert`-ів є [`std.exception.enforce`](https://dlang.org/phobos/std_exception.html#.enforce),
який буде видавати виключення (`Exception`), які дозволяється перехоплювати.

### Поглиблення

- [`assert` та `enforce` у _Programming in D_](http://ddili.org/ders/d.en/assert.html)
- [Контрактне програмування у _Programming in D_](http://ddili.org/ders/d.en/contracts.html)
- [Контрактне програмування для структур і класів у _Programming in D_](http://ddili.org/ders/d.en/invariant.html)
- [Контрактне програмування у D spec](https://dlang.org/spec/contracts.html)
- [`std.exception`](https://dlang.org/phobos/std_exception.html)

## {SourceCode:incomplete}

```d
import std.stdio : writeln;

/**
  Спрощений тип Date
  Замість нього
  використовуйте std.datetime
*/
struct Date {
    private {
        int year;
        int month;
        int day;
    }

    this(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }

    invariant() {
        assert(year >= 1900);
        assert(month >= 1 && month <= 12);
        assert(day >= 1 && day <= 31);
    }

    /**
      Серіалізує об'єкт типу Date з рядка
      у форматі РРРР-ММ-ДД.

      Параметри:
          date = рядок, який потрібно
                 серіалізувати

      Повертає: об'єкт Date.
    */
    void fromString(string date)
    in {
        assert(date.length == 10);
    }
    body {
        import std.format : formattedRead;
        // formattedRead обробляє
        // формативаний рядок,
        // і повертає результат
        // через змінні
        formattedRead(date, "%d-%d-%d",
            &this.year,
            &this.month,
            &this.day);
    }

    /**
      Серіалізує об'єкт типу Date у рядок
      у форматі РРРР-ММ-ДД

      Повертає: рядкове представлення Date
    */
    string toString() const
    out (result) {
        import std.algorithm : all, count,
                              equal, map;
        import std.string : isNumeric;
        import std.array : split;

		// переконуємось, що повертається
        // формат РРРР-ММ-ДД
        assert(result.count("-") == 2);
        auto parts = result.split("-");
        assert(parts.map!`a.length`
                    .equal([4, 2, 2]));
        assert(parts.all!isNumeric);
    }
    body {
        import std.format : format;
        return format("%.4d-%.2d-%.2d",
                      year, month, day);
    }
}

void main() {
    auto date = Date(2016, 2, 7);

    // Таке введення викличе помилку в
    // інваріанті. Не перевіряйте
    // користувацьке введення
    // контрактами, натомість
    // користуйтеся винятками.
    date.fromString("2016-13-7");

    date.writeln;
}
```
