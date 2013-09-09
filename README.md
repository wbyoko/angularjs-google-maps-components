## Making Google Maps Components with AngularJS

I have been hacking with AngularJS since the start of the year.  I really enjoy the framework, though outside of personal and side projects, I really hadn't found a good way to get it into the toolkit at work. I thought maps would be a good opportunity. I had already chipped in with two Google Maps pieces this year. Unfortunately though, I was only helping some people who were inexperienced with javascript, and wouldn't be maintaining those projects. I looked into using some of the open source google maps directives out there (great job on all of them), but at the end it was too much to hand off to someone who wasn't the strongest javascript developer, and ending up acquiescing to use jQuery and Handlebars. 

Finally, a project came around which I would be maintaining and felt this would be a great opportunity to see if AngularJS could fit. After getting to the point of just writing it without pre-made directives, I saw a really simple workflow for setting up and creating your map directive that can lead to a pretty immersive and distinguished map. 

## Link Function
The simplest situation is just setting up a map with a starting location. Using the directive link function instantiate the map seems to do a great job. 

```javascript
angular.module('mapComponentsApp', []).directive('helloMaps', function () {
  return function (scope, elem, attrs) {
    var mapOptions,
      latitude = attrs.latitude,
      longitude = attrs.longitude,
      map;

    latitude = latitude && parseFloat(latitude, 10) || 43.074688;
    longitude = longitude && parseFloat(longitude, 10) || -89.384294;

    mapOptions = {
      zoom: 8,
      center: new google.maps.LatLng(latitude, longitude)
    };

    map = new google.maps.Map(elem[0], mapOptions);
  };
});
```
You could use a directive controller, and would be straightforward using both. I created a [demo](http://wbyoko.co/angularjs-google-maps-components/demos/01-Hello-Maps.html)/[source file]
(https://github.com/wbyoko/angularjs-google-maps-components/blob/master/01-Hello-Maps.html) which shows a simple map example. 

## Custom Controls
Now, it wasn't until I had realized the ease with which I could add data-binded custom controls, that I knew that AngularJS could help making really ambitious maps applications not only possible, but very manageable. After tinkering with a few situations, I recognized that using the [$compile](http://docs.angularjs.org/api/ng.$compile) service to compile HTML into an AngularJS template and link it to the directive's scope fits perfectly. Ends up being three lines. 

```javascript
var controlTemplate = document.getElementById('helloControl').innerHTML.trim();
var controlElem = $compile(controlTemplate)(scope);
map.controls[google.maps.ControlPosition.TOP_LEFT].push(controlElem[0]);
```
I left the templates as strings as there are opportunites to use pre-angular templating systems, such as [John Resig's Micro-Templating Function](http://ejohn.org/blog/javascript-micro-templating/) to offload some of the logic that wouldn't need to be managed by AngularJS.

I created a [demo](http://wbyoko.co/angularjs-google-maps-components/demos/02-Say-Hello.html)/[source file](https://github.com/wbyoko/angularjs-google-maps-components/blob/master/02-Say-Hello.html) to show a data-binded custom controls in action.

## Binding to the Map
There end up being two parts to the binding. one will be reciveing data from map events, the other is making the map reacting to scope changes. 

I tackle this, a lat/long binding would be a good proof of concept and here it is [demo](http://wbyoko.co/angularjs-google-maps-components/demos/03-Say-Where.html)/[source file](https://github.com/wbyoko/angularjs-google-maps-components/blob/master/03-Say-Where.html). 

Responding to map event is done basically the same way Google Maps advices with the slight Angular twist of a ```scope.$apply``` to make sure the changes are reflected in Angular. 

```javascript
function centerChangedCallback (scope, map) {
  return function () {
    var center = map.getCenter();
    scope.latitude = center.lat();
    scope.longitude = center.lng();
    if(!scope.$$phase) scope.$apply();
  };
}
google.maps.event.addListener(map, 'center_changed', centerChangedCallback(scope, map));
```

To get the scope changes pushed back to the map we need to write a binding against the scope. It seemed to be the time to add a controller to the directive. I created a controller instance function ```this.registerMap = function (myMap) {...};``` to pass the map instance to the controller once it had been created. ```ctrl.registerMap(map);```. At this point, it's a straightforward ```$scope.$watch``` to get the scope data reflected back onto the map.

```javascript
$scope.$watch('latitude + longitude', function (newValue, oldValue) {
  if (newValue !== oldValue) { 
    var center = map.getCenter(),
      latitude = center.lat(),
      longitude = center.lng();
    if ($scope.latitude !== latitude || $scope.longitude !== longitude)
      map.setCenter(new google.maps.LatLng($scope.latitude, $scope.longitude));
  }
});
```

## Todo Maps
At this point there only one thing left to do.... todo maps [demo](http://wbyoko.co/angularjs-google-maps-components/demos/04-Todo-Maps.html)/[source file](https://github.com/wbyoko/angularjs-google-maps-components/blob/master/04-Todo-Maps.html). Even thought it end up being a fair bit of code the most important code is the link function of the todoMaps directive (at the bottom of source), which is it still digestable, and is the core of the entire application.

### Tools
The only tools I used for these examples were 
* [AngularJS](http://angularjs.org/)
* [Google Maps](https://developers.google.com/maps/documentation/javascript/tutorial)
* [YUI's Pure Css Framework](http://purecss.io/)

#### Summary
There were no other external dependencies for these samples nor do i believe any would they have made making these components any easier (angular has $http for ajax requests)

### I think this is big
This is an awesome use case that could have AngularJS showing up in a lot of places. As someone writes a map a least once or twice a year, I feel it's a much more pleasant workflow than using jQuery/Handlebars, Not to mention the easy assess to data-binded custom map controls and the savvy look and feature set your map could achieve pretty easily.

### Learn more about AngularJS
Hopefully, if you don't know AngularJS, you are now interested. A great list of [AngularJS resources](https://github.com/jmcunningham/AngularJS-Learning), started by @jmcunningham has tons of great info from articles to videos to books. 

Happy Coding, wbyoko
[Twitter](https://twitter.com/wbyoko) / 
[Github](https://github.com/wbyoko/)
