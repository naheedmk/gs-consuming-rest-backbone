This guide walks you through writing a simple Backbone client that consumes a Spring MVC-based [RESTful web service][u-rest].


What you will build
-------------------

You will build a Backbone client that consumes a Spring-based RESTful web service.
Specifically, the client will consume the service created in [Building a RESTful Web Servce][gs-rest-service].

The Backbone client will be accessed by opening the `index.html` file in your browser, and will consume the service accepting requests at:

    http://rest-service.guides.spring.io/greeting

The service will respond with a [JSON][u-json] representation of a greeting:

```json
{"id":1,"content":"Hello, World!"}
```

The client will render the ID and content into the DOM.

You can customize the greeting with an optional *query string* in the url:

    http://localhost:8080/?User

The code will send a parameter to the REST endpoint and
render a custom greeting into the DOM.


What you'll need
----------------

 - About 15 minutes
 - A favorite text editor
 - A modern web browser
 - An internet connection
 - Node.js and git pre-installed
 - Bower installed as a global node.js package

<a name="scratch"></a>
<a name="initial"></a>
Create bower configuration files
---

First, create a bower control file, `.bowerrc`.  This file tells bower
where to put the JavaScript dependencies.  The `.bowerrc` file should
be located at the root of the project (`gs-consuming-rest-backbone/initial`) and
formatted as JSON:

`.bowerrc`
```bowerrc
{
    "directory": "public/lib"
}
```

From a command prompt at the root of the project, run `bower init`.
This will create a `bower.json` file that describes the
JavaScript packages required by the project.  Bower will ask for several
bits of information such as a project name, license, etc.  If in doubt,
just press `Enter` to accept the defaults.

Next, use bower to install Backbone (since we're using JavaScript modules, we'll use the AMD version of Backbone), jQuery and Lodash (an alternative to Underscore), and an AMD module loader such as curl.js.  From the command prompt, type:

```
bower install --save backbone-amd#~1
```

```
bower install --save jquery#~2
```

```
bower install --save lodash#~1
```

```
bower install --save curl#~0.8
```

Bower will install these packages into the directory we listed
in `.bowerrc`.  Since we specified the `--save` option, bower will
store the package information in the `bower.json` file.

When done, the `bower.json` file should have a "dependencies"
object property that lists "backbone-amd", "jquery", "lodash", and "curl" as property names and their semver information as values:

`bower.json`
```json
{
	"name": "draft-consuming-rest-backbone.js",
	"version": "0.0.1",
	"authors": [
		"Brian Cavalier <bcavalier@gopivotal.com>"
	],
	"license": "http://www.apache.org/licenses/LICENSE-2.0",
	"ignore": [
		"**/.*",
		"node_modules",
		"bower_components",
		"lib",
		"test",
		"tests"
	],
	"dependencies": {
		"curl": "~0.8",
		"backbone-amd": "~1",
		"lodash": "~1",
		"jquery": "~2"
	}
}
```


<a name="scratch"></a>
Create a Backbone Model
-----------------------

Backbone consumes data from a RESTful web services via models and collections.  First, you'll create a Backbone model that represents the data you want to consume from the REST service.

`public/hello/HelloModel.js`
```js
define(function(require) {
	var Backbone = require('Backbone');

	return Backbone.Model.extend({
		urlRoot: 'http://rest-service.guides.spring.io/greeting',
		url: function() {
			return this.urlRoot + '?name=' + this.id;
		}
	});
});
```

The model extends Backbone's base Model, and sets the model's `urlRoot` to the REST service at http://rest-service.guides.spring.io/greeting.  Note that it also passes its `id` field as a url query parameter.  This is used to customize the greeting using the query string, as mentioned above (e.g. `http://localhost:8080/?User`)

Create a Backbone View
----------------------

Next, you'll create a Backbone view to render the data in your `HelloModel`.

`public/hello/HelloView.js`
```js
define(function(require) {
	var Backbone = require('Backbone');
	var $ = require('jquery');
	var _ = require('underscore');

	return Backbone.View.extend({
		initialize: function() {
			this.template = _.template($('#hello-template').html());
			this.listenTo(this.model, 'change', this.render);
		},

		render: function(){
			this.$el.html(this.template(this.model.attributes));
		}
	});
})
```

The view extends Backbone's base View.  The `initialize` method will be called when the view is instantiated.  It uses Underscore to compile a template that will be used to render the model data, saving the compiled template in `this.template`.

Backbone automatically wraps the view's root DOM Node (which will be provided when instantiating the view) in jQuery and makes it available as `this.$el`.  The `render` method renders the compiled template, passing the model data, and then uses jQuery's `html()` method to insert the rendered output into the DOM.

Create a Controller
-------------------

`public/hello/main.js`
```js
define(function(require) {
	var HelloModel = require('./HelloModel');
	var HelloView = require('./HelloView');
	var $ = require('jquery');

	var model = new HelloModel({ id: document.location.search.slice(1) });
	model.fetch();

	$(document).ready(function() {
		var hello = new HelloView({
			el: $('.hello').first(),
			model: model
		});
	});

});
```

This controller instantiates a `HelloModel`, and then invokes its `fetch` method to fetch data from the REST service and populate the model's data fields.  Then it instantiates a `HelloView`, passing the DOM Node where it should render, and the model.  The view will automatically render the model using its compiled template.

Create the Application Page
---------------------------

Now that you have a model, view, and controller, you'll create the HTML page that will load the client into the user's web browser:

`public/index.html`
```html
<!doctype html>
<html>
	<head>
		<title>Hello Backbone</title>
		<script data-curl-run="run.js" src="lib/curl/src/curl.js"></script>
		<script type="text/html" id="hello-template">
			<p>The ID is <%= id %></p>
			<p>The content is <%= content %></p>
		</script>
	</head>
	<body>
		<div class="hello">
		</div>
	</body>
</html>
```

The script element will load curl.js and then load an application boot script named "run.js". The boot script will initialize and configure an AMD module environment and then start the client-side application code.

```html
<script data-curl-run="run.js" src="lib/curl/src/curl.js"></script>
```

Next is the HTML template that your view uses to render the model data.  Note that we use a script tag, with the type `text/html`.  This tells the browser not to try to execute the script tag as JavaScript.  It has an `id` so that it can be easily referenced from the view and compiled.

```html
<script type="text/html" id="hello-template">
    <p>The ID is <%= id %></p>
    <p>The content is <%= content %></p>
</script>
```

Finally, there is the root DOM Node of the view.  The view will render the model data, using the template, into this node:

```html
<div class="hello">
</div>
```

<a name="run"></a>
Run the client
--------------

You can now run the app using the Spring Boot CLI (Command Line Interface). Spring Boot includes an embedded Tomcat server, which offers a simple approach to serving web content. See [Building an Application with Spring Boot][gs-spring-boot] for more information about installing and using the CLI.

```sh
$ spring run app.groovy
```

Once the app starts, open http://localhost:8080 in your browser, where you see:

![Model data retrieved from the REST service is rendered into the DOM.](images/hello.png)

The ID value will increment each time you refresh the page.


Summary
-------

Congratulations! You've just developed a Backbone client that consumes a Spring-based RESTful web service.

[gs-rest-service]: /guides/gs/rest-service/
[gs-spring-boot]: /guides/gs/spring-boot/
[zip]: https://github.com/spring-guides/gs-consuming-rest-backbone/archive/master.zip
[u-rest]: /understanding/REST
[u-json]: /understanding/JSON
[u-git]: /understanding/Git
