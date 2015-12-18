---
layout: post
title: "The Magic Panel - blogspot"
date: 2015-12-18 21:06:18 +0000
comments: true
categories: 
---

The Magic Panel

I'm almost done with the rails tutorial and think I have a plan for the app itself. I want to offer it to empeopled.com as a sort of self-contained friend-feed. I was struck by just how close the community was when I was helping a user tweak his Chrome extension for the site. I'll probably post about that when/if he ever gets it to github. He said he wants to work out the kinks before he feels it's ready for the public to use. He wants the features fleshed out.

With that said, I made a thing. It's still a bit clunky but it does what I think it is supposed to so let's call this an impromptu guide on said thing. So, I like Bootstraps responsive navbar, and I like making it a sidebar instead. I also like having a sticky side-panel and the news feed on the other side. In Bootstrap speak, you have your core profile info in a md-col-4 and the actual news feed in a md-col-8. The sidebar won't move but the feed will on mouse scroll. The catch is that you maintain Bootstrap's responsive behavior all the while. It got real tricky, real fast. Let's jump in, shall we?

The hard part first: I have all my Javascript in a file called extras.js so I can keep track of my own JS and keep it separate from the stuff required/generated for Rails. The full JS file is as follow:

```javascript
//this is stuff I added along the way while going through the tutorial

//needed to add the sidebar slide out on smaller screens
$(document).ready(function() {   
    var sideslider = $('[data-toggle=collapse-side]');
    var sel = sideslider.attr('data-target');
    var sel2 = sideslider.attr('data-target-2');
    sideslider.click(function(event){
        $(sel).toggleClass('in');
        $(sel2).toggleClass('out');
    });
});

//needed to get the dropdown working properly.  
$(document).ready(function() {
  $(".dropdown-toggle").dropdown();
});

//animated hamburger button - uses navbar-toggle so no need changing there.
$(document).ready(function () {
   $(".navbar-toggle").on("click", function () {
      $(this).toggleClass("active");
   });
});

//affix script. Might try to move it to extras.js later
//http://www.tutorialrepublic.com/twitter-bootstrap-tutorial/bootstrap-affix.php
$(document).ready(function () {
    $("#myNav").affix();
});

equipped with all my comments. I literally pulled it directly.  The part we will focus on is

$(document).ready(function () {
    $("#myNav").affix();
});
```

This uses some JQuery magic to affix whatever I use the #MyNav on it's div. It's similar to using static versus fixed in CSS. but it works with the responsive nature of the site.  So we tuck that into the extras.js file and move on to our HTML which will be the easier part:

```html
<div class="row">
  <aside class="col-md-4">
    <div data-spy="affix" data-offset-top="0"> 
    <!-- side panel goes here v-->
    </div>
  </aside>
  <div class="col-md-8">
    <!-- page content goes here -->
  </div>
</div>
```

Here we have the md-col-4 and md-col-8 that I mentioned earlier. Bootstrap uses these to segregate the content for your own amusement and manipulation. Our profile content goes in the -4 and our feed, in the -8. Once we have the Bootstrap logic set up, we can then add our bit of JQuery to the HTML with:

```html
<div data-spy="affix" data-offset-top="0"> 
```

 This added the affix function along with a data-offset function that we set to "0" to keep the div in the same place when we scroll. If you set it to another number it will be positioned differently. On some monitors/browsers, you may notice the div flicker on scroll and occasionally it will scroll and catch and with that said, this isn't perfect, but it does work.

The final bit of magic is using 2 Bootstrap media queries for the separate screen sizes. This bit was hard to get right and took an hour or two of tinkering and crashing my virtual server on Cloud 9 several times. When you spam edit/save the CSS it loads the RAM and then it takes a shit. It's a limit of having a server with only 512 MB of RAM for a dev environment I guess. Anyway, the CSS:

```css
.affix {
  width: 360px;
}

@media (max-width: 1200px) {
  .affix{ width: 293px; 
    position: fixed;
  }
  .microposts{ 
    width: 100%; 
    
  }
}

@media (max-width: 992px) {
  .affix{ width: 100%; 
    position: static;
  }
  .microposts{ 
    width: 100%; 
    
  }
}
```

We start by setting a fixed width for the .affix class because it will resize on scroll otherwise. I never figured out why it naturally wants to change the entire section's size but it does so we fix it to the same width of the &lt;aside&gt; and it stays put. We then add two important media queries that will keep things at a correct spot on different screen sizes. The goal was to have a side-panel for most/all computer screens but go into a single column on tablet and mobile screens.
The first query looks like I arbitrarily used 293px but I came up with that after a series of trials and errors of resizing the browser until it behaved properly. This is for smaller desktop/laptop screens. The second query is for going into a single column, indicated by width: 100%; for both the .affix and .micropost classes.

Now that we have all the pieces in place, you should have a functioning responsive-fixed-hybrid page along with all your other Bootstrappy magic. 

The only real caveat here is that if you are using Turbolinks/Jquery-Turbolinks, the scripts won't work unless you reload the entire page. As, the saying goes, that just isn't the Rails way. We have a full site built on _partials and all sorts of modular pieces and we haven't come this far to get hung up by a stupid page refresh. The onl solution I found to fix Javascript stopping on a link click was to remove the Turbolinks and JQuery-Turbolinks  gems altogether including deleting them from application.js. The other fixes I found didn't work for me. 

Anyway, this was my attempt at a guide. Sorry about any confusion ahead of time. Happy coding!