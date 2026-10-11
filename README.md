# Asistente de biblioteca por WhatsApp

Proyecto individual del Hackathon 2 de Automatización Inteligente con WhatsApp. El asistente permite consultar libros y salas de estudio, resolver dudas del reglamento y mantener el contexto entre mensajes.

## Problema y alcance

Una persona puede necesitar desplazarse a una biblioteca solo para preguntar si existe un libro, cuándo vuelve un ejemplar o qué horarios tiene una sala. El agente reúne estas consultas en WhatsApp.

El catálogo, el reglamento y los horarios usados son datos de demostración. El prototipo consulta información; no realiza reservas ni contacta automáticamente con personal humano.

## Cómo funciona

WhatsApp → webhook FastAPI → filtro de manipulación y Llama Guard → reglas o Llama 3.1 con memoria y herramientas → revisión de salida → WhatsApp.

- **Memoria:** Llama Stack conserva una conversación por usuario durante la sesión.
- **Herramientas:** `consultar_libro`, `buscar_en_reglamento` y `consultar_sala` permiten responder con datos concretos.
- **RAG:** SentenceTransformer genera embeddings del reglamento. NumPy recupera los dos fragmentos más cercanos con un umbral de 0.30.
- **Seguridad:** Llama Guard revisa entrada y salida. Un filtro local detecta determinadas frases de manipulación. Prompt Guard se intenta cargar sin credenciales adicionales, pero su acceso restringido impidió usarlo en esta ejecución; el filtro local no equivale a ese modelo.
- **Operación:** se ignoran mensajes duplicados y se procesan en orden los mensajes de un mismo usuario. Se registran rutas, tiempos, errores y estados de Meta sin imprimir teléfonos o secretos.

## Ejecutar el notebook

[Abrir el notebook en Colab](https://colab.research.google.com/drive/1u31MbJV6j98oNZN2BrfaE4T5yUDPXZtL).

1. Usar una sesión con GPU T4 e instalar las dependencias de la primera celda de código.
2. Reiniciar la sesión tras la instalación y ejecutar las celdas de importaciones y configuración en orden.
3. Configurar en Colab Secrets únicamente `WHATSAPP_TOKEN`, `PHONE_NUMBER_ID`, `VERIFY_TOKEN` y `NGROK_AUTHTOKEN`, y habilitar su acceso al cuaderno. No incluir sus valores en archivos.
4. Configurar en Meta la URL del túnel con `/webhook`, el token de verificación y la suscripción `messages`.
5. Ejecutar los escenarios desde un número propio autorizado. El bloque automático de esta entrega solicita ese número mediante un campo oculto y lo limita al destinatario autorizado de la prueba.

Las entradas automáticas son eventos controlados enviados al webhook público. Las respuestas se envían realmente por WhatsApp Cloud API; esto se distingue de mensajes escritos manualmente desde el teléfono.

## Resultados

Las comprobaciones locales de verificación válida e inválida, catálogo, consulta de sala y regla de manipulación pasaron. El modelo de embeddings recuperó el reglamento y Llama Guard permitió una consulta ordinaria y rechazó la prueba peligrosa.

El 11 de octubre de 2026 se ejecutaron ocho entradas controladas por el webhook público. WhatsApp Cloud API aceptó las ocho respuestas con HTTP 200. La repetición del último evento no produjo otra respuesta. La media de procesamiento de esa tanda fue de 5.0 segundos.

| Caso | Resultado observado |
| --- | --- |
| Saludo | Respuesta por regla fija |
| El principito | Sin disponibilidad; devolución del catálogo: 2026-10-13 |
| ¿Y cuándo regresa? | Falló la coherencia inicial: consultó un fragmento del reglamento |
| Libros a domicilio | Hasta 3 libros por 7 días |
| Sala A | Capacidad 4; horarios 10:00 y 12:00 |
| Atención humana | Indica acudir al mostrador; no promete contacto automático |
| Manipulación del prompt | Bloqueada por el filtro local |
| Contenido peligroso | Bloqueado por Llama Guard de entrada |

La revisión detectó el fallo de seguimiento y añadió un diccionario por usuario con el último libro consultado. La pregunta de devolución se resuelve usando ese título antes de llamar al modelo. La comprobación local de dos turnos pasó y devolvió `2026-10-13`. Después se autorizó y envió una respuesta adicional por WhatsApp: el agente devolvió la fecha correcta con HTTP 200. Esta corrección adapta el patrón de memoria y reglas fijas del curso.

Antes de esa tanda se registraron dos fallos porque Ollama dejó de responder; además, el token anterior fue rechazado con HTTP 401. Se renovó el token y se restablecieron los servicios. El resumen de la tanda conserva esos incidentes: 2 errores en 10 entradas. Los estados de entrega de Meta no llegaron al notebook; HTTP 200 acredita aceptación del envío, no entrega al dispositivo. El destinatario confirmó haber recibido las ocho respuestas en su WhatsApp.

![Pruebas de WhatsApp y deduplicación](pruebas-whatsapp.png)

![Corrección local del seguimiento](seguimiento-corregido.png)

![Seguimiento corregido enviado por WhatsApp](seguimiento-whatsapp.png)

El notebook conserva las respuestas originales y la comprobación de la corrección. No se oculta el resultado incoherente inicial.

## Fuentes



- [Actividad Hackathon 2 en Campus](https://edu.devf.la/es/campus/program/module/automatizacion-inteligente-con-whatsapp/agente-de-whatsapp/project/hackathon-2).
- [Presentación oficial](https://docs.google.com/presentation/d/1mD0beOpNmwAu_uXlJokB9RtxNSYV1JwboePQiV0zQ7c/edit): documentación en Markdown, respuestas coherentes y demostración del agente.
- [Hands-On oficial](https://colab.research.google.com/drive/1Rg_Cpa6dH3nCO3r0VqpXp8XRHphFNySo): base del notebook.
- [Fuentes originales y transcripción](https://notebook.google.com/notebook/0ea55faa-8f92-410b-acd0-dbd6c72167ef).

## Límites

Colab y ngrok son temporales; los servicios pueden detenerse y el token de pruebas debe renovarse. La memoria y la deduplicación se pierden al reiniciar el proceso. El filtro por frases es limitado. El webhook conserva el alcance académico del material y no incorpora verificación criptográfica de la firma de Meta. No se considera un servicio listo para atender al público general.
