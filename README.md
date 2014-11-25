[![Build Status](https://travis-ci.org/russross/blackfriday.svg?branch=master)](https://travis-ci.org/russross/blackfriday)

Blackfriday
===========

Blackfriday is a [Markdown][1] processor implemented in [Go][2]. It
is paranoid about its input (so you can safely feed it user-supplied
data), it is fast, it supports common extensions (tables, smart
punctuation substitutions, etc.), and it is safe for all utf-8
(unicode) input.

HTML output is currently supported, along with Smartypants
extensions. An experimental LaTeX output engine is also included.

It started as a translation from C of [upskirt][3].


Installation
------------

Blackfriday is compatible with Go 1. If you are using an older
release of Go, consider using v1.1 of blackfriday, which was based
on the last stable release of Go prior to Go 1. You can find it as a
tagged commit on github.

With Go 1 and git installed:

    go get github.com/russross/blackfriday

will download, compile, and install the package into your `$GOPATH`
directory hierarchy. Alternatively, you can achieve the same if you
import it into a project:

    import "github.com/russross/blackfriday"

and `go get` without parameters.

Usage
-----

For basic usage, it is as simple as getting your input into a byte
slice and calling:

    output := blackfriday.MarkdownBasic(input)

This renders it with no extensions enabled. To get a more useful
feature set, use this instead:

    output := blackfriday.MarkdownCommon(input)

### Sanitize untrusted content

Blackfriday itself does nothing to protect against malicious content. If you are
dealing with user-supplied markdown, we recommend running blackfriday's output
through HTML sanitizer such as
[Bluemonday](https://github.com/microcosm-cc/bluemonday).

Here's an example of simple usage of blackfriday together with bluemonday:

``` go
import (
    "github.com/microcosm-cc/bluemonday"
    "github.com/russross/blackfriday"
)

// ...
unsafe := blackfriday.MarkdownCommon(input)
html := bluemonday.UGCPolicy().SanitizeBytes(unsafe)
```

### Custom options

If you want to customize the set of options, first get a renderer
(currently either the HTML or LaTeX output engines), then use it to
call the more general `Markdown` function. For examples, see the
implementations of `MarkdownBasic` and `MarkdownCommon` in
`markdown.go`.

You can also check out `blackfriday-tool` for a more complete example
of how to use it. Download and install it using:

    go get github.com/russross/blackfriday-tool

This is a simple command-line tool that allows you to process a
markdown file using a standalone program.  You can also browse the
source directly on github if you are just looking for some example
code:

* <http://github.com/russross/blackfriday-tool>

Note that if you have not already done so, installing
`blackfriday-tool` will be sufficient to download and install
blackfriday in addition to the tool itself. The tool binary will be
installed in `$GOPATH/bin`.  This is a statically-linked binary that
can be copied to wherever you need it without worrying about
dependencies and library versions.


Features
--------

All features of upskirt are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the `--tidy` option.  Without `--tidy`, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself agains JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Thread safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.


Extensions
----------

In addition to the standard markdown syntax, this package
implements the following extensions:

*   **Intra-word emphasis supression**. The `_` character is
    commonly used inside words when discussing code, so having
    markdown interpret it as an emphasis command is usually the
    wrong thing. Blackfriday lets you treat all emphasis markers as
    normal characters when they occur inside a word.

*   **Tables**. Tables can be created by drawing them in the input
    using a simple syntax:

    ```
    Name    | Age
    --------|------
    Bob     | 27
    Alice   | 23
    ```

*   **Fenced code blocks**. In addition to the normal 4-space
    indentation to mark code blocks, you can explicitly mark them
    and supply a language (to make syntax highlighting simple). Just
    mark it like this:

        ``` go
        func getTrue() bool {
            return true
        }
        ```

    You can use 3 or more backticks to mark the beginning of the
    block, and the same number to mark the end of the block.

*   **Autolinking**. Blackfriday can find URLs that have not been
    explicitly marked as links and turn them into links.

*   **Strikethrough**. Use two tildes (`~~`) to mark text that
    should be crossed out.

*   **Hard line breaks**. With this extension enabled (it is off by
    default in the `MarkdownBasic` and `MarkdownCommon` convenience
    functions), newlines in the input translate into line breaks in
    the output.

*   **Smart quotes**. Smartypants-style punctuation substitution is
    supported, turning normal double- and single-quote marks into
    curly quotes, etc.

*   **LaTeX-style dash parsing** is an additional option, where `--`
    is translated into `&ndash;`, and `---` is translated into
    `&mdash;`. This differs from most smartypants processors, which
    turn a single hyphen into an ndash and a double hyphen into an
    mdash.

*   **Smart fractions**, where anything that looks like a fraction
    is translated into suitable HTML (instead of just a few special
    cases like most smartypant processors). For example, `4/5`
    becomes `<sup>4</sup>&frasl;<sub>5</sub>`, which renders as
    <sup>4</sup>&frasl;<sub>5</sub>.

*   **Includes**, support including files with `{{filename}}` syntax.

*   **Indices**, using `(((item, subitem)))` syntax.

*   **Citations**, using the reference syntax `[p. 23][@RFC2535]`, the citation
    can either be informative (default) or normative, this can be indicated by using
    the `i` or `n` modifer: `[][n#RFC2535]`. The sections containing
    the references are outputted automatically, before the end of the document or
    when the back matter starts.
    To make the references work you can optionally include a filename:
    `[][#RFC2335,bib/reference.RFC.2525.xml` this only needs to happen once.
    If the reference is `RFC<number>` a default will be used (cmd line flag bib dir)
    If the reference is ... I-D another default will be used.

*  **Asides**, any paragraph with `A>` at the beginning of all lines is an aside.

*  **Notes**, any parapgraph with `N>` (TODO)

*  **Abstracts**, any paragraph with `AB>` (TODO)

*  **{frontmatter}/{mainmatter}/{backmatter}** Create useful divisions in your document.

*  **IAL**, kramdown's Inline Attribute List syntax. (TODO)

*  **Definitition lists**, (TODO)

Other renderers
---------------

Blackfriday is structured to allow alternative rendering engines. Here
are a few of note:

*   [github_flavored_markdown](https://godoc.org/github.com/shurcooL/go/github_flavored_markdown):
    provides a GitHub Flavored Markdown renderer with fenced code block
    highlighting, clickable header anchor links.

    It's not customizable, and its goal is to produce HTML output
    equivalent to the [GitHub Markdown API endpoint](https://developer.github.com/v3/markdown/#render-a-markdown-document-in-raw-mode),
    except the rendering is performed locally.

*   [markdownfmt](https://github.com/shurcooL/markdownfmt): like gofmt,
    but for markdown.

*   XML output: renders output as XML2RFCv2. 

Todo
----

*   More unit testing
*   Markdown pretty-printer output engine
*   Improve unicode support. It does not understand all unicode
    rules (about what constitutes a letter, a punctuation symbol,
    etc.), so it may fail to detect word boundaries correctly in
    some instances. It is safe on all utf-8 input.
*   Fix `<section>` output


License
-------

Blackfriday is distributed under the Simplified BSD License:

> Copyright © 2011 Russ Ross
> All rights reserved.
>
> Redistribution and use in source and binary forms, with or without
> modification, are permitted provided that the following conditions
> are met:
>
> 1.  Redistributions of source code must retain the above copyright
>     notice, this list of conditions and the following disclaimer.
>
> 2.  Redistributions in binary form must reproduce the above
>     copyright notice, this list of conditions and the following
>     disclaimer in the documentation and/or other materials provided with
>     the distribution.
>
> THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
> "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
> LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
> FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
> COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
> INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
> BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
> LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
> CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
> LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
> ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
> POSSIBILITY OF SUCH DAMAGE.


   [1]: http://daringfireball.net/projects/markdown/ "Markdown"
   [2]: http://golang.org/ "Go Language"
   [3]: http://github.com/tanoku/upskirt "Upskirt"
