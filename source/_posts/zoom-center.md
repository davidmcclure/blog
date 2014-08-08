title: Artificially shifting the "zoom center" of a map
date: 2014-07-08
tags:
  - neatline
  - leaflet
  - openlayers
---

I've worked on a couple of projects recently that involved positioning some kind of container on top of a slippy map that blocks out a significant chunk of the screen. Usually, the motivation for this is aesthetic - it looks neat to bump down the opacity on the overlay container just a bit, allowing the tiles to slide around underneath the overlay content when the map is panned or zoomed:

[![](images/web/doi-border.jpg)]()
[![](images/web/gemini-border.jpg)]()

The problem with this, though, is that it creates a "functional" viewport that's different from the actual height and width of the map container, which is scaled to fill the window. This causes trouble as soon as you need to dynamically focus the map on a given location. For example, in the Declaration of Independence project, Neatline needs to frame the viewport around the annotated signatures on the document when the user clicks on one of the names in the transcription. By default, we get this:

![](images/web/overlap.jpg)

OpenLayers is faithfully centering the entire map container around the geometric centroid of the annotation - but, since we've covered up about a third of the screen with the text, a sizable chunk of the annotation is slipping under the text container, which looks bad. Also annoying is that the map will zoom around a point that's displaced away from the middle of the visible region. If you want to zoom in on Boston, and put Boston in what appears to be the center of the map, Boston will actually scoot off to the right when you click the "+" button, since the map will zoom in around the "true" centroid in the middle of the window. This cramps your style when you want to zoom in a couple of times, and have to manually drag the map to the right after each step to keep the thing you care about inside the viewport.

Really, what we want is to shift the working center point of the map to the right, so that it sits in the middle of the un-occluded region on the map:

![](images/web/centered.jpg)

Mapping libraries generally don't have any native capacity to do this, so I had to MacGyver up something custom. After tinkering around, I realized that there's a really simple and effective solution to this problem - just stretch the map container past the right edge of the window until the center sits in the correct location. For simplicity, imagine the window is 1200px wide, and the overlay is 400px, a third of the width. We want the functional center point of the map to sit 400px to the right of the overlay, or 800px from the left side of the window. In order for the center to sit at 800px, the map has to be twice that, 1600px:

![](images/web/diagram.jpg)

Conveniently, this is always equal to the _width of the window plus the width of the overlay_. So, the positioning logic is only slightly more complicated:

```javascript
var overlay = $('#overlay');
var map = $('#map');

// Cache the right border of the overlay.
var overlayWidth = overlay.outerWidth() + overlay.offset().left;

var position = function() {

  // Map width = window width + overlay width.
  map.outerWidth($(window).width() + overlayWidth);

  // Fill the window height with the content.
  map.add(overlay).outerHeight($(window).height());

};

$(window).resize(position);
position();
```

The one downside to this is that it adds a bit of wasted load to the tile server(s) that are feeding imagery into the map - OpenLayers has no way of knowing that the rightmost 400px of the map is invisible to the user, and will load tiles to fill the space. But, life is short, servers are cheap.
