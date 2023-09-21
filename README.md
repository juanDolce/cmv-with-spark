# cmv-with-spark
### ----- ANTES DE EMPEZAR -----

Para poder utilizar el framework Spark deberemos incluirlo
en la sección de dependencias de Maven de nuestro archivo pom.xml:

```    
     <dependencies>
       ...
       <dependency>
         <groupId>com.sparkjava</groupId>
         <artifactId>spark-core</artifactId>    
         <version>2.9.3</version>
       </dependency>
       ...
     </dependencies>
 ```   

Detalle: en ciertas ocasiones maven no reconoce la dependencia de inmediato, en estos casos hay
que correr el siguiente comando en la terminal local (estando en el repositorio de nuestro proyecto)

```
mvn clean install
```     

y en algunas ocasiones tambien hay que reiniciar el IDE.

  ***POR AHORA CON ESO ES SUFICIENTE***

Llegó el momento de la verdad, decir hola mundo
y construir nuestra primera aplicación del lado del servidor
(la cual denominaremos servidor a secas de ahora en más) con Spark.
El primer paso será construir un main en el que configuraremos:

el puerto en que nuestro servidor escuchará los pedidos HTTP;
la respuesta que dará cuando recibamos un pedido al recurso /,también conocido como la raíz.

```java
import spark.Spark;


public class Main {
 public static void main(String[] args) {
   Spark.port(9000);
   Spark.get("/", (request, response) -> "¡Hola mundo!");
 }
}
```

Una vez que ya esta el main se corre (boton derecho en main -> run 'Main.main()')
Cuando lo corres te tira en la terminal lo siguiente:

```
    SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
    SLF4J: Defaulting to no-operation (NOP) logger implementation
    SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

Pero no hay que darle bola. Cuando corre el main ya esta corriendo el servidor.

Ya con el main corriendo se pueden realizar las consultas, por ejemplo:

```
    http://127.0.0.1:9000/: consulta el recurso raíz del servidor 127.0.0.1 (nuetra propia computadora) en el puerto 9000
    http://localhost:9000/: equivalente al anterior, dado que localhost es un nombre de dominio que sirve de alias a 127.0.0.1
    http://localhost:9000: también equivalente al anterior, dado que el recurso raíz es también el recurso por defecto, y por tanto podemos omitir la /.
```

Si se pone alguna de esas consultas en el navegador va a aparecer una respuesta del tipo 'hola mundo!'


Para realizar cambios en el codigo hay que detener el main. (boton rojo de la terminal de intellij, la misma en la que aparecen esos "errores")



### --- RETORNO DE HTML ---

Para que retorne html hay que modificar la clase main:

```java
import spark.Spark;

public class Main {
 public static void main(String[] args) {
   Spark.port(9000);
   Spark.get("/", (request, response) ->
                     "<h1>¡Hola mundo!</h1>"
                   + "<p>Esta es nuestra primera respuesta HTML</p>");
 }
}

```
    


### --- VISTA DINAMICA ---

¿Y qué sucedería si en lugar de dar una respuesta estática (es decir, que siempre es igual),
quisiéramos dar una respuesta dinámica, que dependa de un parámetro  HTTP (query param)?
Para ello contamos con mensaje como queryParamOrDefault,
que nos permite justamente leer el valor de un parámetro dado.

```java
import spark.Spark;


public class Main {
 public static void main(String[] args) {
   Spark.port(9000);
   Spark.get("/", (request, response) ->
       "<h1>¡Hola " + request.queryParamOrDefault("nombre", "Mundo")+ "!</h1>"
     + "<p>Esta es nuestra primera respuesta HTML</p>");
 }
}
```
    

Si ahora consultamos al recurso raíz como veníamos haciéndolo, obtendremos la siguiente respuesta…

```
    $ curl localhost:9000/
    <h1>¡Hola Mundo!</h1><p>Esta es nuestra primera respuesta HTML</p>
```

 … pero si lo hacemos indicando un valor para el parámetro nombre, obtendremos lo siguiente:

```
    $ curl localhost:9000/?nombre=Feli
    <h1>¡Hola Feli!</h1><p>Esta es nuestra primera respuesta HTML</p>
```

Quizás a esta altura estés pensando “esto de concatenar strings para generar HTML no pinta bien”,
¡y tenés razón! A medida que nuestro HTML se vuelva más complejo, nuestro código Java también lo será.
Acá es donde llega el momento de introducir las plantillas.

### --- PLANTILLAS ---

Cuando lidiamos con código de vista, un problema común es, justamente, lidiar con los componentes
que la conforman: títulos, botones, párrafos, listas, imágenes, casillas de verificación, etc.
No sólo necesitaremos conocer a estos nuevos elementos, sus propiedades y posibilidades,
sino que además los pueden estar expresados en lenguajes totalmente diferentes a los que
utilizamos para construir el dominio (HTML, en nuestro caso particular).

A estas cuestiones se suman dos necesidades siempre vigentes:
Que el código que construyamos sea reutilizable, es decir, que si necesitamos definir dos veces
una pantalla similar, no tengamos que volver a escribir la segunda completamente desde cero;
Que el código que construyamos sea declarativo, es decir, que podamos entenderlo fácilmente
y, tanto al leerlo como al escribirlo, nos permita concentrarnos en el qué representa,
y no en el cómo está compuesto internamente.

Por eso es que la estrategia de concatenar strings sólo servirá para situaciones muy simples y concretas.
Para los demás casos, en general recurriremos a archivos independientes, en un formato híbrido a medio
camino entre HTML y Java, que nos permitirán generar código HTML de forma más sencilla, reutilizable y declarativa.
Estos archivos, denominados plantillas  (o templates, en inglés), están presenten en una gran gama de tecnologías, como por ejemplo:

    * JSP (Java)
    * ERB (Ruby)
    * JSX (JavaScript)
    * Velocity (Java)
    * HAML (Ruby)
    * Pug (JavaScript)
    * Jinja (Python)


### --- HANDLEBARS ---

En nuestro caso, utilizaremos Handlebars, el cual está disponible en varios lenguajes de programación,
Java incluído. Lo primero que haremos será incluirlo a nuestro proyecto:

```
     <dependencies>
       ...
       <dependency>
         <groupId>com.sparkjava</groupId>
         <artifactId>spark-template-handlebars</artifactId>
         <version>2.7.1</version>
       </dependency>
       ...
    </dependencies>
```

Ahora podremos crear un motor de plantillas (template engine)...


```java
import spark.ModelAndView;
import spark.Spark;
import spark.template.handlebars.HandlebarsTemplateEngine;


public class Main {
 public static void main(String[] args) {
   Spark.port(9000);
   HandlebarsTemplateEngine engine = new HandlebarsTemplateEngine();
   ...
 }
}
```
    

...y en lugar de generar un HTML directamente, lo que haremos será:

1-  Devolver un ModelAndView, que no es ni más ni menos que una tupla con el objeto que queremos representar en HTML
y el nombre del archivo plantillas que usaremos;
1.1-    Especificar el modelo (el nombre de la persona, en nuestro caso) como primer argumento;
1.2-    Especificar la plantilla (bienvenida.html.hbs, en nuestro caso) como segundo argumento;
2- Indicar a Spark que esta ruta será servida empleando el motor de plantillas previamente creado.


     
```java
import spark.ModelAndView;
import spark.Spark;
import spark.template.handlebars.HandlebarsTemplateEngine;


public class Main {
 public static void main(String[] args) {
   Spark.port(9000);
   HandlebarsTemplateEngine engine = new HandlebarsTemplateEngine();


   Spark.get("/",
           (request, response) ->
                   new ModelAndView( //1
                       request.queryParamOrDefault("nombre", "Mundo"), // 1.1
                       "bienvenida.html.hbs"), // 1.2
           engine); //2
 }
}
```
    

Finalmente, ahora llegó el turno de crear nuestra plantilla handlebars, que guardaremos
en src/main/resources/templates/bienvenida.html.hbs (tendremos que crear antes los directorios
resources y templates).
El contenido de este archivo será HTML, pero insertando {{this}} en el lugar en donde irá el valor
del modelo (el nombre de la persona):

```  
    <h1>¡Hola {{this}}!</h1>
    <p>Esta es nuestra primera respuesta HTML</p>
```    

¡Y eso es todo! En resumen, ahora cuando Spark atienda el pedido HTTP en /:

Procesará la el bloque de código como siempre, pero éste en lugar de devolver el código HTML
correspondiente, retornará un ModelAndView;
El ModelAndView contendrá lo que queremos representar (el modelo) y la plantilla que se usará
para convertirlo en un HTML;
Spark, antes de enviar una respuesta, llamará al TemplateEngine que creamos y registramos
previamente.
El TemplateEngine buscará a la plantilla correspondiente, la procesará y generará el código HTML,
reemplazando la expresión {{this}} por el valor del modelo.



### --- MVC RECARGADO ---

Ejemplo mas complejo.

¿Y qué pasa si nuestro servidor debe poder responder a diferentes pedidos, cada uno en una ruta distinta?
La respuesta más directa es: ¡agreguemos más rutas en nuestro Main!.

Por ejemplo, si queremos que haya una ruta de bienvenida, otra de despedida, y que la ruta raíz redirija
a la ruta de bienvenida, podríamos hacer lo siguiente:

    
```java
import spark.ModelAndView;
import spark.Spark;
import spark.template.handlebars.HandlebarsTemplateEngine;


public class Main {
 public static void main(String[] args) {
   Spark.port(9000);
   HandlebarsTemplateEngine engine = new HandlebarsTemplateEngine();


   Spark.get("/",
       (request, response) -> {
         response.redirect("/bienvenida");
         return null;
       }); 


   Spark.get("/bienvenida",
       (request, response) ->
         new ModelAndView(
                 request.queryParamOrDefault("nombre", "Mundo"),
                 "bienvenida.html.hbs"),
       engine);


   Spark.get("/despedida",
       (request, response) ->
         new ModelAndView(
                 request.queryParamOrDefault("nombre", "Mundo"),
                 "despedida.html.hbs"),
       engine);
 }
}
```
    

Acá suceden dos cosas novedosas:
        * por un lado, esta vez nuestro main soporta múltiples rutas: /, /bienvenida y /despedida
        * por otro lado, una de ellas (la raíz) en lugar de devolver algo, redirige a otra ruta
         (mediante un código 302 Found) y devuelve null, indicando que esta respuesta no tendrá cuerpo
         (body), es decir, no tiene contenido.

Si además creamos la nueva plantilla para despedidas (src/main/resources/despedida.html.hbs) …

```   
    <h1>¡Adiós {{this}}, que la fuerza te acompañe!</h1>
    <p>Nos veremos de nuevo en otro apunte</p>
```

 … podremos ahora ver en acción a todas nuestras rutas:

* Raíz (observar cómo el primer pedido genera una redirección sin contenido y ésta un nuevo pedido):

```  
    $ curl http://localhost:9000/ -iL
    HTTP/1.1 302 Found
    Date: Fri, 08 Oct 2021 18:42:24 GMT
    Location: http://localhost:9000/bienvenida
    Content-Length: 0
    Server: Jetty(9.4.31.v20200723)


    HTTP/1.1 200 OK
    Date: Fri, 08 Oct 2021 18:42:24 GMT
    Content-Type: text/html;charset=utf-8
    Transfer-Encoding: chunked
    Server: Jetty(9.4.31.v20200723)


    <h1>¡Hola Mundo!</h1>
    <p>Esta es nuestra primera respuesta HTML</p>
```  

* Bienvenida (observar el uso de %20 para representar el espacio, empleando URL encoding):

```
    $ curl http://localhost:9000/bienvenida?nombre=Todo%20el%20mundo -iL
    HTTP/1.1 200 OK
    Date: Fri, 08 Oct 2021 18:47:40 GMT
    Content-Type: text/html;charset=utf-8
    Transfer-Encoding: chunked
    Server: Jetty(9.4.31.v20200723)


    <h1>¡Hola Todo el mundo!</h1>
    <p>Esta es nuestra primera respuesta HTML</p>
```

* Despedida:

```
    $ curl http://localhost:9000/despedida?nombre=lu -iL
    HTTP/1.1 200 OK
    Date: Fri, 08 Oct 2021 18:52:06 GMT
    Content-Type: text/html;charset=utf-8
    Transfer-Encoding: chunked
    Server: Jetty(9.4.31.v20200723)


    <h1>¡Adios lu, que la fuerza te acompañe!</h1>
    <p>Nos veremos de nuevo en otro apunte</p>
``` 

¡Todo funciona! ¿Pero qué problema trae esta estrategia de solución? ¿Qué pasará cuando haya más y más rutas?
¡Nuestro Main crecerá sin control!

 Por ello, al igual que como hicimos con las vistas, a las que llevamos a archivos aparte gracias a las planillas,
acá vamos a nuevamente separar responsabilidades y crear archivos independientes que agrupen rutas en controladores.


###--- CONTROLADORES (LA C DE MVC) ---

Empecemos por crear un nuevo paquete controllers, que contendrá nuestros controladores. Allí incorporaremos dos clases:
        * HomeController: resolverá la ruta raíz.
        * SaludoController: resolverá las rutas de bienvenida y despedida.

Ninguna de estas clases deberá, en principio, heredar clases ni implementar interfaces específicas;
serán simplemente clases planas, comunes y corrientes, a las que moveremos el código de manejo de las rutas existentes.

HomeController:

```java
package controllers;


import spark.Request;
import spark.Response;


public class HomeController {
 public Void index(Request request, Response response) {
   response.redirect("/bienvenida");
   return null;
 }
}
```
    

SaludoController:

    
```java
package controllers;


import spark.ModelAndView;
import spark.Request;
import spark.Response;


public class SaludoController {
 public ModelAndView bienvenida(Request request, Response response) {
   return new ModelAndView(
           request.queryParamOrDefault("nombre", "Mundo"),
           "bienvenida.html.hbs");
 }


 public ModelAndView despedida(Request request, Response response) {
   return new ModelAndView(
           request.queryParamOrDefault("nombre", "Mundo"),
           "despedida.html.hbs");
 }
}
```
    

Como podemos apreciar, hemos convertido en métodos de instancia al código que anteriormente
estaba en bloques de código. Y en el proceso, hemos tenido que explicitar los tipos de datos:
de sus parámetros: spark.Request y spark.Response;
de su retorno: Void en los casos que la respuesta no tiene cuerpo (body), y
spark.ModelAndView cuando sí lo hay.

Luego, en Main instanciaremos los controladores y reemplazaremos las lambdas por referencias
a los métodos correspondientes:

    
```java
import controllers.HomeController;
import controllers.SaludoController;


import spark.ModelAndView;
import spark.Request;
import spark.Response;
import spark.Spark;
import spark.template.handlebars.HandlebarsTemplateEngine;


public class Main {
 public static void main(String[] args) {
   Spark.port(9000);
   HandlebarsTemplateEngine engine = new HandlebarsTemplateEngine();


   HomeController home = new HomeController();
   SaludoController saludo = new SaludoController();


   Spark.get("/", home::index);
   Spark.get("/bienvenida", saludo::bienvenida, engine);
   Spark.get("/despedida", saludo::despedida, engine);
 }
}

    

De esta forma, cada una de estas clases serán mucho más cohesivas y fáciles de mantener y probar.
Es más: ahora tendremos más herramientas para evitar la repetición de lógica!.
