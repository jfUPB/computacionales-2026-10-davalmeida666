# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
### Diagnóstico del problema (análisis):
#### Error 1: Copia superficial del puntero (shallow copy → double-free)
<img width="991" height="398" alt="image" src="https://github.com/user-attachments/assets/abf80c5c-3194-4055-87a2-c423404a7972" />
<img width="470" height="161" alt="image" src="https://github.com/user-attachments/assets/c611fe37-e334-4bc0-b92c-93c3dd31aca6" />

#### ¿Cuál es el error?

La línea Personaje ```copiaHeroe = heroe``` ; usa el copy constructor que el compilador genera automáticamente. Ese constructor copia cada campo del objeto byte a byte. Para nombre (un std::string) esto funciona bien porque string ya sabe copiarse. Para estadisticas (un int*), la copia es simplemente el valor numérico del puntero, es decir, la dirección de memoria, no los datos que hay en esa dirección.

#### ¿Por qué ocurre? (mecanismo en memoria)

Cuando se construye heroe, el constructor llama ```new int[3] ```y reserva un bloque de 12 bytes en el heap. La dirección de ese bloque (digamos 0x5556d2c0) se guarda en el campo estadisticas de heroe, que vive en el stack. Cuando se hace la copia, el compilador copia ese valor 0x5556d2c0 al campo estadisticas de copiaHeroe. Ahora hay dos objetos en el stack que guardan exactamente la misma dirección:
```
STACK                         HEAP
─────────────────────         ──────────────────────
heroe.estadisticas            
  = 0x5556d2c0  ──────────▶  [ 100 ][ 20 ][ 15 ]
                               ▲
copiaHeroe.estadisticas        │  ← mismo bloque
  = 0x5556d2c0  ──────────────┘
  ```
Al salir de ```simularEncuentro()```, C++ destruye los objetos en orden inverso: primero copiaHeroe, luego heroe. Si hay destructor con delete[], el primer delete[] libera el bloque correctamente. El segundo delete[] intenta liberar una dirección que ya no pertenece al programa, lo que es comportamiento indefinido (UB). En la práctica el runtime lanza double free detected y aborta el proceso.

#### ¿Cuál es su consecuencia?

En el código original no hay destructor, así que el double-free queda latente pero el programa "parece" funcionar. Si alguien añade un destructor (que sería lo correcto), el crash aparece inmediatamente. Además, aunque no haya crash, los dos objetos comparten datos: modificar ```opiaHeroe.estadisticas[0]``` cambia también los datos de heroe, lo que es un bug silencioso muy difícil de rastrear. Los datos en la memoria son guardados de la siguiente manera:

<img width="625" height="325" alt="image" src="https://github.com/user-attachments/assets/de491353-0649-41e4-a6ce-16c71973deea" />





## Bitácora de reflexión
