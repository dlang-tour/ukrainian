# Винятки

Ця інструкція призначена тільки для винятків користувача. Системні
винятки, як правило, зі смертельними наслідками, __ніколи__ не повинні
перехоплюватись.

### Перехоплення винятків

Основною задачею винятків є перехоплення введення потенційно неприпустимих
користувацьких даних. Коли трапляється виняток, відбувається проходження
по стеку, доки не буде знайдений перший вiдповiдний обробник винятків.

```d
try
{
    readText("dummyFile");
}
catch (FileException e)
{
    // ...
}
```

Можна мати кілька `catch` блоків і `finally` блок, який виконається
незалежно від того, чи відбулася помилка. Винятки викликаються за
допомогою ключового слова `throw`.

```d
try
{
    throw new StringException("Ти не пройдеш!");
}
catch (FileException e)
{
    // ...
}
catch (StringException e)
{
    // ...
}
finally
{
    // ...
}
```

Пам'ятайте, що [scope guard](gems/scope-guards) зазвичай є кращим рішенням
за паттерн `try-finally`.

### Користувальницькі виключення

Можна легко наслідуватись від `Exception` і створювати власні винятки:

```d
class UserNotFoundException : Exception
{
    this(string msg, string file = __FILE__, size_t line = __LINE__) {
        super(msg, file, line);
    }
}
throw new UserNotFoundException("D-Man знаходиться у відпустці");
```

### Організація безпечного світу за допомогою `nothrow`

Компілятор D може гарантувати, що функція не може спричинити катастрофічні
наслідки. Такі функції можуть бути помічені ключовим словом `nothrow`.
Компілятор D статично забороняє кидати винятки у `nothrow` функції.

```d
bool lessThan(int a, int b) nothrow
{
    writeln("небезпечний світ"); // writeln може генерувати виключення, таким чином, це заборонено
    return a < b;
}
```

Зверніть увагу, що компілятор може вивести атрибути для шаблонного коду
автоматично.

### std.exception

Важливо уникати контрактного програмування для введення користувацьких
даних, так як контракти видаляються при компіляції релізної версії програми.
Для зручності [`std.exception`](https://dlang.org/phobos/std_exception.html)
має ключове слово [`enforce`](https://dlang.org/phobos/std_exception.html#enforce),
яке можна використовувати як `assert`, але буде виникати об'єкт `Exceptions`
замість `AssertError`.

```d
import std.exception : enforce;
float magic = 1_000_000_000;
enforce(magic + 42 - magic == 42, "Математика з плаваючою точкою файна!");

// створити власний виняток
enforce!StringException('a' != 'A', "Чуттєвий до регістру алгоритм");
```

Однак, `std.exception` містить більше цікавих речей. Наприклад, якщо
помилка не матиме смертельні наслідки, можна обробити її з 
[`collect`](https://dlang.org/phobos/std_exception.html#collectException):

```d
import std.exception : collectException;
auto e = collectException(aDangerousOperation());
if (e)
    writeln("Небезпечна операція зазнала невдачі: ", e);
```

Щоб перевірити, чи відбулося виключення (для тестів наприклад),
використовуйте [`assertThrown`](https://dlang.org/phobos/std_exception.html#assertThrown).

### Додатково

- [Виняток безпеки у D](https://dlang.org/exception-safe.html)
- [std.exception](https://dlang.org/phobos/std_exception.html)
- системний рівень [core.exception](https://dlang.org/phobos/core_exception.html)
- [object.Exception](https://dlang.org/library/object/exception.html) - базовий клас для всіх винятків, які є безпечними та можуть перехоплюватись і оброблятись.

## {SourceCode}

```d
import std.file : FileException, readText;
import std.stdio : writeln;

void main()
{
    try
    {
        readText("dummyFile");
    }
    catch (FileException e)
    {
		writeln("Повідомлення:\n", e.msg);
		writeln("Файл: ", e.file);
		writeln("Рядок: ", e.line);
		writeln("Stack trace:\n", e.info);

		// Можна також використати writeln(e)
		// для виводу помилки з форматуванням
		// за замовчуванням
    }
}
```
