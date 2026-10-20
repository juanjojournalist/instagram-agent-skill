# Instagram Agent Skill: fork para RDN

Este es el fork [juanjojournalist/instagram-agent-skill](https://github.com/juanjojournalist/instagram-agent-skill), mantenido por juanjojournalist para el trabajo editorial de RDN. Incluye 13 habilidades de Claude para preparar y revisar contenido de Instagram, además de seis scripts Python de ejecución local.

El proyecto original fue creado por **Jake Schincariol**. Este fork conserva sus créditos y la licencia MIT. Las instrucciones siguientes apuntan al fork; el enlace al proyecto original aparece al final como reconocimiento de autoría.

**Las habilidades preparan contenido; no publican en Instagram.** La revisión y la publicación quedan a cargo del usuario. No se necesitan credenciales de Instagram ni claves de API para ejecutar los scripts Python. Claude Code requiere su propia instalación y acceso al servicio.

## 1. Instalar el plugin en Claude Code

Se necesita Claude Code con soporte de plugins y Git disponible. Estos comandos se escriben **dentro de una sesión de Claude Code**, no en la terminal del sistema:

```text
/plugin marketplace add juanjojournalist/instagram-agent-skill
/plugin install instagram-agent@instagram-agent-rdn
```

En el panel de instalación, confirmar el alcance deseado. Si Claude Code solicita recargar los plugins o reiniciar la sesión, seguir esa indicación.

El marketplace de este fork se llama `instagram-agent-rdn`. El plugin mantiene el nombre `instagram-agent`; por eso se utiliza el identificador completo `instagram-agent@instagram-agent-rdn`. Así se distingue del marketplace original, llamado `instagram-agent-skill`. El campo `source: "./"` carga el plugin desde la raíz de este mismo repositorio.

### Comprobar el origen del plugin

1. Abrir `/plugin` y revisar el marketplace `instagram-agent-rdn`: su origen debe ser `juanjojournalist/instagram-agent-skill` (o la URL HTTPS equivalente), y su página de inicio debe dirigir a este fork.
2. En los plugins instalados, comprobar que `instagram-agent@instagram-agent-rdn` está habilitado.
3. Probar una habilidad con su nombre completo:

```text
/instagram-agent:ig-reel Prepará un guion periodístico en español de Paraguay a partir de los datos que te proporcionaré, sin agregar información.
```

La aparición del comando por sí sola no demuestra el origen: verificar también los dos primeros puntos. Para los plugins, Claude Code utiliza el prefijo `/instagram-agent:`; los nombres cortos de la tabla siguiente corresponden a las habilidades instaladas manualmente.

### Si ya estaba instalada la versión original

En `/plugin`, revisar el origen antes de hacer cambios. Si está habilitado `instagram-agent@instagram-agent-skill`, deshabilitarlo o desinstalarlo antes de activar el fork: ambos comparten el prefijo `/instagram-agent:`. No es necesario eliminar otros marketplaces.

Si el fork se había registrado anteriormente con el nombre antiguo `instagram-agent-skill`, quitar ese registro desde la administración de marketplaces y volver a agregar `juanjojournalist/instagram-agent-skill`. Esto registra su nuevo nombre `instagram-agent-rdn`. Revisar también las copias manuales `ig-*` en `~/.claude/skills/` o `.claude/skills/`, para evitar utilizar por error una copia anterior.

Referencia de sintaxis: [documentación oficial de marketplaces de Claude Code](https://code.claude.com/docs/en/plugin-marketplaces).

## 2. Ejecutar las herramientas Python en forma local

Esta opción es independiente del plugin: **no requiere Claude Code**. Se necesita Git y Python 3. Los seis scripts usan la biblioteca estándar, procesan archivos locales y no necesitan instalar paquetes con `pip` ni conectarse a Instagram. La descarga inicial del repositorio sí requiere conexión.

Los ejemplos de terminal utilizan Bash en Linux, macOS o WSL. Clonar el fork y entrar en su raíz:

```bash
git clone https://github.com/juanjojournalist/instagram-agent-skill.git
cd instagram-agent-skill
python3 --version
```

### Comprobar el repositorio descargado

Desde la raíz del clon:

```bash
git remote get-url origin
git rev-parse --show-toplevel
git log -1 --oneline
```

El primer comando debe mostrar `https://github.com/juanjojournalist/instagram-agent-skill.git` o su equivalente SSH `git@github.com:juanjojournalist/instagram-agent-skill.git`. Si muestra `Jakeschincariol`, esa copia apunta al proyecto original. Clonar el fork en otra carpeta y trabajar allí; cambiar solamente el remoto no descarga los archivos del fork. Los otros comandos permiten identificar la carpeta y la revisión utilizadas.

Verificar también los metadatos de esa copia:

```bash
python3 - <<'PY'
import json
from pathlib import Path

root = Path.cwd()
market = json.loads((root / ".claude-plugin/marketplace.json").read_text(encoding="utf-8"))
plugin = json.loads((root / ".claude-plugin/plugin.json").read_text(encoding="utf-8"))
url = "https://github.com/juanjojournalist/instagram-agent-skill"
assert market["name"] == "instagram-agent-rdn", "Marketplace incorrecto"
entry = next(p for p in market["plugins"] if p["name"] == plugin["name"])
assert plugin["name"] == "instagram-agent", "Plugin incorrecto"
assert entry["source"] == "./", "El plugin debe cargarse desde este repositorio"
assert entry["homepage"] == plugin["homepage"] == plugin["repository"] == url, "Referencias incorrectas"
print("OK: los metadatos identifican el fork de juanjojournalist")
PY
```

Esto comprueba la coherencia de los metadatos locales; no verifica una instalación separada de Claude Code. Para esa instalación, utilizar la comprobación de la sección 1.

### Prueba rápida sin archivos de entrada

Ejecutar desde la raíz del repositorio:

```bash
python3 skills/ig-caption/caption.py --help
python3 skills/ig-human/detect.py --help
```

Si aparecen las opciones de cada programa, Python encuentra los scripts. Esta prueba comprueba su disponibilidad, no la calidad de un contenido ni la instalación del plugin.

### Comandos de trabajo

Crear primero los archivos de entrada en UTF-8 en la raíz del repositorio. Los nombres siguientes son ejemplos: `hooks.txt` contiene un gancho por línea; `guion.txt`, un bloque por línea; los demás `.txt`, el texto que se desea revisar.

```bash
# Puntuar los ganchos y estimar la duración del guion.
python3 skills/ig-reel/hookscore.py hooks.txt
python3 skills/ig-reel/beats.py guion.txt --target 30

# Revisar un copy y buscar las palabras clave indicadas.
python3 skills/ig-caption/caption.py caption.txt --keywords "Asunción,transporte público"

# Limpiar un borrador y comparar ambas versiones.
python3 skills/ig-human/humanize.py borrador.txt -o limpio.txt --report
python3 skills/ig-human/detect.py borrador.txt
python3 skills/ig-human/detect.py borrador.txt limpio.txt

# Ordenar una muestra de reels recogida por el usuario.
python3 skills/ig-viral/swipe.py capturas.tsv --out ./swipe.md
```

`capturas.tsv` debe tener columnas separadas por tabulaciones y una cabecera con `account`, `followers`, `median`, `views` y `hook`. Cada fila representa un reel. `median` es la mediana reciente de visualizaciones de esa cuenta; si no se dispone de ella, el script admite usar seguidores como referencia y lo indica. Ver [swipe.py](skills/ig-viral/swipe.py) para el formato de entrada.

`hookscore.py` devuelve código de salida 1 si ningún gancho alcanza 70 puntos. `caption.py` devuelve código de salida 1 cuando el resultado requiere revisión o corrección. `detect.py` también devuelve 1 cuando su resultado no es `PASS`. Consultar el informe antes de interpretar ese código como un problema de instalación. Cada script ofrece sus opciones con `--help`.

## 3. Alternativa: instalar las habilidades manualmente

Utilizar esta opción **en lugar del plugin** si se prefieren comandos cortos como `/ig-reel`. Desde la raíz de un clon del fork ya verificado:

```bash
mkdir -p ~/.claude/skills
cp -r skills/ig-* ~/.claude/skills/
```

Revisar primero si existen carpetas con esos nombres: la copia puede sobrescribir archivos. Para limitar la instalación a un proyecto, copiar las mismas carpetas a `<tu-proyecto>/.claude/skills/`. Conservar todas las carpetas `ig-*` y sus archivos auxiliares, porque algunos scripts consultan recursos de habilidades vecinas.

Las copias manuales no se actualizan al ejecutar `git pull`: hay que volver a copiarlas. Pegar un `SKILL.md` en un chat transmite sus instrucciones, pero no instala el plugin ni pone los scripts Python a disposición de ese chat.

## 4. Configurar la voz editorial

La plantilla está en [templates/voice.md](templates/voice.md). Si todavía no existe un perfil, copiarla sin sobrescribir uno anterior:

```bash
mkdir -p ~/.claude/instagram
cp -n templates/voice.md ~/.claude/instagram/voice.md
```

Completar `~/.claude/instagram/voice.md` con el tono de RDN, español de Paraguay, criterios de atribución y ejemplos propios. También se pueden proporcionar tres piezas de referencia para preparar ese perfil. Esta configuración orienta la redacción; no cambia los algoritmos ni adapta automáticamente los validadores al español.

## Las 13 habilidades

En el plugin, anteponer `instagram-agent:` al nombre: por ejemplo, `/instagram-agent:ig-caption`.

| Habilidad manual | Función |
| --- | --- |
| `/ig-reel` | Prepara ganchos, guion, textos en pantalla y tiempos, con [26 fórmulas](skills/ig-reel/hooks.json). |
| `/ig-viral` | Guía una investigación de reels y compara su rendimiento con la referencia de cada cuenta. |
| `/ig-caption` | Redacta y revisa el copy, incluida una vista aproximada de los primeros 125 caracteres. |
| `/ig-carousel` | Prepara portada, textos y piezas de carrusel de 1080 × 1350. |
| `/ig-story` | Propone secuencias de historias, stickers y recorridos de respuesta. |
| `/ig-profile` | Evalúa el perfil sobre 100 con una [rúbrica de 12 apartados](skills/ig-profile/rubric.json). |
| `/ig-plan` | Organiza el calendario semanal, formatos y cuentas de referencia. |
| `/ig-human` | Limpia caracteres y expresiones, y aplica cinco comprobaciones heurísticas. |
| `/ig-comment` | Prepara comentarios según el contenido de una publicación. |
| `/ig-reply` | Clasifica y redacta respuestas a comentarios recibidos. |
| `/ig-dm` | Prepara mensajes directos, propuestas de colaboración y seguimientos. |
| `/ig-repurpose` | Reutiliza materiales extensos en reels y carruseles. |
| `/ig-audit` | Revisa publicaciones anteriores y compara indicadores de rendimiento. |

## Alcance y límites de la evaluación

Las puntuaciones son orientativas. El proyecto conserva reglas, vocabulario y patrones centrados en inglés; sus resultados no equivalen a una validación periodística en español de Paraguay. La adaptación de `caption.py` y `detect.py` es un trabajo separado de estas correcciones de instalación.

El autor original informó una prueba sobre 74 ganchos de YouTube Shorts, no de Instagram Reels: AUC de 0,83 para distinguir ganchos deliberadamente débiles y de 0,56 para separar éxitos de resultados menores del mismo creador. Esas cifras corresponden a su evaluación original, no a una validación de este fork ni del contenido de RDN. El clasificador original pasó de reconocer fórmulas en el 8 % al 49 % de aquella muestra. Las puntuaciones no predicen visualizaciones ni viralidad.

`caption.py` utiliza una ventana aproximada de 125 caracteres, configurable con `--truncate`, y un límite de cinco hashtags definido en el código. Son parámetros de la herramienta; contrastarlos con el funcionamiento de Instagram al momento de publicar.

`humanize.py` normaliza caracteres invisibles y tipografía, sustituye términos del léxico [slop.json](skills/ig-human/slop.json) y reporta estructuras que requieren revisión. `detect.py` aplica cinco heurísticas locales: variación de longitud de frases, especificidad, vocabulario, tipografía y voz. No consulta detectores externos ni demuestra quién escribió un texto. La limpieza de caracteres tampoco acredita la eliminación de marcas de agua criptográficas.

Las habilidades no deben inventar cifras, fuentes ni resultados. Si falta un dato, debe solicitarse o marcarse como pendiente. `/ig-viral` orienta una revisión manual de fuentes y no proporciona un recolector automático. Los borradores de comentarios y mensajes directos también requieren revisión y envío por parte del usuario.

## Archivos principales

| Ruta | Contenido |
| --- | --- |
| `.claude-plugin/marketplace.json` | Marketplace del fork y origen local del plugin. |
| `.claude-plugin/plugin.json` | Identidad, autoría, repositorio y licencia del plugin. |
| `skills/ig-reel/hookscore.py` | Evaluación de ganchos. |
| `skills/ig-reel/beats.py` | Estimación de tiempos del guion. |
| `skills/ig-caption/caption.py` | Revisión de captions y vista de truncamiento. |
| `skills/ig-human/humanize.py` | Limpieza de texto. |
| `skills/ig-human/detect.py` | Panel de cinco heurísticas. |
| `skills/ig-viral/swipe.py` | Comparación de reels y clasificación de fórmulas. |
| `templates/voice.md` | Plantilla de voz editorial. |

## Créditos

Proyecto original de **Jake Schincariol**, [opusjake.ai](https://opusjake.ai): [Jakeschincariol/instagram-agent-skill](https://github.com/Jakeschincariol/instagram-agent-skill). Ese enlace identifica al autor y su trabajo original; para instalar este fork, utilizar las instrucciones de arriba.

Fork mantenido por [juanjojournalist](https://github.com/juanjojournalist), con orientación al trabajo de RDN.

Proyecto relacionado del autor original: [linkedin-agent-skill](https://github.com/Jakeschincariol/linkedin-agent-skill).

## Licencia

[MIT](LICENSE). Se conserva íntegro el aviso de copyright de 2026 de Jake Schincariol y el texto de la licencia original.
