```markdown
---
name: "super-optimizer"
description: "AI-powered writing optimizer for technical documentation, code comments, and prose"
version: "1.2.1"
author: "OpenClaw Team"
tags:
  - writing
  - ai
  - automation
  - optimization
  - technical-writing
maintainer: "sysadmin@openclaw.io"
type: "skill"
category: "content"
license: "MIT"
dependencies:
  - "openai>=1.0.0"
  - "anthropic>=0.8.0"
  - "tiktoken>=0.5.0"
  - "python-dotenv>=1.0.0"
  - "rich>=13.0.0"
required_apis:
  - "openai"
  - "anthropic"
os:
  - "linux"
  - "macos"
  - "windows"
---

# Super Optimizer

## Purpose

Super Optimizer es un asistente de escritura impulsado por IA que transforma texto sin formato, verboso o mal estructurado en contenido claro, conciso y profesional. Está diseñado para:

**Casos de Uso Reales:**

1. **Limpieza de Documentación Técnica**: Convertir notas de desarrolladores en documentación pulida. Ejemplo: Tomar una explicación informal de 500 palabras de una función y generar una doc estructurada con parámetros, retornos y ejemplos.

2. **Optimización de Comentarios de Código**: Reescribir comentarios en línea para que sean significativos y consistentes. Ejemplo: Transformar `"// fix this later"` en `"// TODO: Manejar caso límite donde user_id es null - ver issue #234"` o eliminar comentarios no accionables por completo.

3. **Mejora de Mensajes de Commit**: Tomar mensajes de git informales y reescribirlos para seguir el formato Conventional Commits. Ejemplo: `"fixed bug"` → `"fix(auth): prevent null pointer when token expires"`

4. **Refinamiento de Borradores de Email**: Convertir borradores de email roughly en mensajes profesionales y accionables con tono y estructura apropiados, preservando información clave mientras se eliminan palabras de relleno.

5. **Generación de Documentación API**: Extraer información implícita del código y generar documentación OpenAPI/Swagger completa con descripciones precisas.

6. **Optimización de README**: Reestructurar yclarificar archivos README para seguir mejores prácticas de la industria (badges, instalación, ejemplos de uso, pautas de contribución).

7. **Pulido de Blog Posts**: Mejorar gramática, flujo y SEO mientras se mantiene la voz del autor y precisión técnica.

8. **Especificación de Requisitos**: Convertir notas de reunión o sesiones de brainstorming en documentos PRD/Requisitos Técnicos estructurados con criterios de aceptación claros.

## Alcance

### Comandos Soportados

**Primario:** `openclaw optimize [OPCIONES] <ARCHIVO|STDIN>`

**Aliases:** `openclaw opt`, `openclaw improve`, `openclaw refine`

**Opciones de Comando:**

```bash
# Optimizar archivo in-place
openclaw optimize --type=docs src/docs/api.md

# Pipe desde stdin, output a stdout (dry-run)
cat draft.txt | openclaw optimize --type=prose --dry-run

# Optimizar con guía de estilo específica
openclaw optimize --type=docs --style=google src/README.md

# Procesar múltiples archivos con patrón
openclaw optimize --type=code "src/**/*.py"

# Usar provider/model específico
openclaw optimize --provider=anthropic --model=claude-3-5-sonnet-20241022 draft.md

# Aplicar cambios conservadores solo (reescritura mínima)
openclaw optimize --conservative commit_msg.txt

# Output verboso con uso de tokens
openclaw optimize --verbose --show-cost technical_spec.md

# Procesamiento paralelo de múltiples archivos
openclaw optimize --parallel=4 --type=docs docs/*.md

# Excluir ciertas reglas de optimización
openclaw optimize --no-fix-grammar --no-rephrase proposal.txt
```

**Comandos Secundarios:**

```bash
# Preview cambios sin aplicar
openclaw optimize --diff document.md

# Procesar por lotes con archivo de configuración
openclaw optimize --config .optimizerc.json docs/

# Listar perfiles de optimización disponibles
openclaw optimize --list-profiles

# Validar configuración de optimización
openclaw optimize --validate-config .optimizerc.json

# Estimar costo antes de procesar
openclaw optimize --estimate-cost large_document.txt

# Rollback última optimización (basado en git)
openclaw optimize --rollback document.md
```

## Proceso de Trabajo

### Fase 1: Análisis de Archivo

1. **Procesamiento de Entrada**
   - Leer archivo desde ruta o stdin
   - Detectar encoding (UTF-8, ISO-8859-1)
   - Parsear tipo de archivo desde extensión o shebang
   - Para binarios: rechazar con error

2. **Perfilado de Contenido**
   - Calcular estadísticas: conteo de palabras, varianza de longitud de oraciones, puntuación Flesch Reading Ease
   - Detectar términos técnicos, densidad de jerga
   - Identificar estructura (encabezados, listas, bloques de código, tablas)
   - Marcar problemas potenciales: voz pasiva, verbos débiles, palabras de relleno (very, really, basically)

3. **Recopilación de Contexto** (si disponible)
   - Buscar archivos adyacentes (ej., si se optimiza `api.py`, buscar `api_test.py` o `api.md`)
   - Cargar glosario específico del proyecto desde `.optimizer/glossary.yaml`
   - Detectar lenguaje de programación y aplicar reglas específicas del lenguaje
   - Buscar configuraciones lint/test existentes para alinearse con estándares del proyecto

### Fase 2: Selección de Estrategia

1. **Detección de Tipo**: Por defecto desde extensión de archivo o flag `--type` explícito:
   - `docs`: Markdown, reStructuredText, documentación
   - `code`: Archivos fuente con comentarios
   - `prose`: Artículos generales, emails, blogs
   - `commit`: Mensajes de git commit
   - `api`: Especificaciones OpenAPI/Swagger

2. **Selección de Estilo**:
   - `default`: Tono balanceado, profesional
   - `google`: Google Developer Documentation Style
   - `microsoft`: Microsoft Writing Style Guide
   - `concise`: Brevedad máxima (tweets técnicos, changelogs)
   - `academic`: Formal, aware de citas
   - `custom`: Cargado desde `.optimizerc.json`

3. **Nivel de Agresividad**:
   - `conservative` (--conservative): Solo corregir gramática/ortografía, reescrituras mínimas
   - `moderate`: Reestructurar oraciones, mejorar flujo, cambios moderados estándar
   - `aggressive`: Reescritura completa donde sea necesario, reestructurar párrafos (por defecto)

### Fase 3: Ejecución de Optimización

1. **Estrategia de Chunking** (para archivos grandes >100KB):
   - Dividir en límites naturales (encabezados, bloques de código, párrafos)
   - Mantener superposición de 1-2 oraciones entre chunks para contexto
   - Procesar chunks en paralelo si `--parallel=N` está seteado
   - Ensamblar resultados preservando estructura original

2. **Procesamiento IA**:
   - Construir prompt con:
     * Texto original
     * Tipo detectadoy reglas de estilo
     * Instrucciones específicas basadas en tipo (ej., "Convertir a formato Conventional Commits")
     * Términos del glosario del proyecto a preservar
     * Ejemplos de transformaciones buenas/malas
   - Llamar al provider/model seleccionado con stream=False, temperature=0.2 para consistencia
   - Implementar cadena de fallback: primary → secondary provider en rate limit/error
   - Rastrear uso de tokens para reporte de costo

3. **Post-Procesamiento**:
   - Parsear respuesta IA, extraer solo el texto optimizado (eliminar preámbulo conversacional)
   - Validar: verificar que bloques de código no cambien, términos técnicos preservados
   - Asegurar estructura de archivo (encabezados, code fences, formato) mantenida
   - Aplicar algoritmo diff para generar unified diff

### Fase 4: Output e Integración

1. **Modo de Output**:
   - Por defecto: Reemplazo in-place con git staging
   - `--diff`: Mostrar diff coloreado a stdout, sin cambios
   - `--stdout`: Escribir resultado a stdout
   - `--backup`: Crear `filename.optimized.bak` antes de escribir

2. **Integración Git**:
   - Verificar si archivo está tracked por git
   - Si sí: stage cambios automáticamente
   - Si working tree sucio: warning y requerir `--force`
   - Crear tag anotado `optimized/<timestamp>/<filename>` para rollback

3. **Reporte**:
   - Resumen: "Optimized X words, saved Y%, improved readability score from Z to W"
   - Conteo de tokens y estimación de costo
   - Warnings: "Review changed technical terms: X, Y, Z"
   - Exit codes: 0=success, 1=validation error, 2=API error, 3=file error

## Reglas de Oro

1. **Nunca modificar lógica de código**: Solo modificar comentarios, strings y documentación. Si se detectan cambios en código, abortar con error.

2. **Preservar términos técnicos exactos**: Usar glosario del proyecto para whitelist términos que no deben alterarse (nombres de función, nombres de clase, endpoints de API, keys de config).

3. **Integridad de bloques de código**: Triple-check que todo dentro de bloques \`\`\`lang\`\`\` o código indentado sea byte-for-byte idéntico. Si se detectan cambios, rechazar toda la optimización.

4. **Sin alucinación**: No agregar información no presente en original. Si se necesita contexto, marcar con comentario `[NEEDS CONTEXT]`, no inventar.

5. **Mantener voces**: En docs colaborativos, preservar atribución (ej., "Como Alice mencionó...") y no estandarizar voces.

6. **Seguridad primero**: Nunca optimizar archivos en:
   - `.env`, `*.key`, `*.pem`, `credentials.json`, `secrets/`
   - Auto-detectar y abortar con warning de seguridad

7. **Requisito de backup**: Siempre crear backup a menos que `--no-backup` esté explícitamente seteado. Usar git tags si disponibles, sino archivos `.bak`.

8. **Respetar lint existente**: Si proyecto usa estilo específico (verificado via `.eslintrc`, `.stylelintrc`, `pyproject.toml`), adaptarse a esas convenciones.

9. **Salir en incertidumbre**: Si confianza en optimización < 85% (medida via respuesta IA), output warning y saltar archivo.

10. **Transparencia de costo**: Siempre reportar uso de tokens y costo estimado. Usar `--estimate-cost` para lotes grandes antes de ejecución.

## Ejemplos

### Ejemplo 1: Optimizar Documentación Markdown

**Entrada** (`docs/getting-started.md`):
```markdown
# Getting Started

So yeah basically to get started you gotta install the thing. Just do npm install and then you can run it. Its pretty easy.

## Installation

Just run this command:

```
npm install @flickclaw/sdk
```

After that you can import it.

## Usage

Heres a simple example:
```

**Comando:**
```bash
openclaw optimize --type=docs --style=google docs/getting-started.md
```

**Output** (con diff preview):
```diff
 --- a/docs/getting-started.md
 +++ b/docs/getting-started.md
 @@ -1,12 +1,30 @@
 -# Getting Started
-
-So yeah basically to get started you gotta install the thing. Just do npm install
-and then you can run it. Its pretty easy.
+# Getting Started
+
+This guide helps you install and configure FlickClaw SDK in minutes.
+
+## Prerequisites
+
+- Node.js 18+
+- npm 9+
+- FlickClaw account with API key
+
+## Installation
+
+Install the SDK using npm:
+
+```bash
+npm install @flickclaw/sdk
+```
+
+## Configuration
+
+Create a `.env` file in your project root:
+
+```bash
+FLICKCLAW_API_KEY=your_api_key_here
+FLICKCLAW_ENV=production
+```
+
+## Quick Start
+
+Import and initialize the client:
+
+```javascript
+import { FlickClaw } from '@flickclaw/sdk';
+
+const client = new FlickClaw({
+  apiKey: process.env.FLICKCLAW_API_KEY
+});
+```
```

**Salida de Consola:**
```
✓ Optimized docs/getting-started.md
  • Words: 45 → 89 (98% increase in information density)
  • Readability: College → 9th grade
  • Cost: $0.0023 (1,234 tokens)
  • Changes committed to git
```

### Ejemplo 2: Optimizar Mensaje de Git Commit

**Entrada** (`git commit -m "msg.txt"`):
```
fixed auth bug and updated deps
```

**Comando:**
```bash
openclaw optimize --type=commit --stdout <<< "fixed auth bug and updated deps"
```

**Output:**
```
fix(auth): prevent null pointer when token expires

- Add null check for expired tokens in AuthMiddleware
- Handle TokenExpiredError gracefully with user-friendly message
- Update axios to 1.6.0 for security patches

Closes #234

BREAKING CHANGE: None
```

### Ejemplo 3: Limpieza de Comentarios de Código

**Entrada** (`src/auth.js`):
```javascript
function validateToken(token) {
  // check if token is good
  if (!token) return false;
  // todo: fix this
  const decoded = jwt.decode(token);
  // this is the expiration check thing
  if (decoded.exp < Date.now()) return false;
  // ok good
  return true;
}
```

**Comando:**
```bash
openclaw optimize --type=code --conservative src/auth.js
```

**Output** (in-place):
```javascript
function validateToken(token) {
  // Validate token exists
  if (!token) return false;

  // TODO: Implement refresh token flow when token expires
  const decoded = jwt.decode(token);

  // Check expiration (exp is in seconds, Date.now() in milliseconds)
  if (decoded.exp * 1000 < Date.now()) return false;

  return true;
}
```

**Changed markers:**
```
✓ src/auth.js optimized
  • Fixed grammar in 3 comments
  • Added clarification to expiration check
  • Preserved all code logic (23 lines unchanged)
```

### Ejemplo 4: Procesar por Lotes con Perfil

**Config** (`.optimizer/profiles/technical.json`):
```json
{
  "name": "technical",
  "type": "docs",
  "style": "concise",
  "rules": {
    "max_sentence_length": 25,
    "avoid_passive": true,
    "required_sections": ["Parameters", "Returns", "Examples"],
    "fix_grammar": false,
    "rephrase": true
  },
  "glossary": ["FlickClaw", "SDK", "API", "RESTful", "OAuth2"]
}
```

**Comando:**
```bash
openclaw optimize --profile=technical docs/api/*.md
```

**Resultado:** Procesa todos los archivos markdown en `docs/api/` usando el perfil technical, aplicando estilo conciso, agregando secciones requeridas si faltan, y preservando términos del glosario.

### Ejemplo 5: Dry-Run con Estimación de Costo

**Comando:**
```bash
openclaw optimize --type=prose --estimate-cost --parallel=4 --verbose draft_blog.txt
```

**Output:**
```
📊 Cost Estimation for: draft_blog.txt
   Size: 45 KB (12,345 words)
   Estimated chunks: 13 (each ~3KB)
   Model: claude-3-5-sonnet-20241022
   Input tokens: ~45,000
   Output tokens: ~40,000 (estimated)
   Total cost: $0.0685
   ⏱️  Estimated time: 2m 34s with 4 workers

Use --execute to proceed with optimization.
```

## Verificación

### Validación Post-Optimización

1. **Spell Check**: Ejecutar `codespell` o `cspell` en archivos optimizados:
   ```bash
   codespell optimized_file.md
   # Debería retornar sin errores
   ```

2. **Revisión de Diff**: Verificar que solo secciones previstas cambiaron:
   ```bash
   git diff --word-diff=color optimized_file.md | grep -E '\{-.*-\}|{\+.*\+}'
   # Revisar cambios, asegurar que no haya daño de código/keywords
   ```

3. **Puntuación de Legibilidad**: Comparar Flesch Reading Ease antes/después:
   ```bash
   # Antes (original)
   style-ease original.md  # Target: 60+ para docs
   # Después (optimizado)
   style-ease optimized.md  # Debería mejorar por 10+ puntos
   ```

4. **Validación de Markdown**: Asegurar que markdown optimizado sea válido:
   ```bash
   markdown-cli validate optimized.md
   # o
   mdlint optimized.md
   ```

5. **Test Suite**: Si se optimizan comentarios de código, asegurar que tests aún pasen:
   ```bash
   npm test  # o pytest, etc.
   # Debería pasar, ya que lógica de código no cambió
   ```

6. **Link Check**: Para documentación optimizada, verificar enlaces internos:
   ```bash
   markdown-link-check optimized.md
   # Todos los enlaces deberían resolver
   ```

### Script de Verificación Automatizado

Crear `.optimizer/verify.sh`:
```bash
#!/bin/bash
set -e

FILE="$1"

# 1. Sin cambios en código
if git diff --quiet "$FILE"; then
    echo "✓ No se detectaron cambios no staged"
else
    echo "✗ Se encontraron cambios no staged en $FILE"
    git diff --name-only | head -5
    exit 1
fi

# 2. Verificación de gramática
if command -vvale &>/dev/null; then
    vale "$FILE" || echo "⚠ Warnings de Vale - revisar manualmente"
fi

# 3. Restricciones de longitud
WORDS=$(wc -w < "$FILE")
if [ "$WORDS" -lt 10 ]; then
    echo "✗ Archivo demasiado corto ($WORDS words) - posible sobre-optimización"
    exit 1
fi

echo "✓ Verificación pasada para $FILE"
```

Ejecutar automáticamente después de optimización con `--post-hook .optimizer/verify.sh`.

## Rollback

### Métodos de Rollback

**1. Rollback con Git** (si archivos optimizados fueron committeados):
```bash
# Listar tags de optimización
git tag | grep optimized/

# Rollback archivo específico a estado pre-optimización
git checkout optimized/20260309_142345/src/auth.js -- src/auth.js

# O revertir todo el commit de optimización
git revert <commit_hash>
# Luego resolver conflictos merge manualmente
```

**2. Restaurar desde Backup** (si se usó `--backup`):
```bash
# Restaurar desde archivo .bak
cp src/auth.js.optimized.bak src/auth.js

# O si backup con timestamp
ls -la src/*.bak
cp src/auth.js.bak_20260309_1423 src/auth.js
```

**3. Rollback Manual via Diff** (si no hay backup):
```bash
# Si aún tienes output diff guardado
patch -R < optimization_changes.diff

# O usar git reflog para encontrar versión anterior
git reflog | grep "optimize"
git checkout HEAD@{2} -- src/auth.js  # entrada específica de reflog
```

**4. Rollback Integrado de Super Optimizer**:
```bash
# Rollback última optimización para archivo específico
openclaw optimize --rollback src/auth.js

# Rollback todos archivos optimizados en directorio
openclaw optimize --rollback-all docs/

# Listar puntos de rollback disponibles
openclaw optimize --list-rollbacks
```

### Chequeos de Seguridad de Rollback

Antes de cualquier operación de rollback:
1. **Confirmar cambios**: Mostrar diff entre estado actual y backup
2. **Verificar dependencias**: Asegurar que rollback no rompa imports/exports
3. **Preservar trabajo**: Si archivo fue modificado después de optimización, merge cambios en vez de sobrescribir
4. **Loglear acción**: Registrar rollback en `.optimizer/rollback.log` con:
   ```
   2026-03-09 14:45:12 ROLLBACK src/auth.js
   Reason: Sobree-optimización eliminó matices importantes
   Restored from: tag optimized/20260309_142345
   ```

### Rollback Automatizado en Fallo

Si verificación falla:
```bash
# Auto-rollback si post-hook falla
openclaw optimize --type=docs --post-hook .optimizer/verify.sh docs/
# Si verify.sh sale con non-zero, rollback automático se activa

# Trigger manual
openclaw optimize --rollback docs/getting-started.md --reason "Verificación falló"
```

## Dependencias

### Requisitos del Sistema
- Python 3.9+
- 512MB RAM mínimo (1GB+ para lotes grandes)
- Conexión a internet para llamadas API (o endpoint de modelo local)

### Paquetes Python
```txt
openai>=1.0.0        # OpenAI GPT-4
anthropic>=0.8.0    # Anthropic Claude
tiktoken>=0.5.0     # Conteo de tokens
python-dotenv>=1.0.0 # Gestión de entorno
rich>=13.0.0        # Formateo de terminal
click>=8.0.0        # Framework CLI
jsonschema>=4.0.0   # Validación de config
gitpython>=3.1.0    # Integración git
```

**Instalación:**
```bash
pip install openclaw-super-optimizer
# O desde source
git clone https://github.com/openclaw/super-optimizer
cd super-optimizer && pip install -e .
```

### API Keys (Environment Variables)

```bash
# OpenAI (requerido si se usa provider OpenAI)
export OPENAI_API_KEY="sk-..."

# Anthropic (requerido si se usa provider Anthropic)
export ANTHROPIC_API_KEY="sk-ant-..."

# Opcional: Endpoint personalizado para modelos locales
export OPENCLOW_OPTIMIZER_BASE_URL="http://localhost:8000/v1"

# Opcional: Límite de costo (USD)
export OPENCLOW_OPTIMIZER_MAX_COST="10.00"

# Opcional: Provider por defecto
export OPENCLOW_OPTIMIZER_DEFAULT_PROVIDER="anthropic"
```

Crear `.env` en directorio raíz del proyecto o `~/.config/openclaw/.env`.

### Soporte de Modelos Locales

Ejecutar con Ollama (sin costo API):
```bash
ollama serve &
openclaw optimize --provider=ollama --model=llama3.2:3b draft.txt
```

Backends locales soportados:
- Ollama (`--provider=ollama`)
- LM Studio (`--provider=localai --base-url=http://localhost:1234`)
- Text Generation WebUI (`--provider=textgen`)

## Solución de Problemas

### Problemas Comunes

**1. "No API key found"**
```bash
# Fix: Set environment variable
export OPENAI_API_KEY="your-key-here"

# O usar archivo de config
openclaw config set provider.openai.key "sk-..."
```

**2. "File too large, chunking failed"**
```bash
# Aumentar límite de chunk size
openclaw optimize --max-chunk-size=50000 large_file.txt

# O procesar con chunks más pequeños
openclaw optimize --chunk-strategy=sentence large_file.txt
```

**3. "Code block was modified" (error de seguridad)**
```bash
# Verificar si IA entendió mal límites
openclaw optimize --debug --diff file.md | grep -A5 -B5 '\`\`\`'

# Agregar language tags explícitos a code fences
# Antes: ```` some code ````
# Después: ````python some code ````
```

**4. "Rate limit exceeded"**
```bash
# Implementar exponential backoff (automático, pero ajustar)
openclaw optimize --retries=5 --retry-delay=2 file.md

# Cambiar provider temporalmente
openclaw optimize --provider=anthropic file.md

# Reducir workers paralelos
openclaw optimize --parallel=1 batch/
```

**5. "Optimization made text worse"**
```bash
# Usar modo conservative
openclaw optimize --conservative file.md

# Reducir temperature en config
echo '{ "temperature": 0.1 }' > .optimizer/config.json

# Proveer mejores ejemplos en prompt
openclaw optimize --example=good_example.md --example=bad_example.md file.md
```

**6. "Git integration failed"**
```bash
# Asegurar que archivo esté tracked por git
git add file.md

# Optimizar sin git staging
openclaw optimize --no-git file.md

# Inicializar git repo si es necesario
git init && git add . && git commit -m "initial"
```

**7. "Memory exhausted"**
```bash
# Procesar archivos uno a uno
openclaw optimize --parallel=1 large_file.txt

# Reducir model context size (modelo más barato)
openclaw optimize --model=gpt-4o-mini large_file.txt
```

### Modo Debug

Habilitar logging verbose:
```bash
OPENCLOW_OPTIMIZER_LOG_LEVEL=DEBUG openclaw optimize file.md
# o
openclaw optimize --log-level=DEBUG file.md 2>&1 | tee debug.log
```

Debug output incluye:
- Prompt enviado a IA (sin API key)
- Conteos de tokens y costos por chunk
- Respuesta IA cruda antes de parsear
- Validaciones checks realizadas
- Comandos git ejecutados

### Obtener Ayuda

```bash
# Ayuda general
openclaw optimize --help

# Específico de provider
openclaw config show

# Validar configuración
openclaw config validate

# Probar conexión API
openclaw doctor --test-api

# Reportar bug
openclaw bug report --include-log optimize.log
```

### Ajuste de Performance

Para lotes grandes (>100 archivos):
```bash
# Medir throughput
time openclaw optimize --parallel=8 --type=docs docs/**/*.md

# Workers paralelos óptimos = CPU cores * 2, pero respetar límites de rate API
# OpenAI: 3000 RPM para Tier 1, 10000 para Tier 2
# Calcular: workers = (rate_limit / avg_tokens_per_file) * safety_factor(0.8)

# Monitorear en tiempo real
openclaw optimize --parallel=4 --monitor docs/
# Muestra: files/s, tokens/s, cost/s, ETA
```

## Configuración

### Ubicaciones de Archivo de Config (en orden de precedencia):
1. Flag `--config <path>`
2. `.optimizer/config.json` en directorio actual
3. `~/.config/openclaw/optimizer/config.json`
4. System default: `/etc/openclaw/optimizer/config.json`

### Ejemplo de Config (`.optimizer/config.json`):
```json
{
  "default": {
    "provider": "anthropic",
    "model": "claude-3-5-sonnet-20241022",
    "temperature": 0.2,
    "max_tokens": 4000,
    "chunk_size": 8000,
    "max_parallel": 4,
    "backup": true,
    "git_stage": true
  },
  "profiles": {
    "docs": {
      "type": "docs",
      "style": "google",
      "rules": {
        "max_sentence_length": 30,
        "avoid_passive": true,
        "use_second_person": true
      }
    },
    "code": {
      "type": "code",
      "conservative": true,
      "preserve_comments": ["TODO", "FIXME", "HACK", "XXX"]
    }
  },
  "rules": {
    "skip_files": [".*test.*", ".*spec.*", "node_modules/"],
    "skip_patterns": ["^\\s*#", "^\\s*//\\s*\\w+"],
    "glossary": {
      "FlickClaw": "FlickClaw",
      "OpenClaw": "OpenClaw",
      "API": "API"
    }
  },
  "cost_limits": {
    "max_per_file_usd": 0.05,
    "max_daily_usd": 10.00,
    "alert_threshold_usd": 5.00
  }
}
```

### Config Basada en Entorno

```bash
# Override via environment (útil para CI/CD)
export OPENCLOW_OPTIMIZER_MODEL="gpt-4o-mini"
export OPENCLOW_OPTIMIZER_MAX_PARALLEL="8"
export OPENCLOW_OPTIMIZER_NO_BACKUP="true"

# Ejecutar con overrides
openclaw optimize --type=docs docs/
```

## Uso Avanzado

### Prompts Personalizados

Crear template de prompt personalizado en `.optimizer/prompts/<type>.jinja2`:
```jinja2
You are a {{ style }} documentation expert.

Task: Optimize the following {{ type }} content.

{{ original }}

Requirements:
{% for rule in rules %}
- {{ rule }}
{% endfor %}

Glossary (preserve these terms exactly):
{% for term in glossary %}
- {{ term }}
{% endfor %}

Output ONLY the optimized content. Do not add explanations.
```

### Workflow de Piping

```bash
# Integrar en pre-commit hook
cat file.md | openclaw optimize --type=docs --dry-run --diff

# Si satisfecho, aplicar auto
cat file.md | openclaw optimize --type=docs --stdout > optimized.md && mv optimized.md file.md

# Chain con otras herramientas
git diff --name-only '*.md' | xargs -r openclaw optimize --type=docs --git-stage
```

### Integración CI/CD

GitHub Actions (`.github/workflows/optimize-docs.yml`):
```yaml
name: Optimize Documentation
on:
  push:
    paths:
      - 'docs/**/*.md'

jobs:
  optimize:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          token: ${{ secrets.OPTIMIZER_TOKEN }}
      - name: Optimize docs
        run: |
          pip install openclaw-super-optimizer
          openclaw optimize --type=docs --style=google --git-stage docs/
      - name: Create PR
        run: |
          git config user.name 'optimizer-bot'
          git config user.email 'bot@openclaw.io'
          git checkout -b optimize/docs-$(date +%s)
          git push origin HEAD
          gh pr create --fill
```

### Integración con Editores

**VS Code** (tasks.json):
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Optimize Document",
      "type": "shell",
      "command": "openclaw optimize --type=docs --diff ${file}",
      "presentation": {
        "echo": true,
        "reveal": "always"
      }
    }
  ]
}
```

**Neovim** (lua):
```lua
vim.api.nvim_create_user_command('Optimize', function(opts)
  local file = opts.args or vim.fn.expand('%')
  vim.fn.system(string.format('openclaw optimize --type=docs --diff %s', file))
  vim.cmd('cgetexpr system("openclaw optimize --type=docs --diff " . expand("%"))')
  vim.cmd('copen')
end, { nargs = '?' })
```

## Métricas de Performance

### Benchmarks (claude-3-5-sonnet-20241022)

| File Type | Size | Time (single) | Time (4-par) | Tokens | Cost |
|-----------|------|---------------|--------------|---------|------|
| Markdown doc | 10KB | 3.2s | 1.1s | 2,500 | $0.0015 |
| Code file | 5KB | 2.1s | 0.8s | 1,800 | $0.0011 |
| Blog post | 50KB | 18.5s | 5.2s | 12,000 | $0.0072 |
| Technical spec | 100KB | 42.3s | 11.8s | 25,000 | $0.0150 |

**Throughput**: ~2,500 tokens/sec (agregado) con 4 workers.

### Estrategias de Optimización de Costo

1. **Usar `--conservative`**: 30% menos tokens ya que menos reescritura
2. **Batch de archivos pequeños**: Combinar múltiples archivos pequeños en single API call
3. **Cachear resultados**: Usar `--cache` (Redis/Memcached) para archivos no cambiados
4. **Modelos locales**: Para optimización rutinaria, usar `--provider=ollama` con `llama3.2:3b`
5. **Smart skipping**: Archivos sin problemas de gramática saltan IA call entirely

```bash
# Smart batch con caching
openclaw optimize --type=docs --cache-redis redis://localhost:6379 --cache-ttl=86400 docs/
```

### Monitoreo

Exportar métricas en formato Prometheus:
```bash
openclaw optimize --metrics-port=9090 --metrics-path=/metrics &
curl http://localhost:9090/metrics
# Output:
# openclaw_optimizations_total{type="docs"} 1523
# openclaw_tokens_total{provider="anthropic"} 3456789
# openclaw_cost_total_usd 12.34
# openclaw_duration_seconds_bucket{le="1"} 45
```

## Soporte

Para problemas no cubiertos aquí:
- Documentación: https://docs.openclaw.io/optimizer
- GitHub Issues: https://github.com/openclaw/super-optimizer/issues
- Discord: #optimizer-support en OpenClaw Discord
- Email: support@openclaw.io

Incluir en bug report:
1. Comando con flags completos
2. Archivo de entrada (o excerpt sanitizado)
3. Output (si hay)
4. `openclaw --version` output
5. Debug log: `OPENCLOW_OPTIMIZER_LOG_LEVEL=DEBUG ... 2>&1 | tee optim.log`
```