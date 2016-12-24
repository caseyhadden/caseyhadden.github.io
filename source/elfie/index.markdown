---
layout: page
title: "Goodbye and see you next year!"
date: 2015-12-24 22:03
---

I used your Daddy's computer to make my own web page. This seems like a
fun place to explore when I come back next year!

Santa helped a bit with the code and the picture!

{% img rotate90 http://caseyhadden.com/images/elfie.jpg 175 300 %}

### Update: 23Dec2016

I've been taking classes back at the north pole to learn how to program the
computer just like your Dad.

<button onclick="changeColor()">Click me!</button>

<div id="elfie-target" style="elfie-target" style="background:#ffffff;height:200px;width:50%">
Check out my skills!
</div>

It's not perfect, but I'm learning!

Here's a couple of pictures of my friends working in class.

{% img http://localhost:4000/images/elfie-1-class.png %}
{% img http://localhost:4000/images/elfie-2-class.png %}
{% img http://localhost:4000/images/elfie-3-class.png %}

<script type="text/javascript">
function changeColor() {
  var colors = ["#ffffff","blue","red","yellow"];
  var rand = Math.floor(Math.random()*colors.length);
  $('#elfie-target').css("background-color", colors[rand]);
}
</script>
