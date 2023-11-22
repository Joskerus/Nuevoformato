---
title: 'Taller Día 2 - Introducción a la analítica de brotes'
author: "Anne Cori, Natsuko Imai, Finlay Campbell, Zhian N. Kamvar, Thibaut Jombart,José
  M. Velasco-España, Cándida Díaz-Brochero, Zulma M. Cucunubá"
date: "2022-10-25"
output:
  html_document: default
  pdf_document: default
image: null
licenses: "CC-BY"
editor_options:
  markdown:
    wrap: 72
teaching: 90
exercises: 4
---

:::::::::::::::::::::::::::::::::::::: questions 

### Pregunta introductoria 

- ¿Cómo modelar y analizar un brote?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

# Objetivos 

Al final de este taller usted podrá:

- Identificar los parámetros necesarios en casos de  transmisión de enfermedades infecciosas de persona a persona. 

- Estimar e interpretar la tasa de crecimiento y el tiempo en que se duplica la epidemia. 

- Estimar el intervalo serial a partir de los datos pareados de individuos infectantes/ individuos infectados.

- Estimar e interpretar el número de reproducción instantáneo de la epidemia

- Estimar la tasa de letalidad (CFR) 

- Calcular y graficar la incidencia

::::::::::::::::::::::::::::::::::::::::::::::::




## Tiempos de ejecución

Explicación del taller (5 minutos)

Realizar taller (100 minutos taller + 5 minutos imprevistos)

- [1. Preparación (3 minutos ejecución + 3 minutos solución)](#sección-1)

- [2. CFR (3 minutos explicación + 5 minutos ejecución + 2 minutos solución)](#sección-2)

- [3. Incidencia (4 minutos de explicación + 9 minutos de ejecución + 7 minutos solución y reflexión)](#sección-3)
  
  - [3.1. Curva de incidencia diaria (3min ejecución + 4min reflexión)](#sección-3.1)
  
  - [3.2. Cálculo de la incidencia semanal (6 minutos ejecución + 3 minutos de reflexión)](#sección-3.2)

- [4. Tasa de crecimiento (3 minutos de explicación + 18 minutos lectura y ejecución + 14 minutos reflexión)](#sección-4)

  - [4.1. Modelo log-lineal  (5 minutos lectura + 6 minutos de reflexión)](#sección-4.1)

  - [4.2. Modelo log-lineal con datos truncados (5 minutos lectura + 5 minutos ejecución y reflexión)](#sección-4.2)

  - [4.3. Tasa de crecimiento y tiempo de duplicación: extracción de datos (5 minutos lectura y ejecución + 6 minutos solución y reflexión)](#sección-4.3)
  
- [6. Intervalo serial y Rt (5 minutos explicación + 7 minutos lectura y ejecución + 8 minutos reflexión)](#sección-6)

  - [6.1. Estimación del intervalo serial (SI)](#sección-6.1)
  
  - [6.2. Estimación de la transmisibilidad variable en el tiempo, R(t)](#sección-6.2)

Discusión 30 minutos

## Introducción

Un nuevo brote de virus del Ébola (EVE) en un país ficticio de África occidental


#### Conceptos básicos a desarrollar

En esta práctica se desarrollarán los siguientes conceptos:


- Transmisión de enfermedades infecciosas de persona a persona 

- Número reproductivo básico 

- Número reproductivo efectivo 

- Probabilidad de muerte (IFR, CFR) 

- Intervalo serial 

- Tasa de crecimiento 

- Incidencia

## 1. Preparación {#sección-1}


#### Preparación previa

Antes de comenzar, recuerde crear el proyecto en R `RProject`. Este paso no solo le ayudará a cumplir con las buenas prácticas de programación en R, sino también a mantener un directorio organizado, permitiendo un desarrollo exitoso del taller.

Puede descargar la carpeta con los datos y el proyecto desde [Carpetas de datos](https://drive.google.com/drive/folders/1T0uZ2FNhwFAnFcCNxfLX8V6Ir3IsJO6y?usp=sharing) . Ahí mismo encontrará un archivo .R para instalar las dependencias necesarias para este taller.

#### Cargue de librerías: 

Cargue las librerías necesarias para el análisis epidemiológico y para análisis de incidencia y contactos. Los datos serán manipulados con tidyverse que es una colección de paquetes para la ciencia de datos.


```r
library(tidyverse) # contiene ggplot2, dplyr, tidyr, readr, purrr, tibble
library(readxl) # para leer archivos Excel
library(binom) # para intervalos de confianza binomiales
library(knitr) # para crear tablas bonitas con kable()
library(incidence) # para calcular incidencia y ajustar modelos
library(EpiEstim) # para estimar R(t)
```


#### Cargue de bases de datos

Se le ha proporcionado la siguiente base de datos de casos `directorio_casos` y datos de contacto `contactos`:

`directorio_casos`: una base de datos de casos que contiene información de casos hasta el 1 de julio de 2014; y

`contactos_20140701.xlsx`: una lista de contactos reportados por los casos hasta el 1 de julio de 2014. “infectante” indica una fuente potencial de infección y “id_caso” con quién se tuvo el contacto.

Para leer en R, descargue estos archivos y use la función `read_xlsx` del paquete `readxl` para importar los datos y la función `read_rds` de `tidyverse`. Cada grupo de datos importados creará una tabla de datos almacenada como el objeto `tibble.`


```r
directorio_casos <- read_rds("files/directorio_casos.rds")
```



```r
contactos <- read_excel("files/contactos_20140701.xlsx", na = c("", "NA"))
```


#### Estructura de los datos

Explore la estructura de los datos. Para esto puede utilizar `glimpse` de `tidyverse` el cual nos proporciona una visión rápida y legible de la estructura interna de nuestros conjuntos de datos.


```r
glimpse(contactos)
```

```{.output}
Rows: 60
Columns: 3
$ infectante <chr> "d1fafd", "f5c3d8", "0f58c4", "f5c3d8", "20b688", "2ae019",…
$ id_caso    <chr> "53371b", "0f58c4", "881bd4", "d58402", "d8a13d", "a3c8b8",…
$ fuente     <chr> "otro", "otro", "otro", "otro", "funeral", "otro", "funeral…
```


Como puede observar contactos tiene 3 columnas (variables) y 60 filas de datos. En un rápido vistazo puede observar que la columna fuente (fuente del contagio) puede contener entre sus valores otro o funeral. Así como que las tres variables están en formato `caracter`.


```r
glimpse(directorio_casos)
```

```{.output}
Rows: 166
Columns: 11
$ id_caso                  <chr> "d1fafd", "53371b", "f5c3d8", "6c286a", "0f58…
$ generacion               <dbl> 0, 1, 1, 2, 2, 0, 3, 3, 2, 3, 4, 3, 4, 2, 4, …
$ fecha_de_infeccion       <date> NA, 2014-04-09, 2014-04-18, NA, 2014-04-22, …
$ fecha_inicio_sintomas    <date> 2014-04-07, 2014-04-15, 2014-04-21, 2014-04-…
$ fecha_de_hospitalizacion <date> 2014-04-17, 2014-04-20, 2014-04-25, 2014-04-…
$ fecha_desenlace          <date> 2014-04-19, NA, 2014-04-30, 2014-05-07, 2014…
$ desenlace                <chr> NA, NA, "Recuperacion", "Muerte", "Recuperaci…
$ genero                   <fct> f, m, f, f, f, f, f, f, m, m, f, f, f, f, f, …
$ hospital                 <fct> Military Hospital, Connaught Hospital, other,…
$ longitud                 <dbl> -13.21799, -13.21491, -13.22804, -13.23112, -…
$ latitud                  <dbl> 8.473514, 8.464927, 8.483356, 8.464776, 8.452…
```

En el caso del directorio de casos encuentra: 

- El identificador `id_caso` al igual que en contactos 

- La generación de infectados (cuantas infecciones secundarias desde la fuente hasta el sujeto han ocurrido) 

- La fecha de infección 

- La fecha de inicio de síntomas 

- La fecha de hospitalización 

- La fecha del desenlace que como se puede observar en la siguiente variable puede tener entre sus opciones NA (no hay información hasta ese momento o no hay registro), recuperación y muerte 

- La variable género que puede ser `f` de femenino o `m` de masculino 

- El lugar de hospitalización, en la variable hospital 

- Y las variables longitud y latitud

Note que las fechas ya están en formato fecha (`Date`).

## 2. CFR {#sección-2}


### Probabilidad de muerte en los casos reportados (`CFR`, por Case Fatality Ratio)


```r
table(directorio_casos$desenlace, useNA = "ifany")
```

```{.output}

      Muerte Recuperacion         <NA> 
          60           43           63 
```

::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 1  

Calcule la probabilidad de muerte en los casos reportados (`CFR`) tomando el número de muertes y el número de casos con resultado conocido del objeto directorio_casos. 


```r
numero_muertes <-  #COMPLETE

numero_casos_resultado_conocido <- sum(directorio_casos$desenlace %in% c("Muerte", "Recuperacion")) 

CFR <- #COMPLETE / COMPLETE
```


```{.output}
[1] 0.5825243
```
:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Solución 1  


```r
numero_muertes <- sum(directorio_casos$desenlace %in% "Muerte") 

numero_casos_resultado_conocido <- sum(directorio_casos$desenlace %in% c("Muerte", "Recuperacion")) 

CFR <- numero_muertes / numero_casos_resultado_conocido

print(CFR)
```

```{.output}
[1] 0.5825243
```

```r
#Otra posible solución.

# numero_muertes <- directorio_casos %>%
#   filter(desenlace == "Muerte") %>%
#   tally()
# 
# numero_casos_resultado_conocido <- directorio_casos %>%
#   filter(desenlace %in% c("Muerte", "Recuperacion")) %>%
#   tally()
# 
# CFR <- numero_muertes$n / numero_casos_resultado_conocido$n
```
 
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

Luego, determine el CFR con sus intervalos de confianza utilizando la función `binom.confint`.

La función `binom.confint` se utiliza para calcular intervalos de confianza para una proporción en una distribución binomial, que es por ejemplo cuando tenemos el total de infecciones con desenlace final conocido (recuperado o muerte). Esta función pide tres argumentos: 1) el número de muertes; 2) el número total de casos con desenlace final conocido, sin importar que hayan muerto o se hayan recuperado, pero no cuenta los datos con `NA`; 3) el método que se utilizará para calcular los intervalos de confianza, en este caso "`exact`" (método Clopper-Pearson).
::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 2  

```r
CFR_con_CI <- binom.confint(COMPLETE, COMPLETE, method = "COMPLETE") %>%
  kable(caption = "**¿QUE TITULO LE PONDRÍA?**")

CFR_con_CI
```


Table: **tasa de letalidad con intervalos de confianza**

|method |  x|   n|      mean|     lower|     upper|
|:------|--:|---:|---------:|---------:|---------:|
|exact  | 60| 103| 0.5825243| 0.4812264| 0.6789504|
:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Solución 2  


```r
CFR_con_CI <- binom.confint(numero_muertes, 
                                       numero_casos_resultado_conocido, method = "exact") %>%
  kable(caption = "**tasa de letalidad con intervalos de confianza**")

CFR_con_CI
```



Table: **tasa de letalidad con intervalos de confianza**

|method |  x|   n|      mean|     lower|     upper|
|:------|--:|---:|---------:|---------:|---------:|
|exact  | 60| 103| 0.5825243| 0.4812264| 0.6789504|
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

## 3. Incidencia {#sección-3}

### 3.1. Curva de incidencia diaria {#sección-3.1}

El paquete `incidence` es de gran utilidad para el análisis epidemiológico de datos de incidencia de enfermedades infecciosas, dado que permite calcular la incidencia a partir del intervalo temporal suministrado (por ejemplo, diario, semanal). Dentro de este paquete esta la función `incidence` la cual tiene varios argumentos: 

1. `dates` contiene una variable con fechas que representan cuándo ocurrieron eventos individuales, como por ejemplo la fecha de inicio de los síntomas de una enfermedad en un conjunto de pacientes. 

2. `interval` es un intervalo de tiempo fijo por el que se quiere calcula la incidencia. Por ejemplo, `interval = 365` para un año. Si no se especifica, por defecto es diario. 

3. `last_date` fecha donde se establecerá un limite temporal para los datos. Por ejemplo, última fecha de hospitalización. Para este tercer argumento, podemos incluir la opción `max` y la opción  `na.rm`. La primera para obtener la última fecha de una variable y la segunda para ignorar los `NA` en caso de que existan. 


Por ejemplo, se podría escribir `last_date = max(base_de_datos$vector_ultima_fecha, na.rm = TRUE)`

Con esta información la función agrupa los casos según el intervalo de tiempo especificado y cuenta el número de eventos (como casos de enfermedad) que ocurrieron dentro de cada intervalo.
::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 3  

Calcule la incidencia diaria usando únicamente el primer argumento de la función `incidence` ¿Qué fecha sería la más adecuada? Tenga en cuenta que se esperaría sea la que pueda dar mejor información, menor cantidad de `NA`.


```r
incidencia_diaria <- incidence(COMPLETE)
incidencia_diaria
```
:::::::::::::::::::::::::::::::::

 El resultado es un objeto de incidencia que contiene el recuento de casos para cada intervalo de tiempo, lo que facilita su visualización y análisis posterior. Como puede observar la función produjo los siguientes datos: 


```{.output}
<incidence object>
[166 cases from days 2014-04-07 to 2014-06-29]

$counts: matrix with 84 rows and 1 columns
$n: 166 cases in total
$dates: 84 dates marking the left-side of bins
$interval: 1 day
$timespan: 84 days
$cumulative: FALSE
```

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Solución 3  


```r
incidencia_diaria <- incidence(directorio_casos$fecha_inicio_sintomas)
incidencia_diaria
```

```{.output}
<incidence object>
[166 cases from days 2014-04-07 to 2014-06-29]

$counts: matrix with 84 rows and 1 columns
$n: 166 cases in total
$dates: 84 dates marking the left-side of bins
$interval: 1 day
$timespan: 84 days
$cumulative: FALSE
```
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

Como resultado de la función se produjo un objeto tipo lista. Este objeto arroja estos datos: tiene `166 casos` contemplados entre los días `2014-04-07` al `2014-06-29` para un total de `84 días`, se menciona que el intervalo es de `1 día`, dado que no se utilizo este parámetro quedo por defecto. Finalmente se menciona "`cumulative`: `FALSE`" lo que quiere decir que no se esta haciendo el acumulado de la incidencia. Son únicamente los casos de ese intervalo, es decir, de ese día en especifico.


Ahora haga una gráfica de la incidencia diaria. 


```r
plot(incidencia_diaria, border = "black")
```

<img src="fig/TallerEbola-rendered-unnamed-chunk-16-1.png" style="display: block; margin: auto;" />


En el `Eje X (Fechas)`: Se puede observar fechas van desde el `7 de abril de 2014` hasta una fecha posterior al `21 de junio de 2014`. Estas fechas representan el período de observación del brote.

En el `Eje Y (Incidencia Diaria)`: La altura de las barras indica el número de nuevos casos reportados cada fecha según el tipo de fecha escogido.

Dado que no se agregó el parámetro `interval` la incidencia quedó por defecto diaria, produciéndose un histograma en el que cada barra representa la incidencia de un día, es decir, los casos nuevos. Los días sin barras sugieren que no hubo casos nuevos para esa fecha o que los datos podrían no estar disponibles para esos días.

A pesar de que hay una curva creciente, hay periodos con pocos o ningún caso. ¿Porque cree que podrían darse estos periodos de pocos a pesar de la curva creciente?


:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Ideas discusión: 

Usualmente el inicio de la transmisión en la fase exponencial, y dependiendo el periodo de incubación y el intervalo serial, se van a ver días sin casos. Eso no significa que la curva no sea creciente. Usualmente, al agrupar por semana ya no se verá la ausencia de casos.

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

### 3.2. Cálculo de la incidencia semanal {#sección-3.2}

Teniendo en cuenta lo aprendido con respecto a la incidencia diaria, cree una variable para incidencia semanal. Luego, interprete el resultado y haga una gráfica. Para escoger la fecha que utilizará como última fecha en el tercer argumento de la función `incidence` ¿Qué fecha sería la más adecuada? Tenga en cuenta que la fecha debe ser posterior a la fecha que se haya escogido como el primer argumento.

::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 4  


```r
incidencia_semanal <- incidence(PRIMER ARGUMENTO,  #COMPLETE
                                SEGUNDO ARGUMENTO, #COMPLETE 
                                TERCER ARGUMENTO)  #COMPLETE
```



```{.output}
<incidence object>
[166 cases from days 2014-04-07 to 2014-06-30]
[166 cases from ISO weeks 2014-W15 to 2014-W27]

$counts: matrix with 13 rows and 1 columns
$n: 166 cases in total
$dates: 13 dates marking the left-side of bins
$interval: 7 days
$timespan: 85 days
$cumulative: FALSE
```

<img src="fig/TallerEbola-rendered-unnamed-chunk-18-1.png" style="display: block; margin: auto;" />

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Solución 4  


```r
incidencia_semanal <- incidence(directorio_casos$fecha_inicio_sintomas, 
                                interval = 7, 
                                last_date = max(directorio_casos$fecha_de_hospitalizacion,
                                              na.rm = TRUE))
incidencia_semanal
```

```{.output}
<incidence object>
[166 cases from days 2014-04-07 to 2014-06-30]
[166 cases from ISO weeks 2014-W15 to 2014-W27]

$counts: matrix with 13 rows and 1 columns
$n: 166 cases in total
$dates: 13 dates marking the left-side of bins
$interval: 7 days
$timespan: 85 days
$cumulative: FALSE
```

```r
plot(incidencia_semanal, border = "black")
```

<img src="fig/TallerEbola-rendered-unnamed-chunk-19-1.png" style="display: block; margin: auto;" />

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

Compare la gráfica de incidencia diaria con la de incidencia semanal. ¿Qué observa? ¿Los datos se comportan diferente? ¿Es lo que esperaba? ¿Logra observar tendencias?

## 4. Tasa de crecimiento {#sección-4}

### 4.1. Modelo log-lineal {#sección-4.1}

#### Estimación de la tasa de crecimiento mediante un modelo log-lineal


Para observar mejor las tendencias de crecimiento en el número de casos se puede visualizar la incidencia semanal en una escala logarítmica. Esto es particularmente útil para identificar patrones exponenciales en los datos.


Grafique la incidencia transformada logarítmicamente:


```r
incidencia_semanal_df <- as.data.frame(incidencia_semanal)

  ggplot(incidencia_semanal_df) + 
  geom_point(aes(x = dates, y = log(counts))) + 
  scale_x_date(date_breaks = "1 week", date_labels = "%d-%b") +
  xlab("Fecha") +
  ylab("Incidencia semanal logarítmica") + 
  theme_minimal()
```

<img src="fig/TallerEbola-rendered-unnamed-chunk-20-1.png" style="display: block; margin: auto;" />
 
  
#### Ajuste un modelo log-lineal a los datos de incidencia semanal {#interpretación-del-modelo}


```r
ajuste_modelo <- incidence::fit(incidencia_semanal)
ajuste_modelo
```

```{.output}
<incidence_fit object>

$model: regression of log-incidence over time

$info: list containing the following items:
  $r (daily growth rate):
[1] 0.04145251

  $r.conf (confidence interval):
          2.5 %     97.5 %
[1,] 0.02582225 0.05708276

  $doubling (doubling time in days):
[1] 16.72148

  $doubling.conf (confidence interval):
        2.5 %   97.5 %
[1,] 12.14285 26.84302

  $pred: data.frame of incidence predictions (12 rows, 5 columns)
```

::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 5  

¿Qué observa en este resultado?

:::::::::::::::::::::::: solution 

## Solución 

`$model`: Indica que se ha realizado una regresión del logaritmo de la incidencia en función del tiempo. Esto implica que la relación entre el tiempo y la incidencia de la enfermedad ha sido modelada como una función logarítmica para entender mejor las tendencias de crecimiento.


`$info`: Contiene varios componentes importantes del análisis:

1. `$r (daily growth rate)` Tasa de crecimiento diaria: `0.04145251` 

La tasa de crecimiento diaria estimada del brote es de `0.0415`. Esto significa que cada día la cantidad de casos está creciendo en un `4.15%` con respecto al día anterior, bajo la suposición de un crecimiento exponencial constante durante el periodo modelado.


Si quisiera acceder a esta información sin ingresar al modelo podría hacerlo con el siguiente código


```r
tasa_crecimiento_diaria <- ajuste_modelo$info$r
cat("La tasa de crecimiento diaria es:", tasa_crecimiento_diaria, "\n")
```

```{.output}
La tasa de crecimiento diaria es: 0.04145251 
```

2. `$r.conf` (confidence interval):  2.5 %  0.02582225   97.5 %  0.05708276

El intervalo de confianza del `95%` para la tasa de crecimiento diaria está entre `0.0258 (2.58%)` y `0.0571 (5.71%)`.

`$doubling` (doubling time in days): 16.72148

3. El tiempo de duplicación estimado del número de casos nuevos es de aproximadamente `16.72 días`. Esto significa que, bajo el modelo actual y con la tasa de crecimiento estimada, se espera que el número de casos de la curva epidémica actual se duplique cada `16.72 días`.

`$doubling.conf` (confidence interval):  2.5 %  12.14285 97.5 % 26.84302

4. El intervalo de confianza del `95%` para el tiempo de duplicación está entre aproximadamente `12.14` y `26.84 días`. Este amplio rango refleja la incertidumbre en la estimación y puede ser consecuencia de la variabilidad en los datos o de un tamaño de muestra pequeño.

`$pred`: Contiene las predicciones de incidencia observada. Incluye las fechas, la escala de tiempo en días desde el inicio del brote, los valores ajustados (predicciones), los límites inferior y superior del intervalo de confianza para las predicciones.

Si quiere conocer un poco más de este componente puede explorarlo con esta función.


```r
glimpse(ajuste_modelo$info$pred)
```

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
 
Antes de continuar ¿Porqué considera más adecuado usar una gráfica semanal para buscar un ajuste de los datos?

¿Porqué calcular la tasa de crecimiento diaria con el ajuste semanal y no con el ajuste diario?

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Ideas para responder:

La tasa de crecimiento diaria se calcula utilizando el ajuste de la incidencia semanal en lugar de la incidencia diaria debido a que los datos diarios pueden ser muy volátiles en los primeros días de la curva exponencial. Esto puede suceder por varias razones:

- Las fluctuaciones naturales, ciclos de informes, retrasos en el reporte y los errores de medición, que pueden no reflejar cambios reales en la transmisión de la enfermedad. 

- Los datos diarios pueden tener más lagunas o inexactitudes. 

- Eventos de superdispersión o las intervenciones de control.

El uso de datos semanales puede suavizar estas fluctuaciones, dando una mejor idea de la tendencia subyacente. Al utilizar una media móvil semanal, se suavizan estas fluctuaciones, lo que proporciona una imagen más clara de la tendencia subyacente. Esto permite mejorar la precisión de la estimación y evitar el sesgo de los días de la semana, así como mejorar el modelo al reducir el número total de puntos, dado que puede ayudar a evitar el sobreajuste y mejorar la generalización del modelo.

Ejemplo: Algunos fenómenos pueden variar sistemáticamente según el día de la semana. Por ejemplo, el número de pruebas de COVID-19 realizadas podría ser menor los fines de semana, lo que podría afectar a la incidencia reportada. Al utilizar una media móvil semanal, se evita este tipo de sesgo.
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::



Grafique la incidencia incluyendo una línea que represente el modelo.

Dos formas de hacerlo

1. Con `plot`

```r
plot(incidencia_semanal, fit = ajuste_modelo)
```

<img src="fig/TallerEbola-rendered-unnamed-chunk-24-1.png" style="display: block; margin: auto;" />



2. Con `ggplot` 


```r
ajuste_modelo_df <- as.data.frame(ajuste_modelo$info$pred) #Predicciones del modelo

ggplot() +
  geom_bar(data = incidencia_semanal_df, aes(x = dates, y = counts),  #Histograma
           stat = "identity", fill = "grey", color = "black") +
  geom_ribbon(data = ajuste_modelo_df, aes(x = dates, ymin = lwr, ymax = upr), alpha = 0.2) + #Intervalo de confianza del ajuste
  geom_line(data = ajuste_modelo_df, aes(x = dates, y = fit), #Línea del ajuste
            color = "blue", size = 1) +
  scale_x_date(date_breaks = "1 week", date_labels = "%d-%b") + #Formato para los ejes
  xlab("Fecha") +
  ylab("Incidencia semanal") +
  theme_minimal()
```

<img src="fig/TallerEbola-rendered-unnamed-chunk-25-1.png" style="display: block; margin: auto;" />



Tras ajustar el modelo log-lineal a la incidencia semanal para estimar la tasa de crecimiento de la epidemia el gráfico muestra la curva de ajuste superpuesta a la incidencia semanal observada. 

Al final del gráfico se puede observar que la incidencia semanal disminuye. 

¿Porqué cree que podría estar pasando esto? ¿Cómo lo solucionaría?

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

Si se grafica por fecha de inicio de síntomas mientras el brote está creciendo, siempre se va a ver un descenso artificial en la curva de la incidencia. Este sólo corresponde al rezago administrativo (del diagnóstico y reporte de casos), pero no indica necesariamente una reducción de la incidencia real.

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

### 4.2. Modelo log-lineal con datos truncados {#sección-4.2}


#### Encontrando una fecha límite adecuada para el modelo log-lineal, en función de los rezagos (biológicos y administrativos).

Dado que esta epidemia es de ébola y la mayoría de los casos van a ser hospitalizados, es muy probable que la mayoría de las notificaciones ocurran en el momento de la hospitalización. De tal manera que podríamos examinar cuánto tiempo transcurre entre la fecha de inicio de síntomas y la fecha de hospitalización para hacernos una idea del rezago para esta epidemia.


```r
summary(as.numeric(directorio_casos$fecha_de_hospitalizacion - directorio_casos$fecha_inicio_sintomas))
```

```{.output}
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   0.00    1.00    2.00    3.53    5.00   22.00 
```

Al restar la fecha de hopsitalización a la fecha de inicio de síntomas podría haber valores negativos. ¿Cual cree que es su significado?


Para evitar el sesgo debido a rezagos en la notificación, se pueden truncar los datos de incidencia. Pruebe descartar las últimas dos semanas. Este procedimiento permite concentrarse en el periodo en que los datos son más completos para un análisis más fiable.

Semanas a descartar al final de la epicurva


```r
semanas_a_descartar <- 2
fecha_minima <- min(incidencia_diaria$dates)
fecha_maxima <- max(incidencia_diaria$dates) - semanas_a_descartar * 7

# Para truncar la incidencia semanal
incidencia_semanal_truncada <- subset(incidencia_semanal, 
                         from = fecha_minima, 
                         to = fecha_maxima) # descarte las últimas semanas de datos

# Incidencia diaria truncada. No la usamos para la regresión lineal pero se puede usar más adelante
incidencia_diaria_truncada <- subset(incidencia_diaria, 
                        from = fecha_minima, 
                        to = fecha_maxima) # eliminamos las últimas dos semanas de datos
```

Ahora utilizando los datos truncados `incidencia_semanal_truncada` vuelva a ajustar y a graficar el modelo logarítmico lineal. Puede emplear el método que más le haya gustado.

::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 6  

Primero monte el modelo

```{.output}
<incidence_fit object>

$model: regression of log-incidence over time

$info: list containing the following items:
  $r (daily growth rate):
[1] 0.05224047

  $r.conf (confidence interval):
          2.5 %    97.5 %
[1,] 0.03323024 0.0712507

  $doubling (doubling time in days):
[1] 13.2684

  $doubling.conf (confidence interval):
        2.5 %   97.5 %
[1,] 9.728286 20.85893

  $pred: data.frame of incidence predictions (10 rows, 5 columns)
```
:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Solución 6  


```r
ajuste_modelo_semanal <- incidence::fit(incidencia_semanal_truncada)
ajuste_modelo_semanal
```

```{.output}
<incidence_fit object>

$model: regression of log-incidence over time

$info: list containing the following items:
  $r (daily growth rate):
[1] 0.05224047

  $r.conf (confidence interval):
          2.5 %    97.5 %
[1,] 0.03323024 0.0712507

  $doubling (doubling time in days):
[1] 13.2684

  $doubling.conf (confidence interval):
        2.5 %   97.5 %
[1,] 9.728286 20.85893

  $pred: data.frame of incidence predictions (10 rows, 5 columns)
```
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

¿Como interpreta estos resultados?

::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 7  

Ahora grafique el modelo.
<img src="fig/TallerEbola-rendered-unnamed-chunk-30-1.png" style="display: block; margin: auto;" />
:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Solución 7  


```r
ajuste_modelo_semanal_df <- as.data.frame(ajuste_modelo_semanal$info$pred) #Predicciones del modelo

incidencia_semanal_truncada_df <- as.data.frame(incidencia_semanal_truncada)


ggplot() +
  geom_bar(data = incidencia_semanal_truncada_df, aes(x = dates, y = counts),  #Histograma
           stat = "identity", fill = "grey", color = "black") +
  geom_ribbon(data = ajuste_modelo_semanal_df, aes(x = dates, ymin = lwr, ymax = upr), alpha = 0.2) + #Intervalo de confianza del ajuste
  geom_line(data = ajuste_modelo_semanal_df, aes(x = dates, y = fit), #Línea del ajuste
            color = "blue", size = 1) +
  scale_x_date(date_breaks = "1 week", date_labels = "%d-%b") + #Formato para los ejes
  xlab("Fecha") +
  ylab("Incidencia semanal") +
  theme_minimal()
```

<img src="fig/TallerEbola-rendered-unnamed-chunk-31-1.png" style="display: block; margin: auto;" />
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::


¿Qué cambios observa?


Observe las estadísticas resumidas del ajuste:


```r
summary(ajuste_modelo_semanal$model)
```

```{.output}

Call:
stats::lm(formula = log(counts) ~ dates.x, data = df)

Residuals:
     Min       1Q   Median       3Q      Max 
-0.73474 -0.31655 -0.03211  0.41798  0.65311 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept) 0.186219   0.332752   0.560 0.591049    
dates.x     0.052240   0.008244   6.337 0.000224 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 0.5241 on 8 degrees of freedom
Multiple R-squared:  0.8339,	Adjusted R-squared:  0.8131 
F-statistic: 40.16 on 1 and 8 DF,  p-value: 0.0002237
```

El modelo muestra que hay una relación significativa entre el tiempo (`dates.x`) y la incidencia de la enfermedad, con la enfermedad mostrando un crecimiento exponencial a lo largo del tiempo. 

### 4.3. Tasa de crecimiento y tasa de duplicación: extracción de datos {#sección-4.3}

#### Estimacion de la tasa de crecimiento 


Para estimar la tasa de crecimiento de una epidemia utilizando un modelo log-lineal, se necesita realizar un ajuste de regresión a los datos de incidencia. Dado que ya tiene un objeto de incidencia truncado y ajustado un modelo log-lineal, puede proceder a calcular la tasa de crecimiento diaria y el tiempo de duplicación de la epidemia.

El modelo log-lineal ajustado proporcionará los coeficientes necesarios para estos cálculos. El coeficiente asociado con el tiempo (la pendiente de la regresión) se puede interpretar como la tasa de crecimiento diaria cuando el tiempo está en días.

Con el modelo ajustado truncado, es hora de realizar la estimación de la tasa de crecimiento. Estos datos los puede encontrar en el objeto ajuste modelo semana, que tiene los datos ajustados de incidencia semanal truncada. 

::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 8  

Por favor escriba el código para obtener los siguientes valores:


```{.output}
La tasa de crecimiento diaria es: 0.05224047 
```

```{.output}
Intervalo de confianza de la tasa de crecimiento diaria (95%): 0.03323024 0.0712507 
```
:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Solución  8 


```r
# Estimación de la tasa de crecimiento diaria
tasa_crecimiento_diaria <- ajuste_modelo_semanal$info$r

cat("La tasa de crecimiento diaria es:", tasa_crecimiento_diaria, "\n")
```

```{.output}
La tasa de crecimiento diaria es: 0.05224047 
```

```r
# Intervalo de confianza de la tasa de crecimiento diaria
tasa_crecimiento_IC <- ajuste_modelo_semanal$info$r.conf

cat("Intervalo de confianza de la tasa de crecimiento diaria (95%):", tasa_crecimiento_IC, "\n")
```

```{.output}
Intervalo de confianza de la tasa de crecimiento diaria (95%): 0.03323024 0.0712507 
```

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

Si no lo recuerda vuelva por pistas a la sección [Ajuste un modelo log-lineal a los datos de incidencia semanal](#interpretación-del-modelo)


Ahora que ya ha obtenido la tasa de crecimiento diaria y sus intervalos de confianza, puede pasar a estimar el tiempo de duplicación.


#### Estimacion del tiempo de duplicación


Esta información también la encontrará calculada y lista para utilizar el objeto `ajuste_modelo_semanal`, que tiene los datos ajustados de incidencia semanal truncada. 

::::::::::::::::::::::::::::::::::::: challenge  

## Desafío 9  

Por favor escriba el código para obtener los siguientes valores:


```{.output}
El tiempo de duplicación de la epidemia en días es: 13.2684 
```

```{.output}
Intervalo de confianza del tiempo de duplicación (95%): 9.728286 20.85893 
```
:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

## Solución 9  


```r
# Estimación del tiempo de duplicación en días
tiempo_duplicacion_dias <- ajuste_modelo_semanal$info$doubling
cat("El tiempo de duplicación de la epidemia en días es:", tiempo_duplicacion_dias, "\n")
```

```{.output}
El tiempo de duplicación de la epidemia en días es: 13.2684 
```

```r
# Intervalo de confianza del tiempo de duplicación
tiempo_duplicacion_IC <- ajuste_modelo_semanal$info$doubling.conf
cat("Intervalo de confianza del tiempo de duplicación (95%):", tiempo_duplicacion_IC, "\n")
```

```{.output}
Intervalo de confianza del tiempo de duplicación (95%): 9.728286 20.85893 
```
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

Si no lo recuerda vuelva por pistas a la sección [Ajuste un modelo log-lineal a los datos de incidencia semanal](#interpretación-del-modelo)


### 6.1. Intervalo serial (SI) {#sección-6.1}

¿Qué es el intervalo serial?

El intervalo serial en epidemiología se refiere al tiempo que transcurre entre el momento en que una persona infectada (el caso primario) comienza a mostrar síntomas y el momento en que la persona que fue infectada por ella (el caso secundario) comienza a mostrar síntomas.

Este intervalo es importante porque ayuda a entender qué tan rápido se está propagando una enfermedad y a diseñar estrategias de control como el rastreo de contactos y la cuarentena. Si el intervalo serial es corto, puede significar que la enfermedad se propaga rápidamente y que es necesario actuar con urgencia para contenerla. Si es largo, puede haber más tiempo para intervenir antes de que la enfermedad se disemine ampliamente.

Para este brote de ébola asumiremos que el intervalo serial tiene una `media` de `8.7 días` con una `desviación estándar` de `6.1 días` que proviene de una distribución `gamma.` En la práctica del día 4 estudiaremos cómo estimar el intervalo serial.


```r
mean_si <- 8.7
std_si <-  6.1
```


### 6.2. Estimación de la transmisibilidad variable en el tiempo, R(t) {#sección-6.2}

Cuando la suposición de que ($R$) es constante en el tiempo se vuelve insostenible, una alternativa es estimar la transmisibilidad variable en el tiempo utilizando el número de reproducción instantánea ($R_t$). Este enfoque, introducido por Cori et al. (2013),  se implementa en el paquete `EpiEstim.` Estima ($R_t$) para ventanas de tiempo personalizadas (el valor predeterminado es una sucesión de ventanas de tiempo deslizantes), utilizando la probabilidad de Poisson.  A continuación, estimamos la transmisibilidad para ventanas de tiempo deslizantes de 1 semana (el valor predeterminado de `estimate_R`):

***


```r
configuracion_rt <- make_config(mean_si = mean_si, # Media de la distribución SI
                                std_si = std_si,   # Desviación estándar de la distribución SI 
                                t_start = 2,     # Día de inicio de la ventana de tiempo
                                t_end = length(incidencia_diaria_truncada$counts)) # Último día de la ventana de tiempo
```



```r
config <- make_config(list(mean_si = mean_si, std_si = std_si))  
# t_start y t_end se configuran automáticamente para estimar R en ventanas deslizantes para 1 semana de forma predeterminada.
```


```r
# use estimate_R using method = "parametric_si"
estimacion_rt <- estimate_R(incidencia_diaria_truncada, method = "parametric_si", 
                            si_data = si_data,
                            config = configuracion_rt)
# Observamos las estimaciones más recientes de R(t)
tail(estimacion_rt$R[, c("t_start", "t_end", "Median(R)", 
                         "Quantile.0.025(R)", "Quantile.0.975(R)")])
```

```{.output}
  t_start t_end Median(R) Quantile.0.025(R) Quantile.0.975(R)
1       2    70  1.262905            1.0483          1.504981
```


Grafique la estimación de $R$ sobre el tiempo:


```r
plot(estimacion_rt, legend = FALSE)
```

<img src="fig/TallerEbola-rendered-unnamed-chunk-41-1.png" style="display: block; margin: auto;" />


***

#### Sobre este documento

Este documento ha sido una adaptación de los materiales originales disponibles en [RECON Learn](https://www.reconlearn.org/)

#### Contribuciones
Autores originales:

- Anne Cori

- Natsuko Imai

- Finlay Campbell

- Zhian N. Kamvar

- Thibaut Jombart


Cambios menores y adaptación a español:

- José M. Velasco-España

- Cándida Díaz-Brochero

- Nicolas Torres

- Zulma M. Cucunubá


::::::::::::::::::::::::::::::::::::: keypoints 

## Puntos clave 

Revise si al final de esta lección adquirió estas competencias:

- Identificar los parámetros necesarios en casos de  transmisión de enfermedades infecciosas de persona a persona. 

- Estimar e interpretar la tasa de crecimiento y el tiempo en que se duplica la epidemia. 

- Estimar el intervalo serial a partir de los datos pareados de individuos infectantes/ individuos infectados.

- Estimar e interpretar el número de reproducción instantáneo de la epidemia

- Estimar la tasa de letalidad (CFR) 

- Calcular y graficar la incidencia

::::::::::::::::::::::::::::::::::::::::::::::::