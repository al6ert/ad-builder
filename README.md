# ad-builder

Repositorio de trabajo centrado en un skill local llamado `skill-creator`, pensado para crear, validar, evaluar y mejorar skills de agentes de forma iterativa.

Ahora mismo el valor principal del proyecto está en `.agents/skills/skill-creator/`: ahí viven las instrucciones del skill, scripts de automatización, agentes auxiliares, referencias de esquemas y un visor HTML para revisar resultados de evaluación.

## Qué incluye

- Un skill base en [`.agents/skills/skill-creator/SKILL.md`](/Users/persualia/Projects/ad-builder/.agents/skills/skill-creator/SKILL.md)
- Scripts Python para validar, evaluar y optimizar skills
- Agentes auxiliares para análisis, comparación y grading
- Un visor HTML para revisar evals localmente
- Assets y referencias para el flujo de trabajo del skill

## Estructura

```text
.agents/
└── skills/
    └── skill-creator/
        ├── SKILL.md
        ├── agents/
        │   ├── analyzer.md
        │   ├── comparator.md
        │   └── grader.md
        ├── assets/
        ├── eval-viewer/
        │   ├── generate_review.py
        │   └── viewer.html
        ├── references/
        │   └── schemas.md
        └── scripts/
            ├── aggregate_benchmark.py
            ├── generate_report.py
            ├── improve_description.py
            ├── package_skill.py
            ├── quick_validate.py
            ├── run_eval.py
            ├── run_loop.py
            └── utils.py
```

## Requisitos

- Python 3.10+
- `claude` CLI disponible en el entorno para los flujos que usan `claude -p`
- `PyYAML` para la validación de frontmatter

Instalación mínima:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pyyaml
```

## Flujo principal

El skill está pensado para este ciclo:

1. Definir o editar un skill.
2. Preparar prompts de prueba.
3. Ejecutar evaluaciones comparando comportamiento con y sin skill, o frente a una versión anterior.
4. Revisar resultados cualitativos y cuantitativos.
5. Ajustar el `SKILL.md` y repetir.

## Scripts útiles

### Validar un skill

```bash
python3 .agents/skills/skill-creator/scripts/quick_validate.py .agents/skills/skill-creator
```

Comprueba que exista `SKILL.md`, que el frontmatter YAML sea válido y que campos como `name` y `description` cumplan el formato esperado.

### Evaluar trigger de descripción

```bash
python3 .agents/skills/skill-creator/scripts/run_eval.py --help
```

Sirve para medir si la descripción del skill hace que el agente se active cuando debería, usando `claude -p`.

### Ejecutar bucle de mejora

```bash
python3 .agents/skills/skill-creator/scripts/run_loop.py --help
```

Itera entre evaluación y mejora automática de la descripción del skill, guardando historial y métricas.

### Agregar resultados de benchmark

```bash
python3 .agents/skills/skill-creator/scripts/aggregate_benchmark.py <benchmark_dir>
```

Resume medias, desviaciones y diferencias entre configuraciones como `with_skill` y `without_skill`.

### Empaquetar un skill

```bash
python3 .agents/skills/skill-creator/scripts/package_skill.py .agents/skills/skill-creator
```

Genera un paquete distribuible del skill, excluyendo archivos no deseados como `__pycache__`, `node_modules` o `evals/` en raíz.

## Evaluación y revisión visual

El directorio [`eval-viewer`](/Users/persualia/Projects/ad-builder/.agents/skills/skill-creator/eval-viewer) contiene:

- [`viewer.html`](/Users/persualia/Projects/ad-builder/.agents/skills/skill-creator/eval-viewer/viewer.html): interfaz local para inspeccionar prompts, outputs, métricas y resultados.
- `generate_review.py`: generador de datos o artefactos de revisión para alimentar el visor.

El skill también define esquemas JSON documentados en [`.agents/skills/skill-creator/references/schemas.md`](/Users/persualia/Projects/ad-builder/.agents/skills/skill-creator/references/schemas.md), incluyendo:

- `evals.json`
- `grading.json`
- `metrics.json`
- `timing.json`
- `history.json`

## Notas del repositorio

- [`.gitignore`](/Users/persualia/Projects/ad-builder/.gitignore) ignora `output/`.
- El repositorio no expone todavía una app completa en `src/`; el foco actual es el skill y sus utilidades de evaluación.
- Hay cambios borrados en el working tree fuera de este `README`; no los he tocado.

## Próximos pasos razonables

- Añadir un `requirements.txt` o `pyproject.toml` para fijar dependencias.
- Documentar un ejemplo completo de `evals.json`.
- Añadir un script de arranque para abrir o generar automáticamente el visor de revisión.
