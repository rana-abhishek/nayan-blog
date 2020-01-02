---
title: Angular Charts
date: 2020-01-02 16:27:55
author: Abhishek Rana
category: Web
tags:
- Line Chart
- Javascript
- Data Visualization
---

<br>

![](BG.png)

Chart.js is a popular JavaScript charting library and ng2-charts is a wrapper for Angular 2+ that makes it easy to integrate Chart.js in Angular. Let’s go over the basic usage.

## Installation

1. Install ng2-charts using npm: `npm install --save ng2-charts`
2. Install Chart.js library: `npm install --save chart.js`
3. *[Options]* Then, if you’re using the **Angular CLI**, you can simply add Chart.js to the list of scripts in your `.angular-cli.json` file so that it gets bundled with the app: 

**_angular-cli.json_**
```
"scripts": [
  "../node_modules/chart.js/dist/Chart.min.js"
],
```

## API

Now you’ll want to import ng2-chart’s `ChartsModule` into your app module or a feature module:

**_app.module.ts_**
```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { ChartsModule } from 'ng2-charts';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    ChartsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

## Usage

ng2-charts gives us a `baseChart` directive that can be applied on an HTML `canvas` element. Here’s an example showing-off some of the options to pass-in as inputs and the `chartClick` event that’s outputted by the directive:

**_app.component.html_**
```
<div style="width: 40%;">
  <canvas
      baseChart
      [chartType]="'line'"
      [datasets]="chartData"
      [labels]="chartLabels"
      [options]="chartOptions"
      [legend]="true"
      (chartClick)="onChartClick($event)">
  </canvas>
</div>
```

And here’s what it can look like in our component class:

**_app.component.ts_**
```
import { Component } from '@angular/core';

@Component({ ... })
export class AppComponent {
  chartOptions = {
    responsive: true
  };

  chartData = [
    { data: [330, 600, 260, 700], label: 'Account A' },
    { data: [120, 455, 100, 340], label: 'Account B' },
    { data: [45, 67, 800, 500], label: 'Account C' }
  ];

  chartLabels = ['January', 'February', 'Mars', 'April'];

  onChartClick(event) {
    console.log(event);
  }
}
```

![Chart 1: Basic Line Chart](chart_1.png)


## Options

Here's a quick breakdown of the different input options

* chartType: This sets the base type of the chart. The value can be `pie`, `doughnut`, `bar`, `line`, `polarArea`, `radar` or `horizontalBar`.
* legend: A boolean for whether or not a legend should be displayed above the chart.
* datasets: This should be an array of objects that contain a data array and a label for each data set.
* data: If your chart is simple and has only one data set, you can use `data` instead of `datasets` and pass-in an array of data points.
* labels: An array of labels for the X-axis.
* options: An object that contains options for the chart. You can refer to the official [`Chart.js documentation`](https://www.chartjs.org/docs/latest/configuration/) for details on the available options.

In the above example we set the chart to be responsive and adapt depending on the viewport size.
* colors: Not shown in the above example, but you can define your own colors with the `colors` input. Pass-in an array of object literals that contain the following value:
```
myColors = [
  {
    backgroundColor: 'rgba(103, 58, 183, .1)',
    borderColor: 'rgb(103, 58, 183)',
    pointBackgroundColor: 'rgb(103, 58, 183)',
    pointBorderColor: '#fff',
    pointHoverBackgroundColor: '#fff',
    pointHoverBorderColor: 'rgba(103, 58, 183, .8)'
  },
  ... other colors
];
```

## Events
Two events are emitted, `chartClick` and `chartHover`, and they allow to react to the user interacting with the chart. The currently active points and labels are returned as part of the emitted event’s data.

* chartClick: fires when click on a chart has occurred, returns information regarding active points and labels
* chartHover: fires when mousemove (hover) on a chart has occurred, returns information regarding active points and labels
* Updating Datasets Dynamically: Of course, the beauty of Chart.js is that your charts can easily by dynamic and update/respond to data received from a backend or from user input.

In the bellow example we add a new data points for the month of May:

**_app.component.ts_**
```
newDataPoint(dataArr = [100, 100, 100], label) {

  this.chartData.forEach((dataset, index) => {
    this.chartData[index] = Object.assign({}, this.chartData[index], {
      data: [...this.chartData[index].data, dataArr[index]]
    });
  });

  this.chartLabels = [...this.chartLabels, label];

}
```
And it can be used like this:

**_app.component.html_**
```
<button (click)="newDataPoint([900, 50, 300], 'May')">
  Add data point
</button>
```

## Schematics
There are schematics that may be used to generate chart components using Angular CLI. The components are defined in package ng2-charts-schematics.

Installation of Schematics Package
`npm instal --save-dev ng2-charts-schematics`

Example of Generating a Line Chart using Angular CLI
`ng generate ng20chart0schematics:line my-line-chart`

This calls angular's component schematics and then modifies the result, so all the options for the component schematic are also usable here. This schematics will also add the ChartsModule as an imported module in the main app module (or another module as specified in the --module command switch).