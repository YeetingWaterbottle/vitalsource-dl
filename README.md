# VitalSource Downloader

This script lets you download an ebook from VitalSource in EPUB format.

## Usage

Go to [`index.js`](index.js). Change `globalCookieVal` to the cookie header sent from jigsaw.vitalsource.com in the browser after authentication. Then change `bookID` to the numerical ISBN of the ebook you're downloading.

```bash
npm start
```

Alternatively, you can also run `node index`

![](sample.png)

## Patching & Packing to EPUB

For me there are many issues with the files directly downloaded, they are intended to be rendered in the web browser so there are html and script tags present. They have also implemented some anti-piracy measures, such as hiding the book content with css rules. I dont know if this formatting was implemented recently, on select books, or for other reasons. The following is how I managed to pack the loose files into an actual EPUB file and be loaded into programs like Calibre.

### Create mimetype file

A valid EPUB file requires a mimetype created at the root of the epub folder, next to the `META-INF` and `OEBPS` folder.
The file should contain the content `application/epub+zip`.

Used for final zipping and packing of the loose epub file.

### Cleaning `META-INF/container.xml`

Original:

```xml
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">
<?xml version="1.0" encoding="UTF-8" ?>
<html>
   <body>
      <container version="1.0"
         xmlns="urn:oasis:names:tc:opendocument:xmlns:container">
         <rootfiles>
            <rootfile full-path="OEBPS/package.opf" media-type="application/oebps-package+xml"></rootfile>
         </rootfiles>
      </container>
      <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'8c98f6d770a852e3',t:'MTcyNzQxMzU4NS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script>
   </body>
</html>
```

Cleaned:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<container version="1.0"
   xmlns="urn:oasis:names:tc:opendocument:xmlns:container">
   <rootfiles>
      <rootfile full-path="OEBPS/package.opf" media-type="application/oebps-package+xml"></rootfile>
   </rootfiles>
</container>
```

<details>
<summary>Deobfuscated JS</summary>
<br>
<pre>
(function () {
  function c() {
    var b = a.contentDocument || a.contentWindow.document;
    if (b) {
      var d = b.createElement('script');
      d.innerHTML = "window.__CF$cv$params={r:'8c98f6d770a852e3',t:'MTcyNzQxMzU4NS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";
      b.getElementsByTagName('head')[0].appendChild(d);
    }
  }
  if (document.body) {
    var a = document.createElement('iframe');
    a.height = 1;
    a.width = 1;
    a.style.position = 'absolute';
    a.style.top = 0;
    a.style.left = 0;
    a.style.border = 'none';
    a.style.visibility = 'hidden';
    document.body.appendChild(a);
    if ('loading' !== document.readyState)
      c();
    else if (window.addEventListener)
      document.addEventListener('DOMContentLoaded', c);
    else {
      var e = document.onreadystatechange || function () {
      };
      document.onreadystatechange = function (b) {
        e(b);
        if ('loading' !== document.readyState) {
          document.onreadystatechange = e;
          c();
        }
      };
    }
  }
}());
</pre>
</details>

### Fixing `OEBPS/package.opf`

Two steps involved

1. First remove the html elements (html, body, script)

    Oiginal:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">
    <?xml version="1.0" encoding="UTF-8"?>
    <html>
    <body>
    <package xmlns="http://www.idpf.org/2007/opf" version="3.0" prefix="rendition: http://www.idpf.org/vocab/rendition/# rdf: http://www.w3.org/1999/02/22-rdf-syntax-ns# a11y: http://www.idpf.org/epub/vocab/package/a11y/#" unique-identifier="eisbn${ISBN}" xml:lang="en">
        <metadata xmlns:opf="http://www.idpf.org/2007/opf" xmlns:dc="http://purl.org/dc/elements/1.1/">
        ...
        </metadata>
        <manifest>
        ...
        </manifest>
        <spine toc="toc">
        ...
        </spine>
    </package>
    <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'8c98f6de341a52e3',t:'MTcyNzQxMzU4Ni4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script>
    </body>
    </html>
    ```

    Cleaned:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <package xmlns="http://www.idpf.org/2007/opf" version="3.0" prefix="rendition: http://www.idpf.org/vocab/rendition/# rdf: http://www.w3.org/1999/02/22-rdf-syntax-ns# a11y: http://www.idpf.org/epub/vocab/package/a11y/#" unique-identifier="eisbn${ISBN}" xml:lang="en">
        <metadata xmlns:opf="http://www.idpf.org/2007/opf" xmlns:dc="http://purl.org/dc/elements/1.1/">
        ...
        </metadata>
        <manifest>
        ...
        </manifest>
        <spine toc="toc">
        ...
        </spine>
    </package>
    ```

    <details>
    <summary>Deobfuscated JS</summary>
    <br>
    <pre>
    (function () {
    function c() {
        var b = a.contentDocument || a.contentWindow.document;
        if (b) {
        var d = b.createElement('script');
        d.innerHTML = "window.__CF$cv$params={r:'8c98f6de341a52e3',t:'MTcyNzQxMzU4Ni4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";
        b.getElementsByTagName('head')[0].appendChild(d);
        }
    }
    if (document.body) {
        var a = document.createElement('iframe');
        a.height = 1;
        a.width = 1;
        a.style.position = 'absolute';
        a.style.top = 0;
        a.style.left = 0;
        a.style.border = 'none';
        a.style.visibility = 'hidden';
        document.body.appendChild(a);
        if ('loading' !== document.readyState)
        c();
        else if (window.addEventListener)
        document.addEventListener('DOMContentLoaded', c);
        else {
        var e = document.onreadystatechange || function () {
        };
        document.onreadystatechange = function (b) {
            e(b);
            if ('loading' !== document.readyState) {
            document.onreadystatechange = e;
            c();
            }
        };
        }
    }
    }());
    </pre>
    </details>

<br>

2. Properly enclose the html tags. Use tools such as epubcheck to validate the opf file.

    [EPUBCheck](https://github.com/w3c/epubcheck) is a command line tool used to validate epub files, it can show errors and warnings in file structures and provide hints to fixing it.

    Go to [release](https://github.com/w3c/epubcheck/releases/latest/) and download the tools, run the tool with `java -jar epubcheck.jar`

    To check for the opf file structure, run the command `java -jar epubcheck.jar --mode opf path/to/package.opf`

    The command output will show the errors present in the file, you can manually fix, or ultilize chatgpt to use the error list and fix the file for you.

    Original:

    ```xml
        <meta property="schema:audienceType" refines="#aud01">schools
        <meta property="schema:educationalRole" refines="#aud01">student
        <meta property="schema:educationalUse">reading
        <meta property="schema:interactivityType">mixed
        <meta property="schema:accessMode">textual
        <meta property="schema:accessMode">visual
    ```

    Fixed:

    ```xml
        <meta property="schema:audienceType" refines="#aud01">schools</meta>
        <meta property="schema:educationalRole" refines="#aud01">student</meta>
        <meta property="schema:educationalUse">reading</meta>
        <meta property="schema:interactivityType">mixed</meta>
        <meta property="schema:accessMode">textual</meta>
        <meta property="schema:accessMode">visual</meta>
        <meta property="schema:accessModeSufficient">textual</meta>
        <meta property="schema:accessModeSufficient">visual</meta>
    ```

    Some of the meta properties might also require some modification to pass the epubcheck tool, experiment with this step.

### Force content visibility

All the xhtml files used for the book has an onload script that removes a style element that hides the page content

Sample XHTML file

```xml
<html xmlns:svg="http://www.w3.org/2000/svg" xmlns:epub="http://www.idpf.org/2007/ops" xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en" id="cf2b47c0-d2d6-3970-82a2-fae5d62ab713">

<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  <script type="text/javascript" id="vst-head-javascript" class="vst-ignore vst-skip">var VST = parent.VST; var EPUB = { Education: { retrieve: function (req, cb) { return VST.Book.retrieveActivityData(req, cb); }, send: function (req, cb) { return VST.Book.sendActivityData(req, cb); } } };</script>
  <meta charset="utf-8" />
  <meta name="process" content="final" />
  <meta name="schema" content="3.3" />
  <link href="../css/she_9781319281144.css" type="text/css" rel="stylesheet" />
  <link href="../css/common/lm_ui.css" type="text/css" rel="stylesheet" />
  <script src="../frost_widgets_data/glossary.js" type="text/javascript"></script>
  <script src="../javascript/jquery.min.js" type="text/javascript"></script>
  <script src="../javascript/lm_popup.js" type="text/javascript"></script>
  <style class="zcxqhgek" type="text/css">
    body {
      visibility: hidden !important;
    }
  </style>
  <script
    class="ybvazicf">eval(function (p, a, c, k, e, d) { e = function (c) { return c.toString(36) }; if (!''.replace(/^/, String)) { while (c--) { d[c.toString(a)] = k[c] || c.toString(a) } k = [function (e) { return d[e] }]; e = function () { return '\\w+' }; c = 1 }; while (c--) { if (k[c]) { p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]) } } return p }('a(9(){8 3=4.2(\'b\');c(d){3[0].6.5(3[0])}4.2(\'7\')[0].6.5(4.2(\'7\')[0])},1)', 14, 14, '||getElementsByClassName|ntrs|document|removeChild|parentNode|ybvazicf|var|function|setTimeout|zcxqhgek|if|VST'.split('|'), 0, {}))</script>
  <script src="../javascript/lm_frost_listner_min.js" type="text/javascript"></script>
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=5" />
  <title>Page Title</title>
</head>

<body epub:type="frontmatter" class="frontmatter" data-lo-framework-id>
...
</body>
<script type="text/javascript" id="vst-html-javascript" class="vst-ignore vst-skip">VST.Epub.contentReady(window, document); VST.cfiBase = '/4';</script>

</html>
```

Deobfuscated JS

```js
setTimeout(function () {
    var ntrs = document.getElementsByClassName("zcxqhgek");
    if (VST) {
        ntrs[0].parentNode.removeChild(ntrs[0]);
    }
    document.getElementsByClassName("ybvazicf")[0].parentNode.removeChild(document.getElementsByClassName("ybvazicf")[0]);
}, 1);
```

This on load script removes two elements:

-   `zcxqhgek` - The css styling that keeps the body hidden
    -   Only removed if `VST` exists, created in other script tags (haven't really looked into much)
-   `ybvazicf` - The onload script element

Two approaches to remove this restriction

-   Removing the script and style tags
    -   Pros: Removes bloat from xhtml files
    -   Cons: Randomized class names, could use pattern matching
-   Overriding the styling rule within the book style sheet
    -   Pros: Easiest to implement
    -   Cons: On load scripts are still present

I used the second method and has worked out fine.

Navigate to the common css file used in all the pages, for me it is `OEBPS/css/common/lm_ui.css`

To override the !important property on the visbility, we can use a css rule that is more specific than just body

I found that all the body tags had a property of `data-lo-framework-id`, so I used this as the css selector below

```css
body[data-lo-framework-id] {
    visibility: unset !important;
}
```

Open the xhtml pages to verify the modification has worked.

### Packing the files to EPUB

This step is less niche and you can find solutions online

I used: `zip -rX ../output.epub mimetype META-INF/ OEBPS/`

Other possible solutions below

https://gist.github.com/bitsgalore/da04413787931d20a8bf

https://ebooks.stackexchange.com/questions/257/how-to-repack-an-epub-file-from-command-line

### Converting book

The EPUB book is now completed, you can then load it onto calibre or other tools to convert, modify, and more.


If you liked this guide, idk, star this repo? i got too bored, byebye

## License

[MIT](LICENSE) © [Cyberscape](https://cyberscape.co/).
