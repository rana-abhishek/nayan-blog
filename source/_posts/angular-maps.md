---
title: Angular Maps | MarkerCluster
date: 2020-03-04 11:18:37
author: Abhishek Rana
category: Web
tags:
- Angular
- Google Maps
- Large markers
---

<br>

{% asset_img BG.png l %}

## Marker Cluster
The marker clustering utility helps you to manage multiple markers at different zoom levels.When a user views the map at a high zoom level, the individual markers show on the map. When the user zooms out, the markers gather together into clusters, to make viewing the map easier.
If you have a lot of markers on the map, it’s better to use Marker Cluster setting to organize them better visually.

## Why marker clustering?
The marker clustering utility helps you manage large number of google markers at different zoom levels. To be precise, the 'markers' are actually 'items' at this point, and only become 'Markers' when they're rendered. Rendering large number of google markers on google map can be very resouce extensive tasks and UI experince is also not good even if we achieve to render them. When a user views the map at a high zoom level, the individual markers show on the map. When the user zooms out, the markers gather together into clusters, to make viewing the map easier.

## How marker clustering works
The MarkerClustererPlus library uses the grid-based clustering technique that divides the map into squares of a certain size (the size changes at each zoom level), and groups the markers into each square grid. It creates a cluster at a particular marker, and adds markers that are in its bounds to the cluster. It repeats this process until all markers are allocated to the closest grid-based marker clusters based on the map's zoom level. If markers are in the bounds of more than one existing cluster, the Maps JavaScript API determines the marker's distance from each cluster, and adds it to the closest cluster.

## How to use Marker Cluster in Angular Apps

### Installation of modules
We need to install AGM (Angular Google Maps), js-marker-cluster(peer dependency)
  1. NPM: `npm install js-marker-clusterer @agm/js-marker-clusterer --save`
  2. Yarn: `yarn add js-marker-clusterer @agm/js-marker-clusterer`

**Module**: `https://www.npmjs.com/package/@agm/js-marker-clusterer`

### Usage
  1. Import the module in `module.ts` file of your anulgar application
    ```
    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';
    import { AppComponent } from './app.component';
    
    // add these imports
    import { AgmCoreModule } from '@agm/core';
    import { AgmJsMarkerClustererModule } from '@agm/js-marker-clusterer';
    
    @NgModule({
    declarations: [
        AppComponent
    ],
    imports: [
        BrowserModule,
        AgmCoreModule.forRoot({
        apiKey: ['YOUR_API_KEY_HERE']
        }),
        AgmJsMarkerClustererModule
    ],
    providers: [],
    bootstrap: [AppComponent]
    })
    export class AppModule { }
    ```
  2. Import the modules in angular component 
    ```
    import * as MarkerClusterer from "@google/markerclusterer"

    new MarkerClusterer(map, opt_markers, opt_options)
    ```
  3. Use in your angular component
    ```
    <agm-map style="height: 300px" [latitude]="51.673858" [longitude]="7.815982">
        <agm-marker-cluster imagePath="https://raw.githubusercontent.com/googlemaps/v3-utility-library/master/markerclustererplus/images/m">
            <agm-marker [latitude]="51.673858" [longitude]="7.815982">
            </agm-marker><!-- multiple markers -->
        </agm-marker-cluster>
    </agm-map>
    ```
### Customize your marker clusters
There are many ways to adjust how your marker clusters look and function. Many of them won’t even require that you make edits to the underlying library. Instead, there are a number of options you can set when you create your clusters.
  1. `gridSize`: the number of pixels within the cluster grid

  2. `zoomOnClick`: whether to zoom in on a cluster when clicked

  3. `maxZoom`: what farthest level you can zoom in before regular markers are always displayed

  4. `styles`: an array of objects for each cluster type that includes textColor, textSize, and other features of the cluster

  Example:
    ```
    const clusterOptions = {
        imagePath: "https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m",
        gridSize: 30,
        zoomOnClick: false,
        maxZoom: 10,
    }
    ```
  OutPut:
    {% asset_img custom.png l %}
