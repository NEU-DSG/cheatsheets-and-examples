# Leaflet maps setup
## L.map
### first step to setting up a map

```js
let map = L.map({


});

```


## L.tileLayer
### sets up the tiles to use as a map components

```js

L.tileLayer (url, {


}).addTo(map);

```


## L.geoJSON
###

```js
// 
L.geoJson(geoJsonData, {
    
}).addTo(map);

```

## L.icon
### setup the icon to use for popups

```js
let myIcon = L.icon({
	iconSize: '' ,
	iconUrl: '' ,
	iconAnchor: '' ,
	popupAnchor: '' ,
});
```

 
## L.marker

```js

// see https://leafletjs.com/reference.html#marker-l-marker
// see https://leafletjs.com/reference.html#latlng
// see https://leafletjs.com/reference.html#marker-option
let currentMarker = L.marker(coords, {
	icon: myIcon,
	
});

currentMarker.bindPopup(htmlString);


```


## Clustering markers
See https://github.com/Leaflet/Leaflet.markercluster
