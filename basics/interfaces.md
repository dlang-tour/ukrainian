# Інтерфейси

D дозволяє визначити тип `interface`, який технічно подібний до `class`,
але функції якого повинні бути обов'язково перевизначені будь-яким
класом, який імплементує цей `interface`.

    interface Animal {
        void makeNoise();
    }

Функція `makeNoise` повинна бути перевизначена класом `Dog`, тому що
він імплементує інтерфейс `Animal`.
По суті `makeNoise` працює як `abstract`-на функція у базовому класі.

    class Dog: Animal {
        override makeNoise() {
            ...
        }
    }

    auto dog = new Animal;
    Animal animal = dog; // неявне приведення до типу інтерфейсу
    dog.makeNoise();

Немає обмежень на кількість `interface`-ів, які можуть бути
імплементовані у `class`-і, але він може наслідуватись лише від
*одного* класу.

### NVI патерн (ідіома не віртуального інтерфейсу)

[NVI патерн](https://en.wikipedia.org/wiki/Non-virtual_interface_pattern)
є альтернативою загальному патерну використання інтерфейсів, який
дозволяє використовувати _не віртуальні_ методи для інтерфейсів.
D дозволяє просто використовувати NVI патерн, визначивши `final`
функції у `interface`, які не можуть бути перевизначені. Це забезпечує
дотримання очікуваної поведінки, якщо перевизначити інші `interface`
функції.

    interface Animal {
        void makeNoise();
        final doubleNoise() // NVI патерн
        {
            makeNoise();
            makeNoise();
        }
    }

### Додатково

- [Інтерфейси в _Programming in D_](http://ddili.org/ders/d.en/interface.html)
- [Інтерфейси в D](https://dlang.org/spec/interface.html)

## {SourceCode}

```d
import std.stdio;

interface Animal {
    /*
      Віртуальна функція,
      яка повинна бути перевизначена!
    */
    void makeNoise();

    /*
      NVI патерн. Внутрішнє використання
      makeNoise для зміни поведінки
      класів-нащадків.
    
      Параметри: 
          n = кількість повторень
    */
    final void multipleNoise(int n) {
        for(int i = 0; i < n; ++i) {
            makeNoise();
        }
    }
}

class Dog: Animal {
    override void makeNoise() {
        writeln("Гав!");
    }
}

class Cat: Animal {
    override void makeNoise() {
        writeln("М'яу!");
    }
}

void main() {
    Animal dog = new Dog;
    Animal cat = new Cat;
    Animal[] animals = [dog, cat];
    foreach(animal; animals) {
        animal.multipleNoise(5);
    }
}
```
