# Unidad 5
## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

### Fase 1: Aplicación Funcional



https://github.com/user-attachments/assets/82899757-d9ad-4228-84c1-410ffcb4d66a


#### Código:

ofApp.h
```.c++
#pragma once
#include "ofMain.h"
#include <vector>

// -------------------------------------------------
// Clase base abstracta: Particle
// -------------------------------------------------
class Particle {
public:
    virtual ~Particle() {}
    virtual void update(float dt) = 0;
    virtual void draw() = 0;
    virtual bool isDead() const = 0;
    virtual bool shouldExplode() const { return false; }
    virtual glm::vec2 getPosition() const { return glm::vec2(0, 0); }
    virtual ofColor getColor() const { return ofColor(255); }
};

// -------------------------------------------------
// RisingParticle
// -------------------------------------------------
class RisingParticle : public Particle {
protected:
    glm::vec2 position;
    glm::vec2 velocity;
    ofColor color;
    float lifetime;
    float age;
    bool exploded;
public:
    RisingParticle(const glm::vec2& pos, const glm::vec2& vel,
                   const ofColor& col, float life)
        : position(pos), velocity(vel), color(col),
          lifetime(life), age(0), exploded(false) {}

    void update(float dt) override {
        position += velocity * dt;
        age += dt;
        velocity.y += 9.8f * dt * 8;

        float explosionThreshold = ofGetHeight() * 0.15f + ofRandom(-30, 30);
        if (position.y <= explosionThreshold || age >= lifetime) {
            exploded = true;
        }
    }

    void draw() override {
        ofSetColor(color);
        ofDrawCircle(position, 10);
    }

    bool isDead() const override { return exploded; }
    bool shouldExplode() const override { return exploded; }
    glm::vec2 getPosition() const override { return position; }
    ofColor getColor() const override { return color; }
};

// -------------------------------------------------
// ZigZagParticle
// -------------------------------------------------
class ZigZagParticle : public Particle {
private:
    glm::vec2 position;
    glm::vec2 velocity;
    ofColor color;
    float age;
    float lifetime;
    float amplitude;
    float frequency;

public:
    ZigZagParticle(const glm::vec2& pos)
        : position(pos),
          velocity(glm::vec2(0, -ofRandom(200, 300))),
          color(ofColor::fromHsb(ofRandom(255), 200, 255)),
          age(0),
          lifetime(ofRandom(1.5f, 3.0f)),
          amplitude(ofRandom(50, 100)),
          frequency(ofRandom(2, 5)) {}

    void update(float dt) override {
        age += dt;
        position += velocity * dt;
        position.x += sin(age * frequency) * amplitude * dt;
    }

    void draw() override {
        ofSetColor(color);
        ofDrawCircle(position, 8);
    }

    bool isDead() const override { return age >= lifetime; }
    bool shouldExplode() const override { return age >= lifetime; }
    glm::vec2 getPosition() const override { return position; }
    ofColor getColor() const override { return color; }
};

// -------------------------------------------------
// SpiralParticle
// -------------------------------------------------
class SpiralParticle : public Particle {
private:
    glm::vec2 center;
    float angle;
    float radius;
    float radialSpeed;
    float verticalSpeed;
    float age;
    float lifetime;
    ofColor color;

public:
    SpiralParticle(const glm::vec2& pos)
        : center(pos),
          angle(0),
          radius(10),
          radialSpeed(ofRandom(20, 50)),
          verticalSpeed(ofRandom(-250, -150)),
          age(0),
          lifetime(ofRandom(1.5f, 3.0f)),
          color(ofColor::fromHsb(ofRandom(255), 220, 255)) {}

    void update(float dt) override {
        age += dt;
        angle += dt * 6;
        radius += radialSpeed * dt;
        center.y += verticalSpeed * dt;
    }

    void draw() override {
        glm::vec2 pos;
        pos.x = center.x + cos(angle) * radius;
        pos.y = center.y + sin(angle) * radius;

        ofSetColor(color);
        ofDrawCircle(pos, 6);
    }

    bool isDead() const override { return age >= lifetime; }
    bool shouldExplode() const override { return age >= lifetime; }
    glm::vec2 getPosition() const override { return center; }
    ofColor getColor() const override { return color; }
};

// -------------------------------------------------
// ExplosionParticle
// -------------------------------------------------
class ExplosionParticle : public Particle {
protected:
    glm::vec2 position;
    glm::vec2 velocity;
    ofColor color;
    float age;
    float lifetime;
    float size;
public:
    ExplosionParticle(const glm::vec2& pos, const glm::vec2& vel,
                      const ofColor& col, float life, float sz)
        : position(pos), velocity(vel), color(col),
          age(0), lifetime(life), size(sz) {}

    void update(float dt) override {
        position += velocity * dt;
        age += dt;
        float alpha = ofMap(age, 0, lifetime, 255, 0, true);
        color.a = alpha;
    }

    bool isDead() const override { return age >= lifetime; }
};

// -------------------------------------------------
// Tipos de explosión
// -------------------------------------------------
class CircularExplosion : public ExplosionParticle {
public:
    CircularExplosion(const glm::vec2& pos, const ofColor& col)
        : ExplosionParticle(pos, glm::vec2(0, 0), col, 1.2f, ofRandom(16, 32)) {
        float angle = ofRandom(0, TWO_PI);
        float speed = ofRandom(80, 200);
        velocity = glm::vec2(cos(angle), sin(angle)) * speed;
    }

    void draw() override {
        ofSetColor(color);
        ofDrawCircle(position, size);
    }
};

class RandomExplosion : public ExplosionParticle {
public:
    RandomExplosion(const glm::vec2& pos, const ofColor& col)
        : ExplosionParticle(pos, glm::vec2(0, 0), col, 1.5f, ofRandom(16, 32)) {
        velocity = glm::vec2(ofRandom(-200, 200), ofRandom(-200, 200));
    }

    void draw() override {
        ofSetColor(color);
        ofDrawRectangle(position.x, position.y, size, size);
    }
};

class StarExplosion : public ExplosionParticle {
public:
    StarExplosion(const glm::vec2& pos, const ofColor& col)
        : ExplosionParticle(pos, glm::vec2(0, 0), col, 1.3f, ofRandom(20, 40)) {
        float angle = ofRandom(0, TWO_PI);
        float speed = ofRandom(90, 180);
        velocity = glm::vec2(cos(angle), sin(angle)) * speed;
    }

    void draw() override {
        ofSetColor(color);
        int rays = 5;
        float outerRadius = size;
        float innerRadius = size * 0.5f;

        ofPushMatrix();
        ofTranslate(position);
        for (int i = 0; i < rays; i++) {
            float theta = ofMap(i, 0, rays, 0, TWO_PI);
            float xOuter = cos(theta) * outerRadius;
            float yOuter = sin(theta) * outerRadius;
            float xInner = cos(theta + PI / rays) * innerRadius;
            float yInner = sin(theta + PI / rays) * innerRadius;

            ofDrawLine(0, 0, xOuter, yOuter);
            ofDrawLine(xOuter, yOuter, xInner, yInner);
        }
        ofPopMatrix();
    }
};

// -------------------------------------------------
// NUEVA: RingExplosion
// -------------------------------------------------
class RingExplosion : public ExplosionParticle {
public:
    RingExplosion(const glm::vec2& pos, const ofColor& col)
        : ExplosionParticle(pos, glm::vec2(0, 0), col, 1.4f, ofRandom(10, 20)) {

        float angle = ofRandom(0, TWO_PI);
        float speed = ofRandom(120, 180);
        velocity = glm::vec2(cos(angle), sin(angle)) * speed;
    }

    void draw() override {
        ofSetColor(color);
        ofNoFill();
        ofDrawCircle(position, size);
        ofFill();
    }
};

// -------------------------------------------------
// ofApp
// -------------------------------------------------
class ofApp : public ofBaseApp {
public:
    void setup();
    void update();
    void draw();
    void mousePressed(int x, int y, int button);
    void keyPressed(int key);

    std::vector<Particle*> particles;
    ~ofApp();

private:
    void createRisingParticle();
};
```
ofApp.cpp
```.c++
#include "ofApp.h"

// --------------------------------------------------------------
void ofApp::setup() {
    ofSetFrameRate(60);
    ofBackground(0);
}

// --------------------------------------------------------------
void ofApp::update() {
    float dt = ofGetLastFrameTime();

    for (int i = 0; i < particles.size(); i++) {
        particles[i]->update(dt);
    }

    for (int i = particles.size() - 1; i >= 0; i--) {

        if (particles[i]->shouldExplode()) {

            int explosionType = (int)ofRandom(4);
            int numParticles = (int)ofRandom(20, 30);

            for (int j = 0; j < numParticles; j++) {

                if (explosionType == 0) {
                    particles.push_back(new CircularExplosion(
                        particles[i]->getPosition(), particles[i]->getColor()));

                } else if (explosionType == 1) {
                    particles.push_back(new RandomExplosion(
                        particles[i]->getPosition(), particles[i]->getColor()));

                } else if (explosionType == 2) {
                    particles.push_back(new StarExplosion(
                        particles[i]->getPosition(), particles[i]->getColor()));

                } else {
                    particles.push_back(new RingExplosion(
                        particles[i]->getPosition(), particles[i]->getColor()));
                }
            }

            delete particles[i];
            particles.erase(particles.begin() + i);

        } else if (particles[i]->isDead()) {

            delete particles[i];
            particles.erase(particles.begin() + i);
        }
    }
}

// --------------------------------------------------------------
void ofApp::draw() {
    for (int i = 0; i < particles.size(); i++) {
        particles[i]->draw();
    }
}

// --------------------------------------------------------------
void ofApp::createRisingParticle() {

    float minX = ofGetWidth() * 0.35f;
    float maxX = ofGetWidth() * 0.65f;
    float spawnX = ofRandom(minX, maxX);

    glm::vec2 pos(spawnX, ofGetHeight());

    int type = (int)ofRandom(3);

    if (type == 0) {
        glm::vec2 target(ofGetWidth() / 2.0f + ofRandom(-300, 300),
                         ofGetHeight() * 0.10f + ofRandom(-30, 30));

        glm::vec2 direction = glm::normalize(target - pos);
        glm::vec2 vel = direction * ofRandom(250, 350);

        ofColor col;
        col.setHsb(ofRandom(255), 220, 255);

        float lifetime = ofRandom(1.5f, 3.5f);

        particles.push_back(new RisingParticle(pos, vel, col, lifetime));

    } else if (type == 1) {

        particles.push_back(new ZigZagParticle(pos));

    } else {

        particles.push_back(new SpiralParticle(pos));
    }
}

// --------------------------------------------------------------
void ofApp::mousePressed(int x, int y, int button) {
    createRisingParticle();
}

// --------------------------------------------------------------
void ofApp::keyPressed(int key) {
    if (key == ' ') {
        for (int i = 0; i < 1000; i++) {
            createRisingParticle();
        }
    }

    if (key == 's') {
        ofSaveScreen("screenshot_" + ofToString(ofGetFrameNum()) + ".png");
    }
}

// --------------------------------------------------------------
ofApp::~ofApp() {
    for (int i = 0; i < particles.size(); i++) {
        delete particles[i];
    }
    particles.clear();
}
```
### Fase 2: Fase 2 — Evidencias de comprensión con el depurador

### Evidencia 1 — Herencia en memoria

#### Breakpoint:

Se colocó el breakpoint en la línea donde se llama particles[i]->update(dt) dentro del método update() de la aplicación. Este punto fue elegido porque permite inspeccionar objetos a través de un puntero de tipo base (Particle*), lo cual es ideal para observar cómo la herencia se representa en memoria cuando el objeto real pertenece a una clase derivada como ZigZagParticle o SpiralParticle.

#### Captura:
<img width="1107" height="469" alt="image" src="https://github.com/user-attachments/assets/5c78638b-4fbd-46c2-a3d7-7d178be8e019" />


#### Explicación:
En la captura del depurador se observa un elemento del vector particles, el cual es un puntero de tipo base Particle*. Al expandirlo, el depurador revela que el tipo real del objeto es ZigZagParticle, lo que evidencia el uso de polimorfismo.

Dentro del objeto se pueden observar los atributos definidos en la clase derivada, como position, velocity, age, lifetime, amplitude y frequency. Estos campos corresponden específicamente a la implementación de SpiralParticle.

Aunque la clase base Particle no define atributos propios, el hecho de que el objeto sea accesible a través de un puntero base demuestra que en memoria el objeto contiene tanto la estructura base como la derivada, organizadas de forma contigua según el modelo de herencia de C++.

#### Justificación:
Esta evidencia demuestra comprensión del concepto de herencia en memoria, ya que permite observar cómo un objeto de una clase derivada (SpiralParticle) es tratado como un objeto de su clase base (Particle) mediante un puntero.

El depurador muestra que, aunque se accede al objeto a través de un tipo base, en memoria se conserva toda la estructura de la clase derivada, incluyendo sus atributos específicos. Esto confirma que en C++ los objetos derivados contienen la información de la clase base y extienden su estructura.

Además, esta observación valida el uso de polimorfismo, ya que el método update() es invocado desde el tipo base pero ejecuta la implementación correspondiente al tipo real del objeto, lo que evidencia despacho dinámico en tiempo de ejecución.

### Evidencia 2 — La _vtable de tu nuevo tipo

#### Breakpoint:
Se colocó el breakpoint en la llamada a particles[i]->update(dt) dentro del método update(), ya que en este punto se trabaja con un puntero de tipo base (Particle*) que puede referenciar distintos tipos reales de objetos. Esto permite comparar la _vtable de dos clases derivadas (SpiralParticle y RisingParticle) y analizar cómo sus implementaciones afectan la tabla de funciones virtuales

#### Captura:
<img width="1054" height="207" alt="image" src="https://github.com/user-attachments/assets/83fe3f95-3488-41de-887a-80197fc76995" />
<img width="891" height="205" alt="image" src="https://github.com/user-attachments/assets/7803250e-14fb-4ece-9e25-d7d3e937c0c9" />



#### Explicación:
En la captura del depurador se observa el puntero _vptr de dos objetos distintos: uno de tipo SpiralParticle y otro de tipo RisingParticle. Este puntero referencia la _vtable de cada objeto, la cual contiene las direcciones de las funciones virtuales que se ejecutarán en tiempo de ejecución.

Ambos objetos heredan directamente de la clase base Particle, por lo que sus _vtable contienen las mismas entradas en términos de métodos (update(), draw(), isDead() y shouldExplode()). Sin embargo, las direcciones a las que apuntan estas entradas son diferentes, ya que cada clase proporciona su propia implementación de estos métodos.

Por ejemplo, aunque ambos objetos tienen una entrada para update(), en un caso apunta a la implementación de SpiralParticle y en el otro a la de RisingParticle, lo que refleja comportamientos completamente distintos en ejecución.

#### Justificación:
Esta evidencia demuestra comprensión del funcionamiento del polimorfismo en C++ a nivel de la _vtable, ya que se observa que, aunque los objetos comparten la misma interfaz definida por la clase base Particle, cada uno posee su propia tabla de funciones virtuales.

La comparación evidencia que la estructura de la _vtable es similar en ambos casos debido a que comparten la misma clase base, pero las implementaciones asociadas a cada entrada son diferentes. Esto confirma que el despacho dinámico permite que una llamada a un método como update() o draw() a través de un puntero de tipo base ejecute la versión correspondiente al tipo real del objeto en tiempo de ejecución.

De esta forma, se valida que el polimorfismo no depende únicamente de la existencia de métodos virtuales, sino de la asociación entre el objeto y su _vtable específica.

### Evidencia 3 — Polimorfismo en tiempo de ejecución

#### Breakpoint:

Se colocaron dos breakpoints: uno en la llamada particles[i]->update(dt) dentro del método update() de la aplicación, y otro dentro del método update() de la clase SpiralParticle. Esta elección permite verificar si, al invocar el método a través de un puntero de tipo base (Particle*), el flujo de ejecución entra en la implementación correcta correspondiente al tipo real del objeto.
#### Captura:

<img width="1096" height="290" alt="image" src="https://github.com/user-attachments/assets/37aec924-cec7-49f8-a835-b16100617802" />
<img width="1083" height="309" alt="image" src="https://github.com/user-attachments/assets/12a88aeb-252f-4592-8197-6cc49c27ed7c" />

#### Explicación:

En la captura se observa que el programa se detiene inicialmente en la llamada particles[i]->update(dt), donde particles[i] es un puntero de tipo base Particle*. Al avanzar paso a paso (Step Into), el flujo de ejecución entra en el método update() de la clase SpiralParticle.

Esto indica que, aunque la llamada se realiza desde un puntero de tipo base, el sistema identifica correctamente el tipo real del objeto en tiempo de ejecución y ejecuta la implementación correspondiente.

Además, se puede observar en el depurador que el objeto inspeccionado es de tipo SpiralParticle, lo que confirma la coherencia entre el tipo dinámico del objeto y el método ejecutado.
#### Justificación:
Esta evidencia demuestra el funcionamiento del polimorfismo en tiempo de ejecución en C++, específicamente el despacho dinámico de métodos virtuales. A pesar de que el método update() es invocado a través de un puntero de tipo base (Particle*), el programa ejecuta la versión definida en la clase derivada (SpiralParticle).

Esto ocurre gracias al uso de la _vtable, la cual permite que en tiempo de ejecución se seleccione la implementación correcta del método según el tipo real del objeto. La transición observada en el depurador confirma que no se ejecuta una implementación genérica ni la de otra clase, sino la específica del tipo dinámico.

De esta forma, se valida que el sistema de partículas utiliza correctamente el polimorfismo para permitir comportamientos distintos bajo una misma interfaz.

### Evidencia 4 — Encapsulamiento en el contexto de herencia

#### Breakpoint:

Se colocó el breakpoint en el método update() de la clase ZigZagParticle, accedido a través de un puntero de tipo base (Particle*). Este punto permite inspeccionar el objeto en ejecución y analizar cómo se organizan sus atributos dentro del contexto de herencia, observando específicamente qué miembros son propios de la clase derivada y cómo se representa el encapsulamiento en el depurador.

#### Captura:
<img width="1076" height="199" alt="image" src="https://github.com/user-attachments/assets/9590b79f-f75c-44d4-9be8-fcd418755454" />


#### Explicación:
En la captura del depurador se observa el objeto ZigZagParticle inspeccionado a través del puntero this. Dentro de su estructura se pueden identificar claramente los atributos definidos en la clase derivada, como position, velocity, color, age, lifetime, amplitude y frequency.

Estos campos representan el estado interno propio del objeto y son los únicos accesibles directamente dentro de la subclase. Además, se observa la presencia de la estructura interna asociada a la clase base Particle, representada indirectamente a través del puntero virtual (_vptr), que permite el funcionamiento del polimorfismo.

El depurador muestra que, aunque el objeto contiene toda su estructura en memoria, el acceso a los miembros sigue estando restringido por el nivel de encapsulamiento definido en el código, respetando las reglas de acceso de C++.

#### Justificación:
Esta evidencia demuestra comprensión del encapsulamiento dentro de una jerarquía de herencia en C++. El objeto ZigZagParticle contiene múltiples atributos propios que representan su comportamiento, los cuales son visibles en el depurador cuando se inspecciona a través de this.

Sin embargo, el acceso a estos datos sigue estando controlado por los modificadores de acceso del lenguaje. Los atributos privados permanecen encapsulados dentro de la clase, mientras que los miembros heredados mantienen su nivel de visibilidad según la definición de la clase base.

Esto demuestra que, aunque el depurador permite visualizar la estructura completa del objeto en memoria, el encapsulamiento sigue siendo respetado en tiempo de ejecución y en el diseño del programa.

### Evidencia 5 — Ciclo de vida completo de tu partícula

#### Breakpoint:

Se utilizaron tres puntos de inspección en el depurador para observar el ciclo de vida completo de un objeto ZigZagParticle. El primero en el momento de su creación al ser insertado en el vector particles, el segundo durante su actualización en el método update(), y el tercero en el momento de su eliminación del vector y liberación de memoria. Esta selección permite analizar el comportamiento completo del objeto desde su creación hasta su destrucción.
#### Captura:
<img width="857" height="238" alt="image" src="https://github.com/user-attachments/assets/3e26c16e-2bbc-4902-a38a-8087e095a07d" />
<img width="1090" height="398" alt="image" src="https://github.com/user-attachments/assets/c2815124-25a8-42e3-9f92-9a70203c0187" />


#### Explicación:
En la primera etapa se observa la creación de una instancia de ZigZagParticle, la cual es añadida al vector particles mediante asignación dinámica. En este punto, el objeto es creado en memoria heap y su dirección es almacenada en el contenedor.

Durante la segunda etapa, el objeto es actualizado en cada frame mediante el método update(), donde sus atributos como position, velocity y age cambian progresivamente. Esto representa la fase activa del ciclo de vida del objeto.

Finalmente, en la tercera etapa, se evalúa si la partícula ha alcanzado su condición de eliminación. Cuando esto ocurre, el objeto es eliminado explícitamente con delete y posteriormente removido del vector mediante erase, liberando así la memoria asociada.

#### Justificación:

Esta evidencia demuestra comprensión del ciclo de vida completo de un objeto dinámico en C++. Se observa cómo una instancia de ZigZagParticle es creada en memoria dinámica, almacenada dentro de un contenedor, actualizada durante múltiples iteraciones del programa y finalmente eliminada cuando cumple su condición de vida útil.

El uso del depurador permite visualizar claramente cada fase del ciclo: creación (asignación en heap), uso (actualización de atributos en tiempo de ejecución) y destrucción (liberación de memoria). Esto evidencia el manejo correcto de memoria dinámica y la relación entre objetos y su gestión dentro de un sistema basado en punteros.

Además, se confirma que el programa evita acumulación de memoria al eliminar correctamente los objetos que ya no son necesarios.

### Evidencia 6 — Sin fugas de memoria

#### Breakpoint:

Se colocó el breakpoint en el bloque donde se elimina una partícula del vector particles, específicamente en las instrucciones delete particles[i] y particles.erase(...). Este punto permite observar directamente cómo se libera la memoria dinámica asociada al objeto y cómo el puntero es removido del contenedor, asegurando que no queden referencias inválidas.
#### Captura:

<img width="1113" height="423" alt="image" src="https://github.com/user-attachments/assets/2ba5b169-b4db-4fb2-a7c7-f04022e0598f" />

#### Explicación:

Se colocó el breakpoint en el bloque donde se elimina una partícula del vector particles, específicamente en las instrucciones delete particles[i] y particles.erase(...). Este punto permite observar directamente cómo se libera la memoria dinámica asociada al objeto y cómo el puntero es removido del contenedor, asegurando que no queden referencias inválidas.
#### Justificación:
Esta evidencia demuestra comprensión del manejo de memoria dinámica en C++ y del correcto ciclo de liberación de objetos. El uso combinado de delete y erase asegura que primero se libere la memoria asignada en el heap y luego se elimine la referencia dentro del vector.

El delete destruye físicamente el objeto en memoria, mientras que erase elimina el puntero del contenedor, evitando accesos a memoria inválida (dangling pointers).

La verificación en el depurador permite observar que el tamaño del vector disminuye y que el objeto deja de existir en la estructura de datos, confirmando que no hay fugas de memoria en el sistema.

### Evidencia 7 — Prueba de condición límit

#### Breakpoint:

Se diseñó un escenario de prueba deliberado en el cual se presiona la tecla de espacio para generar una creación masiva de partículas (1000 instancias simultáneas). Este caso extremo permite evaluar el comportamiento del sistema bajo carga elevada, especialmente el crecimiento del vector particles, la ejecución de múltiples explosiones y la correcta gestión de memoria en condiciones de alta demanda.
#### Captura:
<img width="1086" height="405" alt="image" src="https://github.com/user-attachments/assets/86d18516-0e7c-44a7-93fc-44a79e42f47b" />
<img width="895" height="517" alt="image" src="https://github.com/user-attachments/assets/4907dd76-74ca-4c72-82ce-675e303cb82f" />


#### Explicación:
En la captura del depurador se observa el sistema en una condición de carga extrema, donde se han generado cientos de instancias de partículas simultáneamente mediante la creación masiva activada por el usuario.

El vector particles crece rápidamente debido a la inserción de múltiples objetos dinámicos, lo que provoca un aumento significativo en la cantidad de actualizaciones y verificaciones en cada frame.

Durante esta ejecución, se puede observar cómo cada partícula sigue su ciclo de vida completo (creación, actualización y eventual eliminación), incluso bajo condiciones de alta carga, lo que permite evaluar la estabilidad del sistema.

#### Justificación:

Esta evidencia demuestra la capacidad del sistema para manejar condiciones límite relacionadas con la creación masiva de objetos dinámicos. Al generar un número elevado de partículas en un corto periodo de tiempo, se pone a prueba tanto la eficiencia del manejo del vector como la correcta liberación de memoria.

El análisis en el depurador permite verificar que, a pesar de la alta carga, el sistema continúa ejecutando el ciclo de vida de las partículas de forma coherente, eliminando correctamente los objetos cuando cumplen sus condiciones de destrucción.

Esto confirma que la implementación es robusta frente a escenarios extremos y que el manejo de memoria dinámica evita acumulaciones permanentes que podrían afectar el rendimiento.



## Bitácora de reflexión
