---
title: Detecting CSS animation support
slug: Web/CSS/CSS_Animations/Detecting_CSS_animation_support
tags:
  - Advanced
  - CSS
  - CSS Animations
  - Guide
  - JavaScript
  - Junk
  - Reference
---
{{CSSRef}}

CSS animations make it possible to do creative animations of content using nothing but CSS. However, there are likely to be times when this feature isn't available, and you may wish to handle that case by using JavaScript code to simulate a similar effect. This article, based on [this blog post](https://hacks.mozilla.org/2011/09/detecting-and-generating-css-animations-in-javascript/) by Chris Heilmann, demonstrates a technique for doing this.

## Testing for CSS animation support

This code will check to see if CSS animation support is available:

```js
var animation = false,
    animationstring = 'animation',
    keyframeprefix = '',
    domPrefixes = 'Webkit Moz O ms Khtml'.split(' '),
    pfx  = '',
    elem = document.createElement('div');

if( elem.style.animationName !== undefined ) { animation = true; }

if( animation === false ) {
  for( var i = 0; i < domPrefixes.length; i++ ) {
    if( elem.style[ domPrefixes[i] + 'AnimationName' ] !== undefined ) {
      pfx = domPrefixes[ i ];
      animationstring = pfx + 'Animation';
      keyframeprefix = '-' + pfx.toLowerCase() + '-';
      animation = true;
      break;
    }
  }
}
```

For starters we define a few variables. We assume that animation is not supported by setting `animation` to `false`. We set the `animationstring` to `animation` which is the property we want to set later on. We create an array of browser prefixes to loop over and we set `pfx` to an empty string.

Then we check if the CSS {{cssxref("animation-name")}} property on the style collection for the element specified by the variable `elem` is set. This means the browser supports CSS animation without any prefix, which, to date, none of them do.

If the browser does not support non-prefixed animation and `animation` is still `false`, we iterate over all the possible prefixes, since all the major browsers are currently prefixing this property and changing its name to `AnimationName` instead.

Once this code is finished running, the value of animation will be `false` if CSS animation support isn't available, or it will be `true`. If it is `true` then both the animation property name and the keyframe prefix will be the right ones. So if you use a new Firefox, the property will be `MozAnimation` and the keyframe prefix `-moz-` and with Chrome it'll be `WebkitAnimation` and `-webkit-`. Notice browsers don't make it easy with the switching between camelCase and hyphen-ation.

## Animating using the correct syntax for different browsers

Now that you know if CSS animation is supported or not, we can animate.

```js
if( animation === false ) {

  // animate in JavaScript fallback

} else {
  elem.style[ animationstring ] = 'rotate 1s linear infinite';

  var keyframes = '@' + keyframeprefix + 'keyframes rotate { '+
                    'from {' + keyframeprefix + 'transform:rotate( 0deg ) }'+
                    'to {' + keyframeprefix + 'transform:rotate( 360deg ) }'+
                  '}';

  if( document.styleSheets && document.styleSheets.length ) {

      document.styleSheets[0].insertRule( keyframes, 0 );

  } else {

    var s = document.createElement( 'style' );
    s.innerHTML = keyframes;
    document.getElementsByTagName( 'head' )[ 0 ].appendChild( s );

  }

}
```

This code looks at the value of `animation`; if it's `false`, we know we need to use our JavaScript fallback code to perform our animation. Otherwise, we can use JavaScript to create the desired CSS animation effects.

Setting the animation property is easy; update its value in the style collection. However, adding keyframes is trickier, since they're not defined using traditional CSS syntax (which makes them more flexible, but harder to define from script).

To define our keyframes using JavaScript, we need to write them out as a CSS string. All we do is set up a `keyframes` variable, prefixing each attribute as it's constructed. This variable, once constructed, contains the complete description of all the keyframes needed by our animation sequence.

The next task is to actually add the keyframes to the page's CSS. The first thing to do is look to see if there's already a style sheet on the document; if so, we insert the keyframe description into that stylesheet; this is done in lines 13-15.

If there isn't already a style sheet, a new {{HTMLElement("style")}} element is created, and its content is set to the value of keyframes. Then the new {{HTMLElement("style")}} element is inserted into the document's {{HTMLElement("head")}}, thereby adding the new style sheet to the document.

[View on JSFiddle](https://jsfiddle.net/codepo8/ATS2S/8/embedded/result)

## See also

- [CSS animations](/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations)
