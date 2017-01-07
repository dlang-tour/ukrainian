# Функції

З однією функцією ми вже знайомі: `main()` - точка входу кожної програми
на D. Фукнції можуть повертати якесь значення (або бути оголошеними з
ключовим словом `void`, якщо не треба нічого повертати) та приймати
будь-яку кількість аргументів на вхід:

    int add(int lhs, int rhs) {
        return lhs + rhs;
    }

### Повернення типу `auto`

Якщо тип, який функція буде повертати заданий як `auto`, компілятор
виведе тип автоматично. Якщо функція має декілька інструкцій `return`,
вони повинні повертати значення сумісних типів.

    auto add(int lhs, int rhs) { // повертається тип `int`
        return lhs + rhs;
    }

    auto lessOrEqual(int lhs, int rhs) { // повертається тип `double`
        if (lhs <= rhs)
            return 0;
        else
            return 1.0;
    }

### Аргументи за замовчуванням

Функції можуть опціонально задати аргументам значення, які будуть
використовуватись за замовчуванням. Це дозволяє уникнути набридливого
оголошення надлишкових перевантажень.

    void plot(string msg, string color = "red") {
        ...
    }
    plot("D rocks");
    plot("D rocks", "blue");

Після того, як було задано значення для аргументу за замовчуванням, всі
наступні аргументи повинні мати також значення за замовчуванням.

### Локальні функції

Функції також можна оголошувати всередині інших функцій. Такі функції
можуть використовуватись локально і не доступні з зовнішнього світу.
При цьому такі функції можуть мати доступ до батьківської області
видимості:

    void fun() {
        int local = 10;
        int fun_secret() {
            local++; // це допустимо
        }
        ...

Такі вкладені функції називаються делегатами і ми поговоримо про них
більш детально [трішки пізніше](basics/delegates).

### Додатково

- [Функції у _Programming in D_](http://ddili.org/ders/d.en/functions.html)
- [Параметри функцій у _Programming in D_](http://ddili.org/ders/d.en/function_parameters.html)
- [Специфікація на функції](https://dlang.org/spec/function.html)

## {SourceCode}

```d
import std.stdio;
import std.random;

void randomCalculator()
{
    // Визначено 4 локальні функції для
    // 4 різних математичних операцій
    auto add(int lhs, int rhs) {
        return lhs + rhs;
    }
    auto sub(int lhs, int rhs) {
        return lhs - rhs;
    }
    auto mul(int lhs, int rhs) {
        return lhs * rhs;
    }
    auto div(int lhs, int rhs) {
        return lhs / rhs;
    }

    int a = 10;
    int b = 5;

    // uniform генерує число між START
    // та END, в яке END НЕ входить.
    // Залежно від результату ми викликаємо
    // одну з математичних операцій.
    switch (uniform(0, 4)) {
        case 0:
            writeln(add(a, b));
            break;
        case 1:
            writeln(sub(a, b));
            break;
        case 2:
            writeln(mul(a, b));
            break;
        case 3:
            writeln(div(a, b));
            break;
        default:
            // спеціальний код який помічає
            // НЕДОСЯЖНИЙ варіант
            assert(0);
    }
}

void main()
{
    randomCalculator();
    // add(), sub(), mul() та div()
    // НЕ видимі поза їх областю видимості
    static assert(!__traits(compiles,
                            add(1, 2)));
}

```
