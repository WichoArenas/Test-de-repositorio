# 🛠️ Práctica 02: Control GPIO - Semáforo Peatonal Interactivo

## 📝 Objetivo de la Práctica
Aplicar el manejo de entradas y salidas digitales (GPIO) para diseñar un semáforo vehicular con cruce peatonal. El sistema operará normalmente en verde para los autos, pero cambiará la secuencia para dar paso a los peatones cuando se presione un botón.

## 📦 Materiales Utilizados
* 1x Raspberry Pi Pico 2 W
* 1x Cable Micro-USB
* 5x LEDs (2 Verdes, 1 Amarillo, 2 Rojos)
* 5x Resistencias de 220Ω (Para los LEDs)
* 1x Push button (Botón de cruce peatonal)
* 1x Resistencia de 10kΩ (Pull-down para el botón)

## 🔌 Diagrama de Conexión y Simulación
![Diagrama de conexión](./diagrama.png)

**🔗 Enlace a la simulación en Wokwi:**
[Haz clic aquí para ver y correr la simulación](https://wokwi.com/projects/297322571959894536)

## 💻 Código y Explicación

### MicroPython (`main.py`)
```python
import machine
import time

# Configuración de salidas (LEDs)
led_auto_verde = machine.Pin(2, machine.Pin.OUT)
led_auto_amarillo = machine.Pin(3, machine.Pin.OUT)
led_auto_rojo = machine.Pin(4, machine.Pin.OUT)
led_peaton_verde = machine.Pin(6, machine.Pin.OUT)
led_peaton_rojo = machine.Pin(7, machine.Pin.OUT)

# Configuración de entrada (Botón con Pull-Down interno)
btn_peaton = machine.Pin(15, machine.Pin.IN, machine.Pin.PULL_DOWN)

# Estado inicial
led_auto_verde.value(1)
led_peaton_rojo.value(1)

while True:
    if btn_peaton.value() == 1:
        # Secuencia de precaución vehicular
        led_auto_verde.value(0)
        led_auto_amarillo.value(1)
        time.sleep(2)
        
        # Alto vehicular, paso peatonal
        led_auto_amarillo.value(0)
        led_auto_rojo.value(1)
        led_peaton_rojo.value(0)
        led_peaton_verde.value(1)
        time.sleep(5)
        
        # Regreso a estado inicial
        led_peaton_verde.value(0)
        led_peaton_rojo.value(1)
        led_auto_rojo.value(0)
        led_auto_verde.value(1)
        
    time.sleep(0.1) # Pequeño retardo para no saturar el procesador
```
**Explicación de la lógica en Python:**
Utilizamos la librería `machine` para configurar los pines 2, 3, 4, 6 y 7 como salidas digitales (`Pin.OUT`) y el pin 15 como entrada (`Pin.IN`). Se activó la resistencia `PULL_DOWN` interna de la Pico para evitar lecturas flotantes. El bucle `while True` evalúa constantemente si el botón ha sido presionado; si es así, ejecuta los retardos (`time.sleep`) para cambiar el estado de los LEDs y dar paso al peatón.

### C/C++ (`main.c`)
```c
#include "pico/stdlib.h"

// Definición de pines
#define AUTO_VERDE 2
#define AUTO_AMARILLO 3
#define AUTO_ROJO 4
#define PEATON_VERDE 6
#define PEATON_ROJO 7
#define BTN_PEATON 15

int main() {
    stdio_init_all();

    // Inicializar pines
    gpio_init(AUTO_VERDE); gpio_set_dir(AUTO_VERDE, GPIO_OUT);
    gpio_init(AUTO_AMARILLO); gpio_set_dir(AUTO_AMARILLO, GPIO_OUT);
    gpio_init(AUTO_ROJO); gpio_set_dir(AUTO_ROJO, GPIO_OUT);
    gpio_init(PEATON_VERDE); gpio_set_dir(PEATON_VERDE, GPIO_OUT);
    gpio_init(PEATON_ROJO); gpio_set_dir(PEATON_ROJO, GPIO_OUT);
    
    // Configurar botón con pull-down
    gpio_init(BTN_PEATON);
    gpio_set_dir(BTN_PEATON, GPIO_IN);
    gpio_pull_down(BTN_PEATON);

    // Estado inicial
    gpio_put(AUTO_VERDE, 1);
    gpio_put(PEATON_ROJO, 1);

    while (true) {
        if (gpio_get(BTN_PEATON)) {
            // Secuencia
            gpio_put(AUTO_VERDE, 0);
            gpio_put(AUTO_AMARILLO, 1);
            sleep_ms(2000);
            
            gpio_put(AUTO_AMARILLO, 0);
            gpio_put(AUTO_ROJO, 1);
            gpio_put(PEATON_ROJO, 0);
            gpio_put(PEATON_VERDE, 1);
            sleep_ms(5000);
            
            // Regreso al inicio
            gpio_put(PEATON_VERDE, 0);
            gpio_put(PEATON_ROJO, 1);
            gpio_put(AUTO_ROJO, 0);
            gpio_put(AUTO_VERDE, 1);
        }
        sleep_ms(100);
    }
}
```
**Explicación de la lógica en C/C++:**
Usamos el Pico SDK, incluyendo `pico/stdlib.h`. A diferencia de Python, aquí requerimos dos pasos por pin: `gpio_init()` para habilitarlo y `gpio_set_dir()` para establecer si es entrada o salida. Usamos `gpio_get()` para leer el estado del botón y la función `sleep_ms()` para manejar los tiempos de bloqueo del semáforo.

## 🎥 Evidencia en Video
[▶️ Ver demostración en video del semáforo aquí](https://www.youtube.com/watch?v=dQw4w9WgXcQ)

## 🤔 Conclusiones o Retos Superados
Al inicio el botón registraba múltiples pulsaciones seguidas (efecto rebote o *bouncing*). Comprendí que incluir el retardo general de `100 ms` al final del bucle principal ayuda a mitigar las lecturas falsas. Además, configurar el entorno CMake para C++ fue más complejo que simplemente darle "Play" en Thonny, pero me ayudó a entender el proceso de compilación de la tarjeta.
