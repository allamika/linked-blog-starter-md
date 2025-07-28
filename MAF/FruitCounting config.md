### Step 1: Rename camera

The soft only recognize camera following the following naming convention:
Fruit_Counting_{number} : ex "Fruit_Counting_0"
It can be rename in the teliviewer soft at the field "DeviceUserID".
![[Pasted image 20250728171532.png]]
### Step2: MQTT configuration

The mqtt serveur configuration use json config file with the following structure:
```json
{
   "MQTT_BROKER" : "adress of the mqtt broker",
   "port" : "(int) port of the mqtt broker",
   "mqtt_topic": "a dictionary that link a camera (the key) to an mqtt topic (the value). All count of on the "key" camera will be transmit an the "value" topic of the mqtt server. For now only one mqtt server per application. "
}
```

Exemple:
```json
{
   "MQTT_BROKER" : "localhost",
   "port" : 1883,
   "mqtt_topic":{
      "Fruit_Counting_0":"RECEPTION/C1",
      "Fruit_Counting_12":"RECEPTION/C2"
   }
}
```