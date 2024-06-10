# TALLER_JSL2024
## 1. Introducción al análisis de datos lidar con R, y modelización 3d con Blender.

A largo de este breve taller introductorio, se mostrará un flujo de trabajo básico, para el trabajo con datos LiDAR desde el entorno de R, con el objetovo de generar un modelo digital del terreno y un modelo digital de superficie que posteriormente, se visualizará en forma de escenario tridimensional, utilizando para ello el programa de diseño 3d Blender.

## 2. Software necesario para el desarrollo del taller:

* **[R](https://www.r-project.org/)** y **[Rstudio](https://posit.co/download/rstudio-desktop/)**
* **[QGIS](https://qgis.org/en/site/)**
* **[Blender](https://www.blender.org/)**

## 3. Datos que se utilizarán en el taller:

Básicamente se van a utilizar datos LiDAR, que puedan descargarse de cualquier geoportal abierto de datos tales como el visor de descargas del **[ICGC](http://www.icc.cat/appdownloads/)** o el Centro de Descargas del **[CNIG](https://centrodedescargas.cnig.es/CentroDescargas/index.jsp)**, entre otros.

Para el presente taller podéis descargar los datos desde aquí:

* [1 fichero LAZ](https://drive.google.com/file/d/11XUt-XEteQU_O-_m4G38X1BKu6lyam8W/view?usp=sharing)
* [conjunto de ficheros LAZ](https://drive.google.com/file/d/161cLiO64T_0-hW5k2JQh1QBFw-gw9FmO/view?usp=sharing)

## 4. Trabajando con datos LiDAR desde R
### 4.1. Configurando el espacio de trabajo

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

### 4.2. Lectura y comprobaciones básicas sobre los ficheros LAS/LAZ
#### 4.2.1. Importando la totalidad del fichero

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

#### 4.2.2. Importando parte del fichero: la selección de atributos -> SELECT

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

#### 4.2.3. Importando parte del fichero: la selección de puntos -> FILTER

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

#### 4.2.4. La validación de los objetos creados

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

#### 4.2.5. Eliminar los puntos duplicados en un fichero LAS

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

#### 4.2.6. Eliminar los puntos etiquetados como 'withheld'

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

### 4.3. Generación de productos derivados del fichero LAS
#### 4.3.1. Modelos digitales del terreno

Por defecto, el algoritmo **_rasterize_terrain()_** utiliza, para la generación de un modelo digital del terreno, únicamente aquellos puntos que estén clasificados como suelo (2) o agua (9). Así pues bastará con definir la resolución del modelo, y el algoritmo de interpolación a utilizar.

##### a) Red de triangulos irregulares (TIN)

```r
# TIN
dtm_tin <- rasterize_terrain(las1_filtrado, res = 1, algorithm = tin())   # interpolación
lidR::plot(dtm_tin)   # representación 2D
lidR::plot_dtm3d(dtm = dtm_tin)   # representación 3D
```
##### b) Distancia Inversa Ponderada (IDW)

```r
# IDW
mdt_idw <- rasterize_terrain(las1_filtrado , algorithm = knnidw(k = 10L, p = 2))  # interpolación
lidR::plot(mdt_idw)   #representación 2D
lidR::plot_dtm3d(mdt_idw, bg = "black")   #representación 3D 
```

##### c) Kriging

```r
# Kriging
mdt_kriging <- rasterize_terrain(las, algorithm = kriging(k = 40))   # interpolación
lidR::plot(mdt_kriging)   # representación 2D
lidR::plot_dtm3d(mdt_kriging, bg = "black")   # representación 3D
```

![TIN](/image/tin.png)

#### 4.3.2.Modelos digitales de superficie
A diferencia de la función anterior, **_rasterize_canopy()_** utilizará el primer rebote almacenado para generar un modelo digital de superficie. Y del mismo modo que sucedía en el caso de la función rasterize_terrain(), en esta ocasión también disponemos de varios algoritmos para llevar a cabo la interpolación: p2r(), dsmtin(), pitfree().  

##### Red de triangulos irregulares (TIN)
```r
# DSM TIN
dsmtin <- rasterize_canopy(las1_filtrado, res = 0.5, algorithm = dsmtin())
lidR::plot(dsmtin)
lidR::plot_dtm3d(dsmtin)
```
##### Algoritmo PITFREE

```r
# DSM PITFREE
dsm_pitfree <- rasterize_canopy(las1_filtrado, res = 1, pitfree())
lidR::plot_dtm3d(dsm_pitfree)
```
![DSM2](/image/dsm2.png)

## 5. Visualizaciones 3D en R

Además de la función **_plot_dtm3()_** del paquete lidR, existen en R otras opciones para visualizar datos en 3D, como por ejemplo utilizando el paquete **rayshader**.

### 5.1. Trabajando con el paquete rayshader
Como siempre en R, en caso de ser necesario instalarmos el paquete rayshader y lo activaremos:

```r
install.packages("rayshader")
library(rayshader)
```

A continuación, necesitaremos convertir nuestro objeto raster (SpatRaster)  en una matriz, con ayuda de la función **_raster_to_matrix()_**:

```r
# aplicar textura a la matriz, y visulizar en 2D
matriz_mds |> 
   sphere_shade(texture = "desert") |> 
   plot_map()

# podemos añadir una capa o layer de iluminación/sombras
matriz_mds |> 
   sphere_shade(texture = "desert") |> 
   add_shadow(ray_shade(matriz_mds), 0.5) |> 
   plot_map()

# y generar un escenario 3D
matriz_mds |> 
   sphere_shade(texture = "desert") |> 
   add_shadow(ray_shade(matriz_mds, zscale = 3), 0.5) |> 
   add_shadow(ambient_shade(matriz_mds), 0) |> 
   plot_3d(matriz_mds, zscale = 10, fov = 0, theta = 135, zoom = 0.75, phi = 45, windowsize = c(1000, 800))
```

## 6. Visualizaciones 3D en Blender

### 6.1. La preparación de los datos
Antes de pasar a trabajar con Blender para recrear un escenario 3D, pasaremos por una breve tarea de preparación de los datos a fin de aprovechar al máximo las capacidades de Blender. Ello implicará guardar el modelo digital del terreno o de elevaciones, como una imagen de valores enteros de 16 bits sin signo (UInt16), con el objetivo de disponer de una nueva capa con valores comprenidos entre 0 y 65535. Esta transformación, puede hacerse des de QGIs, por ejemplo (mediante la calculadora raster) o bien desde R. Para realizar dicha transformación hay que aplicar la formula "**_(valor del raster - valor mínimo) / (valor máximo - valor mínimo) * 65535_**":

```r
# obtener el mínimo y máximo de la image
min_val <- minmax(dsm_pitfree)[1]
max_val <- minmax(dsm_pitfree)[2]

# álgebra de mapas
dsm_blender <- (dsm_pitfree - min_val) / (max_val - min_val) * 65535

# exportar objeto raster a capa raster
writeRaster(dsm_blender,"datos_lidar/dsm_blender.tif", datatype  = 'INT2U')
```

### 6.2. La configuración de Blender, paso a paso

#### 6.2.1. El espacio de trabajo
* Al abrir el programa, se seleccionan todos los elementos del escenario que aparecen por defecto (un cubo, una fuente de luz y una cámara), y se eliminan.
* En el menú Edit > Preferences, hay que activar la extensión **Import Images as Planes**.

#### 6.2.2. La importación del modelo digital de superficie o del terreno
* A continuación, debe debe importarse el modelo digital preparado anteriormente, en forma de un nuevo plano. Para ello basta con dirigirse al menú File > Import > Images as Planes. Se selecciona el fichero **dsm_blender.tif** y se acepta.
* Para evaluar el proceso anterior, se puede conmutar la vista entre **sólido**, **material** y **_rendered_**.
* En el apartado **Render**, hay que cambiar el parámetro **Render Engine** a **Cycles** y el **Feature Set**, configurarlo como **Experimental**.
* El siguiente paso consistirá en añadir al plano de trabajo, todos nodos que sean necesarios para poder **"deformar"** el plano y representar el relieve del modelo digital. Para ello, en el apartado **Modifiers**, hay que añadir un nuevo modificador (Add Modifier > Generate > Subdivision surface), activar el botón **Simple** y activar la casilla **Adaptative Subdivision**.
* De nuevo en el apartado **Render** para mejorar la velocidad de representación durante la fase de diseño, se modifica el valor presente en el parámetro Viewport > Max samples a **512**, y el valor del parámetro Render > Max samples a **30**. Estos valores podrán modificarse segun sea conveniente.
* Se divide la interfaz de Blender en dos espacios. El primero quedará configurado como **3D Viewport** y el segundo, como **Shade Editor**.
* En el apartado **Shade Editor** se observan tres piezas distintas: el modelo digital, un algoritmo que configura cómo se va a _renderizar_ el modelo, y el objeto de salida.

#### 6.2.3. Convertir el plano en un relieve
* Llegados a este punto, es el momento de aplicar las deformaciones al plano de trabajo, en base a la interpretación de la paleta de colores y de los valores del modelo digital de superficie importado como plano.
* Se añade un nuevo nodo desde el menú Add > Vector > Displacement. Y se enlaza el conector **Color** del MDS al conector **Height** del módulo encargado de configurar el desplazamiento, y el conector **Displacement** de este último, al conector **Displacement** del nodo **Material Output**.

![blender1](/image/blender1.png)

* Al conmutar la vista de material a renderizado, aun no puede apreciarse el desplazamiento que ha sufrido el plano orginal. Para ello, en el apartado **Material > Settings**, deberá modificarse el valor del parámetro **Displacement** de **Bump only** a **Displacement only**. A continuación hay que ajustar el valor de exageración en el parámetro **Scale** del nodo **Displacement**.

#### 6.2.4. Añadir la iluminación
* Desde el menú Add > Light > Sun se añade la fuente de luz necesaria para ilumina el escenario. En el apartado **Data** (icono en forma de bombilla eléctrica) pueden configurarse aspectos tales como la intensidad de la luz (por defecto 1) o el valor de ángulo (por defecto).
* En el apartado de propiedades, se pueden configurar los parámetros de localización, rotación, o escalado, por ejemplo. Los valores de **Rotation** pueden configurarse del siguiente modo (x = 0, y = 45, z = 135). Estos valores pueden configurarse de manera manual, seleccionando y arrastrando el punto de luz visible en el escenario.

#### 6.2.5. Añadir la cámara
* Desde el menú Add > camera se añade una cámara al escenario. Este elemento es indispensable para poder renderizar el escenario. Para comprobar que es lo que está observando la cámara (es decir, que parte del escenario se va a renderizar y desde que perspectiva) basta con presionar la tecla 0.
* Se puede configurar la vista del escenario de manera manual y a continuación, activar el menú View > Align View > Align Active Camera to View, y acabar de ajustar manualmente la perspectiva seleccionando la cámara en el panel de objetos y presionando la tecla **G**.
* Para ver una primera renderización del escenario, basta con presionar la tecla **F12**.
