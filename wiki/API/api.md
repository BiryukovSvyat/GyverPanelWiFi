## Общие понятия

Управление устройством выполняется путем отправки ему команды через один из доступных каналов - UDP или MQTT.  
Не имеет значения по какому каналу устройство получило команду - обработка выполняется одинаково.  
Команда представляет собой управляющую последовательность, начинающуюся с символа `'$'`, далее идет код операции и параметры.  
Завершается команда символом `';'`. Код команды и параметры разделены символом `' '`(пробел), состав параметров - зависит от конкретной команды.  

```
Команда: $<код> <параметры>; 
```

Сообщения, получаемые по MQTT-каналу, могут состоять из нескольких "склеенных" команд. 
Склеенные команды должны следовать "вплотную" друг к другу - пробелы или другие разделительные символы между командами не допускаются.
После получения "мультикомандного" сообщения его текст разбивается на отдельные команды по разделителю `";$"` и далее выполняются в порядке их получения. 

## Команды управления

    3 - управление играми из приложение WiFi Panel Player
      - $3 0; - включить на устройстве демо-режим
      - $3 1; - включить игру "Лабиринт" в режиме ожидания начала игры
      - $3 2; - включить игру "Змейка" в режиме ожидания начала игры
      - $3 3; - включить игру "Тетрис" в режиме ожидания начала игры
      - $3 4; - включить игру "Арканоид" в режиме ожидания начала игры
      - $3 10 - кнопка вверх
      - $3 11 - кнопка вправо
      - $3 12 - кнопка вниз
      - $3 13 - кнопка влево
      - $3 14 - центральная кнопка джойстика (ОК)

    4 - общая яркость
     - $4 0 D; установить текущий уровень общей яркости  
            D - яркость в диапазоне 1..255

    5 - рисование
      - $5 0 RRGGBB; - установить активный цвет рисования в формате RRGGBB
      - $5 1;        - очистить матрицу (заливка черным)
      - $5 2;        - заливка матрицы активным активным цветом рисования 
      - $5 3 X Y;    - рисовать точку активным цветом рисования в позицию X Y

    6 - передача текстовых параметров в устройство
     - $6 N|text     где N - назначение текста:
     - $6 0|текст  - текст бегущей строки "$6 0|X|text" - X - 0..9,A..Z - индекс строки, text - сохраняемый текст
     - $6 1|текст  - имя сервера NTP
     - $6 2|текст  - SSID сети подключения
     - $6 3|текст  - пароль для подключения к сети 
     - $6 4|текст  - имя точки доступа
     - $6 5|текст  - пароль к точке доступа
     - $6 6|текст  - Настройки будильника в формате "$6 6|DD EF WD AD HH1 MM1 HH2 MM2 HH3 MM3 HH4 MM4 HH5 MM5 HH6 MM6 HH7 MM7"
                       DD    - установка продолжительности рассвета (рассвет начинается за DD минут до установленного времени будильника)
                       EF    - установка эффекта, который будет использован в качестве рассвета
                       WD    - установка дней пн-вс как битовая маска
                       AD    - продолжительность "звонка" сработавшего будильника
                       HHx   - часы дня недели x (1-пн..7-вс)
                       MMx   - минуты дня недели x (1-пн..7-вс)
     - $6 7|текст  - строка запрашиваемых текущих значений параметров, 
                     например - "$6 7|CE|CC|CO|CK|NC|SC|C1|DC|DD|DI|NP|NT|NZ|NS|DW|OF;" или
                                "$6 7|CE CC CO CK NC SC C1 DC DD DI NP NT NZ NS DW OF;"
     - $6 8|текст  - имя сервера MQTT
     - $6 9|текст  - имя пользователя MQTT
     - $6 10|текст - пароль к MQTT-серверу
     - $6 11|Y colorHEX X|colorHEX X|...|colorHEX X   
                   - Отправка изображения построчно. X в строке изменяется от 0 до WIDTH-1, Y изменяется от 0 до HEIGHT-1   
                     В ответ ожидается строка формата "#L Y-X ackNNN;" где NNN - произвольный, обычно последовательно увеличивающийся номер ответа.   
                     При получении ответа, что строка разобрана, Y увеличивается на 1 и в устройство отправляется следующая строка  
                     Отправка строк изображения повторяется пока Y < HEIGHT  
                     Если приложение не получило ответа - через время таймаута осуществляется повторная отправка строки с последним значением Y  
     - $6 12|X colorHEX Y|colorHEX Y|...|colorHEX Y   
                   - Отправка изображения по колонкам. Y в строке изменяется от 0 до HEIGHT-1, Y изменяется от 0 до WIDTH-1   
                     В ответ ожидается строка формата "#C X-Y ackNNN;" где NNN - произвольный, обычно последовательно увеличивающийся номер ответа.   
                     При получении ответа, что строка разобрана, X увеличивается на 1 и в устройство отправляется следующая колонка   
                     Отправка колонок изображения повторяется пока X < WIDTH  
                     Если приложение не получило ответа - через время таймаута осуществляется повторная отправка колонки с последним значением X  

       *** Примечание: 
           Способ отправки изображения - по строкам или по колонкам определяется по меньшей строне изображения:  
           Если Высота изображения меньше ширины - изображение отправляется по колонкам слева направо, иначе изображение отправляется по строкам сверху вниз.

       *** Примечание:
           Команды этого раздела с кодами операции 0..6 и 8..11 не могут передаваться в пакете с другими командами
           и не должны содержать в конце после текста символа завершения команды ';', если этот символ не является частью передаваемой строки
           Команда этого раздела с кодом операции 7 может использоваться в пакете с другими командами, если в конце присутствует символ завершения команды ';' 
           Для одиночной команды символ завершения ';' не обязателен. Ключи разделяются символом '|' или пробелом.

    8 - управление эффектами
      - $8 0 N; включить эффект N
      - $8 1 N D;   - D -> параметр #1 для эффекта N;
      - $8 2 N X;   - вкл/выкл использовать в демо-режиме; N - номер эффекта, X=0 - не использовать X=1 - использовать 
      - $8 3 N D;   - D -> параметр #2 для эффекта N;
      - $8 4 N X;   - вкл/выкл оверлей текста поверх эффекта; N - номер эффекта, X=0 - выкл X=1 - вкл 
      - $8 5 N X;   - вкл/выкл оверлей часов поверх эффекта; N - номер эффекта, X=0 - выкл X=1 - вкл 
      - $8 6 N D;   - D -> контрастность эффекта N;

    11 - Настройки MQTT-канала (см. также $6 для N=8,9,10)
      - $11 1 X;    - использовать управление через MQTT сервер; X=0 - не использовать; X=1 - использовать
      - $11 2 D;    - порт MQTT
      - $11 3 X;    - Флаг - использует ли MQTT сервер префикс - имя пользователя при формировании топика
      - $11 4 D;    - Задержка между последовательными обращениями к MQTT серверу
      - $11 5;      - Разорвать подключение к MQTT серверу, чтобы он иог переподключиться с новыми параметрами

    12 - Настройки погоды
      - $12 3 X;      - использовать цвет для отображения температуры X=0 - выкл X=1 - вкл в дневных часах
      - $12 4 X;      - использовать получение погоды с погодного сервера 0 - выкл; 1 - Yandex; 2 - OpenWeatherMap
      - $12 5 I С C2; - интервал получения погоды с сервера в минутах (I) и код региона C - Yandex и код региона C2 - OpenWeatherMap
      - $12 6 X;      - использовать цвет для отображения температуры X=0 - выкл X=1 - вкл в ночных часах

    13 - Настройки бегущей cтроки  
      - $13 0 N;    - активация для редактирования строки с номером N - запрос текста строки
      - $13 1 N;    - активация прокручивания строки с номером N
      - $13 2 I;    - запросить текст бегущей строки с индексом I как есть, без обработки макросов
      - $13 3 I;    - запросить текст бегущей строки с индексом I с обработкой макросов для формирования списка выбора в приложении
      - $13 9 I;    - сохранить настройку I - интервал в секундах отображения бегущей строки
      - $13 11 X;   - режим цвета бегущей строки X: 0,1,2,           
      - $13 13 X;   - скорость прокрутки бегущей строки
      - $13 15 00FFAA; - цвет текстовой строки для режима "монохромный", сохраняемый в globalTextColor
      - $13 18 X;   - сохранить настройку X "Бегущая строка в эффектах" (общий, для всех эффектов)

    14 - быстрая установка ручных режимов с пред-настройками
      - $14 0;      - Черный экран (выкл);  
      - $14 1;      - Белый экран (освещение);  
      - $14 2;      - Цветной экран;  
      - $14 3;      - Огонь;  
      - $14 4;      - Конфетти;  
      - $14 5;      - Радуга;  
      - $14 6;      - Матрица;  
      - $14 7;      - Светлячки;  
      - $14 8;      - Часы ночные;
      - $14 9;      - Часы бегущей строкой;
      - $14 10;     - Часы простые;  

    15 - скорость таймеров
      - $15 D N; D - скорость 0..255; 
                  N - таймер, где N
                      0 - таймер эффектов

    16 - Режим смены эффектов: 
      - $16 N; где N:
                   0 - ручной режим;  
                   1 - авторежим; 
                   2 - PrevMode; 
                   3 - NextMode; 
                   5 - вкл/выкл случайный выбор следующего режима

    17 - Время автосмены эффектов и бездействия: 
      - $17 N1 N2;
            N1 - время в секундах демонстрации эффекта, затем переход к следующему эффекту
            N2 - время в секундах до автоматического перехода из ручного режима в автоматический

    18 - Запрос текущих параметров программой: 
      - $18 page; 
         page - страница настройки в программе на смартфоне (1..12) или специальный параметр (91..99)
           1: - Настройки
           2: - Эффекты
           3: - Настройки бегущей строки
           4: - Настройки часов
           5: - Настройки будильника
           6: - Настройки подключения
           7: - Настройки режимов автовключения по времени
          10: - Загрузка картинок
          11: - Рисование
          12: - Игры
          91: - Запрос текста бегущих строк для заполнения списка в программе (как есть, без обработки макроса)
          92: - Запрос текста бегущих строк для заполнения списка в программе (Добавлен индекс, обработка макроса)
          93: - Запрос списка звуков будильника
          94: - Запрос списка звуков рассвета
          95: - Ответ состояния будильника - сообщение по инициативе сервера
          96: - Ответ демо-режима звука - сообщение по инициативе сервера
          99: - Запрос списка эффектов

    19 - работа с настройками часов
      - $19 1 X;    - сохранить настройку X "Часы в эффектах"
      - $19 2 X;    - Использовать синхронизацию часов NTP  X=0 - нет, X=1 - да
      - $19 3 N Z;  - Период синхронизации часов NTP и Часовой пояс
      - $19 4 X;    - Выключать индикатор TM1637 при выключении экрана X=0 - нет, X=1 - да
      - $19 5 X;    - Режим цвета часов оверлея X: 0,1,2,3
      - $19 6 X;    - Ориентация часов  X: 0 - горизонтально, 1 - вертикально
      - $19 7 X;    - Размер часов X: 0 - авто, 1 - малые 3х5, 2 - большие 5x7
      - $19 8 YYYY MM DD HH MM; - Установить текущее время YYYY.MM.DD HH:MM
      - $19 9 X;    - Показывать температуру вместе с малыми часами X=1 - да; X=0 - нет
      - $19 10 X;   - Цвет ночных часов:  0 - R; 1 - G; 2 - B; 3 - C; 3 - M; 5 - Y; 6 - W;
      - $19 11 X;   - Яркость ночных часов:  1..255;
      - $19 12 X;   - скорость прокрутки часов оверлея или 0, если часы остановлены по центру
      - $19 14 00FFAA; - цвет часов оверлея, сохраняемый в globalClockColor
      - $19 16 X;   - Показывать дату в режиме часов  X=0 - нет, X=1 - да
      - $19 17 D I; - Продолжительность отображения даты / часов (в секундах)

    20 - настройки и управление будильников
      - $20 0;       - отключение будильника (сброс состояния isAlarming)
      - $20 2 X VV MA MB;
           X    - исп звук будильника X=0 - нет, X=1 - да 
          VV    - максимальная громкость
          MA    - номер файла звука будильника
          MB    - номер файла звука рассвета
      - $20 3 X NN VV; - пример звука будильника
           X - 1 играть 0 - остановить
          NN - номер файла звука будильника из папки SD:/01
          VV - уровень громкости
      - $20 4 X NN VV; - пример звука рассвета
           X - 1 играть 0 - остановить
          NN - номер файла звука рассвета из папки SD:/02
          VV - уровень громкости
      - $20 5 VV; - установить уровень громкости проигрывания примеров (когда звук уже воспроизводится плеером)
          VV - уровень громкости

    21 - настройки подключения к сети / точке доступа
      - $21 0 X; - использовать точку доступа: X=0 - не использовать X=1 - использовать
      - $21 1 IP1 IP2 IP3 IP4; - установить статический IP адрес подключения к локальной WiFi сети, пример: $21 1 192 168 0 106;
      - $21 2; Выполнить переподключение к сети WiFi

    22 - настройки включения режимов матрицы в указанное время
       - $22 HH1 MM1 NN1 HH2 MM2 NN2 HH3 MM3 NN3 HH4 MM4 NN4;
             HHn - час срабатывания
             MMn - минуты срабатывания
             NNn - эффект: -3 - выключено; -2 - выключить матрицу; -1 - ночные часы; 0 - случайный режим и далее по кругу; 1 и далее - список режимов EFFECT_LIST 

    23 - прочие настройки
       - $23 0 VAL;  - лимит по потребляемому току

## Получение текущих настроек

Для получения текущих значений параметров устройства следует отправить ему команду
```
    $6 7|<список>
``` 
где <список> - строка запрашиваемых текущих значений параметров   
Пример:
 
    "$6 7|CE|CC|CO|CK|NC|SC|C1|DC|DD|DI|NP|NT|NZ|NS|DW|OF;"
 
или  

    "$6 7|CE CC CO CK NC SC C1 DC DD DI NP NT NZ NS DW OF;"  

Команда может использоваться в пакете с другими командами, если в конце присутствует символ завершения команды `';'`  
Для одиночной команды символ завершения `';'` не обязателен. Ключи разделяются символом `'|'` или `' '`(пробел).  
Список ключей и формат возвращаемых значений приведены в таблице  


  | X   | **Ключ** | **Ответ**       | **Значение** 
  | --- | -------- | --------------- | -------- 
  | X | **W**    | **W:число**       | ширина матрицы
  | X | **H**    | **H:число**       | высота матрицы
  | X | **AA**   | **AA:[текст]**    | пароль точки доступа
  | X | **AD**   | **AD:число**      | продолжительность рассвета, мин
  | X | **AE**   | **AE:число**      | эффект, использующийся для будильника
  | X | **AO**   | **AO:X**          | включен будильник 0-нет, 1-да
  | X | **AL**   |**AL:X**           | сработал будильник 0-нет, 1-да
  | X | **AM1H** | **AM1H:HH**       | час включения режима 1     00..23
  | X | **AM1M** |**AM1M:MM**        | минуты включения режима 1  00..59
  | X | **AM1E** |**AM1E:NN**        | номер эффекта режима 1:   -3 - не используется; -2 - выключить матрицу; -1 - ночные часы; 0 - включить случайный с автосменой; 1 - номер режима из списка EFFECT_LIST
  | X | **AM2H** |**AM2H:HH**        | час включения режима 2     00..23
  | X | **AM2M** |**AM2M:MM**        | минуты включения режима 2  00..59
  | X | **AM2E** |**AM2E:NN**        | номер эффекта режима 1:   -3 - не используется; -2 - выключить матрицу; -1 - ночные часы;  0 - включить случайный с автосменой; 1 - номер режима из списка EFFECT_LIST
  | X | **AM3H** |**AM3H:HH**        | час включения режима 1     00..23
  | X | **AM3M** |**AM3M:MM**        | минуты включения режима 1  00..59
  | X | **AM3E** |**AM3E:NN**        | номер эффекта режима 1:   -3 - не используется; -2 - выключить матрицу; -1 - ночные часы; 0 - включить случайный с автосменой; 1 - номер режима из списка EFFECT_LIST
  | X | **AM4H** |**AM4H:HH**        | час включения режима 2     00..23
  | X | **AM4M** |**AM4M:MM**        | минуты включения режима 2  00..59
  | X | **AM4E** |**AM4E:NN**        | номер эффекта режима 1:   -3 - не используется; -2 - выключить матрицу; -1 - ночные часы;  0 - включить случайный с автосменой; 1 - номер режима из списка EFFECT_LIST
  | X | **AN**   |**AN:[текст]**     | имя точки доступа
  | X | **AT**   |**AT: DW HH MM**   | часы-минуты времени будильника для дня недели DW 1..7 -> например "AT:1 09 15"
  | X | **AU**   |**AU:X**           | создавать точку доступа 0-нет, 1-да
  | X | **AW**   |**AW:число**       | битовая маска дней недели будильника b6..b0: b0 - пн .. b7 - вс
  | X | **BE**   |**BE:число**       | контрастность эффекта
  | X | **BR**   |**BR:число**       | яркость
  | X | **C1**   |**C1:цвет**        | цвет режима "монохром" часов оверлея; цвет: 192,96,96 - R,G,B
  | X | **C2**   |**C2:цвет**        | цвет режима "монохром" бегущей строки; цвет: 192,96,96 - R,G,B
  | X | **CС**   |**CС:X**           | режим цвета часов оверлея: 0,1,2
  | X | **CE**   |**CE:X**           | оверлей часов вкл/выкл, где Х = 0 - выкл; 1 - вкл (использовать часы в эффектах)
  | X | **CK**   |**CK:X**           | размер горизонтальных часов, где Х = 0 - авто; 1 - малые 3x5; 2 - большие 5x7 
  | X | **CL**   |**CL:цвет**        | цвет рисования в формате RRGGBB
  | X | **CO**   |**CO:X**           | ориентация часов: 0 - горизонтально, 1 - вертикально
  | X | **CT**   |**CT:X**           | режим цвета текстовой строки: 0,1,2
  | X | **DC**   |**DC:X**           | показывать дату вместе с часами 0-нет, 1-да
  | X | **DD**   |**DD:число**       | время показа даты при отображении часов (в секундах)
  | X | **DI**   |**DI:число**       | интервал показа даты при отображении часов (в секундах)
  | X | **DM**   |**DM:Х**           | демо режим, где Х = 0 - ручное управление; 1 - авторежим
  | X | **DW**   |**DW:X**           | показывать температуру вместе с малыми часами 0-нет, 1-да
  | X | **EF**   |**EF:число**       | текущий эффект
  | X | **IP**   |**IP:xx.xx.xx.xx** | Текущий IP адрес WiFi соединения в сети
  | X | **IT**   |**IT:число**       | время бездействия в секундах
  | X | **LE**   |**LE:[список]**    | список эффектов, разделенный запятыми, ограничители [] обязательны
  | X | **MA**   |**MA:число**       | номер файла звука будильника из SD:/01
  | X | **MB**   |**MB:число**       | номер файла звука рассвета из SD:/02
  | X | **MD**   |**MD:число**       | сколько минут звучит будильник, если его не отключили
  | X | **MP**   |**MP:папка.файл**  | номер папки и файла звука который проигрывается
  | X | **MU**   |**MU:X**           | использовать звук в будильнике 0-нет, 1-да
  | X | **MV**   |**MV:число**       | максимальная громкость будильника
  | X | **MX**   |**MX:X**           | MP3 плеер доступен для использования 0-нет, 1-да
  | X | **NA**   |**NA:[текст]**     | пароль подключения к сети
  | X | **NB**   |**NB:Х**           | яркость цвета ночных часов, где Х = 1..255
  | X | **NC**   |**NС:Х**           | цвет ночных часов, где Х = 0 - R; 1 - G; 2 - B; 3 - C; 4 - M; 5 - Y;
  | X | **NP**   |**NP:Х**           | использовать NTP, где Х = 0 - выкл; 1 - вкл
  | X | **NS**   |**NS:[текст]**     | сервер NTP, ограничители [] обязательны
  | X | **NT**   |**NT:число**       | период синхронизации NTP в минутах
  | X | **NW**   |**NW:[текст]**     | SSID сети подключения
  | X | **NZ**   |**NZ:число**       | часовой пояс -12..+12
  | X | **OF**   |**OF:X**           | выключать часы вместе с лампой 0-нет, 1-да
  | X | **OM**   |**OM:X**           | сколько ячеек осталось свободно для хранения строк
  | X | **PD**   |**PD:число**       | продолжительность режима в секундах
  | X | **QA**   |**QA:X**           | использовать MQTT 0-нет, 1-да
  | X | **QD**   |**QD:число**       | задержка отправки сообщения MQTT
  | X | **QP**   |**QP:число**       | порт подключения к MQTT серверу
  | X | **QR**   |**QR:X**           | топик использует префикс в виде имени пользователя для формирования топика
  | X | **QS**   |**QS:[text]**      | имя MQTT сервера, например QS:[srv2.clusterfly.ru]
  | X | **QU**   |**QU:[text]**      | имя пользователя MQTT соединения, например QU:[user_af7cd12a]
  | X | **QW**   |**QW:[text]**      | пароль MQTT соединения, например QW:[pass_eb250bf5]
  | X | **QZ**   |**QZ:X**           | сборка поддерживает MQTT 0-нет, 1-да
  | X | **PW**   |**PW:число**       | ограничение по току в миллиамперах
  | X | **RM**   |**RM:Х**           | смена режимов в случайном порядке, где Х = 0 - выкл; 1 - вкл
  | X | **S1**   |**S1:[список]**    | список звуков будильника, разделенный запятыми, ограничители [] обязательны
  | X | **S2**   |**S2:[список]**    | список звуков рассвета, разделенный запятыми, ограничители [] обязательны
  | X | **SC**   |**SC:число**       | скорость смещения часов оверлея
  | X | **SD**   |**SD:X**           | наличие и доступность SD карты: Х = 0 - нат SD карты; 1 - SD карта доступна
  | X | **SE**   |**SE:число**       | скорость эффектов
  | X | **SS**   |**SS:число**       | параметр #1 эффекта
  | X | **SQ**   |**SQ:спец**        | параметр #2 эффекта; спец - "L>val>itrm1,item2,..itemN" - список, где val - текущее, далее список; "C>x>title" - чекбокс, где x=0 - выкл, x=1 - вкл; title - текст чекбокса
  | X | **ST**   |**ST:число**       | скорость смещения бегущей строки
  | X | **TA**   |**TA:X**           | активная кнопка, для которой отправляется текст - 0..9,A..Z
  | X | **TE**   |**TE:X**           | оверлей текста бегущей строки вкл/выкл, где Х = 0 - выкл; 1 - вкл (использовать бегущую строку в эффектах)
  | X | **TI**   |**TI:число**       | интервал отображения текста бегущей строки
  | X | **TS**   |**TS:строка**      | строка состояния кнопок выбора текста из массива строк: 36 символов 0..5, где<br>- 0 - серый - пустая<br>- 1 - черный - отключена<br>- 2 - зеленый - активна - просто текст, без макросов<br>- 3 - голубой - активна, содержит макросы кроме даты <br>- 4 - синий - активная, содержит макрос даты<br>- 5 - красный - для строки 0 - это управляющая строка  
  |   | **TX**   |**TX:[текст]**     | текст для активной строки. Ограничители [] обязательны
  |   | **TY**   |**TY:[Z:текст]**   | обработанный текст для активной строки, после преобразования макросов, если они есть. Ограничители [] обязательны Z - индекс строки в списке 0..35; текст ответа предваряется префиксом: 'Z > текст'; 
  |   | **TZ**   |**TZ:[Z:текст]**   | То же, что 'TY". Служит для фоновой загрузки всего массива сохраненных строк в смартфон для формирования элементов списка выбора строки. Получив этот ответ приложение на смартфоне берет следующий индекс и отправляет команду `'$13 3 I;'` для получения следующей строки.
  | X | **UC**   |**UC:X**           | использовать часы поверх эффекта 0-нет, 1-да
  | X | **UE**   |**UE:X**           | использовать эффект в демо-режиме 0-нет, 1-да
  | X | **UT**   |**UT:X**           | использовать бегущую строку поверх эффекта 0-нет, 1-да
  | X | **W1**   |**W1**             | текущая погода ('ясно','пасмурно','дождь'и т.д.)
  | X | **W2**   |**W2**             | текущая температура
  | X | **WC**   |**WC:X**           | Использовать цвет для отображения температуры в дневных часах  X: 0 - выключено; 1 - включено
  | X | **WN**   |**WN:X**           | Использовать цвет для отображения температуры в ночных часах  X: 0 - выключено; 1 - включено
  | X | **WR**   |**WR:число**       | Регион погоды Yandex
  | X | **WS**   |**WS:число**       | Регион погоды OpenWeatherMap
  | X | **WT**   |**WT:число**       | Период запроса сведений о погоде в минутах
  | X | **WU**   |**WU:X**           | Использовать получение погоды с сервера: 0 - выключено; 1 - Yandex; 2 - OpenWeatherMap
  | X | **WZ**   |**WZ:X**           | Прошивка поддерживает погоду USE_WEATHER == 1 - 0 - выключено; 1 - включено

```
  *** Примечание: Ключи, не отмеченные 'X' не могут использоваться в команде получения значения "$6 7|<ключи>", т.к. они используют внутренние индексы, 
                  предварительно устанавливаемые другими командами.

                  Ключи TX,TY,TZ не предназначены для получения значений командой "$6 7|TX TY TZ";
                  Получить значения этих ключей можно в ответ на команды:
                  "$13 2 N;"  - получить текст строки с индексом N (N -> 0..35) как есть, без обработки макросов
                  "$13 3 N;"  - получить текст строки с индексом N (N -> 0..35) c обработкой макросов для формирования списка выбора в приложении
                  "$18 3;"    - получить все настройки, связанные с бегущей строкой
```

## Сообщения в MQTT канал, отправляемые по инициативе устройства

MQTT протокол обеспечивает двунаправленную передачу сообщений - как со стороны управляющей программы (браузерной панели управления) в устройство,
так и из устройства в управляющую программу по инициативе устройства.  

Наличие двустороннего канала связи позволяет устройству инициативно уведомлять управляющего о событиях, произошедших в устройстве.

### Уведомления

Уведомления отправляются в следующие топики MQTT канала:
 - **`nfo`** для информационных сообщений
 - **`err`** для сообщений об ошибках
 - **`ack`** для уведомлений, что команда принята и обработана, ответных данных не будет

### Информационные уведомления - тип NFO

- Нотификация о старте / перезапуске прошивки  
  `NFO:{"act":"START"}`  

- Нотификация о смене режима - какой эффект включен  
  `NFO:{"act":"MODE","id":24,"name":"'Мерцание'"}`  
  - **id**   - код режима (эффекта) который включен
  - **name** - название включенного эффекта
	   
- Нотификация о успешной / неуспешной синхронизации времени  
  `NFO:{"act":"TIME","server_name":"ru.pool.ntp.org","server_ip":"192.36.143.130","result":"REQUEST"}`  
  `NFO:{"act":"TIME","server_name":"ru.pool.ntp.org","server_ip":"192.36.143.130","result":"TIMEOUT"}`  
  `NFO:{"act":"TIME","server_name":"ru.pool.ntp.org","server_ip":"192.36.143.130","result":"OK","time":1601729577}`  
	   
- Нотификация о получении погоды с сервера погоды  
  `NFO:{"act":"WEATHER","region":66,"result":"TIMEOUT"}` - таймат ожидания ответа сервера  
  `NFO:{"act":"WEATHER","region":66,"result":"ERROR","status":"<http_code>"}` - неожиданный ответ сервера в `<http_code>` (ожидается `"OK 200"`)  
  `NFO:{"act":"WEATHER","region":66,"result":"ERROR","status":"unexpected answer"}` - получен ответ от сервера, который не содержит HTTP заголовков  
  `NFO:{"act":"WEATHER","region":66,"result":"ERROR","status":"json error"}` - полученный ответ - не информация о погоде в формате json  
  `NFO:{"act":"WEATHER","region":66,"result":"ERROR","status":"no data"}` - получен json-ответ, но в нем нет информации о погоде. Возможно в напросе указан неизвестный код региона погоды  
  `NFO:{"act":"WEATHER","region":66,"result":"OK","status":"<условия>","temp":23,"night":false,"icon":"skc","sky":"#57bbfe","town":"Omsk"}` - получен ответ, содержащий сведения о погоде  

  При удачном получении информации о погоде, в ответе содержатся следующие поля:
  - **region** - код региона для которого выполен запрос информации
  - **status** - текущие погодные условия - "ясно", "пасмурно", "дождь" и так далее
  - **temp**   - текущая температура
  - **night**  - true - темное время суток, false - светлое время суток
  - **icon**   - иконка погоды, соответствует текущим погодным условиям
  - **sky**    - цвет фона неба
  - **town**   - название населенного пункта, согласно переданному коду региона погоды
<br><br>

  | Код иконки         | Значения
  | ------------------ | --------
  | **bkn-minus-ra-d** | облачно с прояснениями, небольшой дождь (день)
  | **bkn-minus-sn-d** | облачно с прояснениями, небольшой снег (день)
  | **bkn-minus-sn-n** | облачно с прояснениями, небольшой снег (ночь)
  | **bkn-d**          | переменная облачность (день)
  | **bkn-n**          | переменная облачность (ночь)
  | **bkn-ra-d**       | переменная облачность, дождь (день)
  | **bkn-ra-n**       | переменная облачность, дождь (ночь)
  | **bkn-sn-d**       | переменная облачность, снег (день)
  | **bkn-sn-n**       | переменная облачность, снег (ночь)
  | **bl**             | метель
  | **fg-d**           | туман
  | **ovc**            | пасмурно
  | **ovc-minus-ra**   | пасмурно, временами дождь
  | **ovc-minus-sn**   | пасмурно, временами снег
  | **ovc-ra**         | пасмурно, дождь
  | **ovc-sn**         | пасмурно, снег
  | **ovc-ts-ra**      | пасмурно, дождь, гроза
  | **skc-d**          | ясно (день)
  | **skc-n**          | ясно (ночь)
   
	   
- Нотификация о срабатывании / отключении работающего будильника  
  `NFO:{"act":"ALARM","state":"on","type":"dawn"} `  - будильник - состояние "рассвет, без звука"  
  `NFO:{"act":"ALARM","state":"on","type":"alarm"}`  - будильник - состояние "звук"  
  `NFO:{"act":"ALARM","state":"off","type":"auto"}`  - будильник - состояние "выкл. автоматически после истечения времени"  
  `NFO:{"act":"ALARM","state":"off","type":"stop"}`  - будильник - состояние "выкл. кнопкой или командой из приложения"  
	   
- Нотификация о срабатывании авторежима 1..4  
  `NFO: {"act":"AUTO","mode":X,"text":"<text>"}`  
  - **mode** - код режима (эффекта) который включен
  - **text** - информационное сообщение в виде текста о наступившем событии

- Нотификация о начале и окончании показа текста бегущей строки  
  `NFO: {"act":"TEXT","run":true,"text":"<text>"}`  
  `NFO: {"act":"TEXT","run":false}`  
  - **run**  - true - начало отображения бегущей строки; false - окончание отображения текста бегущей строки
  - **text** - отображаемый текст

- Нотификация о загрузке и воспроизведении файла эффекта с SD-карты  
  `NFO: {"act":"SDCARD","result":"OK","file":"<file>"}`  
  `NFO: {"act":"SDCARD","result":"ERROR"}`  
  - **result** - true - эффект загружен и воспроизводится; false - ошибка загрузки файла эффекта
  - **file**   - имя файла <file>, загруженного с SD-карты

### Сообщения об ошибках - тип ERR 

Сообщение отправляется устройством в ответ на нераспознанную команду или некорректные параметры команды

  `ERR: {"message":"unknown page","text":"нет страницы с номером X"}` - ответ на команду `$18 X;`, где X- неизвестная страница  
  `ERR: {"message":"unknown command","text":"неизвестная команда '<команда>'"}` - ответ на несуществующую или нераспознанную команду `'<команда>'`  

### Сообщения о выполнении полученной команды - тип ACK

  `ACK` - cообщение состоит из единственной сигнатуры `ACK` и не содержит каких либо дополнительных параметров  
<br><br><br><br><br>
