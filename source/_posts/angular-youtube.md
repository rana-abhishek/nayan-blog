---
title: Youtube Data API in Angular app for realtime chanels video data
date: 2020-06-21 16:36:18
author: Abhishek Rana
category: Web
tags:
  - angular
  - youtube
  - Youtube data API
  - Nayan
  - Cloud Data
---

<br>

![Youtube data API in Angular app](/blog/Web/angular-youtube/blog_banner.png)

Facing issues getting data of your Youtube channel on your website? This blog is a guide demonstrating on how to integrate Youtube data API in your angular web application with few easy steps.

## YouTube Data API v3

The first step is to get the api key. By visiting https://developers.google.com/youtube/v3/getting-started you’ll find the procedures you need to get your authorization credentials.
In a nutshell, you need:

1. Go to the Google Developers Console.
2. Select a project.
3. In the left sidebar, select APIs and authorization. In the list of APIs, make sure the status is ON for the YouTube Data API v3.

After the project is created the next step is to register the HttpClientModule module in the main module (app.module.ts).

```
import { HttpClientModule } from '@angular/common/http';
```

And declare in imports:

```
imports: [
BrowserModule,
AppRoutingModule,
HttpClientModule,
NgxSpinnerModule
```

I’ll use the NGX-Spinner library to display a spinner while loading the videos.

Install the library with:

```
$ npm install ngx-spinner --save
```

And declare the module in the imports, as per the code above.
We can now create a service to make calls to the Youtube API.

In the terminal, write:

```
$ ng g service youtube
```

```
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class YoutubeService {

  apiKey : string = 'YOUR-APIKEY-YOUTUBE';

  constructor(public http: HttpClient) { }

    getVideosForChanel(channel, maxResults): Observable<Object> {
    let url = 'https://www.googleapis.com/youtube/v3/search?key=' + this.apiKey + '&channelId=' + channel + '&order=date&part=snippet &type=video,id&maxResults=' + maxResults
    return this.http.get(url)
      .pipe(map((res) => {
        return res;
      }))
  }
}
```

We create an apiKey variable that stores the value of the API obtained in the first step.

Then we inject the HttpClient class into the constructor. It provides methods for performing HTTP requests.

Let’s implement a method that returns a list of videos. We name it getVideosForChanel(). We pass two arguments, the first is the channel ID. The second limit the number of videos.

We concatenate this information in the API URL, passing other parameters as the order (‘& order = date), part = snippet that contains other properties that identify the title, the description, among others, and the type of resource (type = video).

```
let url = ‘https://www.googleapis.com/youtube/v3/search?key=' + this.apiKey + ‘&channelId=’ + channel + ‘&order=date&part=snippet &type=video,id&maxResults=’ + maxResults
```

In the component class (app.component.ts), we declare an array for the result of the videos:

```
export class AppComponent {
videos: any[];
```

In the constructor method, we inject the service created for requesting videos (YoutubeService) and a class to display a spinner (NgxSpinnerService).

```
constructor(private spinner: NgxSpinnerService, private youTubeService: YoutubeService) { }
```

Then, in the ngOnInit( ) method, we invoke the method by passing the Channel ID, in this example the channel is my child’s :), and a maximum number of .getVideosForChanel results.

```
ngOnInit() {
this.spinner.show()
setTimeout(()=>
{
this.spinner.hide()
},3000)
this.videos = [];
this.youTubeService
.getVideosForChanel('UC_LtA_EtCr7Jp5ofOsYt18g', 15)
.pipe(takeUntil(this.unsubscribe$))
.subscribe(lista => {
for (let element of lista["items"]) {
this.videos.push(element)
}
});
}
```

In the result, .subscribe (list => {, retrieve the items property and add each object in the created array.

At the beginning of the function, we included a timeout of 3 seconds to close the spinner in.

```
setTimeout(()=>
{
this.spinner.hide()
},3000)
```

Let’s finalize, coding the component template (app.component.html:

```
<div *ngFor="let video of videos" class="col-xl-3 col-md-6 mb-4">
<div class="card border-0 shadow vh-50">
<a href="https://www.youtube.com/watch?v={{video.id.videoId}}" target="_blank">
<img [src]="video.snippet.thumbnails.medium.url" class="card-img-top" alt="..."></a>
<div class="card-body text-center">
<h5 class="card-title mb-0">
<a href="https://www.youtube.com/watch?v={{video.id.videoId}}">{{video.snippet.title}}
</a></h5>
<div class="card-text text-black-50">{{video.snippet.description.slice(0, 100)}}</div>
<p class="card-text">{{video.snippet.description}}</p>
</div>
</div>
</div>
```

We loop the array using the \* ngFor directive. We have defined a link to view the video through videoID at href = “https://www.youtube.com/watch?v={{video.id.videoId}}".

Resources:

1. https://developers.google.com/youtube/v3
2. https://developers.google.com/youtube/v3/docs/videos/list

Previous blog: https://nayan.co/blog/Web/angular-maps/
