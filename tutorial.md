# ¿Qué es una API?
*Those already familiar with APIs may want to skip straight to the [BBC API introduction](https://github.com/basilesimon/using-an-api-tutorial/blob/master/tutorial.md#what-you-will-find-in-the-bbc-api) and [further reading](https://github.com/basilesimon/using-an-api-tutorial/blob/master/tutorial.md#beyond-this-tutorial).*

Es posible que hayas utilizado muchas APIs sin saber incluso que lo hacías:

* ¿has pinchado alguna vez en Twitter en la opción de "Compartir"?
* ¿has abierto Facebook o Twitter?

Entonces has utilizado una API.

Una APIs es una herramienta para interactuar con proveedores de Internet. Se pueden utilizar para: Se usan para:

* **recopilar datos** de un proveedor (esto es lo que ocurre cuando lees tuits en tu timeline),
* o **enviar datos** a un servicio (cuando envías un tuit desde una aplicación externa a twitter)

# Seguridad
A menudo los proveedores de contenido ponen restricciones al uso de API por cuestiones de seguridad. Para ello, solicitan una clave para poder acceder al servicio de la API. También suelen existir restricciones para el número de veces que puedes utilizar una API por hora.

# ¿Por qué la API de la BBC?
Para este tutorial vamos a utulizar *BBC linked data API*. Linked data hace referencia a las relaciones entre concreptos presentes en el contenido. Por ejemplo, un artículo que mencione a Barack Obama será considerao por el servicio como un artículo sobre Obama. 
Esta API se llama "the Juicer" y es una de las APIs más sencillas de utilizar. Además tiene una aplicación periodística y puede ser utilizada para investigación. Ha sido desarrollado por BBC News Labs:

![juicer](juicer.png)

* The Juicer scrapes noticias de la BBC y de más de 150 medios y los indexa en una base de datos guardando titular, autor, fuente, imágenes...
* It then performs entity extraction: in the articles' bodies, it will recognises some concepts ("David Cameron", "smoking", "Leicester", "Apple Inc"...), which can be people, places, organisations, or intangibles.
* It will match these concepts with a unified concepts base containing information about these concepts, thus providing some consistency.
* Finally, it will draw relationships between the concepts: how often they co-occur together, with which other concepts they appear, their frequency over time...

Every step of this process can be interrogated by talking to the API directly. You might only be interested by reading articles about the geographic zone 15 miles around your current GPS position, for example. Or you might just want to know what are the other politicians co-occurring with Nigel Farage in the last two months, and in which proportions. Or, you might be curious to know who is mentioned the most alongside Adolf Hitler in news articles.

Now, it might seem all a bit silly and broad, but you can use these APIs for quite nice purposes. If you have downloaded the new BBC News smartphone app (available on Android and iOS), you probably noticed that you can *follow* topics, such as your constituency or geographical area, major news developments ( *Islamic State crisis*, *counter-terrorism*, *Jeremy Clarkson*, *Cymru*, *Election 2015*...). These pages are aggregated automatically by the APIs (and probably involve a certain amount of human curation, but I must admit I don't know a whole lot about that).

# Hacer peticiones
Para obtener los datos de los que hablamos, tenemos que realizar una petición. Ésta tiene la forma de una url a la que incluimos las variables por las que queremos filtrar y una clave de acceso para conectarnos a la API.

**La clave para el evento es `YB0MY3VMHyllzPqEf5alVj5bUvGpvDVi`.**

### Example queries
Try the following queries by opening the URLs in your browser (you'll need to paste in the API key at the very end):
* All sources being juiced - http://juicer.api.bbci.co.uk/sources?apikey=YB0MY3VMHyllzPqEf5alVj5bUvGpvDVi
* Scope to only BBC content - http://juicer.api.bbci.co.uk/articles?sources[]=1&apikey=YB0MY3VMHyllzPqEf5alVj5bUvGpvDVi
* Search for the word "London" in all BBC sources - http://juicer.api.bbci.co.uk/articles?q=London&sources[]=1&apikey=YB0MY3VMHyllzPqEf5alVj5bUvGpvDVi
* Search for the word "London" in all BBC sources, faceted (filtered) by the concept "David Cameron" - http://juicer.api.bbci.co.uk/articles?q=London&sources[]=1&facets[]=http://dbpedia.org/resource/David_Cameron&apikey=YB0MY3VMHyllzPqEf5alVj5bUvGpvDVi
* Search for the word "London" in all BBC sources, faceted (filtered) by the concept "David Cameron", and show results in reverse chronological order - http://juicer.api.bbci.co.uk/articles?q=London&sources[]=1&facets[]=http://dbpedia.org/resource/David_Cameron&recent_first=true&apikey=YB0MY3VMHyllzPqEf5alVj5bUvGpvDVi
* An article from the Juicer - http://juicer.api.bbci.co.uk/articles/4c3cfc0b24ba06bd204c6a24bd14e43bb006b0ea?apikey=YB0MY3VMHyllzPqEf5alVj5bUvGpvDVi

 
# Getting a response
Una vez hayas realizado tu petición, el servidor responderá a ell.a 
After you've sent your query, the server will reply in a certain format. One of the most common is JSON (which stands for *JavaScript Obect Notation*). This format is very standard and easy to parse in any language, and though it can be a bit bigger than flat CSV datasets, it is quite good at expressing relationships and hierarchy. JSON is also the format returned by the Juicer, and thus the one that we will use for this tutorial.

# Exploiting the received data
Okay, so now that we're comfortable with the Juicer API, it's time to actually put the data you've queried to good use. One of the most simple and frequent thinkg you'll do with data obtained is to throw some bits of it into a web page. 

This is surprisingly easy with some Javascript (actually, *jQuery*, but sssh, don't wake up the trolls).

En este taller cubriremos los siguientes ejercicios:
* Realizar una petición sobre los artículos más recientes que versen sobre *Madrid*.
* Mostrar los titulares y descripciones de esos titulares en una página web muy básica.

#### Esqueleto HTML5
Abre un editor de textos ( [Sublime Text](https://www.sublimetext.com),[Atom](https://atom.io/) ) y crea un archivo. Llámalo `index.html` y pega el código siguiente en él.

En este esqueleto incluimos las etiquetas base de HTML y enlazamos a jQuery, una librería de Javascript.

```html
    <!DOCTYPE HTML>
    <html>
    <head>
      <meta charset="utf-8" />
      <title>API with jQuery</title>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
    </head>

    <body>
      <h1>Hello!</h1>
      <button>Get JSON data</button>

      <script>
        // aquí incluiremos nuestro código de Javascript
      </script>

    </body>
    </html>
```

#### Making a request
To cut to the chase: jQuery es la librería que utilizaremos para obtener los datos estructurados en formato JSON de la API sin tener que parsearlos o diseccionarlos: `$.getJSON()`. El código que tenemos que escribir es el siguiente:

```javascript
    // Introduciremos la clave API
    var apikey = ;

    // La query o consulta que quieres realizar
    var query = "http://juicer.api.bbci.co.uk/articles?q=London&apikey=" + apikey;

    $("button").click(function(){
      $.getJSON(query, function(data){
        console.log(data);
      })
    });
```

Abre `index.html` en tu navegador y abre las *Herramientas de desarrollo* o *Developer Tools* (`Shift + Ctrl + K` en Firefox, `Shift + Ctrl + J` en Chrome). Una vez hecho esto, pincha en el botón "Get Json Data".

¡Ya puedes ver los datos!

> **Note:** If you don't have an API key for the BBC API, use [this cached version](https://rawgit.com/basilesimon/using-an-api-tutorial/master/cacheJSON.json) instead.
>
> So in the above example, you should use this line:
> 
> `var query = "https://rawgit.com/basilesimon/using-an-api-tutorial/master/cacheJSON.json";`

#### Using the request to build your web page
Good, good. Now we've got some JSON back directly in the browser. What do you say we take parts of this JSON and injects it directly into our web page to create some content, eh?

If you paid attention earlier, you know that we're querying to the Juicer a list of articles mentioning *"London"*. This JSON looks like this: 

![json response](responseJSON.png)

Now, some technicalities:

* A JSON is made to be navigated in, hierarchically. In that case, we're going to want to target the `hits` group, and leave `aggregations` and `timeseries` alone. When we're into `hits`, we have a list (numbered from 0, as it is the norm in most programming languages). Into each element of this list, there's the information we will want to display: a *title*, a *url*, a *description*...
* To access properties in the JSON, consider that the dot (`.`) expresses levels of hierarchy. Thus, considering that `data` is the variable representing your JSON, `data.subdata` is a sub-element of the JSON object. And `data.subdata.article` is a sub-element of `subdata`, which is itself a sub-element of the JSON object. Go deeper with dots.
* In this particular case, if I want to access the `title` element, you might expect that we need to read (assuming that `data` represents the JSON object we queried) `data.hits.title`. *However, that won't work.*
* This is because when you have a list, you need to say which element you want to read in the list (numbered from 0, remember). Thus, to read the first element (trust me on this one), you will want to read `data.hits[0].title`. Try to `console.log` this after your `getJSON`.
* When you have several elements in a list, you can easily write loops to perform a single operation on each one, without having to write the instruction *x* times. That's one of the things that makes computers so good at repetitive tasks.

Anyway, to the code:

```javascript
    var apikey = ;
    var query = "http://juicer.api.bbci.co.uk/articles?q=London&apikey=" + apikey;

    $("button").click(function(){    
        $.getJSON( query, function( data ) {
          var items = [];       // Tenemos que crear los `items`, which is an empty list. Rellenaremos este hueco con el contenido que queremos obtener de la página web.

          // Then, with `$.each()`, we're writing this loop to peform an operation on each element of `data.hits`.
          $.each( data.hits, function( key, val ) {     

            // We are then *pushing* a piece of HTML to `items`, that we created earlier.
            // This piece of HTML contains a `<li>` element (a list item) which, look at this, has `val.title` for value.
            // `val.title`, I forgot to explain, represents `data.hits.title`, that's just a shortcut we declared.
            items.push( "<li>" + val.title + "</li>" );
          });

          $( "<ul/>", {                 // Then, after the white line, we're grabbing the `<ul>` element in our HTML,
            html: items.join( "" )      // And saying that its HTML should be what's contained in `items`.
          }).appendTo( "body" );        // Finally, we *append* these list elements to our HTML *body*.
        });
    });
```

Are you still here? Good. Refresh the page and click the button. Magic happened.

Nothing changed from before at the top: we give it an API key, a query to perform with this API key, and we're using `getJSON()` to make the query.

> **Note:** Reminder that if you don't have an API key for the BBC API, use [this cached version](https://rawgit.com/basilesimon/using-an-api-tutorial/master/cacheJSON.json) instead. Like so:
>
> `var query = "https://rawgit.com/basilesimon/using-an-api-tutorial/master/cacheJSON.json";`

# Challenge round!

Here's three tasks to take this web page a step further. Answers are at the bottom of this page, but I'll give you a big hint: each of these requires no more than a single line of code to be changed.

## Crea un titular enlazado a una url

Wouldn't it be useful to be able to click through to each article? The URLs are returned in the API response - can you get those into the HTML?

## Cambia el término de búsqueda
London is boring. So is the prime minister, but at least his name poses a challenge: that space in the URL will need encoding.

## Busca artículos publicados antes de 2014
The API request URL contains a query (`q=London`) and an API key (`apikey={apikey}`). But it can contain other things, too. Check [the API documentation](http://docs.bbcnewslabs.co.uk/Juicer-2.html) for additional parameters, including one that can limit the time period of returned articles.

# Beyond this tutorial
You pretty much know the basics right now, so here are a couple of things you could do to learn a bit more.

#### Getting to know Juicer better with Postman
You don't need much to talk to an API. Although some tools are nicer than the others.
Have a look at [Google Chrome extension Postman](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm?hl=en), which is a pretty tool to make requests clear to humans. Go ahead and install it!

You can them import the Juicer API using [this URL](https://www.getpostman.com/collections/cebe998a209d9862135b). This allows you to easily try out the API and explore how it works without writing any code.

You will need to configure an 'Environment' within Postman to be able to make cals:

![environment](http://docs.bbcnewslabs.co.uk/img/juicer-postman.png)

And you best ally is, as always, [the documentation](http://docs.bbcnewslabs.co.uk/Juicer-2.html)!

### Want to code a bit more?
Why not writing a [Google News](https://news.google.com/)-y kind of thing? If you've followed the tutorial, here's what you should do:

* List the titles (as we did before)
* Transform the list elements into links (`<a>` elements)
* Change the source of the links to the corresponding URLs (tip: change `href`)

# The answers

## Make each headline hyperlinked

Each item in the API response contains not only a `title`, but also a `url`. We can use it in a hyperlink like so:

```js
items.push( "<li><a href='" + val.url + "'>" + val.title + "</a></li>" );
```

## Fetch articles on David Cameron instead of London
We can switch out "London" for "David Cameron" easily enough, but that space will need encoding as `%20`.

```js
var query = "http://juicer.api.bbci.co.uk/articles?q=David%20Cameron&apikey=" + apikey;
```

## Fetch articles published before 2010

We'll need to make use of the `published_before` URL parameter, as noted in the [API documentation](http://docs.bbcnewslabs.co.uk/Juicer-2.html), to achieve this:

```js
var query = "http://juicer.api.bbci.co.uk/articles?q=London&published_before=2010-01-01T00:00:00.000Z&apikey=" + apikey;
```