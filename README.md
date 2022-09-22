This package is a Ruby client for [Gotenberg](https://gotenberg.dev), a developer-friendly API to interact with powerful 
tools like Chromium and LibreOffice for converting numerous document formats (HTML, Markdown, Word, Excel, etc.) into 
PDF files, and more!

## Requirement

This packages requires [Gotenberg](https://gotenberg.dev), a Docker-powered stateless API for PDF files:

* 🔥 [Live Demo](https://gotenberg.dev/docs/get-started/live-demo)
* [Docker](https://gotenberg.dev/docs/get-started/docker)
* [Docker Compose](https://gotenberg.dev/docs/get-started/docker-compose)
* [Kubernetes](https://gotenberg.dev/docs/get-started/kubernetes)
* [Cloud Run](https://gotenberg.dev/docs/get-started/cloud-run)

## Installation

```ruby
gem "gotenberg-ruby"
```

## Usage

* [Send a request to the API](#send-a-request-to-the-api)
* [Chromium](#chromium)
* [LibreOffice](#libreOffice)
* [PDF Engines](#pdf-engines)
* [Webhook](#webhook)

### Send a request to the API

After having created the HTTP request (see below), you have two options:

1. Get the response from the API and handle it according to your need.
2. Save the resulting file to a given directory.

> In the following examples, we assume the Gotenberg API is available at http://localhost:3000.

### Chromium

The [Chromium module](https://gotenberg.dev/docs/modules/chromium) interacts with the Chromium browser to convert HTML documents to PDF.

#### Convert a target URL to PDF

See https://gotenberg.dev/docs/modules/chromium#url.

Converting a target URL to PDF is as simple as:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.url 'https://my.url'
end
```

You may inject `<link>` and `<script>` HTML elements thanks to the `extra_link_tags` and `extra_script_tags` arguments:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.url 'https://my.url', ['https://my.css'], ['https://my.js']
end
```

Please note that Gotenberg will add the `<link>` and `<script>` elements based on the order of the arguments.

#### Convert an HTML document to PDF

See https://gotenberg.dev/docs/modules/chromium#html.

You may convert an HTML document string with:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
end
```

You may also send additional files, like images, fonts, stylesheets, and so on. The only requirement is that their paths
in the HTML DOM are on the root level.

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.assets ['/path/to/my.css', '/path/to/my.js']
end
```

#### Convert one or more markdown files to PDF

See https://gotenberg.dev/docs/modules/chromium#markdown.

You may convert markdown files with:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.markdown wrapper_html, ['/path/to/file.md']
end
```

The first argument is a `Stream` with HTML content, for instance:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>My PDF</title>
  </head>
  <body>
    {{ toHTML "file.md" }}
  </body>
</html>
```

Here, there is a Go template function `toHTML`. Gotenberg will use it to convert a markdown file's content to HTML.

Like the HTML conversion, you may also send additional files, like images, fonts, stylesheets, and so on. The only 
requirement is that their paths in the HTML DOM are on the root level.

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.markdown wrapper_html, ['/path/to/file.md', '/path/to/my2.md']
  doc.assets ['/path/to/my.css', '/path/to/my.js']
end
```

#### Paper size

You may override the default paper size with:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.paper_size width, height
end
```

Examples of paper size (width x height, in inches):

* `Letter` - 8.5 x 11 (default)
* `Legal` - 8.5 x 14
* `Tabloid` - 11 x 17
* `Ledger` - 17 x 11
* `A0` - 33.1 x 46.8
* `A1` - 23.4 x 33.1
* `A2` - 16.54 x 23.4
* `A3` - 11.7 x 16.54
* `A4` - 8.27 x 11.7
* `A5` - 5.83 x 8.27
* `A6` - 4.13 x 5.83

#### Margins

You may override the default margins (i.e., `0.39`, in inches):

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.margins top: 1, bottom: 1, left: 0.39, right: 0.39
end
```

#### Prefer CSS page size

You may force page size as defined by CSS:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.prefer_css_page_size
end
```

#### Print the background graphics

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.print_background
end
```

#### Landscape orientation

You may override the default portrait orientation with:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.landscape
end
```

#### Scale

You may override the default scale of the page rendering (i.e., `1.0`) with:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.scale 2
end
```

#### Page ranges

You may set the page ranges to print, e.g., `1-5, 8, 11-13`. Empty means all pages.

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.native_page_ranges '1-2'
end
```

#### Header and footer

You may add a header and/or a footer to each page of the PDF:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.header header_html
  doc.html index_html
  doc.footer footer_html
  doc.margins top: 1, bottom: 1
end
```

Each of them has to be a complete HTML document:

```html
<html>
<head>
    <style>
    body {
        font-size: 8rem;
        margin: 4rem auto;
    }
    </style>
</head>
<body>
<p><span class="pageNumber"></span> of <span class="totalPages"></span></p>
</body>
</html>
```

The following classes allow you to inject printing values:

* `date` - formatted print date.
* `title` - document title.
* `url` - document location.
* `pageNumber` - current page number.
* `totalPages` - total pages in the document.

⚠️ Make sure that:

1. Margins top and bottom are large enough (i.e., `margins(top: 1, bottom: 1, left: 0.39, right: 0.39)`)
2. The font size is big enough.

⚠️ There are some limitations:

* No JavaScript.
* The CSS properties are independent of the ones from the HTML document.
* The footer CSS properties override the ones from the header;
* Only fonts installed in the Docker image are loaded - see the [Fonts chapter](https://gotenberg.dev/docs/customize/fonts).
* Images only work using a base64 encoded source - i.e., `data:image/png;base64, iVBORw0K....`
* `background-color` and color `CSS` properties require an additional `-webkit-print-color-adjust: exact` CSS property in order to work.
* Assets are not loaded (i.e., CSS files, scripts, fonts, etc.).

#### Wait delay

When the page relies on JavaScript for rendering, and you don't have access to the page's code, you may want to wait a
certain amount of time (i.e., `1s`, `2ms`, etc.) to make sure Chromium has fully rendered the page you're trying to generate.

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.wait_delay '3s'
end
```

#### Wait for expression

You may also wait until a given JavaScript expression returns true:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.wait_for_expression("window.status === 'ready'")
end
```

#### User agent

You may override the default `User-Agent` header used by Gotenberg:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.html index_html
  doc.user_agent("Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0")
end
```

#### Extra HTTP headers

You may add HTTP headers that Chromium will send when loading the HTML document:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.url 'https://my.url'
  doc.extra_http_headers [
    'My-Header-1' => 'My value',
    'My-Header-2' => 'My value'
  ]
end
```

#### Fail on console exceptions

You may force Gotenberg to return a `409 Conflict` response if there are exceptions in the Chromium console:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.url 'https://my.url'
  doc.fail_on_console_exceptions
end
```

#### Emulate media type

Some websites have dedicated CSS rules for print. Using `screen` allows you to force the "standard" CSS rules.
You may also force the `print` media type:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.url 'https://my.url'
  doc.emulate_media_type 'screen'
end
```

#### PDF Format

See https://gotenberg.dev/docs/modules/pdf-engines#engines.

You may set the PDF format of the resulting PDF with:

```ruby
document = Gotenberg::Chromium.call(ENV['GOTENBERG_URL']) do |doc|
  doc.url 'https://my.url'
  doc.pdf_format 'PDF/A-1a'
end
```
