# Uso de la IA

La IA se utilizó fue ChatGPT de Open IA como apoyo para interpretar los errores de ejecución del notebook en la extención de Google Colab y proponer ajustes de configuración de PySpark.

Los promts utilizados fueron relacionados a analizar los errores entregados por el output de la celda y entregar un analisis del error y presentar una alternativa para su solución. Los principales errores presentados fueron: 

## 1. Conflicto de dependencias

La instalación inicial fijaba `pyspark==3.5.3`, pero el entorno tenía `dataproc-spark-connect 1.1.0`, que requiere `pyspark[connect]~=4.0.0`. Pip informó que ambas versiones eran incompatibles y lo presentaba en los output; se le entrego el error presentado dio la siguiente alternativa:

**Alternativas propuestas:** Chat propone utilizar la versión compatible ya instalada en Colab o cambiar la celda de instalación a:

```python
%pip -q install "pyspark[connect]~=4.0.0" pandas
```

Se actualizó el notebook con esta segunda opción, solucionando el problema.

## 2. Sesión de Spark con la versión anterior

Después del cambio apareció un error `Py4JJavaError`, cuya causa indicaba que no se encontraba `spark-hive_2.12-3.5.3.jar`. Esto señalaba que la sesión Java conservaba referencias a la instalación anterior.

**Solución propuesta:** reiniciar el kernel de Colab y ejecutar las celdas desde el inicio. Si la instalación cambiaba nuevamente la versión, reiniciar antes de continuar. Se explicó que ejecutar únicamente `spark.stop()` no sustituía el reinicio del kernel.

Con este diagnóstico se agregó una comparación entre la versión de PySpark y la de Spark en Java. Para revisar que versión estaba corriendo el programa, luego de la verificación se retiró esa comprobación para simplificar la celda. 

