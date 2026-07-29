# Prácticas mecanismos avanzados de seguridad #2

## Ejercicio 4

### Flujos no autorizados y bloqueo de tráfico

Libreta de práctica para los estudiantes del curso CIB-209, Temas Especiales en Seguridad de Datos y Sistemas. Eje del curso: seguridad de sistemas, conexiones entre componentes.

## Aviso sobre los datos

Los datos son sintéticos y no corresponden al tráfico real de ninguna red. Los conjuntos vienen incluidos junto a la libreta en cuatro archivos: flujos_autorizados.csv con la matriz aprobada en el diseño, flujos_autorizados_corregida.csv con esa misma matriz más los dos servicios legítimos que nunca se documentaron, conexiones_observadas.csv con las 600 conexiones de la semana y su servicio de negocio asociado, e historial_conexiones_etiquetado.csv con las 1200 conexiones que sirven para entrenar. Los datos ya están generados y no se calculan al ejecutar, por lo que todos los grupos ven exactamente los mismos números.

## ¿Cómo ejecutar en Binder?

1. Abrir el repositorio en Binder con el botón de abajo:

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/wrodriguez-cenfotec/ejercicio4_flujos_no_autorizados/HEAD?urlpath=%2Fdoc%2Ftree%2Fejercicio4_flujos_no_autorizados.ipynb)

2. La primera carga tarda unos minutos mientras se construye el entorno. Tener paciencia.
3. Ejecutar las celdas en orden, de arriba hacia abajo.
4. La sesión de Binder es temporal. Antes de cerrar, guardar las capturas de los resultados.

## ¿Qué técnica se usa y qué conviene recordar?

Este ejercicio combina dos técnicas distintas, en dos etapas.

La etapa 1 no aprende nada. Compara cada conexión observada contra la matriz de flujos aprobada, exigiendo coincidencia exacta en cuatro campos: segmento de origen, segmento de destino, puerto y protocolo. Lo que no está en la matriz queda separado como no autorizado. Ese es el modelo de denegación por omisión.

La etapa 2 sí usa aprendizaje automático supervisado. Una máquina de vectores de soporte se entrena con 1200 conexiones ya etiquetadas como benignas o maliciosas y busca la frontera que mejor separa las dos clases. Para una conexión nueva, la confianza que reporta indica de qué lado de esa frontera cae y qué tan lejos está de ella, expresado entre 0 y 1. No es una probabilidad de culpabilidad ni una prueba: es la distancia a una frontera aprendida de ejemplos pasados.

Vocabulario que conviene tener claro:

- White list: lista de lo que está expresamente permitido.
- Tupla: combinación de origen, destino, puerto y protocolo que se compara contra la white list.
- Denegación por omisión: lo que no está expresamente permitido queda prohibido.
- Frontera de decisión: separación que el modelo aprendió entre las dos clases.
- Confianza: qué tan lejos de la frontera cae la conexión, del lado malicioso.
- Umbral: valor de confianza a partir del cual se bloquea. Es una decisión de negocio.

## ¿Qué hace la libreta?

- Carga la matriz de flujos autorizados entre seis segmentos, las 600 conexiones observadas durante una semana y el historial de 1200 conexiones etiquetadas que sirve para entrenar.
- Etapa 1, filtrado por coincidencia exacta de tupla contra la white list bajo el modelo de denegación por omisión: compara cada conexión observada con la matriz aprobada en las llaves origen, destino, puerto y protocolo, y separa las que no tienen coincidencia. Esta etapa es determinista y no usa aprendizaje automático.
- Etapa 2, máquina de vectores de soporte con núcleo lineal sobre las conexiones no autorizadas: asigna una confianza de que la conexión es maliciosa y decide bloquear o permitir según el umbral configurado.
- Genera la tabla de reglas DENY y ALLOW para el cortafuegos y el resumen de impacto con los servicios de negocio interrumpidos y las conexiones maliciosas que quedaron permitidas.

## ¿Cómo se lee el gráfico de barras?

Cada barra horizontal es una conexión no autorizada, identificada a la izquierda por su código y por el servicio de negocio al que pertenece. El largo de la barra es la confianza de que la conexión sea maliciosa, de 0 a 1.

La línea negra vertical es el umbral configurado. Las barras rojas quedaron a la derecha del umbral y se bloquean. Las grises quedaron por debajo y se permiten.

Lo que conviene revisar es el nombre del servicio de cada barra roja, para ver si lo que se corta es tráfico sin dueño o un servicio del que depende la operación, y también las barras grises más largas, que son las conexiones que el control dejó pasar aunque estuvieran cerca del umbral.

## ¿Qué debe modificar el grupo?

Solo la celda CONFIGURACIÓN. Los parámetros son matriz_de_flujos_autorizados y umbral_de_confianza.

| Corrida | matriz_de_flujos_autorizados | umbral_de_confianza |
| --- | --- | --- |
| 1 | original | 0.5 |
| 2 | corregida | 0.5 |
| 3 | corregida | 0.8 |

Efecto de cada parámetro. matriz_de_flujos_autorizados determina cuánto tráfico legítimo llega a la etapa 2: si la white list está incompleta, servicios aprobados por el negocio pero nunca documentados en el diseño caen al clasificador y terminan bloqueados, y en ese caso el problema no es el modelo sino el inventario. umbral_de_confianza fija cuánta certeza se exige para cortar tráfico: bajarlo bloquea más y produce interrupciones de servicio, subirlo protege la disponibilidad y deja pasar lo que se parece a tráfico normal.

## ¿Qué se entrega?

Hoja de evidencia del ejercicio 4, con los valores de las tres corridas y tres capturas. El detalle está en el documento Practicas_Mecanismos_Avanzados_Seguridas_2.docx de la práctica.

## Referencias

- https://scikit-learn.org/stable/modules/svm.html
- https://pandas.pydata.org/docs/user_guide/groupby.html
- https://matplotlib.org/stable/api/pyplot_summary.html
- https://attack.mitre.org/
