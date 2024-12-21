<!DOCTYPE html>
<html>
<head>
	<title>Leaflet Layers Control Example (Pekanbaru)</title>
	<meta charset="utf-8" />

	<meta name="viewport" content="width=device-width, initial-scale=1.0">

	<link rel="stylesheet" href="https://cdn.leafletjs.com/leaflet/v0.7.7/leaflet.css" />

	<style>
      html, body, #map{
        height: 100%;
        margin: 0px;
        padding: 0px
      }
			.leaflet-control-attribution { display:none!important}

		.info {
			padding: 6px 8px;
			font: 14px/16px Arial, Helvetica, sans-serif;
			background: white;
			background: rgba(255,255,255,0.8);
			box-shadow: 0 0 15px rgba(0,0,0,0.2);
			border-radius: 5px;
		}
		.info h4 {
			margin: 0 0 5px;
			color: #777;
		}

		.legend {
			text-align: left;
			line-height: 18px;
			color: #555;
		}
		.legend i {
			width: 18px;
			height: 18px;
			float: left;
			margin-right: 8px;
			opacity: 0.7;
		}
	</style>
</head>
<body>
	<div id="map"></div>

	<script src="https://cdn.leafletjs.com/leaflet/v0.7.7/leaflet.js"></script>
	<script type="text/javascript" src="https://gist.github.com/kampar/430220edbe06c3237e66/raw/ca02fb8bcffc0523832070d231ba9592918e0b3f/kelurahan.js"></script>
	<script type="text/javascript">
		//perhatikan bahwa format di sini adalah LatLong, sedangkan di JSON adalah LongLat
		var sukajadi=[0.526452500644619,101.43842513074193]; 
		
		//senter map di pusatnya 
		// eh ... kamsud saya pusatkan peta di sukajadi, zoom 14
		var map = L.map('map').setView(
			sukajadi
			, 11
		);

		//var map = L.map('map').setView([37.8, -96], 4);

		L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpandmbXliNDBjZWd2M2x6bDk3c2ZtOTkifQ._QA7i5Mpkd_m30IGElHziw', {
			maxZoom: 18,
			attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, ' +
				'<a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
				'Imagery © <a href="http://mapbox.com">Mapbox</a>',
			id: 'mapbox.light'
		}).addTo(map);


		// control that shows state info on hover
		var info = L.control();

		info.onAdd = function (map) {
			this._div = L.DomUtil.create('div', 'info');
			this.update();
			return this._div;
		};

		info.update = function (props) {
			this._div.innerHTML = '<h4>Mahasiswa Sistem Informasi</h4>' +  (props ?
				'<b>' + props.Kelurahan + '</b><br />' + props.MhsSIF + ' orang'
				: 'Gerakkan mouse Anda');
		};

		info.addTo(map);


		// get color depending on MhsSIF value
		// perhatikan bahwa saya menggunakan pretty break 8 warna
		// karena kita tahu bahwa data MhsSIF adalah data palsu random dari 0 - 100
		function getColor(d) {
			return d > 87  ? '#800026' :
			       d > 75  ? '#BD0026' :
			       d > 62  ? '#E31A1C' :
			       d > 50  ? '#FC4E2A' :
			       d > 37  ? '#FD8D3C' :
			       d > 25  ? '#FEB24C' :
			       d > 12  ? '#FED976' :
			                 '#FFEDA0';
		}

		function style(feature) {
			return {
				weight: 2,
				opacity: 1,
				color: 'white',
				dashArray: '3',
				fillOpacity: 0.7,
				fillColor: getColor(feature.properties.MhsSIF)
			};
		}

		function highlightFeature(e) {
			var layer = e.target;

			layer.setStyle({
				weight: 5,
				color: '#666',
				dashArray: '',
				fillOpacity: 0.7
			});

			if (!L.Browser.ie && !L.Browser.opera) {
				layer.bringToFront();
			}

			info.update(layer.feature.properties);
		}

		var geojson;

		function resetHighlight(e) {
			geojson.resetStyle(e.target);
			info.update();
		}

		function zoomToFeature(e) {
			map.fitBounds(e.target.getBounds());
		}

		function onEachFeature(feature, layer) {
			layer.on({
				mouseover: highlightFeature,
				mouseout: resetHighlight,
				click: zoomToFeature
			});
		}

		geojson = L.geoJson(kelurahan, {
			style: style,
			onEachFeature: onEachFeature
		}).addTo(map);

		//map.attributionControl.addAttribution('Population data &copy; <a href="http://census.gov/">US Census Bureau</a>');


		var legend = L.control({position: 'bottomright'});

		legend.onAdd = function (map) {

			var div = L.DomUtil.create('div', 'info legend'),
				//grades = [0, 10, 20, 50, 100, 200, 500, 1000],
				grades = [0, 12, 25, 37, 50, 62, 75, 87], //pretty break untuk 8
				labels = [],
				from, to;

			for (var i = 0; i < grades.length; i++) {
				from = grades[i];
				to = grades[i + 1];

				labels.push(
					'<i style="background:' + getColor(from + 1) + '"></i> ' +
					from + (to ? '&ndash;' + to : '+'));
			}

			div.innerHTML = '<h4>Legenda:</h4><br>'+labels.join('<br>');
			return div;
		};

		legend.addTo(map);

	</script>
</html>
