---
author: Lise Henry
title: 'Simulating a call to "parent" virtual method in Rust'

output.html: rust_inheritance.html
use_initials: false
---

Simulating inheritance in Rust 
==============================

Context 
-------

[Crowbook](https://github.com/lise-henry/crowbook) is a tool that
renders Markdown books (or articles) to HTML, Epub and PDF. In summary
it works like this.

First, a parser (using
[pulldown-cmark](https://crates.io/crates/pulldown-cmark)) reads all
the Markdown files and outputs some kind of AST (Abstract Syntax
Tree), which is basically a list of `Token`s, which look like
this:

```rust
enum Token {
    Str(String),
    Paragraph(Vec<Token>),
    Header(i32, Vec<Token>),
    Emphasis(Vec<Token>),
    ...
}
```

Then, a `Book` groups all the ASTs corresponding to each Markdown
file, information about the book options (author, title, where to
ouput the files, and so on).

Finally, the book is rendered in various formats, using different
renderers. There are currently 5 different renderers:

* `LatexRenderer` renders to text;
* `OdtRenderer` renders to Odt (well, it doesn't really, because it
doesn't really work currently, but the renderer exists anyway);
* `HtmlRenderer` renders to a single, self-contained HTML file;
* `HtmlDirRenderer` renders to multiple HTML files;
* `EpubRenderer` renders to an Epub file.

Now, let's look at the last three. Obviously, they share much of their
job, because they all deal with generating HTML, and, for most
variants of `Token`, they generate exactly the same code (the only necessary
differences are for `Link`s, since internal links must be a bit tweaked to work when there are
multiple files. and for `Image`s, which are included as base64 code in
the single self-contained file).

The object-oriented solution
----------------------------

Now, had I written Crowbook in an object-oriented programming
language, I would have had an obvious solution in mind: define a
`Renderer` interface or abstact class, then have all renderers
implement it/extend it. Then I guess I could have all three HTML-based
renderers derive from a common class. So, basically, something like
this ugly pseudo-UML graph:

![Class hierarchy](uml.png)

The current status
------------------

### A Renderer trait ###

Now, Rust isn't object-oriented, so you can't really do that. There
are, however, traits, that are more or less similar-ish to interfaces
in Java. So, all Renderers can implement a `Renderer` trait. And,
what's cool is that traits can have default implementations, so we
can avoid duplicating code:

```rust
trait Renderer {
    fn render_token(&mut self, token: &Token) -> Result<String>;
    fn render_vec(&mut self, tokens: &[Token]) -> Result<String> {
        tokens.iter()
            .map(|token| self.render_token(token))
            .collect::<Result<Vec<_>>>()
            .map(|vec| vec.join(""))
    }
}
```

So, we have a `render_token` method that must be implemented by each
renderer, and `render_vec` which is basically a fancy `for` loop.

And, well, for the renderers that are *really* different, that's quite
enough, because there isn't that much similarity between generating
LaTeX and HTML.

However, there are three differents HTML renderers, who share a
bit more code. 


### Composition over inheritance ###

Now, there is this whole idea of
[composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)
which, as far as I get it, is that you can avoid inheritance by
composing elements. So, in this light, it would seem logical that each
HTML-based renderer include the HTML common renderer[^1] and call it
when necessary.

[^1]: In practice, this isn't exactly the case: because the
single-file HTML renderer, `HtmlRenderer`, was the first I wrote, is
doesn't include anything, and both `EpubRenderer` and
`HtmlDirRenderer` (the renderer for multiple HTML files) include
it. But it should probably be refactored some day.


Now, this looks nice, because if the EPUB renderer just has to render
`Emphasis` tokens with `<i>` instead of `<em>`[^2], you could do
something like that:

```rust
impl Renderer for EpubRenderer {
    fn render_token(&mut self, token: &Token) -> String {
        match *token {
            Token::Emphasis(ref inner) => format!("<i>{}</i>", self.render_vec(inner)),
            _ => self.html.render_token(token)
        }
    }
}
```

That is, render `Emphasis` variant of `Token` differently, but fall
back on the HTML implementation to render all other variants.

In practice, unfortunately, this doesn't work, as you can see with the
full example on
[Rust playground](https://play.rust-lang.org/?gist=a22520c912e42f42b870222c768d0381&version=stable&backtrace=0). The
problem is that unlike a call to `super` in an object-oriented
language, calling `self.html.render_token` means that every other
methods call will be from `self.html`. So, the interest of having a
polymorphic method (thanks to trait) is lost.


[^2]: It doesn't, real differences are a bit more complicated than
that, but let's pretend that's it.


Her comes AsMut
---------------

Now, is there a way to emulate this kind of "inheritance" behaviour
with trait? Clearly, using only the `Renderer` trait is not
enough. The good news, though, is that it's still possible to have
some kind of polymorphism similar to virtual methods, even when there
was a previous call to a default implementation.

The idea is to move the implementation of `Renderer` for
`HtmlRenderer` in a separate, standalone function, and to make the
`impl` block call it:

```rust
struct HtmlRenderer {
    n_para: u32,
}
    
fn html_render_token<T:AsMut<HtmlRenderer>+Renderer>(this: &mut T, token: &Token) -> String {
    match *token {
        Token::Str(ref content) => content.clone(),
        Token::Paragraph(ref inner) => {
            this.as_mut().n_para += 1;
            let n = this.as_mut().n_para;
            format!("<p id = '{}'>{}</p>", n, this.render_vec(inner))
        },
        Token::Emphasis(ref inner) => format!("<em>{}</em>", this.render_vec(inner))
    }
}

impl AsMut<HtmlRenderer> for HtmlRenderer {
    fn as_mut(&mut self) -> &mut HtmlRenderer {
        self
    }
}

impl Renderer for HtmlRenderer {
    fn render_token(&mut self, token: &Token) -> String {
        html_render_token(self, token)
    }
}
```

Now, this separate function is basically the same code as the previous
implementation of `Renderer`, except it is now for
`AsMut<HtmlRenderer>`. This necessitates some boilerplate
(implementing `AsMut` for `HtmlRenderer`), but the good news is that
now, we can have `EpubRenderer` call it without problem:

```rust
impl AsMut<HtmlRenderer> for EpubRenderer {
    fn as_mut(&mut self) -> &mut HtmlRenderer {
        &mut self.html
    }
}

impl Renderer for EpubRenderer {
    fn render_token(&mut self, token: &Token) -> String {
        match *token {
            Token::Emphasis(ref inner) => format!("<i>{}</i>", self.render_vec(inner)),
            _ => html_render_token(self, token),
        }
    }
}
```

Notice that when `html_render_token` is called, it is with `self` and
not `self.html`. This means that, inside of it, it is the correct
implementation that will be called. Therefore,

```rust
let ast = // ...
let mut html = HtmlRenderer::new();
let mut epub = EpubRenderer::new();
println!("{}", html.render_vec(&ast));
println!("{}", epub.render_vec(&ast));
```

gives us different results, with the HTML printing `<em>` and the
EPUB one correctly printing `<i>`.


Conclusion 
----------

So, is it possible to do inheritance in Rust? No. Is it possible to
simulate a call to a parent virtual method by (ab)using traits and with some perseverance?
Yes. It took me a while to come to this solution, but I feel that the
boilerplate it adds, even if it *is* more complicated than a call to `super.foo()` isn't actually that bad[^3].

Thanks for all the people on Rust's reddit, I definitely wouldn't
have come with this idea without your help :)

Full working example is on [Rust Playground](https://play.rust-lang.org/?gist=64767cdcd417b7bd2ddccca014c83695&version=stable&backtrace=0).


[^3]: Unlike some discarded previous solutions I came up with, like
[this one](https://play.rust-lang.org/?gist=26fbe010ec854856a6bb96deddea726b&version=stable&backtrace=0).
