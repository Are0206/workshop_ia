# Respuestas del workshop RAG
Escribe aquí tus respuestas con tus propias palabras. Pega tablas, resultados y enlaces a las capturas de `evidencias/`.

---

# Bloque 0 · Teoría

## T1 · Cuántos nodos viven en cada capa

_Evidencia:_ …

_Mi respuesta / análisis:_


## T2 · ef, k y el tamaño de la cola

_Evidencia:_ …

_Mi respuesta / análisis:_


## T3 · Inserción frente a búsqueda

_Evidencia:_ …

_Mi respuesta / análisis:_


## T4 · El invariante de FAISSVectorStore

_Evidencia:_ …

_Mi respuesta / análisis:_


## T5 · El costo de la fuerza bruta

_Evidencia:_ …

_Mi respuesta / análisis:_


---

# Bloque A · App RAG empresarial

## A1 · Ejecuta la app RAG de punta a punta

_Evidencia:_ 


_Mi respuesta / análisis:_

| # | Pregunta | Respuesta | Fuentes recuperadas |
|---|---|---|---|
| 1 | ¿Cuál es el precio del plan Enterprise de CloudSync Pro? | El plan Enterprise cuesta $49/mes por usuario con compromiso anual. | CloudSync Pro — Precios; Onboarding; Política de reembolsos |
| 2 | ¿Cuántos días pueden trabajar remotamente los empleados? | Los empleados pueden trabajar de forma remota hasta 3 días por semana. | Política de trabajo remoto; Onboarding; Política de reembolsos |
| 3 | ¿Qué pasa con los reembolsos mayores a $100? | Requieren la aprobación del supervisor; una vez aprobados, se procesan en 3 a 5 días hábiles al método de pago original. | Política de reembolsos; Política de trabajo remoto; CloudSync Pro — Precios |
| 4 | ¿Qué ocurre durante la primera semana de onboarding? | Se completan los módulos obligatorios de seguridad, cumplimiento y cultura de la empresa. | Onboarding; Política de trabajo remoto; Política de reembolsos |
| 5 | ¿Cuál es el horario de atención al cliente? *(no está en los documentos)* | No tengo esa información en el contexto proporcionado. | Política de trabajo remoto; CloudSync Pro — Precios; Política de reembolsos |

**Observaciones:**
- En las preguntas 1 a 4, la **primera fuente recuperada es siempre el documento correcto**.
- Siempre aparecen 3 fuentes porque el retriever usa k = 3 y solo hay 4 documentos; con un
  umbral de similitud de 0.3, casi todos lo superan. Las otras dos fuentes son ruido.
- En la pregunta 5 se recuperaron documentos, pero ninguno tenía la respuesta, y el modelo
  respondió que no tiene esa información en lugar de inventar, porque el prompt le indica
  usar únicamente el contexto.


## A2 · Tu propio corpus de documentos

_Evidencia:_ …

_Mi respuesta / análisis:_

| # | Pregunta | Respuesta | Fuentes recuperadas | ¿Correcta? |
|---|---|---|---|---|
| 1 | ¿Cuánto cuesta un latte con leche de avena? | Cuesta $10.000. | Menú de bebidas calientes; Menú de bebidas frías y postres | ✅ Sí ($8.000 + $2.000) |
| 2 | ¿Puedo ir el domingo a la sede Laureles? | No, los domingos solo abre la sede El Poblado; la sede Laureles está cerrada. | Horarios y sedes; Reservas y eventos | ✅ Sí |
| 3 | Soy celíaco, ¿qué postre puedo comer? | El brownie de chocolate, ya que es el único postre sin gluten. | Alergias e ingredientes; Programa de fidelización; Menú de bebidas frías y postres | ✅ Sí |
| 4 | ¿Con cuánta anticipación debo reservar para un evento? | Con al menos 5 días de anticipación. | Reservas y eventos; Normas para empleados | ✅ Sí |
| 5 | ¿Puedo pagar con Nequi? | Sí, puedes pagar con Nequi. | Medios de pago y facturación; Programa de fidelización | ✅ Sí |
| 6 | ¿Tienen wifi gratis? *(no está en los documentos)* | No tengo esa información en el contexto proporcionado. | Programa de fidelización; Domicilios | ✅ Sí (no inventó) |
| 7 | ¿Venden almuerzos? *(no está en los documentos)* | No tengo esa información en el contexto proporcionado. | Normas para empleados; Horarios y sedes; Menú de bebidas frías y postres | ✅ Sí (no inventó) |

**Observaciones:**
- En todas las preguntas con respuesta, la **primera fuente es el documento correcto**.
- La pregunta 1 combina dos datos del mismo documento (precio del latte + recargo por leche
  de avena), y el modelo hizo la suma correctamente.
- La pregunta 3 no menciona la palabra "gluten", pero el sistema recuperó el documento de
  alergias: la búsqueda por embeddings funciona por **significado**, no por palabras exactas.
- En las preguntas 6 y 7 se recuperaron documentos poco relacionados, pero el modelo no
  inventó una respuesta.

**Ajuste de chunk_size:** mis documentos son cortos (entre 245 y 332 caracteres). Con el valor
original de 400, cada documento quedaba en un solo chunk, que mezclaba varios temas (por
ejemplo, todos los precios del menú juntos). Lo bajé a 200, con 30 de overlap, para que cada
documento se divida en unos 2 fragmentos más específicos. Así el retriever trae el fragmento
que realmente responde la pregunta. El splitter corta por puntos, así que las oraciones no
quedan partidas.


## A3 · Ingeniería del prompt (antes y después)

_Evidencia:_ …

_Mi respuesta / análisis:_


## A4 · Muestra el score de similitud en las fuentes

_Evidencia:_ …

_Mi respuesta / análisis:_


---

# Bloque B · RAG sobre papers

## B1 · Ejecuta el RAG de papers y evalúa con el ground truth

_Evidencia:_ …

_Mi respuesta / análisis:_


## B2 · Experimentos con chunk_size, k y score_threshold

_Evidencia:_ …

_Mi respuesta / análisis:_


## B3 · Preguntas trampa: ¿el RAG admite que no sabe?

_Evidencia:_ …

_Mi respuesta / análisis:_


## B4 · Preguntas en español sobre papers en inglés

_Evidencia:_ …

_Mi respuesta / análisis:_


## B5 · Escribe tu propio quiz con ground truth

_Evidencia:_ …

_Mi respuesta / análisis:_


---

# Reflexión

## R1 · Reflexión final con tus datos

_Evidencia:_ …

_Mi respuesta / análisis:_


---

# Retos

## C1 · HNSW contra fuerza bruta con FAISS

_Evidencia:_ …

_Mi respuesta / análisis:_


## C2 · Un tercer paper: el RAG que explica HNSW

_Evidencia:_ …

_Mi respuesta / análisis:_


## C3 · App Gradio para los papers con filtro por paper

_Evidencia:_ …

_Mi respuesta / análisis:_


## C4 · Evaluación automática de la recuperación

_Evidencia:_ …

_Mi respuesta / análisis:_


## C5 · MMR frente a similitud pura

_Evidencia:_ …

_Mi respuesta / análisis:_


## C6 · Persistencia del índice en disco

_Evidencia:_ …

_Mi respuesta / análisis:_

