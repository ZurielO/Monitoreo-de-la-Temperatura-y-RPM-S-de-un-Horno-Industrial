# Monitoreo de Temperatura y RPM'S de un Horno Industrial
EQUIPO 2


## Objetivo
El objetivo del programa es controlar un conjunto de dispositivos (sensores, actuadores y motores) basados en el ESP32. A través de la conexión WiFi y MQTT, el sistema monitorea la temperatura mediante un sensor DHT22, controla motores dependiendo de rangos de temperatura específicos, y gestiona LEDs como indicadores visuales. Además, publica los datos del sensor en un servidor MQTT para ser utilizados por otros dispositivos o aplicaciones.

## CODIGO
```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

#include <Stepper.h>

int RPM;
int Tempfalse;

const int stepsPerRevolution = 5000;

Stepper stepper1(stepsPerRevolution, 2, 4, 16,18);
Stepper stepper2(stepsPerRevolution, 22, 23, 19, 21);
Stepper stepper3(stepsPerRevolution, 17, 5, 13, 25);
Stepper stepper4(stepsPerRevolution, 33, 32, 34, 35);

const int led1 = 14;
const int led2 = 12;
const int led3 = 26;
const int led4 = 27;

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "35.172.255.228";
String username_mqtt="AntonioM";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

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

pinMode(led1, OUTPUT);
pinMode(led2, OUTPUT);
pinMode(led3, OUTPUT);
pinMode(led4, OUTPUT);

stepper1.setSpeed(1);
stepper2.setSpeed(2);
stepper3.setSpeed(3);
stepper4.setSpeed(5);

}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();

RPM=(data.temperature*50);
Tempfalse=(data.temperature*6);

if (data.temperature>=14 && data.temperature<=20)
{
  digitalWrite(led1, HIGH);
  digitalWrite(led2, LOW);
  digitalWrite(led3, LOW);
  digitalWrite(led4, LOW);

stepper1.step(stepsPerRevolution);


  delay (2000);
  
}
else if (data.temperature>=34 && data.temperature<=40)
{
  digitalWrite(led1, HIGH);
  digitalWrite(led2, HIGH);
  digitalWrite(led3, LOW);
  digitalWrite(led4, LOW);

stepper2.step(stepsPerRevolution);


  delay (2000);
  
}
else if(data.temperature>=54 && data.temperature<=60) 
{
  digitalWrite(led1, HIGH);
  digitalWrite(led2, HIGH);
  digitalWrite(led3, HIGH);
  digitalWrite(led4, LOW);
 
stepper3.step(stepsPerRevolution);


  delay (2000);
  
  }
  else if(data.temperature>=70 && data.temperature<=80) 
{
  digitalWrite(led1, HIGH);
  digitalWrite(led2, HIGH);
  digitalWrite(led3, HIGH);
  digitalWrite(led4, HIGH);

stepper4.step(stepsPerRevolution);

  delay (2000);
 
  }
else
{
 digitalWrite(led1,  LOW);
  digitalWrite(led2, LOW);
  digitalWrite(led3, LOW);
  digitalWrite(led4, LOW);
}

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
    doc["TEMPERATURA"] = String(Tempfalse) + "°C";
    doc["RPM"] = String(RPM);
    doc["NOMBRE"]="ANTONIO DE JESUS MH";
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("PRACTICA1", output.c_str());
  }
}
```


## Descripción del Código
### 1.	Inicialización de Bibliotecas: El programa utiliza varias bibliotecas clave:
-	ArduinoJson: Para la creación de mensajes JSON que se publican en el servidor MQTT.
-	WiFi.h: Para manejar la conexión a redes Wi-Fi.
-	PubSubClient.h: Para la implementación del protocolo MQTT (Message Queuing Telemetry Transport).
-	DHTesp.h: Para interactuar con el sensor de temperatura y humedad DHT22.
-	Stepper.h: Para controlar los motores paso a paso.
### 2.	Conexión Wi-Fi y MQTT:
-	La función setup_wifi() permite que el dispositivo se conecte a una red Wi-Fi usando las credenciales definidas.
-	El cliente MQTT se conecta al servidor especificado (35.172.255.228) utilizando las credenciales de usuario y contraseña.
-	El servidor MQTT es utilizado para publicar y suscribirse a temas (topics) como "outTopic" (para mensajes de salida) y "inTopic" (para mensajes entrantes).
### 3.	Lectura de Sensores y Control de Actuadores:
-	El sensor DHT22 se configura para medir la temperatura y la humedad en el pin 15 del ESP32.
-	Dependiendo de la temperatura medida, se activan diferentes motores paso a paso y se controlan LEDs específicos como indicadores:
-	Temperaturas entre 14°C y 20°C: Se activa el motor 1 (controlado por stepper1).
-	Temperaturas entre 34°C y 40°C: Se activa el motor 2 (controlado por stepper2).
-	Temperaturas entre 54°C y 60°C: Se activa el motor 3 (controlado por stepper3).
-	Temperaturas entre 70°C y 80°C: Se activa el motor 4 (controlado por stepper4).
-	Los LEDs se encienden de manera proporcional al rango de temperatura detectado, con hasta cuatro LEDs posibles dependiendo de la condición.
### 4.	Publicación de Datos:
-	Cada dos segundos, el programa recopila los datos de temperatura, calcula un valor de "RPM" basado en la temperatura (multiplicando la temperatura por 50), y genera un mensaje JSON con la información del dispositivo, la temperatura y los valores calculados.
-	El mensaje JSON es luego publicado en el servidor MQTT bajo el topic "PRACTICA1", lo que permite que otros dispositivos o servicios recojan esta información.
### 5.	Estrategias de Reconexión:
-	Si la conexión MQTT se pierde, la función reconnect() maneja los intentos de reconexión automáticamente, intentando volver a conectarse al servidor MQTT cada 5 segundos hasta lograrlo.
Consideraciones Técnicas:
-	Seguridad de la Red y MQTT: El programa utiliza credenciales básicas para la conexión WiFi y MQTT (usuario y contraseña), lo cual es adecuado para entornos controlados, pero no es recomendable en aplicaciones críticas debido a la falta de cifrado o mecanismos de autenticación más robustos. Se sugiere utilizar TLS/SSL para asegurar la comunicación MQTT.
-	Eficiencia en el Manejo de Sensores: La lectura del sensor DHT22 se realiza cada segundo (con un delay de 1000 ms en el loop), lo cual es apropiado para muchas aplicaciones, pero en entornos donde se necesite mayor precisión temporal o eficiencia, este tiempo podría optimizarse o configurarse de manera más dinámica.
-	Manejo de Motores Paso a Paso: Los motores paso a paso se controlan de manera secuencial según las condiciones de temperatura. Sin embargo, no se contempla un control de velocidad más avanzado o dinámico que permita ajustar la operación de los motores en función de otros parámetros o condiciones externas.


## Enlace entre Wokwi y RedNote
El enlace entre Wokwi y RedNote se establece a través del protocolo MQTT. El programa en el ESP32, corriendo en Wokwi, genera datos y los publica a través de MQTT a un servidor MQTT que RedNote está escuchando. Los datos en formato JSON (como el que se muestra) son enviados periódicamente y RedNote puede recibirlos, procesarlos y mostrarlos en su interfaz de usuario (por ejemplo, en la parte de debug).

## Generación de Datos en Wokwi

En el código, el ESP32 (simulado en Wokwi) lee datos del sensor de temperatura y calcula la cantidad de RPM en función de la temperatura. Estos datos son empaquetados en un mensaje JSON con los campos mencionados (DEVICE, Anho, TEMPERATURA, RPM, NOMBRE).
Publicación del Mensaje a MQTT:

Una vez que los datos están formateados en el mensaje JSON, se publican en un topic MQTT (por ejemplo, "PRACTICA1") utilizando el cliente MQTT configurado en el ESP32. Esto es lo que permite que RedNote reciba esos datos.

## Recepción y Visualización en RedNote

RedNote, al estar suscrito a ese topic MQTT, recibe el mensaje JSON publicado desde el ESP32. Dependiendo de cómo RedNote esté configurado, estos datos pueden ser procesados y visualizados en su interfaz, por ejemplo, en el debug o en un dashboard visual. En el debug, puedes ver el contenido de los mensajes que el servidor MQTT recibe, lo cual es útil para depuración y monitoreo en tiempo real.
En Resumen:
El código en Wokwi genera datos de sensores y actuadores en el ESP32, los empaqueta en un mensaje JSON, y los publica mediante MQTT. RedNote, al estar suscrito a los topics correspondientes, recibe estos mensajes y puede visualizarlos en su interfaz (como en la parte de debug). Los campos JSON como "DEVICE", "TEMPERATURA", y "RPM" representan los datos específicos que se están enviando, y estos se utilizan en RedNote para monitorear y analizar el rendimiento del sistema.

Este flujo de trabajo permite una comunicación bidireccional entre Wokwi (ESP32) y RedNote, donde el ESP32 envía datos y RedNote los procesa para su visualización y análisis.

(![Texto alternativo](https://github.com/ZurielO/Monitoreo-de-la-Temperatura-y-RPM-S-de-un-Horno-Industrial/blob/main/Captura%20de%20pantalla%202024-12-15%20165555.png).



## Conclusión
El programa es una solución funcional que integra sensores, actuadores, y comunicación a través de MQTT en un entorno IoT con ESP32. Es adecuado para aplicaciones en las que se requiere monitorear y reaccionar ante cambios de temperatura, controlar dispositivos como motores paso a paso y generar reportes en tiempo real a través de MQTT. No obstante, para una implementación en producción, se recomienda mejorar aspectos de seguridad y optimizar la gestión de recursos del sistema.
