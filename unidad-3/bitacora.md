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


### Correción del Problema

#### Código nuevo

```.c++
#include <iostream>
#include <string>
#include <memory>
#include <array>
#include <stdexcept>

class Personaje {
public:

    // Constructor
    Personaje(const std::string& nombre, int vida, int ataque, int defensa)
        : nombre_(nombre),
          estadisticas_(std::make_unique<std::array<int, 3>>(
              std::array<int, 3>{vida, ataque, defensa}))
    {
        if (vida < 0 || ataque < 0 || defensa < 0)
            throw std::invalid_argument("Las estadisticas no pueden ser negativas");

        std::cout << "[Constructor] Nace '" << nombre_
                  << "' | heap @ " << estadisticas_.get() << '\n';
    }

    // Constructor de copia profunda
    // unique_ptr deshabilita la copia implicita, asi que la definimos
    // nosotros: cada copia recibe su PROPIO bloque en el heap
    Personaje(const Personaje& otro)
        : nombre_(otro.nombre_),
          estadisticas_(std::make_unique<std::array<int, 3>>(*otro.estadisticas_))
    {
        std::cout << "[Copia] '" << nombre_
                  << "' | nuevo bloque @ " << estadisticas_.get() << '\n';
    }

    // Operador de asignacion por copia profunda
    Personaje& operator=(const Personaje& otro) {
        if (this == &otro) return *this;  // proteccion contra auto-asignacion
        nombre_       = otro.nombre_;
        estadisticas_ = std::make_unique<std::array<int, 3>>(*otro.estadisticas_);
        std::cout << "[Asignacion] '" << nombre_
                  << "' | nuevo bloque @ " << estadisticas_.get() << '\n';
        return *this;
    }

    // Movimiento: unique_ptr transfiere propiedad sin copiar datos
    Personaje(Personaje&&) noexcept = default;
    Personaje& operator=(Personaje&&) noexcept = default;

    // Destructor: unique_ptr libera el heap automaticamente
    // No necesitamos escribir delete[] en ningun lugar
    ~Personaje() {
        std::cout << "[Destructor] '" << nombre_
                  << "' | libera @ " << estadisticas_.get() << '\n';
    }

    // Getters
    const std::string& nombre()  const { return nombre_; }
    int vida()    const { return (*estadisticas_)[0]; }
    int ataque()  const { return (*estadisticas_)[1]; }
    int defensa() const { return (*estadisticas_)[2]; }
    const int* punteroHeap() const { return estadisticas_->data(); }

    void imprimir() const {
        std::cout << "Personaje: " << nombre_
                  << " | Vida: "   << vida()
                  << " | ATK: "    << ataque()
                  << " | DEF: "    << defensa()
                  << " | heap @ "  << estadisticas_.get() << '\n';
    }

private:
    std::string                         nombre_;
    std::unique_ptr<std::array<int, 3>> estadisticas_;
};


// -----------------------------------------------------------
void simularEncuentro() {
    std::cout << "\n--- Iniciando encuentro ---\n";

    Personaje heroe("Aragorn", 100, 20, 15);

    // Copia profunda: cada objeto tiene su propio bloque
    Personaje copiaHeroe = heroe;
    copiaHeroe = Personaje("Boromir", 90, 18, 12);

    std::cout << "\n[Verificacion de bloques]\n";
    heroe.imprimir();
    copiaHeroe.imprimir();

    bool mismoBloque = (heroe.punteroHeap() == copiaHeroe.punteroHeap());
    std::cout << "Comparten bloque heap: "
              << (mismoBloque ? "SI — ERROR" : "NO — correcto") << '\n';

    std::cout << "\n--- Saliendo del encuentro ---\n";
}

int main() {
    try {
        simularEncuentro();

        std::cout << "\n[Prueba de movimiento]\n";
        Personaje temporal("Gandalf", 200, 50, 40);
        Personaje movido = std::move(temporal);
        movido.imprimir();

        std::cout << "\n[Prueba de validacion]\n";
        Personaje invalido("Sauron", -1, 999, 0);
    }
    catch (const std::exception& e) {
        std::cerr << "[Excepcion capturada]: " << e.what() << '\n';
    }

    std::cout << "\nSimulacion terminada.\n";
    return 0;
}
```

<img width="1185" height="630" alt="image" src="https://github.com/user-attachments/assets/a01423eb-8c1c-43d9-b96d-d426de6f1a56" />


### Explicación y Justificación

#### Cambio 1: ```int* → std::unique_ptr<std::array<int,3>>cpp```
```.c++
// Antes
int* estadisticas;

// Después
std::unique_ptr<std::array<int, 3>> estadisticas_;
```

```unique_ptr ```aplica RAII: libera el heap automáticamente cuando el objeto sale de scope. Esto elimina el leak porque ya no necesitas destructor manual. Además, unique_ptr no se puede copiar implícitamente, lo que hace imposible el aliasing accidental de punteros que causaba el double-free.

#### Cambio 2: Copy constructor y operador de asignación explícitos

```.c++
cppPersonaje(const Personaje& otro)
    : estadisticas_(std::make_unique<std::array<int, 3>>(*otro.estadisticas_))
{}
```

```make_unique``` siempre reserva un bloque nuevo en el heap. Cada copia recibe su propia dirección de memoria — los dos objetos nunca comparten el mismo bloque. Esto elimina directamente el double-free.

#### Cambio 3: Movimiento en ```= default```

```.c++
cppPersonaje(Personaje&&) noexcept = default;
Personaje& operator=(Personaje&&) noexcept = default;
```
Al definir la copia manualmente, el compilador deja de generar el movimiento. Declararlo en = default activa la semántica de movimiento de unique_ptr: transfiere la propiedad del bloque sin copiarlo y deja el original en nullptr. Sin aliasing, sin double-free.

#### Cambio 4: Miembros privados con getters

```. c++
cpp// Antes: public int* estadisticas  → cualquiera podía hacer delete[] desde fuera
// Después: private + getters que devuelven int por valor

```
Con ```estadisticas_ privado```, ningún código externo puede tocar el puntero directamente. El``` unique_ptr``` es el único dueño y la única vía de liberación

## Bitácora de reflexión
