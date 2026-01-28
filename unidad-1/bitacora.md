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




## Bitácora de aplicación 

```asm
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






