# Práctica Sistemas Empotrados
## Descripción de la práctica

Se busca diseñar e implementar un controlador para una máquina expendedora que esté
basado en Arduino UNO y en los sensores/actuadores que se enumeran a continuación:

● LCD

● Joystick

● Sensor temperatura/Humedad DHT11

● Sensor Ultrasonido

● Botón

● 2 LEDS Normales (LED1, LED2)


En la primera parte del programa tenemos incluidas las librerías que vamos a utilizar además de definir los pines que vamos a implementar para cada sensor, LED o botón. También definimos alguna que otra variable global que tenga relación con cada elemento para poder ubicarlas más rápido.

Para el LCD:
```
#include <LiquidCrystal.h> 
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
LiquidCrystal lcd(12, 11, 5, 4, 3, 7);
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
Para el botón:
```
//button
const int buttonPin = 7;
unsigned long tiempoInicio = 0;
//---------------
```

En la función 'setup()', ya que solo va a ser ejecutada una vez, la utilizaremos, no solo para inicializar los sensores sino que también implementaremos la funcionalidad de arranque. Vemos como se inicializa una interrupción y un thread que más adelante explicaremos.


Dentro de esta función encontramos:

```
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
  attachInterrupt(digitalPinToInterrupt(buttonPin), buttonInterruption, CHANGE);

  //Thread
  myThread.enabled = true;
  myThread.setInterval(300);
  myThread.onRun(thread_showTempHum);
  controller.add(&myThread);
```
Funcionalidad de arranque:
```
  //---------------------------------
   //arrancar el sistema
  // Limpiamos la pantalla
  lcd.clear();
 
  // Situamos el cursor en la columna 0 fila 0
  lcd.setCursor(0,0);
 
  lcd.print("CARGANDO...");
  for (int i = 0; i < 3; ++i) {
    digitalWrite(ledPin, HIGH);
    delay(1000);
    digitalWrite(ledPin, LOW);
    delay(1000);
  }
  lcd.clear();

  //--------------------------------
```
Después de esto, vamos a implementar el resto de funcionalidades de servicio y modo administrador dentro de la función 'loop()'.

En el código, definimos algunas funciones importantes ya sea para medir con los sensores o para implementar alguna funcionalidad a la hora de navegar por los menús, como por ejemplo podrían ser:

La función 'ping' que se utiliza para medir la distancia con el sensor de ultrasonidos y nos la devuelve.
```
int ping(int TriggerPin, int EchoPin) {
  long duration, distanceCm;
  
  digitalWrite(TriggerPin, LOW);  //para generar un pulso limpio ponemos a LOW 4us
  delayMicroseconds(4);
  digitalWrite(TriggerPin, HIGH);  //generamos Trigger (disparo) de 10us
  delayMicroseconds(10);
  digitalWrite(TriggerPin, LOW);
  
  duration = pulseIn(EchoPin, HIGH);  //medimos el tiempo entre pulsos, en microsegundos
  
  distanceCm = duration * 10 / 292/ 2;   //convertimos a distancia, en cm
  return distanceCm;
}
```
## ArduinoThreads
En el código hemos implementado ArduinoThreads, por ejemplo para la siguiente función, cuyo funcionamiento se basa en la medida de la temperatura y humedad, además de imprimirlas por el display del LCD:

```
void thread_showTempHum(){
  float t = 0.00;
  float h = 0.00;
  // Leemos la humedad relativa
  
  h = dht.readHumidity();
  // Leemos la temperatura en grados centígrados (por defecto)
  t = dht.readTemperature();

  // Comprobamos si ha habido algún error en la lectura
  if (isnan(h) || isnan(t)) {
    Serial.println("Error obteniendo los datos del sensor DHT11");
    return;
  }
  
  lcd.setCursor(0,0);
  lcd.clear();
  lcd.print("Temp :");
  lcd.print(t);
  lcd.setCursor(0,1);
  lcd.print("Humedad: ");
  lcd.print(h);
}
```
Pero antes de ello, para poder utilizar el thread debemos de incluir: 
```
#include <Thread.h>
#include <StaticThreadController.h>
#include <ThreadController.h>
```
Para inicializar el thread y el controlador para los threads:
```
ThreadController controller = ThreadController();
Thread myThread = Thread();
```

Para implementar el joystick, lo haremos de la siguiente manera:

Para tomar las medidas: 
```
 Xvalue = analogRead(pinJoyX);
    delay(50);//es necesaria una pequeña pausa entre lecturas analógicas
    Yvalue = analogRead(pinJoyY);
    buttonValue = digitalRead(pinJoyButton);
    char* dir = joystickDir(Xvalue,Yvalue,buttonValue);
```
Esa dirección, nos la devuelve la función 'joystickDir()' ya que analiza los valores recogidos de forma analógica y en función de estos, se determinará la dirección del joystick para después emplearla en el menu.
```

char* joystickDir(int x, int y, int buttonValue){
  if (x>600){
    return "UP";
  }
  else if (x<400){
    return "DOWN";
  }
  else if (y>600){
    return "RIGHT";
  }
  else if (y<400){
    return "LEFT";
  }
  else if (buttonValue == 0){
    return "R3";
  }
  else{
    return "NONE";
  }
}
```
## Interrupciones
Además de threads vamos a utilizar una interrupción hardware para el uso del botón. Es por eso que en la función 'setup()' a la hora de inicializar el botón encontramos:
```
//Button
  pinMode(buttonPin, INPUT);
  attachInterrupt(digitalPinToInterrupt(buttonPin), buttonInterruption, CHANGE);
```
Cuando realizamos este callback, se llama a la función 'buttonInterruption()'que controla el tiempo que mantenemos pulsado el botón:
```
void buttonInterruption() {
  if (digitalRead(buttonPin) == HIGH) {
    tiempoInicio = millis();  // Se presionó el botón, registra el tiempo
    botonPresionado = true;
  } else {
    botonPresionado = false;  // Se soltó el botón, reinicia el temporizador
  }
}
```
## Watchdog
Se ha implementado Watchdog, que podemos incluir con la biblioteca: <avr/wdt.h>.
Para ello, primero debemos de incluir estas dos líneas en la funcion setup():
```
void setup() {

  wdt_disable();
  wdt_enable(WDTO_8S);
}
```
Donde la desactivación del Watchdog Timer mediante wdt_disable() evita que se produzca un reinicio automático. Y después vamos a configurar el Watchdog Timer con un tiempo de espera de 8 segundos. 

Para luego más tarde, a lo largo del código, poder resetear el contador llamando a la función wdt_reset().

## Circuito

Imagen del circuito en [Fritzing](https://fritzing.org/)

![image](https://github.com/srobledo2021/Practica_arduino/assets/113594786/415cd6ae-7215-4af0-82a4-7f017483b280)



Las librerías que estamos usando son:

- ArduinoThread (by Ivan Seidel v2.1.1)
- DHT sensor library (by Adafruit v1.4.6)

A parte de las que ya están incluidas como por ejemplo sería:

- LiquidCrystal.h
- avr/wdt.h
