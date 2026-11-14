# Evaluador de Trabajos

Agente de IA para calificar anteproyectos por lotes. Funciona sobre [Claude Code](https://claude.com/claude-code): aprende el criterio del profesor a partir de trabajos de ejemplo, aplica una rúbrica y genera un reporte de calificación por cada lote.

> **Estado del proyecto:** la estructura y las skills de terceros están listas. La skill propia que automatiza el flujo completo todavía no existe; por ahora cada paso se pide a Claude con las instrucciones de este documento.

## Cómo funciona

1. **Entrenamiento.** El agente lee trabajos buenos y malos ya calificados por el profesor y escribe dos archivos con las reglas generales que el profesor aplica.
2. **Rúbrica.** El agente lee la rúbrica en PDF que corresponde al tipo de trabajo.
3. **Calificación.** El agente evalúa cada PDF del lote con la rúbrica y los criterios, y escribe el reporte.

## Estructura de carpetas

```
agentCalificar/
├── entradas/                  Lotes a calificar: una carpeta por lote, con sus PDF
├── reporte/                   Salida: una carpeta por lote con el reporte de calificación
├── reglas/
│   ├── buenosTrabajos/        Ejemplos de trabajos buenos, con su nota
│   ├── malosTrabajos/         Ejemplos de trabajos malos, con su nota
│   ├── rubricas/              Rúbricas en PDF, una por tipo de evaluación
│   ├── criterios-buen-trabajo.md   Lo genera el agente en el entrenamiento
│   └── criterios-mal-trabajo.md    Lo genera el agente en el entrenamiento
└── skills/                    Herramientas del agente
```

## Instalación

### Requisitos

- **Claude Code** con una cuenta activa. Es lo único obligatorio.
- **Git**, para clonar el repositorio.
- **Python 3.11 o superior** (opcional). Solo hace falta para los scripts auxiliares de las skills: estadísticas, exportación a Excel y validación de rúbricas.
- **pdftotext** (opcional). Solo lo usa la skill `pdf-to-markdown`. Claude lee los PDF sin él.

En macOS, Git y Python se instalan juntos con:

```bash
xcode-select --install
```

### Pasos

1. Clona el repositorio:

```bash
git clone https://github.com/SamyuMT/agentCalificar.git
```

2. Abre la carpeta `agentCalificar` en Claude Code.

3. Si vas a usar los scripts de Python, instala sus dependencias:

```bash
pip3 install -r skills/homework-grader/scripts/requirements.txt
```

No se necesita ninguna clave de API. El script `skills/homework-grader/scripts/batch_score.py` pide una, pero el agente califica directamente desde Claude Code y no lo usa.

## Entrenar al agente

El entrenamiento se hace una vez y se repite cuando cambian los ejemplos.

1. Copia en `reglas/buenosTrabajos/` entre 3 y 10 trabajos que el profesor calificó bien.
2. Copia en `reglas/malosTrabajos/` entre 3 y 10 trabajos que el profesor calificó mal.
3. Indica la nota de cada trabajo. Sirve cualquiera de estas formas:
   - en el nombre del archivo, por ejemplo `anteproyecto-energia-solar_nota-18.pdf`;
   - en un archivo `notas.md` dentro de la carpeta, con una línea por trabajo y, si existen, los comentarios del profesor.
4. Copia las rúbricas en `reglas/rubricas/`, con un nombre que identifique cada una, por ejemplo `rubrica-anteproyecto-ingenieria.pdf`.
5. Pide a Claude:

```
Lee los trabajos de reglas/buenosTrabajos y reglas/malosTrabajos con sus notas,
y genera reglas/criterios-buen-trabajo.md y reglas/criterios-mal-trabajo.md
con las reglas generales que el profesor aplica al calificar.
```

6. **Revisa los dos archivos generados.** Son la interpretación que hace el agente del criterio del profesor. Corrige a mano lo que no coincida: el agente calificará con lo que quede escrito ahí.

Cuantos más comentarios del profesor acompañen a las notas, mejor capta el agente por qué un trabajo es bueno o malo.

## Calificar un lote

1. Crea una carpeta dentro de `entradas/` con el nombre del lote, por ejemplo `entradas/seccion-A-2026/`.
2. Copia ahí los PDF a calificar, uno por trabajo.
3. Pide a Claude:

```
Califica el lote entradas/seccion-A-2026 con la rúbrica
reglas/rubricas/rubrica-anteproyecto-ingenieria.pdf y los criterios de reglas/.
Guarda el reporte en reporte/seccion-A-2026/.
```

4. Abre el reporte en `reporte/seccion-A-2026/`.

### Revisión del profesor

Las notas del agente son una propuesta. El profesor debe revisarlas antes de entregarlas, sobre todo los trabajos cerca del límite de aprobación y los que el agente marque como dudosos. Si una nota no coincide con su criterio, conviene añadir ese trabajo como ejemplo y repetir el entrenamiento.

## Qué se publica en GitHub y qué no

El archivo `.gitignore` del repositorio ya aplica estas reglas.

| Se publica | No se publica |
|---|---|
| `README.md` y `.gitignore` | Los PDF de `entradas/`: son trabajos de estudiantes |
| La carpeta `skills/` completa, con sus licencias | Todo el contenido de `reporte/`: contiene notas de estudiantes |
| La estructura de carpetas vacía (archivos `.gitkeep`) | Los trabajos de `reglas/buenosTrabajos/` y `reglas/malosTrabajos/`, y sus `notas.md` |
| | Los PDF de `reglas/rubricas/` |
| | Los dos archivos `criterios-*.md` |
| | Claves de API, archivos `.env` y `.DS_Store` |

Motivos:

- **Trabajos, notas y reportes** llevan nombres y calificaciones de estudiantes. Publicarlos expone datos personales y obra ajena sin permiso.
- **Rúbricas** suelen pertenecer a la institución. Si la rúbrica es tuya y quieres compartirla, quita la línea `reglas/rubricas/*.pdf` del `.gitignore`.
- **Criterios generados** describen cómo califica un profesor concreto y pueden citar fragmentos de los trabajos de ejemplo. Si los revisas, no contienen datos de estudiantes y quieres publicarlos como ejemplo, quita la línea `reglas/criterios-*.md` del `.gitignore`.

Antes del primer `git push`, comprueba qué se va a subir:

```bash
git status
```

Si un archivo privado ya se subió, borrarlo en un commit posterior no lo elimina del historial.

## Skills incluidas

Las tres son de terceros, con licencia MIT. Cada carpeta conserva su archivo de licencia.

| Carpeta | Uso en este proyecto | Origen |
|---|---|---|
| `skills/homework-grader/` | Método de calificación por rúbrica, calibración contra el profesor y control de sesgo | [ChantillyAn/homework-grader](https://github.com/ChantillyAn/homework-grader) |
| `skills/scholar-evaluation/` | Marco de criterios académicos para la retroalimentación | [K-Dense-AI/scientific-agent-skills](https://github.com/K-Dense-AI/scientific-agent-skills) |
| `skills/research-feedback/` | Criterios de revisión de propuestas y planes de investigación; conversión de PDF a texto | [claesbackman/AI-research-feedback](https://github.com/claesbackman/AI-research-feedback) |

Límites conocidos:

- `scholar-evaluation` advierte que no debe usarse para decisiones con consecuencias. Aquí aporta criterios y retroalimentación; la nota sale de la rúbrica y de la revisión del profesor.
- `review-grant` y `review-pap` lanzan seis agentes por documento. Se usan como referencia de criterios, no para procesar lotes.

## Licencia

Falta elegir la licencia del proyecto. Las skills de terceros mantienen la suya (MIT).
