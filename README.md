  # Maestría en Ciencia de Datos - Taller II
  ### UNIVERSIDAD YACHAY TECH 
### María Brigith Suárez Rodríguez 

## Título: Aprendizaje de Máquina para el Análisis del Rendimiento de Ventas en Comercio Electrónico

### Objetivos:
- 1. Algoritmos de aprendizaje automático: Utilizar algoritmos de aprendizaje automático para el análisis de datos de comercio
- 2. Clasificación de clientes: Clasificación de clientes en varias categorías.
- 3. Segmentación de Clientes: Utilizar técnicas de agrupamiento para segmentar a los clientes basándose en su comportamiento de compra.
- 4. Predicción de Ventas: Desarrollar un modelo predictivo para pronosticar ventas futuras basándose en datos históricos.
 
#### Parte 0: Preparación
- Exploración y limpieza de Datos
- Descargar el conjunto de datos Online Retail II.
- Cargar el conjunto de datos utilizando pandas y realizar una exploración inicial para entender la estructura y contenido de los datos.
- Limpiar los datos manejando valores faltantes, eliminando registros duplicados y filtrando entradas irrelevantes (por ejemplo, transacciones
sin ID de clientes o pedidos cancelados).

###### 1. Mapeo y Normalización de Columnas (Consistencia Multianual)


El Problema: El conjunto de datos abarca dos periodos distintos (2009-2010 y 2010-2011). En el Excel original de UCI, las columnas tenían nombres inconsistentes entre hojas (por ejemplo, 'Customer ID' con espacio en una hoja y 'CustomerID' en otra, o 'Price' en lugar de 'UnitPrice').


La Solución: Creamos un diccionario inteligente de mapeo (COLUMN_MAPPING) que normalizó los nombres de las columnas a un estándar unificado (InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country) antes de concatenar la información.

###### 2. Conversión Correcta de Tipos de Datos (Data Types)

InvoiceNo y StockCode: Se convirtieron a tipo de texto (string) explícito y se eliminaron espacios en blanco laterales. También se removió el sufijo .0 residual que a veces genera la lectura de números desde Excel.


InvoiceDate: Se parseó al formato estándar de fecha y hora (datetime64[ns]) interpretando correctamente el orden de los días.


CustomerID: Los identificadores de clientes venían con formato decimal (ej. 12345.0) o nulos. Se transformaron primero a números enteros (Int64) para remover el punto decimal y luego se guardaron como texto limpio.

Quantity y UnitPrice: Se aseguraron como numéricos puros utilizando pd.to_numeric con manejo de errores coercitivos.

###### 3. Filtrado de Registros Críticos (Garantía de Calidad)

El algoritmo eliminó de manera estricta cualquier fila que tuviera valores nulos (NaN) en columnas esenciales para la facturación: InvoiceNo, StockCode, Quantity, UnitPrice, InvoiceDate y Country. Esto asegura que cada registro represente una transacción real evaluable.


###### 4. Tratamiento de Descripciones Faltantes


Para no perder transacciones valiosas que únicamente carecían de texto explicativo en el producto, las descripciones nulas se completaron por defecto con el valor genérico 'Unknown' (Desconocido).

###### 5. Creación de Variables de Valor Predictivo

IsCancellation (Booleana): Identifica si la transacción es una cancelación o devolución (aquellas cuyo InvoiceNo comienza con la letra 'C'). Esto es fundamental para análisis de comportamiento de clientes, cálculo de tasas de retorno y depuración de ingresos netos.

TotalPrice (Numérica): Se calculó de forma directa la multiplicación de Quantity * UnitPrice para cada línea de la factura. Esta columna es la base para predecir ventas (ventas brutas, valor de vida del cliente o LTV, y análisis de demanda monetaria).


La base resultante con 1,067,371 filas se guardó en data/online_retail_ii_clean.csv, lista y óptima para cualquier modelo predictivo (como pronósticos de series de tiempo de demanda o segmentación de clientes).

- Análisis Exploratorio de Datos
- Analizar la distribución de variables clave (por ejemplo, cantidad de productos comprados, precio de los productos y fecha de compra).
- Identificar los productos más vendidos y visualizar sus tendencias de ventas a lo largo del tiempo.
- Explorar los patrones estacionales en los datos de ventas e identificar los períodos de máximas ventas.

**DISTRIBUCIONES CANTIDAD PRECIO**

<img width="1950" height="750" alt="image" src="https://github.com/user-attachments/assets/99c2535c-dda5-4846-977b-213fb7ba9a65" />

**ESTACIONALIDAD MENSUAL**

<img width="1800" height="750" alt="image" src="https://github.com/user-attachments/assets/2eebfc1b-c8fe-4ffb-9a97-938b39906d13" />

**TOP DE PRODUCTOS**

<img width="2100" height="1050" alt="image" src="https://github.com/user-attachments/assets/568376b2-8f01-4374-a46c-5b9c65732a0f" />

**VENTAS MENSUALES**

<img width="2100" height="750" alt="image" src="https://github.com/user-attachments/assets/6b432d70-425b-4f8d-8d87-51856a4b8386" />


#### Parte 1: Clasificación de clientes:
- Definir una nueva columna en los datos para trabajar con dos clases: “Clientes Normales” y “Clientes Premium”. ¿Cómo podríamos definir estas categorías, por volumen de compras, cantidades, etc? Definir este campo.
- Entrenar un modelo de clasificación para la clasificación de los clientes en estas dos categorías.

###### Propuesta para definir "Clientes Normales" y "Clientes Premium":

Para clasificar a los clientes de forma analítica y objetiva en lugar de usar números arbitrarios, utilizaremos la metodología estándar de la industria llamada RFM (Recency, Frequency, Monetary) agregando los datos a nivel de cliente (CustomerID):

- Monetary (Ingresos totales): Suma de la facturación neta de cada cliente (TotalPrice).

- Frequency (Frecuencia): Cantidad de transacciones únicas (InvoiceNo) realizadas.

- Recency (Recencia): Días desde su última compra hasta el final del periodo del dataset.

Regla de Clasificación: Definiremos como "Cliente Premium" a aquel cliente que se encuentre en el top 20% superior en cuanto a volumen de compra neto (Monetary) o número de transacciones (Frequency). El restante 80% será clasificado como "Cliente Normal".

#### Parte 2: Segmentación de Clientes
- Usar el agrupamiento K-means para categorizar a los clientes en grupos distintos y analizar las características de cada segmento.
- Usar el agrupamiento Mean Shift para categorizar a los clientes en grupos distintos y analizar las características de cada segmento.
- Comparar los resultados de las dos técnicas anteriores


#### Parte 3: Predicción de Ventas
- Crear nuevas características, si es necesario, para ayudar en el modelado predictivo (por ejemplo, época del año, categorías de productos).
- Dividir los datos en conjuntos de entrenamiento y prueba.
- Elegir un algoritmo de aprendizaje automático adecuado 
lineal, árbol de decisión o bosque aleatorio) para predecir ventas futuras.
- Evaluar el modelo utilizando métricas de rendimiento apropiadas.
