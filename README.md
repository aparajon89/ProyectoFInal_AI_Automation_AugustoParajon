# Proyecto Final — Automatización con n8n y LLMs

## 1. Objetivo del proyecto

Diseñar e implementar un workflow automático en n8n que procese mensajes operativos provenientes de un grupo de WhatsApp vinculado a la logística de una empresa de retail, interprete información estructurada y no estructurada mediante modelos de lenguaje (LLMs), tome decisiones condicionales basadas en esas salidas y consolide los resultados en una base de datos utilizada como fuente para dashboards operativos.

El objetivo principal es automatizar la captura, interpretación y validación de información logística crítica (remitos, destinos, estados e inconsistencias), reducir la intervención manual y generar alertas y reportes accionables para los sectores de Compras e Ingresos Logísticos.

---

## 2. Descripción general del sistema

El sistema se apoya en un flujo automático compuesto por varias ramas funcionales y temporales. Los mensajes enviados a un grupo de WhatsApp entre un operador logístico externo y compradores internos son capturados mediante un webhook y almacenados en una hoja de cálculo base.

A partir de esa base, distintos procesos programados analizan imágenes y textos, extraen información relevante utilizando LLMs, aplican lógica condicional para clasificar y validar los datos, y actualizan la tabla que sirve como insumo para dashboards operativos y reportes diarios.

Este proyecto reemplaza el ejemplo académico de análisis de tweets por un caso real de mensajería logística, manteniendo la misma arquitectura exigida por la consigna: trigger automático, clasificación con LLM, decisión condicional y notificación.

---

## 3. Triggers y ejecución del flujo

El workflow utiliza exclusivamente triggers automáticos. No existen ejecuciones manuales.

Se implementan los siguientes disparadores:
- Webhook continuo para la recepción en tiempo real de mensajes de WhatsApp (texto e imágenes).
- Trigger programado a la 01:00 AM para procesar los mensajes del día anterior y analizar imágenes de remitos.
- Trigger programado a las 07:00 AM para detectar registros que requieren intervención del sector de Ingresos Logísticos.
- Trigger programado a las 10:00 AM para generar reportes operativos segmentados para el sector de Compras.
- Trigger por recepción de correos electrónicos para procesar respuestas de compradores con información faltante.

---

## 4. Arquitectura del workflow

El flujo se organiza en ramas independientes que operan sobre una misma base de datos central:

- Captura y almacenamiento de mensajes.
- Procesamiento nocturno y enriquecimiento de información logística.
- Clasificación condicional de destino.
- Validación de inconsistencias y control operativo.
- Comunicación automática con áreas internas.
- Actualización incremental de la tabla base.

Cada rama es trazable, auditable y puede ejecutarse sin interferir con las demás.

---

## 5. Modelos de Lenguaje (LLMs) utilizados

### LLM 1 — Clasificación de destino logístico

Este modelo tiene un objetivo único y acotado: determinar el destino logístico del remito.

La salida del modelo está estrictamente limitada a uno de los siguientes valores:
- `TUC`
- `MELI`

El prompt fuerza una respuesta cerrada, sin texto adicional. La salida es normalizada y utilizada directamente por un nodo condicional del workflow. Este LLM cumple el rol de clasificador dentro del flujo.

### LLM 2 — Interpretación y enriquecimiento de información

Este modelo se utiliza para interpretar información no estructurada proveniente de imágenes de remitos y mensajes de texto, así como para analizar respuestas de los compradores.

Entre sus funciones se incluyen:
- Extracción de datos relevantes desde imágenes (proveedor, número de remito, fechas, productos).
- Interpretación contextual de mensajes para completar información faltante.

Este LLM cumple un rol de generación y transformación de contenido, claramente diferenciado del LLM de clasificación.

---

## 6. Lógica condicional

La decisión principal del workflow se basa en la salida del LLM de clasificación de destino.

Un nodo condicional evalúa el valor devuelto:
- Si el valor es `TUC`, el flujo continúa y se actualiza la tabla base.
- Si el valor es `MELI`, el flujo se detiene o deriva sin impactar la base principal.

Se contempla una rama adicional para valores inesperados, que registra el evento y dispara una notificación de error.

---

## 7. Actualización de la base de datos

Toda la información procesada se consolida en una hoja de cálculo que funciona como base para dashboards operativos.

- Las columnas críticas se completan automáticamente.
- Los registros incompletos quedan identificados para revisión manual.
- Se aplican validaciones para evitar duplicación o inconsistencias.

---

## 8. Notificaciones

El sistema envía notificaciones automáticas en los siguientes escenarios:
- Registros que requieren intervención del sector de Ingresos Logísticos.
- Reportes diarios segmentados por comprador para el sector de Compras.
- Errores de ejecución o respuestas inesperadas de los LLMs.
- Confirmación cuando no se detectan inconsistencias.

Las notificaciones se envían por correo electrónico utilizando cuentas de prueba.

---

## 9. Manejo de errores y robustez

El workflow contempla fallos en:
- Respuestas inesperadas de los LLMs.
- Problemas en la lectura de imágenes.
- Errores en la actualización de la base de datos.

En estos casos, el flujo registra el error y notifica a los responsables correspondientes, evitando fallos silenciosos.

---

## 10. Ejecución, pruebas y limitaciones de acceso

El workflow integra servicios externos (WhatsApp, correo electrónico y hojas de cálculo) que requieren credenciales y accesos específicos de la organización. Por este motivo, el evaluador no podrá ejecutar pruebas de campo completas sin replicar dicho entorno.

Esta limitación es inherente al tipo de integración implementada y no afecta la validez técnica del proyecto. Para compensar esta restricción, se incluye evidencia visual de ejecuciones reales del flujo, donde se observa el funcionamiento end-to-end del sistema: inputs recibidos, salidas de los LLMs, decisiones condicionales, notificaciones enviadas y actualización de la base de datos.

---

## 11. Importación y configuración

1. Importar el archivo `workflow.json` en n8n.
2. Configurar las credenciales necesarias (Google Sheets, correo electrónico, webhook).
3. Definir las variables de entorno indicadas en la documentación.
4. Activar el workflow.

No se incluyen credenciales reales en los archivos entregados.

---

## 12. Limitaciones conocidas

- La interpretación de imágenes depende de la calidad de los remitos enviados.
- Los LLMs pueden presentar sesgos o errores ante información ambigua.
- El flujo está diseñado para un contexto operativo específico y puede requerir ajustes para otros entornos.
