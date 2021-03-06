# Hipótesis: Los esports cuentan con un ecosistema sustentable en constante crecimiento.

A lo largo de este proyecto hemos observado el auge de los eSports, así como de la salud que goza en la actualidad en los distintos géneros. Encontramos a los mejores equipos y los mejores jugadores, cuyas hazañas se han transmitido a millones a través de servicios como Twitch. Y posteriormente encontramos una correlación positiva entre los factores más importantes de los eSports. Entonces surge una nueva incógnita.

### ¿Seguirán existiendo los esports o son solo un fenómeno fugaz? 

Para poder responder de una manera más concisa, haremos uso de una prediccón basándonos en series de tiempo, utilizando un [método](https://github.com/beduExpert/Programacion-con-R-Santander/tree/master/Sesion-06/Ejemplo-02) visto durante las sesiones del curso. Para encontrar el desarrollo de las variables de Jugadores, Torneos y ganancias a lo largo del tiempo nos apoyaremos del data frame `join`, el cual debemos recordar es la unión de `GeneralEsportData.csv` e `HistoricalEsportData.csv`:

```R

df <- join %>%          #Agrupamos por fecha la suma de cada campo
  group_by(Date) %>%
  summarise(Jugadores = sum(Players), Torneos = sum(Tournaments), Ganancias = sum(Earnings)) 

df <- df %>% filter( year(Date) >= 2001) #Con la función year() de lubridate y un filtro, conservamos datos a partir de 2001 los cuales presentan continuidad.

```

La predicción se realiza con una función, que obtiene 3 entradas:

- Dato: Columna del data frame sobre la que se realizará la predicción
- Inicio: Fecha de inicio de la serie de tiempo en años desde 2001.
- Fin: Fecha de término de la serie de tiempo en años hasta 2020.

```R
Serie_tiempo <- function(Dato,inicio,fin){
  
  Dato.ts <- ts(Dato, start = c(inicio,1), end =  c(fin,12), freq = 12)        #Se crea la serie de tiempo con los datos de entrada y una frecuencia de 12 meses.
  
```  

El siguiente paso es crear modelo, aunque este no se ajustará del todo, porque en los residuales aún quedarán autocorrelaciones diferentes de cero:
  
```R  

  Time <- 1:length(Dato.ts)                                          
  Imth <- cycle(Dato.ts)                                             
  Dato.lm <- lm(log(Dato.ts) ~ Time + I(Time^2) + factor(Imth))       
  
  plot(resid(Dato.lm), type = "l", main = "", xlab = "", ylab = "")   
  title(main = "Serie de residuales del modelo de regresión ajustado",
        xlab = "Tiempo",
        ylab = "Residuales")
```        
 
Posteriormente se busca encontrar el mejor modelo ARMA(p, q) considerando el AIC (Akaike Information Criterion). El ajuste se realiza para la serie de tiempo de los residuales del ajuste anterior:
        
```R
  
  best.order <- c(0, 0, 0)
  best.aic <- Inf
  for(i in 0:2)for(j in 0:2){
    model <- arima(resid(Dato.lm), order = c(i, 0, j))
    fit.aic <- AIC(model)
    if(fit.aic < best.aic){
      best.order <- c(i, 0, j)
      best.arma <- arima(resid(Dato.lm), order = best.order)
      best.aic <- fit.aic
    }
  }
  
  best.order
  
  
  acf(resid(best.arma), main = "")
  title(main = "Serie de residuales del modelo ARMA ajustado",
        sub = "Serie de residuales del modelo de regresión ajustado a los datos")
  
```  

Las predicciones pueden ser mejoradas con un modelo "más adecuado" :

  
```R  
  
  new.time <- seq(length(Dato.ts)+1, length = 36)
  new.data <- data.frame(Time = new.time, Imth = rep(1:12, 3))
  predict.lm <- predict(Dato.lm, new.data)
  predict.arma <- predict(best.arma, n.ahead = 36)
  Dato.pred <- ts(exp(predict.lm + predict.arma$pred), start = 2021, freq = 12)
  
```

Finalmente establecemos un filtro para las etiquetas de nuestros gráficos resultantes en las distintas variables:

```R
  
  if(all(Dato == df$Torneos)){                                      #All() compara elemento a elemento la igualdad
    Titulo <- "Total de torneos de Esports por mes"
    ylabel <- "Torneos totales"
  }
  if(all(Dato == df$Jugadores)){
    Titulo <- "Total de participantes en torneos de Esports por mes"
    ylabel <- "Participantes totales"
  }
  
  if(all(Dato == df$Ganancias)){
    Titulo <- "Total de ganancias de Esports por mes"
    ylabel <- "Ganancias totales"
  }
  
```

Para graficar nos apoyamos de `ts_ggplot()`, una función del paquete `tsbox` para graficar series de tiempo en `ggplot`:
  
```R  
  graph <- ts_ggplot(Evolucion = Dato.ts, Prediccion = Dato.pred)+                                
    theme_tsbox() + scale_color_tsbox()+
    labs(title= Titulo , subtitle = "Serie mensual desde Enero de 2001",
         x="Años", y= ylabel, caption="Predicción de 36 meses")+
    theme(plot.title = element_text(hjust = 0.5),plot.subtitle = element_text(hjust = 0.5))+
    geom_line(size=1)
  
  return(graph)
  
}

```

Llamamos a las funciones con las columnas que nos interesan en el periodo adecuado:

```R

(A1 <- Serie_tiempo(df$Torneos,2001,2020))
(A2 <- Serie_tiempo(df$Jugadores,2001,2020))
(A3 <- Serie_tiempo(df$Ganancias,2001,2020))
   
```

### Resultados

Los gráficos devueltos por la función son los siguientes:

<p align="center">
<img src="../../Imágenes/Proyecto6.1.jpeg">
</p>

<p align="center">
<img src="../../Imágenes/Proyecto6.2.jpeg">
</p>

<p align="center">
<img src="../../Imágenes/Proyecto6.3.jpeg">
</p>

<p align="center">
<img src="../../Imágenes/Proyecto6.4.jpeg">
</p>

<p align="center">
<img src="../../Imágenes/Proyecto6.5.jpeg">
</p>

<p align="center">
<img src="../../Imágenes/Proyecto6.6.jpeg">
</p>

<p align="center">
<img src="../../Imágenes/Proyecto6.7.jpeg">
</p>

<p align="center">
<img src="../../Imágenes/Proyecto6.8.jpeg">
</p>

<p align="center">
<img src="../../Imágenes/Proyecto6.9.jpeg">
</p>

### Conclusión

Con base en las predicciones obtenidas, y también en el desarrollo de los eSports a lo largo de los años, podemos concluir que los eSports no son un fenómeno fugaz. Los eSports son una tendencia cada vez más creciente que ha sabido sacar provecho de sus seguidores y participantes por igual. A pesar de haber enfrentado dificultades en sus comienzos y una leve caída a causa del COVID-19, nos encontramos ante un medio que mueve masas, que interactúa con sus consumidores y que busca incorporarse al mundo de los deportes tradicionales como un nuevo estándar competitivo.
