---
layout: post
title: Reactive graphs with Ractive.js and Paths.js
---

##Ractive


Ractive.js is a nice, lightweight library that allows you to easily add reactivity to your web page.

It's not the first lib that allows you to use reactive programming on html markpus: during the last year a number of them have been released, from [React](http://facebook.github.io/react/) to the amazing [AngularJS](http://angularjs.org) from Google.
Arguably, Angular is actually the most complete, and best mantained. But its many features makes the learning curve a bit steep, and definitely not lightweight.
Don't get me wrong, I used Angular in a few projects (for instance, [Tag Cloud](http://mlarocca.github.io/01-14-2013/tagcloud.html)) and it is awesome - especially because it allows you (or forces you) to adhere to the MCV pattern. But sometimes it is far more stuff than you'd need.

The Difference between Angular and Ractive are thoroughly addressed on [Ractive's blog](http://blog.ractivejs.org/posts/whats-the-difference-between-angular-and-ractive): mainly their scope is different, since Angular addresses also routing, validation, server communication, testing etc..., while Ractive leave all that stuff to you, allowing you to choose whatever solution you like for them - if you do need them.

If you are new to Ractive, there is nothing to worry about: it is super-easy, super-useful, and there is lot of documentation available. You might want to start here: [60 second setup](https://github.com/RactiveJS/Ractive/wiki/60-second-setup)

Here it is how the example in the setup would look like, assuming you downloaded the script for Ractive and saved it into the js subfolder inside your project.

{% highlight html %}
<!doctype html>
<html>
  <head>
  </head>
  <body>
    <div id='container'></div>

    <script src='js/Ractive.js'></script>

    <script id='myTemplate' type='text/ractive'>
        <p>{{greeting}}, {{recipient}}!</p>
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

To learn more about ractive's features, please take the [Interactive tutorial](http://learn.ractivejs.org/).
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
    <line class='freezing' x1='0' y1='{{ yScale(0) }}' x2='{{width}}' y2='{{ yScale(0) }}'/>

    {{#selectedCity}}
      
      <!-- the band -->
      <polygon fill='url(#gradient)' stroke='url(#gradient)' class='temperature-band' points='{{ getBand(months,xScale,yScale) }}'/>

      {{#months:i}}
        <!-- point markers for average highs -->
        <g class='marker' transform='translate({{ xScale(i+0.5) }},{{ yScale(high) }})'>
          <circle r='2'/>
          <text y='-10'>{{ format(high,degreeType) }}</text>
        </g>

        <!-- point markers for average lows -->
        <g class='marker' transform='translate({{ xScale(i+0.5) }},{{ yScale(low) }})'>
          <circle r='2'/>
          <text y='15'>{{ format(low,degreeType) }}</text>
        </g>
      {{/months}}
    {{/selectedCity}}
  </svg>
{% endhighlight %}

If you know D3, Raphael, or other similar SVG libraries, you are more used to create SVG nodes from JavaScript. Here, instead, thanks to reactive programming, we take a different direction, creating the chart inside the html file; this enables us to better separate data from the structure of the charts.

##Paths.js

Paths.js is another minimal library by [Andrea Ferretti](https://github.com/andreaferretti) that is, explicitely and by design, oriented to support reactive programming by generating SVG paths that can be used with template engines. It looks as a perfect match for Ractive, and it actually is: take a look at this [demo](http://andreaferretti.github.io/paths-js-demo/) - source available on [GitHub](https://github.com/andreaferretti/paths-js-demo).
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
        <select value='{{selectedDataset}}' on-change='loadData'>
          {{#datasets}}
            <option value='{{.["filename"]}}'>{{.["label"]}}</option>
          {{/datasets}}
        </select>

        <div class="panel-body">
          <p class="alert alert-info">Here is a pie chart example. Sectors are clickable.</p>

          <svg width=375 height=400>
            <g transform="translate(200, 200)">
              {{# Pie({center: center, r: r, R: R, data: countries, accessor: accessor, colors: colors}) }}
                {{# curves:num }}
                  <g transform="translate({{ move(sector.centroid, expanded[num]) }})">
                    <linearGradient id = "grad-{{ num }}">
                      <stop stop-color = "{{ color_string(color) }}" offset = "0%"/>
                      <stop stop-color = "{{ lighten(color) }}" offset = "100%"/>
                    </linearGradient>
                    <path on-click="expand" d="{{ sector.path.print() }}" fill="url(#grad-{{ num }})" />
                    <text text-anchor="middle" transform="translate({{ point(sector.centroid) }})">{{ item.name }}</text>
                  </g>
                {{/ curves }}
              {{/ end of pie}}
            </g>
          </svg>

          {{# countries: num }}
            {{# expanded[num] == 1}}
              <div class="country-info">
                <h4>{{ name }}</h4>
                <p>Population: <span class="label label-info">{{ population }}</span></p>
              </div>
            {{/ end if }}
          {{/ countries }}
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
          .done( function ( data ) {
            data = JSON.parse(data);
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
            datasets: [{label: "Mixed", filename: "json/countries.json"}, {label: "Europe", filename: "json/europe.json"}, {label: "Asia", filename: "json/asia.json"}],
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

The rest of the code (a couple of utility js modules, the css file, and the jsons in the example) can be found here [on github](https://github.com/mlarocca/PathsJS-demo)

You can take a look at the [working demo](http://mlarocca.github.io/pathsjsdemo/demo.html)

As you can see, I took a step further with respect to the pathjs demo, allowing the chart to be dynamically populated, and animating the transitions. 
The effect, in my opinion, is stunning, especially considering that it took me like 5 minutes to add this new feature to the example.
Of course, my contribution was really minimal: it's just that using both libraries together makes it extremely simple to manipulate svg elements, and animations are a benefit you get basically for free.
Notice, for example, how you declare the chart, and then iterate on its building blocks:

{% highlight html %}
  {{# Pie({center: center, r: r, R: R, data: countries, accessor: accessor, colors: colors}) }}
    {{# curves:num }}
      <g transform="translate({{ move(sector.centroid, expanded[num]) }})">
        <linearGradient id = "grad-{{ num }}">
          <stop stop-color = "{{ color_string(color) }}" offset = "0%"/>
          <stop stop-color = "{{ lighten(color) }}" offset = "100%"/>
        </linearGradient>
        <path on-click="expand" d="{{ sector.path.print() }}" fill="url(#grad-{{ num }})" />
        <text text-anchor="middle" transform="translate({{ point(sector.centroid) }})">{{ item.name }}</text>
      </g>
    {{/ curves }}
  {{/ end of pie}}    
{% endhighlight %}

Key points:

1. Ractive allows you to iterate over objects returned by function calls. It lists the dependencies for the expression, so that every time one of the objects it depends on is changed, the nodes affected by this change (and only those nodes) are redrawn.

2. Paths.js Pie method returns an object with an array, curves, listing the shapes to draw. This allows you to separate the data from their visualization, and make the latter more explicit, improving manutenability and easiness of access, simply by moving it to a template rather than keeping it hidden in the belly of a JS library.

3. Scoping. Even for anonymous objects (like the one returned from the call to Pie): curves is actually Pie(...).curves, and sector in the next line is Pie(...).curves.sector

4. Attributes. They are automatically updated too.

5. The number of elements doesn't matter, nor does their order: Ractive takes care of every thing. It even animates elements creation/disposal (see transitions on dataset change).