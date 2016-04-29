# spam
spam.js is a small library to create modern [Canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) maps with [D3](https://github.com/mbostock/d3). It makes it easy to create static or zoomable maps with automatic centering and retina resolution. Custom projections, `d3.geo` path generators and multiple map features are also supported.

When using spam.js you are still in charge of painting everything. However it creates the canvas boilerplate and tries to handle as much as possible without putting constraints on the user. In order to improve performance, spam.js uses two painting phases. The 'static' layer is created once (for every zoom level) and should contain the majority of operations. After these operations complete, the canvas is saved into a picture.
Now every time the canvas needs a repaint (e.g. for hover effects), spam.js enters the 'dynamic' painting phase. Here we provide the option to paint dynamic content, while painting the 'static' image in the background.

In order to get the most out of spam.js, we encourage you to think about which parts of your map are static and dynamic beforehand and then use the appropriate callbacks. The more code runs in the 'static' functions, the faster spam.js will become.

## Getting started
spam.js depends on [D3](https://github.com/mbostock/d3), [rbush](https://github.com/mourner/rbush) and [TopoJSON](https://github.com/mbostock/topojson).

Due to bugs in D3 and TopoJSON you'll need to use our forks. Grab them [here](https://github.com/lukasappelhans/d3) and [here](https://github.com/lukasappelhans/topojson). We are expecting a PR soon.

Clone the repository (or download the zip) and include `spam.js` after D3, TopoJSON and rbush in your website.

````html
<script src="d3.min.js"></script>
<script src="topojson.min.js"></script>
<script src="rbush.min.js"></script>
<script src="spam.js"></script>
```

Here's the most basic map you can do:

```javascript
d3.json("map.json", function(error, d) {
    topojson.presimplify(d)

    var map = new StaticCanvasMap({
        element: "body",
        data: [
            {
                features: topojson.feature(d, d.objects["map"]),
                static: {
                    paintfeature: function(parameters, d) {
                        parameters.context.stroke()
                    }
                }
            }
        ]
    })
    map.init()
})
```

And that's it! A simple, static map in just a few lines of code! It will be automagically projected and centered in your container, nothing else needed.

## Examples
Explore some of the maps we already did with spam:
- [Static map](http://bl.ocks.org/martgnz/c48aa019de720fcd86030d3b07990d8d)
- [Zoomable map](http://bl.ocks.org/martgnz/fa8187c716c8a6d788eab7d51095b419)

---
- [Static map with labels](http://bl.ocks.org/martgnz/e5c0387a5bb675b061a2c0a9f573f86a)
- [Static choropleth with tooltip, legend and graticule](http://bl.ocks.org/martgnz/1c0fa3985d0a7b51437cdfd326cc2fda)
- [Static data visualization with custom projection](http://bl.ocks.org/martgnz/9023a67f080cca8b31ef5d6b1dcf4637)
- [Zoomable choropleth with multiple features and tooltip](http://bl.ocks.org/martgnz/a61c2da0e45a108c857e)
- [Custom projection I + graticule](http://bl.ocks.org/martgnz/d8bc3d6c29e712e3255f095671a51967)
- [Custom projection II + graticule](http://bl.ocks.org/martgnz/cce95512ca18c226b4cc)

## API
spam.js exports two classes: StaticCanvasMap and ZoomableCanvasMap. The only difference between them is that the latter provides a *zoom*-function which takes a feature as parameter.

Both constructors take a parameters object, while both of them accept the same members.

### element
This can be any term that works with d3.select() and is used to lookup the element that is used as the parent of the DOM-elements the spam.js code will create.

```javascript
element: ".container"
```

### width (optional)
Takes a value with the desired width of the map. If width is not specified, spam.js will use the width of the parent element.

```javascript
width: 960
```

### height (optional)
Takes a value with the desired height of the map. If height is not specified, spam.js will automagically define it by using the minimum height needed for the map.

```javascript
height: 500
```

### zoomScale (optional)
Takes a value between `0` and `1` which sets the zooming factor of the map. The default value is 0.5.

```javascript
zoomScaleFactor: 0.5
```

### projection (optional)
You can specify a projection to override the default (mercator). Declare it the same way as you would in D3, as it supports the usual stuff (`translate`, `center`, `scale`).

It supports custom projections, so you are able to use a projection from [`d3.geo.projection`](https://github.com/d3/d3-geo-projection/) or [`d3-composite-projections`](https://github.com/rveciana/d3-composite-projections) if you load them before:

```javascript
projection: d3.geo.robinson()
    .translate([960 / 2, 500 / 2])
    .scale(1000)
```

### data
This is an array of objects that define what will be rendered by spam.js. spam.js can render multiple datasets, the first element in the array gets painted first. The only mandatory property is *features* which takes a [FeatureCollection](https://github.com/mbostock/topojson/wiki/API-Reference#feature).

```javascript
data: [{
    property: input,
    property: input
}]
```

You can also nest different objects if you need to paint multiple maps.

```javascript
data: [{
        property: input,
        property: input
    },
    {
        property: input,
        property: input
    }
]
```

#### Properties
<a name="features" href="README.md#features">#</a> **features**: topojson.feature(d, d.objects["map"]).

The TopoJSON FeatureCollection you want to paint.

##### static

The following functions are used to construct the static part of the map.

<a name="prepaint" href="README.md#prepaint">#</a> **prepaint**: function(parameters, d) {}

Fires up before `paintfeature` and is useful for creating elements that only need to be painted once, for example [graticules](http://support.esri.com/en/knowledgebase/GISDictionary/term/graticule).

<a name="paintfeature" href="README.md#paintfeature">#</a> **paintfeature**: function(parameters, d) {}

The main painting event. This is where you can use canvas to paint the stroke of your map or fill it with colors to create a choropleth. This function gets called once for every feature in features.

<a name="postpaint" href="README.md#postpaint">#</a> **postpaint**: function(parameters, d) {}

It gets called once after all calls to `paintfeature` are done. You can use this event to create objects on the top of the map like labels, annotations, circles or bubbles.

##### dynamic

The following functions are used for dynamic painting. These functions get called a lot more often than the static ones, be careful when putting performance critical tasks in there.

Both functions get called when the map needs to be repainted, but the constraints are not touched. For example when the mouse moves over a different feature, these functions be called.

<a name="dynamicprepaint" href="README.md#dynamicprepaint">#</a> **prepaint**: function(parameters, hover) {}

This function gets called before the static image gets painted on the map. This makes it the first thing that gets called after the canvas is cleared. It takes `parameters` and `hover`, which contains the properties of the current hovered object.

<a name="dynamicpostpaint" href="README.md#dynamicpostpaint">#</a> **postpaint**: function(parameters, hover) {}

This function gets called after the static image gets painted. It takes `parameters` and `hover`, which contains the properties of the current hovered object. This is also useful for creating tooltips.

<a name="click" href="README.md#click">#</a> **click**: function(parameters, d) {}

This function gets called everytime the user clicks on the canvas. The 'd' parameter will be the clicked feature or null, in case the click was not on any feature.

## License
MIT © [newsapps.io](https://github.com/newsappsio).
