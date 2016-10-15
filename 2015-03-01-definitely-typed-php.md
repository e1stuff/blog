---
title:          "Definitely Typed PHP"
date:           2015-03-01 +0200
categories:     [ php, DefinitelyTypedPhp, laravel ]
---

# Definitely Typed PHP

> ### Someone should implement .d.ts for PHP. Really.

Everything started with this basic idea. 
An ability to describe PHP classes and functions with special support files like 
[TypeScript Definition files](http://definitelytyped.org/guides/best-practices.html#getter-setter).


## Introduction

A week ago I started to learn Laravel Framework. 
The 5.0 version to be exact.
The framework is pretty mature and is really great. 
Its ideology is to keep things as much simple as possible.
It was really like a breath of fresh air for me after months spent with [Symfony2](http://symfony.com/) 
(which is way more complicated than Symfony 1 was).


## Laravel’s magic

Laravel is great at being simple. 
But that simplicity comes at cost. 
It uses lots of magic methods and overloaded functions.

I use [PhpStorm](https://www.jetbrains.com/phpstorm/) as my primary IDE and like it really much. 
The IDE has a strong autocompletion engine that helps to write code faster and make less mistakes. 
It’s understandable, that the engine cannot analyze magic methods.
So the typical Laravel controller looks like this:

<figure>
	<img src="images/typical-laravel-controller.png" alt="Typical Laravel controller opened with PhpStorm">
	<figcaption>Typical Laravel controller opened with PhpStorm</figcaption>
</figure>

Note the methods with olive background: `findOrFail` and `where`. 
Those methods are not recognized by IDE — they do not exist actually. 
They are magic. 
That’s wy we have IDE warnings and have no autocompletion.


## Problem with special IDE plugins

You can still continue to develop, it’s ok, but not really convenient. 
When you look though your code these warnings are distracting. 
Though there is [Laravel plugin for PhpStorm](https://github.com/Haehnchen/idea-php-laravel-plugin)
that solves some of the problems, for now it looks like it’s not adapted for Laravel 5 yet 
(Laravel 5 was officially released on Feb 4 2015; 25 days ago since I’m writing this article).

Problems with IDE plugins are:
 - Need to support plugins for each IDE
 - Need to know how to develop IDE plugins, which almost always requires skills not related to your main occupation
   (i.e. to develop PhpStorm plugin one need to know Java)
 - Need to support plugins for different framework versions
 - Time lag between framework and plugin development


## TypeScript Definition files

I was thinking about these problems for some time and then had one of that aha factors! 
Javascript has the same problems and more! 
All IDEs have really hard time trying to understand Javascript code and define return types to suggest you some autocompletion options. 
And I know one great approach to fight this. 
TypeScript definition files!

Javascript community has created a definition files repository to support IDEs with vital information for autocompletion engine: 
https://github.com/borisyankov/DefinitelyTyped.
All you need is to download definition file and link it to your typescript file explicitly:

~~~ ts
    /// <reference path=”jquery/jquery.d.ts” />
~~~

Also an IDE can use .d.ts internally. 
It can automatically download .d.ts files for libraries being used in project and link them implicitly. 
Wouldn’t it be great?!

Definition files solve all the problems of framework autocompletion plugins:
 - Files are IDE-independent. All is needed is to implement that files support for IDEs. Once for each IDE.
 - No need to know Java/C++/ObjectiveC/any-other-language to implement autocompletion for any PHP library
 - Community can easily support plugins for each framework version
 - Framework community can bundle up-to-date definition files with a framework!

### Examples

[Sample definition](http://definitelytyped.org/guides/best-practices.html#getter-setter) 
from DefinitelyTyped.org site (overloaded function in javascript):

~~~ ts
    declare function duration(): number;
    declare function duration(value: number): void;
~~~

JQuery definition file, fragment with `css()` function:

~~~ ts
    /**
     * Get the value of style properties for the first element
     * in the set of matched elements.
     *
     * @param propertyName A CSS property.
     */
    css(propertyName: string): string;
    /**
     * Set one or more CSS properties for the set of matched elements.
     *
     * @param propertyName A CSS property name.
     * @param value A value to set for the property.
     */
    css(propertyName: string, value: string|number): JQuery;
    /**
     * Set one or more CSS properties for the set of matched elements.
     *
     * @param propertyName A CSS property name.
     * @param value A function returning the value to set.
     *              this is the current element. Receives the index
     *              position of the element in the set and the old
     *              value as arguments.
     */
    css(propertyName: string, value: (index: number, value: string) => string|number): JQuery;
    /**
     * Set one or more CSS properties for the set of matched elements.
     *
     * @param properties An object of property-value pairs to set.
     */
    css(properties: Object): JQuery;
~~~

It is interesting that jQuery itself is a library of overloaded helper functions.


## PHP Definition files idea

We could use the same approach for PHP too! 
These definition files can solve some problems with PHP coding:
 - Magic methods hinting
 - Overloaded functions hinting
 - Application-specific service container keys hinting

### Magic methods

Let’s take the first example with Laravel’s Eloquent model class. 
It’s definition could look like this (a little simplified):

~~~ ts
    namespace App {
       class Question {
           public string $title;
           public string $body;
           // and so on ...
           public static find(int|string $id): Question;
           public static find(array $ids): Collection;
           // and so on ...
       }
       // ... other model classes
    }
~~~

### Overloaded functions

Though the main cause of thinking of PHP Definition Files was to hint magic methods, 
it also can help with the overloaded functions (like TypeScript Definition does)!

Think of PHP parse_url function:

> ~~~
> mixed parse_url ( string $url [, int $component = -1 ] )
> ~~~
>
> If the `$component` parameter is omitted, an associative array is returned.
>
> If the `$component` parameter is specified, `parse_url()` returns a *string*
> (or an *integer*, in the case of `PHP_URL_PORT`) instead of an array.
>
> If the requested component doesn’t exist within the given URL, `NULL` will be returned.


Currently PHPDoc cannot cover this cases. 
The most you can do is to set return type to mixed.
But we can document that behavior using special definition file:

~~~ ts
    parse_url(string $url): string[];
    // Note constant matching
    parse_url(string $url, PHP_URL_PORT): int|null;
    parse_url(string $url, int $component): string|null;
~~~

Another example. 
Session helper from Laravel framework:

~~~ php
<?php
    /**
     * Get / set the specified session value.
     *
     * If an array is passed as the key,
     * we will assume you want to set an array of values.
     *
     * @param  array|string  $key
     * @param  mixed  $default
     * @return mixed
     */
    function session($key = null, $default = null)
    {
        if (is_null($key)) return app('session');

        if (is_array($key)) return app('session')->put($key);

        return app('session')->get($key, $default);
    }
~~~

Definition code for this function could be something like:

~~~ ts
    /**
     * Get Session object
     */
    session(): SessionInterface;
    /**
     * Get specified session value
     */
    session(string $key, mixed $default = null): mixed;
    /**
     * Set specified session values [$key => $value, ... ]
     */
    session(array $values): void;
~~~

### Service Container hinting

Today is the era of Service Containers and Dependency Injection in PHP. 
Every modern framework uses some kind of Service Container implementation. 
And everytime we use a method of some service we get IDE warning.

~~~ php
<?php
    $container->get("session")->set("user_id", $user_id);
~~~

To make IDE ok with that we need to add some boring boilerplate code:

~~~ php
<?php
    /** @var $session SessionInterface */
    $session = $container->get("session");
    $session->set("user_id", $user_id);
~~~

A simple one-liner now is three times bigger! 
And it looks more complicated when you try to look through your code quickly!
What if IDE could tell return type that service container’s get() calls? 
Actually some specialized Framework plugins do this
(see [Working with the Service Container — Symfony Development using PhpStorm](php-storm-symfony-support)).

Try to look at this Symfony 2 plugin’s repo at GitHub:
[Haehnchen/idea-php-symfony2-plugin](idea-php-symfony2-plugin).
It’s nearly impossible that a usual PHP developer could code it!

Instead, in a Perfect World™, a developer could hint service container entries with application-specific definition files:

~~~ php
<?php
    class ServiceContainer {
      public function get('session'): SessionInterface;
      public function get('files'): FilesystemInterface;
      public function get('templating'): \Twig\Environment;
      // .. and so on
      // general case
      public function get(string $key): mixed;
    }
~~~

That could give IDE ideas on what’s going on. 
Also developers could have dedicated definition generator scripts for each framework to leverage this functionality automatically!


## Alternatives

Yes, there are some alternative approaches to solve the same issues with existing tools.

### PHPDoc blocks

You can use PHPDoc blocks with `@method` hints to denote magic methods. 
Yes, it works. And also you can auto-generate them and inline into existing classes. 
But that solves only the first issue.

Maybe we can extend PHPDoc to allow describing overloaded functions in future. 
But it definitely won’t help to deal with service containers hinting.

### Autogenerated helper files

It is possible to generate special helper files with all PHPDocs included to help IDE understand the code.
Actually there are some plugins that do so.
See [Laravel IDE Helper Generator](https://github.com/jonphipps/laravel4-idehelper-generator).

To hint existing classes we can:
- Generate separate file with dummy class declaration of the same name with all PHPDocs included.
  But IDE will notice it and display a warning: *duplicated class declaration*
- Inline generated PHPDoc blocks into existing files. 
  Which is not really convenient as it can replace our hand-crafted doc blocks.
  And also cannot be used to describe out-of-vcs 3-rd party classes (i.e. all files from vendor folder).


## Conclusion

Implementing such functionality require a significant amount of work to be done. 
Also we need to introduce a new language (though it almost matches PHP itself) and support it’s specification. 
Some of the problems Definition Files could solve may be solved by improving existing tools. 
So we should really think if it’s worth powder and shot first.

As for me, I think it is a great idea. 
It may be a serious game-changer! Many developers would benefit from it:

- framework communities could easily provide rich methods specifications for IDE autocompletion
- web application developers could increase development speed be leveraging autocompletion for 100%
- IDE developers could use these definition files as specifications source additionally to PHPDoc blocks

I’ve created a feature request at JetBrains issue tracker system: https://youtrack.jetbrains.com/issue/WI-26563

Any thoughts and feedback will be appreciated.


## Update

Added on July 3, 2015.

Today I’ve found out there is a PhpDoc PSR proposal draft that can solve some of described problems in a standardized
way (which, I believe, will be a huge step forward for the entire PHP community).

Take a look at [PhpDoc PSR proposal draft](https://github.com/phpDocumentor/fig-standards/blob/master/proposed/phpdoc.md).
At current state [@typedef](https://github.com/phpDocumentor/fig-standards/blob/master/proposed/phpdoc.md#822-typedef)
tag can solve at least “undocumented magic methods on 3rd party code” problem.

- I’ve also updated PhpStorm feature request to denote what problems can be solved with PSR https://youtrack.jetbrains.com/issue/WI-26563
- I’ve created an issue to PSR GitHub repository with proposal to support overloaded functions: https://github.com/phpDocumentor/fig-standards/issues/82



[php-storm-symfony-support]: https://confluence.jetbrains.com/display/PhpStorm/Working+with+the+Service+Container+-+Symfony+Development+using+PhpStorm
[idea-php-symfony2-plugin]: https://github.com/Haehnchen/idea-php-symfony2-plugin/tree/master/src/fr/adrienbrault/idea/symfony2plugin
