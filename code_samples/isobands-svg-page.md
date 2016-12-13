---
layout: default
title: "Drawing isobands"
---
Drawing isobands - SVG
----------------------
SVG example from the [drawing isobands]({{ site.baseurl }}{% post_url 2016-12-13-isobands %}) section.

<iframe frameborder="no" border="0" scrolling="no" marginwidth="0" marginheight="0" width="690" height="510" src="{{ site.baseurl }}/code_samples/isobands-svg.html"></iframe>

{% highlight js %}
<!DOCTYPE html>
<meta charset="utf-8">
<style>

</style>
<body>

<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="geotiff.min.js"></script>
<script src="raster-marching-squares.min.js"></script>
<script src="http://d3js.org/topojson.v1.min.js"></script>
<script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>

<script>
var width = 680,
    height = 500;
var projection = d3.geoAzimuthalEqualArea()
    .rotate([-55.5, -24])
    .scale(1100);

var path = d3.geoPath()
    .projection(projection);

var svg = d3.select("body").append("svg")
    .attr("width", width)
    .attr("height", height);

d3.request("tz850.tiff")
  .responseType('arraybuffer')
  .get(function(error, tiffData){
d3.json("world-110m.json", function(error, topojsonData) {
  var countries = topojson.feature(topojsonData, topojsonData.objects.countries);

  var tiff = GeoTIFF.parse(tiffData.response);
  var image = tiff.getImage();
  var rasters = image.readRasters();
  var tiepoint = image.getTiePoints()[0];
  var pixelScale = image.getFileDirectory().ModelPixelScale;
  var geoTransform = [tiepoint.x, pixelScale[0], 0, tiepoint.y, 0, -1*pixelScale[1]];

  var tempData = new Array(image.getHeight());
  for (var j = 0; j<image.getHeight(); j++){
      tempData[j] = new Array(image.getWidth());
      for (var i = 0; i<image.getWidth(); i++){
          tempData[j][i] = rasters[1][i + j*image.getWidth()];
      }
  }

  var intervalsTemp = [14,17,20,23,26,29, 32, 35, 38];
  var bandsTemp = rastertools.isobands(tempData, geoTransform, intervalsTemp);
  var colorScale = d3.scaleSequential(d3.interpolateRdBu)
      .domain([38, 14]);
  bandsTemp.features.forEach(function(d, i) {
    svg.insert("path", ".streamline")
        .datum(d)
        .attr("d", path)
        .style("fill", colorScale(intervalsTemp[i]))
        .style("stroke", "None");
  });

  svg.insert("path", ".map")
      .datum(countries)
      .attr("d", path)
      .style("opacity", "0.4")
      .style("fill", "#ccc")
      .style("stroke", "#777")
      .style("stroke-width", "1.5px");

});
});
</script>

</body>
{% endhighlight %}
