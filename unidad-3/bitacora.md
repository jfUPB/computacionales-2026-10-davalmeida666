# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
### Diagnóstico del problema (análisis):
#### Error 1: Copia superficial del puntero (shallow copy → double-free)
<img width="991" height="398" alt="image" src="https://github.com/user-attachments/assets/abf80c5c-3194-4055-87a2-c423404a7972" />
<img width="470" height="161" alt="image" src="https://github.com/user-attachments/assets/c611fe37-e334-4bc0-b92c-93c3dd31aca6" />
<img width="1108" height="637" alt="image" src="https://github.com/user-attachments/assets/b1c88b00-0016-4d22-8c41-b3837b670180" />

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

### Error 2: Fuga de memoria (memory leak)

<img width="1108" height="222" alt="image" src="https://github.com/user-attachments/assets/99da5da1-550c-4e7c-9d4d-43d18273f032" />

#### ¿Cuál es el error?

La clase no define destructor. Cada vez que el constructor llama ```new int[3]```, reserva 12 bytes en el heap. Esa memoria nunca se libera porque nadie llama ```delete[]```.

#### ¿Por qué ocurre? (mecanismo en memoria)

El heap es una región de memoria que el programa administra manualmente. Cuando llamas new, el sistema operativo entrega un bloque y lo marca como "en uso". Ese bloque permanece reservado hasta que el programa llame explícitamente ```delete[]``` sobre él. C++ no tiene recolector de basura: si la única variable que conocía esa dirección (el puntero en el stack) desaparece al salir del scope, el bloque queda huérfano, ocupando espacio sin que nadie pueda reclamarlo. El OS solo lo recuperará cuando el proceso entero termine.
Al salir de ```simularEncuentro()```:
```
  ┌──────────────────────────────────────┐
  │  stack frame se destruye             │
  │  heroe.estadisticas desaparece  ─x─  │  ← el puntero muere
  │  copiaHeroe.estadisticas        ─x─  │  ← el puntero muere
  └──────────────────────────────────────┘

  HEAP: [ 100 ][ 20 ][ 15 ]  @0x5556d2c0   ← sigue ocupado, huérfano
  ```

#### ¿Cuál es su consecuencia?

En un programa pequeño o de vida corta, el OS recupera toda la memoria al terminar el proceso y el efecto es invisible. Pero en un juego que crea y destruye miles de personajes durante horas, cada ```new int[3]``` sin ```delete[]``` acumula 12 bytes perdidos. Eso escala a megabytes de heap inutilizable, causando lentitud progresiva, fallos de asignación o crash por agotamiento de memoria.





## Bitácora de reflexión
