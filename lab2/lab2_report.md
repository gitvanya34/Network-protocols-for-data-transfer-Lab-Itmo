# Лабораторная работа 2. Работа с mqtt протоколом. 

## Задача: Подключиться к брокеру и считать топик, отправить в топик свое сообщение 
##### Брокер:  broker.hivemq.com
#####  Порт: 1883 
##### топик: ITMO/StudentN/ValueD 

> N — номер в таблице 

> D = 1, 2, 3 

> Считать топики: «свои» ITMO/StudentN/Value1 + ITMO/StudentN/Value2 
> 
> И для всех /Value3

 Запускаем два скрипта паралельно 
 #### [Публикатор](lab2NetworkPub.ipynb) - публикует пять сообщений брокеру раз в секунду, а потом отсылает сообщение на конец передачи

```python
import paho.mqtt.client as mqtt
import time
from datetime import datetime

broker_url = "broker.hivemq.com"
broker_port = 1883

client = mqtt.Client()
client.connect(broker_url, broker_port)

for i in range(0,5):
  strPayload = str(datetime.now())+" TestingPayload"+str(i)
  client.publish(topic="ITMO/Student3/Value1", payload=strPayload, qos=1, retain=False)
  print(str(datetime.now())+" Send payload")
  time.sleep(1)

strPayloadDisconnect = str(datetime.now())+" TestingPayloadDisconnect"+str(i)
client.publish(topic="ITMO/Student3/Value1", payload=strPayloadDisconnect, qos=1, retain=False)
print(str(datetime.now())+" Send payload Disconnect")
     
```

```log
2023-02-14 15:28:25.441925 Send payload
2023-02-14 15:28:26.443373 Send payload
2023-02-14 15:28:27.444853 Send payload
2023-02-14 15:28:28.447162 Send payload
2023-02-14 15:28:29.448723 Send payload
2023-02-14 15:28:30.450537 Send payload Disconnect 
```
 #### [Подписчик](lab2NetworkSub.ipynb) - получает сообщение через брокера от публикатора, получает 5 сообщений и сообщение о конце передаче(выходит из режима прослушивания)
```python
from paho.mqtt import client as mqtt_client
import random
import time
from datetime import datetime

broker_url = "broker.hivemq.com"
broker_port = 1883

def on_message(client, userdata, message):
  print("Message Recieved: "+message.payload.decode())
  if str(message.payload.decode()).find("TestingPayloadDisconnect") !=-1:
    client.disconnect()


client = mqtt.Client()
client.on_message = on_message
client.connect(broker_url, broker_port)
print(str(datetime.now()))
client.subscribe("ITMO/Student3/Value1", qos=2)

client.loop_forever()
client.disconnect
```
```log
Connected to MQTT Broker!
2023-02-14 15:28:21.048714
Connected to MQTT Broker!
Connected to MQTT Broker!
Connected to MQTT Broker!
Connected to MQTT Broker!
Message Recieved: 2023-02-14 15:28:25.441806 TestingPayload0
Connected to MQTT Broker!
Message Recieved: 2023-02-14 15:28:26.443123 TestingPayload1
Connected to MQTT Broker!
Message Recieved: 2023-02-14 15:28:27.444589 TestingPayload2
Connected to MQTT Broker!
Message Recieved: 2023-02-14 15:28:28.446864 TestingPayload3
Connected to MQTT Broker!
Message Recieved: 2023-02-14 15:28:29.448441 TestingPayload4
Connected to MQTT Broker!
Message Recieved: 2023-02-14 15:28:30.450182 TestingPayloadDisconnect4
<bound method Client.disconnect of <paho.mqtt.client.Client object at 0x7ff30948deb0>>
```


---
Следующим этапом мы прослушаем топики ITMO/Student(1-9)/Value3 и индивидуальные топики ITMO/Student3/Value(1-3)
#### [Подписчик нескольких топиков](lab2NetworkSubTeach.ipynb) - Запустим скрипт прослушивания нескольких топиков одновременно, публикатором в данный момент является преподавательский генератор 
```python
from paho.mqtt import client as mqtt_client
import paho.mqtt.client as mqtt
import random
import time
from datetime import datetime
     

broker_url = "broker.hivemq.com"
broker_port = 1883
count = 0

def on_message(client, userdata, message):
  global count 
  if "TestingPayloadDisconnect" in str(message.payload.decode()):
    client.disconnect()
    
  if count == 15:
    print("\n\tITMO/Student3/\t\tValue1\tValue2\tValue3")
    count+=1

  if count > 15:
    if "ITMO/Student3/Value1" in str(message.topic):
      print(f"{datetime.now()}\t|`{message.payload.decode()}`\t|\t|")
    
    if "ITMO/Student3/Value2" in str(message.topic):
      print(f"{datetime.now()}\t|\t|`{message.payload.decode()}`\t|")
      
    if "ITMO/Student3/Value3" in str(message.topic):
      print(f"{datetime.now()}\t|\t|\t|`{message.payload.decode()}`")
  elif "Value3" in message.topic:
    print(f"{datetime.now()} {message.topic}    {message.payload.decode()}")
    count+=1
   

client = mqtt.Client()
client.on_message = on_message
client.connect(broker_url, broker_port)

print(str(datetime.now()))
for i in range(1,10):
  client.subscribe("ITMO/Student"+str(i)+"/Value3",1)

client.subscribe([("ITMO/Student3/Value1", 1),("ITMO/Student3/Value2", 1),("ITMO/Student3/Value3",1)])

client.loop_forever()
```
```log
2023-02-15 14:51:11.198335
2023-02-15 14:51:12.262758 ITMO/Student1/Value3    776
2023-02-15 14:51:12.263112 ITMO/Student2/Value3    775
2023-02-15 14:51:12.264393 ITMO/Student3/Value3    774
2023-02-15 14:51:12.264797 ITMO/Student4/Value3    773
2023-02-15 14:51:12.265238 ITMO/Student7/Value3    770
2023-02-15 14:51:12.265617 ITMO/Student9/Value3    768
2023-02-15 14:51:12.266062 ITMO/Student8/Value3    769
2023-02-15 14:51:12.266440 ITMO/Student5/Value3    772
2023-02-15 14:51:12.266827 ITMO/Student6/Value3    771
2023-02-15 14:51:13.265520 ITMO/Student1/Value3    776
2023-02-15 14:51:13.266656 ITMO/Student2/Value3    775
2023-02-15 14:51:13.266844 ITMO/Student3/Value3    774
2023-02-15 14:51:13.266898 ITMO/Student4/Value3    773
2023-02-15 14:51:13.268278 ITMO/Student5/Value3    772
2023-02-15 14:51:13.271192 ITMO/Student6/Value3    771

	ITMO/Student3/		Value1	Value2	Value3
2023-02-15 14:51:14.264693	|	|	|`774`
2023-02-15 14:51:14.265855	|`215`	|	|
2023-02-15 14:51:14.266138	|	|`4`	|
2023-02-15 14:51:15.271067	|	|`4`	|
2023-02-15 14:51:15.271216	|	|	|`774`
2023-02-15 14:51:15.271270	|`215`	|	|
2023-02-15 14:51:16.261072	|`215`	|	|
2023-02-15 14:51:16.261220	|	|	|`774`
2023-02-15 14:51:16.268164	|	|`4`	|
2023-02-15 14:51:17.273449	|	|	|`774`
2023-02-15 14:51:17.273721	|	|`4`	|
2023-02-15 14:51:17.273800	|`215`	|	|
2023-02-15 14:51:18.263958	|	|	|`774`
2023-02-15 14:51:18.264228	|	|`4`	|
2023-02-15 14:51:18.264315	|`215`	|	|
2023-02-15 14:51:19.263352	|	|	|`774`
2023-02-15 14:51:19.264808	|	|`4`	|
2023-02-15 14:51:19.267458	|`215`	|	|
2023-02-15 14:51:20.270083	|`214`	|	|
2023-02-15 14:51:20.270236	|	|`5`	|
2023-02-15 14:51:20.270323	|	|	|`774`
2023-02-15 14:51:21.268509	|	|	|`774`
2023-02-15 14:51:21.268690	|`214`	|	|
2023-02-15 14:51:21.268908	|	|`5`	|
2023-02-15 14:51:22.271268	|	|	|`774`
2023-02-15 14:51:22.272235	|	|`5`	|
2023-02-15 14:51:22.272378	|`214`	|	|
```