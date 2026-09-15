# Análisis — Laboratorio 4

Análisis escrito de los resultados de [`laboratorio_4_barca_cafe.ipynb`](laboratorio_4_barca_cafe.ipynb):
36 reseñas en español de Barça Cafe (ver [`data/`](data/)). Todas las cifras salen de los CSV de
[`resultados/`](resultados/).

## Resultados en una tabla

| | Modelo por defecto | Modelo multilingüe (flujo integrado) |
|---|---|---|
| Modelo | `distilbert-base-uncased-finetuned-sst-2-english` | `nlptown/bert-base-multilingual-uncased-sentiment` |
| Clases | 2 (POS / NEG) | 5 estrellas → POS / NEU / NEG |
| Reseñas POS / NEU / NEG | 9 / 0 / 27 | 16 / 7 / 13 |
| Score promedio | 0.872 | 0.578 (polaridad 0.744) |
| Accuracy en reseñas polares (29) | 55.2% | 72.4% |
| F1 POSITIVE / NEGATIVE | 0.43 / 0.63 | 0.80 / 0.72 |

Humano: 15 positivas, 14 negativas, 5 mixtas, 2 neutras.

**NER** (`mrm8488/bert-spanish-cased-finetuned-ner`, agregación `first`): 27 entidades,
0.75 por reseña, 15 de 36 reseñas con al menos una.
Tipos: LOC (10), MISC (8), PER (5), ORG (4).

---

## Parte E — Casos difíciles

### #9 · Sarcasmo

> Genial, 40 minutos esperando una hamburguesa que llegó fría. Una experiencia inolvidable, de verdad.

- **Predicción:** POSITIVE (5 stars, 0.914) · score de polaridad 0.983 · modelo por defecto NEGATIVE (0.683)
- **Entidades:** —
- **Interpretación humana:** NEGATIVO. Queja por 40 minutos de espera y comida fría; 'genial' e 'inolvidable' son ironía.
- **¿Coinciden?** No (modelo por defecto: Sí)
- **Posible causa:** Pesan las palabras positivas ('Genial', 'inolvidable') y el modelo no detecta que chocan con los hechos narrados (40 minutos, llegó fría). Reconocer ironía exige razonamiento pragmático que un clasificador entrenado con reseñas literales no aprende.

### #34 · Sarcasmo con ancla léxica

> Cinco estrellas... para el aire acondicionado, que es lo único que funcionaba.

- **Predicción:** POSITIVE (5 stars, 0.945) · score de polaridad 0.974 · modelo por defecto POSITIVE (0.553)
- **Entidades:** —
- **Interpretación humana:** NEGATIVO. Las 'cinco estrellas' son para el aire acondicionado, lo único que funcionaba.
- **¿Coinciden?** No (modelo por defecto: No)
- **Posible causa:** 'Cinco estrellas' es casi literalmente la etiqueta '5 stars' con la que se entrenó; en reseñas reales esa frase casi siempre acompaña a una nota de 5. Lo más probable es que el modelo haya aprendido ese atajo y no procese el giro '...para el aire acondicionado'.

### #27 · Doble negación

> No puedo decir que no me haya gustado: el bocadillo de jamón estaba bueno y no tardaron nada.

- **Predicción:** NEGATIVE (2 stars, 0.559) · score de polaridad 0.820 · modelo por defecto NEGATIVE (0.977)
- **Entidades:** —
- **Interpretación humana:** POSITIVO. 'No puedo decir que no me haya gustado' equivale a 'me gustó'; 'no tardaron nada' es un elogio.
- **¿Coinciden?** No (modelo por defecto: No)
- **Posible causa:** Tres marcadores negativos ('no', 'no', 'nada') que el modelo parece sumar como señales negativas en lugar de componerlos: dos negaciones se cancelan y 'no tardaron nada' es positivo.

### #11 · Negación con sentido positivo (litote)

> No está nada mal para ser un sitio turístico. Las bravas no tienen nada que envidiar a las de cualquier bar de Les Corts.

- **Predicción:** NEUTRAL (3 stars, 0.446) · score de polaridad 0.446 · modelo por defecto NEGATIVE (0.995)
- **Entidades:** LOC:Les Corts (1.00)
- **Interpretación humana:** POSITIVO. 'No está nada mal' y 'no tienen nada que envidiar' son elogios.
- **¿Coinciden?** Parcial (modelo por defecto: No)
- **Posible causa:** Acumula 'no', 'nada', 'mal' y 'envidiar'. El modelo duda (3 estrellas) y no llega a invertir la polaridad: sumando 4 y 5 estrellas la probabilidad positiva es 0.48, apenas por encima del 0.45 neutro.

### #23 · Opinión mixta

> Los camareros son un amor, pero la cocina es un desastre: el pollo seco y las patatas congeladas.

- **Predicción:** NEGATIVE (2 stars, 0.433) · score de polaridad 0.653 · modelo por defecto NEGATIVE (0.973)
- **Entidades:** —
- **Interpretación humana:** MIXTO. Servicio excelente ('un amor') y cocina muy mala ('un desastre').
- **¿Coinciden?** Parcial (modelo por defecto: Parcial)
- **Posible causa:** Una sola etiqueta no puede expresar dos aspectos opuestos. El modelo pesa más la cláusula que sigue al 'pero' y da 2 estrellas; el score bajo refleja la mezcla. Haría falta análisis de sentimiento por aspectos (servicio / comida / precio).

### #13 · Texto muy corto

> Carísimo.

- **Predicción:** POSITIVE (5 stars, 0.324) · score de polaridad 0.544 · modelo por defecto POSITIVE (0.985)
- **Entidades:** —
- **Interpretación humana:** NEGATIVO. Una sola palabra que critica el precio.
- **¿Coinciden?** No (modelo por defecto: No)
- **Posible causa:** Sin contexto, y con un vocabulario multilingüe que trocea la palabra en 'car' + '##isi' + '##mo', el modelo no tiene señal. Su score es de los más bajos del lote: avisa que no sabe. El modelo por defecto, en cambio, se equivoca con 0.985.

### #29 · Entidad desconocida + jerga

> La Blaugrana Burger con salsa Culé está brutal, pídanla sin dudar.

- **Predicción:** NEGATIVE (1 star, 0.326) · score de polaridad 0.577 · modelo por defecto NEGATIVE (0.984)
- **Entidades:** MISC:Blaugrana Burger (1.00) | MISC:Culé (1.00)
- **Interpretación humana:** POSITIVO. En España 'brutal' significa 'buenísimo'; 'Blaugrana Burger' y 'salsa Culé' son productos.
- **¿Coinciden?** No (modelo por defecto: No)
- **Posible causa:** El sentido normativo de 'brutal' (cruel, violento), que coincide con el inglés, es negativo y probablemente es el que dominó en el entrenamiento. Los nombres inventados no aportan señal. En NER, con 'simple' 'Culé' salía partido en 'Cul' + 'é'; con 'first' ambos productos quedan como MISC.

### #16 · Errores ortográficos

> q buen sitio!! los nachos buenisimos y el camarero super atento, bolveremos seguro

- **Predicción:** POSITIVE (5 stars, 0.738) · score de polaridad 0.966 · modelo por defecto NEGATIVE (0.754)
- **Entidades:** —
- **Interpretación humana:** POSITIVO. Elogio claro pese a 'q', 'buenisimos' y 'bolveremos'.
- **¿Coinciden?** Sí (modelo por defecto: No)
- **Posible causa:** Coincide. Las faltas no borran las señales fuertes ('buen sitio', 'super atento', '!!') y el modelo es uncased y quita acentos, así que 'buenisimos' y 'buenísimos' se tokenizan igual. El modelo por defecto falla porque no conoce el español.

### #28 · Negatividad implícita + NER

> Esperaba algo digno del Museu del Barça y me encontré un bar de aeropuerto con precios de Plaça de Catalunya.

- **Predicción:** NEUTRAL (3 stars, 0.460) · score de polaridad 0.460 · modelo por defecto POSITIVE (0.879)
- **Entidades:** MISC:Museu del Barça (0.69) | LOC:Catalunya (0.97)
- **Interpretación humana:** NEGATIVO. Compara el local con un bar de aeropuerto con precios de zona turística.
- **¿Coinciden?** Parcial (modelo por defecto: No)
- **Posible causa:** No hay palabras negativas: la crítica está en la comparación, y entenderla requiere saber que un 'bar de aeropuerto' es caro y mediocre. El modelo queda en 3 estrellas. En NER, 'Plaça de Catalunya' se reduce a 'Catalunya' y con 'simple' 'Museu' salía partido en 'Muse' + 'u'.

### Patrones que dejan los casos difíciles

1. **El score no protege contra el sarcasmo.** Las dos predicciones con mayor score de todo el
   lote (#34 con 0.945 y #9 con 0.914) son sarcasmos clasificados como 5 estrellas. El modelo está
   más seguro justo cuando más se equivoca.
2. **La negación se lee como vocabulario, no como operador.** "No está nada mal" (#11) y
   "no puedo decir que no me haya gustado" (#27) contienen varias palabras negativas y el modelo las
   acumula en vez de invertir su efecto. El modelo por defecto falla las dos con scores de 0.99 y 0.98.
3. **Cuando falta señal, el modelo sí lo avisa.** En el texto de una palabra (#13), en la reseña
   informativa (#17) y en la crítica implícita (#28) el score cae por debajo de 0.5. En esos casos el
   score funciona como alarma útil.
4. **Robusto a faltas, frágil ante la jerga.** Las reseñas con errores ortográficos (#15 y #16) se
   clasifican bien, pero una sola palabra de uso regional ("brutal", #29) invierte la predicción.
5. **En NER hay dos tipos de error.** Los de *agregación* (entidades partidas en subpalabras)
   desaparecen con `first`. Los de *etiquetado* se quedan: una persona pública marcada como `MISC`,
   una entidad truncada y un falso positivo por mayúscula inicial.

---

## Preguntas de reflexión

### 1. ¿El modelo parece apropiado para reseñas en español?

**El modelo por defecto, no.** `distilbert-base-uncased-finetuned-sst-2-english` se entrenó con
reseñas de cine en inglés. Sobre las 36 reseñas predijo `NEGATIVE` en 27, acertó el **55 %** de
las polares (apenas mejor que una moneda) y solo reconoció **1 de cada 3** positivas (recall 0.33).
Sus predicciones positivas más seguras son justo las dos reseñas con palabras en inglés ("Top!!" y
"Amazing… very friendly", ambas con 0.9998).
Lo más problemático es que falla con seguridad: su score promedio es 0.87.

**El modelo multilingüe es aceptable como primer filtro, pero no es ideal.**
`nlptown/bert-base-multilingual-uncased-sentiment` sí vio reseñas en español y llega al **72 %** de
acierto, con F1 de 0.80 en positivas y 0.72 en negativas. Aun así se entrenó con reseñas de
*producto*, no de restaurantes, y no conoce la jerga de España. El siguiente paso sería un modelo
nativo en español (en una prueba exploratoria con seis frases,
`pysentimiento/robertuito-sentiment-analysis` resolvió mejor "no está mal" pero también cayó en
el sarcasmo) o, mejor, afinar con unos cientos de reseñas de restaurantes etiquetadas.

### 2. ¿Qué errores de sentimiento encontraste?

- **Sarcasmo** clasificado como positivo y con el score más alto del lote (#9, #34).
- **Negación compuesta:** litote (#11 → 3 ★) y doble negación (#27 → 2 ★).
- **Jerga regional:** "está brutal" leído como negativo (#29 → 1 ★).
- **Texto muy corto:** "Carísimo." → 5 ★ (#13), aunque con score bajo.
- **Reseña informativa** sin opinión (#17) → 5 ★. El modelo no tiene clase "sin opinión": todas las
  reseñas con las que se entrenó valoran algo.
- **Crítica implícita** por comparación (#28 → 3 ★).
- **Mixtas:** tres de cinco caen en 3 ★ y dos en 2 ★. Es razonable, pero pierde el aspecto positivo.
- **Sesgo del modelo por defecto:** casi todo lo que no entiende lo manda a `NEGATIVE`, y así acierta
  negativas "por accidente".

### 3. ¿Qué problemas observaste en NER?

- **Fragmentación en subpalabras** con `aggregation_strategy="simple"`: 8 fragmentos de 32 entidades
  en el modelo en español (`Nu`+`ria`, `Spo`+`tify Camp Nou`) y 22 de 46 en el modelo por defecto.
  Con `first` bajan a 0.
- **Falsos positivos del modelo en inglés** sobre palabras en mayúscula o trozos de palabras:
  `del` → LOC, `ca` → PER, `Me` (de "Meh.") → PER, `Carís` → ORG, y `mb`/`ues` → MISC dentro de
  "hamburguesa(s)" (posiblemente por su parecido con *Hamburg*).
- **Tipo equivocado:** `Lamine Yamal` sale como `MISC` y no como `PER`.
- **Entidades truncadas:** `Plaça de Catalunya` → `Catalunya`; de "los hermanos Iglesias" solo queda
  `Iglesias`, que además es un sustantivo común.
- **Mayúscula = entidad:** `Ambiente único` se marca `MISC` solo por empezar oración.
- **Categorías que no encajan en el dominio:** CoNLL-2002 son noticias. No existe una clase para
  platos o productos (`Blaugrana Burger` cae en `MISC`), y `Spotify Camp Nou` es a la vez estadio
  (LOC) y marca patrocinada (ORG), así que el modelo lo manda a `MISC`.
- **Lo que funcionó:** los cuatro nombres ficticios de personas (Nuria, Santiago, Mateo, Sergi) y las
  organizaciones y países se detectaron, todos con score superior a 0.9.

### 4. ¿Cómo influye el score en tu confianza?

El score es la probabilidad softmax de la clase ganadora, **no la probabilidad de acertar**.

- En el modelo por defecto, de las 23 predicciones polares con score > 0.8 solo acertó el **61 %**.
- En el multilingüe, el rango de score > 0.8 acierta menos (75 %) que el de 0.5–0.8 (89 %), porque
  ahí están los dos sarcasmos.
- Con cinco clases el score de la estrella ganadora **subestima** la confianza: #24 es claramente
  positiva, acierta y tiene 0.30, porque el resto de la probabilidad se fue a 4 estrellas. Sumando la
  polaridad sube a 0.55. Ese `score_polaridad` se comporta mejor: 82 % de acierto por encima de 0.8
  y 0 % por debajo de 0.5 (aunque ese rango solo tiene 3 reseñas).

Conclusión: **un score bajo es una buena alarma** para mandar la reseña a revisión humana, pero **un
score alto no es garantía**. Además no es comparable entre modelos con distinto número de clases, y
saber si está calibrado exige datos etiquetados.

### 5. ¿Qué datos etiquetarías manualmente para calcular precision, recall y F1?

**Sentimiento**
- Una muestra **estratificada** de reseñas reales: todas las estrellas, idiomas y longitudes, y
  sobremuestreo de los casos difíciles (sarcasmo, negación, mixtas), porque son raros pero caros.
- Al menos 200–300 reseñas, para tener unas 30 por clase aun con la distribución real, que es muy
  polarizada (176 / 33 / 34 / 44 / 137).
- Etiquetas `POSITIVO` / `NEGATIVO` / `NEUTRO` / `MIXTO` y marcas de fenómeno, asignadas por **dos
  anotadores** con una guía escrita, midiendo su acuerdo (kappa de Cohen). Las estrellas del usuario
  sirven como señal débil, pero no reemplazan la etiqueta: la nota y el texto no siempre coinciden.
- Reportar **F1 macro**, porque las clases están desbalanceadas.

En este laboratorio ya se hizo una versión mínima: la columna `etiqueta_humana` permitió calcular
las métricas de la Parte D.4.

**NER**
- Spans exactos con su tipo, siguiendo una guía que decida de antemano los casos ambiguos: estadio =
  LOC, Barça Cafe = ORG, platos = una clase `PRODUCTO` nueva y figuras públicas = PER.
- Evaluación **a nivel de entidad** (coincidencia exacta de span y tipo, como `seqeval`), y aparte
  las coincidencias parciales para cuantificar las truncaciones (`Catalunya` vs. `Plaça de Catalunya`).

### 6. ¿Usarías estas predicciones para tomar decisiones automáticas importantes? ¿Por qué?

**No.**
- El 72 % de acierto sale de 29 reseñas: la muestra es pequeña y el margen de error, amplio.
- Los errores **no son aleatorios**: se concentran en sarcasmo y negación, que abundan justo en las
  reseñas enojadas, las que más importan.
- Los errores más graves llegan con score alto, así que un umbral de confianza no los filtra.
- El NER extrae nombres de empleados ("el encargado Sergi"). Tomar decisiones sobre personas a
  partir de eso tiene implicaciones laborales y de privacidad.

Sí lo usaría como **apoyo**, con una persona en el circuito:
- **Tendencias agregadas**, por ejemplo el porcentaje de reseñas negativas por mes o por tema. Ahí
  los errores individuales se compensan y basta con una auditoría humana periódica.
- **Triaje:** mandar a revisión humana primero lo `NEGATIVE` o con score de polaridad bajo.

No lo usaría para responder públicamente de forma automática, borrar reseñas, fijar precios ni
evaluar o sancionar a un empleado mencionado.
