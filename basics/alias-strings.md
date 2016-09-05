# Alias & String

Тепер, коли ми знаємо, що таке масиви, дізнались про `immutable` та
пробіглись по основним типам, час познайомитись відразу з двома новими
конструкціями:

    alias string = immutable(char)[];

Термін `string` (далі `рядки`) оголошено, як інструкція `alias`
(псевдонім), яка визначає його як зріз `immutable(char)`'ів. Це
означає, що як тільки `рядок` побудований, його вміст більше не може
бути змінений. І це одна з тем, з якою познайомить цей розділ:
`рядки` у кодуванні UTF-8!

Через свою незмінність `рядки` можуть бути чудово розподілені між різними
потоками. Так як `рядки` являються зрізами, їх частини можуть бути зв’язані
без додаткового виділення пам’яті. Наприклад, коли стандартна функція
`std.algorithm.splitter` розділяє рядок на частинки, додаткова пам’ять
не виділяється.

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
робити і з рядками, але працювати це буде на рівні значень Юнікод, а не на рівні
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
(діакритичний знак, який комбінується з літерою), і [`walkLength`]
(https://dlang.org/library/std/range/primitives/walk_length.html) (функція
стандартної бібліотеки для підрахунку довжини довільного діапазону) повертає
2, так як підраховується лише два елементи Юнікоду. І нарешті, `byGrapheme` -
це досить дороге обчислення, яке показує, що ці два елементи є один видимим
символом.

Правильна обробка Юнікоду може бути доволі непростою, але зазвичай D-розробники
можуть просто вважати змінні `string`, як магічний набір байтів і покладатися
на алгоритми стандартної бібліотеки. Велика частина функціональності Юнікоду
представлена у модулі [`std.uni`](https://dlang.org/library/std/uni.html),
а деякі базові примітиви доступні у [`std.utf`](https://dlang.org/library/std/utf.html).

### Багаторядковий текст

Щоб створити багаторядковий текст використовуйте `string str = q{ ... }` синтаксис.

    string multiline = q{ Це
        може бути
        довгий документ
    };

### Необроблені (Raw) рядки

Також можливо використовувати необроблені рядки для мінімізації ручного введення
зарезервованих символів екранування. Необроблені рядки можуть бути оголошені
з використанням зворотних апострофів (`` ` ... ` ``) чи префікса r(aw) (`r" ... "`).

    string raw  =  `raw "string"`; // raw "string"
    string raw2 = r"raw "string""; // raw "string"

### Додатково

- [Символи у _Programming in D_](http://ddili.org/ders/d.en/characters.html)
- [Рядки у _Programming in D_](http://ddili.org/ders/d.en/strings.html)
- [std.utf](http://dlang.org/phobos/std_utf.html) - Алгоритми (де)кодування UTF
- [std.uni](http://dlang.org/phobos/std_uni.html) - Алгоритми Юнікод

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

