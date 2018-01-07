# Універсальний синтаксис виклику функції (UFCS)

**UFCS** є ключовою особливістю мови D та дозволяє повторне використання
коду та масштабування за допомогою чітко сформульованої інкапсуляції.

UFCS допускає, що будь-який виклик вільної функції `fun(a)` можна
записати у вигляді виклику функції `a.fun()`.

Якщо компілятор бачить `a.fun()`, а тип не має власну функцію під
назвою `fun()`, він намагається знайти глобальні функції, перший
параметр яких співпадає з `а`.

Ця можливість особливо корисна, коли треба поєднати виклики
складних функцій. Замість того, щоб писати

    foo(bar(a))

Можна написати

    a.bar().foo()

Крім того, у мові D не потрібно використовувати круглі дужки для функцій
без аргументів, що означає, що _будь-яка_ функція може бути використана,
як властивість:

    import std.uni : toLower;
    "D найкращий".toLower; // "d найкращий"

UFCS особливо важливий при роботі з *діапазонами (ranges)*, де кілька
алгоритмів можна поєднати разом, щоб виконувати складні операції, все
ще дозволяючи писати чіткий та підтримуваний код.

    import std.algorithm : group;
    import std.range : chain, retro, front, retro;
    [1, 2].chain([3, 4]).retro; // 4, 3, 2, 1
    [1, 1, 2, 2, 2].group.dropOne.front; // tuple(2, 3u)

### Поглиблення

- [UFCS у _Programming in D_](http://ddili.org/ders/d.en/ufcs.html)
- [_Uniform Function Call Syntax_](http://www.drdobbs.com/cpp/uniform-function-call-syntax/232700394) від Walter Bright
- [`std.range`](http://dlang.org/phobos/std_range.html)

## {SourceCode}

```d
import std.stdio : writefln, writeln;
import std.algorithm.iteration : filter;
import std.range : iota;

void main()
{
    "Привіт, %s".writefln("Світ");

    10.iota // повертає числа від 0 до 9
       // відфільтрувати лише парні
      .filter!(a => a % 2 == 0)
      .writeln(); // вивести у stdout

    // Традиційний стиль:
    writeln(filter!(a => a % 2 == 0)
                   (iota(10)));
}
```
