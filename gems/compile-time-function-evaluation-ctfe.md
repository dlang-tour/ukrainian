# Оцінка Функції Часу Компіляції (ОФЧК чи CTFE)

ОФЧК – це механізм, який дозволяє компілятору виконати функції
**під час компіляції (compile time)**.
Мова D не містить набору спеціальних механізмів, необхідних для
використання цих можливостей. Тому кожен раз, коли функція залежатиме
тільки від часу компіляції (компілятор зможе розрахувати вираз відразу
на основі відомих значень), компілятор D може інтерпретувати її
безпосередньо під час компіляції.

    // результат буде вирахований під час
    // компіляції. Перевірте машинний код, він
    // не міститиме виклику функції!
    static val = sqrt(50);

Ключові слова `static`, `immutable` або `enum` інструктують компілятор,
як використовувати ОФЧК, коли існує така можливість. Найкраще у цій
технології те, що непотрібно переписувати функції, щоб використовувати
ОФЧК, адже той самий код може бути чудово розподілений:

    int n = doSomeRuntimeStuff();
    // така ж функція як вище-зазначена, але
    // на цей раз вона викликається класичним
    // часом виконання (run-time).
    auto val = sqrt(n);

Один відомий приклад у мові D це бібліотека [std.regex](https://dlang.org/phobos/std_regex.html).
Вона забезпечує тип `ctRegex`, який використовує *string mixins* і ОФЧК
для створення високо-оптимізованих регулярних виразів, які генеруються
під час компіляції. Той самий базовий код повторно використовується
під час виконання run-time версії `regex`, що дозволяє компілювати
регулярні вирази, доступні тільки під час run-time. 

    auto ctr = ctRegex!(`^.*/([^/]+)/?$`);
    auto tr = regex(`^.*/([^/]+)/?$`);
    // ctr і tr можуть використовуватися поступово
    // але команда ctr є більш швидкою!

Не всі властивості мови є доступними під час ОФЧК, але набір
можливостей збільшується з кожним новим релізом компілятора D.

### Поглиблення

- [Введення у регулярні вирази мови D](https://dlang.org/regular-expression.html)
- [std.regex](https://dlang.org/phobos/std_regex.html)
- [Умовна компіляція](https://dlang.org/spec/version.html)

## {SourceCode}

```d
import std.stdio : writeln;

/**
Обчислити квадратний корінь з числа
використовуючи схему апроксимації Ньютона.

Параметри:
    x = число для піднесення в квадрат
    
Повертає: квадратний корінь з x 
*/
auto sqrt(T)(T x) {
    // наш епсилон, коли зупинити апроксимацію,
    // тому що ми вважаємо, що обмін не
    // потребує більше ітерацій.
    enum GoodEnough = 0.01;
    import std.math : abs;
    // вибраємо гарне стартове значення.
    T z = x*x, old = 0;
    int iter;
    while (abs(z - old) > GoodEnough) {
        old = z;
        z -= ((z*z)-x) / (2*z);
    }

    return z;
}

void main() {
    double n = 4.0;
    writeln("Корінь квадратний
        з 4 у runtime = ", sqrt(n));
    static cn = sqrt(4.0);
    writeln("Корінь квадратний
        з 4 у compile time = ",
        cn);
}
```