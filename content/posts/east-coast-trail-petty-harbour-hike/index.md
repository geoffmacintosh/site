+++
title = "East Coast Trail: Petty Harbour Hike"
author = ["Geoff MacIntosh"]
draft = false
+++

|                |              |
|----------------|--------------|
| Distance       | 8.60 km      |
| Duration       | 3:12         |
| Elevation Gain | 426 m        |
| Pace           | 22:30 min/km |

The [East Coast Trail](https://www.eastcoasttrail.com/) runs along Newfoundland's East coast. It's a network of impressively well-maintained and signposted hikes, almost entirely within view of the ocean. Since it's pretty great, I'm going to attempt to hike as many of them as possible. I've actually already hiked a bunch of them, but I didn't start making GPS tracks until now, so this is where I'm starting.

{{< figure src="2020-06-06-ect-petty-harbour.jpg" >}}

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
   integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ=="
      crossorigin=""/>

 <script src="https://unpkg.com/leaflet@1.6.0/dist/leaflet.js"
   integrity="sha512-gZwIG9x3wUXg2hdXF6+rVkLF/0Vi9U8D2Ntg4Ga5I5BZpVkVxlJWbSQtXPSiUTtC0TjtGOmxa1AJPuV0CPthew=="
         crossorigin=""></script>

<script src="https://api.tiles.mapbox.com/mapbox.js/plugins/leaflet-omnivore/v0.3.1/leaflet-omnivore.min.js"></script>

<div id="mapid" style="height:800px;"></div>

<script>
  var mymap = L.map('mapid');
  L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery &copy; <a href="https://www.mapbox.com/">Mapbox</a>',
      //maxZoom: 18,
      id: 'mapbox/outdoors-v11',
      tileSize: 512,
      zoomOffset: -1,
      accessToken: 'pk.eyJ1IjoiZ2VvZmZtYWNpbnRvc2giLCJhIjoiY2tiNDZ0czdmMHJ5djJvbGVmcTlkYzhuaSJ9.066rfzuYA3kUzlCMaoyzYw'
  }).addTo(mymap);

  var runLayer = omnivore.gpx('2020-06-06-ect-petty-harbour.gpx')
.on('ready', function() {
    mymap.fitBounds(runLayer.getBounds());
}).addTo(mymap);
  </script>

{{< figure src="2020-06-06-ect-petty-harbour-panorama.jpg" >}}
