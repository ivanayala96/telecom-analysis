Análisis de uso de clientes — ConnectaTel

Análisis exploratorio del comportamiento de uso de servicios móviles (llamadas y mensajes) de ConnectaTel, empresa de telecomunicaciones con operaciones en México y Colombia.

🎯 Objetivo

Construir una visión confiable y accionable sobre cómo los clientes utilizan realmente los servicios móviles, con el fin de optimizar la oferta comercial y mejorar la experiencia del usuario.

El análisis responde a cuatro preguntas de negocio:

¿Qué segmentos de clientes muestran mayor o menor uso de llamadas y mensajes?
¿Qué usuarios presentan valores atípicos que puedan indicar comportamientos inusuales, fraude o errores de registro?
¿Cómo varía el uso según la edad y el tipo de plan contratado?
¿Qué patrones pueden ayudar a diseñar mejores planes y mejorar la satisfacción del cliente?
📊 Datasets

El análisis integra tres fuentes con granularidades distintas:

Archivo	Granularidad	Contenido
plans.csv	1 fila por plan (2 registros)	Catálogo de planes: precio mensual, minutos y mensajes incluidos, GB, costos por excedente
users_latam.csv	1 fila por cliente (4,000 registros)	Datos del cliente: edad, ciudad, fecha de registro, plan contratado, fecha de cancelación
usage.csv	1 fila por evento (40,000 registros)	Detalle de uso real: tipo de evento (llamada/mensaje), fecha, duración en minutos, longitud en caracteres

Relaciones entre tablas

usage.user_id → users_latam.user_id
users_latam.plan → plans.plan_name (los nombres de columna difieren entre ambas tablas)

Nota metodológica. usage está en formato largo: cada fila es una llamada o un mensaje, nunca ambos. Por ello se agrega a nivel usuario antes de combinarla con users_latam. Unir primero y agregar después produciría promedios ponderados por volumen de eventos en lugar de por cliente.

🔄 Etapas del análisis
1. Carga y exploración

Carga de los tres datasets, revisión de dimensiones, tipos de dato y vista preliminar de la estructura.

2. Identificación de problemas de calidad

Conteo de nulos declarados, detección de valores sentinel (invisibles para isna()), revisión de valores únicos en categóricas y validación de rangos temporales.

Hallazgos principales:

Problema	Columna	Registros	%
Nulos estructurales (cliente activo)	churn_date	3,534	88.35%
Nulos + sentinel "?"	city	565	14.13%
Sentinel -999	age	55	1.38%
Fechas futuras (año 2026)	reg_date	40	1.00%
Ceros sin interpretación	length	133	0.60%
Registros con duration y length simultáneos	usage	28	0.07%

Adicionalmente, churn_date venía almacenada como texto en notación científica con coma decimal ("1,71832E+18"), lo que requirió un proceso de conversión en tres pasos antes de poder leerla como fecha.

3. Limpieza
age: sentinel -999 convertido a nulo e imputado con la mediana (47 años)
city: sentinel "?" unificado como nulo
reg_date: fechas de 2026 marcadas como NaT
churn_date: corrección de formato y conversión desde timestamp Unix
usage: eliminación de las 28 filas contradictorias
duration y length: nulos estructurales conservados (verificación MAR contra type con dependencia del 100%)
4. Agregación y estadísticos por usuario

Construcción de user_profile: una fila por cliente con cant_mensajes, cant_llamadas y cant_minutos_llamada. Unión mediante left join desde users_latam para preservar al cliente sin actividad registrada.

5. Visualización y outliers

Histogramas segmentados por plan y boxplots de las cuatro variables clave. Cálculo de límites superiores mediante el método IQR.

6. Segmentación
Por uso: Bajo (<5 llamadas y <5 mensajes), Medio (<10 y <10), Alto (resto)
Por edad: Joven (<30), Adulto (<60), Adulto Mayor (60+)
7. Insight ejecutivo

Traducción de los hallazgos a conclusiones y recomendaciones comerciales.

📈 Resultados principales
Tasa de abandono del 11.65% (466 de 4,000 clientes), concentrada íntegramente entre junio y diciembre de 2024.
La oferta está sobredimensionada: el plan Básico incluye 100 minutos y el Premium 600, pero el consumo máximo de toda la base es de 155.69 minutos y la mediana apenas alcanza 19.73 minutos.
Ni la edad ni el plan contratado predicen el consumo. La distribución de edad es uniforme y las curvas de uso de ambos planes se superponen casi por completo.
El 73.6% de los clientes se concentra en el segmento de Uso medio, frente a un 19.5% de Bajo uso y un 6.9% de Alto uso.
⚙️ Cómo ejecutar el notebook
Opción A — Google Colab (recomendada)
Abre Google Colab
Selecciona File → Upload notebook y carga el archivo .ipynb de este repositorio
Sube los tres CSV al entorno con el panel de archivos (ícono de carpeta en la barra lateral)
Ajusta las rutas de pd.read_csv() si tus archivos no están en /datasets/
Ejecuta las celdas en orden con Runtime → Run all
Opción B — Entorno local
bash
git clone <URL-DEL-REPOSITORIO>
cd <NOMBRE-DEL-REPOSITORIO>
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook
🔁 Guía de reproducción

Requisitos

Python 3.9 o superior
pandas, numpy, matplotlib, seaborn

Pasos

Coloca plans.csv, users_latam.csv y usage.csv en la ruta que indican las celdas de carga
Ejecuta el notebook de principio a fin y en orden

⚠️ Importante: las celdas no son independientes. Cada paso de limpieza modifica los DataFrames en memoria, por lo que ejecutar una celda fuera de orden —o dos veces— puede producir resultados incorrectos. Si algo falla, reinicia el kernel (Runtime → Restart runtime) y vuelve a ejecutar desde el inicio.

Puntos de control

Si quieres verificar que la reproducción es correcta, estos valores deben coincidir:

Punto	Valor esperado
Filas de users_latam	4,000
Filas de usage tras eliminar contradictorios	39,972
Nulos en city tras unificar el sentinel	565
Fechas de cancelación válidas	466
Filas de user_profile	4,000
📁 Estructura del repositorio
.
├── README.md
├── notebooks/
│   └── connectatel_analysis.ipynb
└── datasets/
    ├── plans.csv
    ├── users_latam.csv
    └── usage.csv
🛠️ Stack

Python · pandas · numpy · matplotlib · seaborn · Jupyter Notebook
