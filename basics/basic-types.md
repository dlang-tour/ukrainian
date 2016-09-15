# Базовi типи данних

D надає ряд основних типів, які завжди мають той же
розмір **незалежно**  від платформи - єдиний виняток
є тип `real`, який забезпечує максимально можливу
точність числа з плаваючою точкою. Немає ніякої різниці
між розміром цілого незалежно від того, програма
скомпільована для 32- або 64-бітової системи.

<table class="table table-hover">
<tr><td width="250px"><code class="prettyprint">bool</code></td> <td>8-bit</td></tr>
<tr><td><code class="prettyprint">byte, ubyte, char</code></td> <td>8-bit</td></tr>
<tr><td><code class="prettyprint">short, ushort, wchar</code></td> <td>16-bit</td></tr>
<tr><td><code class="prettyprint">int, uint, dchar</code></td> <td>32-bit</td></tr>
<tr><td><code class="prettyprint">long, ulong</code></td> <td>64-bit</td></tr>
</table>

#### Типи з плаваючою точкою:

<table class="table table-hover">
<tr><td width="250px"><code class="prettyprint">float</code></td> <td>32-bit</td></tr>
<tr><td><code class="prettyprint">double</code></td> <td>64-bit</td></tr>
<tr><td><code class="prettyprint">real</code></td> <td>залежно вiд платформи, 80-бiт на Intel x86-32</td></tr>
</table>

Префікс `u` позначає *беззнаковi* типи. `char` є 
UTF-8 символом, `wchar` використовується в UTF-16 рядках і `dchar`
в UTF-32 рядках.

Перетворення між змінними різних типів 
допускається компілятором лише якщо немає втрати точностi. Хоча конверсія
між плаваючих типів (наприклад `double` до `float`)
допускається.

Перетворення в інший тип може бути примусовим при використаннi вираза
`cast(TYPE) myVar`. Перетворення необхідно використовувати з великою обережністю, оскiльки
виразу `cast` дозволено порушувати систему типів.

Спеціальне ключове слово `auto` створює змінну і виводить iї
тип з типу правого боку виразу. `auto MyVar = 7`
виведе тип `int` для `myVar`. Зверніть увагу, що тип буде
встановлен під час компіляції і не може бути змінений пiзнiше - так само, як з будь-якою іншою
змінною з явно заданим типом.

### Властивостi типiв

Всі типи даних мають властивість `.init` - значення яким вони iнiциализуються.
Для всіх цілих чисел це "0", для типiв з плаваючою точкою це `nan` (* Не число *).

Інтегральні і типи з плаваючою точкою мають `.max` властивість для найвищого значення яке
вони можуть представляти. Цілі типи також мають властивість `.min` для найменшого значення
яке вони можуть представляти, в той час як типи з плаваючою точкою мають властивість `.min_normal`
яка визначається найменшим представимим нормованним значенням, таким що не дорiвнює 0.

Плаваючі типи точок також мають властивості `.nan` (NaN-значення), `.infinity`
(значення нескінченності), `.dig` (точнiсть у кількостi десяткових цифр),` .mant_dig`
(кількість бітів в мантиси) і багато інших.

Кожен тип також має властивість `.stringof`, яка дає його назву у вигляді рядка.

### Iндекси в D

У D індекси мають зазвичай тип псевдоніма `size_t`, так як це є досить великий тип, щоб представляти зміщення в усій адресуємой пам'яті - тобто `uint` для 32-розрядних і `ulong` для 64-розрядних архітектур.

### Asserts

`assert` є вбудований у компілятор вираз, який перевіряє умови в режимі налагодження і завершує роботу
з `AssertionError`, якщо умова не виконується.

### Детальнiше

#### Основнi посилання

- [Присвоювання](http://ddili.org/ders/d.en/assignment.html)
- [Змiннi](http://ddili.org/ders/d.en/variables.html)
- [Арiфметика](http://ddili.org/ders/d.en/arithmetic.html)
- [Плаваюча точка](http://ddili.org/ders/d.en/floating_point.html)
- [Фундаментальнi типи в _Programming in D_](http://ddili.org/ders/d.en/types.html)

#### Детальнiши посилання

- [Огляд усiх базових типiв у D](https://dlang.org/spec/type.html)
- [`auto` та `typeof` в _Programming in D_](http://ddili.org/ders/d.en/auto_and_typeof.html)
- [Властивостi типiв](https://dlang.org/spec/property.html)

## {SourceCode}

```d
import std.stdio;

void main()
{
    // Велики числа можуть бути
    // вiдокремленi пiдкреслюванням  "_"
    // для полiпшення читанiстi.
    int b = 7_000_000;
    short c = cast(short) b; // потрiбне перетворення
    uint d = b; // fine
    int g;
    assert(g == 0);

    auto f = 3.1415f; // f позначає float

    // typeid(VAR) повертає iнформацию про тип
    // виразу.
    writeln("type of f is ", typeid(f));
    double pi = f; // fine
    // для плаваючих типiв
    // неявне понижуючє приведення допускається
    float demoted = pi;

    // доступ до властивостей типiв
    assert(int.init == 0);
    assert(int.sizeof == 4);
    assert(bool.max == 1);
    writeln(int.min, " ", int.max);
    writeln(int.stringof); // цiле
}
```
