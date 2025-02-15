Proyecto patrocinado por:

<img src="https://raw.githubusercontent.com/MalagaTechPark/iabd-tfm-2223/main/malaga_tech_park.png" height="60px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://raw.githubusercontent.com/MalagaTechPark/iabd-tfm-2223/main/IABD_400.png" height="60px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://raw.githubusercontent.com/MalagaTechPark/iabd-tfm-2223/main/accenture.svg" height="60px">


# 🎮 KnoxQuack 🦆

## Sistema de recomendación de videojuegos

![pato](https://user-images.githubusercontent.com/116188406/224566327-b77b0702-a616-4753-ad8c-1208481be215.jpg)

[Video explicando el sistema de recomendación](https://drive.google.com/file/d/1ZtnuXhJrD-VZCucKAaIZ9A2WQ-YfnbI6/view?usp=share_link)

## 1. Justificación y Descripción 

_KnoxQuack es proyecto en el cuál se pueden realizar busquedas de información sobre los lanzamientos de videojuegos, de manera que al introducir el nombre de un videojuego, el programa te devolvera la caratula de dicho juego, la información general de este y una pequeña aportación de reseñas positivas y negativas sobre este mismo, esto esta pensado para que el usuario sea consciente si realmente le merece la pena o no de adquirir dicho producto._

##  2. Obtención de los datos, limpieza y descripción 
<img src="https://upload.wikimedia.org/wikipedia/commons/4/48/Metacritic_logo.svg" alt="metacritic_logo" height="60px">

Los datos se han obtenido de la página web [metacritic](https://www.metacritic.com/browse/games/release-date/available/pc/metascore). Hemos 
decidido centrarnos para este proyecto en los juegos de pc y las reviews que realizan los usuarios. Aunque para mostrar los datos de juego 
hemos tomado también el valor de puntuación de la crítica, lo relevante en este proyecto son los datos de reseñas de usuario, de las que se han 
tomado fundamentalmente las puntuaciones. 

Todos los tanto obtenidos como generados del procesamiento y la transformación para este proyecto, así como los scripts de scraping que se 
han utilizado se encuentran en la carpeta `metacritic_scrape` que ejerce las funciones de data lake en nuestro proyecto. 

Para realizar la obtención de los datos, hemos dividido el proceso en dos pasos:
1. Obtenemos los datos de los juegos junto con las urls para las reviews 
2. A partir de las urls que hemos conseguido para las reviews, obtenemos tanto estas como los datos de usuarios

Se ha realizado así para evitar realizar demasiadas peticiones de golpe, que el proceso demorase demasiado o que por algún error falle 
todo a mitad del programa y después de esperar un montón de tiempo te quedes sin nada. A alguien que conozco muy bien le pasó hace poco de hecho
El proceso se ha llevado a cabo con la librería `BeautifulSoup` de `python`.

Para evitar también hacer un proceso de limpieza de datos posterior, se han extremado las precauciones y se ha depurado conciezudamente para 
conseguir que los datos salgan lo más limpios posibles y en un formato correcto para su manipulación. Para ello se han usado funciones para 
formatear fechas al formato usado en España o para detectar los identificadores numéricos, limpiarlos de todo lo que no sea dígito y realizarle
un casteo para guardarlo en el formato debido. 


![Captura desde 2023-03-12 13-45-05](https://user-images.githubusercontent.com/116188406/224564714-a1392716-f145-4a9e-8cd9-695c0a3daf7a.png)


Era necesario configurar las cabeceras de las peticiones, que la url realizaba redirecciones, así que hubo que hacer algunos 
ajustes para sortear ese problema

![Captura desde 2023-03-12 15-12-40](https://user-images.githubusercontent.com/116188406/224564849-095c7703-165e-45ea-a890-0c40b9821a07.png)

Solucionado el problema de las redirecciones ya pudimos iterar sobre las urls cogiendo los datos que buscábamos y almacenándolos 

![Captura desde 2023-03-12 15-15-28](https://user-images.githubusercontent.com/116188406/224564934-88118596-895d-49cc-8863-ad51e7bb0c86.png)

Para los juegos hemos seleccionado los siguientes campos: 
* `game_id` - Identificador del juego tomado de su propio número en la página
* `title` - Titulo del juego 
* `description` - Descripción del juego, está en inglés, del original
* `score` - Puntuación del juego otorgada por la crítica especializada
* `date` - Fecha de lanzamiento 
* `img_url` - url de la imagen en la página para poder mostrarla en la aplicación 

Pudimos haber cogido todos los juegos, pero por precaución seleccionamos 1100 juegos, muestra de datos más que suficiente 
para poder llevar a cabo el algoritmo junto con los 55.000 registros de reviews. Una vez tomados y ordenados se escribieron 
al archivo `games.json`. Tambié se escribe un archivo `reviews_urls.csv` con los datos de id del juego y la url de las reviews
relativas al juego, este archivo será el que lea el script `reviews_metacritic.py` para recoger los datos de las reviews. 

En el script `reviews_metacritic.py` se agregan unas funciones para crear un registro de usuario, el objetivo es comprobar 
si ese usuario existe en el registro, si no existe se le asigna un identificador que se extrae de una pila de números que 
empiezan desde los 10 millones hasta el 0 y se guarda en el registro, si el usuario ya existe simplemente se añade su review 
y listo. 


![Captura desde 2023-03-12 15-34-55](https://user-images.githubusercontent.com/116188406/224565555-d7d0d43e-646a-4fb1-acb7-1254baf12f04.png)

Volvemos a realizar las iteraciones en busca de los datos y en este caso la ejecución demora más, puesto que el script está recogiendo más 
datos que en el primer paso. 

![Captura desde 2023-03-12 15-42-10](https://user-images.githubusercontent.com/116188406/224565617-4addfa61-f681-45a7-8862-d62c4f54f28d.png)

Una vez terminado el proceso, se escriben, de nuevo dos archivos `reviews_data.json` en el que se recogen los datos de las reviews y 
el registro de usuarios `users_data.csv`  con los campso que se muestran a continuación:

Datos de reviews: 
* `username`- Nombre de usuario que realizó la review
* `review_date` - Fecha en la que se realizó la review
* `score`- Puntuación que le dio el usuario 
* `game_id` - Identificador del juego 
* `user_id`- Identificador de usuario

Datos de usuarios:
* `user_id` - Identificador de usuario
* `username` - Nombre de usuario

Estos datos se agregaron posteriormente al cluster de mongodb 


<img src="https://upload.wikimedia.org/wikipedia/commons/9/93/MongoDB_Logo.svg" height="60px">

##  3. Exploración y visualización de los datos 

De los datos almacenados podemos contar los siguientes registros:

* 11000 juegos 
* alrededor de 55000 reviews 
* alrededor de 40000 usuarios 

La información para cada juego se presenta en la página con la información disponible del juego:

![Captura desde 2023-03-12 16-34-37](https://user-images.githubusercontent.com/116188406/224555440-01e87d88-7149-4028-8e76-b732123bc4b5.png)

Para facilitar la exposición también se ha facilitado una tabla con el número de reviews hechas por un único usuario siempre 
que estas sean mayor a 5. Se ha decidido así porque el algoritmo trabaja mucho mejor cuantas más reviews se han hecho aunque 
no siempre da los mismos datos de afinidad ya que hay usuarios que tan solo puntúan los juegos que no le gustan con 0 u otros 
que puntúan con muchos 10. 

![Captura desde 2023-03-12 16-40-06](https://user-images.githubusercontent.com/116188406/224555713-bdafc0cc-fc26-4fce-a58e-d9687e826a72.png)

También se pueden ver directamente en la página las reviews que han hecho esos usuarios y de la exploración, como puede observarse en la 
captura de abajo, hay usuarios que puntuaron varias veces el mismo juego. No se trata en este caso de duplicados sino de reviews distintas
realizadas en fechas diferentes del mismo juego, probablemente por cambios de opinión o errores. Sería interesante arrojar más luz a este 
detalle y puede ser útil para otros modelos de ML pero no era necesario para el desarrollo del sistema de recomendación. 

![Captura desde 2023-03-12 16-40-06](https://user-images.githubusercontent.com/116188406/224555975-b6bcec95-8933-43b3-8190-b3696b6e180a.png)



##  4. Preparación de los datos para los algoritmos de Machine Learning 

Para el desarrollo de este sistema de recomendación hemos usado `spark` en su versíon `3.2.3` Este procedimiento puede verse en el cuaderno 
de [Google Colab](https://colab.research.google.com/drive/1GhP9Fg4NEvbYbyqJa-ggkU5qzY09ti5G) que también se adjunta en este proyecto en la carpeta `recommendation_model` serán necesarios los archivos `games.json` y `reviews_data.json`, disponibles en el datalake `metacritic_scrape`.

El cuaderno viene con todo el proceso de instalación y configuración para que `spark` funcione. Es necesario mencionar que el lenguaje nativo 
de`spark` es `java`:coffee:, por lo que para su uso en `python`:snake: se usa `pyspark`. Para que no de problemas esta librería debe 
instalarse en la misma versión que la de spark. Además también es necesario configurar las variables de entorno con las rutas a donde se encuentran
tanto `spark` como la máquina virtual de `java`, en este caso en su versión 8. También usaremos, la librería de spark `mlib` que viene preparada 
con multitud de algoritmos listos para ser usados. 

![Captura desde 2023-03-12 16-54-49](https://user-images.githubusercontent.com/116188406/224556412-3d224be7-7504-4205-86cf-d35084678ce8.png)

Con `spark` podemos convertir fácilmente y rápidamente nuestros archivos `json` en dataframes que se presentan en formatos tabulares. 

![Captura desde 2023-03-12 17-03-25](https://user-images.githubusercontent.com/116188406/224557015-9d43683b-79d3-48c3-8201-47702988c43e.png)

Podemos visualizar también los tipos de datos que tenemos en nuestro dataframe puede usarse `printSchema()` es importante usar esta función 
porque debemos asegurarnos que todo lo que le pasamos al algoritmo sean datos numéricos. 

![Captura desde 2023-03-12 17-04-11](https://user-images.githubusercontent.com/116188406/224557069-04935de9-8a8f-4f39-9838-fc0b22096dd0.png)

Seleccionamos los campos que vamos a necesitar para pasárselos al algoritmo para entrenamiento, al que le pasaremos los datos de las reviews `user_id`,
`game_id` y `score`

![Captura desde 2023-03-12 17-08-04](https://user-images.githubusercontent.com/116188406/224557358-9f363ebf-bc94-4386-8c0c-09578a2582fe.png)

Por último antes de pasar al entrenamiento nos aseguramos de que todos los datos que tenemos sean numéricos

![Captura desde 2023-03-12 17-10-23](https://user-images.githubusercontent.com/116188406/224557413-195dac18-a549-48cd-a677-82c1ed637183.png)


##  5. Entrenamiento del modelo y resultados 

Para este caso, se ha decidido utlizar un algoritmo de recomendación llamado `Alternating Least Squares` o `ALS` para los amigos. Este algoritmo realiza
como todos, operaciones de productos de matrices para tratar de estimar una puntuación y está pensado específicamente para usuarios, productos 
y puntuaciones. En un experimento realizado hace algunas semanas dio buenos resultados para los datos de puntuaciones de películas de imdb. Este 
algoritmo además es usado por grandes compañías como Netflix o Amazon entre otras. 

También se consideró el algoritmo `a priori` disponible en la librería de `apyori` de `python`, funcionaba bien para datos de cestas de la compra 
pero no hubo tiempo de probarlo, pues la implementación de este sistema se comió gran parte del tiempo de este proyecto. 

Para entrenar el modelo se le pasa el número máximo de iteraciones `maxIter`, parámetro de regularización `regParam`, `userCol` la columna 
que contiene los datos de usuario, `itemCol` la columna que contiene los datos de producto y por último la columna de puntuación, `ratingCol`
si los datos son númericos nuestro modeloy los datos no son aleatorios no dará errores. Esto último es importante según recoge la propia 
[documentación de spark para ALS](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.recommendation.ALS.html) en la 
que asegura que datos no deterministas pueden causar errores. 

![Captura desde 2023-03-12 17-28-55](https://user-images.githubusercontent.com/116188406/224558347-a65de9d8-5eb7-4384-8823-56a19ab3af12.png)

En este caso no tenemos una manera de asegurar que nuestros datos son correctos con datos de test de validación ya que los gustos personales
a fin de cuentas son arbitrarios y ni siquiera muchas veces los propios sistemas de recomendación de las grandes compañías aciertan. En realidad
estos algoritmos se basan casi siempre en la premisa de valorar que los usuarios compren, no tanto que les guste algo más o menos, también 
es importante tenerlo en cuenta. 


Para probar nuestro modelo realizamos una conversión de nuestros datos nombre de juegos a diccionario de python para facilitar su visualización, 
ya que lo que nos llegará en última instancia son los `game_id` junto con `user_id` de las recomendaciones para ese usuario. También realizamos 
un filtrado de juegos populares, ojo, no quiere decir que sean los que más les gustó al público sino los que tienen más puntuaciones. Luego 
ajustamos la columna de `user_id` con el id del usuario cuyas recomendaciones queramos conocer y aplicamos la función `transform` del modelo
para realizar las predicciones. Luego ordenaremos las predicciones por puntuación de mayor a menor y tomaremos solo 20 del total. 


![Captura desde 2023-03-12 17-44-59](https://user-images.githubusercontent.com/116188406/224559274-20271b57-aaa6-49a4-a661-41981d4f28fa.png)



Estas son las puntuaciones que tenía por ejemplo este usuario con 9984689


![Captura desde 2023-03-12 17-39-04](https://user-images.githubusercontent.com/116188406/224558906-21a16497-e535-4d1a-b5a2-03c9f380f1d7.png)

Y estas fueron las recomendaciones. Podemos ver que recomienda juegos similares

![Captura desde 2023-03-12 17-40-04](https://user-images.githubusercontent.com/116188406/224558948-39b5e9b5-3b54-4360-b1dd-5da1072d56c9.png)

Una vez realizado el modelo y comprobado que funciona es hora de guardarlo para integrarlo en nuestra aplicación. Este proceso es tan 
sencillo como ejecutar `model.save("nombre_deseado")`. Posteriormente se usará otra librería diferente para cargarlo. Esto genera una 
carpeta con los datos del modelo entrenado. 


##  6. Integración del modelo en la aplicación 

El modelo que hemos generado en el paso anterior, en nuestro caso, se encuentra en la carpeta `als_model` para cargarlo usamos una librería 
diferente `ALSModel` que puede ejcutar la función `load` para cargar el modelo que se encuentra nuestro proyecto. Para no realizar todo 
el proceso de golpe cada vez que se realizan recomendaciones hemos creado una clase que implementa el sistema de recomendación para 
así poder dejar algunos pasos precargados y solo cargar los que sean necesarios para determinar las recomendaciones de usuario. 

Con este mismo objetivo se ha separado también los juegos más populares `popular_games.csv` y los nombres de los juegos previamente para 
evitar muchas consultas a la base de datos, lo cual ralentiza muchísimo la aplicación. 

La clase utiliza:

* Un constructor para iniciar la sesión de Spark 
* Método para detener la sesión 
* Método para cargar los nombres de los juegos 
* Método para cargar el modelo 
* Método para cargar los juegos populares
* Método para realizar la predicción 
* Método para transformar los datos de salida de nuevo para evitar consultas reiteradas a la base de datos

![Captura desde 2023-03-12 18-02-49](https://user-images.githubusercontent.com/116188406/224560262-743ca77b-845f-4dcc-8992-bae33e5cb5af.png)

Estos serían los pasos a seguir 

![Captura desde 2023-03-12 18-06-10](https://user-images.githubusercontent.com/116188406/224560469-03f6e052-bd59-4015-8273-ccd935d1d613.png)

En la aplicación se cargan previamente:
* La sesión 
* El modelo 
* Los nombres de los juegos 
* Los juegos populares 

Y a la hora de recibir las recomendaciones se cargan:
* El ajuste de los datos
* La predicción o transformación del modelo 
* La transformación de los datos de salida

##  7. Aplicación de Procesamiento del Lenguaje Natural: Chatbot con dialog flow 

Para el apartado del Lenguaje Natural, se ha decidido usar un Chat bot, concretamente dialog flow, añadido este a los datos principales de la aplicación,
acto seguido se mostrará de manera visual como se funciona dicho añadido al proyeto.

<img width="679" alt="image" src="https://user-images.githubusercontent.com/65163077/224585308-6cd6518e-00e1-471c-9aa5-a8fc1d124287.png">

En esta imagen mostraremos lo principal sobre el mantenimiento y uso de Dialog flow de entre ellos distintas secciones:

* Intents: Es donde colocaremos las respuestas y preguntas realizadas por el bot, dandose asi a una pregunta diferentes respuestas en cuestion.

* Entities: Esta sección va destinada a la hora de que una palabra o varias palabras tengan un sentido en comun, dandose asi casos de sinonimos u otras referencias, mediante las entidades podemos sacar un valor clave en la pregunta y el sistema lo detectara automaticamente dando así una de tantas respuestas segun sea lo que se ha fijado.

* Integrations: Utilizada para poder aportar Dialog Flow a las distintas secciones del proyecto y sus exportaciones del mismo.

Ahora bien, iremos por parte principalmente con dos de las secciones ya mencionadas, esas son Intents y Entities:

<img width="316" alt="image" src="https://user-images.githubusercontent.com/65163077/224585881-c119009f-650a-45c8-84b1-d28ed24ed855.png">

En Intents se menciona las intenciones que tiene el chat para poder devolver una respuesta adecuada a la pregunta proporcionada, en este caso tendremos las de por defecto que viene con Dialog, pero se han añadido 4 nuevas, las cuales son:

* Knox - Como archivo principal de saludos del bot e inicio del mismo.

<img width="577" alt="image" src="https://user-images.githubusercontent.com/65163077/224586046-5528a091-b31e-4d7b-a926-8fcb9e5fa855.png">


* Juegos-Accion: En este archivo estan recopilados pequeñas informaciones sobre los juegos de esa misma categoría, añadiendo también algunas respuestas.

<img width="589" alt="image" src="https://user-images.githubusercontent.com/65163077/224586075-f441a631-0951-4356-9825-fd3e0cdd7995.png">


* Juegos-Terror: En este archivo estan recopilados pequeñas informaciones sobre los juegos de esa misma categoría, añadiendo también algunas respuestas.

<img width="594" alt="image" src="https://user-images.githubusercontent.com/65163077/224586107-4ab244e8-0cee-48d4-be8f-a86206812ab5.png">


*Juegos-Rol: En este archivo estan recopilados pequeñas informaciones sobre los juegos de esa misma categoría, añadiendo también algunas respuestas.

<img width="563" alt="image" src="https://user-images.githubusercontent.com/65163077/224586142-d6cf689c-35e0-4daa-92ca-424d1a0c244a.png">

Ahora pasaremos a las Entities:

<img width="598" alt="image" src="https://user-images.githubusercontent.com/65163077/224586201-e828b667-bbd8-45e5-a306-d0b863fb4f06.png">

Dentro de esta tendremos una serie de recopilación de palabras clave, las cuáles sirven para determinar a la hora de preguntar algo al Bot, que este identifique dicha palabra dentro de los demás cuadros de los Intents

Dialog Flow también aparte un pequeño recuadro de pruebas vease como este:

<img width="171" alt="image" src="https://user-images.githubusercontent.com/65163077/224586287-e33f86eb-d71a-45cf-b8e3-399e1dd1d82b.png">

Ahora procederemos a una visual dentro del mismo proyecto:

<img width="519" alt="image" src="https://user-images.githubusercontent.com/65163077/224586339-a7ee81f2-b723-493f-afd3-731faaf6b666.png">

Dentro de la misma página accederemos al apartado del ChatBot, en este veremos el cuadro a la espera de preguntar distintas opciones:

<img width="216" alt="image" src="https://user-images.githubusercontent.com/65163077/224586396-b6296182-d8a3-4267-90a5-089e2c1cd2c5.png">

Si comenzamos con uno de los comandos que sería inicio, este nos devolvera una respuesta cordial.


<img width="180" alt="image" src="https://user-images.githubusercontent.com/65163077/224586443-85bfeb32-6c9d-41ee-908b-ff3b5e5701af.png">

Si le pedimos que nos devuelva los juegos de rol mejores recomendados por Knox-Quack, este nos devolvera una pequeña lista junto con una pequeña recomendación que otra.

<img width="178" alt="image" src="https://user-images.githubusercontent.com/65163077/224586752-9cf8a742-11e3-45bd-b663-35a5a144be1f.png">

Lo mismo pasa con los juegos de accion y de terror en dichos casos

<img width="203" alt="image" src="https://user-images.githubusercontent.com/65163077/224587103-ca097012-aa00-4e8c-aae8-2556cf8c016f.png">



##  8. Aplicación Web: Dockerización y despliegue

Si bien ajustar las cosas para evitar que nuestra aplicación cargara demasiadas cosas desde la base de datos ha sido 
la parte más tediosa, resolver los problemas que pueden plantear las distintas versiones, la falta de disponibilidad en 
máquinas debian de java 8 o las versiones demasiado antiguas que venían en algunas imágenes de docker supusieron un 
día entero de pruebas y errores. La decisión final fue crear una imagen de docker partiendo de una imagen de ubuntu 
jammy, sistema en el que se realizó y funciona esta aplicación perfectamente. Una vez que se entiend docker es sencillo. 

Aunque el Dockerfile quedó al final un poco sucio con tanta depuración realiza:
* Instalación de `python3` con `pip` 
* pipenv - Se iba a usar y se usó en una prueba para obtener un `requirements.txt` fiable de la aplicación 
* streamlit - aunque se instala con `requirements.txt`
* Java 8 
* `Spark` en su versión `3.2.3`
* Definición de las variables de entorno `$JAVA_HOME` y `$SPARK_HOME` 
* Lo necesario también para instalar mongodb (Este paso es necesario para que la aplicación funcione)
* Definición de variables de mongodb(también necesario)
* Se define el entorno virtual en el proyecto 
* Se instalan los `requirements.txt`
* Se expone el puerto de `streamlit`
* Se ejecuta

![Captura desde 2023-03-12 18-20-11](https://user-images.githubusercontent.com/116188406/224563669-bd1229cb-d9b9-4f60-bc31-538ec595e1a8.png)


Para su despliegue en Streamlit Cloud fue ademaś necesario añadir un archivo `packages.txt` con la especificación del paquete de java 8 si no
se hacía Streamlit arroja una `JavaGatewayException`. 

Por último solo es necesario definir la url del cluster de mongodb en el archivo de producción y ponerla como está con `streamlit.secrets`

##  9. Conclusiones

Este proyecto fue diseñado para que los usuarios utilizaran y navegasen por esta interfaz para resolver sus dudas a la hora de buscar datos sobre un juego en especifico, también de saber si valía la pena adquirir dicho juego o no.

Hay muchas mejoras de por medio, tanto la visualización como ciertos aspectos de tema IA, como lo es nuestro ChatBot integrado, sin duda ha sido un proyecto interesante y ciertamente con ganas de progresar en él para ver hasta cuan lejos puede llegar esta idea con esfuerzo diario.

## Construido con 🛠️

## Framework web
<img src="https://upload.wikimedia.org/wikipedia/commons/7/77/Streamlit-logo-primary-colormark-darktext.png" height="60px">

## Lenguajes de programación
<img src="https://upload.wikimedia.org/wikipedia/commons/c/c3/Python-logo-notext.svg" height="60px">

<img src="https://camo.githubusercontent.com/b713d9f05063e22ec2ef142d1ea4f28d4a735e96f12943efb7afc6a1e30298d6/68747470733a2f2f342e62702e626c6f6773706f742e636f6d2f2d444c41614f62634e4b75512f5842596d34737873745a492f41414141414141414151772f693749784e6d45526c53415945474e4848356e582d4f2d6c6373666c5857484377434c63424741732f73313630302f6a617661382e6a7067" height="100px">

## Bases de datos y sistemas de Machine Learning y Big Data

<img src="https://www.pngitem.com/pimgs/m/90-907131_apache-spark-mllib-logo-hd-png-download.png" height="60px">

<img src="https://upload.wikimedia.org/wikipedia/commons/9/93/MongoDB_Logo.svg" height="60px">

<img src="https://upload.wikimedia.org/wikipedia/commons/e/ed/Pandas_logo.svg" height="60px">

## Despliegue 

<img src="https://logos-world.net/wp-content/uploads/2021/02/Docker-Logo.png" height="60px">


## Servicio de despliegue utilizado 🖇️

_El servicio de despliegue usado para esta ocasión es Streamlit, debido a su guía y accesibilidad bien expuesta para el usuario de forma que se pueda subir y desplegar con mayor facilidad._

Link de la aplicación: https://aresonay-knoxquack-app-5jrn6s.streamlit.app/

Link del video de la aplicación: https://youtu.be/wbFd0VLiilk


## Autores ✒️

* Ismael Armada González
* Ángel Serón Márquez
