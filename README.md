# Mari — asistente de voz estilo Jarvis (Android / Kotlin)

Flujo: **"Mari"** (wake word) → **STT** → **Claude API** → **TTS** → vuelve a esperar "Mari".

## Estructura

```
MariAssistant/
├── settings.gradle.kts · build.gradle.kts · gradle.properties
├── local.properties.example          ← copia las claves a local.properties
├── gradle/wrapper/gradle-wrapper.properties
└── app/
    ├── build.gradle.kts
    └── src/main/
        ├── AndroidManifest.xml
        ├── assets/mari_android.ppn   ← TÚ lo generas (ver LEEME_WAKEWORD.txt)
        ├── res/ (values, drawable, mipmap)
        └── java/com/mari/assistant/
            ├── MainActivity.kt                 UI host + permisos
            ├── service/MariForegroundService.kt  orquesta todo el flujo
            ├── wakeword/WakeWordDetector.kt    Porcupine
            ├── voice/SpeechToText.kt           SpeechRecognizer
            ├── voice/TTSManager.kt             TextToSpeech
            ├── ai/ApiService.kt                Messages API (OkHttp)
            ├── ai/SystemPrompt.kt              personalidad de Mari
            ├── state/AssistantState.kt         estado compartido Service↔UI
            └── ui/OrbScreen.kt                 orbe brillante (Compose)
```

## Puesta en marcha (5 pasos)

1. Abre la carpeta en **Android Studio** (Koala o superior) y deja que sincronice Gradle.
2. En `local.properties` añade:
   ```
   ANTHROPIC_API_KEY=sk-ant-...
   PICOVOICE_ACCESS_KEY=...
   ```
3. Entra en https://console.picovoice.ai/ → Porcupine → entrena la frase **Mari** (Android) → guarda el archivo como `app/src/main/assets/mari_android.ppn`.
4. Ejecuta en un **teléfono real** (el emulador no capta bien el micrófono ni tiene buen STT).
5. Pulsa **Activar a Mari**, concede micrófono/notificaciones, bloquea la pantalla y di "Mari".

## Notas importantes

- **Batería (Xiaomi, Samsung, Huawei, etc.):** desactiva la optimización de batería para la app, o el sistema matará el servicio con la pantalla apagada.
- **Android 14+:** el servicio de micrófono solo puede iniciarse mientras la app está visible (el botón de la UI lo cumple). Un arranque automático al encender el teléfono requiere un flujo distinto.
- **STT:** `SpeechRecognizer` usa el servicio de Google; en algunos teléfonos emite un pitido al iniciar y requiere internet. Alternativa: grabar con `AudioRecord` y enviar a la API de Whisper.
- **TTS más natural:** reemplaza `TTSManager.speak()` por una llamada a ElevenLabs (`POST /v1/text-to-speech/{voice_id}`) y reproduce el MP3 con `MediaPlayer`, manteniendo la firma `speak(text, onDone)`.
- **Seguridad:** la API key dentro del APK sirve solo para pruebas personales. Para distribuir la app, usa un backend propio como proxy.
- **Modelo:** se configura en `ApiService` (`claude-sonnet-5-5` por defecto).
- **Proactividad real** (alertas sin que la llames): añade un `WorkManager` periódico que consulte calendario/clima y haga hablar a Mari; el system prompt ya está preparado para ese tono.
