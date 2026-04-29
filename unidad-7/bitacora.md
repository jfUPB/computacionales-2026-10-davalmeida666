# Unidad 7

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
### Proyecto Final


https://github.com/user-attachments/assets/b68abef2-2e45-42c9-964d-97a9922c5fe9


### Fase1 1
triangle.cpp
```.c++
#include <glad/glad.h>
#include <GLFW/glfw3.h>

// ─── Constantes de pantalla ────────────────────────────────────────────────
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

// ─── IDs de objetos OpenGL ─────────────────────────────────────────────────
GLuint VAO, VBO;
GLuint shaderProg;

// ─── Construcción del shader program ──────────────────────────────────────
GLuint buildShaderProgram(const char* vertexSrc, const char* fragmentSrc) {
    // Vertex shader
    GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vertexSrc, NULL);
    glCompileShader(vertexShader);

    // Fragment shader
    GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentSrc, NULL);
    glCompileShader(fragmentShader);

    // Linkeo del programa
    GLuint program = glCreateProgram();
    glAttachShader(program, vertexShader);
    glAttachShader(program, fragmentShader);
    glLinkProgram(program);

    // Los shaders individuales ya no son necesarios
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);

    return program;
}

// ─── Configuración del VAO / VBO ──────────────────────────────────────────
void setupTriangle() {
    float vertices[] = {
        -0.5f, -0.5f, 0.0f,   // vértice inferior izquierdo
         0.5f, -0.5f, 0.0f,   // vértice inferior derecho
         0.0f,  0.5f, 0.0f    // vértice superior centro
    };

    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);

    glBindVertexArray(VAO);

    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    // Atributo 0: posición (x, y, z) — corresponde a layout(location = 0)
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    glBindVertexArray(0); // unbind
}

// ─── Entrada principal ────────────────────────────────────────────────────
int main() {
    // Inicialización de GLFW y configuración del contexto OpenGL 4.6 Core
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* mainWindow = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT,
        "Interactive Triangle", NULL, NULL);
    glfwMakeContextCurrent(mainWindow);

    // GLAD debe cargarse DESPUÉS de que el contexto esté activo
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
        return -1;
    }

    // ── Fuente del vertex shader ─────────────────────────────────────────
    const char* vertexShaderSrc = R"(
        #version 460 core
        layout(location = 0) in vec3 aPos;
        uniform vec2 offset;          // posición del mouse en NDC

        void main() {
            vec3 newPos  = aPos;
            newPos.x    += offset.x;
            newPos.y    += offset.y;
            gl_Position  = vec4(newPos, 1.0);
        }
    )";

    // ── Fuente del fragment shader ───────────────────────────────────────
    const char* fragmentShaderSrc = R"(
        #version 460 core
        out vec4 FragColor;
        uniform vec4 ourColor;        // color enviado desde C++

        void main() {
            FragColor = ourColor;
        }
    )";

    // Compilar, linkear y configurar
    shaderProg = buildShaderProgram(vertexShaderSrc, fragmentShaderSrc);
    setupTriangle();

    // Obtener ubicaciones de los uniforms (antes del loop)
    glUseProgram(shaderProg);
    int offsetLocation = glGetUniformLocation(shaderProg, "offset");
    int colorLocation = glGetUniformLocation(shaderProg, "ourColor");

    // ── Game loop ────────────────────────────────────────────────────────
    while (!glfwWindowShouldClose(mainWindow)) {

        // Limpiar framebuffer
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // Leer posición del mouse en píxeles de pantalla
        double xpos, ypos;
        glfwGetCursorPos(mainWindow, &xpos, &ypos);

        // Normalizar a [0, 1]
        float x = (float)xpos / (float)SCR_WIDTH;
        if (x < 0.0f) x = 0.0f;
        if (x > 1.0f) x = 1.0f;

        float y = (float)ypos / (float)SCR_HEIGHT;
        if (y < 0.0f) y = 0.0f;
        if (y > 1.0f) y = 1.0f;

        // Enviar color (R = x, G = y, B = 0, A = 1)
        glUniform4f(colorLocation, x, y, 0.0f, 1.0f);

        // Convertir a NDC:  x → [-1, 1]   y → [1, -1]  (eje Y invertido)
        glUniform2f(offsetLocation, x * 2.0f - 1.0f, 1.0f - y * 2.0f);

        // Dibujar
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);

        // Mostrar frame y procesar eventos
        glfwSwapBuffers(mainWindow);
        glfwPollEvents();
    }

    // ── Limpieza ─────────────────────────────────────────────────────────
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProg);

    glfwDestroyWindow(mainWindow);
    glfwTerminate();
    return 0;
}


```
### Fase 2
#### Evidencia 1
<img width="1018" height="343" alt="image" src="https://github.com/user-attachments/assets/fa138be9-497f-46ff-a415-d67bcbab6807" />

#### Explicación
En el depurador se observa que la variable mainWindow contiene una dirección de memoria válida, lo que indica que GLFW ha creado correctamente la ventana. Además, previamente se ejecutó glfwMakeContextCurrent, lo que garantiza que el contexto de OpenGL está activo en el hilo actual.

Este contexto es necesario para poder utilizar OpenGL, ya que contiene toda la información de estado requerida por la GPU. En este punto del programa se procede a ejecutar gladLoadGLLoader, cuya función es cargar las direcciones de las funciones de OpenGL disponibles en el sistema.

La correcta existencia del contexto antes de esta llamada asegura que GLAD puede inicializarse sin errores y que las funciones gráficas estarán disponibles para su uso posterior.
#### Justificación
GLFW debe inicializarse antes que GLAD porque es el encargado de crear la ventana y, más importante, el contexto de OpenGL. Este contexto es un requisito indispensable para poder interactuar con la API gráfica.

GLAD, por su parte, no implementa OpenGL directamente, sino que actúa como un cargador que obtiene en tiempo de ejecución las direcciones de las funciones de OpenGL a través del contexto activo, utilizando glfwGetProcAddress.

Si GLAD se ejecutara antes de que exista un contexto válido, no podría cargar estas funciones, lo que provocaría fallos en la ejecución del programa. Por ello, el orden de inicialización refleja la dependencia fundamental entre la creación del contexto (GLFW) y la carga de funciones (GLAD), lo cual es un paso esencial en el pipeline de inicialización de OpenGL.

#### Evidencia 2

<img width="1051" height="458" alt="image" src="https://github.com/user-attachments/assets/5d17d379-ca51-47e7-b66c-6936ab9502a4" />

#### Explicación
El arreglo de vértices se define en CPU como un conjunto de valores flotantes que representan las coordenadas del triángulo.

Posteriormente:

glBufferData transfiere ese arreglo desde la memoria del CPU hacia la GPU, almacenándolo en un VBO.
glVertexAttribPointer no mueve datos, sino que define cómo la GPU debe interpretar esos datos.
El layout```(location = 0)``` del vertex shader recibe esa interpretación y la usa como entrada ```(aPos)```.
Finalmente, glDrawArrays activa el pipeline gráfico donde la GPU ejecuta el shader usando esos datos.
#### Justificación
La evidencia demuestra que el arreglo de vértices no es consumido directamente por el shader, sino que existe una separación clara entre:

CPU: define y mantiene el arreglo vertices[]
GPU: almacena y procesa los datos mediante el VBO
```VAO + glVertexAttribPointer:``` actúan como el puente que conecta los datos con el shader

Esto confirma el principio del pipeline de OpenGL:

Los shaders no acceden a memoria del CPU, sino a buffers previamente configurados y enlazados mediante atributos.

Además, el hecho de ver el arreglo en el debugger antes de la configuración del atributo demuestra la transición explícita de datos desde CPU hacia GPU, lo cual es esencial para entender el funcionamiento del pipeline gráfico.

#### Evidencia 3
<img width="1047" height="325" alt="image" src="https://github.com/user-attachments/assets/5428c756-092e-4921-82ed-c901b5a7d4f0" />

#### Explicación
En el render loop de la aplicación se obtienen continuamente las coordenadas del cursor con glfwGetCursorPos, las cuales se normalizan a valores entre 0 y 1 para ser utilizadas como entrada dinámica.

Estos valores se envían al shader mediante el uniform:
``` .c++
glUniform4f(colorLocation, x, y, 0.0f, 1.0f);
```
Este uniform es recibido por el fragment shader como:
```.c++
uniform vec4 ourColor;
FragColor = ourColor;
```
De esta manera, el color del triángulo cambia en tiempo real dependiendo de la posición del mouse, sin modificar los vértices ni el VBO.
#### Justificación
El uso de uniforms permite modificar el comportamiento visual del shader sin alterar la geometría almacenada en la GPU.

En este caso:

El VBO permanece estático (no cambia la forma del triángulo)
El uniform ourColor se actualiza en cada frame
El fragment shader utiliza ese valor para definir el color final del píxel

Esto demuestra una separación fundamental del pipeline de OpenGL:

Los datos geométricos (VBO) son independientes del estado dinámico del shader (uniforms)

Además, el cambio visual en tiempo real confirma que los uniforms son variables globales del shader que pueden ser actualizadas desde la CPU sin necesidad de recompilar ni reenviar buffers completos.
#### Evidencia 4
<img width="1246" height="657" alt="image" src="https://github.com/user-attachments/assets/b55e13a5-ece8-4714-9129-3f32540dce0c" />

#### Explicación
Se realizó una prueba de borde modificando el valor del uniform offset, el cual controla la posición del triángulo en el vertex shader.

En lugar de utilizar los valores normales calculados a partir del mouse:
```.c++
glUniform2f(offsetLocation, x * 2.0f - 1.0f, 1.0f - y * 2.0f);
```
Se forzaron valores extremos como:
```.c++
glUniform2f(offsetLocation, 5.0f, 5.0f);
```
El vertex shader aplica este valor directamente a la posición de los vértices:
```.c++
newPos.x += offset.x;
newPos.y += offset.y;
```
Como resultado, todos los vértices del triángulo fueron desplazados fuera del rango visible del espacio de coordenadas normalizadas (NDC), por lo que el triángulo dejó de aparecer en pantalla.
#### Justificación

#### Evidencia 5

#### Explicación

#### Justificación

## Bitácora de reflexión
