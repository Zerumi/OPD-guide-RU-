# Гайд по БЭВМ-NG

### Дисклеймер
**ЭТО НЕ ЯВЛЯЕТСЯ ПОЛНОЦЕННОЙ ЗАМЕНОЙ [МЕТОДИЧЕСКИХ УКАЗАНИЙ](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e) ДЛЯ ВЫПОЛНЕНИЯ ЛАБОРАТОРНЫХ РАБОТ! ВСЮ РАСШИРЕННУЮ ИНФОРМАЦИЮ СМОТРЕТЬ ТАМ**

**Перевод в прогрессе...**

## Отдельно команды
- **[LOOP](loop.md)**
- **[Micro-commands](microcode.md)**

## Reports examples
 You can found examples of reports which we at least partially accepted by teacher in [`reports`](/reports/) folders.

## Содержание

- [Гайд по БЭВМ-NG](#гайд-по-бэвм-ng)
    - [Дисклеймер](#дисклеймер)
  - [Отдельно команды](#отдельно-команды)
  - [Reports examples](#reports-examples)
  - [Содержание](#содержание)
  - [Модификации запуска](#модификации-запуска)
  - [Ассемблер](#ассемблер)
  - [Загрузка `asm` файла в БЭВМ](#загрузка-asm-файла-в-бэвм)
  - [CLI](#cli)
  - [Трассировка](#трассировка)
  - [Адресация](#адресация)
    - [Notes on *Indirect relative*](#notes-on-indirect-relative)
  - [Command execution stages](#command-execution-stages)

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

В основном, ассемблерный код выглядит как показано ниже:
```
[МЕТКА: ] МНЕМОНИКА АРГУМЕНТ
```
*Note:* квадратные скобки означают, что данная часть не является обязательнй. Загляните для просмотра полного свода правил синтаксиса ассемблера на **страницу 51** [методических указаний](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e)

Метки полезны, чтобы указывать на определенное место в памяти. Больше информации в разделе [адресация](#addressing).

Вот несколько *специальных* комманд которые могут оказаться удобными:
|     Команда     |   Описание  |
|-----------------| ------------|
|  `org ADDRESS`  | указывает компилятору разместить следующее значение <br> (значение переменной/код команды) разместится в ячейке <br> по адресу `ADDRESS`. Последующие команды будут размещены после адреса `ADDRESS` друг за другом.
| `word 0x0000,0xffa4,0xfa` | Указывает значение по указанному адресу в формате "как есть". <br> `?` эквивалентен `0x0000` в данном контексте <br> `0x15` эквивалентен `0x0015` |

*Note:* регистр символов не имеет значения; все перечисленные значения: `0xfA51`, `0xFF`, `0xac` допустимы.

Данные специальные комманды не представлены в памяти `БЭВМ`. Они предназначены для помощи в написании ассемблерного кода.

Простой пример ассемблерного кода:

```asm
org 0x04f
VAR1: word 0x45a9
START: add 0xf
sub VAR1
```
Здесь `0x45a9` будет размещен в памяти как есть по адресу `0x04f`; `add 0xf` по адресу `0x050` и так далее.

## Загрузка `asm` файла в БЭВМ
БЭВМ имеет дополнительный параметр для загрузки с кодом: <code>-Dcode=<em>file</em></code> где `file` это любой валидный путь к текущему файлу:
`foobar.asm`, `./keklol`, `/home/foo/kek` -- все перечисленные вариации валидны.

*Note:* расширение `.asm` опционально и **не требуется**. Файл может иметь любое допустимое вашей OS имя.

Итак, чтобы загрузить некий ассемблерный код и провести его трассировку, вам нужно запустить БЭВМ следующим образом:
```
java -jar -Dmode=dual -Dcode=main.asm bcomp-ng.jar
```

## CLI
CLI это довольно крутая модификация. Наконец-то вы сможете писать 16-ричные машинные коды. Серьезно, просто напишите это:
```
ffae
``` 
Это установит ваш `Input Register` значением, которое вы написали. (И даже не нужен кастомный джарник)

Далее, вы можете написать:
```
a
```
и текущее значение `Input Register` будет загружено в `IP` регистр (аналог нажатия кнопки `(F4) Ввод адреса` в `gui` модификации).
Для полного списка команд вам пригодится стандартная запись `help` в консоли.

Вы также можете написать ассемблерный код прямо в БЭВМ:
```
asm
Введите текст программы. Для окончания введите END
add 0xff
sub (0xf1)
end
```

## Трассировка
Запустите БЭВМ в режиме `dual` и загрузите в него ваш `.asm` файл, как это было описано в [этой](#загрузка-asm-файла-в-бэвм) секции.
Чтобы увидеть прелестную таблицу в вашем терминале, нажмите `continue` кнопку в GUI. Вы увидите, как таблица сама собой появлятся в терминале.

После успешного выполнения программы:
1. Скопируйте таблицу из терминала. 
2. Вставьте ее в сырой текстовый файл, например `text.txt`. 
3. Замените все пробелы (` `) на запятые (`,`). 
4. Переименуйте расширение файла на csv: <code>text<strong>.csv</strong></code>.
5. Откройте файл в Microsoft Excel / Libreoffice Calc или ином процессоре таблиц.
6. Скопируйте таблицу из процессора таблиц в программу, где пишется ваш отчет (Microsoft Word / Libreoffice Writer)
7. Не забывайте оформить заголовок таблицы в соответствии с ГОСТ 7.32 (задушнил.....)

## Адресация

Когда вы видите что-то наподобии этого в вашем машинном коде <code>2<strong>E</strong>F5</code> во втором символе (в данном случае `E`) - это означает так называемый режим адресации. Взгляните на таблицу ниже (`L` отвечает за `Label`):

| Машинный код | Имя | Нотация | Пример | Описание |
|--------------|-----|------------|------------|---|
|  0x0-0x7 | Прямая абсолютная | `add $L` <br> `add ADDR` | `add $VAR1` <br> `add 0xf` | Добавляет число, взятое напрямую по адресу `0xf` или из `$VAR1` метки |
|    0xE   | Прямая относительная | `add L` | `add VAR1` <br> `4EFE` | **Поддерживаются только метки!** `IP + 1 + OFFSET`. Имейте в виду, что `IP` указывает на *следующую* команду. Поэтому мы и имеем `+ 1` в описании. Оффсет (смещение) может быть как *положительным*, так и *отрицательным*. Например, следующие коды `0x80`, `0xfe` обозначают **отрицательное смещение**. Давайте обозначимся, что команда `4EFE` (`add`) имеет адресс `0x010`. Соответственно, она указывает на адрес прямо за `4EFE` (`0xFE` это *отрицательное смещение* = `-2`) т.е. `0x009`. По этой логике `4EFF` адресует сама к себе.
|    0x8   | Indirect relative | `add (L)` | `add (VAR1)` | Works like pointers in C/C++. It's like saying: *Hey, look in this box. Here you find paper which tells you where exactly the thing is.* `add` --(Direct relative)--> `VAR1` --(Absolute)--> `value` <br> **MORE DETAILS [BELOW TABLE](#notes-on-indirect-relative)**|
|    0xA   | Indirect autoIncrement | `add (L)+` | `add (VAR1)+` | Same like above but after absolute address has been loaded into register, address *itself* in memory cell is incremented. <br> ```AD = VAR1```<br>```VAR1 += 1``` <br> `VAR1` is a **pointer**. So **pointer** is modified. Yes, we are crazy here, we do pointer arithmetics |
|    0xB   | Indirect autoDecrement | `add -(L)` | `add -(VAR1)` | ```VAR1 -= 1``` <br> ```AD = VAR1``` <br> Again, pointer is modified, **NOT** value it points to |
|    0xC   | Displacement SP | `add &N` <br> `add (sp + N)` | `add &0x4a` <br> `add (sp + 0x4a)` | TODO |
|    0xF   | Direct Load | `add #N` | `add #0xff` | Load specified value into `AC`. Only one byte value can be set with direct load. The sign of value bit-extends i.e. `0xfe` becomes `0xfffe` and `0x7f` becomes `0x007f`

*Note:* more information in [methodical](https://se.ifmo.ru/documents/10180/38002/Методические+указания+к+выполнению+лабораторных+работ+и+рубежного+контроля+БЭВМ+2019+bcomp-ng.pdf/d5a1be02-ad3f-4c43-8032-a2a04d6db12e) **page 22** and **page 32**

### Notes on *Indirect relative*

Let's consider example:
```asm
org 0x10 ; this command is optional because bcomp places program into this address by default
POINTER: word 0x15 ; absolute address of operand
LD (POINTER)

org 0x15
word 0x45a9 ; actual operand
```
Here, `LD (POINTER)` LD will look like this in memory: `0xA8FE`.

`A` means that the command is `LD`, `8` denotes that addressing mode is *indirect relative*
and `FE` is offset. Since offsets are represented in two's complement form, `FE` means (-2).
As you remember, `IP` is already incremented so at this time it points to `0x11 + 0x1 = 0x12`. 
Therefore **LD** will look for content at address `0x12 - 0x2 = 0x10`. Content of that cell will be interpreted as *absolute address* `0x15`
of where to search for *operand*. So the value at address `0x15` (which is `0x45A9` in the example) will be loaded into *Accumulator register*.


## Command execution stages

*Note:* [0, 2] means from 0 to 2 inclusive on both ends

| Stage             | Amount of memory accesses |
| ----------------- | ------------------------- |
| instruction fetch | 1                         |
| address fetch     | [0, 2]                    |
| operand fetch     | 1                         |
| execution         | [0, 1]                    |
| interrupt         | TODO                      |

*Note:* operand fetch is skipped for commands, which have `1` in their **14** bit:

<code>0<b><em>1</em></b>00 0000 0000 0000</code> here `14` bit is ***highlighted*** and set to `1`


P.S. Помогите улучшить этот перевод! Загляните в [оригинальный репозиторий](https://github.com/owl-from-hogvarts/OPD-guide) и помогите с помощью запросов на вытягивание.
