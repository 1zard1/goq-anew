A. Камни и украшения [тестовая задача]
Ограничение времени	1 секунда
Ограничение памяти	64Mb
Ввод	стандартный ввод или input.txt
Вывод	стандартный вывод или output.txt
Даны две строки строчных латинских символов: строка J и строка S. Символы, входящие в строку J, — «драгоценности», входящие в строку S — «камни». Нужно определить, какое количество символов из S одновременно являются «драгоценностями». Проще говоря, нужно проверить, какое количество символов из S входит в J.
Это разминочная задача, к которой мы размещаем готовые решения. Она очень простая и нужна для того, чтобы вы могли познакомиться с нашей автоматической системой проверки решений. Ввод и вывод осуществляется через файлы, либо через стандартные потоки ввода-вывода, как вам удобнее.
```go
package main
 
import (
    "fmt"
)
 
func main() {
    var s, j string
 
    fmt.Scanf("%s", &s)
    fmt.Scanf("%s", &j)
 
    seen := map[rune]struct{}{}
    for _, letter := range s {
        seen[letter] = struct{}{}
    }
 
    result := 0
    for _, stone := range j {
        if _, ok := seen[stone]; ok {
            result++
        }
    }
 
    fmt.Println(result)
}
```

B. Последовательно идущие единицы
Ограничение времени	1 секунда
Ограничение памяти	64Mb
Ввод	стандартный ввод или input.txt
Вывод	стандартный вывод или output.txt
Требуется найти в бинарном векторе самую длинную последовательность единиц и вывести её длину.

Желательно получить решение, работающее за линейное время и при этом проходящее по входному массиву только один раз.

Формат ввода
Первая строка входного файла содержит одно число n, n ≤ 10000. Каждая из следующих n строк содержит ровно одно число — очередной элемент массива.

Формат вывода
Выходной файл должен содержать единственное число — длину самой длинной последовательности единиц во входном массиве.

Пример
Ввод	Вывод
5
1
0
1
0
1
1

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strconv"
)

func longestSequenceOfOnes(n int, array []int) int {
    maxLength := 0
    currentLength := 0

    for _, num := range array {
        if num == 1 {
            currentLength++
            if currentLength > maxLength {
                maxLength = currentLength
            }
        } else {
            currentLength = 0
        }
    }

    return maxLength
}

func main() {
    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    n, _ := strconv.Atoi(scanner.Text())

    array := make([]int, n)
    for i := 0; i < n; i++ {
        scanner.Scan()
        array[i], _ = strconv.Atoi(scanner.Text())
    }

    fmt.Println(longestSequenceOfOnes(n, array))
}
```

C. Удаление дубликатов
Compiler	Time limit	Memory limit	Input	Output
All compilers	1 секунда	10Mb	stdin or input.txt	stdout or output.txt
Node.js 14.15.5	1 секунда	20Mb
Oracle Java 7	1 секунда	40Mb
Kotlin 1.4.30 (JRE 11)	1.5 секунд	40Mb
Oracle Java 8	1 секунда	40Mb
Scala 2.13.4	1 секунда	20Mb
OpenJDK Java 15	1 секунда	40Mb
Kotlin 1.1.50 (JRE 1.8.0)	1 секунда	40Mb
Kotlin 1.3.50 (JRE 1.8.0)	1 секунда	30Mb
Kotlin 1.5.32 (JRE 11)	1.5 секунд	40Mb
Node JS 8.16	1 секунда	20Mb
Legend
Дан упорядоченный по неубыванию массив целых 32-разрядных чисел. Требуется удалить из него все повторения.

Желательно получить решение, которое не считывает входной файл целиком в память, т.е., использует лишь константный объем памяти в процессе работы.

Input format
Первая строка входного файла содержит единственное число n, n ≤ 1000000.

На следующих n строк расположены числа — элементы массива, по одному на строку. Числа отсортированы по неубыванию.

Output format
Выходной файл должен содержать следующие в порядке возрастания уникальные элементы входного массива.
```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strconv"
)

func main() {
    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    n, _ := strconv.Atoi(scanner.Text())

    var prev, current int
    first := true

    for i := 0; i < n; i++ {
        scanner.Scan()
        current, _ = strconv.Atoi(scanner.Text())
        if first || current != prev {
            fmt.Println(current)
            prev = current
            first = false
        }
    }
}
```
D. Генерация скобочных последовательностей
Ограничение времени	1 секунда
Ограничение памяти	20Mb
Ввод	стандартный ввод или input.txt
Вывод	стандартный вывод или output.txt
Дано целое число n. Требуется вывести все правильные скобочные последовательности длины 2 ⋅ n, упорядоченные лексикографически (см. https://ru.wikipedia.org/wiki/Лексикографический_порядок).

В задаче используются только круглые скобки.

Желательно получить решение, которое работает за время, пропорциональное общему количеству правильных скобочных последовательностей в ответе, и при этом использует объём памяти, пропорциональный n.

Формат ввода
Единственная строка входного файла содержит целое число n, 0 ≤ n ≤ 11

Формат вывода
Выходной файл содержит сгенерированные правильные скобочные последовательности, упорядоченные лексикографически.

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "sort"
)

func generateParenthesis(n int) []string {
    var result []string
    var generate func(string, int, int)
    generate = func(current string, open int, close int) {
        if len(current) == 2*n {
            result = append(result, current)
            return
        }
        if open < n {
            generate(current+"(", open+1, close)
        }
        if close < open {
            generate(current+")", open, close+1)
        }
    }
    generate("", 0, 0)
    sort.Strings(result)
    return result
}

func main() {
    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    var n int
    fmt.Sscanf(scanner.Text(), "%d", &n)

    result := generateParenthesis(n)
    for _, seq := range result {
        fmt.Println(seq)
    }
}
```

E. Анаграммы
Ограничение времени	1 секунда
Ограничение памяти	20Mb
Ввод	стандартный ввод или input.txt
Вывод	стандартный вывод или output.txt
Даны две строки, состоящие из строчных латинских букв. Требуется определить, являются ли эти строки анаграммами, т. е. отличаются ли они только порядком следования символов.

Формат ввода
Входной файл содержит две строки строчных латинских символов, каждая не длиннее 100 000 символов. Строки разделяются символом перевода строки.

Формат вывода
Выходной файл должен содержать единицу, если строки являются анаграммами, и ноль в противном случае.

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "sort"
)

func areAnagrams(s1, s2 string) bool {
    if len(s1) != len(s2) {
        return false
    }

    s1Runes := []rune(s1)
    s2Runes := []rune(s2)

    sort.Slice(s1Runes, func(i, j int) bool {
        return s1Runes[i] < s1Runes[j]
    })
    sort.Slice(s2Runes, func(i, j int) bool {
        return s2Runes[i] < s2Runes[j]
    })

    for i := range s1Runes {
        if s1Runes[i] != s2Runes[i] {
            return false
        }
    }

    return true
}

func main() {
    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    s1 := scanner.Text()
    scanner.Scan()
    s2 := scanner.Text()

    if areAnagrams(s1, s2) {
        fmt.Println(1)
    } else {
        fmt.Println(0)
    }
}
```
G. Интересное путешествие
Ограничение времени	1 секунда
Ограничение памяти	64Mb
Ввод	стандартный ввод или input.txt
Вывод	стандартный вывод или output.txt
Не секрет, что некоторые программисты очень любят путешествовать. Хорошо всем известный программист Петя тоже очень любит путешествовать, посещать музеи и осматривать достопримечательности других городов.
Для перемещений между из города в город он предпочитает использовать машину. При этом он заправляется только на станциях в городах, но не на станциях по пути. Поэтому он очень аккуратно выбирает маршруты, чтобы машина не заглохла в дороге. А ещё Петя очень важный член команды, поэтому он не может себе позволить путешествовать слишком долго. Он решил написать программу, которая поможет ему с выбором очередного путешествия. Но так как сейчас у него слишком много других задач, он попросил вас помочь ему.

Расстояние между двумя городами считается как сумма модулей разности по каждой из координат. Дороги есть между всеми парами городов.

Формат ввода
В первой строке входных данных записано количество городов 
n
 (
2
≤
n
≤
1
0
0
0
). В следующих 
n
 строках даны два целых числа: координаты каждого города, не превосходящие по модулю миллиарда. Все города пронумерованы числами от 1 до 
n
 в порядке записи во входных данных.
В следующей строке записано целое положительное число 
k
, не превосходящее двух миллиардов, — максимальное расстояние между городами, которое Петя может преодолеть без дозаправки машины.

В последней строке записаны два различных числа — номер города, откуда едет Петя, и номер города, куда он едет.

Формат вывода
Если существуют пути, удовлетворяющие описанным выше условиям, то выведите минимальное количество дорог, которое нужно проехать, чтобы попасть из начальной точки маршрута в конечную. Если пути не существует, выведите -1.

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strconv"
)

type City struct {
    x, y int
}

func abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}

func distance(c1, c2 City) int {
    return abs(c1.x-c2.x) + abs(c1.y-c2.y)
}

func bfs(cities []City, start, end, k int) int {
    n := len(cities)
    visited := make([]bool, n)
    queue := []int{start}
    visited[start] = true
    steps := 0

    for len(queue) > 0 {
        nextQueue := []int{}
        for _, current := range queue {
            if current == end {
                return steps
            }
            for i := 0; i < n; i++ {
                if !visited[i] && distance(cities[current], cities[i]) <= k {
                    visited[i] = true
                    nextQueue = append(nextQueue, i)
                }
            }
        }
        queue = nextQueue
        steps++
    }
    return -1
}

func main() {
    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    n, _ := strconv.Atoi(scanner.Text())

    cities := make([]City, n)
    for i := 0; i < n; i++ {
        scanner.Scan()
        fmt.Sscanf(scanner.Text(), "%d %d", &cities[i].x, &cities[i].y)
    }

    scanner.Scan()
    k, _ := strconv.Atoi(scanner.Text())

    scanner.Scan()
    var start, end int
    fmt.Sscanf(scanner.Text(), "%d %d", &start, &end)
    start--
    end--

    result := bfs(cities, start, end, k)
    fmt.Println(result)
}
```

