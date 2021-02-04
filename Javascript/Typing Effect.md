# Typing

The typing effect is a simple JavaScript effect to simulate a user typing on the keyboard. A demo can be found [here](https://jlsajfj.github.io/).

## Table of Contents



## HTML

Since this is a JavaScript plugin for a website, there is an HTML aspect. Shown below is the snippet taken from the demo site.

```HTML
<p>I'm <strong class="carousel" display-text='["seeking internships", "a Python nerd", "in UW Mechatronics", "dating a nerd", "learning cybersecurity"]'></strong><span class="cursor">|</span></p>
<script src="assets/js/typing.js"></script>
```

*Figure 1: HTML side of the plugin*

To be noted is that the display text is set in the HTML, rather than in the JavaScript, so the code does not need to be customized. Additionally the `|` representing the cursor has it's own class.

## JavaScript

The code is split into three major functions: The startup; the flashing cursor; and the typing itself.

### Startup

The startup script is required for a few things. The tag element is obtained, the cursor element is obtained, and the display text array is parsed. The initial text is set, and the flashing cursor is set to begin in 2000ms. The code is found [below](#startup-code).

### Flashing Cursor

Also simple, but took a while to figure out.

## Code

### Startup Code

```Javascript
function startup(){
    tag = document.getElementsByClassName('carousel')[0];// locating the carousel element
    displayText = JSON.parse(tag.getAttribute('display-text'));// parsing the string array of text
    cursor = document.getElementsByClassName('cursor')[0];// locating the cursor element
    tag.innerHTML = displayText[0];// setting initial text
    setTimeout(flashing, 2000);// initializing the flashing
}
```

### Flashing Code

```javascript
async function flashing(){
    cursor.style.opacity = 1;// reset opacity from faded
    for(var i = 0; i < 3; i++){// run three times
        await sleep(250);// set opacity to 0 and wait
        cursor.style.opacity = 0;
        await sleep(500);// set opacity to 1 and wait
        cursor.style.opacity = 1;
        await sleep(250);
    }
    cursor.style.opacity = 0.75;// since some cursors fade while typing, this was added
    typing()// begin typing
}
```

