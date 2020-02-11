---
title: Angular Charts Features
date: 2020-02-11 11:33:31
author: Abhishek Rana
category: Web
tags:
- Charts
- Plugins
- Customization
---

<br>

{% asset_img BG.png l %}

## Animation Configuration

Chart.js animates charts out of the box. A number of options are provided to configure how the animation looks and how long it takes.

The following animation options are available. The global options for are defined in Chart.defaults.global.animation.

  1. **Duration**: Number of milliseconds an animation takes to complete

  2. **Easing**: 
    * `linear`
    * `easeInQuad`
    * `easeOutQuad`
    * `easeInOutQuad`
    * `easeInCubic`
    * `easeOutCubic`
    * `easeInOutCubic`
    * `easeInQuart`
    * `easeOutQuart`
    * `easeInOutQuart`
    * `easeInQuint`
    * `easeOutQ`

  3. **Animation Callbacks**

  The onProgress and onComplete callbacks are useful for synchronizing an external draw to the chart animation. The callback is passed a Chart.Animation instance:


  ```
  {
    // Chart object
    chart: Chart,

    // Current Animation frame number
    currentStep: number,

    // Number of animation frames
    numSteps: number,

    // Animation easing to use
    easing: string,

    // Function that renders the chart
    render: function,

    // User callback
    onAnimationProgress: function,

    // User callback
    onAnimationComplete: function
  }
  ```

### Example of preogress bar animation

```
var chart = new Chart(ctx, {
    type: 'line',
    data: data,
    options: {
        animation: {
            onProgress: function(animation) {
                progress.value = animation.animationObject.currentStep / animation.animationObject.numSteps;
            }
        }
    }
});
```


## Legend Configuration

The chart legend displays data about the datasets that are appearing on the chart.

  1. **Position**
    Position of the legend. Options are:
    * `top`
    * `left`
    * `bottom`
    * `right`

  2. **Align**
    Alignment of the legend. Options are:
    * `start`
    * `center`
    * `end`

  3. **Legend Item Interface**

  Items passed to the legend onClick function are the ones returned from labels.generateLabels. These items must implement the following interface.

  ```
  {
    // Label that will be displayed
    text: string,

    // Fill style of the legend box
    fillStyle: Color,

    // If true, this item represents a hidden dataset. Label will be rendered with a strike-through effect
    hidden: boolean,

    // For box border. See https://developer.mozilla.org/en/docs/Web/API/CanvasRenderingContext2D/lineCap
    lineCap: string,

    // For box border. See https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/setLineDash
    lineDash: number[],

    // For box border. See https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/lineDashOffset
    lineDashOffset: number,

    // For box border. See https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/lineJoin
    lineJoin: string,

    // Width of box border
    lineWidth: number,

    // Stroke style of the legend box
    strokeStyle: Color,

    // Point style of the legend box (only used if usePointStyle is true)
    pointStyle: string | Image,

    // Rotation of the point in degrees (only used if usePointStyle is true)
    rotation: number
  }
  ```

### Example

The following example will create a chart with the legend enabled and turn all of the text red in color.


```
var chart = new Chart(ctx, {
    type: 'bar',
    data: data,
    options: {
        legend: {
            display: true,
            labels: {
                fontColor: 'rgb(255, 99, 132)'
            }
        }
    }
});

```


## Tooltip

### Tooltip Configuration
  1. **Position Modes**
  Possible modes are:

   * `average`
   * `nearest`

  2. **Alignment**
  The titleAlign, bodyAlign and footerAlign options define the horizontal position of the text lines with respect to the tooltip box. The following values are supported.

    * `left`
    * `right`
    * `center`

### Tooltip Callbacks

  1. **Label Callback**

  The label callback can change the text that displays for a given data point. A common example to round data values; the following example rounds the data to two decimal places.

  ```
  var chart = new Chart(ctx, {
    type: 'line',
    data: data,
    options: {
        tooltips: {
            callbacks: {
                label: function(tooltipItem, data) {
                    var label = data.datasets[tooltipItem.datasetIndex].label || '';

                    if (label) {
                        label += ': ';
                    }
                    label += Math.round(tooltipItem.yLabel * 100) / 100;
                    return label;
                }
            }
        }
    }
  });
  ```

  2. **Label Color Callback**

  For example, to return a red box for each item in the tooltip you could do:

  ```
  var chart = new Chart(ctx, {
    type: 'line',
    data: data,
    options: {
        tooltips: {
            callbacks: {
                labelColor: function(tooltipItem, chart) {
                    return {
                        borderColor: 'rgb(255, 0, 0)',
                        backgroundColor: 'rgb(255, 0, 0)'
                    };
                },
                labelTextColor: function(tooltipItem, chart) {
                    return '#543453';
                }
            }
        }
    }
  });
  ```

  3. **Tooltip Item Interface**

  The tooltip items passed to the tooltip callbacks implement the following interface.

  ```
  {
    // Label for the tooltip
    label: string,

    // Value for the tooltip
    value: string,

    // X Value of the tooltip
    // (deprecated) use `value` or `label` instead
    xLabel: number | string,

    // Y value of the tooltip
    // (deprecated) use `value` or `label` instead
    yLabel: number | string,

    // Index of the dataset the item comes from
    datasetIndex: number,

    // Index of this data item in the dataset
    index: number,

    // X position of matching point
    x: number,

    // Y position of matching point
    y: number
  }
  ```


## References
* https://jtblin.github.io/angular-chart.js/
* https://github.com/jtblin/angular-chart.js/blob/master/README.md
* https://valor-software.com/ng2-charts/








