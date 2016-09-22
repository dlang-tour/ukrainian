# Класи пам'ятi

D являється статично типізованой мовой: з моменту як змінна була оголошена,
ії тип не може бути змінений. Це дозволяє
компілятору запобігати помилок з самого початку і забезпечити дотримання обмежень
під час компіляції. Хороша безпека типів дає підтримку, потрiбну для того,
щоб великі програми були більш безпечними і більш легкими в обслуговуванні.

### `immutable`

На додаток до статичної типiзацiї, D забезпечує
класи пам'ятi, які виконують додатковi
обмеження щодо певних об'єктів. Наприклад,
`immutable` об'єкт може бути тiльки iнiцiалiзован,
алє не може змінюватися пiзнiше.

    immutable int err = 5;
    // або: immutable err = 5 а int буде виведено.
    err = 5; // не скомпiлюється

Таким чином `immutable` об'єкти, можуть бути безпечно розподілені між різними потоками,
тому що вони ніколи не змінюються за визначенням. Це означає, що `immutable`
об'єкти можуть бути беспечно кешованими.

### `const`

Об'єкти `const` теж не можуть бути змінені. Це
обмеження дiє тільки для поточної області видимостi. Покажчик `const`
може вказувати на *mutable* або на
`immutable` об'єкт. Це означає, що об'єкт
є `const` для поточної області, але хтось
ще може змінити його в майбутньому. Просто з `immutable`
ви будете впевнені, що значення об'єкта ніколи не буде
змінити. Для API є звичайним прийняття `const` об'єктiв
щоб гарантувати, що вхідні дані не будуть змiненi.

    immutable a = 10;
    int b = 5;
    const int* pa = &a;
    const int* pb = &b;
    *pa = 7; // заборонено

Як `immutable` так і `const` є _транзитивними_ властивостями, що гарантує застосування
`const` як до самого типу, так i рекурсивно до кожного подкомпоненту цього типу.

### Додатково

#### Основнi посилання

- [Immutable в _Programming in D_](http://ddili.org/ders/d.en/const_and_immutable.html)
- [Областi видимостi в _Programming in D_](http://ddili.org/ders/d.en/name_space.html)

#### Розширеннi посилання

- [const(FAQ)](https://dlang.org/const-faq.html)
- [Квалiфiкатори типу в D](https://dlang.org/spec/const3.html)

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
    mutable = 10; // Fine
    const int* cMutable = &mutable; // Fine
    // ERROR:
    // *cMutable = 100;
    writeln("cMutable: ",
        typeof(cMutable).stringof);

    // ПОМИЛКА:
    // immutable int* imMutable = &mutable;
}
```
