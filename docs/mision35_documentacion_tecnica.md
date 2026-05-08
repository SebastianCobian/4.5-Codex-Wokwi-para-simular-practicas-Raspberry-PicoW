# Documentación Técnica Oficial
## Materia: Lenguajes de Interfaz

**Alumno:** Humberto Sebastian Cobian Beas  
**Proyecto:** Misión 35 — API `/random` simulando JSON estilo Mockaroo  
**Plataforma objetivo:** Raspberry Pi Pico W + MicroPython  
**Versión del documento:** 1.0

---

## 1) Lógica en MicroPython

La arquitectura del proyecto implementa un servidor HTTP embebido en la Raspberry Pi Pico W que atiende solicitudes de red y expone una ruta de API (`/random`) para simular telemetría en formato JSON.

### 1.1 Inicialización de periféricos y red

Al arrancar, el script ejecuta tres bloques de preparación:

1. **Pantalla OLED por I2C**
   - Configura `I2C(0)` con `sda=Pin(0)`, `scl=Pin(1)` y frecuencia de `400000`.
   - Instancia `ssd1306.SSD1306_I2C(128, 64, i2c)` para una matriz de 128x64 px.
   - Muestra mensaje de arranque (`SISTEMA / INICIANDO...`).

2. **Conectividad WiFi (modo estación)**
   - Activa la interfaz `network.STA_IF`.
   - Ejecuta `wlan.connect(ssid, password)`.
   - Mantiene un bucle de espera bloqueante hasta obtener enlace con `wlan.isconnected()`.
   - Al conectar, imprime la IP y la dibuja en OLED para diagnóstico local.

3. **Carga de interfaz web**
   - Guarda en `html_page` una plantilla completa HTML/CSS/JS que será entregada en `/`.

### 1.2 Modelo de concurrencia del servidor web

El servidor usa `socket` nativo de MicroPython y ciclo principal `while True`:

- `s.accept()` acepta un cliente TCP.
- `cl.recv(1024)` lee la solicitud HTTP.
- Se extrae la ruta desde la línea inicial (`GET /ruta HTTP/1.1`).
- Se responde según ruta y se cierra conexión con `cl.close()`.

> **Concurrencia implementada:** patrón cooperativo secuencial por conexión corta (request/response). En cada iteración se atiende un cliente completo antes de pasar al siguiente. Aunque no hay hilos ni `uasyncio`, el navegador percibe operación fluida por el bajo tiempo de cómputo de cada transacción y por la asincronía del lado cliente (JavaScript `fetch`).

Este diseño es adecuado para práctica académica y carga ligera (telemetría simulada, un nodo y pocos clientes).

### 1.3 Ruta API `/random` (simulación de datos)

Cuando la ruta solicitada es `/random`, se ejecuta la generación pseudoaleatoria:

- **Temperatura:** `round(random.uniform(25.0, 120.0), 1)`
- **Magnitud sísmica:** `round(random.uniform(0.0, 8.5), 2)`
- **Identificador de sensor:** `SN-` + `random.randint(1000, 9999)`

#### Reglas de clasificación de estado

El estado operativo se determina por umbrales:

- `PELIGRO` si `temperatura > 90` **o** `magnitud_sismica > 6.0`
- `ADVERTENCIA` si no fue peligro y `temperatura > 60` **o** `magnitud_sismica > 4.0`
- `ESTABLE` en caso contrario

#### Integración físico-digital

Cada respuesta `/random` no solo regresa JSON al navegador; además actualiza de inmediato la OLED con:

- Línea de cabecera `--- ALERTA ---`
- Temperatura
- Magnitud sísmica
- Estado actual

Esto convierte la API en un puente directo entre software web y salida física embebida.

#### Formato de respuesta

El backend serializa con `json.dumps(datos)` y envía encabezado:

- `HTTP/1.1 200 OK`
- `Content-Type: application/json`

JSON resultante (estructura):

```json
{
  "sensor_id": "SN-4821",
  "temperatura": 78.4,
  "magnitud_sismica": 3.92,
  "estado": "ADVERTENCIA"
}
```

---

## 2) UIX (Stitch)

La interfaz gráfica está incrustada en el script MicroPython como cadena HTML (`html_page`) y se entrega desde la misma Pico W, sin servidor externo.

### 2.1 Diseño oscuro responsivo

La UI utiliza:

- **Tailwind CSS por CDN** con `darkMode: "class"`
- Paleta personalizada (`primary`, `secondary`, `error`, `background`, etc.)
- Tipografías `Inter` y `Space Grotesk`
- Distribución adaptable con utilidades responsivas (`grid`, `md:grid-cols-2`, `lg:pl-[280px]`)

Resultado visual:

- **Tema oscuro técnico** tipo consola de monitoreo.
- **Layout escalable**: barra lateral en pantallas grandes y flujo compacto en móviles/tablets.
- **Componentes de telemetría** con tarjetas para temperatura, sismo y estado global.

### 2.2 Terminal visual y retroalimentación operativa

El panel incluye un bloque tipo terminal (`json-box`) que presenta la respuesta formateada con `JSON.stringify(data, null, 4)`, simulando salida de ingeniería en tiempo real.

Además, el botón muestra estado de ejecución:

- Texto normal: “Simular Petición /random”
- Durante petición: icono `sync` con animación + “Procesando...”
- Finaliza restaurando su contenido original (bloque `finally`)

### 2.3 Cambio dinámico de color por severidad

La lógica JavaScript ajusta clases CSS de `estado-val` en función del estado recibido:

- `PELIGRO` → `text-error` (rojo)
- `ADVERTENCIA` → `text-secondary` (ámbar/naranja)
- `ESTABLE` → `text-primary` (verde)

Este mapeo semántico-color mejora la lectura operacional y reduce tiempo de interpretación por parte del usuario.

---

## 3) Adaptar Controles: botón “Simular Petición” como control de software

En esta práctica, el botón no es solo un elemento visual; se comporta como **control de software de mando** entre frontend y hardware embebido.

### 3.1 Flujo funcional bidireccional

Al presionar `#btn-simular` ocurre la secuencia:

1. **Evento de entrada (HMI):** `addEventListener('click', async () => { ... })`
2. **Comando al nodo físico:** `fetch('/random')`
3. **Procesamiento en hardware:** la Pico W calcula valores random y estado
4. **Salida física local:** OLED se actualiza con la nueva lectura
5. **Respuesta digital remota:** JSON vuelve al navegador
6. **Render en UI sin recarga:** DOM y terminal JSON se actualizan en caliente

### 3.2 Naturaleza de control adaptado

Se considera “adaptación de control” porque un input web abstracto (click) se traduce en:

- Activación lógica del backend embebido
- Generación de nueva “medición”
- Actualización simultánea de dos superficies de salida:
  - **Física:** pantalla OLED
  - **Virtual:** dashboard web

Esta convergencia valida el enfoque de Lenguajes de Interfaz: el control de usuario gobierna un sistema híbrido (interfaz + dispositivo).

### 3.3 Operación sin recarga

El uso de `fetch` asíncrono evita `refresh` de página:

- Se conserva el estado visual general del panel.
- Solo se reemplazan nodos puntuales (`temp-val`, `sismo-val`, `estado-val`, `json-box`).
- Se reduce latencia percibida y mejora la experiencia de monitoreo en tiempo real.

---

## Conclusión técnica

La Misión 35 implementa correctamente un patrón IoT didáctico completo en MicroPython: servidor HTTP embebido, API de simulación de datos tipo Mockaroo, interfaz oscura responsiva y control de software mediante eventos web asíncronos. El sistema demuestra integración funcional entre capa de presentación (UIX), lógica de aplicación (ruta `/random`) y dispositivo físico (OLED), cumpliendo con los objetivos de la materia de Lenguajes de Interfaz.
