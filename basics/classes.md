# Класи

D забезпечує підтримку класів і інтерфейсів, як Java або C ++.

Будь який `class` успадковує неявно від [` Object`] (https://dlang.org/phobos/object.html).

    class Foo { } // успадковує від Object
    class Bar: Foo { } // Bar є також Foo

Класи в D, як правило, створюються у купі, за допомогою виразу `new`:

    auto bar = new Bar;

Об'єкти класу завжди є посиланням і на відміну від `struct` не
копіруються як значення.

    Bar bar = foo; // bar посилається на foo

Збирач сміття звiльняє пам'ять об'екту тiльки тодi, коли буде переконан, що ніякіх посилань на об'єкт більше не існує.

#### Успадкування

Щоб вказати на те, що функція-член базового класу перевизначається, повинно використати ключове слово
`Override`. Це запобігає ненавмисному перевизначенню функцій.

    class Bar: Foo {
        override functionFromFoo() {}
    }

У D класи можуть успадковувати тільки від одного класу.

#### Фiнальнi та абстрактнi функції-члени

Для заборони перевизначення функція може мати позначення `final` в базовому класі. 
Функція може бути оголошена як `abstract`, щоб змусити перевизначити ії.
Цілий клас може бути оголошений як `abstract`, щоб бути переконним,
що жодного екземпляру класу не буде створено. Щоб отримати доступ до базового класу, треба
використовувати спеціальне ключове слово `super`.

### Додатково

- [Класи _Programming in D_](http://ddili.org/ders/d.en/class.html)
- [Успадування _Programming in D_](http://ddili.org/ders/d.en/inheritance.html)
- [Клас Object в _Programming in D_](http://ddili.org/ders/d.en/object.html)
- [Специфiкацiя на класы в D](https://dlang.org/spec/class.html)

## {SourceCode}

```d
import std.stdio;

/*
Незвичайні типи, які можуть бути використані для
чого-небудь...
*/
class Any {
    // доступ до protected - тiльки з наслiдуючих класiв
    protected string type;

    this(string type) {
        this.type = type;
    }

    // неявно оголошуется public
    final string getType() {
        return type;
    }

    // Це потрiбно перевизначити!
    abstract string convertToString();
}

class Integer: Any {
    // доступ тiльки з Integer
    private {
        int number;
    }

    // конструктор
    this(int number) {
        // виклик конструктору базового класу
        super("integer");
        this.number = number;
    }

    // Неявно. Iнший шлях вказати рiвень захисту:
    public:

    override string convertToString() {
        import std.conv: to;
        // Швейцарський ніж для перетворення.
        return to!string(number);
    }
}

class Float: Any {
    private float number;

    this(float number) {
        super("float");
        this.number = number;
    }

    override string convertToString() {
        import std.string: format;
        // Ми бажаемо контролю над точнiстю.
        return format("%.1f", number);
    }
}

void main()
{
    Any[] anys = [
        new Integer(10),
        new Float(3.1415f)
        ];

    foreach (any; anys) {
        writeln("any's type = ", any.getType());
        writeln("Content = ",
            any.convertToString());
    }
}
```
