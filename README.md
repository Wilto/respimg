# Adaptive Image Element

### Author:
[Mat Marquis](mailto:mat@filamentgroup.com)

### Status of this Document
This is an unofficial draft spec, not formally endorsed by the WHATWG. It is suitable only for reviewing the details of the proposed element.

## Table of Contents

1. Introduction
2. Implementation Examples
  1. Sample Markup Pattern
	2. Functional Polyfill
3. Example Use Cases
	1. Flexible Layouts
	2. High-Resolution Displays
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

### 2.2. Functional Polyfill

[Scott Jehl](https://github.com/scottjehl) has put together a <a href="https://github.com/scottjehl/picturefill">JavaScript polyfill</a> that could be used to bring similar behavior to older browsers should `<picture>` see widespread adoption. As the polyfill is fully dependent on JavaScript, it differs from the behavior of a native implementation in that fallback content is also displayed in the event that JavaScript is unavailable. This ensures a predictable fallback regardless of the presence of JavaScript in older browsers, though a native implementation would have no such dependency on scripting.

## 3. Example Use Cases

### 3.1. Flexible Layouts

Serving full-bleed images within a flexible layout or a layout dictated by media queries requires a source image with the largest necessary inherent size and scaling it down through CSS. On smaller displays such as phones and tablets—where bandwidth can be at a premium—this means an exceptionally wasteful request. 

While there are currently “responsive images” solutions that deliver smaller images by default and conditionally load a larger image above a certain window size, all of these involve a redundant request and rely fully on JavaScript. On large displays, an image’s `src` will be prefetched prior to any logic that swaps the image, in many modern browsers. This is further detailed [in this post](http://www.alistapart.com/articles/responsive-images-how-they-almost-worked-and-what-we-need/).

```
<picture alt="Image of a polar bear blinking during a snowstorm.">
	<!-- Matches by default: -->
	<source src="mobile.jpg" /> 
	<source src="medium.jpg" media="min-width: 600px" /> 	
	<source src="fullsize.jpg" media="min-width: 900px" />
	<img src="mobile.jpg" />
</picture>
```

Assuming a 960px wide window at the time the page is requested and the sample markup pattern in section 3.1, the UA should make a single request for “fullsize.jpg.” Any window/screen smaller than 600px is served “mobile.jpg”, which—as a completely alternate source—could be cropped as well as resized in order to preserve the focus of the image at smaller sizes.

### 3.2. High-Resolution Displays

High resolution screens such as Apple’s Retina display will require high-resolution images, leaving us with a situation similar to the above: either serving larger, high-resolution images to displays that can’t take advantage of them, or forcing high-density displays to first download a low-resolution image then replace it with a high-resolution image. The latter—the approach currently used on [Apple’s website](http://apple.com/)—is far from ideal, and the former is so fraught with concerns that it has recently been adressed in such mainstream publications as the [New York Times](http://bits.blogs.nytimes.com/2012/03/21/ipad-web-retina/).

```
<picture alt="Hero image for new high-resolution device, containing a cringe-worthy portmanteau.">
	<!-- Matches by default: -->
	<source src="standard-res.jpg" /> 	
	<source src="high-res.jpg" media="[-webkit-]min-device-pixel-ratio: 2" />
	<img src="standard-res.jpg" />
</picture>
```

In this instance, the standard resolution image is served by default and as a fallback in cases where `<picture>` is unsupported. The high resolution image is served instead only in cases where the pixel-ratio media query matches.

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
