# Гайд по БЭВМ-NG. 

/ ! \ Не является первоисточником. Это перевод соответствующего [англоязычного](https://github.com/owl-from-hogvarts/OPD-guide) гайда. Автор: **Тернавский Константин Евгеньевич**, группа Р3106

### Дисклеймер
**ТО, ЧТО ВЫ ВИДИТЕ НЕ ЯВЛЯЕТСЯ ПОЛНОЦЕННОЙ ЗАМЕНОЙ [МЕТОДИЧЕСКИХ УКАЗАНИЙ](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e) ДЛЯ ВЫПОЛНЕНИЯ ЛАБОРАТОРНЫХ РАБОТ! ПОЖАЛУЙСТА, НЕ ИГНОРИРУЙТЕ МЕТОДИЧКУ**

**Загляните в оригинальный репозиторий: https://github.com/owl-from-hogvarts/OPD-guide**

## Отдельно про команды
- **[LOOP](loop.md)**
- **[Micro-commands](microcode.md)**

## Reports examples
 You can found examples of reports which we at least partially accepted by teacher in [`reports`](/reports/) folders.

## Примеры отчетов по лабораторным работам
Вы можете найти примеры отчетов которые по крайней мере были приняты преподавателем в [`папке`](/reports/) с отчетами.

## Содержание

- [Гайд по БЭВМ-NG.](#гайд-по-бэвм-ng)
    - [Дисклеймер](#дисклеймер)
  - [Отдельно про команды](#отдельно-про-команды)
  - [Reports examples](#reports-examples)
  - [Примеры отчетов по лабораторным работам](#примеры-отчетов-по-лабораторным-работам)
  - [Содержание](#содержание)
  - [Модификации запуска](#модификации-запуска)
  - [Ассемблер](#ассемблер)
  - [Загрузка `asm` файла в БЭВМ](#загрузка-asm-файла-в-бэвм)
  - [CLI](#cli)
  - [Трассировка](#трассировка)
  - [Адресация](#адресация)
    - [Заметки по *Косвенной адресации*](#заметки-по-косвенной-адресации)
  - [Этапы исполнения команды](#этапы-исполнения-команды)

## Модификации запуска

БЭВМ-NG может быть запущено в разных режимах:
 - `gui` -- стандартный режим (GUI)
 - `cli` -- режим командной строки
 - `decoder` -- дамп стандартной микропрограммы (микрокомманд) в терминал
 - `nightmare` -- натурально ночной кошмар, не запускайте ее в этом режиме, иначе получите психологическую травму
 - `dual` -- запускает БЭВМ сразу в двух режимах: `gui` и `cli` одновременно (**рекомендуется к запуску для [трассировки](#трассировка)**)

Чтобы указать режим запуска БЭВМ определите флаг <code>-Dmode=<em>mode_name</em></code>, заменив `mode_name` на название одного из режимов выше. Например:
```
java -jar -Dmode=dual bcomp-ng.jar
```

## Ассемблер
Ассемблерный код может быть написан как в стороннем текстовом редакторе (VScode/Vi Improved), так и в режиме `cli` БЭВМ-NG.

В основном, строчка ассемблерного кода выглядит так, как показано ниже:
```
[МЕТКА:] МНЕМОНИКА АРГУМЕНТ
```
*Note:* квадратные скобки означают, что данная часть не является обязательнй. Для просмотра полного свода правил синтаксиса ассемблера взгляните на **страницу 51** [методических указаний](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e)

Метки нужны, чтобы указывать на определенное место в памяти. Больше информации в разделе [адресация](#addressing).

Вот несколько *специальных* комманд которые могут оказаться удобными:
|     Команда     |   Описание  |
|-----------------| ------------|
|  `org ADDRESS`  | Указывает компилятору разместить следующее значение <br> (значение переменной/код команды) в ячейке <br> по адресу `ADDRESS`. Последующие значения будут размещены после адреса `ADDRESS` друг за другом.
| `word 0x0000,0xffa4,0xfa` | Указывает прямое шестнадцатеричное число по указанному адресу (в формате "как есть"). <br> `?` это `0x0000` в данном контексте <br> `0x15` эквивалентен `0x0015` |

*Note:* регистр символов не имеет значения; все перечисленные значения: `0xfA51`, `0xFF`, `0xac` допустимы.

Данные специальные комманды не представлены в интерпретаторе `БЭВМ`. Они предназначены для помощи в написании ассемблерного кода.

Простой пример ассемблерного кода:

```asm
org 0x04f
VAR1: word 0x45a9
START: add 0xf
sub VAR1
```
Здесь `0x45a9` (VAR1) будет размещен в памяти как есть по адресу `0x04f`; `add 0xf` (400F) по адресу `0x050` и так далее.

## Загрузка `asm` файла в БЭВМ
БЭВМ имеет дополнительный параметр для загрузки с кодом: <code>-Dcode=<em>file</em></code> где `file` это любой валидный путь к текущему файлу:
`foobar.asm`, `./keklol`, `/home/foo/kek` -- все перечисленные вариации валидны.

*Note:* расширение `.asm` опционально и **не требуется**. Файл может иметь любое допустимое вашей OS имя.

Итак, чтобы загрузить некий ассемблерный код и провести его трассировку, вам нужно запустить БЭВМ следующим образом:
```
java -jar -Dmode=dual -Dcode=main.asm bcomp-ng.jar
```

## CLI
CLI это довольно крутая модификация. Наконец-то вы можете писать 16-ричные машинные коды. Серьезно, просто напишите это:
```
ffae
``` 
Это установит ваш `Input Register` значением, которое вы написали. (И даже не нужен кастомный джарник)

Далее, вы можете написать:
```
a
```
и текущее значение `Input Register` будет загружено в командный регистр `IP` (аналог нажатия кнопки `(F4) Ввод адреса` в `gui` модификации).
Для полного списка команд вам пригодится стандартная запись `help` в консоли (кричите о помощи).

Вместо машинных команд вы также можете написать ассемблерный код прямо в БЭВМ:
```
asm
Введите текст программы. Для окончания введите END
add 0xff
sub (0xf1)
end
```

## Трассировка
Запустите БЭВМ в режиме `dual` и загрузите в него ваш `.asm` файл, как это было описано в [этой](#загрузка-asm-файла-в-бэвм) секции.
Чтобы увидеть прелестную таблицу в вашем терминале, нажмите `(F8) Продолжение` кнопку в GUI. Вы увидите, как таблица сама собой появлятся в терминале.

После успешного выполнения программы:
1. Скопируйте таблицу из терминала. 
2. Вставьте ее в сырой текстовый файл, например `text.txt`. 
3. Замените все пробелы (` `) на запятые (`,`). 
4. Переименуйте расширение файла на csv: <code>text<strong>.csv</strong></code>.
5. Откройте файл в Microsoft Excel / Libreoffice Calc или ином процессоре таблиц.
6. Скопируйте таблицу из процессора таблиц в программу, где пишется ваш отчет (Microsoft Word / Libreoffice Writer)
7. Не забывайте оформить заголовок таблицы в соответствии с ГОСТ 7.32 (задушнил.....)

## Адресация

Когда вы видите что-то наподобии этого в вашем машинном коде <code>2<strong>E</strong>F5</code> во втором символе (в данном случае `E`) - это отвечает за так называемый режим адресации. Взгляните на таблицу ниже (`L` отвечает за `Label` - метку):

| Машинный код | Имя | Нотация | Пример | Описание |
|--------------|-----|------------|------------|---|
|  0x0-0x7  | Прямая абсолютная | `add $L` <br> `add ADDR` | `add $VAR1` <br> `add 0xf` | Добавляет число, взятое напрямую по адресу `0xf` или из `$VAR1` метки |
|    0xE (1110)   | Прямая относительная | `add L` | `add VAR1` <br> `4EFE` | **В ассемблере поддерживаются только метки!** `IP + 1 + OFFSET`. Имейте в виду, что `IP` указывает на *следующую* команду. Поэтому мы и имеем `+ 1` в описании. Оффсет (смещение) может быть как *положительным*, так и *отрицательным*. Например, следующие коды `0x80`, `0xfe` обозначают **отрицательное смещение**. Давайте обозначимся, что команда `4EFE` (`add`) имеет адресс `0x010`. Соответственно, она указывает на адрес прямо за `4EFE` (`0xFE` это *отрицательное смещение* = `-2`) т.е. `0x009`. По этой логике `4EFF` адресует сама к себе.
|    0x8 (1000)   | Косвенная относительная | `add (L)` | `add (VAR1)` | Работает как указатели в C/C++. Как будто мы говорим: *Эй, посмотри в эту коробку! В ней можно найти бумажку в которой сказанно где же определенно можно найти то, что ты ищешь.* `add` --(Косвенная относительная)--> `VAR1` --(Абсолютная)--> `value` <br> **ПОДРОБНЕЕ [СНИЗУ](#notes-on-indirect-relative)**|
|    0xA (1010)  | Косвенная относительная автоинкрементная постфиксная | `add (L)+` | `add (VAR1)+` | Тоже самое, что и выше описанное, но после загрузки абсолютного адреса в регистр, этот адрес *увеличивается* в ячейке памяти, где этот адрес хранится. <br> ```AD = VAR1```<br>```VAR1 += 1``` <br> `VAR1` это **указатель**. Так, мы изменяем **указатель**. Да, мы безумцы, мы выполняем арифметику указателей =). |
|    0xB (1011)   | Косвенная относительная автодекрементная префиксная | `add -(L)` | `add -(VAR1)` | ```VAR1 -= 1``` <br> ```AD = VAR1``` <br> Снова, мы меняем указатель, **НЕ** значение, на которое этот указатель ссылается |
|    0xC (1100)  | Смещение относительно SP | `add &N` <br> `add (sp + N)` | `add &0x4a` <br> `add (sp + 0x4a)` | TODO |
|    0xF (1111)  | Прямая загрузка | `add #N` | `add #0xff` | Загружает значение напрямую в `AC`. Только однобайтовое значение может быть загружено прямой загрузкой. Знак операнда расширится т.е. `0xfe` станет `0xfffe` и `0x7f` станет `0x007f` (значение 7го бита склонируется в 8й, 9й,.. 15й)

*Note:* больше информации смотрите в [методичке](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e) **на странице 22** и **на странице 32**

### Заметки по *Косвенной адресации*

Давайте возьмем следующий пример:
```asm
org 0x10 ; конкретно данная команда опциональна, которму что компилятор ассемблера БЭВМ по умолчанию помещает программу ровно по этому самому адресу
POINTER: word 0x15 ; абсолютный адрес операнда
LD (POINTER)

org 0x15
word 0x45a9 ; нужный нам операнд
```
Здесь, `LD (POINTER)` LD в памяти будет выглядеть следующим образом: `0xA8FE`.

`A` отвечает за код операции команды `LD`, `8` описывает, что режим адресации этой команды *косвенная относительная*
и `FE` это оффсет/смещение. Поскольку оффсет интерпретируется как знаковое число, `FE` означает (-2).
Как вы можете помнить, `IP` уже инкрементирован (это происходило в цикле выборки команды) поэтому в данный момент он адресует на `0x11 + 0x1 = 0x12`. 
Поэтому **LD** будет смотреть в значение по адресу `0x12 - 0x2 = 0x10`. Значение этой ячейки будет рассмотрено/интерпретировано как *абсолютный адрес* (а в нашем случае там `0x15`)
того, где же находится наш *операнд*. Так, значение по адресу `0x15` (которое в нашем примере `0x45A9`) будет загружено в *Регистр общего назначения AC*.

## Этапы исполнения команды

*Note:* [0, 2] означает промежуток от 0 до 2 включая концы

|Стадия исполнения|Кол-во доступов к памяти|
|-----|-------------------------|
|Цикл выборки команды| 1 |
|Цикл выборки адреса | [0, 2]|
|Цикл выборки операнда | 1     |
|Цикл исполнения     | [0, 1]|
|Цикл прерывания     | TODO  |

*Note:* Выборка операнда пропускается для всех команд, которые имеют `1` в **14** бите кода команды:

Для примера <code>0<b><em>1</em></b>00 0000 0000 0000</code> здесь `14` бит ***подсвечен*** и установлен в `1`

P.S. Помогите улучшить этот перевод! Загляните в [оригинальный репозиторий](https://github.com/owl-from-hogvarts/OPD-guide) и помогите с помощью запросов на вытягивание.
