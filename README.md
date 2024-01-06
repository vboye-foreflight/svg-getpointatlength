# svg-getpointatlength
Calculates a path's length or points at length based on raw pathdata.  
This library aims to work as a workaround to emulate natively supported browser methods `getTotalLength()` and `getPointAtLength()` in a non-rendered environment such as node or virtual DOM applications.  

The provided methods calculate points at lengths by measuring all segments lengths and saving them in a reusable lookup object.    

This way you can efficiently calculate hundreds of points on a path without sacrificing too much performance – unlike the quite expensive native `getPointAtlength()` method.


### Usage: browser

Load JS locally or via cdn
```
<script src="https://cdn.jsdelivr.net/gh/herrstrietzel/svg-getpointatlength@main/getPointAtLengthLookup.js"></script>
```

or minified version (~ 9KB/4KB gzipped) 
```
<script src="https://cdn.jsdelivr.net/gh/herrstrietzel/svg-getpointatlength@main/getPointAtLengthLookup.min.js"></script>
```

**Example: calculate pathDength from pathData**  

```
// import methods
let { getPathLengthLookup, getPointAtLengthLookup, getPathLengthFromD } = pathDataLength;


let d = `M3,7 
        L13,7 
        m-20,10 
        l10,0 
        V27 
        H23 
        v10 
        h10
        C 33,43 38,47 43,47 
        c 0,5 5,10 10,10
        S 63,67 63,67       
        s -10,10 10,10
        Q 50,50 73,57
        q 20,-5 0,-10
        T 70,40
        t 0,-15
        A 5, 10 45 1040,20  
        a5,5 20 01 -10,-10
        Z `

// measure path and save metrics in lookup object
let pathLengthLookup = getPathLengthLookup(d)
let totalLength = pathLengthLookup.totalLength

console.log(totalLength)

// point at length
let pt = pathLengthLookup.getPointAtLengthLookup(totalLength/2)
console.log(pt)
```


### Usage: node  

```
npm install svg-getpointatlength
```

```
var pathDataLength = require("svg-getpointatlength");
let { getPathLengthLookup, getPointAtLengthLookup, getPathLengthFromD } = pathDataLength;

let d = `M3,7 
        L13,7 
        m-20,10 
        l10,0 
        V27 
        H23 
        v10 
        h10
        C 33,43 38,47 43,47 
        c 0,5 5,10 10,10
        S 63,67 63,67       
        s -10,10 10,10
        Q 50,50 73,57
        q 20,-5 0,-10
        T 70,40
        t 0,-15
        A 5, 10 45 1040,20  
        a5,5 20 01 -10,-10
        Z `

// measure path and save metrics in lookup object
let pathLengthLookup = getPathLengthLookup(d)
let totalLength = pathLengthLookup.totalLength

console.log(totalLength)

// point at length
let pt = pathLengthLookup.getPointAtLengthLookup(totalLength/2)
console.log(pt)
```


**Save path/segment metrics as a reusable lookup for further calculations**  

Any pathdata (passed as a `d` string ) is first parsed and normalized to all absolute coordinates also approximating `A` arc commands by cubic béziers representations.  

The total length of a path is calculated by measuring and adding up each segment's length via the appropriate formula (linetos and quadratics) or appromximation (cubic béziers).  

Since we can't directly translate `t` values (using the de Casteljau's algorithm) to an actual "proportional" length  (t=0.5 quite often doesn't equal 50% of the curve's length) we measure/approximate path lengths at `t` intervals (36 by default) and save these length-at-t values in the lookup.  

When searching for a point at length we use this lookup to find the closest t based sub-segment length and calculate an interpolated new `t` value to get a close approximation.


## Similar libraries
[rveciana's "svg-path-properties"](https://github.com/rveciana/svg-path-properties) 


## Demos
<a href="https://codepen.io/herrstrietzel/pen/KKEzdPd">Get pathlength and point and lengths</a>

## Credits
* Mike 'Pomax' Kamermans for explaining the theory. See Stackoverflow post ["Finding points on curves in HTML 5 2d Canvas context"](https://stackoverflow.com/questions/3570309/finding-points-on-curves-in-html-5-2d-canvas-context/#76773275)  
* obviously, Dmitry Baranovskiy – a lot of these helper functions originate either from Raphaël or snap.svg – or are at least heavily inspired by some helpers from these libraries
* Jarek Foksa for developping the great [getPathData() polyfill](https://github.com/jarek-foksa/path-data-polyfill) – probably the most productive contributor to the ["new" W3C SVGPathData interface draft](https://svgwg.org/specs/paths/#InterfaceSVGPathData).
* puzrin's for [svgpath library](https://github.com/fontello/svgpath) providing for instance a great [arc-to-cubic approximation](https://github.com/fontello/svgpath/blob/master/lib/a2c.js) 