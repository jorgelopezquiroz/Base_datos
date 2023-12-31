# Practica ultrasonico y DHT22 con node-red y XAMPP
En este repositorio muestra como podemos programar una ESP32 con sensores de distancia (HC-SR04 Ultrasonic Distance Sensor), de humedad y temperatura (DHT22)y conexión a internet mediente Node-red y XAMPP como nuestra base de datos.

## Introducción

### Descripción
La ```Esp32``` la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos un sensor (```HC-SR04 Ultrasonic Distance Sensor```) para adquirir la distancia y el sensor ```DHT22``` para adquirir temperatura y Humedad del medio ambiente; Cabe aclarar que esta practica se usara un simulador llamado [WOKWI](https://https://wokwi.com/)
 , Node-red para la conexción a Internet y XAMPP para guardar nuestros datos obtenidos.

## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/) como simulador.
- Tarjeta ```ESP 32```.
- Sensor ```HC-SR04 Ultrasonic Distance Sensor```.
- Sensor ```DHT22```
- Node-red
- XAMPP


## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/) y tener instalado en tu computadora Node-red y comprobar que funcione de manera correcta.


### Instrucciones de preparación de entorno 

1. Entramos al simulador [WOKWI](https://https://wokwi.com/) y selecionamos la tarjeta ```ESP 32``` como se muestra en la siguiente imagen.

![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/ESP32.png?raw=true)


2. Posteriormente se abrirá la terminal de programación y se coloca la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="Jorge-/lopez/";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;
const int Trigger = 13;   //Pin digital 2 para el Trigger del sensor
const int Echo = 12;   //Pin digital 3 para el Echo del sensor

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
  long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc ["DISTANCIA"]=d;
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("Jorge-/lopez/", output.c_str());
  }
}
```
3. Se necesita instalar las librerias de **DHT sensor library for ESPx**, **ArduinoJson** y **PubSubClient** en el simulador como se muestra en la siguiente imagen.
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/Librerias.png?raw=true)
 
4. Hacer la conexion de **HC-SR04 Ultrasonic Distance Sensor**, **DHT22** y con la **ESP32** como se muestra en la siguente imagen.

![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/Diagrama.png?raw=true)
## Diagrama de flujo en Node-red
5. Empezamos con nuestro mmqtt, elegimos un ***mmqtt in*** como se muestra en la imagen. 
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/mmqtt.jpeg?raw=true)

6. Elegimos nuestro servidor como se muestra en la imagen. 
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/server.jpeg?raw=true)

7. Realizamos nuestro diagrama de flujo como se muestra en la siguiente imagen.
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/Diagram.jpeg?raw=true)

8. Es de gran importancia que nuestro **Parser** sea ***json*** y este configurado como se muestra en la siguiente imagen.
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/Parser.jpeg?raw=true)

9. En nuestro **Dashboard** Agregamos un tab en nuestro caso fue **Practica10** pero puede ser el nombre que gusten y dos groupos de nombre **Graficas** y **Datos** como se muestra en la siguiente imagen.
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/dashboard.jpeg?raw=true)

10. Para nuestro mysql que es el enlace entre Node-red y XAMPP usaremos el que esta en la parte de storage como en la siguiente imagen.
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/image.png?raw=true)

11. Si no aparece en ese apartado, sera nesesario descargarlo, primero abre el apartado en la parte superior derecha y elige la opcion de manage palette.

![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/manage%20palette.png?raw=true)

12. Instala **node-red-node-mysql 1.0.3** como se muestra en la imagen.
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/node-mysql.png?raw=true)

13. En la funcion 1 agrega el siguiente codigo.
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/codigo%20funcion%201.png?raw=true)
**codigo:**
```
var query = "INSERT INTO `diplomai` (`ID`,`FECHA`, `DEVICE`, `TEMPERATURA`, `HUMEDAD`, `DISTANCIA`) VALUES (NULL, current_timestamp(), '";
query = query + msg.payload.DEVICE + "','";
query = query + msg.payload.TEMPERATURA + "','";
query = query + msg.payload.HUMEDAD + "','";
query = query + msg.payload.DISTANCIA + "');'";
msg.topic = query;
return msg;
```

14. Configura el mysql de la siguiente manera como se muestra en la imagen.
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/config.png?raw=true)

## Base de datos XAMPP
 1.  Tenemos que tener instalada XAMPP anteriormente, nesitamos abrila y selecionar Admin en el apartado de MySQL como se muestra en la siguiente imagen.
 ![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/unnamed.png?raw=true)

 2. Posteormente creamos una base de datos nueva como en la siguiente imagen.
 ![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/new%20base.png?raw=true)

 3.- Cuando este lista nuestra base de datos podemos comenzar con nuestra similación.
 ![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/base%20de%20datos.png?raw=true)



### Instrucciónes de operación

1. Iniciar la simulación.
2. Visualizar los datos en el monitor serial.
3. Colocar la temperatura y humedad dando *doble click* al sensor **HC-SR04 Ultrasonic Distance Sensor** para simular la distancia y al sensor **DHT22** para simular humedad y temperatura.
4. Ir a **Node-red** y abrir nuestro dashboard y visualizar nuestros resutaldos de la simulacion
5. Ir a nuestro local Host y comprobar que se esten registrando nuestros datos de manera correta en la base de datos.

## Resultados

Cuando haya compilado, verás los valores dentro del  en el  monitor serial y en nuestro **Dashboard** como muestra en las siguente imagenes.

## Monitor serial
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/Monitor%20serial.jpeg?raw=true)
## Dashboard
![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/DASHB.jpeg?raw=true)


## Evidencias
**Wokwi**

![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/WOKWI.jpeg?raw=true)



**Node-red Dashboard**


![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/Dashboard:1.jpeg?raw=true)


**XAMPP**

![](https://github.com/jorgelopezquiroz/Base_datos/blob/main/XAMPP.jpeg?raw=true)






# Créditos

Desarrollado por Jorge Esteban Lopez Quiroz

- [GitHub](https://github.com/jorgelopezquiroz)