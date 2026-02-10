# Unidad 2

## Bitácora de proceso de aprendizaje
### Actividad 01
Dibujando un punto en la pantalla
```.asm
@SCREEN         
M=1      
@END
(END)
0;JMP
```
<img width="424" height="185" alt="image" src="https://github.com/user-attachments/assets/a0a29273-bfc0-4256-a0d1-bf8a86dfcc42" />

### Actividad 02
Dibujando una línea horizontal

```.asm
@SCREEN         
M=-1      
@END
(END)
0;JMP
```
<img width="455" height="247" alt="image" src="https://github.com/user-attachments/assets/c22736c3-ab1c-4456-8676-6cbc04ec2ecb" />

### Actividad 03
Entrada salida interactiva

```.asm

(start)

@SCREEN
D=A
@i
M=D

@SCREEN
M=-1 

(LOOP)

@KBD
D=M
@100
D=D-A
@derecha
D;JEQ

@KBD
D=M
@105
D=D-A
@izquierda
D;JEQ

@LOOP
0;JMP

(derecha)
@i
A=M
M=0
A=A+1
M=-1
D=A
@i
M=D
@LOOP
0;JMP

(izquierda)
@i
A=M
M=0
A=A-1
M=-1
D=A
@i
M=D
@LOOP
0;JMP



@END
(END)
0;JMP

```

### Actividad 04
Convierte un ciclo while en un ciclo for
Código en C++
```.c++
//Adds 1+...+100.
int sum=0;
for(int i = 1; i <=100; i++){
   sum+= i;
}

```

Código en assembler
```.asm

@0
D=A
@sum
M=D


@1
D=A
@i
M=D

@100
D=A
@limit
M=D

(LOOP)
@i
D=M
@limit
D=D-M
@END
D;JGT

    
@i
D=M
@sum
M=D+M

    
@i
M=M+1

    
@LOOP
0;JMP

(END)
    
@END
0;JMP
```
Imagen:

<img width="1037" height="255" alt="image" src="https://github.com/user-attachments/assets/49a0b298-33ff-4898-967f-c05ee50e0b5a" />





## Bitácora de aplicación 



## Bitácora de reflexión
