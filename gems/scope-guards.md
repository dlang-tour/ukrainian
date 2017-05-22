# Scope guards

Scope guards дозволяють виконувати певні інструкції за умови,
що поточний блок завершився:

* `scope(exit)` завжди будуть викликатись інструкції
* інструкції `scope(success)` викликаються, якщо не було жодних винятків
* `scope(failure)` позначає інструкції, які викликаються за умови
винятку (помилки), зробленого до завершення блоку

Використання scope guards робить код набагато чистішим і дозволяє
розміщувати код для виділення та очищення пам'яті поруч. Ці маленькі
помічники також покращують безпеку, тому що дають гарантію, що певний
код для очищення буде *завжди* викликатися незалежно від того, що сталося
під час виконання.

`Зміст (scope)` мови D ефективно замінює ідіому RAII, що
використовується у мові C++, яка часто веде до спеціальних об’єктів
scope guards для конкретних ресурсів.

Scope guards викликаються у зворотному від зазначеного порядку.

### Поглиблення

- [`scope` у _Programming in D_](http://ddili.org/ders/d.en/scope.html)

## {SourceCode}

```d
import std.stdio : writefln, writeln;

void main()
{
    writeln("<html>");
    scope(exit) writeln("</html>");

    {
        writeln("\t<head>");
        scope(exit) writeln("\t</head>");
        "\t<title>%s</title>".writefln("Привіт");
    } // scope(exit) зазначений раніше
      // викликається тут

    writeln("\t<body>");
    scope(exit) writeln("\t</body>");

    writeln("\t\t<h1>Привіт світ!</h1>");

    // scope guards дозволяє розмістити
    // виділення та очищення пам'яті
    // поруч
    import core.stdc.stdlib : free, malloc;
    int* p = cast(int*) malloc(int.sizeof);
    scope(exit) free(p);
}
```
