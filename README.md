# Práctica Sistemas Empotrados
## Descripción de la práctica

Se busca diseñar e implementar un controlador para una máquina expendedora que esté
basado en Arduino UNO y en los sensores/actuadores que se proporcionan en el kit Arduino.
La práctica tendrá que integrar obligatoriamente los siguientes componentes hardware:

● Arduino UNO

● LCD

● Joystick

● Sensor temperatura/Humedad DHT11

● Sensor Ultrasonido

● Boton

● 2 LEDS Normales (LED1, LED2)


En la primera parte del programa tenemos incluidas las librerías que vamos a utilizar además de definir los pines que vamos a implementar para cada sensor, LED o botón. También definimos alguna que otra variable global que tenga relación con cada elemento para poder ubicarlas más rápido.

Para el LCD:
```
#include <LiquidCrystal.h> // Entre los símbolos <> buscará en la carpeta de librerías configurada
#define COLS 16 // Columnas del LCD
#define ROWS 2 // Filas del LCD
```
El sensor de temperatura y humedad:
```
#include <DHT.h>
// Definimos el pin digital donde se conecta el sensor
#define DHTPIN 8
// Dependiendo del tipo de sensor
#define DHTTYPE DHT11
// Inicializamos el sensor DHT11
DHT dht(DHTPIN, DHTTYPE);

```
LCD pines:
```
// Inicialización del LCD
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
int ledPin = A4;
const int columns = 16;
const int rows = 2;
```

Pines para el sensor de ultrasonidos:
```
//ultrasonidos------------
const int EchoPin = 9;
const int TriggerPin = 6;
int distance;
//------------------------
```
Para el joystick:
```
//Joystick
const int pinJoyX = A0;
const int pinJoyY = A1;
const int pinJoyButton = 13;
//led
const int pinLed = 10;
//---------------
```
```
//button
const int buttonPin = 7;
unsigned long tiempoInicio = 0;
//---------------
```

La función 'setup()' nos quedaría de la siguiente manera para configutar todos estos pines además de iniciar el sensor de humedad y temperatura, entre otras cosas.
```
void setup() {

  // Configuración monitor serie
  Serial.begin(9600);

  // Configuramos las filas y las columnas del LCD en este caso 16 columnas y 2 filas
  lcd.begin(COLS, ROWS);


  //Temp y humedad
  dht.begin();

  //Ultrasonidos
  pinMode(TriggerPin, OUTPUT);
  pinMode(EchoPin, INPUT);

  //Joystick
  pinMode(pinJoyButton , INPUT_PULLUP);  //activar resistencia pull up

  //Button
  pinMode(buttonPin, INPUT);
}
```
