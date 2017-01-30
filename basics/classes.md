# Класи

D забезпечує підтримку класів та інтерфейсів, як Java чи C++.

Будь який `class` неявно успадковується від [`Object`](https://dlang.org/phobos/object.html).

    class Foo { } // успадковується від Object
    class Bar: Foo { } // Bar є також Foo

Класи у D, як правило, створюються у купі, за допомогою виразу `new`:

    auto bar = new Bar;

Об'єкти класу завжди є посиланням, і на відміну від `struct`, не
копіюються за значенням.

    Bar bar = foo; // bar посилається на foo

Збирач сміття звiльняє пам'ять об'єкту лише тодi, коли впевниться, що
ніякіх посилань на об'єкт більше не існує.

#### Успадкування

Щоб вказати на те, що функція базового класу була перевизначеною,
потрібно використати ключове слово `override`. Це дозволяє нам
запобігти ненавмисному перевизначенню функцій.

    class Bar: Foo {
        override functionFromFoo() {}
    }

У D класи можуть успадковуватись тільки від одного класу.

#### Фiнальнi та абстрактнi функції

- Щоб заборони перевизначення, функція може мати позначення `final` у
базовому класі
- Функцію можна оголосити, як `abstract`, щоб змусити перевизначити її
- Цілий клас може бути оголошений, як `abstract`, щоб переконатися у тому,
що жодного екземпляру класу не буде створено
- Щоб отримати доступ до базового класу, потрібно використовувати
спеціальне ключове слово `super`

### Додатково

- [Класи _Programming in D_](http://ddili.org/ders/d.en/class.html)
- [Успадкування _Programming in D_](http://ddili.org/ders/d.en/inheritance.html)
- [Клас Object у _Programming in D_](http://ddili.org/ders/d.en/object.html)
- [Специфiкацiя на класи у D](https://dlang.org/spec/class.html)

## {SourceCode}

```d
import std.stdio;

/*
  Незвичайні типи, які можуть бути
  використані для чого-небудь...
*/
class Any {
    // доступ до protected - тiльки з
    // успадкованих класiв
    protected string type;

    this(string type) {
        this.type = type;
    }

    // неявно оголошується public
    final string getType() {
        return type;
    }

    // Це потрiбно перевизначити!
    abstract string convertToString();
}

class Integer: Any {
    // доступ тiльки з класу Integer
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
        // Ми бажаємо мати контроль над точнiстю.
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
        writeln("Будь-який тип = ", any.getType());
        writeln("Контент = ",
            any.convertToString());
    }
}
```
