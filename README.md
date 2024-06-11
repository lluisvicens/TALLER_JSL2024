# TALLER_JSL2024
## 1. Introducción al análisis de datos lidar con R, y modelización 3d con Blender.

El motivo que persigue este breve taller introductorio, es mostrar un flujo de trabajo elemental, para el procesado de datos LiDAR desde el entorno de R. El objetivo será generar un modelo digital del terreno y un modelo digital de superficie que posteriormente, se podrá visualizar en forma de escenario tridimensional, utilizando para ello el programa de diseño 3d **Blender**.

## 2. Software necesario para el desarrollo del taller:

* **[R](https://www.r-project.org/)** y **[Rstudio](https://posit.co/download/rstudio-desktop/)**
* **[QGIS](https://qgis.org/en/site/)**
* **[Blender](https://www.blender.org/)**

## 3. Datos que se utilizarán en el taller:

Básicamente se van a utilizar datos LiDAR, que pueden descargarse de cualquier geoportal de datos abiertos tales como el visor de descargas del **[ICGC](http://www.icc.cat/appdownloads/)** o el Centro de Descargas del **[CNIG](https://centrodedescargas.cnig.es/CentroDescargas/index.jsp)**, entre otros.

Para el presente taller podéis, descargar los datos desde aquí (fuente de los datos: ICGC):

* [1 fichero LAZ](https://drive.google.com/file/d/11XUt-XEteQU_O-_m4G38X1BKu6lyam8W/view?usp=sharing)
* [conjunto de ficheros LAZ](https://drive.google.com/file/d/161cLiO64T_0-hW5k2JQh1QBFw-gw9FmO/view?usp=sharing)

## 4. Trabajando con datos LiDAR desde R
### 4.1. Configurar el espacio de trabajo

R se trabajará desde el entorno de desarrollo RStudio, que facilita sobremanera la gestión de ficheros, _scripts_, carpetas, proyectos y datos, así como la propia visualización de los mapas, capas o gráficos. Así pues, abriremos RStudio y empezaremos por configurar el entorno de trabajo, bien indicando cual es el **directorio de trabajo** -_setwd()_- o bien, configurando un proyecto desde cero (_.Rproj_). También necesitaremos crear un _script_ de R.

```R
setwd("ruta/de/acceso/a/los/datos")   # establecer la ruta al directorio de trabajo
getwd()   # comprobar la ruta al directorio de trabajo
```

A continuación, se instalará y activará el paquete con el cual se va a trabajar: **lidR**

```R
install.packages("lidR)   # instalación del paquete lidR
library(lidR)   # llamada del paquete lidR
```

### 4.2. Lectura y comprobaciones básicas sobre los ficheros LAS/LAZ
#### 4.2.1. Lectura de un fichero LAS/LAZ

Una vez configurado el punto de partida, el siguiente paso consiste en la lectura del fichero o ficheros LAS/LAZ, y la creación del correspondiente objeto de R que contendrá toda la información. La función del paquete **lidR** que permite la lectura de ficheros LiDAR, es **_readLAS()_**. Como todas las funciones de R, ésta se compone de varios argumentos posibles que pueden configurarse o no, en función de la información que precisemos extraer de los ficheros originales:

```R
# importar 1 fichero las/laz: opción básica
las1 <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz")

# recuperar información básica del objeto recién creado:
print(las1)   # información básica
summary(las1)   # información extendida

# comprobar el sistema de referencia de coordenadas
crs(las1)
epsg(las1)
```
En el caso anterior, la función **_readLAS()_** lee la totalidad del archivo original, y traspasa dicha información al nuevo objeto de R (perteneciente a la clase **LAS**). En ocasiones pero, puede interesar únicamente extraer algunos de los atributos que contiene el fichero LiDAR (valores XYZ, intensidad, clasificación, ...)

![Atributos de un fichero LAS](/image/atributos_las.png)

#### 4.2.2. Importar parte del fichero (I). La selección de atributos: SELECT

Uno de los argumentos que soporta la función básica **_readLAS()_**, es la selección de los atributos que se quieren leer/importar. El argumento en cuestión lleva por nombre, ```select```. Así por ejemplo, puede crearse un nuevo objeto que contenga únicamente, parte los atributos originales:

```r
# seleccionar los atributos a leer/importar
las_xyz <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", select = "xyz")   # xyz
las_clasificado <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", select = "xyzc")   # xyz y clasificación
```
Lógicamente, la cantidad de atributos que se leen/importan en R afectará:

* al tamaño del objeto,
* y a la cantidad de información disponible y, por consiguiente, lo que podamos hacer con este objeto.

![Tamaño del objeto LAS](/image/peso_las.png)

En los ejemplos anteriores se han importado las coordenadas XY y el valor de Z en el primer caso (xyz), y se ha añadido el atributo de clasificación en el segundo (c). A continuación, se muestran algunas de las abreviaciones del resto de atributos posibles:

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

#### 4.2.3. Importar parte del fichero (II). La selección de puntos: FILTER

Además de escoger qué atributos se van a leer, también es posible seleccionar únicamente una parte de las geometrías que conforman la nube de puntos LiDAR. Para ello, se utilizará el argumento ```filter``` dentro de la función **_readLAS()_**.

```r
# seleccionar los puntos a importar, en base a sus características:
las_xyz_fr <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz",
                   select = "xyz",
                   filter = "-keep_first")   # xyz y primer rebote

las_xyz_ground <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz",
                      select = "xyz",
                      filter = "-keep_class 2")   # xyz y clasificado como suelo

las_xyz_1k_2k <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz",
                          select = "xyz",
                          filter = "-keep_z 1000 2000")   # xyz comprendidos entre 1000 y 2000 metros
```
Para comprobar las características de cada uno de los objetos generados, basta con ejecutar las funciones **_print()_** para obtener una versión simplificada, o **_summary()_** para obtener más detalles. Con relación a la clasificación presente en una nube de puntos LiDAR, la referencia es la especificación de la ASPRS (_The American Society for Photogrammetry & Remote Sensing_):

![Clasificación de puntos lidar](/image/las_classes.png)

Para consultar todas las posibilidades que admite el argumento ```filter```, basta con ejecutar la siguiente función:

```r
readLAS(filter = "-help")
```

#### 4.2.4. Validar los objetos creados con la función readLAS()

La validación de los objetos creados con la función readLAS() o dicho de otro modo, la confirmación que los datos son válidos para su procesamiento y para generar, por ejemplo, un modelo digital del terreno, pasa por comprobar si estos objetos o datos se ajustan a las especificaciones emitidas por la ASPRS. La función encargada de dicha validación es **_las_check()_:**

```r
las_check(las1)
las_check(las_xyz_ground)
```
Uno de los problemas que, de manera frecuente, afectan o pueden afectar a los datos LiDAR es la presencia de puntos duplicados. La función **_las_check()_** informa de la presencia de estos puntos, lo que permite que puedan ser eliminados del objeto con el cual se va a trabajar.

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
Además, durante el proceso de validación se informa también de la presencia o no de puntos etiquetados como ```withheld```. Estos, son puntos que se han etiquetado de este modo porqué no son confiables, y no deberían ser tomados en cuenta para ningún tipo de análisis:

```r
  - Checking degenerated ground points... ✓
  - Checking attribute population... ✓
  - Checking gpstime incoherances ✓
  - Checking flag attributes...
    🛈 1496514 points flagged 'withheld'
  - Checking user data attribute... ✓
```
Por lo general, se tata de puntos que son considerados errores o ruido. Frente a la presencia de este tipo de puntos etiquetados como 'withheld', pueden adoptarse varias estrategias:

* Filtrar los puntos etiquetados como 'withheld' del objeto, antes de proceder a generar productos derivados.
* En algunos casos, si no se tata de una gran cantidad de puntos, cabe la posibilidad de valorar si parte de estos pueden integrarse en futuros análisis o no.
* Y si la cantidad de puntos no confiables es muy alta, convendria además de no tomarlos en cuenta, investigar su origen y notificar esta cuestión al proveedor de los datos.

En cualquier caso, para empezar conviene solucionar el primer error o aviso que lanza la función **_las_check()_**: la presencia de **puntos duplicados**.

#### 4.2.5. Eliminar los puntos duplicados en un objeto LAS

Para eliminar los puntos que están duplicados en una nube de puntos, basta con utilizar la función _**filter_duplicates()**_. El nuevo objeto que se va a generar, va a estar libre de estos duplicados. Esta situación puede comprobarse utilizando la función de validación de datos.

```r
las1_nodup <- filter_duplicates(las1)   # eliminación de puntos duplicados
las_check(las1_nodup)   # validación del nuevo objeto
```
En el resultado que muestra la consola, se aprecia que los 1943 puntos duplicados ya no están presentes en el nuevo objeto:

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

En el contexto de este taller, para deshacernos de estos puntos no confiables, puede utilizarse una doble vía:

* o bien se utiliza una función especifica para el filtrado de puntos,
* o bien se genera de nuevo un objeto LAS con **_readLAS()_**, pero esta vez, utilizando el argumento ```filter``` para desechar estos puntos desde un buen inicio:

```r
# crear nuevamente el objeto LAS, aplicando un filtro durante su lectura
las1_nowithheld <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", filter = "-drop_withheld")   # lectura
print(las1_nowithheld)   # resumen
las_check(las1_nowithheld)   # validación

# filtrar el objeto anteriormente creado, que contiene los puntos etiquetados como ruido
las1_filtrado <- filter_poi(las1_nodup, Classification != 7)   # filtrado
print(las1_filtrado)   # resumen
las_check(las1_filtrado)   # validación
```

Sea cual sea el método aplicado, se acabará por llegar al mismo punto:

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
**Nota**: no siempre ambos sistemas, pueden aplicarse indistintamente:

```r
# error!
las_xyz <- readLAS("datos_lidar/1_fichero_laz/LIDARCATv02ls12f360716ed02.laz", select = "xyz")   # lectura
# Warning message:
# There are 1496514 points flagged 'withheld'.

las1_nowithheld <- filter_poi(las_xyz, Classification != 7)   # filtrado con error!
# Error in eval(expr, data, expr_env) : object 'Classification' not found
```
La cantidad de atributos que se importen inicialmente para la creación de un objeto de la clase LAS, puede determinar en cierto modo, las funciones que vayamos a poder aplicar posteriormente.

### 4.3. Generar productos derivados del objeto LAS
#### 4.3.1. Modelos digitales del terreno

Por defecto, el algoritmo **_rasterize_terrain()_** utiliza, para la generación de un modelo digital del terreno, únicamente aquellos puntos que estén clasificados como suelo (2) y agua (9). Así pues bastará con definir la resolución del modelo, y el algoritmo de interpolación a utilizar, sin necesidad de aplicar préviamente la función **_filter_poi()_** en conjunción con el argumento ```Classification``` para seleccionar los puntos de interés:

```r
suelo <- filter_poi(las1_filtrado, Classification == c(2,9))
```
A continuación se muestran tres procesos de interpolación para generar una superficie continua de valores:

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
A diferencia de la función anterior, **_rasterize_canopy()_** utilizará el primer rebote almacenado para generar un modelo digital de superficie. Y del mismo modo que sucedía en el caso de la función **_rasterize_terrain()_**, en esta ocasión también disponemos de varios algoritmos para llevar a cabo la interpolación: **p2r()**, **dsmtin()**, **pitfree()**.  

##### Red de triangulos irregulares (TIN)
```r
# MDS TIN
mds_tin <- rasterize_canopy(las1_nowithheld , res = 0.5, algorithm = dsmtin())
lidR::plot(mds_tin)
lidR::plot_dtm3d(mds_tin)
```
##### Algoritmo PITFREE
```r
# MDS PITFREE
mds_pitfree <- rasterize_canopy(las1_nowithheld, res = 0.5, algorithm = pitfree())
lidR::plot(mds_pitfree)
lidR::plot_dtm3d(mds_pitfree)
```
##### Algoritmo p2r
```r
# MDS p2r
mds_pitfree <- rasterize_canopy(las1_nowithheld, res = 0.5, algorithm = pitfree())
lidR::plot(mds_pitfree)
lidR::plot_dtm3d(mds_pitfree)
```

![DSM2](/image/dsm2.png)

## 5. Visualizaciones 3D en R

Además de la función **_plot_dtm3()_** propia del paquete lidR, existen en R otras opciones para visualizar datos en 3D, como por ejemplo utilizando el paquete **rayshader**.

### 5.1. Trabajando con el paquete rayshader
Como siempre en R, en caso que sea necesario, hay que instalar y llamar el paquete **rayshader**,:

```r
install.packages("rayshader")
library(rayshader)
```

A continuación, será preciso convertir el objeto raster (perteneciente a la clase SpatRaster) en una matriz, con ayuda de la función **_raster_to_matrix()_**:
```r
matriz_mds <- raster_to_matrix(mds_tin)
```

Y a continuación, aplicar una textura a la matriz, y visualizarla en un plot 2D:

```r
# aplicar textura a la matriz, y visualizar en 2D
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

## 6. Trabajando con varias hojas LiDAR

Hasta el momento, se ha visto cómo trabajar con una sola hoja de datos LiDAR. De todos modos, lo normal es trabajar con extensiones de terreno superiores a lo abarca una única hoja de datos LiDAR, por lo que en ocasiones será necesario trabajar varias hojas a la vez. En estos castos, se va a trabajar con otro tipo de objeto del paquete lidR: un objeto de la clase **LAScatalog**.

Para empezar, se deben leer todos los ficheros LiDAR contenidos en una carpeta, y generar el objeto con el cual se va a trabajar:

```r
library(lidR)
library(terra)

setwd("/ruta/a/carpeta/de/datos_lidar/")

# creación de un catálogo
lasCat <- readLAScatalog("hojas_laz/", filter = "-drop_withheld")

class(lasCat)
lasCat   # ver características del catálogo
plot(lasCat)   # visualizamos la disposición de hojas LiDAR

las_check(lasCat)   # validación del catálogo
```
Una vez agrupadas todas las hojas LiDAR en un objeto de tipo **LAScatalog**, será el momento de definir los fragmentos, o teselas en las que se dividirá el catálogo generado. Cierto es que se puede procesar el catálogo hoja a hoja, siguiendo la extensión de las hojas originales, o bien con ayuda de las funciones **_opt_chunk_size()_** y **_opt_chunk_buffer()_**, definir un nuevo tamaño de tesela y la zona de solape entre las distintas hojas, a fin de evitar las posibles y temidas líneas de separación entre hojas.

```r
# definición del tamaño de las hojas o teselas, y el solape
opt_chunk_size(lasCat) <- 1000
opt_chunk_buffer(lasCat) <- 50

# y visualizamos la nueva disposición de hojas
plot(lasCat, chunk_pattern = TRUE)
```
A continuación, se puede procesar el objeto **lasCat**, para generar un modelo digital del terreno utilizando para ello el algoritmo **_tin()_**

```r
# generación del modelo digital de elevaciones
lasCat_class <- rasterize_terrain(lasCat, res = 1,algorithm = tin())

plot(lasCat_class)   # gráfico 2D
plot_dtm3d(lasCat_class)   # gráfico 3D
```
## 7. Visualizaciones 3D en Blender

### 7.1. La preparación de los datos
Antes de pasar a trabajar con Blender para recrear un escenario 3D, deben prepararse los datos a fin de aprovechar al máximo, las capacidades de Blender para el modelado del relieve. Ello implicará guardar el modelo digital del terreno o de elevaciones, como una imagen de valores enteros de 16 bits sin signo (UInt16). El propósito es obtener una nueva imagen con valores comprendidos entre 0 y 65535. Esta transformación, puede hacerse des de QGIS, utilizando la calculadora raster, o bien desde R. Para realizar dicha transformación hay que aplicar la fórmula "**_(valor del raster - valor mínimo) / (valor máximo - valor mínimo) * 65535_**":

En el _script_ de R, deberán añadirse las siguientes líneas de código:

```r
# obtener el mínimo y máximo de la imagen
library(terra)
min_val <- minmax(lasCat_class)[1]
max_val <- minmax(lasCat_class)[2]

# aplicar álgebra de mapas
mdtLasCat <- (lasCat_class - min_val) / (max_val - min_val) * 65535

# exportar objeto raster a capa raster
writeRaster(mdtLasCat, "/home/lluis/mdt_lasCat.tif", datatype = 'INT2U')
```
Ahora sí, es el turno de **Blender**.

### 7.2. La configuración de Blender, paso a paso

#### 7.2.1. El espacio de trabajo
* Al abrir el programa, siempre aparecen en la ventana principal, tres elementos básicos: un **cubo**, una fuente de **luz**, y una **cámara**. Se seleccionará el cubo, y se eliminará ya que en su lugar, se utilizará la imagen creada en el paso anterior.
* En el menú Edit > Preferences, hay que activar la extensión o función **Import Images as Planes**.

#### 7.2.2. La importación del modelo digital de superficie o del terreno
* Antes de nada, se dividirá el espacio de trabajo en un doble panel: el **3D Viewport** y el **Shader Editor**. Este último, se utilizará para configurar el escenario mientras que, en el primero, se irá visualizando una muestra del escenario que se está configurando.
* A continuación, hay que importar el modelo digital (en formato de imagen) preparado anteriormente, como un nuevo objeto de tipo **plano** (_plane_). Para ello, basta con dirigirse al menú File > Import > Images as Planes,  seleccionar el fichero **dsm_blender.tif**, y aceptar.
* El plano recién importado, puede visualizarse de tres modos distintos: **sólido**, **material** y **_rendered_**, pinchando sobre les tres iconos en forma de esfera visibles en el ángulo superior derecho de la ventana **3D Viewport**.
* En el apartado **Render**, hay que modificar el parámetro **Render Engine** a **Cycles** y el **Feature Set**, configurarlo como **Experimental**.
* El siguiente paso, consistirá en añadir al plano de trabajo, todos nodos que sean necesarios para poder **"deformar"** el plano y representar el relieve del modelo digital. Para ello, en el apartado **Modifiers**, hay que añadir un nuevo modificador (Add Modifier > Generate > Subdivision surface), activar el botón **Simple** y activar la casilla **Adaptative Subdivision**.
* De nuevo en el apartado **Render** para mejorar la velocidad de representación durante la fase de diseño, se pueden modificar los valores presentes en el parámetro Viewport > Max samples a **512**, y el valor del parámetro Render > Max samples a **30**. Estos valores podrán modificarse segun sea conveniente en cada caso y situación.
* Ahora, en la ventana del **Shader Editor**, se observan tres piezas distintas: el modelo digital (imagen), un algoritmo que configura cómo se va a _renderizar_ el modelo, y el objeto de salida, conectados entre sí.

#### 7.2.3. Convertir el plano en un relieve
* Llegados a este punto, es el momento de aplicar las deformaciones al plano que contiene la imagen, en base a la interpretación de la paleta de colores y de los valores del modelo digital de superficie.
* Se añade un nuevo nodo desde el menú Add > Vector > Displacement. Y se enlaza el conector **Color** de la imagen, al conector **Height** del módulo encargado de configurar el desplazamiento, y el conector **Displacement** de este último, al conector **Displacement** del nodo **Material Output**.

![blender1](/image/blender1.png)

* Al conmutar la vista de material a renderizado, aun no puede apreciarse el desplazamiento que ha sufrido el plano orginal. Para ello, en el apartado **Material > Settings**, deberá modificarse el valor del parámetro **Displacement** de la opción **Bump only** a la opción **Displacement only**. A continuación, hay que ajustar el valor de exageración en el parámetro **Scale** del nodo **Displacement** hasta el valor que resulte más apropiado.

#### 7.2.4. Configurar la iluminación
* Para que el escenario vaya ganando en realismo y se añada el correspondiente sombreado, hay que configurar el foco de luz que ilumina la escena.
* Hay que selecionar el objeto luz (**Light**) en el panel de objetos y, en el apartado de propiedades, se pueden configurar los parámetros de localización, rotación, o escalado, por ejemplo. Los valores de **Rotation** pueden configurarse del siguiente modo (x = 0, y = 45, z = 135). Estos valores también pueden configurarse de manera manual, seleccionando y arrastrando el punto de luz visible en el escenario.
* A continuación, hay que activar el apartado Data (bombilla de color verde), y escoger el sol, como fuente de luz, ajustar la potencia a 4.5, y el ángulo a **2**. 

#### 7.2.5. Configurar la cámara
* Este elemento es indispensable para poder _renderizar_ el escenario. Para comprobar que es lo que está observando la cámara (es decir, qué parte del escenario se va a _renderizar_ y desde que perspectiva), basta con presionar la tecla 0.
* Se puede configurar la vista del escenario de manera manual y a continuación, activar el menú View > Align View > Align Active Camera to View, y acabar de ajustar manualmente la perspectiva seleccionando la cámara en el panel de objetos y presionando la tecla **G**.
* Para ver una primera renderización del escenario, basta con presionar la tecla **F12**.
* Para configurar un plano zenita, basta con configurar la capa de **Perspective** a **Orthogonal**, presionar la tecla 7 para obtener uan visión desde arriba (top) y a continuación, activar el menú View > Align View > Align Active Camera to View. De nuevo en las propiedades de la cámara, se puede ajustar la vista con el parámetro **Orthographic Scale**.

#### 7.2.6. Añadir una paleta de paleta de colores
* Para llevar a cabo esta operación, existen dos posibilidades (como mínimo!): diseñar una paleta de colores en Blender, o bien hacer el trabajo en QGIS, por ejemplo.
* Con QGIS abierto, hay que cargar el modelo digital del terreno con el cual se está trabajando, y aplicar una paleta de colores predefinida como por ejemplo las que hay accesibles en el catálogo **cpt-city**. Una vez aplicada la paleta escogida al mdt, hay que exportar la capa(o capas), como una imágenen 'renderizada', para que se guarde el estilo aplicado.
* De nuevo en Blender, dentro del panel del Shader Editor, se añadirán tantas texturas de imagen como paletas de color se hayan preparado. En cada nuevo nodo de textura de imagen se abrirá una de las nuevas imágenes generadas en QGIS, y deberá conectarse al nodo **PrincipledBSDF** mediante el conector Color > Base color.
* Por lo que respecta a la cámara, antes de renderizar la nueva escena, si se quiere mantener una cámara zenital y añadir una nueva cámara de perspectiva, puede hacerse desde el menú Add > Camera.
* Add > Converter > Color ramp

![escenario](/image/escenario0.png)

### 7.2.7 Añadir una imágen aérea, y otros elementos sobre el relieve
* Del mismo modo que puede configurarse una paleta de colores, y aplicarla sobre el relieve, también es posible aplicar otros elementos tales como fotografías aéreas, imágenes de satélite, u otras capas que ejerzan de máscara como puede ser el caso de láminas de agua (ríos, lagos, mar, ...). Lo recomendable en estos casos, es configurar todas estas capas utilizando QGIS.
* El el caso particular de la fotografía aérea, pueden obtenerse ortofotografías de la zona que se está trabajando con ayuda del complemento ICGC de QGIS. Se tratará pues de descargar la capa, y aplicar un recorte a la capa utilizando la extensión del mdt.
* En el caso por ejemplo de las láminas de agua, pueden digitalizarse manualmente (o extraerse mediante clasificación, capa de cubiertas del suelo, ...) y rasterizar la capa vectorial utilizando la extensión y resolución de algunas de las capas raster ya existentes. Deberá aplicarse un color a las láminas de agua, y guardar la capa como una imagen renderizada.
* Con el objetivo de aligerar el proceso, en el contexto de este taller, estas capas auxiliares pueden descargarse desde el **[siguiente enlace](https://drive.google.com/file/d/1fKwn9fA_OmXd5dnCQPkxt9KzJNK0JEBz/view?usp=sharing)**.
* En Blender de nuevo, dentro del panel del **Shader Editor**, hay que añadir dos nuevas texturas de imagen o bien sustituir alguna de las creadas inicialmente con la paleta de colores, con la imagen del ortofotomapa y la máscara de los lagos. De lo que se trata es de combinar ambas texturas por lo que entre estos dos nuevos nodos y el nodo relativo al shader, habrá que incorporar un nuevo nodo Add > Color > Mix color y establecer las conexiones. 

![mix_color](/image/mix_color.png)

