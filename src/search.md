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
```

```js
const masonData = d3.csv(
  "https://raw.githubusercontent.com/bplmaps/lmec-digital-library-metadata/main/survey-data/mason.csv"
)
```

```js
const bellamyData = d3.csv(
  "https://raw.githubusercontent.com/bplmaps/lmec-digital-library-metadata/main/survey-data/Bellamy.csv"
)
```

```js
const auctionData = d3.csv(
  "https://raw.githubusercontent.com/bplmaps/lmec-digital-library-metadata/refs/heads/main/survey-data/auction-lot.csv"
)
```

```js
const data = d3.merge([masonData, bellamyData, auctionData])
```

## 🔎 Enter a search term

```js
import _ from "lodash"
// import markerClusterGroup from "leaflet.markercluster"
```

```js
const search = view(Inputs.search(data, {
  placeholder: "Search a city, street, name, etc."
}));
```

```js
const table = view(Inputs.table(search, {
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
    "background_notes",
    "surveyor",
    "related_registry_doc",
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
    background_notes: "Background Notes",
    surveyor: "Surveyor",
    related_registry_doc: "Related registry documents",
    link: "Link"
  },
  width: {
    streets: 50
  }
}))
```

<div class="grid grid-cols-1" style="grid-auto-rows: 400px;" width="640">
  <div class="card" id="map" style="padding: 0">
  </div>
</div>

```js
const map = L.map("map").setView([42.3, -71.1], 13);
```

```js
let osmLayer = L.tileLayer(
  "https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}@2x.png",
  {
    attribution:
      '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
  }
).addTo(map);

const markers = L.markerClusterGroup().addTo(map);

let points = table.map((d) => {
  if (testCoord(d.latitude) && testCoord(d.longitude)) {
    L.marker([+d.latitude, +d.longitude])
      .bindPopup(popupParser(d))
      .addTo(markers);
  }
});

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
  <div style="font-size:18px; line-height:1.4;">
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

<!-- A shared color scale for consistency, sorted by the number of launches -->

<!-- ```js
const color = Plot.scale({
  color: {
    type: "categorical",
    domain: d3.groupSort(launches, (D) => -D.length, (d) => d.state).filter((d) => d !== "Other"),
    unknown: "var(--theme-foreground-muted)"
  }
});
``` -->

<!-- Cards with big numbers -->

<!-- <div class="grid grid-cols-4">
  <div class="card">
    <h2>United States 🇺🇸</h2>
    <span class="big">${launches.filter((d) => d.stateId === "US").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Russia 🇷🇺 <span class="muted">/ Soviet Union</span></h2>
    <span class="big">${launches.filter((d) => d.stateId === "SU" || d.stateId === "RU").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>China 🇨🇳</h2>
    <span class="big">${launches.filter((d) => d.stateId === "CN").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Other</h2>
    <span class="big">${launches.filter((d) => d.stateId !== "US" && d.stateId !== "SU" && d.stateId !== "RU" && d.stateId !== "CN").length.toLocaleString("en-US")}</span>
  </div>
</div> -->

<!-- Plot of launch history -->

<!-- ```js
function launchTimeline(data, {width} = {}) {
  return Plot.plot({
    title: "Launches over the years",
    width,
    height: 300,
    y: {grid: true, label: "Launches"},
    color: {...color, legend: true},
    marks: [
      Plot.rectY(data, Plot.binX({y: "count"}, {x: "date", fill: "state", interval: "year", tip: true})),
      Plot.ruleY([0])
    ]
  });
}
``` -->

<!-- <div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => launchTimeline(launches, {width}))}
  </div>
</div> -->

<!-- Plot of launch vehicles -->

<!-- ```js
function vehicleChart(data, {width}) {
  return Plot.plot({
    title: "Popular launch vehicles",
    width,
    height: 300,
    marginTop: 0,
    marginLeft: 50,
    x: {grid: true, label: "Launches"},
    y: {label: null},
    color: {...color, legend: true},
    marks: [
      Plot.rectX(data, Plot.groupY({x: "count"}, {y: "family", fill: "state", tip: true, sort: {y: "-x"}})),
      Plot.ruleX([0])
    ]
  });
}
``` -->

<!-- <div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => vehicleChart(launches, {width}))}
  </div>
</div> -->

<!-- Data: Jonathan C. McDowell, [General Catalog of Artificial Space Objects](https://planet4589.org/space/gcat) -->
