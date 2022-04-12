---
layout: home
title: Atmospheric Winds
key: winds
full_width: true
---

<link rel="stylesheet" href="https://js.arcgis.com/3.20/esri/css/esri.css">

<header>
  <style>
        html,body {
          width:100%; -->
          height: 100%;
          margin: 0;
          padding: 0px 0 0 0;
        }

        #mapCanvas {
          padding:0;
        }
        #credit {
          position: relative;
          bottom: 20px;
          left: 10px;
          color: #fff;
          font-size: 14px;
        }

        #credit a {
          color: #08c;
        }
  </style>

  <script src="./assets/wind-js/windy.js"></script>
  <script>
    var dojoConfig = {
      paths: {
        plugins: "/assets/wind-js/plugins"
      }
    };
  </script>

  <script src="https://js.arcgis.com/3.20compact/"></script>
  <script>
    var map, rasterLayer;
    var canvasSupport;

    require([
      "esri/map", "esri/layers/ArcGISTiledMapServiceLayer",
      "esri/domUtils", "esri/request",
      "dojo/parser", "dojo/number", "dojo/json", "dojo/dom",
      "dijit/registry", "plugins/RasterLayer","esri/layers/WebTiledLayer",
      "esri/config",
      "dojo/domReady!"
    ], function(
      Map, ArcGISTiledMapServiceLayer,
      domUtils, esriRequest,
      parser, number, JSON, dom,
      registry, RasterLayer, WebTiledLayer, esriConfig
    ){
      parser.parse();
      // does the browser support canvas?
      canvasSupport = supports_canvas();

      map = new Map("mapCanvas", {
        center: [-75.076, 39.132],
        zoom: 4,
        basemap: "dark-gray",
      });

      map.on("load", mapLoaded);

      function mapLoaded() {

        // Add raster layer
        if ( canvasSupport ) {
          rasterLayer = new RasterLayer(null, {
            opacity: 0.55
          });
          map.addLayer(rasterLayer);

          map.on("extent-change", redraw);
          map.on("resize", function(){});
          map.on("zoom-start", redraw);
          map.on("pan-start", redraw);

          var layersRequest = esriRequest({
            url: './assets/wind-js/zeus.json',
            content: {},
            handleAs: "json"
          });
          layersRequest.then(
            function(response) {
              windy = new Windy({ canvas: rasterLayer._element, data: response });
              redraw();
          }, function(error) {
              console.log("Error: ", error.message);
          });

        } else {
          dom.byId("mapCanvas").innerHTML = "This browser doesn't support canvas. Visit <a target='_blank' href='http://www.caniuse.com/#search=canvas'>caniuse.com</a> for supported browsers";
        }
      }

      // does the browser support canvas?
      function supports_canvas() {
        return !!document.createElement("canvas").getContext;
      }

      function redraw(){

        rasterLayer._element.width = map.width;
        rasterLayer._element.height = map.height;

        windy.stop();

        var extent = map.geographicExtent;
        setTimeout(function(){
          windy.start(
            [[0,0],[map.width, map.height]],
            map.width,
            map.height,
            [[extent.xmin, extent.ymin],[extent.xmax, extent.ymax]]
          );
        },500);
      }
    });
  </script>
</header>


<body class="">
   <div id="mapCanvas" style="height:600px;">
   </div>
  <div id="credit">Zeus AI</div>

</body>