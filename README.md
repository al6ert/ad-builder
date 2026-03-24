# ad-builder

Repositorio para un flujo de trabajo de creatividad publicitaria en dos pasos:

1. `ad-builder`: genera anuncios en imagen a partir de un brief.
2. `ad-to-pencil`: convierte esas creatividades en un fichero Pencil `.pen` editable.

El repo no contiene una app web completa. El núcleo actual está en los skills locales de Claude dentro de [`.claude/skills/ad-builder`](/Users/persualia/Projects/ad-builder/.claude/skills/ad-builder) y [`.claude/skills/ad-to-pencil`](/Users/persualia/Projects/ad-builder/.claude/skills/ad-to-pencil).

## Qué hace cada skill

### `ad-builder`

Definido en [`.claude/skills/ad-builder/SKILL.md`](/Users/persualia/Projects/ad-builder/.claude/skills/ad-builder/SKILL.md).

Su objetivo es generar creatividades publicitarias profesionales usando Replicate, a partir de:

- título
- subtítulo
- CTA
- color principal o paleta
- restricciones de marca opcionales

El skill está pensado para producir siempre 3 variantes de ratio:

- `1:1`
- `4:5`
- `9:16`

El modelo documentado en el skill es `google/nano-banana-pro`, y el resultado esperado es un conjunto de imágenes descargadas en `output/<timestamp>/`.

### `ad-to-pencil`

Definido en [`.claude/skills/ad-to-pencil/SKILL.md`](/Users/persualia/Projects/ad-builder/.claude/skills/ad-to-pencil/SKILL.md).

Toma la salida de `ad-builder` y la reconstruye como un archivo Pencil editable:

- analiza las 3 imágenes del anuncio
- extrae estructura visual, color, tipografía y capas
- decide qué partes deben ser vectoriales y cuáles raster
- genera un `.pen` con 3 artboards

El objetivo es que el diseño final sea editable por capas, no una imagen plana incrustada.

## Flujo del proyecto

```text
Brief publicitario
  ↓
ad-builder
  ↓
output/<timestamp>/
  ├── ad_1x1.jpg
  ├── ad_4x5.jpg
  └── ad_9x16.jpg
  ↓
ad-to-pencil
  ↓
output/<timestamp>/ad.pen
```

## Estructura relevante

```text
.claude/
└── skills/
    ├── ad-builder/
    │   ├── SKILL.md
    │   └── evals/
    │       └── evals.json
    └── ad-to-pencil/
        ├── SKILL.md
        └── evals/
            └── evals.json

.agents/
└── skills/
    └── skill-creator/

output/                 # ignorado por git
fonts/
README.md
```

Nota: el repositorio también incluye `skill-creator` como herramienta auxiliar de evaluación y mejora de skills, pero no es el producto principal descrito por este `README`.

## Inputs y outputs

### Entrada de `ad-builder`

Un brief con contenido tipo:

- `Título`
- `Subtítulo`
- `CTA`
- `Color primario`
- `Colores adicionales` opcionales
- `Restricciones de marca` opcionales

### Salida de `ad-builder`

Una carpeta en `output/` con las 3 imágenes del anuncio en formatos de aspecto distintos.

### Entrada de `ad-to-pencil`

Una carpeta de `output/` que contenga las imágenes generadas del anuncio.

### Salida de `ad-to-pencil`

Un archivo `.pen` editable en la misma carpeta del anuncio, con 3 artboards:

- `1:1`
- `4:5`
- `9:16`

## Evals incluidas

El repo ya incluye casos de evaluación para ambos skills:

- [`.claude/skills/ad-builder/evals/evals.json`](/Users/persualia/Projects/ad-builder/.claude/skills/ad-builder/evals/evals.json)
- [`.claude/skills/ad-to-pencil/evals/evals.json`](/Users/persualia/Projects/ad-builder/.claude/skills/ad-to-pencil/evals/evals.json)

Los evals de `ad-builder` validan aspectos como:

- guardado de archivos en `output/`
- uso del modelo de Replicate
- respeto de restricciones de color
- presencia y tratamiento del CTA

Los evals de `ad-to-pencil` validan aspectos como:

- lectura de las 3 imágenes fuente
- creación de un `.pen`
- consistencia entre artboards
- uso del pipeline de reconstrucción editable

## Configuración local

El repositorio usa variables de entorno para secrets. El ejemplo está en [`.env.example`](/Users/persualia/Projects/ad-builder/.env.example).

Variables relevantes:

```bash
REPLICATE_API_TOKEN=your_replicate_token_here
```

Tu `.env` local no se versiona.

Si necesitas cargarlo en la sesión shell:

```bash
set -a
source .env
set +a
```

## Estado actual

Este repo documenta y versiona el workflow y los skills, no una aplicación SaaS desplegable.

Si el siguiente paso es productizarlo, lo razonable sería:

- extraer la lógica a un CLI o servicio reproducible
- formalizar dependencias y runtime
- documentar el pipeline de ejecución extremo a extremo
- añadir ejemplos de briefs y salidas esperadas
