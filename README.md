# AEROFLOW / LAB

Simulador CFD interactivo de un gabinete ATX en 3D, implementado como una **única página HTML autocontenida** (`index.html`) con Three.js y Tailwind CSS cargados por CDN.

## Cómo ejecutarlo

Puedes abrir `index.html` directamente en un navegador moderno. Para evitar restricciones de módulos/ESM en algunos navegadores, también puedes servirlo en local:

```bash
python3 -m http.server 8765 --bind 127.0.0.1
# Abre http://127.0.0.1:8765/index.html
```

Requiere conexión a Internet para descargar Tailwind, Three.js `0.160.0` y sus addons desde CDN.

## Controles

- **Clic izquierdo** sobre un ventilador de chasis, el cooler de la CPU o la PSU: encender / apagar.
- **Clic izquierdo** sobre la GPU: instalar / retirar la tarjeta.
- **Clic derecho** sobre un ventilador de chasis, el cooler o la PSU: menú contextual en la posición del cursor.
  - Chasis: intake/exhaust, RPM y power.
  - CPU: dirección de expulsión (atrás, arriba, frente, abajo), RPM y power.
  - PSU: ubicación inferior/superior, ventilador hacia exterior/interior, RPM y power.
- **Arrastrar**: orbitar. **Rueda**: zoom. **Encuadrar**: restaurar la cámara. **Reiniciar**: volver a la configuración inicial.
- **Densidad de partículas**: 100–1500 partículas en tiempo real.

## Modelo de simulación

- Campo vectorial con fuentes tipo *jet* para ventiladores de entrada y tipo *sink* para los de salida. La intensidad escala con las RPM.
- Inyección continua desde los ventiladores en modo intake y recirculación al escapar.
- Colisiones AABB contra componentes y paredes interiores, con rebote y dispersión tangencial para evitar partículas pegadas.
- Transferencia térmica cerca de la GPU, el cooler de la CPU y el VRM; color azul frío → amarillo tibio → rojo caliente.
- Escape por rejillas superiores/traseras según la presión interna (positiva, neutra o negativa).
- Telemetría de CFM de entrada/salida y presión resultante. La PSU solo suma caudal de salida cuando su ventilador mira hacia el interior; el cooler de la CPU se informa como flujo interno.

## Estructura

Todo el HTML, CSS, configuración de Tailwind, geometría, shaders y lógica de simulación están en `index.html`. `docs/PROMPT.md` contiene la especificación original.
