# Adaptive Image Element

### Author:
[Mat Marquis](mailto:mat@filamentgroup.com)

### Status of this Document
This is an unofficial draft spec, not formally endorsed by the WHATWG. It is suitable only for reviewing the details of the proposed element.

## Table of Contents

1. Introduction
2. Implementation Examples
  1. Sample Markup Pattern
	2. Functional Polyfills
3. Example Use Cases
	1. Mobile-first Development
	2. Desktop-first Development
	3. Relative Units
	4. High-Resolution Displays
	5. DOM Scripting and Bandwidth
4. Requirements
5. Prior Discussion

## 1. Introduction

Our goal is a markup-based means of delivering alternate image sources based on device capabilities, to prevent wasted bandwidth and optimize display for both screen and print.

The idea is to use the video tag’s markup pattern as the inspiration, as it’s specced to allow the use of media queries within attributes on its source elements and reliably displays the markup inside the tag in any browser that doesn’t recognize it. Through use of media attributes we would not only be able to reduce wasteful image requests for the sake of users with smaller displays, but we would be able tailor our images’ resolutions for users with high-res displays or for print. 

Much of the surrounding discussion has taken place publicly, in the W3C’s [Responsive Images Community Group](http://www.w3.org/community/respimg/).

## 2. Implementation Examples

Any combination of existing media queries can be used to determine the appropriate picture source, through a `media` attribute on `source` elements. This is identical to the specced behavior of `media` attributes on the `video`’s `source` elements, as [outlined here](http://dev.w3.org/html5/spec/the-source-element.html#attr-source-media). Any implementation of the `picture` tag should allow for the inclusion of fallback markup that is completely ignored by any UA that supports `picture`, and is only displayed in browsers that do not recognize the new tag. Note that older browsers can be polyfilled (see section 3.2) with behavior similar to a native implementation.

### 2.1. Sample Markup Pattern

```
<picture alt="The alt attribute’s content should accurately describe the image represented by all sources, though cropping and zooming of sources may differ.">
	<!-- Matches by default: -->
	<source src="mobile.jpg" /> 
	
	<!-- Overrides the previous source for windows greater than 600px -->
	<source src="medium.jpg" media="min-width: 600px" /> 
	
	<!-- Overrides the previous source for windows greater than 900px -->
	<source src="fullsize.jpg" media="min-width: 900px" /> 
	
	 <!-- Fallback content, only displayed in the event the <picture> tag is unsupported by the browser: --> 
	<img src="mobile.jpg" />
</picture>
```

### 2.2. Functional Polyfills

Currently two polyfills exist to bring the proposed functionality of the `picture` element to non-supporting browsers: 

* Scott Jehl’s [Picturefill](https://github.com/scottjehl/picturefill)
* Abban Dunne’s [jQuery Picture](http://jquerypicture.com)

A reliable native fallback is provided in the event that the new functionality isn’t supported and no polyfill is applied, using the same markup-based pattern as the `video` tag.

## 3. Example Use Cases

Serving full-bleed images within a flexible layout or a layout dictated by media queries requires a source image with the largest necessary inherent size and scaling it down through CSS. On smaller displays such as phones and tablets—where bandwidth can be at a premium—this means an exceptionally wasteful request. 

While there are currently “responsive images” solutions that deliver smaller images by default and conditionally load a larger image above a certain window size, all of these involve a redundant request and rely fully on JavaScript. On large displays, an image’s `src` will be prefetched prior to any logic that swaps the image in many modern browsers. This is further detailed [in this post](http://www.alistapart.com/articles/responsive-images-how-they-almost-worked-and-what-we-need/).

This limitation could be avoided in a native implementation.

### 3.1. Mobile-First Development

```
<picture alt="">
	<!-- Matches by default: -->
	<source src="mobile.jpg" /> 
	<source src="medium.jpg" media="min-width: 600px" /> 	
	<source src="fullsize.jpg" media="min-width: 900px" />
	<img src="mobile.jpg" />
</picture>
```

Assuming a 960px wide window at the time the page is requested and the sample markup pattern in section 3.1, the UA should make a single request for “fullsize.jpg.” Any window/screen smaller than 600px is served “mobile.jpg”, which—as a completely alternate source—could be cropped as well as resized in order to preserve the focus of the image at smaller sizes.

This example could be made more specific through the [`orientation`](http://www.w3.org/TR/css3-mediaqueries/#orientation), [`device-width`](http://www.w3.org/TR/css3-mediaqueries/#device-width), and [`device-height`](http://www.w3.org/TR/css3-mediaqueries/#device-height) media queries.

### 3.2. Desktop-First Development

Following the reverse of the logic above, authors may want to ensure that the largest possible images are served by default. For example: a site intended for browsing across a range of “smart televisions” where one assumes a large viewport, but assets could be optimized in the event that media queries are supported. This use case could be further tailored through the [`aspect-ratio`](http://www.w3.org/TR/css3-mediaqueries/#aspect-ratio) and [`device-aspect-ratio`](http://www.w3.org/TR/css3-mediaqueries/#device-aspect-ratio)` media queries.

```
<picture alt="">
	<!-- Matches by default: -->
	<source src="fullsize.jpg" /> 
	<source src="medium.jpg" media="max-width: 1400px" /> 	
	<source src="smaller.jpg" media="max-width: 1280px" />
	<img src="mobile.jpg" />
</picture>
```

### 3.3. Relative Units
A common practice in creating flexible layouts is to specify the value of one’s media queries in relative units: `em`, `rem`, `vw`/`vh` etc. This is most commonly seen with `em`s in order to reflow layouts based on users’ zoom preferences, or to resize elements through JavaScript by dynamically altering a `font-size` value.

Through the use of media queries this layout flexibility would extend to images as well, ensuring that a source always remains appropriate to the size of its containing element.

```
<picture alt="">
	<!-- Matches by default: -->
	<source src="mobile.jpg" /> 
	<source src="medium.jpg" media="min-width: 37.5em" /> 	
	<source src="fullsize.jpg" media="min-width: 56.25em" />
	<img src="mobile.jpg" />
</picture>
```

### 3.4. High-Resolution Displays

High resolution screens such as Apple’s Retina display will require high-resolution images, leaving us with a situation similar to the above: either serving larger, high-resolution images to displays that can’t take advantage of them, or forcing high-density displays to first download a low-resolution image then replace it with a high-resolution image. The latter—the approach currently used on [Apple’s website](http://apple.com/)—is far from ideal, and the former is so fraught with concerns that it has recently been adressed in such mainstream publications as the [New York Times](http://bits.blogs.nytimes.com/2012/03/21/ipad-web-retina/).

```
<picture alt="">
	<!-- Matches by default: -->
	<source src="standard-res.jpg" /> 	
	<source src="high-res.jpg" media="min-device-pixel-ratio: 1.5" />
	<source src="highest-res.jpg" media="min-device-pixel-ratio: 2.5" />
	<img src="standard-res.jpg" />
</picture>
```

In this instance, the standard resolution image is served by default and as a fallback in cases where `<picture>` is unsupported. The high resolution image is served instead only in cases where the pixel-ratio media query matches. Naturally, this can be used in combination with `min-width` and `max-width` media queries to ensure the most appropriate possible image source.

### 3.5. DOM Scripting and Bandwidth

In treating `sources` as independent elements, we make on-the-fly manipulation of elements via the DOM infinitely more simple compared to parsing and manipulating a string of sources contained in a single attribute. Without an implemented version of `picture` this remains purely speculation, but simplicity in manipulating these sources combined with the rapid adoption of `navigator.connection` may open the door for sources more tailored to a user’s internet connection.

Of course—given the flexibility and constant evolution of media queries—we may even be able to specify connection speed within our `media` attributes one day.

## 4. Requirements

**A conforming user agent must meet the following requirements:**

* The appropriate asset MUST be fetched by way of a single request. A change in window size causing the media attribute to match an alternate source SHOULD trigger a request for said source (to be retrieved from the browser cache, if possible).
* As with the `<video>` and `<audio>` tags, this solution MUST NOT require any client-side scripting, server-side technologies, or headers to reliably deliver content tailored for the end user’s context.
* Similar to the `<video>` tag, fallback markup MUST be rendered in any browser that does not recognize the `<picture>` element. The example in 3.1 uses the “mobile”-sized image as the fallback content, which is the recommended approach: barring the use of a polyfill, the smaller/low-res image should be provided as a fallback to prevent incurring a costly download in contexts that may see no benefit.
* The specification MUST provide at least the same level of accessibility as `<img>`, with an `alt` attribute readily accessible to assistive technology.	

## 5. Prior Discussion

How we arrived at `<picture>` most recently:
https://etherpad.mozilla.org/responsive-assets

Common questions and concerns:
http://www.w3.org/community/respimg/common-questions-and-concerns/

Prior discussion on W3 mailing lists:
http://www.w3.org/Search/Mail/Public/search?type-index=public-html&index-type=t&keywords=picture+element
http://lists.w3.org/Archives/Public/public-html/2007Jun/1057.html
http://lists.w3.org/Archives/Public/public-html/2011May/0386.html
http://lists.whatwg.org/pipermail/whatwg-whatwg.org/2012-February/ (as “Responsive Images”)
