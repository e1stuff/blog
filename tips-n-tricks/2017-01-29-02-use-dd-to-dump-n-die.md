---
layout: post
title: "#2 Use dd() to dump'n'die"
published_at: 2017-01-29
collection: tricks
index: 2
intro: |
      Quick debug with `dd($value)` dump values and die (stop execution).
--- 

# Use dd() to dump'n'die

You can use `dd($value)` function to dump'n'die​ values while debugging your code.

Or `dump($value)` to just dump without dying.

It's so much cooler than plain `var_dump()`.

## Installation

If you're using [Laravel](https://laravel.com/) you already have it. 

For the rest of you, just run this Composer command:

`composer require --dev larapack/dd:1.0`

## Caution

Do not use this function in production code. 
Always remove all the *dd()* calls before committing the code.
Along with *dump()* and *die()* and *var_dump()* where present.
