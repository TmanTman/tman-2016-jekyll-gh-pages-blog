---
layout: post
title: Consideration for Angular project setup
---

I've been building a complex reusable component for Angular. This introduces new challenges for
building the project, and the lessons I learn will undoubtly help me understand complex
projects better in the future.

One big realisation was that your unit test files should be the source files you work on, before your
gulp or grunt transformations have been applied to them. Karma does not reload the rewritten files
by Gulp well, it seems. So doing unit test on files that change due to a Gulp watch does not work.
Instead, the files that Karma should watch is the original source files. This implies that Gulp won't be
able to perform certain transformations on the files before Karma loads it. It's important to consider
what transformations is then valid to apply through Gulp, and which you can't use:

- gulp-ng-html2js loads html templates, transforms them to js files and assigns them to a module. This module
should by loaded by your other Angular modules to use the templates. But to your Karma source files, where
gulp's transform has not been applied, you won't have access to the module and you'll therefore have a error.
- if your code is minified, you need to specify your dependencies in strings, not only in parameter values.
Your code is not minified before Karma reads it, so you can safely with Gulp use ng-annotate and then minify it.
- the use of gulp-wrap to wrap your code in (function(){\n"use strict";\n<%= contents %>\n})();') is not applied to
the code you will test in Karma. To make sure you don't run into the problems that is solved by this IIFE pattern,
it's best to apply it to every file.
