# Análisis del entregable 1 RaSA

## 1. Inventario y granularidad

| Fuente | Filas / columnas | Detalle de una fila |
|---|---|---|
| F1 | 188.815 / 9 | Área de servicio, geografía y fecha. |
| F2 | 849 / 9 | Tipo de beneficio, condiciones y fecha. |
| F3 | 36.036 / 11 | Beneficio de un plan por área, proveedor, fecha, nivel de servicio y condiciones de pago. |
| F4 | 31 / 3 | Condición de pago con descripción y tipo. |

Detalle propuesto; no hay claves declaradas ni comentarios de columnas.

## 2. Identificadores

F1 tiene 5.409 áreas; F2, 178 beneficios; F3, 393 valores de plan (incluido el vacío) y 125 proveedores; F4, 23 condiciones. Filas no equivalen a entidades.

## 3. Perfilamiento

### 3.1 Nulos, cardinalidad y duplicados

F1 presenta nulos en ID de área (3,33 %), geografía (3,38 %), área y densidad (1,27 % cada una). F2 no tiene NULL, pero 559 unidades de límite están vacías (65,84 %). F3 tiene 5,79 % de beneficios y 5,66 % de áreas sin ID; cantidadLimite está ausente en 84,83 % y hay 2.053 planes vacíos (5,70 %). F4 no presenta NULL ni cadenas vacías.

Filas duplicadas excedentes: F1, 43.573; F2, 271; F3, 8.493; F4, 7. F1 tiene 43.573 pares idénticos; quedarían 145.242 filas distintas. Por ID: total = distintos + repeticiones + filas con nulos.

### 3.2 Estadísticas y frecuencias

F1: población mediana 30.885 frente a media 41,48 millones; máximo 47.280 millones. Área mínima -24.707; requiere aclarar unidades y origen. Indiana: 7,49 % de registros; hay 35 estados.

F2: predominan beneficios cubiertos (Yes: 87,51 %); los indicadores mezclan Yes/No, True/False, Si/Nein y "Algunas veces".

F3: copago medio 9,78, mediana 0 y máximo 3.500; coseguro medio 24,29, mediana 0 y máximo 100. El 75 % de copagos es cero. Cantidad límite: mediana 25 y máximo 30.000, solo entre datos presentes. Fecha mezcla formatos y contiene 2.052 registros de 1920.

F4: predomina Copago (61,29 %); aparecen etiquetas como Copagado, Coseguridad y "NaN" textual.

### 3.3 Gráficas

Histogramas y boxplots: asimetría y extremos en población, área, copago y límites. Barras: frecuencias.

## 4. Contraste con RaSA

Cifras observadas frente al enunciado:

Coinciden F1: 5.409 áreas; F4: 5 condiciones de coseguro; F3: máximo de coseguro 100 en 2018. Difieren F2: 178/170 beneficios; F4: 14/15 condiciones de copago; F3: 202/301 planes en 2017, 286/422 en 2018 y copago máximo 3.500/3.300. F1 contiene 1800 y F1 y F2 no muestran 2019.

## 5. Relaciones

F3 puede vincularse con F1 por IdAreaDeServicio_T, con F2 por IdTipoBeneficio_T y con F4 mediante sus IDs de condición de copago/coseguro hacia IdCondicionesDePago_T. 

**Cardinalidad:** F2-F3 sería uno a muchos si cada beneficio fuera único en F2 y tuviera varias filas en F3. Los IDs se repiten en ambas; unir solo por ID puede multiplicar filas (ejemplo: 3 por 10 producen 30). Para confirmarla, contar por ID y vigencia, separando duplicados de las variables.
