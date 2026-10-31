# Análisis del entregable 2 RaSA · Calidad de datos

**Objetivo:** Evaluar el nivel de calidad de las fuentes de datos provistas (`F1` a `F4`) en la base de datos `RaSaTransaccional` a través de las cuatro dimensiones de calidad vistas en el curso (**Completitud, Validez, Unicidad y Consistencia**), contrastando los hallazgos con las reglas de negocio declaradas por RaSA.

---

## 1. Resumen ejecutivo por dimensión y fuente

| Fuente | Tabla | Completitud | Validez | Unicidad | Consistencia |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **F1** | `FuenteAreasDeServicio_Copia_E` | Nulos en `IdAreaDeServicio_T` (6.288; 3,33 %), `IdGeografia_T` (6.378; 3,38 %), `Area` y `Densidad` (2.407 simultáneos; 1,27 %). | 6.282 registros con año atípico `1800` en `Fecha`. | 43.573 filas excedentes (duplicados exactos). `0` colisiones de llave compuesta tras deduplicar. | 6.177 áreas negativas ($\le 0$), 6.142 poblaciones $> 20\text{M}$ (máx. $4,72 \times 10^{10}$), 6.182 densidades que no cumplen $\text{PoblacionAct}/\text{Area}$ y 6.282 años en `Fecha` que difieren del sufijo del ID. |
| **F2** | `FuenteTiposBeneficio_Copia_E` | Sin `NULL`, pero con 559 cadenas vacías (`""`) en `UnidadDelLimite` (65,84 %). | Mezcla de idiomas/formatos en booleanos (`Yes`/`No`, `True`/`False`, `Si`/`Nein`, `"Algunas veces"`) y textos concatenados en unidades. | 271 filas excedentes. **360 colisiones de llave** `(IdTipoBeneficio_T, Fecha)` con atributos contradictorios. | 33 registros contradicen internamente `TieneLimiteCuantitativo` vs. presencia de `UnidadDelLimite`. Ausencia del año 2019. |
| **F3** | `FuentePlanesBeneficio_Copia_E` | Nulos en `IdTipoBeneficio_T` (2.086; 5,79 %), `IdAreaDeServicio_T` (2.041; 5,66 %), `cantidadLimite` (30.571; 84,83 %) y 2.053 vacíos (`""`) en `IdPlan_T` (5,70 %). | 2.056 fechas con formato `Dec 31,YYYY` (vs. `YYYY-MM-DD`) y 2.052 registros en el año `1920`. | 8.493 filas excedentes. `0` colisiones de llave compuesta en los 22.968 registros válidos distintos. | Incumple Regla #2 en 14.471 casos (sin límite cuando `F2` lo exige) y 4.774 inversos. Copago máx. 2018 de $\$3.500$ ($> \$3.300$ en 8 filas). Huérfanos hacia `F1` (4.979 filas) y `F2` (2.023 filas). |
| **F4** | `FuenteCondicionesDePago_Copia_E` | Sin `NULL` técnicos, pero con 2 registros `"NaN"` textuales y 1 `"SinTipoCopago"` en `Tipo`. | Categorías fuera de dominio (`Copagado`, `Coseguridad`, `SinTipoCopago`, `NaN`) y ausencia de `Anticipado`. | 7 filas excedentes (24 distintas para 23 IDs). **1 colisión de llave primaria** en `IdCondicionesDePago_T = 204`. | Al homologar `Copagado` $\rightarrow$ `Copago` se logran las 15 condiciones esperadas por RaSA; en Coseguro resultan 6 (vs. 5 esperadas). `0` huérfanos desde `F3`. |

---

## 2. Detalle de hallazgos por dimensión de calidad

### 2.1 Completitud
1. **F1 (`FuenteAreasDeServicio`):**
   * Presenta ausencia de identificadores críticos de negocio y geografía: **6.288 nulos en `IdAreaDeServicio_T`** (3,33 %) y **6.378 nulos en `IdGeografia_T`** (3,38 %), lo que impide vincular dichos registros con planes o condados específicos.
   * Los **2.407 valores nulos en `Area` y `Densidad`** (1,27 %) ocurren exactamente en las mismas filas y corresponden a ciudades independientes del estado de Virginia (ej. *Alexandria City, Richmond City, Hampton City*), donde sí se reporta `PoblacionAct` pero se omitió la superficie territorial.
2. **F2 (`FuenteTiposBeneficio`):**
   * Aunque no registra valores `NULL` a nivel de base de datos, presenta **559 cadenas vacías (`""`) en `UnidadDelLimite`** (65,84 %), asociadas mayoritariamente a beneficios que no poseen límite cuantitativo.
3. **F3 (`FuentePlanesBeneficio`):**
   * Registra **2.086 nulos en `IdTipoBeneficio_T`** (5,79 %), **2.041 nulos en `IdAreaDeServicio_T`** (5,66 %) y **2.053 cadenas vacías (`""`) en `IdPlan_T`** (5,70 %). Estas ausencias en llaves foráneas e identificadores de plan degradan la trazabilidad de la oferta.
   * La columna `cantidadLimite` presenta **30.571 nulos (84,83 %)**, lo cual es parcialmente atribuible a beneficios sin tope cuantitativo, pero también afecta a beneficios que sí lo requieren (ver *Consistencia*).
4. **F4 (`FuenteCondicionesDePago`):**
   * No contiene valores `NULL` ni cadenas vacías, pero encubre datos faltantes mediante cadenas de texto como **`"NaN"` (2 registros)** y **`"SinTipoCopago"` (1 registro)** en la columna `Tipo`.

---

### 2.2 Validez
1. **Formatos y dominios temporales (`Fecha`):**
   * En **F1** y **F2**, el campo `Fecha` está tipado como entero de 4 dígitos (`YYYY`), pero en **F1** existen **6.282 registros con el año `1800`**, un valor fuera del dominio temporal del negocio (2017–2019).
   * En **F3**, la columna `Fecha` (tipo texto) **carece de un formato único**: combina el estándar `YYYY-MM-DD` (33.980 filas) con el formato de texto en inglés `Dec 31,YYYY` (**2.056 filas**: 1.029 de `Dec 31,2017` y 1.027 de `Dec 31,2018`). Adicionalmente, contiene **2.052 registros con fecha `1920-12-31`**, año inválido para la operación analizada.
2. **Dominios categóricos y booleanos:**
   * En **F4**, la columna `Tipo` incluye valores no válidos según el diccionario de datos: **`Copagado` (1)**, **`Coseguridad` (1)**, **`SinTipoCopago` (1)** y **`NaN` (2)**.
   * En **F2**, los atributos booleanos presentan falta de estandarización léxica e idiomática: `TieneLimiteCuantitativo` mezcla `No` (542), `Yes` (271), **`Nein` (24)** y **`Si` (12)**; `EsEHB` incluye **`True` (20)**; `EstaCubiertaPorSeguro` incluye **`False` (8)**; y `ExcluidoDelDesembolsoMaximoDentroDeLaRed` contiene **`"Algunas veces"` (2)**. Asimismo, `UnidadDelLimite` presenta cadenas triplicadas por error de concatenación (ej. `"Visit(s) per YearVisit(s) per Year..."`).

---

### 2.3 Unicidad
A partir del perfilamiento inicial se identificaron **duplicados exactos de fila completa** en las cuatro tablas (`F1`: 43.573 excedentes; `F2`: 271; `F3`: 8.493; `F4`: 7). Al eliminar dichos duplicados exactos (`dropDuplicates()`) y evaluar la unicidad de las llaves de negocio se concluye:

1. **Colisión de llave primaria en `F4`:**
   * De las 24 filas distintas resultantes existen **23 `IdCondicionesDePago_T` únicos**. El identificador **`IdCondicionesDePago_T = 204`** viola la unicidad de llave primaria al aparecer en dos registros con diferente descripción: *"Copay per Stay after deductible"* y *"Copay per Day"*.
2. **Colisión masiva de llave en `F2` `(IdTipoBeneficio_T, Fecha)`:**
   * De las 578 filas distintas en `F2`, solo existen **218 combinaciones únicas de `(IdTipoBeneficio_T, Fecha)`**, generando **360 filas en colisión de llave**. Un mismo beneficio en un mismo año presenta múltiples registros con valores opuestos en `EsEHB`, `EstaCubiertaPorSeguro`, `TieneLimiteCuantitativo` y exclusiones de red (ej. los IDs `5` y `10` en 2017). Esto evidencia que `F2` no se extrajo como un catálogo maestro puro, sino que conserva variaciones de condiciones por plan sin la llave del plan.
3. **Unicidad de llave compuesta en `F1` y `F3`:**
   * En **F1**, tras descartar duplicados exactos y nulos en identificadores, las **135.560 filas válidas** corresponden exactamente a **135.560 combinaciones únicas de `(IdAreaDeServicio_T, IdGeografia_T, Fecha)`** (`0` colisiones).
   * En **F3**, tras descartar duplicados exactos y llaves nulas/vacías, las **22.968 filas válidas** corresponden a **22.968 combinaciones únicas** de la llave compuesta `(IdPlan_T, IdTipoBeneficio_T, IdAreaDeServicio_T, IdNivelServicio_T, IdCondicionDePagoCopago_T, IdCondicionDePagoCoseguro_T, Fecha)` (`0` colisiones).

---

### 2.4 Consistencia
1. **Consistencia intra-tabla y demográfica en `F1`:**
   * **Áreas negativas y poblaciones extremas:** Se registran **6.177 filas con `Area <= 0`** (mínimo de `-24.707`) y **6.142 filas con `PoblacionAct > 20.000.000`** (máximo de $4,728 \times 10^{10}$, superior a la población mundial), lo que genera **6.182 registros donde la `Densidad` reportada no concuerda con la razón matemática `PoblacionAct / Area`**.
   * **Desfase entre `IdAreaDeServicio_T` y `Fecha`:** Los últimos 4 dígitos de `IdAreaDeServicio_T` codifican el año del registro (`2017` o `2018`). Existen exactamente **6.282 registros** donde dicho sufijo no coincide con la columna `Fecha`, los cuales corresponden en su totalidad a las filas con `Fecha = 1800`. Esto confirma que el año `1800` es un error de carga recuperable desde el propio identificador.
2. **Consistencia de reglas de negocio entre `F2` y `F3`:**
   * **Contradicción interna en `F2`:** Incluso homologando `Yes`/`Si` y `No`/`Nein`, existen **33 registros en `F2`** donde `TieneLimiteCuantitativo` contradice la presencia o ausencia de texto en `UnidadDelLimite`.
   * **Regla de negocio #2 (Límites cuantitativos $\ne 0$):** Al cruzar `F3` con los beneficios marcados con límite cuantitativo en `F2` (`Yes`/`Si`), **14.471 de 19.322 registros (74,9 %) incumplen la regla** al presentar `cantidadLimite` nulo o $\le 0$. Por otro lado, **4.774 de 31.844 registros** asociados a beneficios sin límite (`No`/`Nein`) reportan `cantidadLimite > 0` (efecto directo de la colisión de llaves en `F2`).
   * **Regla de negocio #5 (Topes 2018 de Copago $\$3.300$ y Coseguro $100\%$):** En 2018 el coseguro máximo cumple el tope de `100` (`0` excesos), pero el copago alcanza un máximo de **$\$3.500$**, superando el umbral de $\$3.300$ en **8 registros**. Además, el número de planes únicos observados en `F3` (202 en 2017, 286 en 2018 y 212 en el año anómalo 1920) difiere de los declarados por RaSA (301 en 2017 y 422 en 2018).
3. **Integridad referencial inter-tabla (`F3` hacia `F1`, `F2` y `F4`):**
   * **Orfandad `F3 -> F1`:** **4.979 registros en `F3`** (que agrupan **1.097 `IdAreaDeServicio_T` distintos**) no existen en `F1`. De los 6.496 IDs de área presentes en `F3`, solo 5.399 cruzan con los 5.409 de `F1`.
   * **Orfandad `F3 -> F2`:** **2.023 registros en `F3`** (que agrupan **107 `IdTipoBeneficio_T` distintos**) no existen en `F2`. De los 285 IDs de beneficio en `F3`, solo 178 cruzan con `F2` (donde RaSA declaraba 170).
   * **Integridad `F3 -> F4` y homologación:** Existe integridad referencial completa (`0` huérfanos) entre los códigos de copago/coseguro de `F3` y `F4`. Al homologar `Copagado` $\rightarrow$ `Copago` se obtienen exactamente los **15 IDs únicos de Copago** indicados por RaSA; al homologar `Coseguridad` $\rightarrow$ `Coseguro` se obtienen **6 IDs únicos de Coseguro** (frente a 5 indicados por el negocio).
   * **Relación `IdPlan_T` vs. `IdProveedor_T`:** En el 100 % de los registros con `IdPlan_T` no vacío (`0` inconsistencias), los primeros 5 dígitos del código del plan coinciden exactamente con `IdProveedor_T`.

---

## 3. Conclusiones generales de calidad y acciones de limpieza requeridas (ETL)

1. **Deduplicación obligatoria en capa de preparación:** Todas las tablas requieren eliminar duplicados exactos antes de cualquier cruce. En `F2` y `F4` se requiere además definir una regla de negocio con RaSA para resolver las colisiones de llave (`IdCondicionesDePago_T = 204` y los 360 registros en colisión de `F2`), ya que hacer un `JOIN` directo entre `F3` y `F2` sin desduplicar `F2` a nivel de `(IdTipoBeneficio_T, Fecha)` multiplica artificialmente las filas de planes.
2. **Estandarización y recuperación de fechas:** Es indispensable estandarizar en `F3` el formato `Dec 31,YYYY` a `YYYY-MM-DD`, imputar el año `1800` en `F1` extrayendo los últimos 4 dígitos de `IdAreaDeServicio_T` y validar con RaSA si los 2.052 registros de `1920-12-31` en `F3` corresponden a un error tipográfico de `2017` o `2018` (lo que explicaría los planes faltantes frente a los 301 y 422 esperados).
3. **Homologación de catálogos y filtrado de anomalías numéricas:** Se deben normalizar los campos booleanos de `F2` (`Si`/`Yes`/`True` $\rightarrow$ `Yes`; `Nein`/`No`/`False` $\rightarrow$ `No`), homologar `Copagado` y `Coseguridad` en `F4`, y excluir o corregir en `F1` los registros con áreas negativas y poblaciones distorsionadas ($> 20\text{M}$) para no sesgar los análisis de cobertura y concentración poblacional.