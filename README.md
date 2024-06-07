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

Una vez configurado el punto de partida, el siguiente paso consistirá en la lectura del fichero o ficheros LAS/LAZ, para generar los objetos de R que son con os cuales vamos a trabajar. La función principal para la lectura de ficheros LiDAR, es **readLAS()**. Como todas las funciones de R, ésta se compone de varios argumentos posibles que podemos configurar o no, en función de la información que precisemos extraer de los ficheros originales:

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
En el caso anterior, la función **readLAS()** lee la totalidad del archivo original y traspasa dicha información al nuevo objeto de R (perteneciente a la clase **LAS**). En ocasiones pero, puede interesar únicamente extraer algunos de los atributos que contiene el fichero LiDAR (valores XYZ, intensidad, clasificación, ...)

![Atributos de un fichero LAS](/image/atributos_las.png)

#### Importando parte del fichero: la selección de atributos -> SELECT

Uno de los argumentos que soporta la función básica readLAS() es la selección de los atributos que se quieren importar. El argumento en cuestión lleva por nombre, **select**. Así por ejemplo podemos crear un nuevo objeto que contenga únicamente parte los atributos originales:

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

Además de escoger qué atributos se van a leer, también es posible seleccionar parte de las geometrías que conforman la nube de puntos LiDAR. Para ello, podemos echar mano del argumento **filter**.

```r
# seleccionar los atributos a importar
las_xyz <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", select = "xyz", filter)   # xyz
las_clasificado <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", select = "xyzc")   # xyz y clasificación
```

Con relación a la clasificación de una nube de puntos LiDAR,la referencia es la especificación de la ASPRS (The American Society for Photogrammetry & Remote Sensing):

![Clasificación de puntos lidar](/image/las_classes.png)

Además, para ver todas las posibilidades que adminten los argumentos select y filter que se acaban de ver, basta con ejecutar las funciones:

```r
readLAS(select = "-help")
readLAS(filter = "-help")
```
