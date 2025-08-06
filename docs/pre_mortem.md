# Pre-Mortem del Proyecto "Ratoneando"

---
**Navegación de la documentación:**

- [README](../README.md)
- [Arquitectura](architecture.md)
- [Lean Canvas](lean_canvas.md)
- [User Story Map v1.0](user_story_map_v1.0.md)

---

## Anticipando el Fracaso para Asegurar el Éxito

Este documento realiza un ejercicio de **Pre-Mortem**, una técnica de mitigación proactiva de riesgos. El objetivo es imaginar que el proyecto ha fracasado para identificar las posibles causas y desarrollar planes de acción para prevenirlas. El análisis se basa en la información de los documentos `lean_canvas.md`, `README.md`, y `user_story_map_v1.0.md`.

### Escenario de Fracaso: Seis Meses Post-Lanzamiento

**Imaginemos el peor escenario :** Han pasado seis meses desde el lanzamiento oficial de "Ratoneando" en Santa Cruz de la Sierra. El proyecto es un fracaso total. La tasa de adopción está por debajo del 5%, muy lejos del 30% mensual proyectado. El engagement es casi nulo, con usuarios que no superan una sesión semanal. "Ratoneando" no se ha convertido en la plataforma de referencia para el consumidor  y los costos operativos han superado las proyecciones, haciendo insostenible el proyecto.

### Análisis de las Causas del Fracaso

Se realiza una lluvia de ideas sobre las posibles causas que llevaron a este fracaso.

#### **Causas Relacionadas con la Tecnología y los Datos**

* **Inexactitud Crítica de los Datos:** El sistema de *web scraping* resultó ser frágil. Los cambios constantes en las webs de los comercios generaron precios desactualizados, destruyendo la confianza del usuario, pilar fundamental de la propuesta de valor.
* **Fallo en las Integraciones (APIs):** La búsqueda de integraciones directas vía APIs con proveedores no prosperó. Los grandes comercios no tuvieron interés y los pequeños carecían de la infraestructura técnica, dejando al *scraping* como único método, que resultó insuficiente.
* **Experiencia del MVP Demasiado "Mínima":** El MVP, aunque funcional, se percibió como incompleto. La ausencia de funcionalidades como el "Escaneo de código de barras" (planificada para la Release 2)  hizo que el proceso de agregar productos fuera tedioso en comparación con las alternativas existentes.

#### **Causas Relacionadas con el Mercado y el Usuario**

* **Baja Adopción por Desconfianza:** El público objetivo (15 a 45 años)  desconfió de la veracidad de los datos de una nueva app y prefirió seguir con sus métodos tradicionales, como recorrer locales físicamente.
* **Propuesta de Valor No Percibida:** El mensaje de "ahorro" no fue lo suficientemente potente. Los usuarios no percibieron un ahorro significativo que justificara el cambio de sus hábitos de compra.
* **Falta de Tracción en los Canales Definidos:** A pesar de los esfuerzos en redes sociales (Facebook, Instagram, TikTok), el contenido no logró volverse viral ni generar el crecimiento esperado. El marketing no resonó con los segmentos de clientes clave (jefes de hogar, estudiantes).

#### **Causas Relacionadas con el Modelo de Negocio y Estrategia**

* **Modelo de Monetización Rechazado:** La introducción de publicidad contextual y un modelo Freemium en las etapas iniciales fue percibida como intrusiva, alienando a los "early adopters" que buscaban una herramienta limpia y enfocada en la utilidad.
* **Costos Operativos Subestimados:** Los gastos en mantenimiento de servidores y bots de *scraping*  fueron más altos de lo previsto. Esto, combinado con los bajos ingresos, agotó el capital antes de poder alcanzar una masa crítica de usuarios.
* **Ventaja Diferencial No Sostenible:** El "enfoque 100% local" y la "integración de supermercados y farmacias"  no fueron barreras de entrada suficientes. Un competidor con mayor financiamiento replicó la funcionalidad básica rápidamente y ejecutó una mejor campaña de marketing.

### Priorización de Amenazas Críticas

De la lista anterior, se identifican las 3 amenazas más probables y de mayor impacto:

1.  **Datos de Precios Inexactos o Desactualizados:** Es la amenaza más crítica, ya que ataca el núcleo de la propuesta de valor de transparencia y confianza.
2.  **Baja Tasa de Adopción Inicial:** Si no se logra una masa crítica de usuarios rápidamente, el proyecto no será sostenible ni atractivo para futuras alianzas o modelos de ingreso.
3.  **Fallo en la Estrategia de Adquisición de Datos:** La dependencia total del *web scraping* es un punto único de fallo. Sin datos, la plataforma es inútil.

### Planes de Acción y Mitigación

Para cada amenaza, se desarrolla un plan de acción concreto para prevenirla.

**1. Amenaza: Datos de Precios Inexactos**
* **Acción Preventiva:** Implementar en el MVP una función visible y simple para que los usuarios puedan "Reportar un precio incorrecto". Esto no solo ayuda a depurar los datos, sino que crea una comunidad implicada.
* **Acción Preventiva:** Antes del lanzamiento, desarrollar prototipos de *scraping* para los 5 principales comercios objetivo y medir su tasa de error durante dos semanas para validar la robustez de la tecnología.
* **Acción Mitigadora:** Asignar recursos para una verificación manual semanal de los 20 productos más buscados en la canasta básica para garantizar su exactitud.

**2. Amenaza: Baja Tasa de Adopción Inicial**
* **Acción Preventiva:** Iniciar la campaña de marketing en redes sociales  un mes *antes* del lanzamiento, enfocada en educar sobre el problema del gasto no optimizado y crear expectativa.
* **Acción Preventiva:** Durante la fase de "Pruebas y Lanzamiento Beta" (2 meses), reclutar activamente a 100 "early adopters" de los segmentos clave (estudiantes de la UAGRM, profesionales jóvenes) y ofrecerles incentivos (ej. sorteos) por feedback detallado.
* **Acción Mitigadora:** Simplificar al máximo la experiencia del MVP: el usuario debe poder realizar una búsqueda y ver una comparación útil en menos de 3 clics, sin necesidad de registrarse para las funciones básicas de búsqueda y comparación.

**3. Amenaza: Fallo en la Estrategia de Adquisición de Datos**
* **Acción Preventiva:** Preparar una propuesta de valor clara y directa para los comercios, enfocada en cómo "Ratoneando" les dará mayor visibilidad frente a las grandes cadenas, una de las ventajas competitivas del proyecto.
* **Acción Preventiva:** Iniciar conversaciones con al menos 5 farmacias o supermercados pequeños/independientes *antes* de finalizar el desarrollo del MVP para explorar alianzas y entender sus barreras tecnológicas.
* **Acción Mitigadora:** Diseñar una herramienta de carga manual de precios extremadamente simple, para que los comercios aliados sin API puedan actualizar sus precios más importantes de forma voluntaria.

Este documento de Pre-Mortem debe ser revisado periódicamente para adaptar las estrategias a medida que el proyecto evoluciona y se obtiene más aprendizaje del mercado.