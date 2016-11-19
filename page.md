---
import_config: common.book
author: ""
title: Lise Henry's page
lang: en
rendering.num_depth: 3

output.html: index.html
---

Lise Henry's Github page
=============================

Not really a webpage or a blog, just a space where I put stuff.

Articles
--------
* [A hacky localization library with macros](http://lise-henry.github.io/articles/localization.html)
* [Optimising string processing in Rust](http://lise-henry.github.io/articles/optimising_strings.html)
* [Simulating a call to "parent" virtual method in Rust](http://lise-henry.github.io/articles/rust_inheritance.html)

Projects and repositories
-------------------------

### Crowbook ###

[![Crates.io](https://img.shields.io/crates/v/crowbook.svg)](https://crates.io/crates/crowbook)
[![Crates.io](https://img.shields.io/crates/d/crowbook.svg)](https://crates.io/crates/crowbook)
[![Crates.io](https://img.shields.io/crates/l/crowbook.svg)](https://crates.io/crates/crowbook)
![Travis](https://img.shields.io/travis/lise-henry/crowbook.svg)

Renders a Markdown book in various formats: HTML, Epub, PDF. Though
it was not really made to make a website, this page was rendered using
this software.

* [Source code on Github](https://github.com/lise-henry/crowbook)
* [Crates.io](https://crates.io/crates/crowbook)

The development of Crowbook also spawned a number of subprojects, that are available as separate libraries:

#### crowbook-text-processing

[![Crates.io](https://img.shields.io/crates/v/crowbook-text-processing.svg)](https://crates.io/crates/crowbook-text-processing)
[![Crates.io](https://img.shields.io/crates/d/crowbook-text-processing.svg)](https://crates.io/crates/crowbook-text-processing)
[![Crates.io](https://img.shields.io/crates/l/crowbook-text-processing.svg)](https://crates.io/crates/crowbook-text-processing)

Some text processing functions initially written for Crowbook and
moved in a separate library (and a more permissive license) so they
can be used in other projects.

* [Source code on Github](https://github.com/lise-henry/crowbook-text-processing)
* [Crates.io](https://crates.io/crates/crowbook-text-processing)
* [Library documentation](https://docs.rs/crowbook-text-processing/)

#### crowbook-intl

[![Crates.io](https://img.shields.io/crates/v/crowbook-intl.svg)](https://crates.io/crates/crowbook-intl)
[![Crates.io](https://img.shields.io/crates/d/crowbook-intl.svg)](https://crates.io/crates/crowbook-intl)
[![Crates.io](https://img.shields.io/crates/l/crowbook-intl.svg)](https://crates.io/crates/crowbook-intl)

An attempt at writing a localization library using macros.

* [Source code on Github](https://github.com/lise-henry/crowbook-intl)
* [Crates.io](https://crates.io/crates/crowbook-intl)
* [Library documentation](https://docs.rs/crowbook-intl/)

### Caribon ###

[![Crates.io](https://img.shields.io/crates/v/caribon.svg)](https://crates.io/crates/caribon)
[![Crates.io](https://img.shields.io/crates/d/caribon.svg)](https://crates.io/crates/caribon)
[![Crates.io](https://img.shields.io/crates/l/caribon.svg)](https://crates.io/crates/caribon)
![Travis](https://img.shields.io/travis/lise-henry/caribon.svg)

A repetition detector written in Rust. Code is
[here](https://github.com/lise-henry/caribon), but you can also test
it [in your browser](http://vps184889.ovh.net/caribon/) because [caribon-server](https://github.com/lise-henry/caribon-server) runs it 
as a Webservice.

* [Source code on Github](https://github.com/lise-henry/caribon)
* [Crates.io](https://crates.io/crates/caribon)
* [Web service](http://vps184889.ovh.net/caribon/)

### Rust-ispell ###

[![Crates.io](https://img.shields.io/crates/v/ispell.svg)](https://crates.io/crates/ispell)
[![Crates.io](https://img.shields.io/crates/d/ispell.svg)](https://crates.io/crates/ispell)
[![Crates.io](https://img.shields.io/crates/l/ispell.svg)](https://crates.io/crates/ispell)

Easily call isell/aspell/hunspell from Rust programs.

* [Source code on Github](https://github.com/lise-henry/rust-ispell)
* [Crates.io](https://crates.io/crates/ispell)
* [Documentation](https://lise-henry.github.io/rust-ispell/ispell/)


### Stemmer-rs ###

[![Crates.io](https://img.shields.io/crates/v/stemmer.svg)](https://crates.io/crates/stemmer)
[![Crates.io](https://img.shields.io/crates/d/stemmer.svg)](https://crates.io/crates/stemmer)
[![Crates.io](https://img.shields.io/crates/l/stemmer.svg)](https://crates.io/crates/stemmer)
![Travis](https://img.shields.io/travis/lise-henry/stemmer-rs.svg)

Rust bindings to the Snowball C implementation, allowing you to stem a
word in various language. Used by Caribon.

* [Source code on Github](https://github.com/lise-henry/stemmer-rs)
* [Crates.io](https://crates.io/crates/stemmer)

### Tiny 'Nux Tarot ###

(Not maintained anymore.)

[![GitHub tag](https://img.shields.io/github/tag/lise-henry/tnt.svg)](https://github.com/lise-henry/tnt)
[![GitHub license](https://img.shields.io/github/license/lise-henry/tnt.svg)](https://github.com/lise-henry/tnt)


A tarot game written in Vala, using Gtk+.

* [Source code on Github](https://github.com/lise-henry/tnt)
* [Website](http://tnt.ouvaton.org/)
