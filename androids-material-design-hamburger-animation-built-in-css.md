After a recent update to Google's Play Store I spotted this slick animation when opening the sidebar.

{<1>}![native play store hamburger animation](https://i.imgur.com/r6HNziW.gif)
<sub>*Image sourced from [Android Police](http://www.androidpolice.com/).*</sub>

I figured I'd try and replicate it in CSS as a fun experiment.

I began by checking out some existing examples and implementations of animated hamburgers, some of the most useful resources I came across were:

* [Elijah Manor's CSS Animated Hamburger Icon
Problem](http://www.elijahmanor.com/css-animated-hamburger-icon/)
* [Sara Soueidan & Bennett Feely's Navicon Transformicons: Animated Navigation Icons with CSS Transforms](http://sarasoueidan.com/blog/navicon-transformicons/)
* [Chris Coyier's Three Line Menu Navicon](http://css-tricks.com/three-line-menu-navicon/)

I created a simple hamburger icon using the `before` and `after` [pseudo elements](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-elements).

```language-html
<section>
  <button class="material-design-hamburger__icon">
    <span class="material-design-hamburger__layer"></span>
  </button>
</section>
```

```language-css
button {
  border: none;
  background: none;
}
.material-design-hamburger__icon {
  padding: 2rem 1rem;
}
.material-design-hamburger__layer {
  display: block;
  width: 100px;
  height: 10px;
  background: #000;
  position: relative;
}
.material-design-hamburger__layer:before, .material-design-hamburger__layer:after {
  width: inherit;
  height: 10px;
  position: absolute;
  background: inherit;
  left: 0;
  content: '';
}
.material-design-hamburger__layer:before {
  bottom: 200%;
}
.material-design-hamburger__layer:after {
  top: 200%;
}
```
<sub>A simple hamburger.</sub>

I applied a basic CSS transition to rotate the elements. By leveraging the [Element.classList API](https://developer.mozilla.org/en-US/docs/Web/API/Element.classList) I attached an event listener to the main hamburger element and was thus able to toggle, apply and remove classes to trigger the transitions.

```language-javascript
document.querySelector('.material-design-hamburger__icon').addEventListener('click', function() {
  this.classList.toggle('material-design-hamburger__icon--to-arrow');
});
```
<sub>An example of toggling a class via an event listener.</sub>

I soon realised, however, that CSS transitions didn't quite give me the control I'd like. They were perfect for transitioning to the arrow icon but only allowed a basic reversal back to the hamburger upon the next click. As you can see above the animation initially rotates 180 degrees clockwise followed by another 180 degrees clockwise rotation on the next click, although I'm sure there's a way I struggled to achieve this with transitions.

On to CSS animations it was then. The centre line in the hamburger was simple, it's just a 180 degree rotation. The top and bottom lines were more tricky as they rotate to create the arrow and their length shortens during transition. After some trial and error I was able to find the perfect keyframes.

```language-css
@keyframes material-design-hamburger__icon--slide-before {
  0% {
  }
  100% {
    transform: rotate(45deg);
    margin: 3% 37%;
    width: 75%;
  }
}
@keyframes material-design-hamburger__icon--slide-after {
  0% {
  }
  100% {
    transform: rotate(-45deg);
    margin: 3% 37%;
    width: 75%;
  }
}
```
<sub>Keyframes for the top and bottom hamburger lines to form the arrow on click.</sub>

The animation back to the hamburger icon involved more of the same but reversing various values to revert back to the original view.

I finished up by creating a menu which slid into the viewport on click, adding some [Material Design colours](http://www.google.co.uk/design/spec/style/color.html) and using the Android font [Roboto](http://www.google.co.uk/design/spec/style/typography.html).

You can see the final product below.

<p data-height="368" data-theme-id="0" data-slug-hash="cFtzb" data-default-tab="result" data-user="swirlycheetah" class='codepen'>See the Pen <a href='http://codepen.io/swirlycheetah/pen/cFtzb/'>Material Design Hamburger</a> by Chris Wheatley (<a href='http://codepen.io/swirlycheetah'>@swirlycheetah</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

You can download the code from [GitHub](https://github.com/swirlycheetah/material-design-hamburger) or install via [npm](https://github.com/swirlycheetah/material-design-hamburger#npm) or [Bower](https://github.com/swirlycheetah/material-design-hamburger#bower) to integrate it on your own website quickly and easily. You'll begin with a blank template, like below, which can be styled to your liking.

{<2>}![css js hamburger animation](https://i.imgur.com/B0PT1Lb.gif)

Currently it's only compatiable with modern browsers (IE10+) due to Element.classList being a relatively new API.  Older browser support is imminent, via a shim.

[Download now](https://github.com/swirlycheetah/material-design-hamburger/releases/latest).

Got an improvement suggestions? Having trouble integrating with your website? Let me know below and I'll be sure to get back to you. 
