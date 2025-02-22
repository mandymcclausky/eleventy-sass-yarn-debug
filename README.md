# eleventy-sass

Yet another Sass/SCSS plugin for [Eleventy](https://www.11ty.dev/).

## Why another Sass plugin?

There are several [Eleventy plugins](https://www.npmjs.com/search?q=keywords%3Aeleventy-plugin%20sass) to support Sass/SCSS files, already.
Even the official Eleventy website has [a page](https://www.11ty.dev/docs/languages/custom/#example-add-sass-support-to-eleventy) which describes how to handle Sass/SCSS files with your [Eleventy](https://www.11ty.dev/) project.

I'm not satisfied with the above solutions. Because:

- The existing plugins do watch Sass/SCSS files and write CSS files by themselves or by using another toolkit, such as gulp.js, instead of using [Eleventy](https://www.11ty.dev/)'s file watching and writing functionality. They might work well, but doesn't seem to be integrated enough with Eleventy.

- [The page in the official Docs](https://www.11ty.dev/docs/languages/custom/#example-add-sass-support-to-eleventy) is great if you only have Sass/SCSS files which do not have `@use` or `@forward` rules.<br />
If you use `@use` in your Sass/SCSS files, for example, and you change a dependency Sass/SCSS file, [Eleventy](https://www.11ty.dev/) will compile it (if its filename doesn't start with "\_"), but won't compile the dependant Sass/SCSS files, if you are following the instructions in [the page of the official Docs](https://www.11ty.dev/docs/languages/custom/#example-add-sass-support-to-eleventy).

In contrast to the existing solutions, [eleventy-sass](https://github.com/kentaroi/eleventy-sass) manages dependencies between Sass/SCSS files and makes maximum use of Eleventy's functionality. It is highly configurable, but it just works if you add it in your Eleventy config file.

For those of you who are curious about how [eleventy-sass](https://github.com/kentaroi/eleventy-sass) handles Sass/SCSS files, [here is a brief explanation](#how-eleventy-sass-handles-sassscss-files).

## Installation
```shell
npm install eleventy-sass
```

## Usage
Open up your Eleventy config file (probably `.eleventy.js`) and use `addPlugin()`:

```javascript
const eleventySass = require("eleventy-sass");

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(eleventySass);
};
```

## Options
[eleventy-sass](https://github.com/kentaroi/eleventy-sass) allows you to customize the behavior by options like follows:

```javascript
const path = require("path");
const eleventySass = require("eleventy-sass");

const options = {
  compileOptions: {
    permalink: function(contents, inputPath) {
      return path.format({
        dir: "stylesheets",
        name: path.basename(inputPath, path.extname(inputPath)),
        ext: ".css"
      });
    }
  },
  sass: {
    loadPaths: ["src/_includes"],
    style: "expanded",
    sourceMap: true,
  },
  defaultEleventyEnv: "development"
};

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(eleventySass, options);
};
```

Basically, `options` you pass as the second argument for `addPlugin()` is used for options for `eleventyConfig.addExtension()`, which [eleventy-sass](https://github.com/kentaroi/eleventy-sass) calls internally. The full options list is [provided in the official Docs](https://www.11ty.dev/docs/languages/custom/#full-options-list).

However, there are two exceptions.

One is `defaultEleventyEnv` key. As described later, [eleventy-sass](https://github.com/kentaroi/eleventy-sass) changes behavior based on the environment/shell variable `ELEVENTY_ENV` and if `ELEVENTY_ENV` is not supplied, it considers the environment `production`. If you want to change this default behavior, you can set the default environment to the value for `defaultEleventyEnv` key.

The other exception is `sass` key. The value for `sass` key is used for options for `sass.compileString()`, [a dart-sass API function](https://sass-lang.com/documentation/js-api/modules#compileString), which also [eleventy-sass](https://github.com/kentaroi/eleventy-sass) calls internally. For details, please read [the Sass documentation](https://sass-lang.com/documentation/js-api/modules#StringOptions).

You can use `includes` key instead of `loadPaths` key in this `sass` options, for convenience. The value for `includes` key should be of type Array or String and relative to the `input` directory.

For example, when your `input` directory is the project root,
```javascript
// the following code:
eleventyConfig.addPlugin(eleventySass, {
  sass: {
    includes: "_includes/stylesheets"
  }
});

// is equivalent to:
eleventyConfig.addPlugin(eleventySass, {
  sass: {
    loadPaths: ["_includes/stylesheets"]
  }
});
```

But, when your `input` directory is `src`,
```javascript
// the following code:
eleventyConfig.addPlugin(eleventySass, {
  sass: {
    includes: "_includes/stylesheets"
  }
});

// is equivalent to:
eleventyConfig.addPlugin(eleventySass, {
  sass: {
    loadPaths: ["src/_includes/stylesheets"]
  }
});
```

The value of `includes` should be of type Array or String.
```javascript
// The following:
eleventyConfig.addPlugin(eleventySass, {
  sass: {
    includes: "_includes/stylesheets, _some/other/directory"
  }
});

// is equivalent to:
eleventyConfig.addPlugin(eleventySass, {
  sass: {
    includes: ["_includes/stylesheets", "_some/other/directory"]
  }
});
```

### Default options
If you do not specify any options, [eleventy-sass](https://github.com/kentaroi/eleventy-sass) uses the default options. By default, [eleventy-sass](https://github.com/kentaroi/eleventy-sass) checks `ELEVENTY_ENV` environment/shell variable, and if it is "production", "prod", "p", or something like that or if there is no `ELEVENTY_ENV` variable, it considers the environment `production`.

```javascript
// If the environment is production, the default options are:
{
  outputFileExtension: "css",
  compile: async function(inputContent, inputPath) { /* snip */ },
  compileOptions: {
    cache: true,
    getCacheKey: function(contents, inputPath) { /* snip */ },
  },
  getData: async function(inputPath) {
    return { eleventyComputed: { layout: false } };
  },
  sass: {
    loadPaths: [/* The includes directory of your Eleventy project */],
    style: "compressed",
    sourceMap: false,
    sourceMapIncludeSources: true
  },
  defaultEleventyEnv: "production"
}

// If not, the default options are:
{
  outputFileExtension: "css",
  compile: async function(inputContent, inputPath) { /* snip */ },
  compileOptions: {
    cache: true,
    getCacheKey: function(contents, inputPath) { /* snip */ },
  },
  getData: async function(inputPath) {
    return { eleventyComputed: { layout: false } };
  },
  sass: {
    loadPaths: [/* The includes directory of your Eleventy project */],
    style: "expanded",
    sourceMap: true,
    sourceMapIncludeSources: true
  },
  defaultEleventyEnv: "production"
}
```

For example, if you add the following line in your `.bash_profile`, `.bashrc`, `.zshrc`, etc. on your local PC,
```bash
export ELEVENTY_ENV=development
```
the environment is `development` locally, and you have a `production` environment, for example, in [GitHub Actions](https://github.com/features/actions) by default, because [eleventy-sass](https://github.com/kentaroi/eleventy-sass) regards no `ELEVENTY_ENV` variable as `production`. (This default behavior can also be modified by setting the value for `defaultEleventyEnv` key in the options for `addPlugin()`.)

And, of course, you can specify an environment from your shell like this:
```bash
ELEVENTY_ENV=production npx @11ty/eleventy --serve
```

### Some recipes
#### How to change output directory for CSS files compiled from Sass/SCSS
For example, you have Sass/SCSS files in `scss` directory in your `input` directory. To output CSS files in `css` directory in your `output` directory, you can write a permalink function like this:

```javascript
const eleventySass = require("eleventy-sass");

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(eleventySass, {
    compileOptions: {
      permalink: function(contents, inputPath) {
        return (data) => {
          return data.page.filePathStem.replace(/^\/scss/, "/css") + ".css";
        };
      }
    }
  });
};
```
For details, please refer to the [compileOptions.permalink](https://www.11ty.dev/docs/languages/custom/#compileoptions.permalink-to-override-permalink-compilation) section in the official documentation.

#### How to minify CSSes
By default, [eleventy-sass](https://github.com/kentaroi/eleventy-sass) minifies CSSes if `ELEVENTY_ENV` is `production`.

To enable minification whatever `ELEVENTY_ENV` is:

```javascript
const eleventySass = require("eleventy-sass");

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(eleventySass, {
    sass: {
      style: "compressed"
    }
  });
};
```

To disable minification:

```javascript
const eleventySass = require("eleventy-sass");

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(eleventySass, {
    sass: {
      style: "expanded"
    }
  });
};
```

#### How to add source maps
By default, [eleventy-sass](https://github.com/kentaroi/eleventy-sass) adds source maps if `ELEVENTY_ENV` is not `production`.

To add source maps whatever `ELEVENTY_ENV` is:

```javascript
const eleventySass = require("eleventy-sass");

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(eleventySass, {
    sass: {
      sourceMap: true
    }
  });
};
```

## How [eleventy-sass](https://github.com/kentaroi/eleventy-sass) handles Sass/SCSS files
[Eleventy](https://www.11ty.dev/) watches files.

When [Eleventy](https://www.11ty.dev/) detects a file update, it emits an `eleventy.beforeWatch` event, and [eleventy-sass](https://github.com/kentaroi/eleventy-sass) will receive the event and check if the file is a Sass/SCSS file.

If the file is a Sass/SCSS file, [eleventy-sass](https://github.com/kentaroi/eleventy-sass) changes the cache keys for the updated file and its dependant Sass/SCSS files, which invalidates the cached CSSes, so that [Eleventy](https://www.11ty.dev/) will compile all of the files affected.

When compiling Sass/SCSS files, [Eleventy](https://www.11ty.dev/) calls `compile` function of the [elventy-sass](https://github.com/kentaroi/eleventy-sass).

The `compile` function of [elventy-sass](https://github.com/kentaroi/eleventy-sass) will compile Sass/SCSS files if they don't start with "\_".

The actual compilation is done by [sass](https://www.npmjs.com/package/sass), which is [dart-sass](https://github.com/sass/dart-sass).

By using the result of the compilation from [dart-sass](https://github.com/sass/dart-sass), [eleventy-sass](https://github.com/kentaroi/eleventy-sass) not only returns the compiled CSS, but also, if necessary, writes its source map file and updates the dependency map, which [eleventy-sass](https://github.com/kentaroi/eleventy-sass) has internally and uses for invalidating cached CSSes of dependant files as described above.
