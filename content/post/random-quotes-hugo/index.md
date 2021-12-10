---
title: "Random Quotes for Hugo"
subtitle: "A Dynamic Shortcode & Widget"
summary: ""
authors:
  - david-finster
tags:
  - Hugo
date: 2021-12-04T20:46:48-05:00
lastmod: 2021-12-04T20:46:48-05:00
featured: false
draft: false

quote: 
  style: 2
  display: true
  heading: "Quote of the Day"

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false
---
I wanted to put some random quotes on this website, but Hugo is a static site generator. I looked around to see if anyone had solved this problem, but I couldn't find any widgets. I thought this would be an excellent way to learn more about extending Hugo, so I wrote one. 

To see it in action, look at the bottom of this page. The quote will change each time you reload. 

If you'd like to add this to your Hugo site, it's pretty easy. 

## Source Files

I wanted to use the quotes on my home page and blog posts. So, I needed both a page partial and a shortcode. I wanted a DRY approach without code duplication, but Hugo passes the parameters differently to shortcodes than partials. All my attempts to avoid code duplication made it more confusing, and it seemed cleaner to clone it in both places. I may revisit this later if I find a better solution.

* The [shortcode is here](https://github.com/dfinr/www.dfinr.com/blob/master/layouts/shortcodes/quote.html). To use it, add it to your `/layouts/shortcodes` directory in your template.
* The [page partial is here](https://github.com/dfinr/www.dfinr.com/blob/master/layouts/partials/quote.html), and you should add it to `/layouts/partials`.
* You also need to place your [CSS styles](https://github.com/dfinr/www.dfinr.com/blob/master/assets/scss/custom.scss) in `/assets/scss/custom.scss`.

Finally, you need a quote file named `quotes.csv` at the root of your Hugo project. Here's [a link to mine](https://github.com/dfinr/www.dfinr.com/blob/master/quotes.csv).

The quote file has two fields, delimited with quotes and a pipe symbol. It looks like this:

    "This is the 'first' quote."|"— Someone"
    "This is the “second” quote."|"— Someone Else"

You'll notice that the fields are delimited with double-quotes. Because the file is read by Hugo and converted into JavaScript, it's not possible to escape double quotes. If you need double quotes in your file, use curly quotes as shown in the second example above.

## Shortcode Usage

To display a quote on your page, use this shortcode, which allows two optional parameters: style and heading.

```txt
{{%/* quote [style] [heading] */%}}
```

**style**: integer 

* Style `1` has no formatting or CSS. It displays a random quote and the source, separated by a single space. Style 1 does not support a heading.
* Style 2 is the default if you omit the parameter. It shows an optional heading, text, and the source, which can be formatted with CSS.

**heading**: string

* An optional title to display above quote.

### Examples

Here is a formatted quote in style 2 with no heading.

```txt
{{%/* quote */%}}
```

Here is an unformatted quote.

```txt
{{%/* quote 1 */%}}
```

Here is a formatted quote with a heading.

```txt
{{%/* quote 2 "Quote of the Day" */%}}
```

## Page Partial Usage

### Requirements

If you use the page partial version, you'll need to include some fields in the page front-matter:

```yaml
quote: 
  style: 2
  display: true
  heading: "Quote of the Day"
```

**style**: integer  

* Use the same style values as the shortcode.

**display**: bool

* true = display
* false = hide

**heading**: string

* Text to display above the quote.

To call the Page Partial:

```txt
{{ partial "quote.html" . }}
```

You can see it in action [on the home page](https://www.dfinr.com/), and the code example is near [the bottom of this page](https://github.com/dfinr/www.dfinr.com/blob/master/layouts/partials/widgets/about.html).

## Limitations

You can only use the random quotes one time on each page. This is a possible area for improvement in a future release. 

---
{{% quote 2 "Quote of the Day" %}}
