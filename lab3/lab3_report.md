# Лабораторная работа 3. Работа с Modbus протоколом. 

## Задача: Обратиться к данным, начиная от номера регистра: №Студента*100
##### IP Адрес 109.167.241.225
#####  Порт Modbus TCP – 502
##### Адрес Modbus устройства – 9988
##### Адрес Modbus устройства – 9988

* ID транзакции (2 байта) – всегда 0
* ID протокола (2 байта) - всегда 0
* Адрес сервера (0..247 )  (1 байт)  - 1
* Функциональный код (читаем Analog Output Holding Registers) (1 байт) - 03

* 1 регистр в modbus это 16 бит
Данные, которые должны прочитаться:
* 2 числа unsigned integer U16
* Строка ASCII 8 bit – 10 символов.
>Число высчитывается исходя времени, поэтому должно быть зафиксировано время.
Признак правильного выполнения для студента: корректные ASCII символы.


#### Запускаем скрипт [lab3Network.ipynb](lab3Network.ipynb)
* В коде производим запрос по указанному адрессу и порту на нужный регистр, в ответ получаем массив из 7-ми int значений, из условия мы знаем что ответ представляет собой  два числа unsigned integer 16bit*2 и десять символов 8bit\*10, итого **112bit**;
* В ответе мы получаем массив из 7 чисел, первые два числа представленны сразу в нужном для нас виде;
* оставшиеся числа в массиве переводим в двоичный вид
* полученные пять 16битных значений разделяем впополам
* получаем 10 8-ми битных значенияй
* переводим 8-ми битные значения в символы
```python
!pip install pyModbusTCP
!pip install pymodbus
from pyModbusTCP.client import ModbusClient
from datetime import datetime

def decodeToAscii(arr):
  decodeArr = []
  message = ""
  
  for i in arr:
    decodeArr.append((bin(i)).replace("b","")) 

  print(f"BinResponse:{decodeArr}")

  for i in decodeArr[2:]:
    message += chr(int(i[0:8],2)) + chr(int(i[8:16],2))

  return message

c = ModbusClient(host="109.167.241.225", port=502, unit_id=1, auto_open=True)

regs = c.read_holding_registers(300, 7)
if regs:
  print(f"Time: {datetime.now()}")
  print(f"Response: {regs}")
  print(f"Message: {decodeToAscii(regs)}")
  print(f"Number1: {regs[0]}")
  print(f"Number2: {regs[1]}")
else:
  print("ERROR READ")
```

```log
Time: 2023-02-18 00:50:26.304646
Response: [30250, 35886, 17220, 17734, 18248, 18762, 19276]
BinResponse:['0111011000101010', '01000110000101110', '0100001101000100', '0100010101000110', '0100011101001000', '0100100101001010', '0100101101001100']
Message: CDEFGHIJKL
Number1: 30250
Number2: 35886
```
 