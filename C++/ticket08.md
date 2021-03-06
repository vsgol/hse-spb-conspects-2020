## Билет 08 "Виртуальное наследование"
Автор: Владимир Федоров

В множественном наследовании были проблемы с ромбовидным наследованием (мы имели 2 базовых класса, из-за чего возникала неопределенность во время обращения к полям этого класса). Хотим решение проблем. Для таких случаев есть Виртуальное наследование.

### Синтаксис. Пример.
```C++
struct Person { 
    std::string name;
    Person(...) {
    ...
    std::cout << “Person: 1\n”;
    }
};
struct Student : virtual public Person {
    Student (...) : Person(...), ... { //конструктор Person игнорируется
    ...
    std::cout << “Student: 2\n”;
    }
    ...
};
struct Employee : virtual public Person {
    Employee (...) : Person(...), ... { //конструктор Person игнорируется
    ...
    std::cout << “Employee: 3\n”; 
    }
    ...
};
struct MagicStudent : public Student, public Employee {
    MagicStudent(...) :  Person(...), Student(...), Employee(...) ... {...}
    ... 
};
```

### Порядок инициализации/уничтожения подобъектов и полей, как и где передать параметры конструктором
Теперь у нас один Person и ответственным за его создание является класс MagicStudent. То есть именно вызов конструктора Person в конструкторе MagicStudent создаст единственных объект класса Person.(остальные игнорируются, см Пример)
Если посмотреть на пример, то после вызова конструктора MagicStudent(...) вывод будет такой:
```
Person: 1
Student: 2
Employee: 3
```
### Возможное представление в памяти
Классы Student и Employee имеют указатель на базовый класс Person. Итого получаем, что каждое слово virtual в иерархии это плюс указатель. Есть другой вариант, где Person в начале. Разницы особой нет. Зависит от компилятора.
![ticket8](https://user-images.githubusercontent.com/34138678/83932436-35147200-a7ab-11ea-9870-3ac8fc80586d.png)

### Невозможность простого приведения типа от виртуальной базы к наследнику
Из-за особенностей расположения в памяти не можем сделать так, потому что не можем на этапе компиляции определить, как на самом деле наследуется Person:
```C++
Person *p = new Person();
MagicStudent *s = static_cast<MagicStudent*>(p); // не скомпилируется
MagicStudent *cs = (MagicStudent *)p; // не скомпилируется
MagicStudent *rs = reinterpret_cast<MagicStudent*>(p); // указатель непонятно где
```
Например, в примере выше MagicStudent и Person расположены одним образом относительно друг друга в памяти, а вот например в случае ```class SuperMagicStudent : MagicStudent``` могут быть иным образом расположены, поэтому компилятор не может догадаться, как расположены относительно друг друга классы Person и MagicStudent, поэтому имеем беды с кастами. 

### Использование для интерфейсов (абстрактных классов)
* Абстрактный класс (интерфейс) -- это класс, который имеет в себе только (чисто) виртуальные функции. Полей такой класс не имеет.
* Обычно виртуальное наследование используется именно для таких классов.
* Все конструкторы интерфейсов тривиальны
* При виртуальном наследовании много указателей создаётся
* Каждый метод интерфейса встречается ровно один раз
```C++
struct Person { virtual getName() const; }
struct Student : virtual public Person { virtual getGPA() const; };
struct Employee : virtual public Person { virtual getSalary() const; };
struct final MagicStudent : virtual public Student, virtual public Employee {
    std::string name; virtual getName() override { return name; }
};
```

### Что происходит, если один класс объявляется и виртуальным, и не виртуальным
Все виртуальные схлопываются в одно. Все не виртуальные честно копируются и возникает неоднозначность. 
### Взаимодействие с dynamic_cast
Была проблема с кастами. Проблемы нет, если применить dynamic_cast. Правда, работает только с полиморфными объектами (объектами, которые содержат виртуальные функции) по определению dynamic_cast.
