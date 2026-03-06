# 📚 Bootcamp AI Engineer — Notas de Estudio

> Curso de [Código Facilito](https://codigofacilito.com) · Instructor: Ramsés Alejandro Camas Nájera  
> 16 clases · 4 módulos · 1 proyecto transversal

---

## 🗺️ Estructura del bootcamp

| Módulo | Tema |
|--------|------|
| I | Setup, tokens, prompts, outputs estructurados |
| II | RAG serio y memoria externa |
| III | ReAct y multiagentes |
| IV | Evaluación, optimización, seguridad y despliegue |

**Proyecto transversal:** *DocOps Agent* — copiloto empresarial que procesa documentos privados y asiste en tareas operativas.

---

## 🧱 El stack del AI Engineer (6 capas)

```
Capa 6 → Producción         (observabilidad, seguridad, despliegue)
Capa 5 → Evaluación         (métricas, testing, guardrails)
Capa 4 → Agentes            (ReAct, multiagentes, herramientas)
Capa 3 → Datos y contexto   (RAG, vector stores, memoria)
Capa 2 → Orquestación       (LangChain, LangGraph, DSPy)
Capa 1 → Modelos            (GPT, Claude, Llama, Gemini…)
```

---

## 📖 Clase 01 — Setup profesional y cliente LLM

### ¿Qué es un AI Engineer?

No es un Data Scientist (no entrena modelos desde cero), ni un ML Engineer (no optimiza pipelines de entrenamiento), ni un "prompt engineer" (eso es una habilidad, no un rol).

> **Un AI Engineer diseña, construye y despliega productos que integran modelos de IA en sistemas reales.** Piensa en: pipelines de RAG, agentes autónomos, copilotos empresariales. Es el puente entre los modelos y los usuarios finales.

---

### Conceptos clave

**Tokens**
- Los LLMs no procesan palabras sino *tokens* (~3-4 caracteres en inglés, variable en español).
- Pagas por tokens consumidos (input + output). La ventana de contexto también se mide en tokens.
- **Optimizar tokens = optimizar costos y rendimiento.**

**Estructura de mensajes** — la unidad fundamental de interacción con cualquier LLM:
```
system    → define el comportamiento del modelo
user      → la petición del usuario
assistant → la respuesta del modelo
```

**Proveedores principales**

| Proveedor | Modelos |
|-----------|---------|
| OpenAI | GPT-4o, o1, GPT-5-nano |
| Anthropic | Claude Sonnet, Opus, Haiku |
| Google | Gemini 3 Flash, Gemini 3 Pro, Gemma |
| Otros | Llama, Mistral, Qwen, DeepSeek |

Un buen AI Engineer puede cambiar de proveedor sin reescribir todo el sistema.

---

### Demo vs Producto

| Demo | Producto |
|------|----------|
| Script de 20 líneas | Maneja errores gracefully |
| | Logging de cada interacción |
| | Control de costos y latencia |
| | Validación de respuestas |
| | Reproducible y desplegable |
| | Con pruebas |

---

### Estructura del proyecto

```
ai-engineer-bootcamp/
├── core/
│   ├── config.py       ← configuración centralizada
│   ├── llm_client.py   ← cliente LLM reutilizable
│   └── logger.py       ← sistema de logging
├── .env                ← secrets (NUNCA al repo)
├── .env.example        ← referencia sin valores reales
├── requirements.txt
└── main.py             ← punto de entrada / CLI
```

`core/` contiene la lógica reutilizable. Cada módulo futuro (`rag/`, `agents/`, `mcp/`) se conectará con `core/`.

---

### Decisiones de diseño

**`core/config.py`**
- Usa `@dataclass(frozen=True)` para hacer la configuración inmutable en runtime.
- Usa `@lru_cache(maxsize=1)` para leer el `.env` una sola vez (patrón singleton).
- Falla explícitamente al arrancar si falta una key crítica — mejor un error claro que un fallo críptico más tarde.
- Patrón: `leer de .env → fallback a valor por defecto → fallar si falta algo crítico`.

**`core/logger.py`**
- Usa el módulo estándar `logging` + `rich` para formato visual en terminal.
- Configura el nivel vía `LOG_LEVEL` en `.env` (DEBUG / INFO / WARNING / ERROR).
- Evita configurar el logger más de una vez chequeando si ya hay handlers registrados.

**`core/llm_client.py`**
- Abstrae el proveedor: el resto del código nunca habla directamente con el SDK de Gemini (u otro).
- El método `chat()` siempre retorna la misma estructura: `{ "response": str, "metadata": dict }`.
- Registra automáticamente tokens, latencia y costo estimado en cada llamada.
- `_extract_usage()` maneja los distintos nombres de campos entre SDKs (`prompt_tokens` vs `prompt_token_count`, etc.) para que agregar un proveedor nuevo sea fácil.

**¿Por qué `logging` y no `print()`?**

| `print()` | `logging` |
|-----------|-----------|
| Desaparece al cerrar la terminal | Persiste y puede escribirse a archivo |
| Sin niveles de severidad | DEBUG / INFO / WARNING / ERROR |
| Sin timestamps | Timestamps automáticos |
| Sin contexto estructurado | Contexto configurable por módulo |

> En IA, la trazabilidad es crítica: si un agente toma una mala decisión, necesitas la traza completa.

---

### Flujo de `main.py`

```
get_settings()
     ↓
setup_logger(log_level)
     ↓
input() ← mensaje del usuario por terminal
     ↓
LLMClient(provider, model, temperature)
     ↓
client.chat([system_msg, user_msg])
     ↓
imprimir result["response"]
imprimir result["metadata"] (provider, model, tokens, latencia, costo)
```

El `metadata` que devuelve `chat()` contiene:
```python
{
    "provider": str,
    "model": str,
    "temperature": float,
    "usage": {
        "prompt_tokens": int,
        "completion_tokens": int,
        "total_tokens": int,
    },
    "latency_ms": float,
    "estimated_cost_usd": float,
}
```

---

### Variables de entorno (`.env`)

```env
GEMINI_API_KEY=...
LLM_PROVIDER=gemini
LLM_MODEL=gemini-3-flash-preview
LLM_TEMPERATURE=0.2
LOG_LEVEL=INFO
INPUT_COST_PER_1M_TOKENS=0.0
OUTPUT_COST_PER_1M_TOKENS=0.0
```

---

### Setup inicial

```bash
git clone <repo>
cd ai-engineer-bootcamp
python -m venv .venv
source .venv/bin/activate       # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env            # completar con tus keys
python main.py
```

---

### Buenas prácticas (día 1)

- `.env` siempre en `.gitignore`. Siempre.
- Nunca hardcodear API keys en el código.
- Siempre medir tokens y latencia.
- Modularizar: cada archivo hace UNA cosa bien.
- Versionar los prompts en Git — son código.

> *"Si no es reproducible, no es ingeniería."*

*Recurso útil: [What is an AI Engineer? — Swyx](https://www.latent.space/p/ai-engineer)*