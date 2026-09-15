# LAB-4-Extraccion-Linguistica-Con-Hugging-Face

**Laboratorio 4 — Extracción lingüística con Hugging Face**
Pipeline de NLP con modelos preentrenados de `transformers` que clasifica el sentimiento, extrae
entidades nombradas y analiza casos difíciles sobre **36 reseñas en español de Barça Cafe**, el
restaurante oficial del FC Barcelona junto al Spotify Camp Nou.

Curso de Procesamiento de Lenguaje Natural — Universidad del Valle de Guatemala.

---

## Qué hay aquí

| Ruta | Contenido |
|---|---|
| [`laboratorio_4_barca_cafe.ipynb`](laboratorio_4_barca_cafe.ipynb) | **Notebook ejecutado.** Partes A–E, estadísticas y reflexión. |
| [`Laboratorio4_ExtraccionLinguistica_Linares.pdf`](Laboratorio4_ExtraccionLinguistica_Linares.pdf) | El notebook en formato de reporte (33 páginas), para entregar. |
| [`entrega/`](entrega/) | Fuente LaTeX del PDF, con el logo y la figura que usa. |
| [`laboratorio_4_resultados.csv`](laboratorio_4_resultados.csv) | **Entregable.** Una fila por reseña: sentimiento, score, entidades. |
| [`ANALISIS.md`](ANALISIS.md) | Casos difíciles y reflexión escrita, fuera del notebook. |
| [`data/resenas_barca_cafe.csv`](data/resenas_barca_cafe.csv) | Dataset de 36 reseñas con etiqueta humana y tipo de caso. |
| [`resultados/`](resultados/) | JSON del flujo integrado, comparaciones, métricas y casos difíciles en CSV. |
| [`resultados/figuras/`](resultados/figuras/) | Gráficas de la Parte D. |

## Dataset

Las reseñas se basan en las opiniones de Google Maps de Barça Cafe (Carrer d'Arístides Maillol 12,
Les Corts, Barcelona): **3.2 ★ y 424 opiniones**, consultadas el 14 de septiembre de 2026.

**No se copiaron reseñas literales.** Cada reseña de Google Maps trae nombre, foto y perfil del
autor, que son datos personales que la guía pide no incluir. Además, el texto pertenece a su autor
y los Términos de Servicio de Google Maps no permiten extraerlo y reutilizarlo. Por eso:

- **6 reseñas son paráfrasis** de las opiniones que Google Maps muestra públicamente, reescritas
  sin autor ni fecha.
- **30 son simuladas** a partir de los temas reales que Google detecta en las 424 opiniones:
  hamburguesa, barça, mesa, cerveza, estadio, baños, vaso de plástico…
- **Todos los nombres de clientes y empleados son ficticios** (Nuria, Sergi, Santiago, Mateo). Se
  mantienen organizaciones, lugares y figuras públicas (FC Barcelona, Les Corts, Lamine Yamal),
  que hacen falta para probar NER.

| Etiqueta humana | Positivo | Negativo | Mixto | Neutro |
|---|---:|---:|---:|---:|
| Reseñas | 15 | 14 | 5 | 2 |

Casos cubiertos: negación (6), texto corto (4), sarcasmo (3), errores ortográficos (2), entidad
desconocida, jerga, mezcla de idiomas y 14 reseñas con entidades.

## Modelos

| Tarea | Modelo | Rol |
|---|---|---|
| Sentimiento | `pipeline("sentiment-analysis")` → `distilbert-base-uncased-finetuned-sst-2-english` | Llamada literal de la guía; línea base |
| Sentimiento | `nlptown/bert-base-multilingual-uncased-sentiment` (1–5 ★) | **Flujo integrado** |
| NER | `pipeline("token-classification", aggregation_strategy="simple")` → `dbmdz/bert-large-cased-finetuned-conll03-english` | Llamada literal de la guía; línea base |
| NER | `mrm8488/bert-spanish-cased-finetuned-ner` (PER / ORG / LOC / MISC) con `simple` y con `first` | **Flujo integrado** (`first`) |

## Resultados principales

- **El modelo por defecto no sirve para español.** Marca 27 de 36 reseñas como `NEGATIVE` y acierta
  el **55 %** de las reseñas polares, con un score promedio de 0.87: se equivoca con mucha seguridad.
- **El modelo multilingüe acierta el 72 %** (F1 0.80 en positivas y 0.72 en negativas). Manda la
  mayoría de las reseñas mixtas a 3 ★.
- **Un score alto no garantiza acierto.** Las dos predicciones más seguras del modelo multilingüe
  (0.945 y 0.914) son **sarcasmos** clasificados como 5 ★. Con cinco clases el score de la estrella
  ganadora además subestima la confianza, así que se reporta también `polarity_score`, la
  probabilidad sumada de la polaridad.
- **`aggregation_strategy="simple"` parte entidades en subpalabras** (`Nu` + `ria`,
  `Spo` + `tify Camp Nou`): 8 fragmentos en el modelo en español y 22 en el de inglés. Con `first`
  quedan en 0.
- **NER en el flujo integrado:** 27 entidades (0.75 por reseña) de tipo LOC, MISC, PER y ORG.
  Siguen errores de etiquetado: `Lamine Yamal` como `MISC`, `Plaça de Catalunya` truncada a
  `Catalunya`.

El análisis de los nueve casos difíciles y las seis preguntas de reflexión están en
[`ANALISIS.md`](ANALISIS.md) y al final del notebook.

## Reproducir

```bash
pip install -r requirements.txt
python -m nbconvert --to notebook --execute --inplace laboratorio_4_barca_cafe.ipynb
```

Para regenerar el PDF (dos pasadas, por las tablas largas):

```bash
cd entrega
pdflatex Laboratorio4_ExtraccionLinguistica_Linares.tex
pdflatex Laboratorio4_ExtraccionLinguistica_Linares.tex
```

La primera ejecución del notebook descarga unos 3 GB de modelos desde el Hub de Hugging Face. También corre en
CPU, solo que más lento; aquí se ejecutó en GPU (RTX 4060). La inferencia es determinista: dos ejecuciones dan las
mismas predicciones y los mismos scores.
