Crea una «fábrica de software» sencilla y visual en este repositorio, utilizando una habilidad de Claude Code, subagentes y archivos Markdown. Mantén el proyecto a pequeña escala: cualquier persona que lo consulte debería poder entenderlo por completo en pocos minutos. Examina primero el repositorio para que los agentes conozcan su lenguaje, el comando de prueba y las convenciones utilizadas.


## El orquestador

La «fábrica» (factory) constituye una funcionalidad específica: `/factory`. Cualquier sesión de Claude Code que la ejecute se convierte en el orquestador: se encarga de ejecutar el bucle que aparece a continuación, pone en marcha a todos los agentes y es el único componente con capacidad de escritura en `factory/board.json`.


## El orquestador

La «fábrica» (factory) constituye una capacidad específica: `/factory`. La sesión de Claude Code que la ejecute se convierte en el orquestador: gestiona el bucle que se muestra a continuación, despliega a todos los agentes y es el único componente con permisos de escritura en `factory/board.json`.

## El bucle (modifícalo para alterar el funcionamiento de la fábrica)

┌──────────────────┐
│  funcionalidad   │
└─────────┬────────┘
▼
┌──────────────────┐
│ redactor de espec.│
│       opus       │
└─────────┬────────┘
▼
┌──────────────────┐
│    constructor   │◄────────────────────┐
│       opus       │                     │
└─────────┬────────┘                     │
┌─────────────┬──────┴──────┬─────────────┐         │
▼             ▼             ▼             ▼         │
┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐   │
│ seguridad │ │     ux    │ │ diseño ui │ │   código  │   │
│    opus   │ │   sonnet  │ │   sonnet  │ │   sonnet  │   │
└─────┬─────┘ └─────┬─────┘ └─────┬─────┘ └─────┬─────┘   │
└─────────────┴──────┬──────┴─────────────┘         │
▼                              │
┌──────────────────┐   si hay CAMBIOS:   │
│ ESPERAR A LAS 4  ├─────────────────────┘
│   revisiones     │   retomar construcción
└─────────┬────────┘   (máx. 2 rondas)
▼ todas APROBADAS
┌──────────────────┐
│    aprobador     │
│       opus       │
└─┬───────────────┬┘
APROBAR  │               │ ESCALAR
▼               ▼
┌─────────────┐ ┌─────────────┐
│ fusionar a  │ │ requiere    │ ◄── /factory approve
│   main      │ │ intervención│
└─────────────┘ └─────────────┘


## Sistema de archivos de destino

.claude/
  skills/factory/SKILL.md          # instrucciones para el orquestador
  agents/                          # un archivo por cada agente en el flujo
factory/
  board.json                       # estado persistente; alimenta el panel de control
  dashboard.html                   # panel visual; consulta board.json
  backlog.md                       # cola de funcionalidades para la siguiente
  jobs/003-rate-limiting/          # una carpeta por tarea (job)
    spec.md
    build.md
    round-1/                       # una carpeta por ronda de revisión
      review-security.md           # un archivo por revisor
      review-ux.md
      review-ui.md
      review-code.md
    decision.md

## Archivos de ejemplo

.claude/agents/<nombre>.md  (Sintaxis para subagentes de Claude Code)
---
name: example-agent
description: Una breve descripción de cuándo el orquestador debe utilizar este agente.
tools: Read, Grep, Glob, Bash, Write
model: sonnet
---
Qué lee este agente, qué tareas realiza y en qué archivo escribe.

factory/board.json
{ "jobs": [ { "id": "003-rate-limiting", "branch": "factory/003-rate-limiting",
"stage": "review", "round": 2,
"reviews": { "security": "CHANGES", "ux": "PASS", "ui": "pending", "code": "PASS" } } ] }

## Reglas
- Los agentes obtienen contexto de los archivos, no de la conversación, y solo escriben en su propio archivo.
- Cada revisión comienza con VERDICT: PASS o CHANGES; el archivo decision.md comienza con
APPROVE o ESCALATE.
- El «builder» (constructor) trabaja y realiza commits en una rama de funcionalidad (factory/<job-id>) y verifica el cumplimiento de los criterios de aceptación de la especificación.
- ESPERAR A TODOS: una ronda de revisión solo se considera finalizada cuando el archivo de cada revisor existe en la carpeta de la ronda. Nunca reanudes el «builder» a mitad de una ronda.
- Inicia el «builder» con un nombre (builder-<job-id>) y retómalo mediante `SendMessage` en cada nueva ronda, indicándole la carpeta correspondiente a dicha ronda.

## Comandos
/factory <feature>               ejecutar una nueva tarea a través del ciclo
/factory next                    ejecutar el primer elemento pendiente en backlog.md
/factory approve <job-id>        fusionar una tarea (que requiere intervención humana) en la rama principal (main)
/factory rework <job-id> <note>  devolver una tarea al «builder» incluyendo comentarios o retroalimentación

## Panel de control (Dashboard)
Un único archivo HTML autocontenido, que se sirve mediante `python3 -m http.server` desde el directorio `factory/`.
Las tareas aparecen como tarjetas que se desplazan a través de las etapas del ciclo. Cada ronda de revisión se visualiza como una fila de indicadores (chips) de los revisores, los cuales se completan a medida que se emiten los veredictos. Los empleos que requieren intervención humana destacan. Haz clic
en una tarjeta para leer sus archivos en formato Markdown. Limpio y minimalista.