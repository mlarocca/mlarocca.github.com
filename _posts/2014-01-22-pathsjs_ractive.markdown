---
layout: post
title: Reactive SVG charts with Ractive.js and Paths.js
---

##Ractive


__Ractive.js__ is a nice, lightweight library that allows you to easily add reactive behaviour to your web page.

It's not the first lib that allows you to use reactive programming on html markups: during the last year several of them have been released, from [__React__](http://facebook.github.io/react/) to the amazing [__AngularJS__](http://angularjs.org) from Google.
Arguably, __Angular__ is actually the most complete, and best maintained (no surprise, since it is a Google project!). But its many features makes it a bit heavy, and the learning curve steep.
Don't get me wrong, I used __Angular__ in a few projects (for instance, [Tag Cloud](http://mlarocca.github.io/01-14-2013/tagcloud.html)) and it is awesome - especially because it allows you (or forces you) to adhere to the [MVC pattern](http://en.wikipedia.org/wiki/Model–view–controller). But sometimes it is far more stuff than you'd need.

The Difference between __Angular__ and __Ractive__ are thoroughly addressed on [__Ractive__'s blog](http://blog.ractivejs.org/posts/whats-the-difference-between-angular-and-ractive): mainly their scope is different, since Angular addresses also routing, validation, server communication, testing etc..., while __Ractive__ leaves all that stuff to each developer, allowing you to choose whatever solution you like for them - if you do need them in the first place.

If you are new to __Ractive__, there is nothing to worry about: it is super-easy, super-useful, and there is plenty of documentation available. You might want to start here: [60 second setup](https://github.com/RactiveJS/Ractive/wiki/60-second-setup)

Here it is how the example in the 60 second setup would look like, assuming you downloaded the script for __Ractive__ and saved it into the js subfolder inside your project.

{% highlight html %}
<!doctype html>
<html>
  <head>
  </head>
  <body>
    <div id='container'></div>

    <script src='js/Ractive.js'></script>

    <script id='myTemplate' type='text/ractive'>
        <p>{% raw  %}{{greeting}}, {{recipient}}{% endraw  %}!</p>
    </script>

    <script>
        var ractive = new Ractive({
            el: 'container',
            template: '#myTemplate',
            data: { greeting: 'Hello', recipient: 'world' }
        });
    </script>

  </body>
</html>
{% endhighlight %}

Want to see some action? Try to run

{% highlight javascript %}
ractive.set("recipient", "everybody");
{% endhighlight %}

in your browser's console! 

Now compare this to how you could reach the same goal without __Ractive__.

{% highlight html %}
<!doctype html>
<html>
  <head>
  </head>
  <body>
    <div id='container'></div>

    <p><span id='greeting'>Hello</span>, <span id='recipient'>world</span>!</p>

  </body>
</html>
{% endhighlight %}

{% highlight javascript %}
var recipient = document.getElementById('recipient');
if (recipient) {
  recipient.innerText = "world";
}
{% endhighlight %}

And this is just a fairly simple example!
Besides the complexity and the nuances, the main difference is that using __Ractive__ you bind a value to a variable, that you can use in more than one place inside your page:
{% highlight html %}
    <script id='myTemplate' type='text/ractive'>
        <p>{% raw  %}{{greeting}}, {{recipient}}{% endraw  %}!</p>
        <input type='text' value='{% raw  %}{{recipient}}{% endraw  %}'>        
    </script>
{% endhighlight %}
Even using [__jQuery__](http://jquery.com) or another equally powerful lib, every time you update that variable, __Ractive__ will redraw every node in your _HTML_ page that depends on it, and it will do in the most efficient possible way (or so).

Without __Ractive__, you have no such binding, you'll have to remember every place in your page that needs to be updated (and with non-trivial examples that can get really messy, believe me!), and every time you have a change in the value, you have to manually update all of them - or at least as many of them as you remember. 

To learn more about __Ractive__'s features, please take the [Interactive tutorial](http://learn.ractivejs.org/).
Once done with that, you can probably have a better grasp at [this](http://learn.ractivejs.org/svg/2/) complex example using SVG

The crucial part we are interested in, is the template:

{% highlight html %}
  <svg id='svg'>

    <!-- gradient - higher temperatures are redder, lower temperatures are bluer -->
    <defs>
      <linearGradient id='gradient' x2='0' y2='100%' gradientUnits='userSpaceOnUse'>
        <stop offset='0%' stop-color='rgb(255,0,0)' />
        <stop offset='100%' stop-color='rgb(0,0,255)' />
      </linearGradient>
    </defs>

    <!-- horizontal line representing freezing -->
    <line class='freezing' x1='0' y1='{% raw  %}{{ yScale(0) }}{% endraw  %}' x2='{{width}}' y2='{% raw  %}{{ yScale(0) }}{% endraw  %}'/>
    {% raw  %}{{#selectedCity}}{% endraw  %}
      
      <!-- the band -->
      <polygon fill='url(#gradient)' stroke='url(#gradient)' class='temperature-band' points='{% raw  %}{{ getBand(months,xScale,yScale) }}{% endraw  %}'/>

      {% raw  %}{{#months:i}}{% endraw  %}
        <!-- point markers for average highs -->
        <g class='marker' transform='{% raw  %}translate({{ xScale(i+0.5) }},{{ yScale(high) }}{% endraw  %})'>
          <circle r='2'/>
          <text y='-10'>{% raw  %}{{ format(high,degreeType) }}{% endraw  %}</text>
        </g>

        <!-- point markers for average lows -->
        <g class='marker' transform='{% raw  %}translate({{ xScale(i+0.5) }},{{ yScale(low) }}{% endraw  %})'>
          <circle r='2'/>
          <text y='15'>{% raw  %}{{ format(low,degreeType) }}{% endraw  %}</text>
        </g>
      {% raw  %}{{/months}}{% endraw  %}
    {% raw  %}{{/selectedCity}}{% endraw  %}

  </svg>
{% endhighlight %}

If you know D3, Raphael, or other similar SVG libraries, this might look a bit odd, since you are probably used to create SVG nodes from JavaScript. Here, instead, thanks to reactive programming, we take a different direction, creating the chart inside the html file; this enables us to better separate data from the structure of the charts, in the same way discussed above for _HTML_.

##Paths.js

Paths.js is another minimal library by [Andrea Ferretti](https://github.com/andreaferretti) that is, explicitly and by design, oriented to support reactive programming by generating SVG paths that can be used with template engines. It looks like a perfect match for __Ractive__, and it actually is: take a look at this [demo](http://andreaferretti.github.io/paths-js-demo/) - source available on [GitHub](https://github.com/andreaferretti/paths-js-demo).
The source is in CoffeeScript, so - if you feel more like a JavaScript guy - take a look at it's JS counterpart!


Html file with templates:
{% highlight html %}
<!doctype html>
<html>
  <head>
    <!--meta charset='utf-8'-->
    <title>Pathjs</title>  </head>
    <link rel="stylesheet" type="text/css" href="test.css">
  <body>
    <!--script type='text/javascript' src='./js/require.js'></script-->
    <!--script src='http://code.jquery.com/jquery-2.0.3.min.js'></script-->
    <div id='pie'></div>
    
    <script id='myTemplate' type='text/ractive'>
      <div class="panel panel-default">
        <div class="panel-heading">
          <h2 class="panel-title">Pie Chart</h2>
        </div>
        
        <select value='{% raw  %}{{selectedDataset}}{% endraw  %}' on-change='loadData'>
          {% raw  %}{{#datasets}}{% endraw  %}
            <option value='{% raw  %}{{.["filename"]}}{% endraw  %}'>{% raw  %}{{.["label"]}}{% endraw  %}</option>
          {% raw  %}{{/datasets}}{% endraw  %}
        </select>

        <div class="panel-body">
          <p class="alert alert-info">Here is a pie chart example. Sectors are clickable.</p>

          <svg width=375 height=400>
            <g transform="translate(200, 200)">
              {% raw  %}{{# Pie({center: center, r: r, R: R, data: countries, accessor: accessor, colors: colors}) }}{% endraw  %}
                {% raw  %}{{# curves:num }}{% endraw  %}
                  <g transform="{% raw  %}translate({{ move(sector.centroid, expanded[num]) }}{% endraw  %})">
                    <linearGradient id = "{% raw  %}grad-{{ num }}{% endraw  %}">
                      <stop stop-color = "{% raw  %}{{ color_string(color) }}{% endraw  %}" offset = "0%"/>
                      <stop stop-color = "{% raw  %}{{ lighten(color) }}{% endraw  %}" offset = "100%"/>
                    </linearGradient>
                    <path on-click="expand" d="{% raw  %}{{ sector.path.print() }}{% endraw  %}" fill="{% raw  %}url(#grad-{{ num }}){% endraw  %}" />
                    <text text-anchor="middle" transform="{% raw  %}translate({{ point(sector.centroid) }}){% endraw  %}">{% raw  %}{{ item.name }}{% endraw  %}</text>
                  </g>
                {% raw  %}{{/ curves }}{% endraw  %}
              {% raw  %}{{/ end of pie}}{% endraw  %}
            </g>
          </svg>

          {% raw  %}{{# countries: num }}{% endraw  %}
            {% raw  %}{{# expanded[num] == 1}}{% endraw  %}
              <div class="country-info">
                <h4>{% raw  %}{{ name }}{% endraw  %}</h4>
                <p>Population: <span class="label label-info">{% raw  %}{{ population }}{% endraw  %}</span></p>
              </div>
            {% raw  %}{{/ end if }}{% endraw  %}
          {% raw  %}{{/ countries }}{% endraw  %}
        </div>
      </div>
    </script>    
    <script data-main="js/test" src="js/require.js"></script>

  </body>
</html>
{% endhighlight %}

test.js file (the relevant part):

{% highlight javascript %}
  function loadCountries(dataset) {
      $.ajax({url: dataset, 
              headers: {'Content-Type': 'application/json'}, 
              processData: false})
          .done(function (data) {
            if (typeof data === 'String') {
              data = JSON.parse(data);
            }
            ractive.animate({
              "countries": data,
              'expanded': util.initArray(0, data.length)
            });
          })
          .fail(function (err) { console.log("ERROR LOADING JSON", err);});
  }

  var palette = Colors.mix({r: 130, g: 140, b: 210}, {r: 180, g: 205, b: 150});

  var ractive = new Ractive({
        el: 'pie',
        template: '#myTemplate',
        data: {
            Pie: Pie,
            center: [0, 0],
            r: 60,
            R: 140,
            countries: [],
            expanded: [],
            datasets: [{label: "Mixed", filename: "json/countries.json"}, 
                       {label: "Europe", filename: "json/europe.json"},
                       {label: "Asia", filename: "json/asia.json"}],
            accessor: function (x) {
                        return x.population;
                      },
            colors: util.palette_to_function(palette),
            move: function (point, expanded) {
              var factor = expanded || 0;

              return (factor * point[0] / 3) + "," + (factor * point[1] / 3);
            },
            point: function (point) {
              return point[0] + "," + point[1];
            },
            lighten: function (color) {
              return Colors.string(Colors.lighten(color));
            },
            color_string: Colors.string
        }
      });

      ractive.on({expand:   function (event) {
                              var index = event.index.num,
                                  target = util.initArray(0, ractive.get('expanded').length);
                              target[index] = 1;
                              ractive.animate('expanded', target, {easing: 'easeOut'});
                            },
                  loadData: function (event) {
                              var options = event.node.options;
                              loadCountries(options[options.selectedIndex].value);
                            }
                });
      loadCountries(ractive.get('datasets')[0].filename);
{% endhighlight %}

The rest of the code (a couple of utility js modules, the css file, and the jsons in the example) can be found here [on github](https://github.com/mlarocca/PathsJS-demo).

You can import __Paths.js__ as a [standalone script](https://github.com/andreaferretti/paths-js#standalone-script), which will add the _paths_ object to the global scope. However, my advice is to use __RequireJS__ and import just the elements you need: it will be lighter on your page, and __RequireJS__ will take care of all the dependencies and the loading for you; moreover, the final result is cleaner and less prone to tampering and interferences, since no global object is exposed.

You can take a look at the [working demo](http://mlarocca.github.io/PathsJS-demo/demo.html)

As you can see, I took a step further with respect to the __Paths.js__ demo, allowing the chart to be dynamically populated, and animating the transitions. 
The effect, in my opinion, is stunning, especially considering that it took me like 5 minutes to add this new features to the example.
Of course, my contribution was really minimal: it's just that using both libraries together makes it extremely simple to manipulate SVG elements, and animations are a benefit you get basically for free.
Notice, for example, how you declare the chart, and then iterate on its building blocks:

{% highlight html %}
  {% raw  %}{{# Pie({center: center, r: r, R: R, data: countries, accessor: accessor, colors: colors}) }}{% endraw  %}
    {% raw  %}{{# curves:num }}{% endraw  %}
      <g transform="{% raw  %}translate({{ move(sector.centroid, expanded[num]) }}){% endraw  %}">
        <linearGradient id="{% raw  %}grad-{{ num }}{% endraw  %}">
          <stop stop-color="{% raw  %}{{ color_string(color) }}{% endraw  %}" offset="0%"/>
          <stop stop-color="{% raw  %}{{ lighten(color) }}{% endraw  %}" offset="100%"/>
        </linearGradient>
        <path on-click="expand" d="{% raw  %}{{ sector.path.print() }}{% endraw  %}" fill="{% raw  %}url(#grad-{{ num }}){% endraw  %}" />
        <text text-anchor="middle" transform="{% raw  %}translate({{ point(sector.centroid) }}){% endraw  %}">{% raw  %}{{ item.name }}{% endraw  %}</text>
      </g>
    {% raw  %}{{/ curves }}{% endraw  %}
  {% raw  %}{{/ end of pie}}{% endraw  %}

{% endhighlight %}

##Key points

1. __Ractive__ allows you to iterate over objects returned by function calls. It lists the dependencies for the expression, so that every time one of the objects it depends on is changed, the nodes affected by this change (and only those nodes) are redrawn.

2. __Paths.js__ _Pie_ method returns an object with an array, curves, listing the shapes to draw. This allows you to separate the data from their visualization, and make the latter more explicit, improving manutenability and easiness of access, simply by moving it to a template rather than keeping it hidden in the belly of a JS library.

3. [Context](http://learn.ractivejs.org/nested-properties/2/). Each section tag from Ractive provides context for nested properties. Even for anonymous objects (like the one returned from the call to Pie): curves is actually Pie(...).curves, and sector in the next line is Pie(...).curves.sector

4. Attributes. They are automatically updated too.

5. The number of elements doesn't matter, nor does their order: __Ractive__ takes care of every thing. It even animates elements creation/disposal (see transitions on dataset change).

6. Animations: it's unlikely that they will slow down your page (as it usually happens), since __Ractive__ handles them using a single animation loop, see [here](http://learn.ractivejs.org/animation/1/).

7. Event handling: for each proxy event only a single handler is added; they are handled in a very efficient way, namely even more efficient than delegation, and as elements are added and removed, their handlers are automatically been taken care of. Overall, that also helps performance. A lot. (You can read more [here](http://learn.ractivejs.org/event-proxies/5/))

##Conclusions

I hope this example raised your interest, or at least your curiosity, about this two great libraries. Of course, they are is no _panacea_ in software development, and you have to evaluate each single project to understand what libraries, framework and technologies best fit your needs.
But hopefully by now you realize you have one more arrow in your quiver, and a particularly good one, I'd say.

Next step is definitely start experimenting with them - or even contributing to them, since - of course - they are open source projects.
