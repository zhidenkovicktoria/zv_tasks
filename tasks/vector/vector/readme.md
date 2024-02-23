# Вектор
_"Как проверить, что человек знает C++? Попросить его написть свой std::vector"_ – Илья Мещерин.

## Пререквизиты

- [lists/list](/tasks/lists/list)
- [tree/bst](/tasks/tree/bst)
---

В этой задаче напишем свой [std::vector](https://en.cppreference.com/w/cpp/container/vector).

---

*vector* – структура данных, в которой последовательно хранятся элементы одного типа. Другое название - массив динамической длины.

## Сложность операций

Вектор позволяет вставлять элемент в конец в среднем за O(1), искать за O(N), получать доступ к элементу за O(1). Удаление происходит за O(N).

## Capacity и Size
`Capacity` (объём) показывает сколько элементов может быть вставлено в буфер данных.

`Size` (размер) показывает, сколько элементов лежит в буфере сейчас.

## Реаллокации (realloc)
Зачастую необходимо увеличить размер буфера, сохранив при этом текущие данные. Для этого придётся выделить буфер в два раза больше текущего, переложить туда элементы и удалить старый буфер.

## Placement new

Напомню работу обычного оператора `new`:
1) Выделить память размера size
2) Вызвать конструктор на эту память.

Т.е. по итогу вы получаете готовые объекты.

Это не всегда то, чего мы хотим. Что, если в конструкторе у нас происходит захват ресурсов: соединения к базе данных, память, общие данные в мьютексе? В таком случае нам нужно просто выделить память (malloc), а дальше, когда пользователь захочет, сконструировать на эту память объект. Делается это при помощи операции [placement new](https://www.geeksforgeeks.org/placement-new-operator-cpp/)

## Копирование
При реаллокации необходимо скопировать элементы на уже выделенную память.

Правильным решением будет использовать [`std::uninitialized_copy`](https://en.cppreference.com/w/cpp/memory/uninitialized_copy), которая копирует элементы на уже выделенную сырую память.

## EmplaceBack

Если PushBack копирует существующий элемент в конец вектора, либо перемещает его туда при помощи `std::move`, то EmplaceBack сразу конструирует объект в векторе. Для этого метод принимает параметры для конструктора объекта при помощи шаблонов переменной длины. Обратите внимание, что параметры принимаются по универсальной ссылке!

## PushBack

Обратите внимание, что PushBack принимает параметры `по значению`. Это разумное поведение: пользователь либо хочет скопировать объект в вектор, либо переместить туда. В обоих случаях вызовется эта версия.

### Почему O(1), если у нас есть реаллокации стоимостью O(N)

Важно, что O(1) `в среднем!`

[Доказательство, что push_back() работает в среднем за O(1)](https://cs.stackexchange.com/questions/9380/why-is-push-back-in-c-vectors-constant-amortized)

## delete[]

Напомню, как работает `delete[]`:
1) Вызвать деструкторы объектов по всему массиву
2) Вызвать `free` на массив

В нашем случае на массив чаще всего выделено больше памяти, чем в нём хранится элементов (`capacity >= size`). Как было описано выше, мы не хотим создавать объект при выделении памяти - создаём только когда пользовать пожелает вставить элемент.

Следовательно, только size элементов в нашем массиве - это объекты. Остальное - мусор. Вызвать деструктор по мусору - UB.

Иначе говоря, `delete[]` - нам не подходит. Подумайте, как сделать правильно.

## Задание

Напишите реализацию [Vector](vector.hpp).