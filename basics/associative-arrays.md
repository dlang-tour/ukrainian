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
import std.stdio;

/**
  Розділяє даний текст на слова і повертає
  асоціативний масив, який відображує слова
  з їх кількістю (скільки раз зустрічались).

  Параметри:
    text = текст для нарізання
*/
int[string] wordCount(string text)
{
    // Функцiя splitter лiниво розбиває
    // вхiдий рядок на дiaпазон
    import std.algorithm.iteration: splitter;
    import std.string: toLower;

    // Проiндексовано словами,
    // повертає кількість
    int[string] words;

    foreach(word; splitter(text.toLower(), " "))
    {
        // Збільшує лiчильник слова, якщо
        // воно було знайдене.
        // За замовчуванням значення
        // дорiвнює 0.
        words[word]++;
    }

    return words;
}

void main()
{
    string text = "D is a lot of fun";

    auto wc = wordCount(text);
    writeln("Word counts: ", wc);

    // можливi ітерації:
    // byKey, byValue, byKeyValue
    foreach (word; wc.byValue)
        writeln(word);
}
```
