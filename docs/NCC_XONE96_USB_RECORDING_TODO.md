# Pendiente: grabación directa de Xone:96 en NCC Audacity

**Estado:** pendiente para una próxima mejora; **no implementado ni probado**.
**Fecha de solicitud:** 2026-10-10.
**Palabras clave:** "grabación del Sony 96 en Audacity", "Xone 96 Audacity", "Master USB 11/12", "Input Channel Mapping".

## Objetivo

Permitir que NCC Audacity (fork basado en Audacity 4) grabe **solo la pareja estéreo USB 11/12** de la **Allen & Heath Xone:96** cuando el selector esté en **MST (Master)**. No usar Loopback, Traktor Audio 6 ni grabación de los 12 canales para después borrar diez.

## Contexto

- MacBook Pro con macOS, Traktor Pro 4 en modo External Mixing.
- Traktor reproduce por la interfaz USB de la Xone:96.
- Audacity es el grabador principal de DJ sets largos (habitualmente 3–5 horas).
- Flujo esperado: Master analógico Xone:96 → conversión A/D interna → USB 11/12 → pista estéreo L/R en Audacity.
- Conservar el flujo de proyectos, edición, normalización/limitación y exportaciones del fork.

## Especificación

1. Agregar **selector de entradas físicas de grabación** en Audio Setup/Preferencias.
2. Enumerar parejas disponibles según dispositivo; para Xone:96 permitir **USB 11/12 — Master estéreo**.
3. Mapear entradas físicas 11 y 12 (índices Core Audio/PortAudio **10** y **11**, base cero) a dos canales de captura L/R.
4. Abrir un flujo de **dos canales solamente**, sin capturar 12 y descartar los 10 restantes.
5. Recordar el dispositivo/par seleccionado, con degradación segura cuando no esté disponible.
6. Conservar comportamiento previo para otros dispositivos y otros sistemas.
7. Verificar grabaciones prolongadas, interrupciones, reconexión, frecuencia de muestreo y correspondencia L/R.

## Investigación técnica inicial

Repositorio: `ncc-music/audacity`.

En `au3/libraries/au3-audio-io/AudioIO.cpp`, alrededor de las líneas 643–644 en la versión consultada:

```cpp
captureParameters.hostApiSpecificStreamInfo = NULL;
captureParameters.channelCount = mNumCaptureChannels;
```

Investigar el uso de `PaMacCoreStreamInfo` y `PaMacCore_SetupChannelMap()` en PortAudio/Core Audio para establecer el mapeo **{10,11}** en `captureParameters.hostApiSpecificStreamInfo` con `channelCount = 2`; asegurar validación y vida útil de las estructuras antes/durante `Pa_OpenStream`. También será necesario exponer preferencia y UI en Audacity 4. Evitar confundir el mapeo de exportación con el mapeo físico de **entrada**.

## Criterios de aceptación

- Xone:96 USB seleccionada + MST en USB 11/12 → Audacity graba la mezcla Master en **una pista estéreo**.
- No almacena los otros 10 canales; mantiene calidad de la captura USB.
- Funciona en sesiones prolongadas sin dropouts ni pérdidas de sincronización.
- La selección persiste y no rompe grabación normal de otros dispositivos.

**Nota:** especificación de mejora futura, sin cambios todavía al motor de audio.
