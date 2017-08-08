# Шаблонне метапрограмування

Якщо Ви коли-небудь зв’яжетеся з *шаблонним метапрограмуванням* у мові
C++, ви дізнаєтесь, які інструменти пропонує мова D, щоб зробити
ваше життя простішим. Шаблонне метапрограмування це техніка, яка
уможливлює процес прийняття рішень у залежності від типу властивостей
шаблону і, таким чином, дозволяє зробити загальні типи ще більш
гнучкішими, основуючись на типі за допомогою якого вони створені.

### `static if` & `is`

Як і звичайний `if`, `static if` умовно компілює кодовий блок,
основуючись на умові, що він може бути оцінений під час компіляції.

    static if(is(T == int))
        writeln("T є int");
    static if (is(typeof(x) :  int))
        writeln("Змінна x неявно конвертується у int");

[Вираз `is`](http://wiki.dlang.org/Is_expression) є загальним
помічником, який оцінює вирази під час компіляції.

    static if(is(T == int)) { // T є параметром шаблону
        int x = 10;
    }

Дужки опускаються, якщо умова є `true` - нова область не створюється.
`{ {` та `} }` явно створює новий блок.

`static if` може бути використаний будь-де у коді: у функціях, у
глобальній області видимості чи у межах визначень типів.

### `mixin template`

Скрізь де ми зустрічаємо *шаблонний код*, `mixin template` є вашим
товаришем:

    mixin template Foo(T) {
        T foo;
    }
    ...
    mixin Foo!int; // int foo доступні тут.

`mixin template` може містити будь-яку кількість складних виразів, які
вставляються на момент конкретизації. Попрощайтеся з препроцесором,
якщо прийшли до нас з мови C!

### Обмеження шаблонів

Шаблон може бути визначений з будь-яким числом обмежень, які забезпечують
перевірку властивостей, які повинен мати тип:

    void foo(T)(T value)
      if (is(T : int)) { // foo!T дійсний лише, якщо T
                         // конвертується у int
    }

Обмеження можуть бути об'єднані у логічний вираз і, навіть, можуть
містити виклики функцій, які будуть викликані під час компіляції.
Наприклад,  `std.range.primitives.isRandomAccessRange` перевіряє чи тип
є діапазоном, який підтримує оператор `[]`.

### Поглиблення

### Основні посилання

- [Підручник по шаблонам D](https://github.com/PhilippeSigaud/D-templates-tutorial)
- [Умовна компіляція](http://ddili.org/ders/d.en/cond_comp.html)
- [std.traits](https://dlang.org/phobos/std_traits.html)
- [Більше про шаблони у  _Programming in D_](http://ddili.org/ders/d.en/templates_more.html)
- [Mixins у  _Programming in D_](http://ddili.org/ders/d.en/mixin.html)

### Додаткові посилання

- [Умовна компіляція](https://dlang.org/spec/version.html)
- [Traits](https://dlang.org/spec/traits.html)
- [Mixin templates](https://dlang.org/spec/template-mixin.html)
- [Специфікація по шаблонам D](https://dlang.org/spec/template.html)

## {SourceCode}

```d
import std.traits : isFloatingPoint;
import std.uni : toUpper;
import std.string : format;
import std.stdio : writeln;

/*
Вектор, який просто працює для
чисел, цілих або з плаваючою точкою.
*/
struct Vector3(T)
  if (is(T: real))
{
private:
    T x,y,z;

    /*
    Генератор для геттерів і сетерів,
    тому що ми дійсно ненавидимо
    шаблонний код!
    
    var -> T getVAR() і void setVAR(T)
    */
    mixin template GetterSetter(string var) {
	    // Використовуємо mixin для
	    // побудови імен функцій
        mixin("T get%s() const { return %s; }"
          .format(var.toUpper, var));

        mixin("void set%s(T v) { %s = v; }"
          .format(var.toUpper, var));
    }

    /*
    Просте генерування getX, setX і т.д.
    функції за допомогою mixin template.
    */
    mixin GetterSetter!"x";
    mixin GetterSetter!"y";
    mixin GetterSetter!"z";

public:
    /*
    Функція dot доступна тільки для
    типів з плаваючою точкою
    */
    static if (isFloatingPoint!T) {
        T dot(Vector3!T rhs) {
            return x * rhs.x + y * rhs.y +
                z * rhs.z;
        }
    }
}

void main()
{
    auto vec = Vector3!double(3,3,3);
    // Це не працює через обмеження
    // шаблону!
    // Vector3!string illegal;

    auto vec2 = Vector3!double(4,4,4);
    writeln("vec dot vec2 = ", vec.dot(vec2));

    auto vecInt = Vector3!int(1,2,3);
    // Не містить функцію dot, тому що
    // ми статично дозволяємо її
    // тільки для float-ів
    // vecInt.dot(Vector3!int(0,0,0));

	// використовуємо згенеровані
	// геттери і сеттери!
    vecInt.setX(3);
    vecInt.setZ(1);
    writeln(vecInt.getX, ",",
      vecInt.getY, ",", vecInt.getZ);
}
```
