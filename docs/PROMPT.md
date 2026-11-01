Actúa como un ingeniero experto en gráficos por computadora (WebGL / Three.js), simulación de dinámica de fluidos computacional (CFD) simplificada para web y arquitectura frontend monolítica.

Necesito que crees una aplicación web interactiva 100% monolítica en un único archivo HTML que renderice en 3D un gabinete de PC con simulación física precisa del flujo de aire, partículas vectoriales y control absoluto de componentes mediante interactividad 3D con Raycasting (Clic izquierdo y Clic derecho).

REQUISITOS TÉCNICOS Y ARQUITECTÓNICOS ESTRICTOS:

1. Estructura y Estilos:
- Un único archivo HTML que incluya Tailwind CSS (via CDN) y Three.js + OrbitControls (via CDN).
- Interfaz moderna, minimalista con paneles translúcidos (glassmorphism) flotando sobre el viewport 3D.
- Panel derecho de telemetría y controles globales. NADA de panel izquierdo innecesario.

2. Modelo 3D del Gabinete y Hardware:
- Gabinete transparente (cristal templado ahumado con aristas metálicas y panel trasero sólido).
- Motherboard, Disipador CPU (con su ventilador orientable), Tarjeta Gráfica (GPU con toggle de presencia).
- Slots de ventiladores obligatorios y funcionales: Frontal (3 slots: f1, f2, f3), Superior / Techo (2 slots: t1, t2), Posterior (1 slot: r1), y Piso / Inferiores (2 slots: b1, b2).
- Fuente de Poder (PSU) configurable: Puede ubicarse en la parte inferior (moderno con shroud) o en la parte superior (clásico económico). La PSU debe tener modelado su ventilador principal (mirando hacia afuera o hacia el interior según configuración) y un segundo ventilador o rejilla en el lateral/costado de conexiones. La PSU debe ser seleccionable y configurable.

3. Motor de Partículas y Dinámica de Fluidos (CFD Real):
- NO uses movimiento aleatorio de partículas tipo ruido estático. Las partículas deben seguir un Campo Vectorial de Flujo (Streamlines).
- Las partículas deben inyectarse de forma continua desde los ventiladores configurados en modo "Intake" (Entrada) y moverse siguiendo la dirección vectorial normal de cada ventilador activo, acelerando o desacelerando según las RPM del ventilador.
- Las partículas deben reaccionar térmicamente: Al pasar cerca de la GPU o del disipador de la CPU, su temperatura interna sube y su color cambia progresivamente (Azul frío -> Amarillo tibio -> Rojo caliente).
- Colisión y rebote elástico: Las partículas rebotan contra las paredes internas del gabinete sin atravesarlas y sin quedarse pegadas. Si la presión es positiva, escapan de forma natural por las ranuras o ventiladores en modo "Exhaust" (Salida).
- Slider en la interfaz para controlar la densidad de partículas en tiempo real (de 100 a 1500 partículas).

4. Interactividad 3D Directa (Raycasting con Ratón):
- Clic Izquierdo en cualquier ventilador 3D (chasis, CPU, PSU): Alterna instantáneamente su estado de encendido/apagado (ON/OFF), actualizando el color del anillo y el giro de las aspas.
- Clic Derecho en cualquier ventilador 3D, en el cooler de la CPU o en la Fuente de Poder (PSU): Abre un Menú Contextual flotante posicionado exactamente en las coordenadas del cursor del mouse.
- El Menú Contextual por Clic Derecho debe permitir:
  - Para Ventiladores de Chasis: Cambiar flujo (Intake/Exhaust), ajustar RPMs con un slider en tiempo real y alternar Power (ON/OFF).
  - Para el Cooler CPU: Ajustar RPM, Power y seleccionar la dirección de expulsión del aire (Hacia Atrás, Hacia Arriba, Hacia el Frente, Hacia Abajo).
  - Para la Fuente de Poder (PSU): Cambiar ubicación (Inferior vs Superior), orientar el ventilador (Hacia el exterior vs Hacia el interior de la cabina) y Power.

5. Telemetría y Cálculos en Tiempo Real:
- Sumar CFM de entrada vs CFM de salida basados en las RPM y estado de cada ventilador.
- Calcular y mostrar si el case se encuentra en Presión Positiva, Negativa o Neutra con indicadores visuales dinámicos.

Escribe el código completo, limpio, autocontenido, funcional y libre de errores de sintaxis listo para abrirse directamente en el navegador.
