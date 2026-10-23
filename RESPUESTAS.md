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

## T2 · ef, k y el tamaño de la cola

1. **k = 10 con ef = 5:** falla porque `ef` es el tamaño de la cola de candidatos en la
   capa 0, y de esos candidatos se eligen los k resultados. Con ef = 5 solo hay 5
   candidatos, así que es imposible retornar 10 vecinos. Por eso siempre debe cumplirse
   **ef ≥ k**.

2. **Valor inicial:** la presentación recomienda empezar con **ef = 2·k** (por ejemplo,
   ef = 20 para k = 10) y **subirlo si el recall es insuficiente**. Para la mayoría de
   aplicaciones RAG, ef = 50–100 con k = 5–10 da más de 95 % de recall. Subirlo mejora
   el recall pero aumenta la latencia.

3. **Se puede cambiar `ef` en producción sin reconstruir el índice.** `ef` solo se usa al
   buscar, así que puede ajustarse en cada consulta. `ef_construction` se usa al
   **construir** el grafo (define qué aristas se crean al insertar cada vector), así que
   cambiarlo exige reconstruir el índice.

## T3 · Inserción frente a búsqueda

_Evidencia:_ …

_Mi respuesta / análisis:_

## T3 · Inserción frente a búsqueda

| | Inserción | Búsqueda |
|---|---|---|
| ¿Se sortea la capa? | **Sí.** Cada vector nuevo recibe al azar su capa máxima, con distribución exponencial. | **No.** Siempre se entra por la capa más alta desde el nodo de entrada fijo. |
| Valor de ef en las capas altas | **ef = 1**, desde la cima hasta la capa sorteada l. | **ef = 1**, desde la cima hasta la capa 1. |
| Parámetro de la fase principal | **ef_construction**, desde la capa l hasta la 0. | **ef**, solo en la capa 0. |
| ¿Modifica el índice? | **Sí.** Crea las aristas del nuevo nodo. | **No.** Solo lee el grafo. |

**Porqué de cada fila:**
- **Sorteo:** en la inserción hay que decidir en qué capas vivirá el nodo; esto es lo que
  forma la jerarquía. La búsqueda no agrega nada, así que no tiene nada que sortear.
- **ef = 1 arriba:** en las capas altas solo interesa llegar rápido a la zona correcta, no
  la precisión, así que basta con seguir al mejor vecino.
- **Fase principal:** al insertar, `ef_construction` decide con qué vecinos se conecta el
  nodo (la calidad del grafo); al buscar, `ef` decide cuántos candidatos se revisan antes
  de devolver los k mejores (el recall).
- **Modificación:** la inserción cambia la estructura del grafo; la búsqueda solo lo recorre.

**En el notebook 03:** `vectorstore.add_documents(...)` dispara la **inserción** (calcula
embeddings y agrega los vectores al índice), y `retriever.invoke(...)` dispara la
**búsqueda** (convierte la pregunta en vector y busca los k más cercanos).


## T4 · El invariante de FAISSVectorStore

_Evidencia:_ …

_Mi respuesta / análisis:_

## T4 · El invariante de FAISSVectorStore

| Posición FAISS | `self.documents` antes | `self.documents` después de borrar B |
|---|---|---|
| 0 | A | A |
| 1 | B | **C** |
| 2 | C | **D** |
| 3 | D | — (no existe) |

1. **¿Qué devuelve la búsqueda que antes daba [1, 2, 3]?** FAISS sigue teniendo los 4
   vectores, así que vuelve a devolver las posiciones [1, 2, 3], pero `self.documents` ya
   solo tiene 3 elementos:
   - posición 1 → **C** (debía ser B)
   - posición 2 → **D** (debía ser C)
   - posición 3 → **IndexError** (la lista solo llega hasta el índice 2)

2. **Errores silenciosos:** las posiciones 1 y 2 devuelven documentos equivocados (C y D)
   sin ningún aviso; el RAG respondería con información incorrecta sin que nadie lo note.
   **Excepción:** la posición 3 lanza `IndexError`, porque esa posición ya no existe en la
   lista.

3. **Relación con Qdrant:** `IndexFlatIP` de FAISS no tiene operación de borrado, y el
   invariante "posición i en FAISS = `self.documents[i]`" lo tiene que mantener nuestro
   propio código. Bases vectoriales como Qdrant guardan el vector y su documento juntos,
   con un ID estable, y manejan borrados y actualizaciones de forma nativa, así que este
   desalineamiento no puede ocurrir.


## T5 · El costo de la fuerza bruta

_Evidencia:_ …

_Mi respuesta / análisis:_

## T5 · El costo de la fuerza bruta

1. **Cuenta:** 768 millones de multiplicaciones por consulta × 100 consultas por segundo
   = 768.000.000 × 100 = **76.800.000.000 multiplicaciones por segundo**.

2. **En notación científica:** **7,68 × 10¹⁰** multiplicaciones por segundo.
   "Ser exacto" deja de ser viable porque la fuerza bruta compara la consulta contra
   **todos** los vectores: su costo es O(N · d) y crece en proporción directa al número de
   documentos. Con 1 millón de vectores ya son decenas de miles de millones de operaciones
   por segundo, y si la colección crece el costo sigue creciendo igual. No escala para
   responder en tiempo real.

3. **ANN:** se sacrifica la garantía de encontrar siempre los vecinos exactos (el recall
   típico es >95 %, no 100 %) y a cambio se gana velocidad: HNSW busca en O(log N), unas
   10.000 veces más rápido que la fuerza bruta con N = 1M y d = 768.


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

## Comparación

| Tipo | Pregunta | Antes | Después |
|---|---|---|---|
| Fácil | ¿Puedo pagar con Nequi? | Sí, puedes pagar con Nequi. | Sí, se acepta Nequi como medio de pago. [Medios de pago y facturación] |
| Ambigua | Soy celíaco, ¿qué postre puedo comer? | Puedes comer el brownie de chocolate, ya que es el único postre sin gluten. | Puedes comer el brownie de chocolate, ya que es el único postre sin gluten. [Alergias e ingredientes] |
| Fuera de contexto | ¿Venden almuerzos? | No tengo esa información en el contexto proporcionado. | No encontré esa información en los documentos disponibles. |

## Conclusión

- **La instrucción con más efecto fue la de citar el documento entre corchetes.** Antes las
  respuestas no indicaban de dónde salía la información; ahora cada respuesta termina con
  el título correcto (`[Medios de pago y facturación]`, `[Alergias e ingredientes]`). Eso
  permite verificar la respuesta contra la fuente.
- **La frase fija se cumplió exactamente:** en la pregunta fuera de contexto respondió
  palabra por palabra *"No encontré esa información en los documentos disponibles."*, y
  correctamente no citó ningún documento, porque no usó ninguno.
- **Ninguna instrucción fue ignorada.** La regla de máximo 3 oraciones no tuvo un efecto
  visible, porque las respuestas ya eran de una sola oración antes del cambio; para ponerla
  a prueba habría que hacer una pregunta que requiera una respuesta más larga.


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

