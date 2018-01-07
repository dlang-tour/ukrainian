# Traits (Характеристики)

Одною з переваг D є її система виконання функцій під час компіляції (CTFE).
У поєднанні з самоаналізом можна написати узагальнені програми та
досягнути потужних оптимізацій.

## Явні контракти

Traits дозволяють явно вказати, що може бути подано на вхід.
Наприклад, `splitIntoWords` може працювати з будь-яким рядковим типом:

```d
S[] splitIntoWord(S)(S input)
if (isSomeString!S)
```

Це відноситься також до параметрів шаблону. `myWrapper` може гарантувати,
що переданий символ можна викликати як функцію:

```d
void myWrapper(alias f)
if (isCallable!f)
```

Проаналізуємо простий приклад, [`commonPrefix`](https://dlang.org/phobos/std_algorithm_searching.html#.commonPrefix)
з `std.algorithm.searching`, який повертає загальний префікс двох діапазонів:

```d
auto commonPrefix(alias pred = "a == b", R1, R2)(R1 r1, R2 r2)
if (isForwardRange!R1 &&
    isInputRange!R2 &&
    is(typeof(binaryFun!pred(r1.front, r2.front)))) &&
    !isNarrowString!R1)
```

Виклик такої функції скомпілюється тільки якщо:

- `r1` можна зберегти (гарантовано `isForwardRange`)
- `r2` підтримує ітерацію (гарантовано `isInputRange`)
- `pred` викликається з типами елементів `r1` та `r2`
- `r1` не "вузький" рядок (`char[]`, `string`, `wchar` чи `wstring`) - для простоти, в іншому випадку, може бути необхідне декодування

### Спеціалізація

Багато API націлені на розширення області застосування, проте вони
не хочуть платити швидкодією за таке узагальнення.
Із силою самоаналізу та CTFE, можна сфокусуватися на методах
часу компіляції для досягнення максимальної продуктивності, враховуючи
певні вхідні типи.

Найпоширеніша проблема, що на відміну від масивів, ми повинні знати
точну довжину потоку даних або списку перш, ніж працювати з ним.
Тому ми маємо простий метод `walkLength` з `std.range`,
який є узагальненим для будь-якого ітерабельного типу:

```d
static if (hasMember!(r, "length"))
    return r.length; // O(1)
else
    return r.walkLength; // O(n)
```

#### `commonPrefix`

У бібліотеці `Phobos` самоаналіз під час компіляції використовується
всюди. Наприклад, `сommonPrefix` розрізняє діапазони з випадковим
доступом (`RandomAccessRange`) та лінійні ітерабельні діапазони, тому
що у `RandomAccessRange` можна "перестрибувати" між позиціями і, таким
чином, прискорювати виконання алгоритму.

#### Більше магії CTFE

[std.traits](https://dlang.org/phobos/std_traits.html) - оболонка навколо
більшості [характеристик (traits) D](https://dlang.org/spec/traits.html),
за винятком деяких, наприклад `compiles`, який неможливо "обернути",
так як це призведе до негайної помилки компіляції:

```d
__traits(compiles, obvious error - $%42); // false
```

#### Спеціальні ключові слова

Крім того, D надає деякі спеціальні ключові слова для відлагодження:

```d
void test(string file = __FILE__, size_t line = __LINE__, string mod = __MODULE__,
          string func = __FUNCTION__, string pretty = __PRETTY_FUNCTION__)
{
    writefln("файл: '%s', рядок: '%s', модуль: '%s',\nфункція: '%s', гарна функція: '%s'",
             file, line, mod, func, pretty);
}
```

А завдяки обчислень з командного рядка, можна забути про `time` - буде
використано CTFE!

```d
rdmd --force --eval='pragma(msg, __TIMESTAMP__);'
```

## Поглиблення

- [std.range.primitives](https://dlang.org/phobos/std_range_primitives.html)
- [std.traits](https://dlang.org/phobos/std_traits.html)
- [std.meta](https://dlang.org/phobos/std_meta.html)
- [Специфікація по Traits у D](https://dlang.org/spec/traits.html)

## {SourceCode}

```d
import std.functional : binaryFun;
import std.range.primitives : empty, front,
    popFront,
    isInputRange,
    isForwardRange,
    isRandomAccessRange,
    hasSlicing,
    hasLength;
import std.stdio : writeln;
import std.traits : isNarrowString;

/**
Повертає загальний префікс двох діапазонів,
особливий випадок з авто-декодуванням
не розглядаємо.

Параметри:
    pred = Предикат для визначення "спільності"
    r1 = Діапазон forward range.
    r2 = Діапазон input range.

Повертає:
Зріз r1, який містить символи,
однакові на початку двох діапазонів.
 */
auto commonPrefix(alias pred = "a == b", R1, R2)
                 (R1 r1, R2 r2)
if (isForwardRange!R1 && isInputRange!R2 &&
    !isNarrowString!R1 &&
    is(typeof(binaryFun!pred(r1.front,
                             r2.front))))
{
    import std.algorithm.comparison : min;
    static if (isRandomAccessRange!R1 &&
               isRandomAccessRange!R2 &&
               hasLength!R1 && hasLength!R2 &&
               hasSlicing!R1)
    {
        immutable limit = min(r1.length,
                              r2.length);
        foreach (i; 0 .. limit)
        {
            if (!binaryFun!pred(r1[i], r2[i]))
            {
                return r1[0 .. i];
            }
        }
        return r1[0 .. limit];
    }
    else
    {
        import std.range : takeExactly;
        auto result = r1.save;
        size_t i = 0;
        for (;
             !r1.empty && !r2.empty &&
             binaryFun!pred(r1.front, r2.front);
             ++i, r1.popFront(), r2.popFront())
        {}
        return takeExactly(result, i);
    }
}

void main()
{
    // виводить: "hello, "
    writeln(commonPrefix("hello, world"d,
                         "hello, there"d));
}
```
