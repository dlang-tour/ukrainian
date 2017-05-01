# std.parallelism

Модуль `std.parallelism` реалізує примітиви високого рівня для
зручного паралельного програмування.

### parallel

[`std.parallelism.parallel`](http://dlang.org/phobos/std_parallelism.html#.parallel) 
дозволяє автоматично розподіляти основу функції `foreach` для різних
потоків:

    // паралельне піднесення у квадрат даних arr
    auto arr = iota(1,100).array;
    foreach(ref i; parallel(arr)) {
        i = i*i;
    }

`parallel` внутрішньо застосовує оператор `opApply`. Глобальна
`parallel` виступає ярликом для `taskPool.parallel`, яка у свою чергу
є `TaskPool` із використанням робочими потоками *загальної кількості
процесорів - 1 (мінус один)*. Створення власної копії дозволяє
контролювати ступінь паралелізму.

Слідкуйте за тим, щоб основа робочого циклу `parallel` не мала змоги
модифікувати доступні для інших робочих блоків елементи.

Вибірковий `workingUnitSize` визначає кількість елементів, які мають
оброблятися на кожен робочий потік.

### reduce

Функція [`std.algorithm.iteration.reduce`](http://dlang.org/phobos/std_algorithm_iteration.html#reduce)
відома завдяки іншим функціональним контекстам, таким як *accumulate*
або *foldl*. Викликаючи функцію `fun(acc, x)` для кожного елемента `x`,
`acc` вважається попереднім результатом:

    // 0 є "основою (seed)"
    auto sum = reduce!"a + b"(0, elements);

[`Taskpool.reduce`](http://dlang.org/phobos/std_parallelism.html#.TaskPool.reduce)
є паралельним аналогом функції `reduce`:

    // Знаходимо суму діапазону паралельно, використовуючи перший
    // елемент кожного робочого блоку в якості основи.
    auto sum = taskPool.reduce!"a + b"(nums);

`TaskPool.reduce` розділяє діапазон на піддіапазони, які паралельно
поділені пополам. Після розрахунку результату, вони автоматично
зводяться назад.

### `task()`

[`task`](http://dlang.org/phobos/std_parallelism.html#.task) 
є обгорткою для функції, яка триває певний час, будучи виконаною
у власному робочому потоці. Вона також може бути переміщена у
пулі задач:

    auto t = task!read("foo.txt");
    taskPool.put(t);

Або виконана одразу у власному, новому потоці:

    t.executeInNewThread();

Для того щоб отримати результат задачі, необхідно викликати команду
`yieldForce`. Вона буде заблокованою доти, доки результат не стане доступним. 

    auto fileData = t.yieldForce;

### Поглиблення

- [Паралелізм у _Programming in D_](http://ddili.org/ders/d.en/parallelism.html)
- [std.parallelism](http://dlang.org/phobos/std_parallelism.html)

## {SourceCode}

```d
import std.parallelism : task,
    taskPool, TaskPool;
import std.array : array;
import std.stdio : writeln;
import std.range : iota;

string theTask()
{
    import core.thread : dur, Thread;
    Thread.sleep( dur!("seconds")(1) );
    return "Hello World";
}

void main()
{
    // пул задач з двома потоками
    auto myTaskPool = new TaskPool(2);
    // Необхідно зупинити пул задач!
    scope(exit) myTaskPool.stop();

    // Розпочинаємо довготривалу задачу
    // і в той час можемо зайнятися
    // іншими справами...
    auto task = task!theTask;
    myTaskPool.put(task);

    auto arr = iota(1, 10).array;
    foreach(ref i; myTaskPool.parallel(arr)) {
        i = i*i;
    }

    writeln(arr);

    import std.algorithm.iteration : map;

    // Викорисовуємо функцію зменшення,
    // щоб підрахувати сумму
    // всіх кубів паралельно.
    auto result = taskPool.reduce!"a+b"(
        0.0, iota(100).map!"a*a");
    writeln("Сума кубів: ", result);

    // Отримауємо результат, який був 
    // відправлений у фоні до цього.
    writeln(task.yieldForce);
}
```
