# Часть 19

 В этой главе, у нас уже достаточно знаний и умений, чтобы реверсить оригинальный крекми **CRUEHEAD**а. Поэтому открываем его в **ЗАГРУЗЧИКЕ**, выключая опцию **MANUAL** **LOAD**. Исходный файл не упакован, так что нет необходимости загружать его вручную.  
  
![1.png](https://wasm.in/attachments/1-png.1784/)   
  
Загрузчик остановился здесь. В нашем случае, поскольку это не консольное приложение, то нужно не только анализировать функцию **MAIN**. Мы знаем, что в оконных приложениях существует **ЦИКЛ**сообщений, который обрабатывает взаимодействие пользователя с окном, его выполненные щелчки, нажатия клавиш, движения мыши и т.д. и согласно каждому действию пользователя, цикл запрограммирован для выполнения различных действий с помощью этого кода.  
  
Уже знаем, что первую вещь, которую мы должны сделать - это попробовать найти строки. Если из этого ничего не выйдет, мы должны искать **API** функции или функции, которые использует программа. В нашем случае, строки хорошо видны. Так что будем следовать этому пути.  
  
![2.png](https://wasm.in/attachments/2-png.1785/)   
  
Переходя в строку **NO** **LUCK,** мы попадаем в область, где программа принимает какое-то решение. Для этого делаем двойной щелчок по этой строке и попадаем в это место.  
  
![3.png](https://wasm.in/attachments/3-png.1786/)   
  
Я могу видеть перекрёстные ссылки с помощью нажатия на клавишу **X** или **CTRL + X**  
  
После нажатия на эту клавишу, видно, что существует две перекрестные ссылки на эту строку.  
  
![4.png](https://wasm.in/attachments/4-png.1787/)   
  
Давайте посмотрим первую из них.  
  
![5.png](https://wasm.in/attachments/5-png.1788/)   
  
Я закрашиваю этот блок в красный цвет, так как из-за этих инструкций происходит ошибка или плохое сообщение.  
  
![6.png](https://wasm.in/attachments/6-png.1789/)   
  
Давайте посмотрим, как в него можно попасть из программы.  
  
![7.png](https://wasm.in/attachments/7-png.1790/)   
  
Из этой картинки видно, что соседний блок должен вести в хорошее сообщение. Давайте посмотрим, что внутри этой функции по адресу **0x40134D**.  
  
![8.png](https://wasm.in/attachments/8-png.1791/)   
  
После закрашивания, листинг будет выглядеть так.  
  
![9.png](https://wasm.in/attachments/9-png.1792/)   
  
Другая перекрёстная ссылка на строку **NO LUCK** указывает сюда.  
  
![10.png](https://wasm.in/attachments/10-png.1793/)   
  
В другое плохое сообщение можно попасть отсюда.  
  
![11.png](https://wasm.in/attachments/11-png.1794/)   
![12.png](https://wasm.in/attachments/12-png.1795/)   
  
Видим, что аргумент, который передаётся в эту функцию, это адрес \(**OFFSET, СМЕЩЕНИЕ, прим. Яши**\) глобальной переменной, которую мы будем называть **STRING.** Если сделаем щелчок по этой переменной, то перейдём к адресу где она размещена в программе.  
  
![13.png](https://wasm.in/attachments/13-png.1796/)   
  
Видно, что это буфер длиной **3698** байт расположенный по адресу **0x40218E** находящийся в секции **DATA**.  
  
Он имеет две ссылки. Чтобы увидеть, где в коде идёт работа с этой переменной, нажимаем **X.**  
  
![14.png](https://wasm.in/attachments/14-png.1797/)   
  
Видим, что существует перекрёстная ссылка, которая ведёт сюда.  
  
![15.png](https://wasm.in/attachments/15-png.1798/)   
  
**API** Функция **GetDlgItemTextA** используется для ввода каких либо данных в программу. Давайте посмотрим информацию про неё в **MSDN**.  
  
![16.png](https://wasm.in/attachments/16-png.1799/)   
  
Из описания понимаем, что функция помещает в буфер некоторый текст, который введён в контрол с помощью клавиатуры.  
  
![17.png](https://wasm.in/attachments/17-png.1800/)   
  
Видно, что существует две записи с тем же дескриптором **HWND**. Следовательно, я предполагаю, что они должны быть полями для ввода имени пользователя и пароля, которые поступают в крэкми, когда мы нажимаем кнопку в окне **REGISTER**.  
  
![18.png](https://wasm.in/attachments/18-png.1801/)   
  
Также, видно, что они имеют такие номера контрола **nIDDlgItem** : **0x3E8** и **0x3E9**.  
  
![19.png](https://wasm.in/attachments/19-png.1802/)   
  
Используя программу **GREATIS** **WINDOWSE**, можно получить информацию о тех окнах, над которыми находится курсор. Взять её можно здесь.  
  
[http://www.greatis.com/wdsetup.exe  
  
![20.png](https://wasm.in/attachments/20-png.1803/) ](http://www.greatis.com/wdsetup.exe)  
  
Я вижу, что верхний **EDIT** **BOX** **CONTROL** равен **0x3E8,** а нижний - **0x3E9**.  
  
Также, я могу переименовать буферы, в которые попадают введённые строки. Первый будет называться **STRING\_USER**, а второй **STRING\_PASSWORD**. Оба допускают только максимум **0x0B**символов, несмотря на то, что имеют буферы намного больше.  
  
![21.png](https://wasm.in/attachments/21-png.1804/)   
  
Хорошо, мы уже знаем где программа сохраняет введенный аккаунт, который я вводил. Посмотрим, что она будет с ним делать.  
  
Мы уже видели, что программа обрабатывает буфер **STRING\_USER** здесь.  
  
![22.png](https://wasm.in/attachments/22-png.1805/)   
  
Давайте анализировать, что программа делает здесь с этим буфером, но прежде, мы можем изменить имена функций, так как по-видимому первая обрабатывает буфер **STRING\_USER,** а вторая функция обрабатывает буфер **STRING\_PASSWORD.**  
  
![23.png](https://wasm.in/attachments/23-png.1806/)   
  
Сейчас давайте анализировать функцию **PROCESA\_USER**.  
  
![24.png](https://wasm.in/attachments/24-png.1807/)   
  
При переименовании переменной, я использую то же самое имя, которое выбрала **IDA**, хотя я мог бы написать **P**\_**STRING**\_**USER**, так как это также указатель на буфер.  
  
С помощью **SET** **TYPE** я меняю тип функции и её аргументы и обращаю внимание, чтобы аргументы распространились в комментарии.  
  
![25.png](https://wasm.in/attachments/25-png.1808/)   
  
Видим, что пояснение, которое я добавил, совпадает с именем аргумента.  
  
![26.png](https://wasm.in/attachments/26-png.1809/)   
  
Видно, что существует **ЦИКЛ**, который будет читать **БАЙТЫ** буфера **STRING\_USER**. **ЦИКЛ** будет повторяться, пока он не будет равен нулю, т.е. пока не закончится строка и тогда программа сможет перейти на **ЗЕЛЁНУЮ** стрелку.  
  
Здесь Вы видите **ЦИКЛ**. Он увеличивает **ESI**, чтобы читать побайтно каждый символ буфера **STRING\_USER**, и сравнивает каждый из них с числом **0x41**.  
  
![27.png](https://wasm.in/attachments/27-png.1810/)   
![28.png](https://wasm.in/attachments/28-png.1811/)   
  
Мы можем сделать правый щелчок по числу **0x41** и изменить его на символ **A**, который является символом **ASCII** для этого значения.  
  
![29.png](https://wasm.in/attachments/29-png.1812/)   
  
Если значение в **AL** ниже, чем символ **A,** программа перебросит нас в зону **NO** **LUCK**. Если мы посмотрим в таблицу **ASCII**, увидим, что программа не принимает числа в имени **ПОЛЬЗОВАТЕЛЯ**, а только буквы, так как они больше или равны **A.**  
  
![30.png](https://wasm.in/attachments/30-png.1813/)   
  
Так что, программа проверяет, чтобы все символы буфера **STRING\_USER** были больше чем **0x41**, т.е. больше или равны символу **A**.  
  
![31.png](https://wasm.in/attachments/31-png.1814/)   
  
Программа, также, с помощью инструкции **JNB** проверяет, чтобы символ был не ниже символа **Z** и если это так передаёт управление на **БЛОК** по адресу **0x401394**. А если иначе, берет следующий символ и повторяет цикл.  
  
Таким образом, программа обрабатывает все заглавные символы за исключением **Z**. Если символ больше или равен **Z**, то программа переходят в блок по адресу **0x401394**. Давайте посмотрим, что там делает программа.  
  
![32.png](https://wasm.in/attachments/32-png.1815/)   
  
Я назвал эту функцию **RESTA\_20**, потому что это то, что она делает. Если символ больше символа **Z**, программа вычитает из него число, сохраняет результат и выходит из функции.  
  
![33.png](https://wasm.in/attachments/33-png.1816/)   
  
Другими словами, если вы введете символ с кодом **0x61**, что является маленькой буквой “**a**”, программа вычтет из значения **0x20**, и получится значение **0x41**, что является большой буквой “**A**“.  
Программа делает то же самое со всеми символами, которые больше или равны **Z**.  
  
Если символ равен **Z**, то вычитая из него значение **0x20**, получим результат **0x3A**, что является символом двух точек «**:**»  
  
![34.png](https://wasm.in/attachments/34-png.1817/)   
  
Вопрос таков. Можем ли мы уже создать скрипт для **PYTHON**, чтобы получить кейген?  
  
Код \(C++\):

1. user=raw\_input\(\)
2. largo=len\(user\)
3. if \(largo&gt; 0xB\):
4. exit\(\)
5. 6. USERMAY=""
7. 8. for i in range\(largo\):
9. if \(ord\(user\)&lt;0x41\):
10. print "CARACTER INVALIDO"
11. exit\(\)
12. if \(ord\(user\) &gt;= 0x5A\):
13. USERMAY+= chr\(ord\(user\)-0x20\)
14. else:
15. USERMAY+= chr\(ord\(user\)\)
16. 17. 18. print "USER",USERMAY

  
Мы видим, что скрипт делает то же самое, что и программа. Он берет по одному символы строки **ПОЛЬЗОВАТЕЛЯ** и сравниваем их с кодом **0x41**. Если символ меньше, он говорит нам, что это недопустимый символ и переносит нас на **ВЫХОД**. Но если он больше или равен **0x5A,** то программа вычитает из него **0x20** и добавляет его к строке **USERMAY**.  
  
Мы видим, что если я введу имя **pePP,** программа транслирует его в имя **PEPP**.  
  
![35.png](https://wasm.in/attachments/35-png.1818/)   
  
А если я ввожу символ **Z**, скрипт преобразовывает его в символ **«:»** как мы и говорили выше.  
  
![36.png](https://wasm.in/attachments/36-png.1819/)   
  
До этого момента, скрипт делает то же самое, что и программа. Посмотрим, что делает программа после выхода из **ЦИКЛА**. Она продолжает выполняться здесь.  
  
![37.png](https://wasm.in/attachments/37-png.1820/)   
  
Когда крекми найдёт символ, который равен нулю, он покинет **ЦИКЛ** и перейдёт к блоку по адресу **0x40139C.**  
  
![38.png](https://wasm.in/attachments/38-png.1821/)   
  
Видим, что перед увеличением **ESI**, крекми **КЛАДЕТ** его в стек, чтобы сохранить исходное значение, которое указывает на начало строки, и затем с помощью инструкции **POP** **ESI** программа восстанавливает регистр **ESI**, перед тем как войти в функцию по адресу **0x4013С2**.  
  
![39.png](https://wasm.in/attachments/39-png.1822/)   
  
Мы видим, что это **ЦИКЛ**, который складывает все байты, поэтому я буду называть его **SUMATORIA**\(**СУММА**\), так что мы можем добавить этот блок в наш скрипт.  
  
![40.png](https://wasm.in/attachments/40-png.1823/)   
  
![41.png](https://wasm.in/attachments/41-png.1824/)   
  
Скрипт суммирует все байты и печатает сумму.  
  
Чтобы проверить правилен ли скрипт, я помещаю **BP** на следующей строке после вызова функции **SUMATORIA** и ввожу **pepe** в поле **user** и **989898** в поле **password**.  
  
![42.png](https://wasm.in/attachments/42-png.1825/)   
  
И вижу, что получается сумма равная **0x12A**, так что всё работает правильно.  
  
В этой строке, сумма **XOR**ится с помощью ключа **0x5678**, поэтому я добавляю это выражение также в скрипт.  
  
![43.png](https://wasm.in/attachments/43-png.1826/)   
  
Если я выполню эту строчку, скрипт про**XOR**ит сумму и результат будет такой же, как и в программе.  
  
![44.png](https://wasm.in/attachments/44-png.1827/)   
  
![45.png](https://wasm.in/attachments/45-png.1828/)   
  
Затем крекми переносит результат из **EDI** в регистр **EAX** и выходит из этого блока.  
  
![46.png](https://wasm.in/attachments/46-png.1829/)   
  
Затем программа **ПОМЕЩАЕТ** регистр **EAX** в стек и восстанавливает его с помощью инструкции **POP** **EAX** перед окончательным сравнением. Другими словами в инструкции **CMP** **EAX**, **EBX**, первым членом будет это значение, которое поступает из функции **PROCESA\_USER**.  
  
![47.png](https://wasm.in/attachments/47-png.1830/)   
  
Сейчас давайте посмотрим, что программа будет делать с паролем в функции **PROCESA\_PASS**.  
  
Войдя в неё, мы увидим такой код.  
  
![48.png](https://wasm.in/attachments/48-png.1831/)   
  
Здесь программа считывает каждый байт и помещает его в регистр **BL** и вычитает из этого значения число **0x30**, которое остается в регистре в **EBX**, затем умножает **EDI** на **0x0A** и суммирует полученное значение с **EBX**.  
  
Я дополняю следующую часть скрипта с помощью этого кода.  
  
![49.png](https://wasm.in/attachments/49-png.1832/)   
  
Я не ввожу пароль с клавиатуры. Я просто так его тестирую, потому что в кейгене, мы просто вводим пользователя, но я вижу, что если мой пароль равен например числу **989898**.  
  
**SUM2**, это переменная, в которой хранится сумма умноженная на **0xA** и затем к ней прибавляется очередной байт, из которого вычитается значение **0x30**.  
  
![50.png](https://wasm.in/attachments/50-png.1833/)   
  
Выполнив скрипт, я вижу, что для пароля **989898** у меня получается результат **f1aca**, что является **HEX** значением строки **989898**.  
  
![51.png](https://wasm.in/attachments/51-png.1834/)   
  
Всё это в нашем скрипте может сводиться в конвертирование строки в **HEX** с помощью функции **HEX\(\)**.  
  
![52.png](https://wasm.in/attachments/52-png.1835/)   
  
Скрипт даёт мне точно такой же результат.  
  
![53.png](https://wasm.in/attachments/53-png.1836/)   
  
Наконец, скрипт **XOR**ит этот результат с помощью ключа **0x1234** и выходит из блока, чтобы сравнить результат со значением, которое возвратила функция **PROCESA\_USER** в **EAX**.  
  
![54.png](https://wasm.in/attachments/54-png.1837/)   
  
Так что общая формула будет такой:  
  
**HEX\(пароль\) ^ 0x1234 = XOR**  
  
Где **XOR** - это результат, который вернула функция **PROCESA\_USER**.  
  
Немного изменим уравнение:  
  
**HEX\(пароль\) = XOR ^ 0x1234**  
  
Другими словами, если я **XOR**ю результат ключом **0x1234**, я уже почти получаю ответ.  
  
![55.png](https://wasm.in/attachments/55-png.1838/)   
  
Если запустим скрипт со строкой **PEPE**.  
  
![56.png](https://wasm.in/attachments/56-png.1839/)   
  
![57.png](https://wasm.in/attachments/57-png.1840/)   
  
![58.png](https://wasm.in/attachments/58-png.1841/)   
  
Теперь, у нас уже есть кейген. И нет необходимости преобразовывать результат в десятичную форму, потому что **PYTHON** уже сделал это преобразование.  
  
Здесь я копирую код в кейген.  
  
Код \(C++\):

1. sum=0
2. user=raw\_input\(\)
3. largo=len\(user\)
4. if \(largo&gt; 0xB\):
5. exit\(\)
6. 7. USERMAY=""
8. 9. for i in range\(largo\):
10. if \(ord\(user\)&lt;0x41\):
11. print "CARACTER INVALIDO"
12. exit\(\)
13. if \(ord\(user\) &gt;= 0x5A\):
14. userMAY+= chr\(ord\(user\)-0x20\)
15. else:
16. USERMAY+= chr\(ord\(user\)\)
17. 18. print "USER",USERMAY
19. 20. for i in range\(len\(userMAY\)\):
21. sum+=ord \(userMAY\)
22. 23. print "SUMATORIA", hex\(sum\)
24. 25. xoreado= sum ^ 0x5678
26. print "XOREADO", hex\(xoreado\)
27. 28. TOTAL= xoreado ^ 0x1234
29. 30. print "PASSWORD", TOTAL

  
Даже в редких случаях с символом **Z,** получаем такой результат:  
  
![59.png](https://wasm.in/attachments/59-png.1842/)   
  
![60.png](https://wasm.in/attachments/60-png.1843/)   
  
![61.png](https://wasm.in/attachments/61-png.1844/)   
  
Таким образом, мы отреверсили и сделали кейген для крекми **CRUEHEAD**. Теперь мы увидимся с Вами в **20**-той части.  
  
До следующей главы, друзья.  
  
  
Автор текста: **Рикардо Нарваха** - **Ricardo** **Narvaja** \(**@ricnar456**\)  
Перевод на английский: **IvinsonCLS \(@IvinsonCLS\)**  
Перевод на русский с испанского+английского: **Яша\_Добрый\_Хакер\(Ростовский фанат Нарвахи\).**  
Перевод специально для форума системного и низкоуровневого программирования — **WASM.IN  
22.10.2017  
Версия 1.0**
