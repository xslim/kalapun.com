---
published: true
title: Livereload quick start
---



Sometimes we need to try out something in HTML and want to be able to see our work in the browser instantly. You can reload your browser manually every time you make a change, or you can use "Live Reload" - http://livereload.com

My setup is as follows:

1. Install the server - `npm i -g livereload`
2. Install browser extention - http://help.livereload.com/kb/general-use/browser-extensions
3. Run the server in the folder where the test file is - `livereload .`
4. If using Safari - add this in the `<head>` section in your HTML file:

``` html
<script>
  document.write('<script src="http://' + (location.host || 'localhost').split(':')[0] +
  ':35729/livereload.js?snipver=1"></' + 'script>')
</script>

```


Now open the test file in your browser, edit & enjoy!
