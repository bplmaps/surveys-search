---
theme: dashboard
title: Survey search tool
toc: false
---

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css">
```

```js
import L from "leaflet";
import "leaflet.markercluster";
import _ from "lodash";
```

```js
const masonData = d3.csv(
  "https://raw.githubusercontent.com/bplmaps/lmec-digital-library-metadata/main/survey-data/mason.csv"
)

const bellamyData = d3.csv(
  "https://raw.githubusercontent.com/bplmaps/lmec-digital-library-metadata/main/survey-data/Bellamy.csv"
)

const auctionData = d3.csv(
  "https://raw.githubusercontent.com/bplmaps/lmec-digital-library-metadata/refs/heads/main/survey-data/auction-lot.csv"
)
```

```js
const data = d3.merge([masonData, bellamyData, auctionData])
```

### 🔎 Enter a search term

```js
const search = view(Inputs.search(data, {
  placeholder: "Search a city, street, name, etc."
}));
```

<div class="grid grid-cols-1" style="grid-auto-rows: 300px;" width="640">
  <div id="map" style="padding: 0">
  </div>
</div>

```js
const selected = view(Inputs.table(search, {
  columns: [
    "identifier",
    "city_town",
    "streets",
    "survey_title",
    "date",
    "auction_date",
    "property_owner",
    "neighbors",
    "notes",
    "surveyor",
    "link"
  ],
  header: {
    identifier: "Identifier",
    city_town: "City/Town",
    streets: "Streets",
    survey_title: "Title",
    date: "Date",
    auction_date: "Date of Auction",
    property_owner: "Property Owner",
    neighbors: "Neighbors",
    notes: "Notes",
    surveyor: "Surveyor",
    link: "Link"
  },
  width: {
    // streets: 50
  }
}))
```

```js
let map = L.map("map").setView([42.3, -71.1], 13);
let osmLayer = L.tileLayer(
  "https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}@2x.png",
  {
    attribution:
      '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
  }
).addTo(map);
```

```js
const markers = L.markerClusterGroup().addTo(map);
let points = selected.map((d) => {
  if (testCoord(d.latitude) && testCoord(d.longitude)) {
    L.marker([+d.latitude, +d.longitude])
      .bindPopup(popupParser(d))
      .addTo(markers);
  }
})
```

```js
if (points.length > 0) {
  map.fitBounds(markers.getBounds(), { animate: true, duration: 1.75 });
}
```

<!-- functions and data -->

```js
const testCoord = (c) => {
  if (+c >= -180 && +c <= 180 && c != 0) {
    return true;
  }
}
```

```js
const aeonButton = (d) => {
  let link = `https://readingroom.bpl.org/aeon/aeon.dll?Action=10&Form=20&Value=GenericRequestMaps&ItemTitle=${encodeURIComponent(
    d["survey_title"]
  )}&ItemAuthor=${encodeURIComponent(
    d["surveyor"]
  )}&ItemDate=${encodeURIComponent(d["date"])}&CallNumber=${encodeURIComponent(
    d["identifier"]
  )}&ItemCitation=${encodeURIComponent("Surveyor Collections Search Tool")}`;
  return `<a href=${link} target="_blank">Request this item</a>`;
}
```

```js
const popupParser = (d) => `
  <div style="font-size:14px; line-height:1.4;">
    <h2 style="margin:0; font-size:1.2em;">${d.identifier || "No data"}</h2><br>
    <p style="margin:0;"><strong>Date:</strong> ${
      d.date ? d.date : "No data"
    }</p>
    <p style="margin:0;"><strong>Property Owner:</strong> ${
      d.property_owner ? d.property_owner : "No data"
    }</p>
    <p style="margin:0;"><strong>Survey Title:</strong> ${
      d.survey_title ? d.survey_title : "No data"
    }</p>
    <p style="margin:0;"><strong>Streets:</strong> ${
      d.streets ? d.streets : "No data"
    }</p>
    <p style="margin:0;"><strong>Neighbor:</strong> ${
      d.neighbor ? d.neighbor : "No data"
    }</p>
    <div style="margin-top:0.5em;">${aeonButton(d)}</div>
    <div style="margin-top:0.5em; ${
      d.link ? "visibility:visible" : "visibility:hidden"
    }"><a target="_blank" href=${d.link}>View digitized map</a></div>
  </div>
`
```

<style>

#table {
  overflow:true;
}

</style>