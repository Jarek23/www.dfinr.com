---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Random Quotes for Hugo"
subtitle: "Version 1: A CSV File, a Shortcode, and a Partial Template"
summary: ""
authors:
  - david-finster
tags:
  - Hugo
date: 2021-12-04T20:46:48-05:00
lastmod: 2021-12-04T20:46:48-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false
---

When I ran a Fido BBS, I was a big fan of the random quotes shown to users when they logged on. Unfortunately, that feature fell out of favor in the IT world for some reason. I could try to analyze why or claim that "simpler times were better," but that's not true. Things in IT, on the whole, are much better today. This is just a feature that I liked and missed. I also wanted to learn more about writing code for Hugo, and this was a good beginner challenge. So, I decided to add that feature to my website, and here's how I did it. 

This is the quote in action. If you reload the page, the quote changes.

---

{{% quote 2 "Example Quote" %}}

---

The shortcode looks like this:

```md
{{%/* quote 2 "Example Quote" */%}}
```

I made the shortcode flexible, with two different styles and an optional header. Style 1 is plain text with no formatting. Style 2 separates each element into its own `<div>` with a CSS class. The header is optional. The example above is style 2.

I also wanted to use this feature on the home page, which meant I needed to write a page partial to embed in the Hugo widget. However, I didn't want to maintain two versions, so I decided to focus on the page widget version first, and then make a thin wrapper for the shortcode.

## Using Dynamic Content with Hugo

Back in the BBS days, we curated our quotes in text files. When the BBS needed to show a quote, it would randomly grab one from the list. This is similar to how the Unix `fortune` program worked also. However, Hugo is a static site generator. If I wrote this in pure Hugo template code, I could get random quotes, but they would only change on each rebuild, which might be weeks or months apart. I wanted the quotes to change on every page load. I solved that by mixing in some inline JavaScript. This system isn't ground-shaking; I'm only writing this post because I didn't find anyone else doing it.

### The Flat File Format

I chose a flat-file format because it is easy to maintain, and a lot of my source material comes from archives of the old BBS systems I found online. The file has two fields, delimited with quotes and pipes. It looks like this:

```txt
"Find out who you are. And do it on purpose."|"— DollyParton"
"The only Zen you find on the tops of mountains is the Zen you bring up there."|"— Robert M. Pirsig"
```

The first field is the `quote`, and the second is the `source`. I separated them so I could do different formatting for each, as you saw above. The file is named `quotes.csv`, and it lives in the root of the project. You can [see it on GitHub](https://github.com/dfinr/www.dfinr.com/blob/master/quotes.csv).

### The Widgets

First, I created a folder in my theme called `/layouts/partials/quotes`. You can [see it on GitHub](https://github.com/dfinr/www.dfinr.com/tree/master/layouts/partials/quotes). In that folder, I created the main widget and two sub-widgets. 

This probably isn't the best structure, and I might refactor it later. I knocked this out on a Saturday afternoon, and it's just a cheezy quote feature, so 🤷🏻‍♂️ whatever.

The primary file is `quote.html`. It's well-documented in the comments, so see the raw file for more information. Here are the working guts:

```
{{ $ := .root }}
{{ $page := .page }}
{{ $qstyle := $page.Params.quote.style | default 1 }}
{{ $qdisplay := $page.Params.quote.display | default false }}
{{ $qheading := $page.Params.quote.heading | default "" }}

{{if eq $qdisplay true}}
    {{if eq $qstyle 1}}
        {{ partial "quotes/quotestyle1.html" . }}
    {{end}}
    {{if eq $qstyle 2}}
      {{ partial "quotes/quotestyle2.html" $qheading }}   
    {{end}}
{{end}}
```

It's not terribly complicated. It grabs three parameters from the page front-matter:

* **$qstyle**: Should Hugo show style 1 or 2? Style 2 is the default.
* **$qdisplay**: Show the quote, true of false?
* **$qheading**: An optional quote header.

Next, it performs two nested `if` conditions to pick the right widget, either `quotestyle1.html` or `quotestyle2.html`. That it. If you use style two, it passes `$qheading` as a parameter to the page partial or null if no heading is available. You can see this [in action on the home page](https://raw.githubusercontent.com/dfinr/www.dfinr.com/master/content/home/about.md), where the front-matter block looks like this:

```json
quote: 
  style: 2
  display: true
  heading: "Random Quote of the Minute"
```

The `/content/home/about.md` file is just a widget declaration for the home page which calls [the real page widget](https://github.com/dfinr/www.dfinr.com/blob/master/layouts/partials/widgets/about.html) in `/layouts/partials/widgets/about.htm`, and that's where the actual quote widget is embedded near the bottom:

```
<div class="col-md-7">
  {{ partial "quotes/quote.html" . }}
</div>        
```

That Hugo partial passes the `.` variable, which makes all the front-matter available to the widget as you saw above, and it calls one of the two random quote generators.

## The Actual Code

So far, this has all just been set up and templating. The real code lives in `layouts/partials/quotes/quotestyle2.html`, and the companion `quotestyle1.html`. Those are very similar, so I'll just explain style 2 here because it's the more interesting version.

This is the complete `quotestyle2.html` file:

```
<!-- Style 2: formatted random quotes-->
<div id="qheading" class="qheading">{{ . }}</div>
<div id="qtext" class="qtext"></div>
<div id="qsource" class="qsource"></div>

<script>
    var qtext=new Array();
    var qsource=new Array();

    {{ range $i, $quote := getCSV "|" "quotes.csv"  }}
      {{ $q := split (index . 0) "|" }}
      {{ $s := split (index . 1) "|" }}
      qtext[{{ $i }}] = {{ $q }}; 
      qsource[{{ $i }}] = {{ $s }};
    {{ end }}

    index = Math.floor(Math.random() * qtext.length);
    document.getElementById("qtext").innerHTML = qtext[index];
    document.getElementById("qsource").innerHTML = qsource[index];
</script>
```

It's three divs and a small block of mixed JavaScript and Hugo code. 

The first div, the heading, receives the heading string as the dot variable. The other two divs are empty placeholders for the JavaScript to populate. They are styled by [custom CSS you can inspect here](https://github.com/dfinr/www.dfinr.com/blob/master/assets/scss/custom.scss).

Because Hugo is a static site generator, this block of Hugo code will only run one time when the site is built. What you see here isn't the full JavaScript, it's a Hugo JavaScript builder. If you want to see the full JavaScript, you'll need to view the HTML source for this page in your browser. This Hugo code reads the CSV file, loops over the range, and turns it into two JavaScript arrays populated with the quote file. 

`qtext[]` is the array of quotes. `qsource[]` is the array of corresponding sources. After the arrays are populated, the last three lines handle the display on each page load. The code picks a random number on each page load and injects the corresponding text into the divs.

Performace-wise, it seems fine. As far as the browser is concerned, it's just text, and it's all pre-built. So a few hundred quotes should be no problem.

And that's it! Oh wait, I nearly forgot the shortcode wrapper.

## The Shortcode Wrapper

Up to this point, we have a page partial to include in other static page widgets, but there's no way yet to put a quote in the middle of a blog post or other Markdown content. For that, we need a shortcode. You'll [find it here ](https://github.com/dfinr/www.dfinr.com/blob/master/layouts/shortcodes/quote.html)in `/layouts/shortcodes/quote.html`. It's well documented in the file comments; here are the highlights.

The shortcode can take zero, one, or two parameters as we explained at the top. As a reminder, here's that full example shortcode again:

```md
{{%/* quote 2 "Example Quote" */%}}
```

And here's the actual code that executes in `/layouts/shortcodes/quote.html`:

```
{{ $qstyle := .Get 0 | default 2 }}
{{ $qheading := .Get 1 | default "" }}

{{if eq $qstyle 1}}
  {{ partial "quotes/quotestyle1.html" . }}
{{end}}
{{if eq $qstyle 2}}
  {{ partial "quotes/quotestyle2.html" $qheading }}   
{{end}}
```

First, it grabs any available parameters in `$qstyle` and `$qheading`. Then, it makes the same calls to the page partials as before. This Don't Repeat Yourself (DRY) structure means there's only one place to edit; this is just syntactic sugar to make Hugo happy. 

## Conclusion

That's it. It's not hard, and it took about ten times more words to explain it than it did to write it. Unfortunately, I couldn't find any other working example online, so this is my approach. I'm a Hugo beginner, and you might see things I could have done better. If so, please let me know! I'm happy with the result, but I'd like to learn if this is the best practice. 

You can contact me through any of the ways listed below. 
