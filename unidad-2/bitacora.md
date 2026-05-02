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

### Actividad 05
Punteros
Programa 1 en C++
```.c++

int a = 10;
int* p;
p = &a;
*p = 20;
```
Programa 1 en asm
```.asm

@10
D=A
@a
M=D


@a
D=A
@p
M=D


@20
D=A
@p
A=M   
M=D   


(END)
@END
0;JMP

```

<img width="1127" height="246" alt="image" src="https://github.com/user-attachments/assets/48f071d7-34fb-44d2-9f49-92f5bc40102f" />


Programa 2 en C++

``` .c++

int a = 10;
int b = 5;
int *p;
p = &a;
b = *p;
```
Programa 2 en asm

``` .asm


@10
D=A
@a
M=D


@5
D=A
@b
M=D


@a
D=A
@p
M=D


@p
A=M
D=M
@b
M=D



(END)
@END
0;JMP


```
<img width="1140" height="248" alt="image" src="https://github.com/user-attachments/assets/601abf3f-0951-4973-9c71-8f54f0af10d4" />




## Bitácora de aplicación 
 1
 ```.asm

@10
D=A
@pa
M=D


@20
D=A
@pb
M=D


@pa
D=A
@R0
M=D


@pb
D=A
@R1
M=D


@RET_MAIN
D=A
@R15
M=D

@SWAP
0;JMP

(RET_MAIN)


@0
D=A
@R0
M=D

(END)
@END
0;JMP




(SWAP)


@R0
A=M        
D=M        
@R13
M=D        


@R1
A=M        
D=M        
@R0
A=M        
M=D        


@R13
D=M       
@R1
A=M       
M=D        


@R15
A=M
0;JMP
 ```

<img width="1487" height="736" alt="image" src="https://github.com/user-attachments/assets/201492eb-2491-47d2-8841-83bffeeb6553" />
<img width="1220" height="814" alt="image" src="https://github.com/user-attachments/assets/69d1dbcf-edc1-46f7-8cd0-26a4b5a81a60" />
<img width="1197" height="577" alt="image" src="https://github.com/user-attachments/assets/a50511d2-b69d-46bc-9607-a068c9961cd5" />




2
 ```.asm
(start)


// sumResult = 0 
@sumResult 
M=0

// Crear arreglo arr[]

// arr[0] = 10 
@10 
D=A 
@arr 
M=D

// arr[1] = 15 
@15 
D=A 
@arr 
A=A+1 
M=D

// arr[2] = 2 
@2 
D=A 
@arr 
A=A+1 
A=A+1 
M=D

// arr[3] = 3 
@3 
D=A 
@arr 
A=A+1 
A=A+1 
A=A+1 
M=D

// arr[4] = 50 
@50 
D=A 
@arr 
A=A+1 
A=A+1 
A=A+1 
A=A+1
M=D

// Pasar argumentos

@arr 
D=A 
@R0 
M=D // R0 = base

@5 
D=A 
@R1 
M=D // R1 = size

// Guardar retorno 
@returnFromCalSum 
D=A 
@R15 
M=D

@calSum 
0;JMP

(returnFromCalSum)

// Guardar resultado 
@R0 
D=M 
@sumResult 
M=D

@fin 
(fin) 
0;JMP

// FUNCIÓN calSum

(calSum)

// sum = 0 
@0 
D=A 
@R13 
M=D

// i = 0 
@0 
D=A 
@R14 
M=D

(loop)

// if i >= size salir 
@R14 
D=M 
@R1 
D=D-M 
@endLoop 
D;JGE

// D = base 
@R0 
D=M

// D = base + i 
@R14 
D=D+M

// A = base + i 
A=D

// D = *(parr+i) 
D=M

// sum += D 
@R13 
M=D+M

// i++ 
@R14 
M=M+1

@loop 0;JMP

(endLoop)

// return sum 
@R13 
D=M 
@R0 
M=D

@R15 
A=M 
0;JMP
 ```
<img width="1446" height="813" alt="image" src="https://github.com/user-attachments/assets/64cbf9e7-7b10-4bec-a45e-e714202e7240" />

<img width="1032" height="602" alt="image" src="https://github.com/user-attachments/assets/75ef7cf5-b028-4856-8f24-d3f7b88c19fb" />

<img width="974" height="482" alt="image" src="https://github.com/user-attachments/assets/7ee451fd-fdd5-42c1-bf84-8539d3167255" />



## Bitácora de reflexión




