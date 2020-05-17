# chartist-plugin-legend

Implements a legend for [Chartist](https://github.com/gionkunz/chartist-js) charts.

**[Demo](https://codeyellowbv.github.io/chartist-plugin-legend/)**

## Why this Fork

Since I needed to use this plugin with a Vue.js project and I needed to append the legend to a DOM element which wasn't yet rendered at the time the plugin was loaded, 
I needed to be able to pass the id of the element and leave the trouble of retrieving the DOM element to the plugin itself.
In addition to this, now if you assign a className to your series, the corresponding legend gets the same class too.
Further to this, you can now trigger a reset on the legends in case you need to update your datas and you want the legends to update too.
Please see details below.

## Install

```
$ npm install @matteoraf/chartist-plugin-legend --save
```

As styles are very different with each project, no CSS is included. You can copy paste this to use as base:

```scss
.ct-legend {
    position: relative;
    z-index: 10;

    li {
        position: relative;
        padding-left: 23px;
        margin-bottom: 3px;
    }

    li:before {
        width: 12px;
        height: 12px;
        position: absolute;
        left: 0;
        content: '';
        border: 3px solid transparent;
        border-radius: 2px;
    }

    li.inactive:before {
        background: transparent;
    }

    &.ct-legend-inside {
        position: absolute;
        top: 0;
        right: 0;
    }

    @for $i from 0 to length($ct-series-colors) {
        .ct-series-#{$i}:before {
            background-color: nth($ct-series-colors, $i + 1);
            border-color: nth($ct-series-colors, $i + 1);
        }
    }
}
```

If you are using this within a Vue.js component, you need to wrap it in a `<style lang='scss'>` tag.
Don't forget to import or define the `$ct-series-colors` variable


```scss
<style lang='scss'>
  $ct-series-colors: (
          #d70206,
          #f05b4f,
          #f4c63d,
          #d17905,
          #453d3f,
          #59922b,
          #0544d3,
          #6b0392,
          #f05b4f,
          #dda458,
          #eacf7d,
          #86797d,
          #b2c326,
          #6188e2,
          #a748ca
  ) !default;
  // Your scss code here
</style>
```

Since now the series classNames are passed to the legend element too, you can do something like the following.
This way, your series and legend will have the same colors and you can easily change all of them by just changing the $data-color-map
```scss
<style lang='scss'>
    $data-color-map: ('first-series-class': #f05b4f, 'second-series-class': #f4c63d, 'third-series': #2b922d);
  
    @each $cat, $color in $data-color-map {
      .#{$cat} {
        .ct-point,
        .ct-line,
        .ct-bar,
        .ct-slice-donut {
          stroke: $color
        }
        .ct-slice-pie,
        .ct-area {
          fill: $color
        }
        /* this renders as li.series-name:before */
      @at-root li#{&}:before {
                 background-color: $color;
                 border-color: $color;
               }
      }
    }

    .ct-legend {
        position: relative;
        z-index: 10;
        list-style: none;
    
        li {
          position: relative;
          margin-left: 10px;
          padding-left: 20px;
          margin-bottom: 3px;
          display: inline-block;
          cursor: pointer;
        }
    
        li:before {
          width: 12px;
          height: 12px;
          position: absolute;
          left: 5px;
          bottom: 5px;
          content: '';
          border-style: solid;
          border-width: 3px;
          border-radius: 2px;
        }
    
        li.inactive:before {
          background: transparent;
        }
    
        &.ct-legend-inside {
           position: absolute;
           top: 0;
           right: 0;
        }
    }
</style>
```




## Usage

In an example chart:

```js
require('@matteoraf/chartist-plugin-legend');

new Chartist.Bar('.ct-chart', data, {
        stackBars: true,
        plugins: [
            Chartist.plugins.legend()
        ]
    },
)
```

## Usage in a Vue.js project with the @matteoraf/vue-chartist component

Import the plugin together with chartist in a plugin file or in your main.js
```js
import Vue from 'vue'
import '@matteoraf/chartist/dist/chartist.min.css'
import '@matteoraf/chartist-plugin-legend'

Vue.use(require('@matteoraf/vue-chartist'))
```

Then use it in any of your components the same way you'd normally do:

```html
<chartist
        :data="data"
        :event-handlers="eventHandlers"
        :options="options"
        :ratio="ratio"
        :responsive-options="responsiveOptions"
        :type="type"
        style=""
      />
```

```js
data: {}
options: {
    plugins: [
        this.$chartist.plugins.legend({...}),
            ]
    }
```

## Using the reset method
In the page rendering the chart we'd have set up something like this:
```js
var resetFnc;
function chartResetCallback(chart, reset) {
     resetFnc = reset
}
```

or, if you are in a Vue Component
```js
data () {
    return {
        resetFnc: undefined,
    }
},
methods: {
    chartResetCallback (chart, reset) {
            this.resetFnc = reset
          },
}
```


And then we load our plugin by passing `chartResetCallback` as an option to `resetCallback`

```js
Chartist.plugins.legend({
   resetCallback: chartResetCallback,
})
```

What happens is that we are "passing in" our `chartResetCallback` method to the plugin.
The plugin runs it and this will "pass out" its reset function as an argument to `chartResetCallback()` and it will assign the internal reset function to the `resetFnc` variable.

So now, on our page, we can call `resetFnc()` and this will trigger the internal reset function
                      


| __Option__ | __Description__ | __Type__ | __Default__ |
| ---        | ---             | ---      | ---         |
| `className` | Adds a class to the `ul` element. | `string` | `''` |
| `clickable` | Sets the legends clickable state; setting this value to `false` disables toggling graphs on legend click. | `bool` | `true` |
| `legendNames` | Sets custom legend names. By default the `name` property of the series will be used if none are given. Multiple series can be associated with a legend item using this property as well. See examples for more details. | `mixed` | `false` |
| `onClick` | Accepts a function that gets invoked if `clickable` is true. The function has the `chart`, and the click event (`e`), as arguments. | `mixed` | `false` |
| `classNames` | Accepts a array of strings as long as the chart's series, those will be added as classes to the `li` elements. | `mixed` | `false` |
| `removeAll` | Allow all series to be removed at once. | `bool` | `false` |
| `position` | Sets the position of the legend element. `top`, `bottom` or the `id` of any DOM2 Element are currently accepted. If a DOM Element is given, the legend will be appended as it's last child. | `string` | `'top'` |
| `resetCallback` | Must be a function with two parameters. The first is the `chart` instance and the second is the internal `reset` function. | `function` | `undefined`
