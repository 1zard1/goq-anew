
**B-Tree**
<ul>
B-Tree индекс — это точная структура, ускоряющая поиск. Её задача — всегда находить конкретные адреса, по которым вы сможете пойти в основной файл данных таблицы и вычитать нужное значение.

Таблица в PostgreSQL представлена в виде файла данных, лежащего на диске. Для того, чтобы обеспечить логарифмический поиск, имеется дополнительный файл, соответствующий индексу, который помогает искать информацию внутри таблицы. 

Файл с индексом должен всегда находиться в синхронном состоянии с таблицей. Важно знать, что когда мы создаём индекс на таблице, а потом делаем операции изменения данных, например, INSERT, UPDATE или DELETE, то получаем некое замедление. Оно возникает ровно потому, что в синхронном режиме происходит обновление не только самой таблицы, но и индексного файла. 
</ul>

логическое представление B-Tree индекса на примере таблицы, которая содержит произвольные строки. Они могут находиться в хаосе, не обязательно в упорядоченном виде. 
![image](https://github.com/user-attachments/assets/7c776e4f-b01c-4754-a270-12c288512f7a)


**HASH**
<ul>
<li>Хэш индекс использует хэш функцию. Эта функция отображает тип данных в 32-битное целое число - хэш-код. Хорошая хэш-функция раскидывает вычисляемые значения равномерно по всему доступному диапазону (всего около 4 миллиарда значений).</li>

<li>Хэш-код в свою очередь отображается в номер одного из т.н. бакетов. А в бакетах хранится непосредственно маппинг данного хэш-кода на соответствующие строки таблицы.</li>

<li>Простейшая хэш-функция для входящих целых чисел это деление по модулю. Делим число А на число Б и остаток от целочисленного деления считаем хэш-кодом. Например, чтобы раскидать числа по трем бакетам, используем функцию mod(3)</li>

<li>Когда новое значение добавляется в индекс, система применяет к нему хэш-функцию и помещает хэш-код и указатель на кортеж в бакет.
Когда вы выполняете поиск, система берет значение, вычисляет хэш-код, находит нужный бакет, находит ссылку на нужные кортежи.</li>

<li>PostgreSQL использует специальные хэш-функции, которые гарантируют, что значения в бакете могут быть разделены ровно на два бакета. При разделении бакета для индекса выделяется дополнительное хранилище.</li>

<li>Хэш-индексы не рекомендовались к применению до 10 версии PostgreSQL. Если почитаете доки по индексам версии 9.6, то увидите прямое предостережение. Проблема была в том, что хэш-индексы не записывались в WAL - а значит не могли быть использованы в репликах и не восстанавливались автоматически после крэшей. В версии 10 эти проблемы были устранены</li></ul>

***Отличия индексов хэш и b-tree***
<ul>
<li>Хэш индекс занимает меньше места, чем b-tree</li>
<li>Хэш индекс растет инкрементально. В отличие от b-tree, растущего линейно по мере добавления строк, хэш растет рывками. Это происходит в моменты инциализации разделения индекса и аллокации дополнительной памяти под бакеты.</li>
<li>Хэш индекс не зависит от размера индексируемого ключа. Хэш-индекс хранит хэш-коды (целые числа), а b-tree индекс хранит собственно значения.</li>
<li>Хэш-индекс не зависит от селективности индексируемого значения</li>
</ul>

***Параметр fillfactor***
влияет на механизм разделения хэш-индеса. Он определяет соотношение между хранимыми ссылками на кортежи и количеством бакетов. Чем меньше fillfactor, тем более "пустое" пространство в индексе.
Для b-tree это параметр по умолчанию 90, для хэш-индекса - 75

Меньший fillfactor заставляет бакеты делиться раньше, что приводит к большему общему размеру. В отличие от b-tree, который резервирует место под обновляемые строки, новые значения в хэш-индесе могут попасть в любой бакет.

Множество индексов хотя и ускоряют операции чтения, но имеют противоположный эффект при операциях записи.

***Ограничения хэш индекса***
<ul>
<li>Хэш-индекс не может быть использован для организации уникального индекса</li>
<li>Хэш-индекс не может быть использован для составных индексов на несколько столбцов</li>
<li>Хэш-индекс не поддерживает сортировку</li>
<li>Хэш-индекс не годится для поиска по интервалам</li>
<li>Хэш-индекс не дружит с ORDER BY</li>
</ul>

***Итог***
<ul>
Хэш-таблицы - очевидный выбор, когда нужен быстрый поиск данных по ключу. Однако нюансы реализации хэш-индексов до 10 версии PostgreSQL заставили DBA и разработчиков забыть о таком механизме.

Но что мы имеем в свежих версиях:

Хэш-индекс был доработан и теперь абсолютно продакшн-реди.
Хэш-индекс как правило занимает меньше места, чем аналогичный b-tree.
Хэш-индекс незначительно выигрывает в производительности как при вставке, так и при чтении.
</ul>

Другие индексы:
<ul>
  <li>GiST (Generalized Search Tree)
GiST-индексы являются обобщенными и многоцелевыми, предназначены для работы с сложными типами данных, такими как геометрические объекты, текст и массивы. Они позволяют быстро выполнять поиск по пространственным, текстовым и иерархическим данным.</li>
  <li>GIN (Generalized Inverted Index)
GIN-индексы применяются для полнотекстового поиска и поиска по массивам, JSON и триграммам. Они обеспечивают высокую производительность при поиске в больших объемах данных.</li>
  <li>BRIN (Block Range INdex)
BRIN-индексы используются для компактного представления больших объемов данных, особенно когда значения в таблице имеют определенный порядок. Они эффективны для хранения и обработки временных рядов и географических данных.</li>

  <li>Составные индексы в PostgreSQL — это индексы, которые создаются на основе нескольких колонок (больше 2х) таблицы. Они позволяют ускорить выполнение запросов, которые используют несколько полей в условиях фильтрации или сортировки. </li>
</ul>

