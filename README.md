Лабораторная работа 7. Преобразование и анализ кода с использованием Clang и LLVM
Цель работы: Познакомиться с инструментами Clang и LLVM, научиться собирать AST и IR-промежуточное представление кода на C/C++, а также извлекать базовую информацию о программе (например, список функций). Задачи:
1.	Установить Clang и LLVM;
2.	Скомпилировать простой C-файл с использованием clang и получить его: абстрактное синтаксическое дерево (AST), промежуточное представление LLVM IR;
3.	Использовать opt для применения базовой комплексной оптимизации (например, О2);
4.	Построить граф потока управления (CFG) для оптимизированной программы;
5.	Проанализировать результат, сделать выводы и ответить на контрольные вопросы.
Ход работы
1. Установка и подготовка среды
Работа выполнялась в среде Ubuntu 22.04. Установлены следующие инструменты:
•	clang — компилятор языка C/C++.
•	llvm — инструменты анализа и оптимизации кода.
•	opt — инструмент для работы с LLVM IR и применения оптимизаций.
•	Graphviz — инструмент для визуализации кода.
Команда установки:
sudo apt install clang llvm graphviz


![image](https://github.com/user-attachments/assets/cfbaeddc-ac1a-462f-9f18-29c4a4bf20ec)

2. Исходный код
Пример программы main.c:

#include <stdio.h>

int add(int a, int b) {

    return a + b;   
    
}

int main() {

    return add(3, 4);  
    
}

3. Получение AST

Команда для генерации AST:

clang -Xclang -ast-dump -fsyntax-only main.c




![image](https://github.com/user-attachments/assets/9a745fac-2c7c-430f-adba-709e6379a844)

4. Генерация LLVM IR
Команда для генерации LLVM IR:
clang -S -emit-llvm main.c -o main.ll


![image](https://github.com/user-attachments/assets/f2ea8371-c89f-4286-925d-fa5ed64a05c3)

5. Оптимизация IR
Команда для генерации неоптимизированного IR:

clang -O0 -S -emit-llvm main.c -o main_O0.ll

Особенности IR до оптимизации:

•	Все переменные (a, b) размещены в памяти через alloca.

•	Множество операций load и store.

•	Функция add вызывается как отдельная функция.

![image](https://github.com/user-attachments/assets/7f66ccee-3e35-4e5e-b764-48b1d55b7dd3)

Команда для генерации оптимизированного IR с уровнем -O2:

clang -O2 -S -emit-llvm main.c -o main_O2.ll

Оптимизация -O2 включает более 30 различных оптимизаций, таких как:

•	-inline: встраивание небольших функций (встраивает add в main).

•	-constprop: подстановка констант: если аргументы функции известны (например, add(3, 4)), результат (7) вычисляется на этапе компиляции;

•	-mem2reg: перевод переменных из памяти в регистры (SSA).

•	-instcombine: объединение и упрощение инструкций.

•	-simplifycfg: оптимизация структуры блоков.

•	-reassociate, -gvn, -sroa, -dce и другие.

Особенности IR после оптимизации:

•	Функция square исчезла (встроена через -inline и вычислена через -constprop).

•	Переменные, alloca, store, load удалены (-mem2reg, -dce)..


![image](https://github.com/user-attachments/assets/f610d6e1-e674-407f-95e5-3e8b1cca28ee)

Команда для сравнения IR до и после оптимизации:

diff main_O0.ll main_O2.ll

![image](https://github.com/user-attachments/assets/4984b609-7b70-4542-b345-b065ed77849d)

Изменения после оптимизации:

•	Переменные типа alloca удалены.

•	Код переведён в SSA-форму.

•	Улучшена читаемость и упрощён поток управления.

6. Граф потока управления программы
   
Команда для генерации оптимизированного LLVM IR:

clang -O2 -S -emit-llvm main.c -o main.ll

Команда для генерации .dot-файлов CFG:

opt -passes=dot-cfg -disable-output main.ll

![image](https://github.com/user-attachments/assets/3f0584bd-ca1a-40ee-96c4-d13a02e76cdf)

Вывод:
find . -name "*.dot"

./.main.dot

./.square.dot

Создаются DOT-файлы:

•	.main.dot — для функции main.

•	.square.dot — для функции square (если не была удалена оптимизацией).

Команды для преобразования .dot в .png:

dot -Tpng .main.dot -o cfg_main.png

dot -Tpng .add.dot -o cfg_add.png

Команды для просмотра CFG:

xdg-open cfg_main.png

xdg-open cfg_add.png


![image](https://github.com/user-attachments/assets/43f545c0-713c-4367-afae-51e8fdc680c8)

![image](https://github.com/user-attachments/assets/acb66d68-7b0e-4e72-a3c5-575245ce504b)

Выводы

•	С помощью Clang можно получить полную структуру AST, IR и CFG.

•	LLVM предоставляет гибкие инструменты анализа и оптимизации.

•	Промежуточное представление (IR) удобно для написания компиляторных трансформаций.










