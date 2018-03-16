# Асоціативні масиви

D має вбудованi *асоціативні масиви*, також відомi як HashMap-и.
Асоціативний масив з ключем типу `string` і типом значення
`int` оголошується наступним чином:

    int[string] arr;

За допомогою ключа можна як отримати значення, так i встановити його:

    arr["key1"] = 10;

Щоб перевірити, чи розташований ключ в асоціативному масиві, може
бути використан вираз `in`:

    if ("key1" in arr)
        writeln("Yes");

Вираз `in` повертає вказівник на значення, якщо його можна знайти,
або `null` в іншому випадку. Таким чином, перевірка на існування
і запис можуть бути легко об'єднані:

    if (auto val = "key1" in arr)
        *val = 20;

Доступ до ключу, який не існує дає помилку `RangeError`, яка негайно
припиняє виконання програми. Для безпечного доступу зі значенням за
замовчуванням може бути використано вираз `get(key, ЗначенняЗаЗамовчуванням)`.

Асоциативнi масиви, як i звичайнi, мають властивість `.length`, а також
властивicть `.remove(val)`, що видаляє запис по ключу. В якості вправи
читачу залишається дослідження спеціальних діапазонів `.byKey`
і `.byValue`.

### Бiльше інформації

- [Асоциативнi масиви у _Programming in D_](http://ddili.org/ders/d.en/aa.html)
- [Специфiкацiя на асоціативнi масиви](https://dlang.org/spec/hash-map.html)
- [std.array.byPair](http://dlang.org/phobos/std_array.html#.byPair)

## {SourceCode}

```d
import std.array : assocArray;
import std.algorithm.iteration: each, group,
    splitter, sum;
import std.string: toLower;
import std.stdio : writefln, writeln;

void main()
{
    string text = "Rock D with D";

    // Пробігаємось по всім словам та
    // рахуємо кожне слово один раз
    int[string] words;
    text.toLower()
        .splitter(" ")
        .each!(w => words[w]++);

    foreach (key, value; words)
        writefln("Ключ: %s, значення: %d",
                       key, value);

    // `.keys` та .values` повертають масиви
    writeln("Слова:", words.keys);

    // `.byKey`, `.byValue` та `.byKeyValue`
    // повертають ліниві, ітерабельні
    // діапазони
    writeln("# Слова: ", words.byValue.sum);

    // Новий асоціативний масив може бути
    // створений з `assocArray` підставляючи
    // діапазон кортежів ключ/значення
    auto array = ['a', 'a', 'a', 'b', 'b',
                  'c', 'd', 'e', 'e'];

    // `.group` послідовно збирає
    // еквівалентні елементи у кортеж,
    // який складається з елементів та
    // числа їх повторів
    auto keyValue = array.group;
    writeln("Ключ/Значенян діапазону: ",
                  keyValue);
    writeln("Асоціативний масив: ",
             keyValue.assocArray);
}
```
