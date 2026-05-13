# RPA Consulta Judicial - n8n

Automatización end-to-end para la prueba técnica de TCC: recepción de correo, extracción del nombre, consulta en la Rama Judicial, selección del sexto resultado, descarga del documento y envío por correo con adjunto.[cite:1]

La solución fue implementada en n8n self-hosted y usa Playwright desde un nodo `Execute Command` para resolver la automatización web sobre una interfaz dinámica basada en navegador real.[cite:1]

## 1. Herramienta utilizada y versión

- Herramienta principal: **n8n** self-hosted.[cite:1]
- Integraciones usadas: Gmail Trigger, Gmail, Code, Execute Command, Read/Write Files y HTTP Request.[cite:1]
- Complemento de automatización web: **Playwright** ejecutado desde shell dentro del flujo.[cite:1]

### Justificación técnica

Se eligió n8n porque permite orquestar en un solo workflow el disparador por correo, la preparación de datos, la ejecución del bot web, el manejo de archivos, el almacenamiento opcional y el envío final del correo.[cite:1]

Se eligió Playwright porque la consulta de la Rama Judicial requiere navegador real, renderizado JavaScript, espera por eventos y selección dinámica de elementos. Esta decisión evita una automatización frágil basada en coordenadas o scraping estático.[cite:1]

## 2. Resumen de la solución

El flujo implementa la siguiente secuencia funcional:[cite:1]

1. Detecta un correo entrante en Gmail.[cite:1]
2. Recupera el mensaje completo para acceder al cuerpo real del correo.[cite:1]
3. Extrae y valida el nombre completo de la persona a consultar.[cite:1]
4. Abre la página de consulta por nombre de la Rama Judicial.[cite:1]
5. Selecciona la opción **Todos los Procesos**.[cite:1]
6. Configura **Tipo de Persona = Natural**.[cite:1]
7. Ingresa el nombre consultado y ejecuta la búsqueda.[cite:1]
8. Identifica el sexto resultado público disponible.[cite:1]
9. Abre el detalle del proceso correspondiente.[cite:1]
10. Descarga el documento del proceso.[cite:1]
11. Envía un correo con el archivo adjunto.[cite:1]
12. Como mejora opcional, guarda el documento en Supabase Storage y genera una URL temporal.[cite:1]

### Diagrama del flujo

```text
[Gmail Trigger]
      |
      v
[Gmail - Obtener correo completo]
      |
      v
[Preparar entrada, validar nombre y parámetros]
      |
      v
[Ejecutar RPA Playwright - Shell]
      |
      v
[Validar RPA y preparar Supabase]
      |
      v
[Leer DOC descargado]
      |
      +------------------> [Subir DOC a Supabase Storage]
      |                                  |
      |                                  v
      |                       [Crear URL temporal Supabase]
      |                                  |
      +----------------------------------+
                                         |
                                         v
                 [Preparar correo final con adjunto y URL]
                                         |
                                         v
                               [Send a message]
```

El workflow también contiene una rama manual de pruebas con `Manual Trigger` y `Simular correo Gmail`, deshabilitada por defecto, útil para demos y validación sin depender de correos reales.[cite:1]

## 3. Cómo configurar y ejecutar el bot

### Requisitos previos

- n8n self-hosted operativo.[cite:1]
- Acceso a credenciales Gmail OAuth2 para lectura y envío de mensajes.[cite:1]
- Chromium instalado en el entorno donde corre n8n.[cite:1]
- Playwright disponible para ejecución desde shell.[cite:1]
- Acceso de escritura a carpetas locales para descargas, logs y screenshots.[cite:1]
- Proyecto Supabase con bucket privado, solo si se desea usar el respaldo opcional.[cite:1]

### Variables de configuración principales

La configuración está centralizada en el nodo `Preparar entrada, validar nombre y parámetros`. Allí deben ajustarse al menos estas variables:[cite:1]

| Variable | Descripción |
|---|---|
| `recipientEmail` | Correo destino al que se enviará el documento final.[cite:1] |
| `smtpFrom` | Correo remitente del proceso.[cite:1] |
| `baseUrl` | URL de consulta por nombre de la Rama Judicial.[cite:1] |
| `outputDir` | Carpeta local donde se guarda el documento descargado.[cite:1] |
| `logDir` | Carpeta local para logs de ejecución.[cite:1] |
| `screenshotDir` | Carpeta local para capturas de pantalla.[cite:1] |
| `headless` | Ejecución visible o no visible del navegador.[cite:1] |
| `timeoutMs` | Tiempo máximo de espera para acciones del navegador.[cite:1] |
| `chromiumPath` | Ruta del ejecutable de Chromium en el entorno.[cite:1] |
| `supabaseUrl` | URL del proyecto Supabase.[cite:1] |
| `supabaseBucket` | Bucket donde se guarda el documento como respaldo.[cite:1] |
| `supabaseServiceRoleKey` | Clave service role para cargar archivo y firmar URL.[cite:1] |
| `signedUrlExpiresInSeconds` | Vigencia de la URL temporal generada.[cite:1] |

### Credenciales necesarias

- **Gmail OAuth2** para el trigger y para el envío del mensaje final.[cite:1]
- **Supabase Service Role Key** si se habilita almacenamiento externo opcional.[cite:1]

### Pasos para importar y ejecutar

1. Importar el archivo JSON del workflow en n8n.[cite:1]
2. Configurar las credenciales Gmail OAuth2 en los nodos correspondientes.[cite:1]
3. Verificar que `chromiumPath` apunte a un ejecutable válido.[cite:1]
4. Confirmar que Playwright esté disponible en el entorno del contenedor o servidor.[cite:1]
5. Ajustar variables del nodo `Preparar entrada, validar nombre y parámetros`.[cite:1]
6. Ejecutar primero la rama manual de prueba para validar comportamiento controlado.[cite:1]
7. Revisar logs y screenshots generados por una ejecución exitosa.[cite:1]
8. Activar el `Gmail Trigger - Producción` solo después de validar correctamente la prueba manual.[cite:1]

## 4. Decisiones técnicas clave

### Estrategia para extraer el nombre desde el correo

La extracción del nombre se hace desde el payload completo de Gmail. El flujo decodifica contenido Base64URL cuando aplica, limpia HTML, normaliza espacios y luego intenta detectar un nombre válido por coincidencia explícita o por patrón general de varias palabras.[cite:1]

El flujo no continúa si el nombre detectado no cumple validación mínima. Esto reduce ejecuciones erróneas por correos vacíos, mal formados o sin nombre claro.[cite:1]

### Estrategia para identificar el sexto resultado

La selección del sexto resultado no depende de coordenadas ni de un selector fijo de posición. El script inspecciona filas visibles, excluye encabezados o filas no válidas, detecta radicados públicos y construye una lista dinámica de resultados utilizables antes de tomar el sexto registro real.[cite:1]

Esta decisión mejora la mantenibilidad y se ajusta al criterio de aceptación de la prueba, que exige seleccionar el sexto resultado y no un elemento fijo o el primer registro.[cite:1]

### Manejo de la descarga del archivo

Después de abrir el detalle del proceso, el script identifica la acción `Descargar DOC`, espera el evento real de descarga del navegador y guarda el archivo localmente en una ruta controlada.[cite:1]

La descarga luego se valida verificando existencia del archivo, tamaño mínimo y consistencia con el detalle del proceso esperado. Esto evita falsos positivos donde realmente se descargue una página incorrecta o el listado general.[cite:1]

### Estrategia de navegación estable

La navegación usa Playwright con esperas basadas en `domcontentloaded`, `networkidle`, visibilidad de elementos y verificación del detalle de proceso. Esta aproximación es más robusta que usar `sleep` fijo.[cite:1]

## 5. Manejo de errores y logging

El flujo implementa manejo básico de errores en varias capas: validación previa de parámetros, validación del nombre, control de salida del RPA y captura de errores durante la navegación web.[cite:1]

La ejecución genera un log en formato JSONL llamado `ejecuciones_rpa_consulta_judicial.jsonl`, útil para trazabilidad por corrida. También genera screenshots en etapas clave y una captura final en caso de éxito o fallo.[cite:1]

Cuando el nodo `Execute Command` no retorna la estructura esperada o reporta error, el nodo `Validar RPA y preparar Supabase` detiene el proceso con un mensaje claro y evidencia mínima de `stdout` y `stderr`.[cite:1]

## 6. Estructura del repositorio

La estructura sugerida para la entrega es esta:[cite:1]

```text
/rpa-consulta-judicial
  /flujo
    RPA-Consulta-Judicial-Gmail-Supabase-Adjunto-Version-Estandar-3.json
  /screenshots
    success_*.png
    01_pagina_inicial_*.png
    02_formulario_diligenciado_*.png
    03_resultados_*.png
    04_detalle_proceso_*.png
  /logs
    ejecuciones_rpa_consulta_judicial.jsonl
  README.md
```

## 7. Cumplimiento de la prueba técnica

La solución cumple el flujo principal solicitado: detección de correo, extracción del nombre, navegación a la Rama Judicial, selección del sexto resultado, descarga del documento y envío del correo con adjunto.[cite:1]

El componente de Supabase Storage es una mejora adicional para respaldo documental y URL temporal. No debe presentarse como requisito obligatorio, sino como valor agregado sobre el alcance mínimo de la prueba.[cite:1]

## 8. Limitaciones y trabajo pendiente

### Limitaciones actuales

- La automatización depende de la estructura visual y textual actual del sitio de la Rama Judicial.[cite:1]
- Cambios en selectores, textos o comportamiento de descarga pueden requerir ajustes en Playwright.[cite:1]
- Existe una posible inconsistencia entre `.doc` y `.docx` en parte de la lógica del flujo, que debe corregirse o explicarse antes de la entrega final.[cite:1]
- En el JSON hay parámetros sensibles que no deben publicarse en un repositorio compartido sin sanitización previa.[cite:1]

### Mejoras posibles

- Externalizar parámetros sensibles a credenciales o variables de entorno.[cite:1]
- Eliminar secretos hardcodeados del workflow exportado.[cite:1]
- Añadir reintentos automáticos ante fallos transitorios del sitio.[cite:1]
- Permitir procesamiento de múltiples correos en una sola ejecución.[cite:1]
- Parametrizar aún más el flujo para evitar valores fijos.[cite:1]

## 9. Recomendaciones para la demo técnica

Durante la presentación conviene explicar estos puntos:[cite:1]

- El flujo no depende de coordenadas de pantalla.[cite:1]
- La extracción del nombre se hace desde el cuerpo real del correo.[cite:1]
- La selección del sexto resultado es dinámica y basada en radicados detectados.[cite:1]
- La descarga se valida antes de continuar con el envío.[cite:1]
- Hay evidencia de ejecución mediante logs y screenshots.[cite:1]
- Supabase es un extra desacoplable, no una dependencia obligatoria del flujo principal.[cite:1]

## 10. Ejecución de prueba sugerida

Para una prueba controlada se recomienda usar primero la rama manual con `Ejecutar prueba manual` y `Simular correo Gmail`, validar la descarga, revisar el log y confirmar la recepción del correo final. Después de esa verificación se puede activar el trigger productivo sobre Gmail.[cite:1]
