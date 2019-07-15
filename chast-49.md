# Часть 49

 На протяжении этого курса мы видели основы использования **IDA** для выполнения статического реверсинга, распаковки, эксплоитинга. У нас остаются ещё несколько тем для обсуждения, но внутри темы об эксплуатации у нас остается одна из наиболее тяжелых форм эксплуатации и которая пугает большое количество людей. Мы рассмотрим атаку **USE** **AFTER** **FREE**.  
  
Мы в довольно полной форме в нескольких упражнениях рассмотрели тему переполнений буфера, как эксплуатировать и переполнять буферы в стеке и куче и рассмотрели видеоролики, которые мы загрузили на ютуб.  
  
Для того, чтобы попытаться понять атаку **USE** **AFTER** **FREE**, мы будем использовать упражнение **EXAMEN** **20** в качестве примера поскольку его действительно можно эксплуатировать только как **USEAFTER** **FREE.** В коде нет переполнения и все буферы всегда остаются правильными. Если это так, то как мы можем эксплуатировать уязвимость и отклонить выполнение программы?  
  
Чтобы понять это, Вам нужно сначала загрузить исходный код **EXAMEN** **20**.  
  
[https://drive.google.com/file/d/0B13TW0I0f8O2WDRfQTlvT1JoNnc/view?usp=sharing](https://drive.google.com/file/d/0B13TW0I0f8O2WDRfQTlvT1JoNnc/view?usp=sharing)  
  
Исполняемый файл, который зазипован и имеет пароль **a** находится здесь:  
  
[https://drive.google.com/file/d/0B13TW0I0f8O2N1gtZWNianZpNnM/view?usp=sharing](https://drive.google.com/file/d/0B13TW0I0f8O2N1gtZWNianZpNnM/view?usp=sharing)  
  
И символы  
  
[https://drive.google.com/file/d/0B13TW0I0f8O2c3RRUUpJTFJMaEE/view?usp=sharing](https://drive.google.com/file/d/0B13TW0I0f8O2c3RRUUpJTFJMaEE/view?usp=sharing)  
  
Символы должны быть переименованы и иметь те же имена как у исполняемого файла, но только с расширением **PDB**  
  
![1.png](https://wasm.in/attachments/1-png.3594/) ****  
  
В исходном коде, класс называется **EMPLEADOS.** Его конструктор называется **EMPLEADOS**::**EMPLEADOS** и виртуальные методы, которые говорят **MAL** и **PRONTO**, будут функциями, которые будут использоваться экземплярами этого класса.  
  
Пока здесь всё понятно. Давайте посмотрим экземпляры в функции **MAIN**.  
  
![2.png](https://wasm.in/attachments/2-png.3595/)   
  
Мы видим два экземпляра с именами **PEPE** и другой **JOSE** того же класса **EMPLEADOS.** То же самое, в этом случае, выполняется используя функцию **NEW\(\)**, которая очень похожа на **MALLOC**, но более ориентированна на создание, в этом случае, этих экземпляров, резервируя необходимую память в кучи. \(Мы помним, что когда мы трассировали функцию **NEW\(\)** мы приходили в функцию **MALLOC**\).  
  
![3.png](https://wasm.in/attachments/3-png.3596/)   
  
В исполняемом файле есть два вызова **NEW\(\)** для создания обоих экземпляров **PEPE** и **JOSE**.  
  
И если мы заглянем во внутрь вызова **NEW,** мы увидим, что вызывается функция **MALLOC** с размером, который идёт в качестве аргумента. В этом случае **416** десятичных байт или **0x1A0,** является размером, который необходим каждому экземпляру.  
  
![4.png](https://wasm.in/attachments/4-png.3597/)   
  
![5.png](https://wasm.in/attachments/5-png.3598/)   
  
Мы знаем, что на низком уровне экземпляр подобен переменной структурного типа и должен иметь место для хранения всех атрибутов класса, которые эквивалентны полям структуры. Мы видим тип **INT** для **SALARIO**\_**ACTUAL**, и в публичной части буфер из **200** десятичных байт называемый **CADENA.** Другой буфер называется **NAME** и мы видим, также, объявление виртуальных методов.  
  
Обычно внутри конструктора, на первом месте пространства зарезервированного для каждого экземпляра сохраняется указатель на таблицу адресов виртуальных методов под названием **VTABLE\(ВИРТУАЛЬНАЯ ТАБЛИЦА\)** и в каждом экземпляре будет указатель на ту же таблицу или **VTABLE**.  
  
В предыдущем упражнении **№** **19**, которое также было с классами, хотя там не было оператора **NEW** потому что экземпляр был переменной в стеке, а не в куче, конструктор был хорошо виден.  
  
![6.png](https://wasm.in/attachments/6-png.3599/)   
  
И поскольку внутри него же в первом **DWORD** сохраняется указатель на **VTABLE**.  
  
![7.png](https://wasm.in/attachments/7-png.3600/)   
  
Который указывает на виртуальные методы. Сейчас в этом примере конструктор в **IDA** не рассматривается, если он существует, поскольку мы имеем символы, который программа должна вызывать, называется **EMPLEADOS::EMPLEADOS,** а этот метод не существует.  
  
![8.png](https://wasm.in/attachments/8-png.3601/)   
  
Хорошо. Что происходит, так это то, что компилятор, как я вижу, говорит, что конструктор очень маленький и почти ничего не делает при оптимизации. Я исключаю этот метод и заменяю его инструкциями, которые содержат:  
  
![9.png](https://wasm.in/attachments/9-png.3602/)   
  
Конструктор только устанавливает этот атрибут в нуль и также должен установить **VTABLE** сразу после оператора **NEW,** который создаёт экземпляр. Давайте посмотрим на это в **IDA**.  
  
![10.png](https://wasm.in/attachments/10-png.3603/)   
  
Здесь конструктор делает две вещи - устанавливает в нуль этот атрибут и устанавливает **VTABLE** для её последующего использования.  
  
![11.png](https://wasm.in/attachments/11-png.3604/)   
  
Таким образом, конструктор сохраняет в первое место зарезервированной память каждого экземпляра, указатель на эту таблицу или **VTABLE**.  
  
Прежде чем реверсить полностью пример, что мы увидим в соответствующем видео, давайте посмотрим как выглядит механизм уязвимости **USE** **AFTER** **FREE** и как он эксплуатируется.  
  
Внутри программы, в соответствии с определенными условиями, объект удаляется с помощью оператора **DELETE\(\)**, что эквивалентно функции **FREE\(\)**, экземпляра **PEPE** или **JOSE**.  
  
![12.png](https://wasm.in/attachments/12-png.3605/)   
  
![13.png](https://wasm.in/attachments/13-png.3606/)   
  
После прекращения существования любого из этого экземпляра кому-то в голове придет просить заработную плату всех сотрудников, чтобы составить полные затраты и для этого он использует виртуальный метод get\_Salario примененный к каждому экземпляру.  
  
![14.png](https://wasm.in/attachments/14-png.3607/)   
  
Предположим, что **PEPE** это экземпляр, который удаляется. Когда вы пытаетесь вызвать функцию **GET\_SALARIO**, программа будет искать в своем блоке, который был освобожден \(**FREE**\) и попытается использовать указатель на **VTABLE**, который был там внутри и попытается перейти к методу **GET\_SALARIO**, но поскольку блок освобожден, возможно что программа продолжит выполнение, выделит её там, если она запрашивается и перезапишет указатель на **VTABLE**.  
  
![15.png](https://wasm.in/attachments/15-png.3608/)   
  
Здесь, ниже, мы видим вызовы **GET\_SALARIO.** Каждый экземпляр будет искать внутри в первую очередь его указатель на **VTABLE** и пробовать перейти на этот метод, но эксплуатация состоит в попытке увидеть какой размер удаляемого объекта, который удаляется и выделить тот же размер и заполнить с помощью наших фруктов блок, который ранее занимал экземпляр **PEPE**, и сейчас он будет заполняться нашими фруктами. Таким образом, мы переходим к функции **GET\_SALARIO** так как мы переписали его указатель на **VTABLE.** Мы будем перенаправлять выполнение куда захотим.  
  
Мы увидим что код создается вручную, без создания скрипта.  
  
Я запускаю пример в **IDA**. Я устанавливаю **BP** на **NEW\(\)**.  
  
Я продолжаю использовать **WINDOWS** **7,** которая на данный момент уверяет меня, что только с **MALLOC** я могу перезаписать занятый блок**.** Мы увидим как это сделать в **WINDOWS10**.  
  
![16.png](https://wasm.in/attachments/16-png.3609/)   
  
Я остановился здесь. Я ввожу **416** байт. Прохожу оператор **NEW** с помощью **F8**.  
  
![17.png](https://wasm.in/attachments/17-png.3610/)   
  
В моём случае, программа выделяет мне блок, который начинается по адресу **0x65B510**  
  
![18.png](https://wasm.in/attachments/18-png.3611/)   
  
В первом **DWORD** сохраняется указатель на **VTABLE.** Регистр **EBX** указывает туда где она будет сохраняться.  
  
Здесь находится **VTABLE.** Но мы помним, что в выделенной памяти есть указатель на **THIS**.  
  
![19.png](https://wasm.in/attachments/19-png.3612/)   
  
Давайте пойдём в другой вызов **NEW\(\).**  
  
![20.png](https://wasm.in/attachments/20-png.3613/)   
  
Второй экземпляр **JOSE** будет расположен, в моём случае, по адрес **0x65E000** и ниже будет сохраняться его указатель на **VTABLE**.  
  
![21.png](https://wasm.in/attachments/21-png.3614/)   
  
Конечно, этот указатель находится во втором экземпляре, но указывает на ту же самую **VTABLE**, которая была в первом.  
  
![22.png](https://wasm.in/attachments/22-png.3615/)   
  
Давайте продолжать.  
  
Затем есть часть кода, которая говорит, что мы вводим резюме сотрудников и там есть функция **FGETS**.  
  
![23.png](https://wasm.in/attachments/23-png.3616/)   
  
Мы можем поместить **BP** чуть ниже, на вызове **DELETE** и нажать **RUN**.  
  
![24.png](https://wasm.in/attachments/24-png.3617/)   
  
Я ввожу короткое значение.  
  
![25.png](https://wasm.in/attachments/25-png.3618/)   
  
Когда я нажимаю **ENTER** программа достигает оператора **DELETE\(\)**  
  
Аргумент оператора **DELETE** это адрес экземпляра, который нужной удалить.  
  
![26.png](https://wasm.in/attachments/26-png.3619/)   
  
Программа удалит блок из адрес **0x65B510** который был первым, т.е. **PEPE**.  
  
Если я трассирую функцию **DELETE,** я вижу, что отладчик достиг функции **FREE** освобождая память по адресу **0x65B510**.  
  
![27.png](https://wasm.in/attachments/27-png.3620/)   
  
Затем программа просит меня ввести длину анкеты.  
  
![28.png](https://wasm.in/attachments/28-png.3621/)   
  
И это значение будет тем, что вы используете для выделения и туда будете копироваться значения, чтобы вызвать **USE** **AFTER** **FREE.** Программа должна передать тот же размер удаленных инструкций, т.е. **416** десятичных.  
  
Если я установлю **BP** на функции **MALLOC** и нажму **RUN** я введу программе этот размер **416**  
  
![29.png](https://wasm.in/attachments/29-png.3622/)   
  
При нажатии **ENTER** я дохожу до вызова функции **MALLOC.**  
  
Я вижу, что программа будет делать вызывать **MALLOC\(416\)**. Если программа вернет мне тот же адрес памяти, который я освободил то я пойду в правильном направалении, иначе ничего не получится.  
  
![30.png](https://wasm.in/attachments/30-png.3623/)   
  
Я выделяю в том же адресе. Сейчас я должен только скопировать мои фрукты туда. Они вводятся с помощью **FGETS.** Я нажимаю **RUN,** помещаю **BP** на **PRINTF**.  
  
![31.png](https://wasm.in/attachments/31-png.3624/)   
  
![32.png](https://wasm.in/attachments/32-png.3625/)   
  
Я ввожу мои фрукты и при нажатии **ENTER**.  
  
![33.png](https://wasm.in/attachments/33-png.3626/)   
  
Программа пытается найти указатель на **VTABLE** **PEPE** по адресу **0x65B510** и здесь я заполняю буфер моими символами **A**.  
  
![34.png](https://wasm.in/attachments/34-png.3627/)   
  
Здесь мы видим, как отклоняется выполнение кода, которое контролируется фруктами, которые мы вводим.  
  
Это очевидно является простым примером в сложной программе. Вещь это более сложная для реализации, но идея состоит в том, что важно четко понимать, концепцию того как эксплуатируется баг.  
  
Мы увидим, что он будет глубоко отреверсен, в соответствующем видео на ютубе.  
  
  
**=======================================================  
Автор текста: Рикардо Нарваха** - **Ricardo** **Narvaja** \(**@ricnar456**\)  
Перевод на русский с испанского: **Яша\_Добрый\_Хакер\(Ростовский фанат Нарвахи\).**  
Перевод специально для форума системного и низкоуровневого программирования — **WASM.IN  
06.08.2018  
Версия 1.0**
