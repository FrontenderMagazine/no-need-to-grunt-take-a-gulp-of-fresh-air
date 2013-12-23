Recently, gulp was released to the world as a rival to the very popular
JavaScript task runner, Grunt. Let's take a look at what sets them apart and 
understand why gulp was created.

When it comes to JavaScript task runners, Grunt is king. Well, at least it has
been... Earlier this year, the team at[Fractal][1] [voiced their concerns][2]
with Grunt and came up with a plan to take all the great ideas and benefits that
Grunt introduced and rebuild it. They call their project[gulp][3], and while it
solves the same problems that Grunt does - there is a lot underneath that makes 
them both very different. Let's get started.

## What is a task runner?

Some of you may be familiar with Grunt and some of you may not. For the
uninitiated, let’s do a quick overview of exactly what a JavaScript task runner 
is.

Task runners are small applications that are used to automate many of the time
consuming, boring (but very important) tasks that you have to do while 
developing a project. These include tasks such as running tests, concatenating 
files, minification, and CSS preprocessing. By simply creating a task file, you 
can instruct the task runner to automatically take care of just about any 
development task you can think of as you make changes to your files. It’s a very
simple idea that will save you a lot of time and allow you to stay focused on 
development.

## Differences

So, now that we’re on the same page, you may ask: “How is gulp different
than Grunt and why should I care?
”

### Streaming

Gulp is a *streaming* build system. I wont go into detail about streams in this
article, but[this is a great resource][4] to learn more if you are interested
.

To put it simply, streaming gives you more control over your flow and relieves
you of temporary folders and files. With gulp - you put a file in and you get a 
file out. It’s that simple.

### Plugins

When it comes to extending functionality, it is gulp’s belief that each
plugin should only perform a*single action*. Gulp is simply there to connect
and organize them. There is no shared/conflicting purpose with other plugins or 
core features.

### Code Not Config

My personal favorite improvement is that your gulpfile is code - not *config*.
Since gulp follows the CommonJS spec, if you are familiar with Node then you 
will feel right at home. It is far cleaner and easier to read and because it is 
structured in a familiar way, it's also easier to write.

## Examples

This may not sink in until you actually see some code, so I’ll give you a
couple examples. In the following code snippets we are setting up a gruntfile 
and a gulpfile that will lint, concatenate and minify our project's JavaScript 
files. Then we will set them both up to watch for when those files are changed 
and then run the tasks again.

First we will start with our gruntfile, and then I will show you what those
same tasks would look like in a gulpfile. This will give you a good idea of how 
everything works together and how gulp improves on Grunt's ideas.

#### gruntfile.js

    <br>module.exports = function(grunt) {
      grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        concat: {
          options: {
            separator: ';'
          },
          dist: {
            src: ['src/**/*.js'],
            dest: 'dist/<%= pkg.name %>.js'
          }
        },
        uglify: {
          dist: {
            files: {
              'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
            }
          }
        },
        jshint: {
          files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
          options: {
            globals: {
              jQuery: true,
              console: true,
              module: true,
              document: true
            }
          }
        },
        watch: {
          files: ['<%= jshint.files =>'],
          tasks: ['jshint', 'concat', 'uglify']
        }
      });
    
      // Load Our Plugins
      grunt.loadNpmTasks('grunt-contrib-jshint');
      grunt.loadNpmTasks('grunt-contrib-concat');
      grunt.loadNpmTasks('grunt-contrib-uglify');
      grunt.loadNpmTasks('grunt-contrib-watch');
    
      // Register Default Task
      grunt.registerTask('default', ['jshint', 'concat', 'uglify']);
    
    };
    
    

#### gulpfile.js

    var gulp = require('gulp');
    var jshint = require('gulp-jshint');
    var concat = require('gulp-concat');
    var rename = require('gulp-rename');
    var uglify = require('gulp-uglify');
    
    // Lint JS
    gulp.task('lint', function() {
      gulp.src('./src/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter('default'));
    });
    
    // Concat & Minify JS
    gulp.task('minify', function(){
        gulp.src('./src/*.js')
            .pipe(concat('all.js'))
            .pipe(gulp.dest('./dist'))
            .pipe(rename('all.min.js'))
            .pipe(uglify())
            .pipe(gulp.dest('./dist'));
    });
    
    // Default
    gulp.task('default', function(){
      gulp.run('lint', 'minify');
    
      // Watch JS Files
      gulp.watch("./src/*.js", function(event){
        gulp.run('lint', 'minify');
      });
    });
    

By switching to gulp we have reduced our code from 52 lines to 30. On top of
that you may notice that we required the same number of plugins, but two of them
are different even though we are doing the exact same thing to our code. This 
further illustrates the core difference with plugins that I mentioned above.

With gulp we don't include a watch plugin because *watch* is a core feature -
there is no plugin needed. The functionality you would expect to be included is 
included by default - not by a plugin.

Additionally, with Grunt the renaming of our minified file is handled by the
uglify plugin. One plugin has the responsibility of minfiying the code AND 
renaming it. With gulp, every plugin has a single action - a single 
responsibility. To rename our minified file in gulp, we simply include the gulp-
rename plugin and include it in our minify task.

## Conclusion

Ultimately, this is all up to personal preference. I personally prefer the “
node-like” way of writing my task files with gulp, but I must say that I’ve 
really enjoyed my time with Grunt as well. Knowledge of both is very valuable 
not only to understand task runners and how they work, but to also understand 
the decisions both teams made when developing these tools and why they made 
those decisions. There is a lot to be learned and a lot of time to be saved in 
development. If you'd like to get started, check out[gulp on Github][3].

## Additional Reading

*   [Stream Handbook][5]
*   [gulp Slides][2]
*   [gulp on Github][3]

[comments powered by ][6]

 [1]: http://wearefractal.com "Fractal"
 [2]: http://slid.es/contra/gulp "gulp slideshow on slid.es"
 [3]: https://github.com/wearefractal/gulp "gulp on Github"
 [4]: https://github.com/substack/stream-handbook
 [5]: https://github.com/substack/stream-handbook "Stream Handbook on Github"
 [6]: http://disqus.com