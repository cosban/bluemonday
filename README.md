# bluemonday [![Build Status](https://travis-ci.org/cosban/bluemonday.svg?branch=master)](https://travis-ci.org/cosban/bluemonday)

> bluemonday is a HTML sanitizer implemented in Go. It is fast and highly configurable.

bluemonday takes untrusted user generated content as an input, and will return HTML that has been sanitised against a whitelist of approved HTML elements and attributes so that you can safely include the content in your web page.

If you accept user generated content, and your server uses Go, you **need** bluemonday.

# Why this fork?

This fork of bluemonday was created in an attempt to make the sanitizing process a little bit more flexible.

Normally with bluemonday, if your user provides you with bad content (`bluemonday.UGCPolicy().Sanitize()`) turns this:
```html
Hello <STYLE>.XSS{background-image:url("javascript:alert('XSS')");}</STYLE><A CLASS=XSS></A>World
```

Into a harmless:
```html
Hello World
```

But what if you are looking for something a little more flexible? I frequently wish there was an option to, instead, turn the code into this:

```html
Hello &lt;style&gt;.XSS{background-image:url(&#34;javascript:alert(&#39;XSS&#39;)&#34;);}&lt;/style&gt;&lt;a class="XSS"&gt;&lt;/a&gt;World
```

Which will visually render to the original text on the screen without having to sacrifice the functionality of allowed tags.

But what about invalid attributes within the whitelisted tags? For this, we have opted to simply strip out the attribute and leave the valid parts intact.

This means that if your users try to provide you with this bad content:

```html
<b onclick="alert('XSS')">Hello</b> world!
```

You will be delighted to see that it is sanitized to a safe

```html
<b>Hello</b> world!
```

# Usage

All of the original policies are still available with this fork. The original usage is described in their [github page](https://github.com/microcosm-cc/bluemonday)

For WYSIWYG, install in your `${GOPATH}` using `go get -u github.com/cosban/bluemonday`

Then call it:
```go
package main

import (
	"fmt"

	"github.com/microcosm-cc/bluemonday"
)

func main() {
	p := bluemonday.WYSIWYGPolicy()
	html := p.Sanitize(
		`<a onblur="alert(secret)" href="http://www.google.com">Google</a>`,
	)

	// Output:
	// <a href="http://www.google.com" rel="nofollow">Google</a>
	fmt.Println(html)
}
```

You are able to use all three of the original methods to sanitize with this addition.
```go
p.Sanitize(string) string
p.SanitizeBytes([]byte) []byte
p.SanitizeReader(io.Reader) bytes.Buffer
```

## TODO

* More extensive tests to ensure there are not any cases being left out.
* Keep up-to-date with the original bluemonday
* Possibly submit a pull request for this after more testing.
