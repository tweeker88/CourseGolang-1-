# Многопоточность

Многопоточность бывает 2-ух видов

* конкуретный - приоритеты исполнения, простой ресурсов, ручная передача. Различные задачи конкурируют за ресурсы.
* параллельный - это подвид многопоточного исполнения программ, в котором множество задач используют ресурсы одновременно

## Пример с браузером

Когда мы заходим на какую-нибудь страницу : должно быть выполнено 2 действия

* загрузка html страницы (файла)
* отрисовка (рендеринг) в окне браузера

Если данные задачи выполняются конкуретно, то сначала вы загрузите необходимый объем файлов, а уже затем выполните отрисовку.
Процессор в этой ситуации будет осуществлять переключение контекста (context switch) в нужный момент (по завершении загрузки) и результат будет ожидаем.
С другой стороны, если бы эти 2 задачи выполнялись параллельно, результат был бы немного шокирующим и непредсказуемым.

В GO поддержка конкуретности реализована с исползованием под-программ (со-программ) , т.н. "горутин" (или corutines/gorutines)

## Что такое горутина

Это фукнция или метод, которая запускает другие функции/методы или выполняет какие-то действия.
Горутина , с технической точки зрения, может восприниматься как легковесный тред. На одном системном потоке может одновременно находиться огромное количество конкурирующих за ресурсы горутин

## Преимущества горутин над классическими тредами

* горутина легковесная (размер горутины в миллионы раз меньше, чем размер классического треда в С++/Java)
* исопльзование большого количество горутин занимает меньшее количество потоков ОС (в отличе от Java/C++, где отдельный тред требует выделения отдельного потока в ОС)
* горутины могут общаться друг с другом используя каналы
* горутина ничего не возвращают в обычном смысле, т.е. нету return
* main - по сути тоже горутина, при ее завершения, все горутины, которые были запущены в ней, тоже завершатся

## Каналы Chan

Средство для общения между горутинами.  
Каналы по умолчанию имеют **zeroValue == nil**, поэтому канылы создают через **make**

Для отправки данных в канал используется:

```go
chanVariable := make(chan int)

chanVariable <- 10
```

Для получения данных из канала используется:

```go
otherVariable := <- chanVariable
```

Отправка и получения данных из канала - блокирующая операция!  
Это означает, что если данные отправлены в канал, то выполнение текущей программы останавливается до тех пор, пока с другой
стороны из этого канала кто-то не считает данные  
Аналогично и в обратную сторону. Если кто-то читает из канала, то выполнение текущей программы (горутины) останавливается до тех пор, пока кто-то в этот канал не отправит данные

### Deadlock

Это ситуация, когда кто-то пишет в канал НО ИЗ НЕГО НИКОГДА НИКТО НИЧЕГО НЕ ПРОЧИТАЕТ, или когда кто-то читает из канала НО В НЕГО НИКТО НИКОГДА НЕ ЗАПИШЕТ

### Направление каналов (direct)

Каналы могут иметь направление

Создания канала только на отправку

```go
directChan := make(chan<- int)
```

Создания канала только на чтение

```go
directChan := make(<-chan int)

```

### Закрытие и итерирование каналов

В Go со стороны получателя можно использовать синтаксис

```go
res, ok := <- ch

```

* res - результат
* ok - закрыт ли канал, если закрыт, то в res будет nil

### Буферизация каналов

Это канал с буфером, в который можно напихать со стороны отправителя не 1 пак данных, а столько, сколько позволяет в буфер

### WaitGroup

Используется для оркестрации горутинами  
По сути WaitGroup - это счетчик горутин.  
Когда горутина запускается делается WaitGroup++  
Когда горутина завершается делается WaitGroup--  
Таким образом когда WaitGroup == 0 делаем вывод, что все горутины отработали!

### Select

Это инструмент, позволяющий выбирать из множества канальных операций (чтение/запись) для множества каналов  
Добавление default страхует от появление deadlock в ходе выполнения и берет работу на себя

### Mutex

Это средства защиты от "Resource Sharing"
Во время работы конкурентных программ, главной точкой является тот факт, что множества горутин не должны одновременно использовать какой-то общий экземпляр (файл, переменную, бд) для одновременных модификаций
Для того, чтобы избежать этой проблемы использую мьютексы (мьютекс блокирует ресурс до тех пор , пока его не осводит одна из горутин)
