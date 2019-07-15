# Часть 34

 Давайте начнём с эксплуатации и возможного выполнения кода. Конечно, мы должны учитывать смягчения и учиться противостоять новым защитам, которые были добавлены. Иногда мы сможем избежать их, а иногда и нет. Основная идея состоит в том, чтобы обучаться постепенно. Давайте сначала рассмотрим некоторые важные определения.  
  
**ЧТО** **ТАКОЕ** **DEP** ?  
  
![1.png](https://wasm.in/attachments/1-png.2751/)   
  
Предотвращение выполнения данных \(**DEP**\) представляет собой набор аппаратных и программных технологий, который выполняет дополнительные проверки в памяти, чтобы помочь избежать запуска в системе вредоносного кода. В **SERVICE PACK 2** \(**SP2**\) **MICROSOFT** **WINDOWS** **XP** и в **MICROSOFT** **WINDOWS** **XP** **TABLET** **PC** **EDITION** **2005**, как аппаратное, так программное обеспечение применяют **DEP**.  
  
Основное преимущество **DEP** заключается в том, чтобы помочь избежать выполнения кода из страниц данных. Обычно, код по умолчанию не исполняется в куче или стеке. Аппаратное обеспечение **DEP** обнаруживает код, который запускается из этих областей и вызывает исключение, когда он пытается исполниться. Программное обеспечение **DEP** может помочь предотвратить то, что вредоносный код использует преимущество механизма обработки исключений **WINDOWS**.  
  
Давайте оставим определение **DEP** компании **MICROSOFT'у**. На самом деле, существует несколько способов задействовать **DEP**. Один из таких способов, сделать это через Свойства Системы.  
  
![2.png](https://wasm.in/attachments/2-png.2752/)   
  
Сейчас я нахожусь в **WINDOWS** **10** и **DEP** включен для основных программ и сервисов. Это настройки по умолчанию. Это означает, что существуют программы, у которых **DEP** выключен по умолчанию.  
  
Конечно, Вы можете изменить настройки на другую опцию, чтобы все программы имели включенный **DEP**, что, очевидно, помогает немного больше, чтобы избежать исполнение кода.  
  
Помимо этой конфигурации системы, каждая программа можем активировать функцию **DEP** самостоятельно, используя **API**, который **MICROSOFT** предоставляет для этого.  
  
![3.png](https://wasm.in/attachments/3-png.2753/)   
  
![4.png](https://wasm.in/attachments/4-png.2754/)   
  
В общем, скажем, что, защита **DEP** изменяет разрешения страниц, где хранятся данные, стек, куча и т.д. Чтобы избежать этого, мы можем исполнить код там.  
  
Поскольку **DEP** управляется процессом, то у него есть несколько способов запуститься и это можно сделать прям во время выполнения. Мы можем увидеть список процессов с помощью утилиты **PROCESS** **EXPLORER**, у которой есть столбец, показывающий статус **DEP** для каждого процесса.  
  
[https://technet.microsoft.com/en-us/sysinternals/processexplorer.aspx](https://technet.microsoft.com/en-us/sysinternals/processexplorer.aspx)  
  
![5.png](https://wasm.in/attachments/5-png.2755/)   
  
![6.png](https://wasm.in/attachments/6-png.2756/)   
  
Вот нужная нам настройка. Мы должны запустить утилиту под пользователем Администратор. Мы добавляем столбец, сделав правый щелчок в панели и выбрав пункт **SELECT** **COLUMNS**.  
  
![7.png](https://wasm.in/attachments/7-png.2757/)   
  
Мы видим, что у большинства процессов режим **DEP** активирован, а у некоторых других процессов он отключен.  
  
![8.png](https://wasm.in/attachments/8-png.2758/)   
  
Очевидно, процессы системы всегда имеют его включенным и даже некоторые программы сторонних производителей то же имеет **DEP**. Но есть и процессы, где защита отключена.  
  
Хорошо, основной момент здесь такой, что даже если **DEP** включен, он не имеет большого значения, потому что его можно обойти. **DEP** усилится, когда он объединяется с другими защитами, которые мы увидим позже.  
  
Один из основных способов обхода **DEP** является **ROP** или возвратно-ориентированное программирование.  
  
![9.png](https://wasm.in/attachments/9-png.2759/)   
  
Четверо исследователя из Калифорнийского Университета опубликовали статью под названием "**RETURN**-**ORIENTED** **PROGRAMMING**: **SYSTEMS**, **LANGUAGES** **AND** **APPLICATIONS**" где они показали способ обхода этой защиты. Грубо говоря, он заключается в выполнении фрагментов кода, которые уже существуют в самом коде программы. Таким образом нет необходимости инжектировать свой собственный код. Я постараюсь кратко описать технику и сделать пример приложения. Хотя я советую Вам прочитать оригинальную статью, где это все хорошо объясняется.  
  
Необходимо получить маленькие фрагменты кода, оканчивающиеся инструкцией **RET**. В идеале, с одной ассемблерной инструкцией \(в дополнении к **RET**\), внутри программы, которую мы хотим использовать. Эти маленькие фрагменты кода называются гаджетами. С помощью этих фрагментов мы должны как с конструктором **LEGO**, создать код эксплоита или шеллкод. Как только эти фрагменты получены и правильно упорядочены, мы можем запустить их в установленном порядке. Как мы это сделаем? То, что эти гаджеты заканчиваются инструкцией **RET** не являются случайными. Этот метод заключается во вводе адресов гаджетов в стек в правильном порядком исполнения, так чтобы при выходе из исполняющейся функции адрес возврата возвращался в регистр **%EIP**, где эти гаджеты вызывались бы один за другим в правильном порядке.  
  
Основной смысл заключается в том, что, когда **DEP** не включен, и например, мы перезаписываем адрес возврата при переполнении стека, то обычно мы переходим на инструкцию **JMP** **ESP** или **CALLESP**, которая возвращает выполнение на стек и продолжает выполнять мой код, который расположен ниже инструкции **JMP** **ESP**.  
  
Но основная идея **ROP** заключается в том, что вместо перехода на инструкцию **JMP** **ESP**, происходит переход на кусочки кода, называемые гаджетами, которые являются исполняемым кодом программы, которые завершаются инструкцией **RET** \(гаджеты являются частью некоторого модуля, вот почему мы можем их запускать\) и с помощью которых, вы можете по чуть-чуть делать вызовы некоторых **API**, например такие как **VIRTUALPROTECT** или **VIRTUALALLOC**, которые изменяют и дают разрешение на выполнения кода в стеке или куче, т.е там, где находится мой код и наконец происходит переходит на его выполнение.  
  
Другими словами, если эксплоит, который перезаписывал адрес возврата для примера без **DEP** был:  
  
“**A” \* 200 + АДРЕС\_JMP\_ESP + КОД ДЛЯ ВЫПОЛНЕНИЯ**   
  
То, теперь с **DEP,** в том же случае, код должен быть таким:  
  
“**A” \* 200 + ROP + КОД ДЛЯ ВЫПОЛНЕНИЯ**   
  
Где **ROP** должен предоставить моему коду разрешение на выполнение.  
  
Рассмотрим, пару примеров без защиты **DEP**.  
  
![10.png](https://wasm.in/attachments/10-png.2760/)   
  
Здесь у нас есть программа. Она имеет буфер длиной **30** десятичных байт, и получает строку в качестве аргумента, которая копируется с помощью функции **STRCPY** в буфер без проверки длины. Следовательно, аргумент вызывает переполнение буфера.  
  
Также, программа загружает модуль под названием **MYPEPE**.**DLL**. Позже увидим, нужен ли программе этот модуль или нет.  
  
Давайте откроем программу в **ЗАГРУЗЧИКЕ** **IDA**.  
  
![11.png](https://wasm.in/attachments/11-png.2761/)   
  
Мы видим, что программа имеет только два аргумента в функции **MAIN** - **ARGC** и **ARGV**. Мы знаем, что переменная **ARGC** - это количество аргументов, которые мы вводим через консоль. Поэтому, если их не два \(имя исполняемого файла + второй аргумент после него через пробел\) программа будет закрываться, так как условие не выполняется.  
  
![12.png](https://wasm.in/attachments/12-png.2762/)   
  
Если количество аргументов не равно **2,** программа будет переходить в красный блок, и выводить сообщение об ошибке, и придёт к возврату без каких либо действий. А если количество аргументов корректно, т.е. равно **2,** программа будет переходить в зеленый блок, загрузит **DLL** и затем перейдёт в функцию **SALUDA**. Сейчас имена выглядят уродливо. Я иду в меню **OPTIONS→** **DEMANGLENAMES→** **NAMES**.  
  
![13.png](https://wasm.in/attachments/13-png.2763/)   
  
![14.png](https://wasm.in/attachments/14-png.2764/)   
  
Если кто-то не помнит, переменная **ARGC** - это число аргументов, а переменная **ARGV** - это массив указателей. Каждый элемент указывает на строку, которая является аргументом. Другими словами, в случае:  
  
![15.png](https://wasm.in/attachments/15-png.2765/)   
  
По адресу **0x0040109C** регистр **ECX** имеет значение переменной **ARGV**. Т.е. это массив указателей.  
  
**ARGV** = \[указатель\_на\_имя\_исполняемого\_файла, указатель\_на\_аргумент1, указатель\_на\_аргумент2…\]  
  
В **IDA** мы видим, что программа исполняет инструкцию **SHL** **EAX**, **0**, другими словами сдвигает **0** байтов, оставляя регистр **EAX** равным, как и раньше, т.е. **4** в этом случае.  
  
Затем выражение \[**ECX**+**EAX**\] будет возвращать указатель на аргумент. Если регистр **EAX** равен нулю, программа будет возвращать указатель на имя исполняемого файла. Если он будет равен **4,** то поскольку каждый указатель имеет длину **4** байт, то программа будет читать указатель на следующий аргумент и так далее.  
  
В этом случае, поскольку регистр **EAX** равен **4**, то указатель на второй аргумент, который мы ввели, будет в регистре **EDX**, который передаётся как аргумент в функцию **SALUDA**.  
  
![16.png](https://wasm.in/attachments/16-png.2766/)   
  
В функции **SALUDA**, есть аргумент, который является указателем на аргумент и переменная, которая является буфером, куда будет скопирована строка.  
  
![17.png](https://wasm.in/attachments/17-png.2767/)   
  
Поскольку я скомпилировал программу с символами, **IDA** определяет текст как указатель, и что он указывает на строку \(или массив символов\).  
  
Конечно, этот массив может иметь нужный размер, какой мы захотим, так как мы вводим его и нет никакого ограничения или проверки.  
  
Давайте посмотрим буфер.  
  
![18.png](https://wasm.in/attachments/18-png.2768/)   
  
Поскольку я скомпилировал программу с символами, **IDA** обнаружила, что это буфер. Давайте посмотрим ссылки, где программа использует этот буфер.  
  
![19.png](https://wasm.in/attachments/19-png.2769/)   
  
Ссылками являются инструкции типа **LEA**, что является ещё одним ключом к разгадке, если мы чего-то не знаем. Кроме того, буфер используется в качестве назначения функции **STRCPY**, где он будет использоваться как целевой буфер. Затем буфер будет использоваться для печати его содержимого.  
  
![20.png](https://wasm.in/attachments/20-png.2770/)   
  
Поэтому давайте сделаем правый щелчок и выберем пункт **ARRAY**.  
  
![21.png](https://wasm.in/attachments/21-png.2771/)   
  
Здесь нет никаких сомнений. Ниже нет никаких переменных. Что есть ниже, так это переменные **СОХРАНЕННЫЙ** **EBP** и **АДРЕС ВОЗВАРАТА**. Таким образом, нет никаких сомнений, что буфер рассчитан **IDA** хорошо. \(Кроме того, программа имеет символы, которые помогают **IDA** определить, что это буфер, даже без нашей помощи\)  
  
Поэтому, чтобы перезаписать адрес возврата, какого размера должны быть аргументы, которые мы должны ввести?  
  
Я выбираю область, которую я собираюсь заполнить начиная с буфера, оставляя **АДРЕС ВОЗВРАТА**, делаю правый щелчок и выбираю пункт - **ARRAY** без соглашения, просто, чтобы увидеть размер, который должна иметь срока для переполнения.  
  
![22.png](https://wasm.in/attachments/22-png.2772/)   
  
Другими словами, если я введу **36** десятичных байт, я остановлюсь точно в нужном месте, чтобы перезаписать адрес возврата. Поэтому, если бы мой код был таким:  
  
![23.png](https://wasm.in/attachments/23-png.2773/)   
  
Предположительно, строка **0xCCCCCCCC** просто должна перезаписать адрес возврата. Я мог бы попробовать эту строку. Для этого я устанавливаю **IDA** как **JUST** **IN** **TIME** **DEBUGGER**. Я открываю консоль с правами администратора и иду в папку, где находится исполняемый файл **IDA**.  
  
**-I\#** установит **IDA** как отладчик времени исполнения \(**0** - для выключения и **1** - для включения\)  
  
![24.png](https://wasm.in/attachments/24-png.2774/)   
  
![25.png](https://wasm.in/attachments/25-png.2775/)   
  
При запуске скрипта, я вижу, что он переходит к выполнению кода, по адресу **0xCCCCCCCC**, который я помещаю в него же. Этим я перезаписываю адрес возврата.  
  
![26.png](https://wasm.in/attachments/26-png.2776/)   
  
Здесь, я вижу, что сейчас регистр **ESP** указывает чуть ниже значения **0xCCCCCCCC**. Поэтому, если я добавлю больше кода ниже, и вместо перехода на адрес **0xCCCCCCCC,** программа будет переходить на инструкцию **JMP** **ESP**, чтобы начать выполнять указанный код \(Какие же хорошие были времена, когда не было **DEP**\)  
  
Я ищу в списке модулей библиотеку **MYPEPE.DLL**.  
  
![27.png](https://wasm.in/attachments/27-png.2777/)   
  
Мы сделаем щелчок правой кнопкой, чтобы проанализировать нашу библиотеку и загружаем символы для неё. Тем временем, в любом месте кода мы запускаем плагин **KEYPATCHER** и не принимая соглашение, мы видим, что инструкции **JMP** **ESP** соответствует последовательности байтов **FF** **E4**.  
  
![28.png](https://wasm.in/attachments/28-png.2778/)   
  
Когда анализ закончится, мы увидим, что в списке функций появляется функция **MYPEPE**. Я иду к одной из них.  
  
![29.png](https://wasm.in/attachments/29-png.2779/)   
  
Модуль выглядит хорошо. Давайте посмотрим, есть ли здесь инструкция **JMP** **ESP?**  
  
Выбираем пункт **SEARCH FOR→** **SEQUENCE OF BYTES** и вводим значение **FF E4.**  
  
![30.png](https://wasm.in/attachments/30-png.2780/)   
  
Здесь, по адресу **0x004010BA** есть инструкция **JMP** **ESP**. Мы не можем использовать нуль, но поскольку система добавляет нуль в конце, когда программа исполняет функцию **STRCPY**, мы не будем добавлять нуль.  
  
![31.png](https://wasm.in/attachments/31-png.2781/)   
  
Проблема заключается в том, что инструкция **JMP** **ESP** служит только для перехода если мы добавим ещё код ниже. Но мы не можем добавить больше кода из-за того, что адрес инструкции **JMP** **ESP** заканчивается нулем. Так что мы будем переходить на инструкцию **RET**. Чуть ниже это указатель на нашу строку, которая была передана нами как аргумент.  
  
![32.png](https://wasm.in/attachments/32-png.2782/)   
  
Чуть ниже адреса возврата в стеке, у нас есть указатель на нашу текстовую строку. Поэтому, если мы перейдём к инструкции **RET**, программа будет возвращаться к моему коду, потому что эта инструкция **RET** вернет программу туда используя это указатель, как если бы это был снова адрес возврата.  
  
![33.png](https://wasm.in/attachments/33-png.2783/)   
  
Скажем так - «Это хороший **RET**». Давайте добавим его.  
  
![34.png](https://wasm.in/attachments/34-png.2784/)   
  
Давайте теперь попробуем с ним.  
  
![35.png](https://wasm.in/attachments/35-png.2785/)   
  
Я вижу, что программа уже совершает переход, чтобы выполнить мой код **CCCCCCCC**, который я добавил как шеллкод. Сейчас, я бы мог перестроить код, который я хотел бы туда поместить, и выполнить то, что захочу я захочу, потому что здесь нет **DEP**. Единственное, что у программы мало свободного места, потому что я выделил буфер только длиной **30** байт, который мешает мне делать большие вещи.  
  
Я подготовил шеллкод, который запустит калькулятор:  
  
  
**import** struct shellcode =  
**"\xB8\x40\x50\x03\x78\xC7\x40\x04"** + **"calc"** + **"\x83\xC0\x04\x50\x68\x24\x98\x01\x78\x59\xFF\xD1"**  
  
fruta = shellcode + **"A"** \* \(36-len\(shellcode\)\) + **"\x3a\x10\x40"**  
  
\#0x40103a ret  
  
![36.png](https://wasm.in/attachments/36-png.2786/)   
  
Программа будет «падать», но уже после запуска калькулятора, который является нашей целью.  
  
В следующих частях, мы добавим немного больше упражнений. Некоторые из них будут с **DEP**. Мы также научимся работать с **ROP** и так будем идти шаг за шагом до самой победы.  
  
До встрече в 35 части.  
  
====================================================================  
Автор текста: **Рикардо Нарваха** - **Ricardo** **Narvaja** \(**@ricnar456**\)  
Перевод на английский: **IvinsonCLS \(@IvinsonCLS\)**  
Перевод на русский с испанского+английского: **Яша\_Добрый\_Хакер\(Ростовский фанат Нарвахи\).**  
Перевод специально для форума системного и низкоуровневого программирования — **WASM.IN  
12.03.2018  
Версия 1.0**
