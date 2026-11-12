# DUAT — Guía de implementación de Blueprints de audio
**Motor objetivo:** Unreal Engine 5.8 (según `TopDownProto.uproject`)
**Integración 2D:** PaperZD (plugin declarado en el proyecto)  
**Estado:** especificación para construir dentro del editor; no son assets `.uasset`.

## 1. BP_AudioManager — administrador persistente
**Recomendación:** implementar como `GameInstanceSubsystem` (Blueprint Class > All Classes > GameInstanceSubsystem), si el proyecto lo permite. Alternativa simple: Actor persistente en el nivel principal, con cuidado de no duplicarlo al cambiar de nivel.

### Variables
- `CurrentMusicComponent` — Audio Component (Object Reference)
- `CurrentMusic` — Sound Base (Object Reference)
- `MusicVolume` — Float (0.0–1.0), default 0.7
- `SFXVolume` — Float (0.0–1.0), default 0.8
- `AmbientVolume` — Float (0.0–1.0), default 0.6
- `CurrentState` — Enum `E_DUAT_AudioState`

### Enum E_DUAT_AudioState
`Menu`, `Exploration`, `Combat`, `Boss`, `SafeZone`, `Death`.

### Funciones
**PlayMusic(NewMusic: SoundBase, FadeInSeconds: Float)**
1. Validar `NewMusic`.
2. Si `NewMusic == CurrentMusic` y el componente está reproduciendo, salir.
3. Si existe música actual: `Fade Out` del Audio Component actual (por ejemplo 1.5 s).
4. Crear/reutilizar Audio Component para la nueva pista; asignar Sound = `NewMusic`.
5. Ajustar volumen inicial a 0 y ejecutar `Fade In` (por ejemplo 1.5 s).
6. Guardar referencias actuales.

**StopMusic(FadeOutSeconds: Float)**
- Si existe el componente: `Fade Out`; luego detener/liberar el componente según el diseño.

**SetMusicVolume(Value: Float) / SetSFXVolume(Value: Float)**
- `Clamp (Float)` entre 0 y 1.
- Aplicar mediante Sound Class / Sound Mix o multiplicador de volumen de los componentes.

**PlaySFX(Sound: SoundBase, Location: Vector, Is2D: Boolean)**
- Si `Is2D`: `Play Sound 2D`.
- Si no: `Spawn Sound at Location` (o `Spawn Sound Attached` para un actor).
- Aplicar clase de sonido correspondiente.

### Cambio de estado
Crear `SetAudioState(NewState: E_DUAT_AudioState)`. Usar `Switch on E_DUAT_AudioState` y asignar la pista correspondiente:
- Menu → `MUS_Menu`
- Exploration → `MUS_Exploration`
- Combat → `MUS_Combat`
- Boss → `MUS_Boss`
- SafeZone → `MUS_SafeZone`
- Death → `MUS_Death` (opcional) o `StopMusic`

Las referencias a las pistas se completan después de importar los audios.

## 2. BP_PlayerAudio — sonidos del jugador
Se puede implementar como Actor Component agregado al Blueprint del personaje.
### Eventos/funciones sugeridos
- `PlayFootstep(SurfaceType)`: reproducir una variación de pasos.
- `PlayAttack()`
- `PlayDamage()`
- `PlayDeath()`
- `PlayInteract()`

### Pasos
Preferencia: agregar `Anim Notify` en los cuadros de contacto del pie de cada animación. El Notify llama a `PlayFootstep`.
Si el personaje no usa animaciones con notifies, usar un Timer mientras `Speed > umbral`; detenerlo al quedar quieto. No usar `Play Sound` directamente en `Event Tick`.

### Superficie
Usar `Physical Material`/`Surface Type` si ya está configurado en el proyecto. Si no, comenzar con una variable `E_DUAT_Surface` (`Stone`, `Sand`, `Wood`, `Water`) y conectarla a un `Switch` para elegir el sonido.

## 3. BP_EnemyAudio — sonidos de enemigos
Actor Component agregado a cada enemigo o clase base común.
Funciones: `PlayIdle`, `PlayAlert`, `PlayAttack`, `PlayHit`, `PlayDeath`.
- Idle: reproducción ocasional aleatoria con Timer, no cada Tick.
- Alert: al detectar al jugador.
- Attack: en el evento real de ataque.
- Hit/Death: desde los eventos de daño y muerte existentes.
- Para sonido posicional: `Spawn Sound Attached` al componente/actor enemigo y configurar atenuación.

## 4. BP_EnvironmentAudio — sonidos ambientales
Actor Component o actores colocados en el nivel.
- Loop ambiental: `Spawn Sound 2D` si es una capa global; `Spawn Sound at Location` si pertenece a una zona.
- Objetos puntuales: `Audio Component` con `Auto Activate` según corresponda.
- Trampas/mecanismos: reproducir desde el evento de activación, no desde Tick.
- Configurar atenuación y, si corresponde, reverb por zona.

## 5. Organización de mezcla
Crear Sound Classes:
- `SC_Master`
  - `SC_Music`
  - `SC_SFX`
  - `SC_Ambient`
  - `SC_UI`

Crear Sound Mix opcional `SM_DUAT_Main` para controlar niveles generales. Enlazar cada Sound Wave/Sound Cue/MetaSound a su clase.

## 6. Integración segura
1. Primero crear y probar el Audio Manager en un mapa de prueba.
2. Agregar el componente `BP_PlayerAudio` al personaje después de identificar el Blueprint real del jugador.
3. Conectar eventos existentes de ataque, daño, muerte e interacción; no reemplazar la lógica original.
4. Agregar audio de enemigos individualmente o en una clase base compartida.
5. Agregar zonas ambientales como actores independientes.
6. Compilar cada Blueprint y probar transiciones, volumen, duplicación de música y cambios de nivel.

## 7. Datos que faltan para completar la integración real
- Proyecto fuente `.uproject` de DUAT 5.7.4.
- Blueprint del personaje y sus eventos de movimiento/combate.
- Blueprints o clase base de enemigos.
- Nombres de mapas/niveles y eventos de interacción.
- Audios finales (se pueden agregar más adelante).

## 8. Criterios de aceptación
- No hay cambios en los assets originales del juego.
- Una sola pista musical activa a la vez.
- La música cambia con fades entre estados.
- Los pasos no se disparan en cada Tick.
- Los SFX 3D disminuyen con la distancia.
- Volúmenes independientes de música, SFX y ambiente.
- No hay errores de compilación en los Blueprints.


## 9. Adaptación para PaperZD (proyecto ProtoTopDown)
En este proyecto el jugador y los enemigos utilizan animación 2D. Para pasos y golpes sincronizados:
1. Abrir la PaperZD Anim Sequence correspondiente.
2. Añadir un Anim Notify en el frame visual de contacto (pie contra el suelo, impacto del arma, etc.).
3. Crear/usar notifies dedicados, por ejemplo `PZAN_PlayFootstep` o `PZAN_PlayAttackSFX`.
4. En el evento del AnimBP/PaperZD que recibe el notify, obtener el componente `BP_PlayerAudio` y llamar a la función correspondiente.
5. Mantener el sonido disparado por evento de animación; no por `Event Tick`.

Si el personaje usa distintas animaciones por dirección (Up/Down/Side), colocar los notifies en cada secuencia que corresponda. Evitar usar el mismo notify para un golpe que ocurre en frames distintos de las animaciones.

### Recomendación para variaciones
Crear `E_DUAT_Surface` con `Stone`, `Sand`, `Wood`, `Water`. Si los Physical Materials/Surface Types ya existen, respetarlos; no modificar configuración global de colisiones hasta revisar los assets originales. Para la primera integración es válido asignar `Stone` como fallback.

## 10. Árbol recomendado del contenido importado
La carpeta descargada se debe copiar como:
`TopDownProto/Content/DUAT_Audio_System/`

El resultado de contenido será:
```text
Content/
├── Assets/                 # ORIGINAL — no modificar
└── DUAT_Audio_System/      # NUEVO — añadir por separado
    ├── Audio/
    ├── Blueprints/
    ├── SoundClasses/
    ├── SoundMixes/
    ├── MetaSounds/
    ├── Data/
    └── Documentation/
```

No se han editado archivos del repositorio. Esta entrega contiene estructura y documentos de implementación. Los Blueprints `.uasset`, Sound Classes, Sound Mixes, Enums y MetaSounds se crean y compilan dentro de Unreal Editor 5.8.
