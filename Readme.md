# Practica B: interrupción por timer



```
volatile int interruptCounter;
int totalInterruptCounter;
 
hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;
 
void IRAM_ATTR onTimer() {
  portENTER_CRITICAL_ISR(&timerMux);
  interruptCounter++;
  portEXIT_CRITICAL_ISR(&timerMux);
 
}
 
void setup() {
 
  Serial.begin(115200);
 
  timer = timerBegin(0, 80, true);
  timerAttachInterrupt(timer, &onTimer, true);
  timerAlarmWrite(timer, 1000000, true);
  timerAlarmEnable(timer);
 
}
 
void loop() {
 
  if (interruptCounter > 0) {
 
    portENTER_CRITICAL(&timerMux);
    interruptCounter--;
    portEXIT_CRITICAL(&timerMux);
 
    totalInterruptCounter++;
 
    Serial.print("An interrupt as occurred. Total number: ");
    Serial.println(totalInterruptCounter);
 
  }
}
```

#Funcionaminento de esta practica

Primero de todo empezamos declarando el contador de interrupciones con volatile. Esto evitara que se elimine a causa de optimizaciones del compilador.

`volatile int interruptCounter;`

Declaramos un contador para ver el numero de interrupciones que ocurriran desde el principio del programa. 

`int totalInterruptCounter;`

Para configurar el timer, necesitaremos un puntero:

`hw_timer_t * timer = NULL;`

La ultima variable a declarar es el `portMUX_TYPE` ,  esta la usaremos para sincronizar la el main loop y la ISR:

`portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;`

una vez declarado este void:

```
void setup() {
Serial.begin(9600);
timer = timerBegin(0, 80, true);
timerAttachInterrupt(timer, &onTimer, true);
timerAlarmWrite(timer, 1000000, true);
timerAlarmEnable(timer);
}
```

Tenemos la inicialización de nuestro timer llamando la función `timerBegin`. Esta función recibe como entrada el numero del temporizador que queremos usar, el valor de prescaler y indicamos si va hacia delante o hacia atras.

`timer = timerBegin(0, 80, true);`

A continuació usaremos la función `timerAttachInterrupt`, esta recibe como entrada el puntero al temporizador, la dirección `onTimer` que sirve para manejar la interrupción y finalmente el valor `true` para especificar que es de tipo edge.

`timerAttachInterrupt(timer, &onTimer, true);`

Seguidamente utilizamos la función `timerAlarmWrite` , esta recibe como entrada tres valores. Para la primera entrada recibe el puntero al tempirazor, en el segundo valor del contador donde tenemos que generar la interrupción y para finalizar un indicador de si el temporizador se tiene que recargar de forma automatica.

`timerAlarmWrite(timer, 1000000, true);`

Para finalizar, llamamos a la función `timerAlarmEnable`, en esta funcion pasamos nuestra variable del temporizador.

`timerAlarmEnable(timer);`

Quedaria de esta forma:

`void loop() {
if (interruptCounter > 0) {
portENTER_CRITICAL(&timerMux);
interruptCounter--;
portEXIT_CRITICAL(&timerMux);
totalInterruptCounter++;
Serial.print("An interrupt as occurred. Total number: ");
Serial.println(totalInterruptCounter);
}
}
`

El funcionamiento de este programa consiste en incrementar el contador con el numero total de interrupciones (` totalInterruptCounter`) i imprimirlo al puerto sene.

El funcionamiento de la función ISR consiste en incrementar el contador de interrupciones que indicara el bucle principal que se ha producido una interrupción. El codigo completo ISR queda:

```
void IRAM_ATTR onTimer() {
portENTER_CRITICAL_ISR(&timerMux);
interruptCounter++;
portEXIT_CRITICAL_ISR(&timerMux);
}
```

#Impresión

Lo que este programa imprimiria por pantalla seria el numero de veces que hay una interrupción. Saldria algo asi:

An intrrupt as occurred. Total number: 1
An intrrupt as occurred. Total number: 2
An intrrupt as occurred. Total number: 3
An intrrupt as occurred. Total number: 4
An intrrupt as occurred. Total number: 5
An intrrupt as occurred. Total number: 6
An intrrupt as occurred. Total number: 7
...

Y de esta forma sucesivamente hasta que se pare manualmente.


