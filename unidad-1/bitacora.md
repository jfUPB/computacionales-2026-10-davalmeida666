# Unidad 1

## Bitácora de proceso de aprendizaje
### Actividad 1
#### Experimento 1
<img width="797" height="314" alt="image" src="https://github.com/user-attachments/assets/cac27d92-8533-4a6f-8e61-6038989c35de" />

#### Experimento 2
<img width="793" height="300" alt="image" src="https://github.com/user-attachments/assets/6654a068-cc02-47cc-ae5a-f33ba8cf8c06" />

#### Observaciones en los experimentos:

El experimento 1 permite guardar sumar los valores de D y A, y guardar este resultado en un lugar dentro de la memoria RAM. Después de esto, el programa finaliza.
El Experimento 2 es similar, simplemente cambiando la posición de guardado.

#### ¿Qué diferencia hay entre los datos almacenados en la memoria ROM y en la RAM?

Los datos almacenados en la memoria ROM, son, por decirlo así, más permanentes, pues, son aquellos que dan instrucciones al programa, por lo tanto, son esenciales para el funcionamiento de los computadores.

Por otro lado, los datos almacenados en Memoria RAM son datos temporales, variables, que siguiendo instrucciones de la Memoria ROM, se almacenan en una cierta posición, la cual puede cambiar, si el programa lo solicita.

### Actividad 2

``` program.asm
@SCREEN
D=A
@i
M=D
(READKEYBOARD)
@KBD
D=M
@KEYPRESSED
D;JNE
@i
D=M
@SCREEN
D=D-A
@READKEYBOARD
D;JLE
@i
M=M-1
A=M
M=0
@READKEYBOARD
0;JMP

(KEYPRESSED)
@i
D=M
@KBD
D=D-A
@READKEYBOARD
D;JGE
@16
A=M
M=-1
@i
M=M+1
@READKEYBOARD
0;JMP
```
#### Experimento:
<img width="1321" height="618" alt="image" src="https://github.com/user-attachments/assets/28990ea0-8bf1-4dc6-9fa4-9e26c043e52e" />

##### Identifica una instrucción que use la ALU y explica qué hace.
M=M+1; toma el valor almacenado en la memoria, y le suma uno, esto permite que el programa siga pintando cada pixel de la pantalla.
##### ¿Para qué sirve el registro PC?
Para llevar la cuenta del número de instrucción que el programa va a ejecutar
##### ¿Cuál es la diferencia entre @i y @READKEYBOARD?
@i, o sea una variable declarada, toma el primer espacio libre que encuentre en la memoria, mientras que @READKEYBOARD va a leer el último espacio en la memoria y le va a asignar el valor de la tecla presionada.
##### Describe qué se necesita para leer el teclado y mostrar información en la pantalla.
1ero se necesita guardar el numero de tecla en la memoria, luego crear un programa que compare ese valor con 0, y dependiendo del resultado, ejecutar otra instrucción. Ahora, para que todo funcione, se debe habilitar el teclado en el simulador.
##### Identifica un bucle en el programa y explica su funcionamiento.
Desde @READKEYBOARD al final, se coloca la instrucción, 0;JMP. Esto permite que al "finalizar" el programa, el mismo automaticamente vuelva a la función @READKEYBOARD, volviendo al inicio de leer la tecla y pintar la pantalla si es el caso.
##### Identifica una condición en el programa y explica su funcionamiento.
@READKEYBOARD
D;JLE

Esta instrucción dice, que el programa puede saltar a la función de leer el teclado, si el valor de D (que está asignado al número de carácter) es menor al último espacio de memoria, o sea, si una tecla está siendo pulsada, puede avanzar, sino no.





## Bitácora de aplicación 
### Actividad 04
#### Implementando un ciclo simple
Crea un programa que use un ciclo para sumar los números del 1 al 5 y guarde el resultado en la dirección de memoria 12.


```.asm
@12
M=0

@i
M=1

(LOOP)
@i
D=M
@5
D=D-A
@END
D;JGT

@i
D=M
@12
M=D+M

@i
M=M+1

@LOOP
0;JMP

(END)
@END
0;JMP




```


<img width="967" height="808" alt="image" src="https://github.com/user-attachments/assets/c700872f-9c32-417c-946a-c0d37fc116ba" />




## Bitácora de reflexión
### ACTIVIDAD 05

#### *Sin consultar tus apuntes, el simulador o cualquier otro material, responde con tus propias palabras a las siguientes preguntas. ¡No te preocupes por la perfección! El objetivo es ver qué recuerdas ahora mismo.*

#### 1. Describe con tus palabras las tres fases del ciclo Fetch-Decode-Execute. ¿Qué rol juega el Program Counter (PC) en este ciclo?

R// El ciclo Fetch-Decode-Execute es como el proceso que sigue la computadora para entender y hacer lo que le pides. Primero, busca la instrucción en la memoria (Fetch), luego la entiende (Decode) y finalmente la ejecuta (Execute). El Program Counter (PC) es como un marcador que le dice a la computadora dónde está la siguiente instrucción para no perderse y seguir el orden correcto. Así, la CPU trabaja paso a paso para que todo funcione bien.

#### 2. ¿Cuál es la diferencia fundamental entre una instrucción-A (que empieza con @) y una instrucción-C (que involucra D, M, A, etc.) en el lenguaje ensamblador de Hack? Da un ejemplo de cada una.

R//  - La instrucción A, que empieza con @, le dice a la computadora a qué lugar ir o qué número usar, como @10 que apunta al número 10.
     - La instrucción-C es la que hace las operaciones, como copiar o sumar valores, por ejemplo D=M que copia un dato.
     Básicamente, la instrucción-A señala y la C hace el trabajo con esos datos.

#### 3. Explica la función de los siguientes componentes del computador Hack: el registro D, el registro A y la ALU.

R// - El registro D guarda datos temporales para hacer cálculos o guardar resultados.
    - El registro A apunta a una dirección de memoria o guarda un número para usar en operaciones.
    - La ALU es la calculadora  (por decirle de alguana manera) que hace sumas, restas y otras operaciones con los datos que recibe de A y D. Así, juntos permiten que la computadora procese información paso a paso.

#### 4. ¿Cómo se implementa un salto condicional en Hack? Describe un ejemplo (p. ej., saltar si el valor de D es mayor que cero).

R// En Hack, para hacer un salto condicional se usa una instrucción que dice “salta si…” y una etiqueta con la dirección. Por ejemplo, @ETIQUETA y D;JGT significa que si el valor en D es mayor que cero, la computadora salta a donde dice ETIQUETA. Si no, sigue con la siguiente instrucción. Así se controla el flujo según condiciones.

#### 5. ¿Cómo se implementa un loop en el computador Hack? Describe un ejemplo (p. ej., un loop que decremente un valor hasta que llegue a cero).

R//  En Hack, un loop se hace con una etiqueta y un salto que repite mientras una condición sea verdadera. Por ejemplo:
```
 
  @VALOR      // Es la etiqueta "VALOR"                    
D=M         // Guardar el valor de "VALOR" en D
(LOOP)      // Etiqueta que marca el inicio del loop
D=D-1       // Decrementar D en 1
@VALUE
M=D         // Guardar el nuevo valor en VALUE
@LOOP
D;JGT
```

En español, por asi decirlo, lo que hace este ejemplo es: se resta 1 a un valor y si sigue siendo mayor que cero, salta al inicio del loop para repetir. Así, el programa sigue restando hasta que el valor llegue a cero y se detiene.

#### 6. ¿Cuál es la diferencia entre la instrucción D=M y la instrucción M=D?

R// La diferencia entre D=M y M=D, es:
   - D=M significa que se lee de memoria y se guarda en D.
   - M=D significa que el valor se escribe en memoria desde D. En palabras mas tecnicas, significa que el valor que está en el registro D se copia a la memoria en la dirección apuntada.

#### 7. Describe brevemente qué se necesita para leer un valor del teclado (KBD) y para “pintar” un pixel en la pantalla (SCREEN).

  - Para leer del teclado en Hack, solo hay que leer el valor que está guardado en una dirección especial que indica qué tecla se presionó.
  - Para pintar un pixel en la pantalla, se escribe un valor en la memoria que controla los puntos de la pantalla. Así, leer y pintar se hacen leyendo o escribiendo en direcciones específicas.











