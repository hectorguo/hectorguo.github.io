---
layout: post
title: "Mapbox Demo - American Universities"
categories: ['en']
tags: ['Visualization','Design']
published: True
---

This is a test for using the amazing Mapbox.

<link href='https://api.tiles.mapbox.com/mapbox.js/v2.2.3/mapbox.css' rel='stylesheet' />
<style>
    #map { height:500px; width:100%; position:absolute; left:0}
    .map-container {height:500px}
    #options {
        position: absolute;
        right: 0;
        background: #fff;
        width: 100px;
        padding:10px;
        border:none;
    }
    #options ul {
      list-style:none;
      margin: 0;
    }
</style>
<div class="map-container">
  <div id='map'></div>
  <nav id="options">
    <strong>TOEFL</strong>
    <ul>
      <li>
        <input type="checkbox" id="op1" value="100" checked />
        <label for="op1">
          <span>100+</span>
        </label>
      </li>
      <li>
        <input type="checkbox" id="op2" value="90" checked />
        <label for="op2">
          <span>90~100</span>
        </label>
      </li>
      <li>
        <input type="checkbox" id="op3" value="80" checked />
        <label for="op3">
          <span>80~90</span>
        </label>
      </li>
      <li>
        <input type="checkbox" id="op4" value="1" checked />
        <label for="op4">
          <span>0~80</span>
        </label>
      </li>
    </ul>
  </nav>
</div>
<script src='https://api.tiles.mapbox.com/mapbox.js/v2.2.3/mapbox.js'></script>
<script>
L.mapbox.accessToken = 'pk.eyJ1IjoiaGVjdG9yZ3VvIiwiYSI6ImNpaG0wZGcxcTBvOWp2ZmtsbHd2Y3h5eGIifQ.9gQvy3rj0VJbpl8TS745Kw';

var geoJson = [{
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-76.484781,42.447435]
      },
      "properties": {
          "title": "Cornell",
          "description": "Cornell University - 康奈尔大学",
          "TOEFL": 77,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-72.2908821,43.7044406]
      },
      "properties": {
          "title": "Dartmouth",
          "description": "Dartmouth College - 达特茅斯学院",
          "TOEFL": null,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-117.8442962,33.6404952]
      },
      "properties": {
          "title": "UCI",
          "description": "UC Irvine - 加州大学欧文分校",
          "TOEFL": 80,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-117.2362022,32.8800604]
      },
      "properties": {
          "title": "UCSD",
          "description": "UC San Diego - 加州大学圣地亚哥分校",
          "TOEFL": 80,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-122.2607286,37.8718992]
      },
      "properties": {
          "title": "UCB",
          "description": "UC Berkeley - 加州大学伯克利分校",
          "TOEFL": 90,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-74.0043697,40.740914]
      },
      "properties": {
          "title": "Cornell Tech",
          "description": "Cornell Tech - 康奈尔大学纽约校区",
          "TOEFL": 77,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-79.9447415,40.4424925]
      },
      "properties": {
          "title": "CMU",
          "description": "Carnegie Mellon University － 卡耐基梅隆大学",
          "TOEFL": 100,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-122.0619417,37.41043]
      },
      "properties": {
          "title": "CMU-SV",
          "description": "Carnegie Mellon University (Silicon Valley) - 卡耐基梅隆大学（硅谷校区）",
          "TOEFL": 95,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-120.6831581,46.2454008]
      },
      "properties": {
          "title": "UW",
          "description": "University of Washington - 华盛顿大学",
          "TOEFL": 100,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-76.944743,38.9869183]
      },
      "properties": {
          "title": "UMD",
          "description": "The University of Maryland - 马里兰大学",
          "TOEFL": 100,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }, {
      "type": "Feature",
      "geometry": {
          "type": "Point",
          "coordinates": [-88.2293502,40.1019523]
      },
      "properties": {
          "title": "UIUC",
          "description": "University of Illinois Urbana-Champaign - 伊利诺伊大学厄巴纳-香槟分校",
          "TOEFL": 79,
          "marker-color": "#63b6e5",
          "marker-size": "large",
          "marker-symbol": "college"
      }
  }];

var map = L.mapbox.map('map', 'mapbox.light')
  .setView([37.8, -96], 3);

var uLayer = L.mapbox.featureLayer().addTo(map);
var options = document.querySelectorAll('#options input');

// Custom Layer Icon
uLayer.on('layeradd', function(e) {
  var marker = e.layer,
      feature = marker.feature;
  marker.setIcon(L.divIcon({
        className: 'label',
        html: '<strong style="font-size:1.5em">'+feature.properties.title+'</strong>', // add content inside the marker,
        iconSize: [80, 30],
        popupAnchor: [-16, -16]
      }));
});

// Tooltip showup on mouseover
uLayer.on('mouseover', function(e) {
    e.layer.openPopup();
});
// uLayer.on('mouseout', function(e) {
//     e.layer.closePopup();
// });

uLayer.setGeoJSON(geoJson);

map.tileLayer.setOpacity(0.6);
map.fitBounds(uLayer.getBounds());
map.scrollWheelZoom.disable();


// Filter with TOEFL score
function filterLayer(op){
  uLayer.setFilter(function(f){
    var score = f.properties.TOEFL,
        filter = false;
    for (var i = 0; i < op.length; i++) {
      if(op[i].checked) {
       filter = op[i].value == 100 ? (score >= 100) : (score >= op[i].value && score < op[i-1].value);
      }
      if(filter) return true; // 只要满足其中一个区间就可以
    };
    return false;
  });
}

// ADD Checkbox Event
for (var i = 0; i < options.length; i++) {
  options[i].addEventListener('change', function(){
    filterLayer(options);
  });
};

// Initialize Filter
filterLayer(options);
</script>
