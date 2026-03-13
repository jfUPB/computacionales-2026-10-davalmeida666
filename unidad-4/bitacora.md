# Unidad 4

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

# Actividad 03 — Implementación de una Cola FIFO para Pintura Dinámica

## Introducción

En esta actividad se desarrolló un programa en **C++** utilizando el framework **:contentReference[oaicite:0]{index=0}** con el objetivo de analizar e implementar desde cero una **estructura de datos tipo cola FIFO (First In, First Out)**.

La idea principal del programa es generar un **efecto de pintura dinámica en pantalla**. Cada vez que el usuario mueve o mantiene presionado el mouse, el programa genera un nuevo trazo que se dibuja en la ventana. Sin embargo, para evitar que el programa almacene una cantidad ilimitada de trazos y consuma más memoria de la necesaria, se implementa una **cola con tamaño máximo**.

Esto significa que cuando se alcanza el límite de elementos permitidos, **el trazo más antiguo se elimina automáticamente**, permitiendo que nuevos trazos se agreguen continuamente sin que la memoria crezca indefinidamente. De esta manera se mantiene una cantidad controlada de elementos visibles y se conserva el comportamiento característico de una estructura **FIFO**, donde **el primer elemento en entrar es el primero en salir**.

Una forma sencilla de entender este comportamiento es imaginar una **fila para comprar comida en una cafetería**: la primera persona en llegar es la primera en ser atendida. De la misma forma, en una cola FIFO los datos se procesan en el orden en el que fueron añadidos.

---
##Código Completo
### ofApp.h
```.c++
#pragma once
#include "ofMain.h"

//* Nodo de la cola
struct Node {
	float x;
	float y;
	float radius;
	ofColor color;
	float opacity;

	Node * next;
};

// Cola FIFO
class BrushQueue {
public:
	Node * front;
	Node * rear;
	int size;
	int maxSize;

	BrushQueue(int _maxSize);
	~BrushQueue();

	void enqueue(float x, float y, float radius, ofColor color, float opacity);
	void dequeue();
	void clear();
	bool isEmpty();
};

// Constructor
BrushQueue::BrushQueue(int _maxSize) {
	front = nullptr;
	rear = nullptr;
	size = 0;
	maxSize = _maxSize;
}

// Destructor
BrushQueue::~BrushQueue() {
	clear();
}

// Agregar nodo al final de la cola
void BrushQueue::enqueue(float x, float y, float radius, ofColor color, float opacity) {
	Node * newNode = new Node;
	newNode->x = x;
	newNode->y = y;
	newNode->radius = radius;
	newNode->color = color;
	newNode->opacity = opacity;
	newNode->next = nullptr;

	if (front == nullptr) {
		front = newNode;
		rear = newNode;
	} else {
		rear->next = newNode;
		rear = newNode;
	}

	size++;

	if (size > maxSize) {
		dequeue();
	}
}

// Eliminar el nodo más antiguo
void BrushQueue::dequeue() {
	if (front == nullptr) {
		return;
	}

	Node * temp = front;
	front = front->next;
	delete temp;

	size--;

	if (front == nullptr) {
		rear = nullptr;
	}
}

// Eliminar todos los nodos
void BrushQueue::clear() {
	while (front != nullptr) {
		Node * temp = front;
		front = front->next;
		delete temp;
	}
	rear = nullptr;
	size = 0;
}

// Verificar si la cola está vacía
bool BrushQueue::isEmpty() {
	return front == nullptr;
}

// Clase principal de openFrameworks
class ofApp : public ofBaseApp {
public:
	BrushQueue strokes;
	float backgroundHue;

	ofApp()
		: strokes(50) {
		backgroundHue = 0;
	}

	void setup();
	void update();
	void draw();
	void keyPressed(int key);
};

```
### En ofApp.cpp
```.c++
#include "ofApp.h"

//--------------------------------------------------------------
void ofApp::setup() { ofBackground(0); }

//--------------------------------------------------------------
void ofApp::update() {

backgroundHue = backgroundHue + 0.2;

if (backgroundHue > 255) {
	backgroundHue = 0;
}

if (ofGetMousePressed()) {

	float x = ofGetMouseX();
	float y = ofGetMouseY();

	float radius = ofRandom(5, 25);

	ofColor color;

	color.setHsb(ofRandom(255), 200, 255);

	float opacity = 200;

	strokes.enqueue(x, y, radius, color, opacity);
}
}

//--------------------------------------------------------------
void ofApp::draw() {

ofColor color1;
ofColor color2;

color1.setHsb(backgroundHue, 150, 240);

color2.setHsb(fmod(backgroundHue + 128, 255), 150, 240);

ofBackgroundGradient(color1, color2, OF_GRADIENT_LINEAR);

Node * current;

current = strokes.front;

int index = 0;

while (current != nullptr) {

	float fade;

	fade = ofMap(index, 0, strokes.size, 255, 50);

	ofSetColor(current->color, fade);

	ofDrawCircle(current->x, current->y, current->radius);

	current = current->next;

	index = index + 1;
}
}

//--------------------------------------------------------------
void ofApp::keyPressed(int key) {

if (key == 'c') {
	strokes.clear();
}

if (key == 'a') {

	if (strokes.maxSize == 50) {
		strokes.maxSize = 100;
	} else {
		strokes.maxSize = 50;
	}

}

else if (key == 's') {
	ofSaveFrame();
}
}

```
---

# Estructura de Datos Implementada

Para implementar esta estructura se creó una **cola llamada `BrushQueue`**, la cual se construyó manualmente utilizando **nodos enlazados**. No se utilizaron estructuras de la biblioteca estándar como `std::queue` o `std::list`, ya que el objetivo de la actividad es comprender el funcionamiento interno de la estructura.

Cada elemento almacenado en la cola representa un **trazo de pintura** que será dibujado posteriormente en la pantalla.

---

# Estructura del Nodo

Cada trazo se almacena dentro de una estructura llamada **`Node`**. Este nodo contiene toda la información necesaria para dibujar el trazo en la pantalla.

Los datos almacenados en cada nodo son:

- Posición en el eje **X**
- Posición en el eje **Y**
- **Radio** del trazo
- **Color** del trazo
- **Opacidad**
- Un puntero llamado **`next`** que apunta al siguiente nodo en la cola

Este puntero permite **conectar los nodos entre sí**, formando una **lista enlazada simple**. El uso de punteros es esencial en este tipo de estructuras porque permite que la cantidad de elementos pueda **crecer o disminuir dinámicamente durante la ejecución del programa**.

---

### Imagen 1 — Estructura del nodo en memoria


---

# Implementación de la Cola (BrushQueue)

La estructura principal del programa es la clase **`BrushQueue`**, la cual implementa la cola FIFO.

Esta clase contiene los siguientes elementos principales:

- **`front`** → apunta al primer nodo de la cola (el más antiguo).
- **`rear`** → apunta al último nodo de la cola (el más reciente).
- **`size`** → número actual de nodos almacenados.
- **`maxSize`** → tamaño máximo permitido para la cola.

El puntero **`front`** es el nodo que será eliminado primero cuando se ejecute la operación **dequeue**, mientras que **`rear`** es el nodo donde se agregan los nuevos elementos mediante **enqueue**.

---

# Función enqueue()

La función **enqueue** se encarga de **agregar nuevos nodos al final de la cola**.

Cuando esta función se ejecuta:

1. Se crea un nuevo nodo utilizando **memoria dinámica** con `new`.
2. Se asignan los valores correspondientes al trazo (posición, radio, color y opacidad).
3. Se verifica si la cola está vacía.
4. Si la cola está vacía, el nuevo nodo se convierte tanto en **front como en rear**.
5. Si la cola ya contiene elementos, el nodo actual en **rear** apunta al nuevo nodo y luego se actualiza el puntero **rear**.
6. Finalmente se incrementa la variable **size**.

Después de insertar el nodo, el programa verifica si el tamaño de la cola supera **maxSize**. Si esto ocurre, se ejecuta automáticamente la función **dequeue** para eliminar el nodo más antiguo.

Esto garantiza que la cola **nunca supere el límite máximo definido**.

---

### Imagen 2 — Inserción del primer nodo en la cola

*(Captura del depurador mostrando `front`, `rear` y `size` después del primer enqueue)*

---

# Función dequeue()

La función **dequeue** se encarga de **eliminar el nodo más antiguo de la cola**, es decir, el nodo al que apunta **front**.

El procedimiento es el siguiente:

1. Se verifica si la cola está vacía.
2. Se crea un puntero temporal que apunta al nodo que se va a eliminar.
3. El puntero **front** se actualiza para que apunte al siguiente nodo.
4. Se libera la memoria del nodo antiguo utilizando **`delete`**.
5. Se decrementa el valor de **size**.

Si después de eliminar el nodo la cola queda vacía, entonces el puntero **rear** también se establece en `nullptr`.

Este procedimiento es muy importante porque garantiza que **la memoria se libere correctamente**, evitando **fugas de memoria**.

---

### Imagen 3 — Eliminación del nodo más antiguo (dequeue)

*(Captura del depurador mostrando el cambio del puntero `front` y la eliminación del nodo)*

---

# Función clear()

La función **clear** elimina **todos los nodos de la cola**.

Para lograr esto se utiliza un **ciclo while** que recorre la cola desde `front` hasta que no quedan más nodos.

En cada iteración:

1. Se guarda temporalmente el nodo actual.
2. Se avanza el puntero `front` al siguiente nodo.
3. Se elimina el nodo anterior utilizando `delete`.

Cuando el proceso termina:

- `front` se establece en `nullptr`
- `rear` se establece en `nullptr`
- `size` vuelve a **0**

Esto garantiza que **toda la memoria utilizada por la estructura sea liberada correctamente**.

---

### Imagen 4 — Limpieza completa de la cola (clear)

*(Captura del depurador mostrando la cola completamente vacía)*

---

# Comportamiento del Programa

El comportamiento interactivo del programa ocurre principalmente en las funciones **update**, **draw** y **keyPressed**.

---

# Generación de trazos (update)

En la función **update** se detecta si el usuario está presionando el botón del mouse.

Cuando esto ocurre:

1. Se obtiene la posición actual del cursor.
2. Se genera un radio aleatorio para el trazo.
3. Se genera un color aleatorio utilizando el sistema **HSB**.
4. Se envía el trazo a la cola utilizando **enqueue**.

A medida que el usuario mueve el mouse, se van agregando nuevos trazos a la cola.

Cuando el número de elementos supera el tamaño máximo, los trazos más antiguos se eliminan automáticamente.

---

### Imagen 5 — Inserción continua de nodos manteniendo el orden FIFO

*(Captura del depurador mostrando varios nodos en la cola y el orden de inserción)*

---

# Dibujo de los trazos (draw)

En la función **draw** se recorren todos los nodos de la cola para dibujar los trazos en pantalla.

Para hacerlo se utiliza un puntero llamado **current** que comienza en `strokes.front`. Luego se utiliza un ciclo **while** que avanza nodo por nodo hasta llegar a `nullptr`.

En cada iteración se dibuja un círculo utilizando:

- La posición almacenada en el nodo
- El radio del trazo
- El color almacenado

Además se calcula un valor de **opacidad** dependiendo de la posición del nodo dentro de la cola. Esto genera un efecto visual donde:

- Los trazos **más recientes son más visibles**
- Los trazos **más antiguos se desvanecen gradualmente**

Este efecto produce la sensación de que los trazos **desaparecen con el tiempo**.

---

### Imagen 6 — Recorrido de la cola en draw()

*(Captura mostrando el puntero `current` recorriendo la lista enlazada)*

---

# Interacción con el teclado

El programa también incluye interacción mediante el teclado.

Las teclas implementadas son:

### Tecla **c**

Ejecuta la función **clear**, eliminando todos los trazos almacenados en la cola y limpiando la pantalla.

---

### Tecla **a**

Alterna el tamaño máximo de la cola entre:

- **50 trazos**
- **100 trazos**

Esto permite modificar la cantidad de elementos visibles en pantalla.

---

### Tecla **s**

Guarda una captura del frame actual utilizando la función **`ofSaveFrame()`**.

---

# Control del Tamaño Máximo de la Cola

El programa incluye un control dinámico del tamaño máximo de la cola mediante la variable **maxSize**.

Cuando el usuario presiona la tecla **a**, el valor alterna entre **50 y 100** elementos.

Esto permite modificar la cantidad de trazos que pueden existir simultáneamente antes de que comiencen a eliminarse los más antiguos.

---

### Imagen 7 — Control del tamaño máximo de la cola

*(Captura mostrando el cambio de maxSize de 50 a 100 en el depurador)*

---

# Conclusión

En esta actividad se implementó desde cero una **estructura de datos tipo cola FIFO** utilizando nodos enlazados en C++. A través de esta implementación fue posible comprender cómo funcionan internamente las estructuras dinámicas y cómo se gestionan los punteros y la memoria.

El uso de funciones como **enqueue**, **dequeue**, **clear** e **isEmpty** permitió controlar completamente el comportamiento de la cola y garantizar que los datos se procesaran en el orden correcto.

Además, la integración de esta estructura con **openFrameworks** permitió visualizar el comportamiento de la cola de forma gráfica, generando un efecto de pintura dinámica donde los trazos más recientes permanecen visibles mientras los más antiguos desaparecen gradualmente.

Este ejercicio demuestra cómo las estructuras de datos no solo son conceptos teóricos, sino que también pueden aplicarse directamente para crear **sistemas interactivos y visuales** dentro de programas reales.

## Bitácora de reflexión






