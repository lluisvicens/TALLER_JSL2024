# TALLER_JSL2024
## Introducción al análisis de datos lidar con R, y modelización 3d con Blender.

A largo de este breve taller introductorio, se mostrará un flujo de trabajo básico, para el trabajo con datos LiDAR desde el entorno de R, con el objetovo de generar un modelo digital del terreno y un modelo digital de superficie que posteriormente, se visualizará en forma de escenario tridimensional, utilizando para ello el programa de diseño 3d Blender.

## Software necesario para el desarrollo del taller:

* **[R](https://www.r-project.org/)** y **[Rstudio](https://posit.co/download/rstudio-desktop/)**
* **[QGIS](https://qgis.org/en/site/)**
* **[Blender](https://www.blender.org/)**

## Datos que se utilizarán en el taller:

Básicamente se van a utilizar datos LiDAR, que puedan descargarse de cualquier geoportal abierto de datos tales como el visor de descargas del **[ICGC](http://www.icc.cat/appdownloads/)** o el Centro de Descargas del **[CNIG](https://centrodedescargas.cnig.es/CentroDescargas/index.jsp)**, entre otros.

Para el presente taller podéis descargar los datos desde aquí:

* [1 fichero LAZ]()
* [conjunto de ficheros LAZ]()

## Trabajando con datos LiDAR desde R
### Configurando el espacio de trabajo

R se trabajará desde el entorno de desarrollo RStudio, que facilita sobre manera la gestión de ficheros, scripts, carpetas, proyectos y datos, así como la visualización de los mpas, o gráficos. Así pues, abriremos RStudio y:

* podemos configurar el entorno de trabajo bien indicando cual es el **directorio de trabajo** -_setwd()_- o bien, configurando un proyecto desde zero (_.Rproj_). Tabién será preciso crear un nuevo script de R.
```R
setwd("ruta/de/acceso/a/los/datos")  #establecer directorio de trabajo
getwd() #comprobación del working directory
```

* instalar los paquetes necesarios: **lidR**
* activar el paquete **lidR**

```R
install.packages("lidR)   #instalación del paquete lidR
library(lidR)   #llamada del paquete lidR
```

### Lectura y comprobaciones básicas sobre los ficheros LAS/LAZ
#### Importando la totalidad del fichero

Una vez configurado el punto de partida, el siguiente paso consistirá en la lectura del fichero o ficheros LAS/LAZ, para generar los objetos de R que son con os cuales vamos a trabajar. La función principal para la lectura de ficheros LiDAR, es **_readLAS()_**. Como todas las funciones de R, ésta se compone de varios argumentos posibles que podemos configurar o no, en función de la información que precisemos extraer de los ficheros originales:

```R
# importar 1 fichero las/las: opción básica
las1 <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz")

# recuperamos información básica del objeto recién creado:
print(las1)
summary(las1)   # información extendida

# comprobamos CRS, EPSG, ...
crs(las1)
epsg(las1)
```
En el caso anterior, la función **_readLAS()_** lee la totalidad del archivo original y traspasa dicha información al nuevo objeto de R (perteneciente a la clase **LAS**). En ocasiones pero, puede interesar únicamente extraer algunos de los atributos que contiene el fichero LiDAR (valores XYZ, intensidad, clasificación, ...)

![Atributos de un fichero LAS](/image/atributos_las.png)

#### Importando parte del fichero: la selección de atributos -> SELECT

Uno de los argumentos que soporta la función básica **_readLAS()_** es la selección de los atributos que se quieren importar. El argumento en cuestión lleva por nombre, ```select```. Así por ejemplo podemos crear un nuevo objeto que contenga únicamente parte los atributos originales:

```r
# seleccionar los atributos a importar
las_xyz <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", select = "xyz")   # xyz
las_clasificado <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", select = "xyzc")   # xyz y clasificación
```
Lógicamente, la cantidad de atributos que se leen/importan en R afectará a:

* el tamaño del objeto
* la cantidad de información y disponible y por consiguiente, lo que podamos hacer con este objeto

![Tamaño del objeto LAS](/image/peso_las.png)

En los ejemplos anteriores se han importado las coordenadas XY y el valor de Z en el primer caso (xyz), y se ha añadido el atributo de clasificación en el segundo (c). A continuación, se muestran las abreviaciones del resto de atributos:

| Abreviación | Atributo               |
|-------------|------------------------|
| t           | hora del gps           |
| a           | ángulo de escaneo      |
| i           | intensidad             |
| n           | número de retornos     |
| r           | número de retorno      |
| c           | clasificación          |
| p           | identificador de punto |
| ...         | ...                    |

#### Importando parte del fichero: la selección de puntos -> FILTER

Además de escoger qué atributos se van a leer, también es posible seleccionar parte de las geometrías que conforman la nube de puntos LiDAR. Para ello, podemos echar mano del argumento ```filter```.

```r
# seleccionar los puntos a importar, en base a sus características
las_xyz_fr <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz",
                   select = "xyz",
                   filter = "-keep_first")   # xyz y primer rebote

las_xyz_ground <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz",
                      select = "xyz",
                      filter = "-keep_class 2")   # xyz y clasificado como suelo

las_xyz_ground <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz",
                          select = "xyz",
                          filter = "-keep_class 2")   # xyz y clasificado como suelo


las_xyz_1k_2k <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz",
                          select = "xyz",
                          filter = "-keep_z 1000 2000")   # xyz comprendidos entre 1000 y 2000 metros
```
Para comprobar las características de cada uno de los objetos generados, basta con ejecutar las funciones **_print()_** para una versión simplificada, o **_summary()_** para obtener más detalles. Con relación a la clasificación de una nube de puntos LiDAR,la referencia es la especificación de la ASPRS (The American Society for Photogrammetry & Remote Sensing):

![Clasificación de puntos lidar](/image/las_classes.png)

Para consultar todas las posibilidades que admite el argumento filter, basta con ejecutar la función:

```r
readLAS(filter = "-help")
```

#### La validación de los objetos creados

La validación de los objetos creados con la función readLAS() o dicho de otro modo, la confirmación que los datos son validos para su procesamiento y uso para generar por ejemplo, un modelo digital del terreno, pasa por comprobar si estos objetos o datos, se ajustan a las especificaciones emitidas por la ASPRS. La función encargada de dicha validación es **_las_check()_:**

```r
las_check(las1)
las_check(las_xyz_ground)
```
Uno de los problemas que de manera frecuente afectan o pueden afectar a los datos LiDAR es la presencia de puntos duplicados. La función **_las_check()_** informa de la presencia estos puntos, lo que permite posteriormente, que sean eliminados del objeto con el cual se va a trabajar.

```r
Checking the data
  - Checking coordinates... ✓
  - Checking coordinates type... ✓
  - Checking coordinates range... ✓
  - Checking coordinates quantization... ✓
  - Checking attributes type... ✓
  - Checking ReturnNumber validity... ✓
  - Checking NumberOfReturns validity... ✓
  - Checking ReturnNumber vs. NumberOfReturns... ✓
  - Checking RGB validity... ✓
  - Checking absence of NAs... ✓
  - Checking duplicated points...
    ⚠ 1943 points are duplicated and share XYZ coordinates with other points
  - Checking degenerated ground points... ✓
```
Además nos informa de la presencia o no de puntos etiquetados como ```withheld```. Estos son puntos que se han etiquetado de este modo porqué no son confiables, y no deberían ser tomados en cuenta para ningún tipo de análisis:

```r
  - Checking degenerated ground points... ✓
  - Checking attribute population... ✓
  - Checking gpstime incoherances ✓
  - Checking flag attributes...
    🛈 1496514 points flagged 'withheld'
  - Checking user data attribute... ✓
```
Por lo general se tata de puntos que son considerados errores o ruido. Frente a la presencia de este tipo de puntos etiquetados como 'withheld', podemos adoptar varias estrategias:

* Filtrar los puntos etiquetados como 'withheld' del objeto, antes de proceder a generar productos derivados.
* En algunos casos, si no se tata de una gran cantidad de puntos, cabe la posibilidad de valorar si parte de estos, pueden integrarse en futuros análisis o no.
* Y si la cantidad de puntos no confiables es muy alta, convendria además de no tomarlos en cuenta, investigar su origen y notificar esta cuestión al proveedor de los datos.

En cualquier caso, empezaremos por centrar nuestra atención sobre el primer error o aviso: **los puntos duplicados**.

#### Eliminar los puntos duplicados en un fichero LAS

Para eliminar los puntos que estén duplicados en una nube de puntos, basta con utilizar la función _**filter_duplicates()**_. El nuevo objeto que se va a generar, va a estar libre de estos duplicados, pudiéndolo comprobar mediante la aplicación de la función de validación de datos.

```r
las1_nodup <- filter_duplicates(las1)   # eliminación de puntos duplicados
las_check(las1_nodup)   # validación del nuevo objeto
```
En el resultado se aprecia que los 1943 puntos duplicados ya no están presentes en el nuevo objeto:

```
  - Checking RGB validity... ✓
  - Checking absence of NAs... ✓
  - Checking duplicated points... ✓
  - Checking degenerated ground points... ✓
  - Checking attribute population... ✓
  - Checking gpstime incoherances ✓
  - Checking flag attributes...
    🛈 1494805 points flagged 'withheld'
  - Checking user data attribute... ✓
```

#### Eliminar los puntos etiquetados como 'withheld'

En el contexto de este taller, para deshacernos de estos puntos no confiables, podemos utilizar una doble vía:

* o bien utilizamos una función especifica de filtrado de puntos,
* o bien podemos generar nuevamente el objeto LAS con **_readLAS()_**, pero esta vez, utilizando el argumento ```filter``` para desechar estos puntos:

```r
# crear nuevamente el objeto LAS, aplicando un filtro durante su lectura
las1_nowithheld <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", filter = "-drop_withheld")   # lectura
print(las1_nowithheld)   # resumen
las_check(las1_nowithheld)   # validación

# filtrar el objeto anteriormente creado
las1_filtrado <- filter_poi(las1, Classification != 7)   # filtrado
print(las1_filtrado)   # resumen
las_check(las1_filtrado)   # validación
```

Sea cual sea el método aplicado, acabaremos por llegar al mismo punto:

```
> print(las1_filtrado)   # resumen
class        : LAS (v1.2 format 1)
memory       : 595.2 Mb 
extent       : 360000, 362000, 4716000, 4718000 (xmin, xmax, ymin, ymax)
coord. ref.  : ETRS89 / UTM zone 31N 
area         : 3.98 km²
points       : 9.18 million points
density      : 2.31 points/m²
density      : 1.97 pulses/m²
> print(las1_nowithheld)   # resumen
class        : LAS (v1.2 format 1)
memory       : 560.2 Mb 
extent       : 360000, 362000, 4716000, 4718000 (xmin, xmax, ymin, ymax)
coord. ref.  : ETRS89 / UTM zone 31N 
area         : 3.98 km²
points       : 9.18 million points
density      : 2.31 points/m²
density      : 1.97 pulses/m²
```
Y cabe decir que no siempre ambos sistemas, pueden aplicarse por igual:

```r
# error!
las_xyz <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", select = "xyz")   # lectura
# Warning message:
# There are 1496514 points flagged 'withheld'.

las1_nowithheld <- filter_poi(las_xyz, Classification != 7)   # filtrado con error!
# Error in eval(expr, data, expr_env) : object 'Classification' not found
```
La cantidad de atributos que importemos para generar un objeto de la clase LAS, puede determinar en cierto modo las funciones que vayamos a poder aplicar.

### Generación de productos derivados del fichero LAS
#### Modelos digitales del terreno

##### Red de triangulos irregulares (TIN)

```r
# TIN
dtm_tin <- rasterize_terrain(las1_filtrado, res = 1, algorithm = tin())   # interpolación
lidR::plot(dtm_tin)   # representación 2D
lidR::plot_dtm3d(dtm = dtm_tin)   # representación 3D
```
##### Distancia Inversa Ponderada (IDW)

```r
# IDW
mdt_idw <- rasterize_terrain(las1_filtrado , algorithm = knnidw(k = 10L, p = 2))  # interpolación
lidR::plot(mdt_idw)   #representación 2D
lidR::plot_dtm3d(mdt_idw, bg = "black")   #representación 3D 
```

###### Kriging

```r
# Kriging
mdt_kriging <- rasterize_terrain(las, algorithm = kriging(k = 40))   # interpolación
lidR::plot(mdt_kriging)   # representación 2D
lidR::plot_dtm3d(mdt_kriging, bg = "black")   # representación 3D
```

![TIN](/image/tin.png)
