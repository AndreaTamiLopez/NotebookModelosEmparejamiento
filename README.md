# Modelos de Emparejamiento Semántico Proyecto–Política

Este notebook implementa y compara cinco enfoques progresivamente más avanzados para emparejar indicadores de producto asignados a la estructura prográmatica de los PATR y los indicadores de productos de la MGA utilizados en la formulación de los Planes de Desarrollo Territorial, utilizando técnicas de NLP basadas en embeddings y LLMs.

El objetivo es asignar, para cada subprograma PATR, los indicadores de producto de los PDT que están más alineadas conceptualmente.

---

## 📂 Datos de Entrada

El notebook trabaja con dos bases:

- `df_politicas` → Catálogo de la estructura prográmatica PATR
- `df_proyectos` → Plan operativo de los PDT 

Ambos datasets se cargan desde CSV y contienen campos textuales que serán transformados en embeddings.

---

# 🧠 Evolución de Modelos

El notebook está estructurado en cinco modelos, cada uno agregando mayor sofisticación.

---

## 🔹 Modelo 1 — Embeddings Básicos (Baseline)

**Técnica:**
- SentenceTransformer (MiniLM)
- Similitud coseno
- NearestNeighbors

**Flujo:**

Proyecto → Embedding  
Política → Embedding  
→ Top-K por similitud coseno  

**Características:**
- Rápido
- Determinístico
- Sin re-ranking
- Sin LLM

Es el baseline inicial del sistema.

---

## 🔹 Modelo 2 — Bi-Encoder + Cross-Encoder (Re-Ranking Local)

**Técnica:**
- Embeddings BGE / E5
- Recuperación Top-N
- Re-ranking con CrossEncoder

**Flujo:**

Proyecto → Embedding  
→ Recuperación amplia (Top-N)  
→ Re-ranking con CrossEncoder  
→ Top-K final  

**Ventajas:**
- Mejora precisión
- Totalmente local
- No depende de LLM externo

Introduce separación entre recuperación (recall) y precisión (ranking).

---

## 🔹 Modelo 3 — Embeddings + LLM (Ollama)

**Técnica:**
- Embeddings BGE
- Recuperación amplia
- Re-ranking con LLM local (Ollama)

**Flujo:**

Proyecto  
→ Recuperación Top-N  
→ LLM selecciona mejores candidatos  
→ Score combinado  

**Score final:**

```
final_score = w_llm * llm_score + w_bi * bi_score
```

Incluye:
- Fallback automático si el LLM falla
- Exportación a Excel
- Control de timeout

Este modelo maximiza precisión conceptual.

---

## 🔹 Modelo 4 — LLM Gated + Cache (Optimizado)

Versión optimizada del Modelo 3.

Agrega:

- Gating por confianza
- Llamado al LLM solo si embeddings no son confiables
- Cache persistente en disco
- Control avanzado de latencia

### Gating

Si:

```
top1_bi >= threshold
y
(top1_bi - top2_bi) >= margin
```

→ Se aceptan embeddings directamente  
→ No se llama al LLM  

Si no:

→ Se llama al LLM  

Reduce costo computacional y latencia.

---

## 🔹 Modelo 5 — Embeddings Only (Versión Limpia y Modular)

Versión final simplificada y estable.

**Técnica:**
- BGE-M3
- Similitud coseno
- Top-K directo

Sin LLM  
Sin CrossEncoder  
Sin gating  

Ideal para:

- Producción ligera
- Escenarios sin dependencia de servidor LLM
- Ejecución rápida en CPU

---

# 📊 Comparación Conceptual

| Modelo | Embeddings | Re-ranking | LLM | Gating | Cache |
|--------|------------|------------|-----|--------|-------|
| 1 | ✔ | ✖ | ✖ | ✖ | ✖ |
| 2 | ✔ | CrossEncoder | ✖ | ✖ | ✖ |
| 3 | ✔ | LLM | ✔ | ✖ | ✖ |
| 4 | ✔ | LLM | ✔ | ✔ | ✔ |
| 5 | ✔ | ✖ | ✖ | ✖ | ✖ |

---

# 📈 Output

Dependiendo del modelo, el DataFrame final puede incluir:

- matched_politica_text
- similarity_score
- bi_similarity_score
- llm_score
- final_score
- rank
- used_llm
- confident_by_embeddings
- device_used

---

# 🛠 Stack Tecnológico

- Python
- Pandas
- NumPy
- SentenceTransformers
- PyTorch
- scikit-learn
- Ollama (Modelos 3 y 4)
- XlsxWriter

---

# 🎯 Conclusión

El notebook muestra una evolución metodológica clara:

Baseline → Re-ranking local → LLM → Optimización por gating → Versión productiva ligera.

Permite elegir el equilibrio deseado entre:

- Velocidad
- Precisión semántica
- Costo computacional
- Dependencia de infraestructura LLM
