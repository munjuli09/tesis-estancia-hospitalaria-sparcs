# Modelado de la duración de la estancia hospitalaria mediante aprendizaje automático

Material de reproducción del trabajo de grado *Modelado de la Duración de la Estancia
Hospitalaria mediante Aprendizaje Automático*.

**Juliana Muñoz Gónzalez** y **Daniel Humberto Vallejo Vecil**
Programa de Ciencia de Datos · Departamento de Matemáticas
Universidad Externado de Colombia · Bogotá, 2026
Tutor: Julián Fernando Sánchez López

---

## De qué trata

El trabajo estudia en qué medida la duración de la estancia hospitalaria puede modelarse a partir
de un registro administrativo de egresos, y qué variables concentran el peso explicativo de esas
estimaciones.

Se construye sobre el archivo público de altas hospitalarias del estado de Nueva York
(**SPARCS De-Identified File, 2021**), que reúne 2.135.260 egresos. Se comparan tres familias de
algoritmos y dos estrategias de modelado bajo un protocolo único de preprocesamiento, partición y
validación, y el modelo seleccionado se somete a un análisis de interpretabilidad mediante
valores SHAP.

### Qué se encontró

El mejor modelo —XGBoost con pérdida de Tweedie, en configuración de una sola etapa— alcanza un
R² de 0,4646 sobre el conjunto de prueba, con un MAE de 2,90 días. Tres resultados sostienen las
conclusiones del trabajo:

- **El desempeño es insensible al algoritmo.** La correlación entre los residuos de las distintas
  familias supera 0,96: los modelos fallan en los mismos episodios. El límite lo impone la
  información que el registro contiene, no la capacidad del método.
- **El valor del esquema en dos fases está en cómo se recompone la predicción**, no en separar
  los regímenes. La implementación habitual empeora el desempeño frente a un modelo único; la
  recomposición por esperanza lo iguala.
- **La limitación está en la magnitud, no en la identificación.** El clasificador distingue las
  estancias prolongadas con un AUC-ROC de 0,847, pero estimar cuántos días durarán explica apenas
  el 40,55 % de su variabilidad.

---

## Estructura

```
.
├── notebooks/          Los seis cuadernos, en orden de ejecución
├── data/               Capas de datos (vacío en el repositorio, ver más abajo)
│   ├── 1_bronze/       Archivo de origen, sin transformar
│   ├── 2_silver/       Conjunto depurado
│   └── 3_gold/         Matriz de características y partición
├── modelos/            Modelos entrenados (no versionados, se regeneran)
├── tablas/             Salidas en .csv de las que procede cada tabla del documento
├── figuras/            Gráficas generadas por los cuadernos
├── docs/
│   └── INSTRUCTIVO.md  Guía de ejecución paso a paso
└── requirements.txt
```

Los cuadernos resuelven sus rutas como `Path.cwd().parent`, de modo que **deben ejecutarse desde
`notebooks/`** y las demás carpetas deben ser hermanas suyas. Si se mueven los archivos, las
rutas dejan de resolver.

---

## Los datos

El archivo de origen **no se incluye en el repositorio** por su tamaño, pero es de acceso público
y descarga directa, sin registro ni solicitud previa:

> New York State Department of Health. *Hospital Inpatient Discharges (SPARCS De-Identified):
> 2021.* Portal Health Data NY.
> <https://health.data.ny.gov/Health/Hospital-Inpatient-Discharges-SPARCS-De-Identified/tg3i-cinn>

Descárgalo en formato CSV y colócalo en `data/1_bronze/`. El nombre que esperan los cuadernos es
`Hospital_Inpatient_Discharges_(SPARCS_De-Identified)__2021_20260906.csv`; si el tuyo difiere,
ajusta la constante `RAW_FILE` en el cuaderno 02.

El archivo está desidentificado y no contiene información protegida de salud bajo HIPAA: la edad
está agrupada en rangos, el código postal truncado a tres dígitos y las estancias de 120 días o
más agregadas en una sola categoría.

---

## Reproducir el análisis

```bash
git clone https://github.com/munjuli09/tesis-estancia-hospitalaria-sparcs.git
cd tesis-estancia-hospitalaria-sparcs
python -m venv .venv && source .venv/bin/activate   # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Después, con el archivo de origen ya en `data/1_bronze/`, ejecuta los cuadernos desde
`notebooks/` en el orden de su numeración. Cada uno consume la salida del anterior.

| Cuaderno | Entrada | Salida |
|---|---|---|
| `01_EDA_SPARCS` | Archivo de origen | Tablas descriptivas |
| `02_Preprocessing_SPARCS` | Capa *Bronze* | Capa *Silver* |
| `03_FeatureEngineering_SPARCS` | Capa *Silver* | Capa *Gold* y partición |
| `05_Modeling_SPARCS` | Capa *Gold* | Modelo final y comparación de pérdidas |
| `08_DobleValla_SPARCS` | Capa *Gold* | Esquema en dos fases, tres variantes |
| `11_SHAP_SPARCS` | Modelo final | Atribuciones y figuras |

La numeración conserva la del desarrollo del proyecto. Los números intermedios corresponden a
exploraciones preliminares que no forman parte de los resultados reportados.

El [instructivo](docs/INSTRUCTIVO.md) detalla los tiempos aproximados, los requisitos de memoria
y qué hacer si algo falla.

### Tres condiciones que no pueden alterarse

1. **La semilla es 42** en la partición, en el ajuste de los modelos de ensamble y en el muestreo
   para el análisis SHAP.
2. **La partición es 80/20** y queda fijada en la capa *Gold*. El modelado no vuelve a
   particionar: hacerlo mezclaría registros ya usados para calcular el *Target Encoding*.
3. **El *Target Encoding* se calcula solo sobre entrenamiento** y se aplica a prueba con esos
   valores. Codificar antes de particionar produce fuga de información.

---

## Trazabilidad de las cifras

Cada cuaderno exporta sus resultados a `tablas/` con la nomenclatura `NN_NombreTabla.csv`, donde
`NN` identifica el cuaderno de origen. Toda cifra del documento puede rastrearse hasta el archivo
y la operación que la generaron.

---

## Alcance

El archivo SPARCS es un resumen de egreso: se elabora cuando el episodio ya concluyó y no registra
en qué momento quedó determinado el valor de cada campo. Lo que estos modelos representan es la
**relación entre el registro de egreso y la duración de la estancia**, no una predicción
disponible en el momento del ingreso. Las variables de codificación clínica —APR-DRG, severidad,
riesgo de mortalidad y categorías CCSR— se asignan a partir del registro completo del episodio.

El trabajo no evalúa el modelo en un entorno asistencial ni mide su efecto sobre indicador alguno
de gestión, y no se realizó validación externa sobre otra población.

---

## Cómo citar

```bibtex
@misc{repositorio2026,
  author       = {Muñoz Gónzalez, Juliana and Vallejo Vecil, Daniel Humberto},
  title        = {Modelado de la duración de la estancia hospitalaria mediante
                  aprendizaje automático: material de reproducción},
  year         = {2026},
  howpublished = {\url{https://github.com/munjuli09/tesis-estancia-hospitalaria-sparcs}},
  note         = {Cuadernos, tablas de resultados e instructivo de ejecución}
}
```

---

## Licencia

Código bajo licencia MIT. Los datos de origen pertenecen al New York State Department of Health y
se distribuyen bajo la *Freedom of Information Law* del estado de Nueva York; este repositorio no
los redistribuye.
