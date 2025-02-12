## гост (МАГМА) шифр ( ГОСТ МАГМА шифр )
[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/AjayBadrinath/MAGMA/blob/main/README.md)
[![ru](https://img.shields.io/badge/lang-ru-blue.svg)](https://github.com/AjayBadrinath/MAGMA/blob/main/README.ru.md)
## История

Это шифрование на основе шифра Фиестеля с использованием 64-битных блоков в соответствии со спецификациями, определенными стандартами Российского Союза.
Существует не так много информации об этом шифре, разработанном Советским Союзом в то время, когда АНБ разрабатывало DES-56. Это осталось в качестве альтернативы


## Структура

Это блочный шифр с симметричным ключом и профилем:


    Сеть : Fiestel
    Размер блока : 64 бита
    Размер ключа : 256 бит
    Размер подраздела : 32 бита
    Количество Раундов: 32 Раунда
    S-Box : 8x16
    Размер разделения : 32 бита

![GOSTDiagram](https://github.com/AjayBadrinath/Cryptography/assets/92035508/9f4b7814-ebf1-4e9c-b174-ba1fdb21694c)

## Подробности

### 1.Входные данные

      Ключ: 256 бит
      Сообщение (шестнадцатеричное): n бит (будет разделено на 64-битные блоки)


### Мои комментарии
Ключ должен быть сгенерирован с использованием генератора псевдослучайных битов (256 бит). Пожалуйста, обратитесь к <a href="https://github.com/AjayBadrinath/Cryptography/tree/main/PRNG/Mersenne%20Twister">MersenneTwister</a> <a href="https://github.com/AjayBadrinath/Cryptography/tree/main/PRNG/BBS">BlumBlumShub</a> PRNG в моем репозитории.
<br></br>
Попробуйте настроить скрипт на генерацию 256 псевдослучайных битов. Это будет ваш ключ. Я оставлю это на ваше усмотрение.

### 2.S-BOX

Первоначальное внедрение КГБ (советской версии АНБ) было засекречено. Первоначальные S-боксы были созданы по заказу Советского Союза и держались в секрете от общественности. S-боксы предназначались отдельно для производителей микросхем по ГОСТу (опять же, поскольку КГБ изначально планировал использовать черный ход).

<br></br>

Но.. В рассекреченном GOST_R_3412-2015 есть S-Box, используемый в этой реализации.

Центральный банк Российской Федерации использовал другой S-Box, который предназначался для того, чтобы иметь бэкдоры для взлома КГБ. В идеале S-образный блок является сердцем любого шифра.



### 3.Преобразования :

Есть и другие функции, которые приведены в спецификации, но я явно упоминаю приведенные ниже 2 преобразования. Другие преобразования, приведенные в статье, являются неявными (как в реализованных неявно).


#### 1.T-преобразование.
<hr>
ГОСТ MAGMA использует нелинейную биективную функцию (по сути, причудливый термин для подстановки), являющуюся нелинейной .
Позволь

&#960; быть преобразованием подстановки из S-Box, определенного выше.


Преобразование может быть определено из V<sub>32</sub> -> V<sub>32</sub> (имеется в виду 32-битное отображение векторного пространства)

|| Обратитесь к операции объединения.

V<sub>32</sub> -> V<sub>32</sub> : t(a)=t(a<sub>7</sub>.....|| a<sub>0</sub>) = &#960;<sub>7</sub>||...|| &#960;< sub>0</sub>.

Где a=(a<sub>7</sub>.....|| a<sub>0</sub>) &#x3F5; V<sub>32</sub> , a<sub>i</sub> &#x3F5; V<sub>4</sub> , i=(0...7)


<hr>


#### 2.g-трансформация.


<hr>


V<sub>32</sub> -> V<sub>32</sub> : g[k] (a<sub>1</sub>,a<sub>0</sub>)=t((V<sub>32</sub>(a+k)))<<< 11 ,

Где a<sub>i</sub> &#x3F5; V<sub>32</sub> и ' + ' относятся к сложению по модулю 2<sup>32</sup>
<hr>



### 3.Распределение ключей :


Шифр использует 256-битные ключи и использует итеративный дополнительный ключ для каждого раунда из родительского ключа

Начальные дополнительные ключи для раунда 1-8,9-16,17-24
K1= K255||..||K224
K2= K223||..||K192
. . . .
. . . .
. . . .
K8= K31||.. ||K0

Заключительный раунд 25-32

Измените порядок с K8->K1
Для подведения итогов:

       Раунд (1->8 (вкл.)) : MSB->LSB (32- битное разделение) ===>Фаза возрастания
       Раунд (9->16 (вкл.)) : MSB->LSB (32- битное разделение) ===>Фаза возрастания
       Раунд (17->24 (вкл.)) : MSB->LSB (32- битное разделение) ===>Фаза возрастания
       Раунд (25->32 (вкл.)) : LSB->MSB (32- битное разделение) ===>Фаза убывания
       

### 4. Шифрование:

При этом используется система шифрования Fiestel, в которой мы изначально получаем все подразделы и для удобства сводим двумерную матрицу к 1d

1.Разделяем сообщение на левое и правое (32 бита)

2. Цикл Fiestel раундов до 31 раунда <code>(Left,Right)= (Right,Left^g_function(Right,(key[i])))</code> Это неявная функция G (отличается от функции g)

3. Для последнего раунда выполните <code>((Left^g_function(Right,(key[-1])))<<32 )^Right</code> Это еще одна неявная функция G<sup>*</sup> Функция, определенная в статье.

### 5. Расшифровка:



Расшифровка буквально обратная для шифрования, мы переходим от последнего раунда ко второму
, применяя то же преобразование.


Для раунда 1 выполните <code>((Left^g_function(Right,(key[0])))<<32 )^Right</code>
