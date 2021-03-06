# gulp-remember [![NPM version](https://badge.fury.io/js/gulp-remember.png)](http://badge.fury.io/js/gulp-remember) [![Build Status](https://travis-ci.org/ahaurw01/gulp-remember.svg?branch=master)](https://travis-ci.org/ahaurw01/gulp-remember)

gulp-remember is a [gulp](https://github.com/gulpjs/gulp) plugin that remembers files that have passed through it. gulp-remember adds all files back into the stream it has ever seen.

gulp-remember pairs nicely with [gulp-cached](https://github.com/wearefractal/gulp-cached) when you want to only rebuild those files that changed, but still need to operate on all files in the set.

```javascript
var remember = require('gulp-remember');
```

## Usage

This example shows a scenario in which you want to wrap all script files in some type of module system, then concatenate into one `app.js` file for consumption.

As long as your other plugins can keep up, this example showcases the Holy Grail of Build Tools™ - the ability to build once, `git checkout different-branch` (a branch with drastically different files), and the output is exactly what you would expect.

```javascript
var gulp = require('gulp'),
    header = require('gulp-header'),
    footer = require('gulp-footer'),
    concat = require('gulp-concat'),
    cache = require('gulp-cached'),
    remember = require('gulp-remember');

var scriptsGlob = 'src/**/*.js';

gulp.task('scripts', function () {
  return gulp.src(scriptsGlob)
      .pipe(cache('scripts')) // only pass through changed files
      .pipe(header('(function () {')) // do special things to the changed files...
      .pipe(footer('})();')) // for example, add a stupid-simple module wrap to each file
      .pipe(remember('scripts')) // add back all files to the stream
      .pipe(concat('app.js')) // do things that require all files
      .pipe(gulp.dest('public/'))
});

gulp.task('watch', function () {
  var watcher = gulp.watch(scriptsGlob, ['scripts']); // watch the same files in our scripts task
  watcher.on('change', function (event) {
    if (event.type === 'deleted') { // if a file is deleted, forget about it
      delete cache.caches['scripts'][event.path];
      remember.forget('scripts', event.path);
    }
  });
});
```

## API

### remember(name)

Returns a stream ready to remember files.

#### name (optional)

Type: `String`

The name of a specific cache you want to use. You may want to `remember('javascripts')` in one area of your build while also being able to `remember('images')` in another.

You can choose not to pass any name if you only care about caching one set of files in your whole build.

### remember.forget(name, path)

Drops a file from a remember cache.

#### name (optional)

Type: `String`

The name of the remember cache on which you want to operate. You do not need to pass this if you want to operate on the default remember cache.

#### path (required)

Type: `String`

The path of the file you wish to drop from the remember cache. The path is used under the covers as the unique identifier for the file within one remember cache.

## License

(MIT License)

Copyright (c) 2014 Aaron Haurwitz

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.