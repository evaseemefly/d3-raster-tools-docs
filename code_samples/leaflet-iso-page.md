---
layout: default
title: "Using Leaflet - isobands and isolines"
---
Using Leaflet - isobands and isolines
-------------------------------------
Leaflet example from the [Leaflet section]({{ site.baseurl }}{% post_url 2016-12-13-isolines %}) section. This example only contains the isobands and isolines layers.

<iframe frameborder="no" border="0" scrolling="no" marginwidth="0" marginheight="0" width="690" height="510" src="{{ site.baseurl }}/code_samples/leaflet.html"></iframe>

{% highlight js %}
<!DOCTYPE html>
<html>
<head>

	<title>GeoJSON tutorial - Leaflet</title>

	<meta charset="utf-8" />
	<meta name="viewport" content="width=device-width, initial-scale=1.0">

	<link rel="stylesheet" href="https://unpkg.com/leaflet@1.0.2/dist/leaflet.css" />
	<script src="https://unpkg.com/leaflet@1.0.2/dist/leaflet.js"></script>
  <script src="geotiff.min.js"></script>
  <script src="raster-marching-squares.min.js"></script>

	<style>
		#map {
			width: 680px;
			height: 500px;
		}
	</style>


</head>
<body>

<div id='map'></div>
<script>

var xhr = new XMLHttpRequest();
xhr.open('GET', 'vardah.tiff', true);
xhr.responseType = 'arraybuffer';
xhr.onload = function(e) {

    var tiff = GeoTIFF.parse(this.response);
    var image = tiff.getImage();
    var tiffWidth = image.getWidth();
    var tiffHeight = image.getHeight();
    var rasters = image.readRasters();
    var tiepoint = image.getTiePoints()[0];
    var pixelScale = image.getFileDirectory().ModelPixelScale;
    var geoTransform = [tiepoint.x, pixelScale[0], 0, tiepoint.y, 0, -1*pixelScale[1]];


    var pressData = new Array(tiffHeight);
    var tempData = new Array(tiffHeight);
    var uData = new Array(tiffHeight);
    var vData = new Array(tiffHeight);
    var spdData = new Array(tiffHeight);
    for (var j = 0; j<tiffHeight; j++){
        pressData[j] = new Array(tiffWidth);
        tempData[j] = new Array(tiffWidth);
        uData[j] = new Array(tiffWidth);
        vData[j] = new Array(tiffWidth);
        spdData[j] = new Array(tiffWidth);
        for (var i = 0; i<tiffWidth; i++){
            pressData[j][i] = rasters[0][i + j*tiffWidth];
            tempData[j][i] = rasters[1][i + j*tiffWidth];
            uData[j][i] = rasters[2][i + j*tiffWidth];
            vData[j][i] = rasters[3][i + j*tiffWidth];
            spdData[j][i] = 1.943844492 * Math.sqrt(uData[j][i]*uData[j][i] + vData[j][i]*vData[j][i]);

        }
    }

    var intervalsSpd = [0, 8, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36, 38, 40, 42,
      44, 46, 48, 50, 52, 56, 60, 64, 68, 72, 76, 80, 84, 88, 92, 96 ];
    var bandsWind = rastertools.isobands(spdData, geoTransform, intervalsSpd);

    function getColor(d) {
    return d > 92   ? '#643c32' :
           d > 88   ? '#643c32' :
           d > 84   ? '#a50000' :
           d > 80   ? '#c10000' :
           d > 76   ? '#e11400' :
           d > 72   ? '#ff3200' :
           d > 68   ? '#ff6000' :
           d > 64   ? '#ffa100' :
           d > 60   ? '#ffc13c' :
           d > 56   ? '#ffe978' :
           d > 52   ? '#c9ffbf' :
           d > 50   ? '#b5fbab' :
           d > 48   ? '#97f58d' :
           d > 46   ? '#78f572' :
           d > 44   ? '#50ef50' :
           d > 42   ? '#36d33c' :
           d > 40   ? '#1eb31e' :
           d > 38   ? '#0ea10e' :
           d > 36   ? '#e1ffff' :
           d > 34   ? '#b5f1fb' :
           d > 32   ? '#97d3fb' :
           d > 30   ? '#78b9fb' :
           d > 28   ? '#50a5f5' :
           d > 26   ? '#3c97f5' :
           d > 24   ? '#2883f1' :
           d > 22   ? '#1e6eeb' :
           d > 20   ? '#1464d3' :
           d > 18   ? '#646464' :
           d > 16   ? '#979797' :
           d > 14   ? '#bababa' :
           d > 12   ? '#d1d1d1' :
           d > 8    ? '#e5e5e6' :
           d > 0   ? '#ffffff' :
                    '#ffffff';
            }


    function style(feature) {
    return {
        fillColor: getColor(feature.properties[0].lowerValue),
        weight: 2,
        opacity: 1,
        color: getColor(feature.properties[0].lowerValue),
        dashArray: '3',
        fillOpacity: 0.5
      };
    }

    var bandsWindLayer = L.geoJson(bandsWind, {
      style: style
    });


    var intervalsPress = [970, 972, 974, 976, 978, 980, 982, 984, 986, 988, 990, 992, 994, 996, 998,
      1000, 1002, 1004, 1006, 1008, 1010, 1012, 1014, 1016, 1018, 1020, 1022, 1024, 1026, 1028];
    var isobars = rastertools.isolines(pressData, geoTransform, intervalsPress);

    var isobarsLayer = L.geoJSON(isobars, {
		    style: {
          "color": "#333",
          "weight": 2,
          "opacity": 0.65
        }
	 });


   var baseLayer = L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpandmbXliNDBjZWd2M2x6bDk3c2ZtOTkifQ._QA7i5Mpkd_m30IGElHziw', {
   	maxZoom: 18,
   	attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, ' +
   		'<a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
   		'Imagery © <a href="http://mapbox.com">Mapbox</a>',
   	id: 'mapbox.light'
   });

   var map = L.map('map', {
     layers: [baseLayer, bandsWindLayer]
   }).setView([13, 81], 6);


   L.control.layers(null, {
    "Wind speed": bandsWindLayer,
    "Pressure": isobarsLayer
    }).addTo(map);


  map.on('click', function(e) {
    var xTiff = (e.latlng.lng - geoTransform[0])/geoTransform[1];
    var yTiff = ( e.latlng.lat - geoTransform[3])/geoTransform[5];
    var temp = tempData[Math.round(yTiff)][Math.round(xTiff)];
    var press = pressData[Math.round(yTiff)][Math.round(xTiff)];
    var uValue = uData[Math.round(yTiff)][Math.round(xTiff)];
    var vValue = vData[Math.round(yTiff)][Math.round(xTiff)];
    var spd = Math.sqrt(uValue*uValue + vValue*vValue);
    var dir = 270 + (Math.atan2(-vValue,uValue)*180/Math.PI);
    if(dir<0){dir = dir + 360;}
    if(dir>360){dir = dir - 360;}

    L.popup()
      .setLatLng(e.latlng)
      .setContent("Wind speed: " + spd.toFixed(1) + " kt <br/>Wind dir: " + dir.toFixed(0) +"º <br/>Temp: " + temp.toFixed(1) + " C<br/>Pressure: " + press.toFixed(0) + " hPa")
      .openOn(map);
  });
  };
  xhr.send();
</script>
</body>
</html>

{% endhighlight %}
