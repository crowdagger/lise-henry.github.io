---
import_config: ../common.book
title: A hacky localization library with macros

output.html: localization.html
---

# A hacky localization library with macros

## Context

I am writing this program called [Crowbook](https://github.com/lise-henry/crowbook) which basically converts
books (or, in this case, articles) written in Markdown to PDF, EPUB or
HTML. Now, there are quite a 
few similar programs, which might be better than mine, but what I
think is kind of a specificity of mine is that it does its best, when the book's
language is french, to generate a correct french typography (because
french typography has some rules about non-breaking spaces and stuff
and it can be painful to insert them manually in your Markdown files). 

Except Crowbook was only in english, which felt a bit stupid for a
program that, in my mind, was currently more useful to french
people (and if you ever heard the accent of French, you know that
the average french person isn't that good at english). So, I decided
to localize it.

To be honest, the two previous paragraphs have little to do with
the rest of this article, but it explains why the library is called
[**`crowbook`**`-localize`](https://github.com/lise-henry/crowbook-localize).

## What I mean by localization

Now, I'm really not an expert in the domain of correctly doing
localization, and to be honest I'm not even sure that I correctly get the
difference between `localization` and `internationalization`[^0] (a.k.a
`l10n` and `i18n` because, you know, people in computer science like
to make things clearer and easier to understand[^1]). So, to be clear, by
"localization", here I mean one, and *only one* feature :

[^0]: A simple
    clarification
    [I read on Stack Overflow](http://stackoverflow.com/questions/506743/localization-and-internationalization-whats-the-difference) defined:
	
* **internationalization** as the *process of designing an
application so that it has the functionality to change to a
different language without resorting to programmatic change of the
application* ;
* **localization** as the *process of creating the actual
language-specific texts and formatting.*
    
Which might actually be a bit blurry for a library that actually
*does* resort to *programmatic change of the application* to support
a new language, but generates it automatically.


* displaying the text of my program in a language set by the user of
  the program (obviously, if this translation exists; it's not a
  translating software).
  
  
[^1]: In the moments where I am the most paranoid, I wonder if these
    two abbreviations were actually chosen because `l` looks a bit
    like an uppercase `i` and `8` and `0` are a bit lookalike in some
    monospace fonts.


I'm aware that localization can also mean other things, such as
switching the way dates are printed, or what's the decimal mark, or
timezone stuff, but I tend to think that those are less urgent 
features[^2] than making my program understandable by someone who
doesn't speak english.

[^2]: And honestly, in some cases, features the user might not really
want that much; personally, even though I'm french, I *hate* it 
when a program decides that `3.14` should no longer be written `3.14`
but `3,14` because I set the language to french. I mean, yeah, when I
write on paper I would write `3,14` but I've integrated that when I'm
typing on a keyboard I should translate it to `3.14`, in the same way
that `7` lacks a bar in the middle and `4` has a vertical bar that
goes all the way to the top. More importantly, when I copy/paste a
formula on the Internet, I expect it to keep working even if my locale
is not the same as the one of the user that wrote the formula.



Now I'm not saying it wouldn't be nice if there was a library to do
that, if possible in an intelligent manner, but I don't think it's as
important as being able to provide an actual translation for people
who are not fluent in english.

## The state in Rust

As an ecosystem, Rust has areas where it is pretty good. I'm sorry to
say it, but internationalization/localization is not where it shines
the most. To
begin with, I didn't 
even find a way to get the language that the user is using. On Unix, it's
an environment variable; on Windows, if I understood correctly, some
registry key. That's quite of a problem for a command-line
application, because you can't have the "choose language" as an option
in the menu, and you probably don't want the user to have to run your
program with

```bash
$ myprogram --lang foo <OPTIONS>
```

every single time.

(For now, I don't have a solution for that, except getting the `LANG`
environment variable and, if it doesn't work, sorry for Windows users,
you'll have to speak english because I have no idea how to get that on
Windows.)

Now, are there libraries to actually translate the strings? Good news,
is, I found two (well, three, but one had only one version published
and it has been yanked):

* [l20n](https://crates.io/crates/l20n), an implementation
  of [l20n](http://l20n.org/);
* [message-format](https://crates.io/crates/message-format), an
  implementation
  of [ICU message formats](http://userguide.icu-project.org/formatparse/messages) 

As I said, I'm not a specialist on internationalization and
localization, so I'm not sure what I am going to say about `l20n`[^3] is really relevant,
but I'm going to do it anyway. To me, the 
[l20n](http://l20n.org) approach seems nice for the Web, but I'm not sure it's what 
I want for a command line application. Basically, the problem is I feel it 
requires quite substantial transformation from my existing source
code. Even if I found `gettext` quite painful to use every time I had
to (partly because it needed to be integrated with autoconf/automake),
I quite liked the minimal modification that was needed to the `.c`
files (basically, writing `printf(_("foo"))` instead of
`printf("foo")`.

[^3]: I only discovered
  about [message-format](https://crates.io/crates/message-format) when 
  writing this article, so I really didn't investigate it properly for
  my needs. Oops.


And, well, the thing is, to be honest, even if I'm not good at it, I
somewhat know a little about using gettext and `msgmerge` to update a
documentation, so it seemed to me that if I could use this format, it
would be neat.

So, because of that, and also because I wanted to play with some Rust
concepts that I had, until this point, stayed away from, I decided to try
and write my own library to translate strings.

## The "problem" of Rust

The problem with Rust, in this case, is the way `format!` (and its family:
`print!`, `println!` and so on) works. I mean, it has puzzled me when I learned
Rust: why did the most basic "Hello, World!" example *already need a
fracking macro* for `println!`? But, with time, I've come to like how arguments are
checked at compile time. Except, when you want to have the format
string set at runtime, it can cause difficulties. Consider the following:

```rust
println!("Hi, {}, how are you?", name);
```

Easy, simple. Now, let's say you want to translate this string. You
can't have

```rust
println!(get_translation("Hi, {}, how are you?"), name);
```

because the first argument must be a string literal. Now most of the
times I like that, but when I'm in this kind of situation, I must
admit the rigidity of Rust is a bit painful. It's not like there is a
`format-runtime!` macro, so basically the solution is using some kind
of runtime templating library (and, if I understood correctly, the two
localization/internationalization libraries I mentioned earlier work by
providing you such a library), which I find nice for translating
webpages but not when you 
have a thousand small strings to translate. Of course, this is not a
problem for strings that *don't* have this kind of arguments, in which
case you can just type:

```rust
println!("{}", get_translation("foo");
```

Except, looking at my program, I have:

* 1028 strings that need to be translated;
* 709 that have `{}` in them;

so knowing that only around 70% cause problem might not really be a
relief.

## Macro rules!

When I learned Rust, I read about macros, and I thought:

> Hell, that's complicated! Code that generate code at compile-time?
> This looks like black magic. I guess I'll never have to use that
> anyway.

But in this case, using a macro didn't seem like a bad way to handle
this. So I tried very hard not to be scared, fumbled a bit, cried a
little when it didn't work, 
but finally came with a working example:

```rust
#[macro_export] macro_rules! my_format {
    ("hello, {}", $($arg:tt)*) => ({
        match lang {
            "fr" => format!("bonjour, {}", $($arg)*),
            "es" => format!("hola, {}", $($arg)*),
            _ => format!("hello, {}", $($arg)*),
        }
    });
	($($arg:tt)*) => (format!($($arg)*));
}

```

When you ran it with 

```rust
let mut lang = "en";
println!("{}", my_format!("hello, {}", name));
lang = "fr";
println!("{}", my_format!("hello, {}", name));
lang = "es";
println!("{}", my_format!("hello, {}", name));
```

it translated (or not) the string according to the `lang`
variable. And if you ran it with

```rust
my_format!("other string")
```

it fall back to calling `format!` directly.

This was what I wanted. I didn't have to modify my existing code
besides replacing the `format!` macro with `my_format!` (or adding it
for some strings that were not using `format!`). And it would check
at compile time that my translation had the correct number of arguments.

## Generating the macro

Now, obviously, it wasn't very nice to hard-code all my strings in a
macro, and to ask potential translators to edit a giant macro. So I decided
that, at build-time, my library would read `.po` files containing the
translation in a gettext-like format and generate this
macro.

At this point, honestly, like Bruce Willis when he's about to throw a car to an
helicopter, I told myself: *"This is a bad idea"*. I mean, macros were
already black magic: code that generate code, seriously? So a build
script that generated a macro that would generate code felt a bit like
reciting unknown words from the Necronomicon and hoping for the best.

A bit to my surprise, it turned out to not be that bad. I quickly had
some library that was enough for my initial needs (providing a french
translation of my program). I simply had to replace `format!` to
`lformat!` and actually do the translation, and, well, it worked:

```bash
$ ./target/debug/crowbook guide.book 
Error: LaTeX (README.md): image 'https://travis-ci.org/lise-henry/crowbook.svg?branch=master' doesn't seem to be local; ignoring it.
Info: Successfully generated EPUB file: docs/book/book.epub
Info: Successfully generated HTML directory: docs/book/html
Info: Successfully generated ODT file: docs/book/book.odt
Info: Successfully generated HTML file: docs/book/book.html
Info: Successfully generated PDF file: docs/book/book.pdf

$ LANG=fr ./target/debug/crowbook guide.book 
Erreur : LaTeX (README.md) : l'image 'https://travis-ci.org/lise-henry/crowbook.svg?branch=master' n'a pas l'air d'être locale, elle sera ignorée
Info : Répertoire HTML généré avec succès : docs/book/html
Info : Fichier HTML généré avec succès : docs/book/book.html
Info : Fichier EPUB généré avec succès : docs/book/book.epub
Info : Fichier ODT généré avec succès : docs/book/book.odt
Info : Fichier PDF généré avec succès: docs/book/book.pdf
```

## The cost of black magic

But, well, I had read enough supernatural novels to know that black
magic comes with a cost, so I wondered what I, now, had to pay, and
hoped it wasn't my soul.

I looked at `localized_macros.rs`, the file generated at compile time,
and it was a bit frightening to see a thousand-line macro. While my
initial reaction was to grab the closest crucifix and shout *Vade
retro satanas!* I rationalised that it wasn't as horrifying at it
looked like. The macro (now called `lformat!` and using a global
`RWLock` variable to get 
the language), looked like this: 

```rust
#[macro_export] macro_rules! lformat {
    ("rendering {}:{}", $($arg:tt)*) => ({
        let __guard = $crate::localize_macros::__get_lang();
        match __guard.as_str() {
            "fr" => format!("dans le rendu de {} : {}", $($arg)*),
            _ => format!("rendering {}:{}", $($arg)*),
        }
    });
    ("Parsing chapter: {}...", $($arg:tt)*) => ({
        let __guard = $crate::localize_macros::__get_lang();
        match __guard.as_str() {
            "fr" => format!("Analyse du chapitre: {}...", $($arg)*),
            _ => format!("Parsing chapter: {}...", $($arg)*),
        }
    });
	// A thousand more lines
    ($($arg:tt)*) => (format!($($arg)*));
}
```

So basically, according to the format string, it either expanded to:

* if the string was in one of the `.po` file, a `match` statement;
* else, a direct call to `format!`.

The compile time was a few seconds slower (55s instead of 45s in
release mode), which is always bad news but wasn't as bad as I
had thought it would be (expanding a thousand-line macro a thousand of
times). The binary was a bit larger (3.926MB instead of 
3.795MB), but again, not as much as I had feared.

But how would it look like with not one, but ten translations? Or
one hundred? I wasn't gonna translate my program in a hundred
languages, but I could pretend I had by loading the french translation
a hundred times with different languages codes. The result I got where
the following:

|                  | No localization | en + fr | 10 languages | 100 languages |
|------------------|-----------------|---------|--------------|---------------|
| Macro size       | n/a (0)         | >1000LOC| >2400LOC     | >15,000 LOC   |
| Compile time     | 45s             | 55s     | 65s          | 275s          |
| Binary size      | 3.795MB         | 3.926MB | 4.268MB      | 7,656MB       |

Again, not as bad as I had feared. Yes, the compile time for 100 languages
is quite atrociously long, but when you get to 100 languages you
probably can have some `#[cfg(foo)]` to include the 100 translations
only when you're about to build a new release and not on each debug
build. Obviously, the binary has become quite big for 100 languages,
but it's only around half a meg larger than the non-localized binary plus the
100 `.po` files you'd have to distribute anyway.

(And, well, currently [Cargo](https://crates.io/) makes it pretty easy
to distribute a single binary with `cargo install`, but less to
distribute a binary *plus*
some resources files, so I tend to not bother and put all my resources
in the binary anyway, which might explain why the non-localized binary
is already quite big: it includes some HTML and LaTeX templates, some
`svg` files, and even an `ODT` file. I'm not sure it is the good thing
to do, but I'm not sure it is really a bad one either and, at this
time, it definitely looks like the easy way.)

I didn't talk about runtime performances because I don't think they
really matter in that case, as typically this macro is called only for
strings that will be displayed to the user, so unless your program
writes a million messages per second on `stdout` (or more plausibly in
a log file) it's probably not
where it will take time[^4]. I guess this implementation via a macro could
be argued to be faster than the equivalent localization involving a
runtime check of the arguments, but again I don't think it's really
important except for very specific cases.

[^4]: Since I saw macros as black magic I still ran a
    performance check to verify that the localization had not turned
    my program twice slower, but unsurprisingly there weren't any real difference.


## Macro quirks

### Error messages

An advantage of this approach is that it checks that the format of the
translation is correct at compile-time, which avoid runtime errors. A
disadvantage is that error 
messages aren't really obvious. E.g., on a few occasions I had strings
that were fuzzily matched by `msgmerge` that was being a bit too fuzzy,
which led to something like this in the `fr.po` file:

```
#: /home/lise/Progs/rust/crowbook/src/lib/html_dir.rs:89
msgid "could not create HTML directory {}:{}"
msgstr "impossible de créer le répertoire HTML {}"
```

It led to what I think is a good thing: a compile time error. Except,
not an obvious one:

```
error: argument never used
  --> src/lib/html_dir.rs:89:103
   |
89 |                                         lformat!("could not create HTML directory {}:{}", &dest_path, e))));
```

Though I guess the library could check that the arguments match and
print a more useful error, so maybe it could be enhanced. On the
other hand, it's quite possible this library has a few bugs that will
be generate yet more confusing error messages, so, to be clear, I'm
not saying you should use it right now.

### Strings handling

A bit more confusing to me is how strings are matched in macros and at
which points they are expanded. I mean, I can't write the following
macro, to have a localized variant of `println!`:

```rust
#[macro_export] macro_rules! lprintln {
    ($fmt:expr) => (println!("{}", lformat!($fmt)));
    ($fmt:expr, $($arg:tt)*) => (println!("{}", lformat!($fmt, $($arg)*)));
}
```

or, more exactly, I *can*, but it won't work, as `lformat!` will
always be expanded to the default case, directly calling `format!`:

```rust
($($arg:tt)*) => (format!($($arg)*));
```

and, thus, displaying an untranslated string.

I'm not sure why. I guess it's black magic.

## Limitations (but they could be lifted)

This approach has a few limitations. Obviously, you can't had support
for a new language at runtime, which might or might not be a feature
you need. On the other hand, I think the code generation approach
might actually provide more flexibility:

* It can provide this 'hybrid' approach between runtime and
  compile-time, where translations resources are parsed at compile
  time but the language can be set at runtime.
* It could also provide a "full compile-time" approach, generating
  different binaries for every different language. This would probably not make
  sense for most programs, but for some where the text is actually
  predominant (e.g., an interactive fiction?) it might be useful.
* At the opposite, it could also provide a "full runtime" approach, by
  replacing `lformat!` (or whatever the name of this macro would be
  in this fictional library) calls with functions/methods calls that
  would do the formatting and getting the appropriate resources at
  runtime.
  


Obviously, the current implementation of my library doesn't do that,
and it also has many more
limitations, such as not supporting plurals or gender variants
(because I don't know how to do it and I didn't need it), incomplete
compatibility with `gettext`'s po format, and probably a ton of bugs.

## Conclusion

When I started writing this library, I felt it was hacky and a
terrible idea that I had to try anyway. Now, I still feels it is quite
hacky, but I'm not sure if that's really a bad idea *per se* (yet I wonder if I
feel that way because I have been corrupted by the use of black
magic). 

I certainly would not recommend using this library yet, but I might
actually merge the changes I made to my program to the `master`
branch and hope that I don't regret it later on.

To be honest, I still half expect people to tell me "oh my, you really
shouldn't do that!", but in any case, it was fun playing with macro
generation. 
