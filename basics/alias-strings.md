# Alias & String

Тепер, коли ми знаємо, що таке масиви, дізнались про `immutable` та
пробіглись по основним типам, час познайомитись відразу з двома новими
конструкціями:

    alias string = immutable(char)[];

Термін `string` (далі `рядки`) оголошено, як інструкція `alias`
(псевдонім), яка визначає його як зріз `immutable(char)`'ів. Це
означає, що як тільки `рядок` побудований, його вміст більше не може
бути змінений. Це i є наша наступна тема: зустрiчайте `рядки`
у кодуваннi UTF-8!

Через свою незмінність `рядки` можуть бути чудово розподілені між різними
потоками. Так як `рядки` являються зрізами, їх частини можуть бути отримані
без додаткового виділення пам’яті. Наприклад, коли стандартна функція
[`std.algorithm.splitter`](https://dlang.org/phobos/std_algorithm_iteration.html#.splitter)
розділяє рядок на частинки, додаткова пам’ять не виділяється.

Окрім UTF-8 `рядків`, є ще два типи:

    alias wstring = immutable(wchar)[]; // UTF-16
    alias dstring = immutable(dchar)[]; // UTF-32

Їх найлегше конвертувати між собою за допомогою метода `to` з
пакету `std.conv`:

    dstring myDstring = to!dstring(myString);
    string myString   = to!string(myDstring);

### Юнікод рядки

Це означає, що рядки формату `string` визначені, як масив 8-бітних Юнікод [значень]
(http://unicode.org/glossary/#code_unit). Все, що можна робити з масивами, можна
робити і з рядками, але працювати це буде на рівні значень Юнікоду, а не на рівні
символів. У той же час алгоритми зі стандартної бібліотеки інтерпретують рядки,
як послідовність [елементів Юнікод](http://unicode.org/glossary/#code_point),
і крім цього є спосіб інтерпретувати їх, як послідовність [графемів]
(http://unicode.org/glossary/#grapheme), явно використавши [`std.uni.byGrapheme`]
(https://dlang.org/library/std/uni/by_grapheme.html).

Цей невеликий приклад показує різницю в інтерпретаціях:

    string s = "\u0041\u0308"; // Ä

    writeln(s.length); // 3

    import std.range : walkLength;
    writeln(s.walkLength); // 2

    import std.uni : byGrapheme;
    writeln(s.byGrapheme.walkLength); // 1

У прикладі вище довжина масиву `s` рівна трьом, тому що містить 3 значення
Юнікоду: `0x41`, `0x03` та `0x08`. З них останні два означають один елемент
([діакритичний знак](https://uk.wikipedia.org/wiki/%D0%94%D1%96%D0%B0%D0%BA%D1%80%D0%B8%D1%82%D0%B8%D1%87%D0%BD%D0%B8%D0%B9_%D0%B7%D0%BD%D0%B0%D0%BA), який комбінується з літерою), а [`walkLength`]
(https://dlang.org/library/std/range/primitives/walk_length.html) (функція
стандартної бібліотеки для підрахунку довжини довільного діапазону) повертає
2, так як підраховується лише два елементи Юнікоду. І нарешті, `byGrapheme` -
це досить дороге обчислення, яке показує, що ці два елементи є одним видимим
символом.

Правильна обробка Юнікоду може бути доволі непростою, але зазвичай D-розробники
можуть просто вважати змінні `string`, як магічний набір байтів і покладатися
на алгоритми стандартної бібліотеки. Якщо потрібна ітерація для елементу
(блоку коду), можна використовувати [`byCodeUnit`](http://dlang.org/phobos/std_utf.html#.byCodeUnit).

Авто-декодування у D пояснюється детальніше у розділі [Unicode у мові D](gems/unicode).

### Багаторядковий текст

Рядки у D завжди можуть бути розбиті на кілька рядків:

    string multiline = "
    Це
    може бути
    довгий документ
    ";

Коли у документі з'являються цитати, можна використовувати необроблені
рядки (див. нижче) або [heredoc strings](http://dlang.org/spec/lex.html#delimited_strings).

### Необроблені (Raw) рядки

Також можливо використовувати необроблені рядки для мінімізації ручного введення
зарезервованих символів екранування. Необроблені рядки можуть бути оголошені
з використанням зворотних апострофів (`` ` ... ` ``) чи префікса r(aw) (`r" ... "`).

    string raw  =  `raw "string"`; // raw "string"
    string raw2 = r"raw `string`"; // raw `string`

D забезпечує ще більше способів представлення рядків - без вагань
[дослідіть](https://dlang.org/spec/lex.html#string_literals) їх.

### Додатково

- [Юнікод gem](gems/unicode)
- [Символи у _Programming in D_](http://ddili.org/ders/d.en/characters.html)
- [Рядки у _Programming in D_](http://ddili.org/ders/d.en/strings.html)
- [std.utf](http://dlang.org/phobos/std_utf.html) - Алгоритми (де)кодування UTF
- [std.uni](http://dlang.org/phobos/std_uni.html) - Алгоритми Юнікод
- [String Literals у D специфікації](http://dlang.org/spec/lex.html#string_literals)

## {SourceCode}

```d
import std.stdio;
import std.range: walkLength;
import std.uni : byGrapheme;
import std.string: format;

void main() {
    // format генерує рядок, використовуючи
    // printf-подібний синтаксис. D дозволяє
    // нативну обробку UTF рядків
    string str = format("%s %s", "Hellö",
        "Wörld");
    writeln("My string: ", str);
    writeln("Array length (code unit count)"
        ~ " of string: ", str.length);
    writeln("Range length (code point count)"
        ~ " of string: ", str.walkLength);
    writeln("Character length (grapheme count)"
        ~ " of string: ",
        str.byGrapheme.walkLength);

    // Рядки - це звичайні масиви, тому
    // будь-які операції, що застосовуються до
    // масиву, тут також працюють.
    import std.array: replace;
    writeln(replace(str, "lö", "lo"));
    import std.algorithm: endsWith;
    writefln("Does %s end with 'rld'? %s",
        str, endsWith(str, "rld"));

    import std.conv: to;
    // Перетворення у UTF-32
    dstring dstr = to!dstring(str);
    // .. яка звісно ж, виглядає так само!
    writeln("My dstring: ", dstr);
}
```

