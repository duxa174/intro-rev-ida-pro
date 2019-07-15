# Часть 55

 Давайте рассмотрим следующий пример. Но поскольку, в этом случае, мы будем изменять значение переменной из ядра, мы должны понимать, что мы делаем. Иначе мы спровоцируем синий экран.  
  
Первую вещь, которую мы будем делать, это загрузим предыдущее упражнение и отладим его с помощью **WINDBG**, чтобы увидеть значение, необходимое для этого примера.  
  
Поскольку, я не говорю, что значение является таковым, потому что это смещение изменяется от системы к системе, несмотря на то оно уведомляет меня о том, что я использую **32**-разрядную **WINDOWS** **7** в качестве целевой машины на данный момент, удобно проверять его при передаче. Мы узнаем больше о структурах, которые обрабатываются.  
  
Как мы видели в предыдущей части, мы запускали целевую машину с помощью **WINDBG**, отлаживали удаленно ядро, как объясняется здесь, и когда запускается система, с помощью **OSRLOADER** мы запускали драйвер, а затем прерывались в **WINDBG**, и, как мы видели, мы меняли процесс на **OSRLOADER** с помощью команды  
  
**.PROCESS /I XXXXXXXX**  
  
, помещая число рядом с именем процесса.  
  
![1.png](https://wasm.in/attachments/1-png.3927/)   
  
В моем случае команда такая:  
  
**.PROCESS /I 840F7D40**   
  
И затем мы нажимаем **G**.  
  
Хорошо. Это известный трюк, о которой мы никогда не говорили. Это адрес структуры \_**EPROCESS.** В моем случае он равен **840F7D40**.  
  
![2.png](https://wasm.in/attachments/2-png.3928/)   
  
Если я сделаю дамп структуры. В моем случае, я буду использовать такую команду.  
  
**DT \_EPROCESS 840F7D40**  
  
Я вижу, что в моем случае по смещению **0xB8** есть структура **ACTIVEPROCESSLINKS**, которая является той, которую я ищу. Это смещение меняется от системы к системе. В **XP** эта структура находится по смещению **0x88**, а в других системах структура также будет менять положение, поэтому хорошо проверить её значение на нашей целевой машине.  
  
Это структуры типа \_**LIST**\_**ENTRY**, как эта. В **32-х** битных системах, имеют длину **8** байт и состоят из двух указателей.  
  
![3.png](https://wasm.in/attachments/3-png.3929/)   
  
Я использую команду **DD**. Вы также можете это увидеть.  
  
![4.png](https://wasm.in/attachments/4-png.3930/)   
  
То, что находится по смещению **0xB8** является первым указателем, который называется **FLINK**, значение которого в моем случае **0x85C1DD20.** А указатель **BLINK**, в моем случае равен **0x84119568**.  
  
Это два поля одной и той же структуры **ACTIVEPROCESSLINK.** Они указывают на ту же структуру только для следующего и предыдущего процесса. Поскольку мы знаем, что эта структура в нашей системе находится по смещении **0xB8**, мы можем найти **EPROCESS** до моего предыдущего процесса и следующего к моему, вычитая из обоих указателей значения **0xB8**.  
  
![5.png](https://wasm.in/attachments/5-png.3931/)   
  
Если мы посмотрим **EPROCESS** других процессов с помощью команды **!PROCESS 0 0,** мы увидим:  
  
![6.png](https://wasm.in/attachments/6-png.3932/)   
  
![7.png](https://wasm.in/attachments/7-png.3933/)   
  
Мы видим, что программа показывает нам **EPROCESS** следующего процесса, а предыдущий это - **OSRLOADER**.  
  
Поэтому **FLINK**, который расшифровывается как **FORWARD** **LINK**, который указывает на ту же структуру **ACTIVEPROCESSLINK** следующего процесса и **BLINK**, который расшифровывается как **BACKWARD** **LINK**, указывает на ту же структуру предыдущего процесса списке. В моей системе **СМЕЩЕНИЕ** для **ACTIVEPROCESSLINK** составляет **0xB8**.  
  
Хорошо. Это то, что нам нужно знать, чтобы понять работу следующего драйвера, который мы будем компилировать.  
  
Как всегда, я буду прикреплять исходный код, но, по существу, к предыдущему драйверу, мы добавим функцию, которая вызывается **HIDECALLER**, и мы изучим, что она делает. Скомпилируем драйвер в режиме релиза и скопируем вместе с символами в папку, чтобы открыть его с помощью **IDA**.  
  
![8.png](https://wasm.in/attachments/8-png.3934/)   
  
Мы видим, что мы добавляем функцию под именем **HIDECALLER.** Если мы открываем драйвер в **IDA** с его символами, мы увидим внутри функции **DRIVERDISPATCH** вызов функции **HIDECALLER**.  
  
![9.png](https://wasm.in/attachments/9-png.3935/)   
  
Мы видим, что это простая процедура.  
  
![10.png](https://wasm.in/attachments/10-png.3936/)   
  
Функция **IOGETCURRENTPROCESS** возвращает указатель на структуру **EPROCESS** нашего процесса.  
  
![11.png](https://wasm.in/attachments/11-png.3937/)   
  
Хорошо. Структура **EPROCESS**, которую мы не имеем в **IDA**, должна быть добавлена вручную. Но с тем, что мы уже знаем про **WINDBG**, мы можем создать пустую структуру длинной **0x300** байт, которой нам не хватает.  
  
Мы видим, что вы создаете пустую структуру с помощью метода, который мы показали ранее в курсе.  
  
![12.png](https://wasm.in/attachments/12-png.3938/)   
  
Мы знаем, что по смещению **0xB8** начинается структура из **8** байтов **ACTIVEPROCESSLINK.** Поэтому мы идем на вкладке структур и добавляем ее.  
  
![13.png](https://wasm.in/attachments/13-png.3939/)   
  
![14.png](https://wasm.in/attachments/14-png.3940/)   
  
Выставляем тип - **DWORD**. Изменим её, поскольку переменная имеет тип **\_LIST\_ENTRY**.  
  
![15.png](https://wasm.in/attachments/15-png.3941/)   
  
Мы синхронизируем структуру **\_LIST\_ENTRY**, а затем возвращаемся к структуре, и мы помещаем курсор в поле, и нажимаем **ALT** + **Q**, чтобы изменить это поле на тип структуры, и выбираем структуру **LIST\_ENTRY**, которая содержит **8** байтов.  
  
![16.png](https://wasm.in/attachments/16-png.3942/)   
  
![17.png](https://wasm.in/attachments/17-png.3943/)   
  
Теперь будет намного красивее.  
  
![18.png](https://wasm.in/attachments/18-png.3944/)   
  
Сейчас я нажимаю **T**. Мы видим, что это поле принадлежит **BLINK**, так как оно находится по смещению **0xBC**.  
  
Метод заключается в том, чтобы перезаписать **FLINK** предыдущего процесса, чтобы он переставал указывать на мой процесс и пропускал его в списке, указывая на следующий, и также перезаписать **BLINK** следующего процесса, и не указывал на мой процесс, который делает это с предыдущим. Таким образом, когда драйвер пройдет через список, он пропустит мой процесс.  
  
![19.png](https://wasm.in/attachments/19-png.3945/)   
  
В этом примере, мы видим, что **FLINK** предыдущего процесса, который указывает на адрес **0x20000000.** Мы перезаписываем это значение на **0x30000000** и **BLINK** следующего процесса, который указывает на адрес **0x20000000**, я перезапишу на значение **0x10000000**.  
  
Таким образом, ни предыдущий, ни следующий процесс, который будет после моего процесса, не будут иметь указателей на мой процесс. Процессы пропустят мой процесс при просмотре списка.  
  
Регистр **EAX** указывает на структуру **ACTICEPROCESSLINK** и имеет тип **\_LIST\_ENTRY.** Мы можем, там где есть инструкция **EAX** + **XXX**, нажать **T** и выбрать структуру **LIST\_ENTRY**, чтобы отобразить ее поле.  
  
![20.png](https://wasm.in/attachments/20-png.3946/)   
  
К регистру **EAX**, который содержится **EPROCESS** добавляется смещение **0xB8.**  
  
![21.png](https://wasm.in/attachments/21-png.3947/)   
  
Содержимое моего указателя **FLINK** перемещается в регистр **ECX**, а затем сохраняется в содержимое регистра **EDX**, который имеет поле **BLINK**, поскольку оно указывает на предыдущий процесс. Его содержимым является **FLINK** предыдущего процесса, так что он делает то, о чем говорилось выше, т. е. Перезаписывается указатель **FLINK** предыдущего процесса моим **FLINK**.  
  
![22.png](https://wasm.in/attachments/22-png.3948/)   
  
Затем появляется другой указатель, который должен записываться в адрес **FLINK** + **4** , так как моё поле **FLINK** равно **0x30000000** + **4** дает **BLINK** следующего процесса по адресу **0x30000004**, и при перезаписи его содержимого мы будем разрушать значение, которое у него было равно **0x10000000**, что является моим **BLINK**.  
  
![23.png](https://wasm.in/attachments/23-png.3949/)   
  
Здесь программа сохраняет **BLINK** моего процесса в содержимом **FLINK** + **4** т.е. в содержимое по адресу **0x30000004**, разрушая значение, которое было в **BLINK** следующего процесса по адресу **0x10000000**.  
  
Регистр **EAX**, который имеет адрес структуры **ACTIVEPROCESSLINK**, в своем содержимом находится мой указатель **FLINK** вы его перезаписываете с тем же адресом и то же самое с моим **BLINK.**Теперь мы будем отлаживаем драйвер, чтобы немного уточнить то о чем я сказал.  
  
Я помещаю здесь **BP**.  
  
![24.png](https://wasm.in/attachments/24-png.3950/)   
  
Я перезагружаю компьютер. Я скопировал новый драйвер и присоединил **WINDBG.** Затем, когда он уже запустится, я закрою его и присоединю **IDA**, как мы это делали ранее.  
  
![25.png](https://wasm.in/attachments/25-png.3951/)   
  
![26.png](https://wasm.in/attachments/26-png.3952/)   
  
Когда я вызываю драйвер из скрипта **PYTHON** **USER**.**PY** который был в предыдущем упражнении, драйвер переходит к обработчику и приходит к вызову **HIDECALLER**.  
  
![27.png](https://wasm.in/attachments/27-png.3953/)   
  
При передаче **API** в регистре **EAX** находится адрес **EPROCESS.** В моем случае он равен **844C1D40.**  
  
![28.png](https://wasm.in/attachments/28-png.3954/)   
  
Давайте проверим с помощью **WINDBG** через командную строку.  
  
![29.png](https://wasm.in/attachments/29-png.3955/)   
  
В этом случае, процесс, который вызвал драйвер, является **PYTHON**.**EXE.** И в командной строке там вы можете увидеть **EPROCESS** по адресу **0x844C1D40**.  
  
Рассмотрим структуру **ACTIVEPROCESSLINKS.**  
  
![30.png](https://wasm.in/attachments/30-png.3956/)   
  
Здесь мы видим, что адрес памяти **EPROCESS** + **0xB8** равен **0x844C1DF8.**  
  
![31.png](https://wasm.in/attachments/31-png.3957/)   
  
Cодержимое это адреса, это указатель **FLINK** который равен **0x82757E98**  
  
![32.png](https://wasm.in/attachments/32-png.3958/)   
  
И следующий **DWORD,** который ниже это **BLINK** который равен **0x84A80828.**  
  
Оба являются **FLINK** и **BLINK** моего процесса.  
  
**FLINK** равен **0x82757DE0, а** **BLINK** равен **0x84a80828.**  
  
![33.png](https://wasm.in/attachments/33-png.3959/)   
  
При трассировке и нажатии **T,** я вижу, что драйвер читает мой **BLINK** и передает его регистр **EDX.**  
  
![34.png](https://wasm.in/attachments/34-png.3960/)   
  
В регистре **EAX** остается адрес структуры **ACTIVEPROCESSLINK.** Его содержимое является **FLINK**, который помещается в **ECX**.  
  
![35.png](https://wasm.in/attachments/35-png.3961/)   
  
Затем драйвер будет копировать мой **FLINK** в содержимое **EDX**, где был мой **BLINK**, который укажет на **ACTIVEPROCESSLINK** предыдущего процесса.  
  
![36.png](https://wasm.in/attachments/36-png.3962/)  
  
Я буду перезаписывать этот **FLINK** предыдущего процесса с помощью моего **FLINK**.  
  
![37.png](https://wasm.in/attachments/37-png.3963/)   
  
Затем поднимаем мой **FLINK**, на который, конечно же, указывает в **ACTIVEPROCESSLINK** следующего процесса и добавляет значение 4 и находим содержимое. Давайте посмотрим следующий процесс.  
  
![38.png](https://wasm.in/attachments/38-png.3964/)  
  
Я перезапишу **BLINK** следующего процесса с помощью **BLINK.**  
  
![39.png](https://wasm.in/attachments/39-png.3965/)   
  
Затем он заканчивает перезапись моего **FLINK** и **BLINK** моего процесса адресом структуры **ACTIVEPROCESSLINK**. Посмотрим, как перечисляются процессы.  
  
Мы видим, что мой собственный процесс указывает на **FLINK**, так и **BLINK** указывают на тот же адрес структуры **ACTIVEPROCESSLINK**.  
  
![40.png](https://wasm.in/attachments/40-png.3966/)   
  
Al hacer !process 0 0 lo mismo en la barra de tareas vemos que el proceso python desapareció de la lista a pesar de que esta corriendo y eso es porque al ir recorriendo la lista y llegar al proceso justo anterior el FLINK del mismo ya no apunta a mi proceso python.exe sino al siguiente, lo saltea, lo mismo que el BLINK del siguiente no apunta mas al proceso python.exe sino al anterior, por eso es como si no existiera mas.\(Не смог тут правильно построить предложение\)  
  
Теперь я запускаю его снова. Значения будут меняться.  
  
PROCESS 844b0d00 SessionId: 1 Cid: 09ac Peb: 7ffd4000 ParentCid: 05f0  
DirBase: 3ec33540 ObjectTable: 92850c90 HandleCount: 93.  
Image: cmd.exe  
  
PROCESS 85da2b48 SessionId: 1 Cid: 0ae8 Peb: 7ffdb000 ParentCid: 01a4  
DirBase: 3ec33560 ObjectTable: 9e02eb90 HandleCount: 51.  
Image: conhost.exe  
  
PROCESS 84a76030 SessionId: 0 Cid: 0d88 Peb: 7ffdc000 ParentCid: 020c  
DirBase: 3ec33580 ObjectTable: 9fe4a3d8 HandleCount: 230.  
Image: taskhost.exe  
  
PROCESS 84bfad40 SessionId: 1 Cid: 0e24 Peb: 7ffd7000 ParentCid: 05f0  
DirBase: 3ec33460 ObjectTable: 9e37c198 HandleCount: 119.  
Image: taskmgr.exe  
  
PROCESS 844c1d40 SessionId: 1 Cid: 0dec Peb: 7ffd9000 ParentCid: 09ac  
DirBase: 3ec334a0 ObjectTable: 9a2ab3c0 HandleCount: 51.  
Image: python.exe  
  
WINDBG&gt;dt nt!\_EPROCESS 844c1d40  
+0x000 Pcb : \_KPROCESS  
+0x098 ProcessLock : \_EX\_PUSH\_LOCK  
+0x0a0 CreateTime : \_LARGE\_INTEGER 0x01d362e6\`34e8bf28  
+0x0a8 ExitTime : \_LARGE\_INTEGER 0x0  
+0x0b0 RundownProtect : \_EX\_RUNDOWN\_REF  
+0x0b4 UniqueProcessId : 0x00000dec Void  
+0x0b8 ActiveProcessLinks : \_LIST\_ENTRY \[ 0x82757e98 - 0x84bfadf8 \]  
  
После запуска драйвера и прохождения через обработчик и функцию **HIDECALLER**  
  
PROCESS 844b0d00 SessionId: 1 Cid: 09ac Peb: 7ffd4000 ParentCid: 05f0  
DirBase: 3ec33540 ObjectTable: 92850c90 HandleCount: 93.  
Image: cmd.exe  
  
PROCESS 85da2b48 SessionId: 1 Cid: 0ae8 Peb: 7ffdb000 ParentCid: 01a4  
DirBase: 3ec33560 ObjectTable: 9e02eb90 HandleCount: 51.  
Image: conhost.exe  
  
PROCESS 84a76030 SessionId: 0 Cid: 0d88 Peb: 7ffdc000 ParentCid: 020c  
DirBase: 3ec33580 ObjectTable: 9fe4a3d8 HandleCount: 230.  
Image: taskhost.exe  
  
PROCESS 84bfad40 SessionId: 1 Cid: 0e24 Peb: 7ffd7000 ParentCid: 05f0  
DirBase: 3ec33460 ObjectTable: 9e37c198 HandleCount: 121.  
Image: taskmgr.exe  
  
Список заканчивается здесь. Процесса **PYTHON**.**EXE** нет.  
  
**=======================================================  
Автор текста: Рикардо Нарваха** - **Ricardo** **Narvaja** \(**@ricnar456**\)  
Перевод на русский с испанского: **Яша\_Добрый\_Хакер\(Ростовский фанат Нарвахи\).**  
Перевод специально для форума системного и низкоуровневого программирования — **WASM.IN  
28.10.2018  
Версия 1.0**
