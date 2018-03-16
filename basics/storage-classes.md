# Мутабельність

D є статично типізованою мовою: з моменту, як змінна була оголошена,
її тип не може бути змінений. Це дозволяє компілятору запобігати помилок
з самого початку і забезпечити дотримання обмежень під час компіляції.
Хороша безпека типів дає підтримку, потрiбну для того, щоб великі
програми були більш безпечними і більш легкими в обслуговуванні.

### `immutable`

На додаток до статичної типiзацiї, D забезпечує кваліфікатори типу, які 
накладають додатковi обмеження щодо певних об'єктів. Наприклад,
`immutable` об'єкт може бути тiльки iнiцiалiзованим, але не може
змінюватися пiзнiше.

    immutable int err = 5;
    // або: immutable err = 5 і int буде виведено.
    err = 5; // не скомпiлюється

Таким чином `immutable` об'єкти можуть бути безпечно розподілені між
різними потоками, так як вони ніколи не будуть змінюватись за значенням.
Це означає, що `immutable` об'єкти можуть бути беспечно закешовані.

### `const`

Об'єкти `const` теж не можуть бути змінені. Це обмеження дiє тільки для
поточної області видимостi. Вказівник `const` може вказувати на
*mutable* або на `immutable` об'єкти. Це означає, що об'єкт є `const`
для поточної області, але хтось ще може змінити його у майбутньому.
Просто з `immutable` ви будете впевнені, що значення об'єкта ніколи не
буде змінено. Для API є звичайним прийняття `const` об'єктiв, щоб
гарантувати, що вхідні дані не будуть змiненi.

    immutable a = 10;
    int b = 5;
    const int* pa = &a;
    const int* pb = &b;
    *pa = 7; // заборонено

Як `immutable` так і `const` забезпечують транзитивність, яка гарантує
застосування `const` як до самого типу, так i рекурсивно до кожного
компоненту цього типу.

### Додатково

#### Основнi посилання

- [Immutable у _Programming in D_](http://ddili.org/ders/d.en/const_and_immutable.html)
- [Областi видимостi у _Programming in D_](http://ddili.org/ders/d.en/name_space.html)

#### Розширеннi посилання

- [const(FAQ)](https://dlang.org/const-faq.html)
- [Квалiфiкатори типу у D](https://dlang.org/spec/const3.html)

## {SourceCode}

```d
import std.stdio;

void main()
{
    immutable forever = 100;
    // ПОМИЛКА:
    // forever = 5;
    writeln("forever: ",
        typeof(forever).stringof);

    const int* cForever = &forever;
    // ПОМИЛКА:
    // *cForever = 10;
    writeln("const* forever: ",
        typeof(cForever).stringof);

    int mutable = 100;
    writeln("mutable: ",
        typeof(mutable).stringof);
    mutable = 10; // Чудово
    const int* cMutable = &mutable; // Чудово
    // ПОМИЛКА:
    // *cMutable = 100;
    writeln("cMutable: ",
        typeof(cMutable).stringof);

    // ПОМИЛКА:
    // immutable int* imMutable = &mutable;
}
```
