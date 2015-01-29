---
layout: default
modal-id: 3
date: 2014-07-15
img: GOL.png
alt: image-alt
project-date: April 2014
category: Web Development
repository: https://github.com/RLuckom/jsGameOfLife
description: I wrote a JavaScript version of Conway's Game of life as a way to learn about cellular automata. The original version was written using SVG for the board, but I found that SVG rendered too slowly for large boards so I switched to using a Canvas element. <div id="GOL"></div> To run the game, click on individual squares to toggle them between the black and white states, then click "Play." The game stops when the cursor is over the board and restarts when it leaves. The slider controls the time between steps. My favorite pattern to start with is the one in the image above.
js: var goElement = document.getElementById('GOL'); var gol = new CanvasGameOfLife(300, 300, 20, 20); goElement.appendChild(gol.div);

---
