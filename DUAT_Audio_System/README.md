# DUAT — Audio System (estructura independiente)

**Objetivo:** organizar los audios y servir como guía de integración para el proyecto editable de Unreal Engine 5.7.4.

## Importante: alcance de esta entrega
El ZIP recibido de DUAT contiene una build compilada para Windows (`.exe` y archivos `.pak/.ucas/.utoc`), no el proyecto fuente (`.uproject`, `Content/` editable y Blueprints fuente).
Por eso esta carpeta es una estructura externa de trabajo y documentación. **No contiene Blueprints `.uasset` compilados ni modifica el juego original.** Los `.uasset` nativos deben crearse/importarse y compilarse dentro del Unreal Editor.

## Cómo incorporarla
1. Abrí el proyecto fuente de DUAT en Unreal Engine 5.7.4.
2. Copiá la carpeta `Audio` dentro de `Content/` del proyecto.
3. Copiá `Blueprints`, `SoundClasses`, `SoundMixes`, `MetaSounds` y `Data` dentro de `Content/` también, manteniendo esta organización.
4. Importá tus WAV/OGG/MP3 desde el Content Browser. Para música larga, preferí WAV u OGG según el tamaño/calidad que necesites.
5. Seguí `Documentation/BLUEPRINT_IMPLEMENTATION.md` para crear los Blueprints en el editor.
6. No reemplaces archivos existentes con el mismo nombre sin revisar sus referencias.

## Convención de nombres
- Música: `MUS_Exploration_01`
- Efectos: `SFX_Player_Footstep_Stone_01`
- Ambientes: `AMB_Temple_Loop_01`
- Sound Classes: `SC_Master`, `SC_Music`, `SC_SFX`, `SC_Ambient`
- Blueprints: `BP_AudioManager`, `BP_PlayerAudio`, `BP_EnemyAudio`, `BP_EnvironmentAudio`

## Recomendaciones de audio
- Música e interfaz: normalmente 2D.
- Fuentes del mundo (enemigos, trampas, fuego, mecanismos): 3D con atenuación.
- Evitá reproducir sonidos de pasos en cada `Event Tick`; disparalos mediante Animation Notifies o un temporizador controlado por movimiento.
- Usá fades al cambiar música y evitá iniciar una segunda instancia de la misma pista.
- Configurá límites de concurrencia para sonidos repetitivos.


## Adaptación confirmada al repositorio ProtoTopDown
Repositorio: `MrLufaal/ProtoTopDown`  
Proyecto: `TopDownProto.uproject`  
Motor indicado por el descriptor: **Unreal Engine 5.8**  
Plugin activado: **PaperZD**.

La estructura original contiene recursos 2D bajo `Content/Assets`, incluyendo directorios de personajes y enemigos (por ejemplo, Anubis), además de recursos interactivos. Por eso esta propuesta está preparada para PaperZD y para conectar eventos de audio desde Anim Sequences/Anim Notifies cuando corresponda.

### Ubicación recomendada dentro del proyecto
Para respetar la organización existente, incorporar todo bajo:
`Content/DUAT_Audio_System/`

No se requiere mover, renombrar ni sobrescribir ningún asset original.
