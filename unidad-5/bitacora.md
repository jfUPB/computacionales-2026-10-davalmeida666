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

## Bitácora de reflexión
