## Requirements

* Raku (see https://raku.org/downloads/ for installation instructions)
* Pear

## Rendering the website

After everything is properly setup:

* Clone this repo and cd into `pear-demo`:

```
$ git clone https://github.com/uzluisf/pear-demo.git
$ cd pear-demo
```

* Render all templates and content files:

```
$ pear render
```

A `public` directory will be created. You can either open it with a browser
and navigate to each HTML file to view it. Or else...

* Start a local web server

```
$ pear serve
```

By default, the site will be served at http://localhost:3000/.

**NOTE:** For the urls to be handled properly, you might
need to set the `base-url` (in `config.yaml`) to either the current
directory (if you're navigating the site through the browser's file manager)
or to `/` if you're serving
the site. 
