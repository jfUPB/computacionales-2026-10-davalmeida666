# Unidad 5
## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

```markdown
# ofApp.h
```

```cpp
#pragma once
#include "ofMain.h"
#include <vector>

// -------------------- Clase base --------------------
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

// -------------------- RisingParticle --------------------
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

// -------------------- ZigZagParticle --------------------
class ZigZagParticle : public RisingParticle {
public:
    ZigZagParticle(const glm::vec2& pos, const glm::vec2& vel,
                   const ofColor& col, float life)
        : RisingParticle(pos, vel, col, life) {}

    void update(float dt) override {
        position += velocity * dt;
        age += dt;

        position.x += sin(age * 10) * 5;
        velocity.y += 9.8f * dt * 8;

        if (age >= lifetime) {
            exploded = true;
        }
    }

    void draw() override {
        ofSetColor(color);
        ofDrawRectangle(position.x, position.y, 8, 8);
    }
};

// -------------------- SpiralParticle --------------------
class SpiralParticle : public RisingParticle {
public:
    SpiralParticle(const glm::vec2& pos, const glm::vec2& vel,
                   const ofColor& col, float life)
        : RisingParticle(pos, vel, col, life) {}

    void update(float dt) override {
        age += dt;

        float angle = age * 5;
        position.x += cos(angle) * 4;
        position.y += velocity.y * dt;

        if (age >= lifetime) {
            exploded = true;
        }
    }

    void draw() override {
        ofSetColor(color);
        ofDrawCircle(position, 6);
    }
};

// -------------------- Explosion base --------------------
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

// -------------------- Tipos de explosión --------------------
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
        ofDrawCircle(position, size);
    }
};

// -------------------- ChaosExplosion --------------------
class ChaosExplosion : public ExplosionParticle {
public:
    ChaosExplosion(const glm::vec2& pos, const ofColor& col)
        : ExplosionParticle(pos, glm::vec2(0, 0), col, 1.5f, ofRandom(10, 30)) {
        velocity = glm::vec2(ofRandom(-300, 300), ofRandom(-300, 300));
    }

    void draw() override {
        ofSetColor(color);

        int shape = (int)ofRandom(3);
        if (shape == 0) {
            ofDrawCircle(position, size);
        } else if (shape == 1) {
            ofDrawRectangle(position.x, position.y, size, size);
        } else {
            ofDrawTriangle(position,
                           position + glm::vec2(size, 0),
                           position + glm::vec2(0, size));
        }
    }
};

// -------------------- ofApp --------------------
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

```markdown
# ofApp.cpp
```

```cpp
#include "ofApp.h"

void ofApp::setup() {
    ofSetFrameRate(60);
    ofBackground(0);
}

void ofApp::update() {
    float dt = ofGetLastFrameTime();

    for (int i = 0; i < particles.size(); i++) {
        particles[i]->update(dt);
    }

    for (int i = particles.size() - 1; i >= 0; i--) {

        if (particles[i]->shouldExplode()) {

            int explosionType = (int)ofRandom(4);
            int numParticles = (int)ofRandom(30, 60);

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
                    particles.push_back(new ChaosExplosion(
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

void ofApp::draw() {
    for (int i = 0; i < particles.size(); i++) {
        particles[i]->draw();
    }
}

void ofApp::createRisingParticle() {

    float minX = ofGetWidth() * 0.35f;
    float maxX = ofGetWidth() * 0.65f;

    glm::vec2 pos(ofRandom(minX, maxX), ofGetHeight());

    glm::vec2 target(ofGetWidth() / 2 + ofRandom(-300, 300),
                     ofGetHeight() * 0.1f);

    glm::vec2 dir = glm::normalize(target - pos);
    glm::vec2 vel = dir * ofRandom(250, 350);

    ofColor col;
    col.setHsb(ofRandom(255), 220, 255);

    float life = ofRandom(1.5f, 3.5f);

    int type = (int)ofRandom(3);

    if (type == 0) {
        particles.push_back(new RisingParticle(pos, vel, col, life));
    } else if (type == 1) {
        particles.push_back(new ZigZagParticle(pos, vel, col, life));
    } else {
        particles.push_back(new SpiralParticle(pos, vel, col, life));
    }
}

void ofApp::mousePressed(int x, int y, int button) {
    createRisingParticle();
}

void ofApp::keyPressed(int key) {
    if (key == ' ') {
        for (int i = 0; i < 300; i++) {
            createRisingParticle();
        }
    }
}

ofApp::~ofApp() {
    for (int i = 0; i < particles.size(); i++) {
        delete particles[i];
    }
    particles.clear();
}
```
El programa está hecho usando programación orientada a objetos, lo que significa que todo se organiza en clases.

Hay una clase principal llamada Particle, que funciona como base. A partir de ella se crean otros tipos de partículas que heredan sus características.
<img width="686" height="391" alt="image" src="https://github.com/user-attachments/assets/d6aa89ba-c43e-44c5-ba1e-3f9a1407b334" />
<img width="199" height="228" alt="image" src="https://github.com/user-attachments/assets/d86eeb5d-a5e5-4b72-8f76-cef42cfc82a3" />
<img width="420" height="40" alt="image" src="https://github.com/user-attachments/assets/45d000ff-b821-45cd-aad6-2ee01d710133" />

Existen dos tipos principales de partículas:

RisingParticle: son las partículas que salen desde abajo de la pantalla, suben y luego explotan.
ExplosionParticle: son las partículas que aparecen después de la explosión.

Gracias a la herencia, se pueden crear nuevas partículas con comportamientos diferentes sin cambiar la estructura del programa.

🔧 Modificaciones realizadas

Se añadieron nuevos tipos de partículas para hacer el sistema más interesante:

ZigZagParticle: se mueve en zigzag, cambiando su posición en el eje X usando una función seno.
SpiralParticle: se mueve en forma curva, como si hiciera una espiral.

También se agregó una nueva explosión:

ChaosExplosion: genera partículas en direcciones totalmente aleatorias y dibuja distintas formas (círculos, rectángulos y triángulos).

Además, el sistema ahora es más aleatorio:
<img width="482" height="235" alt="image" src="https://github.com/user-attachments/assets/c25ecad9-b693-4ed6-9be3-21770ed540ea" />
<img width="391" height="139" alt="image" src="https://github.com/user-attachments/assets/ba79497b-aa65-4f0e-ae8e-5e313f52cf24" />

El tipo de partícula cambia
El tipo de explosión cambia
La cantidad de partículas cambia
Incluso las formas que se dibujan cambian

Esto hace que la simulación sea más dinámica sin necesidad de rehacer todo el código.

🧠 Cómo funciona internamente (memoria y herencia)

Cuando se analiza una partícula como ZigZagParticle en el depurador, se puede ver cómo se organizan sus datos:

Primero están los datos de Particle
Luego los de RisingParticle
Y al final los propios de ZigZagParticle

Esto significa que en C++ la herencia se guarda en orden, como si una clase contuviera a la otra dentro.

⚙️ Funciones virtuales y _vtable

Las clases usan funciones virtuales, lo que permite que cada tipo de partícula tenga su propio comportamiento.

Internamente, C++ usa algo llamado _vtable, que es una tabla que guarda qué función debe ejecutar cada objeto.

Por ejemplo:

CircularExplosion y ChaosExplosion comparten funciones base
Pero cada una tiene su propia versión de draw()

Esto demuestra que cada clase decide cómo comportarse, aunque venga de la misma base.

🔁 Polimorfismo (clave del programa)

En el método update() se recorre un vector de Particle*, es decir, un conjunto de partículas sin importar su tipo.

A todas se les llama update(dt), pero:

ZigZagParticle se mueve en zigzag
SpiralParticle se mueve en espiral

Esto funciona porque el programa decide en tiempo real qué función ejecutar.
A esto se le llama polimorfismo.

🔐 Uso de protected

Las variables como position o velocity están marcadas como protected, lo que significa que:

Las clases hijas sí pueden usarlas
Pero no son completamente públicas

Esto permite reutilizar datos sin romper la estructura del código.

🔄 Ciclo de vida de una partícula

Cada partícula sigue este proceso:

Se crea y se guarda en un vector
Se actualiza en cada frame (update)
Cumple una condición (tiempo o altura)
Explota
Se elimina del sistema (delete + erase)

Esto asegura que:

Todo funcione correctamente
No haya errores
No haya fugas de memoria
🚀 Prueba de rendimiento
![Uploading image.png…]()

Se probó el programa generando muchas partículas al mismo tiempo (por ejemplo, presionando la barra espaciadora).

El sistema:

Sigue funcionando bien
Actualiza todas las partículas
Maneja las explosiones correctamente
Libera memoria sin errores

Esto demuestra que el programa es estable y eficiente, incluso con muchos objetos en pantalla.

## Bitácora de reflexión
