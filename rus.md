На днях состоялся релиз gulp (*с англ. «глоток»*), достойной альтернативы очень
популярному JavaScript таск-менеджеру GruntJS. Давайте разберемся, что же
отличает их друго от друга, и попытаемся понять, зачем был создан gulp.

Когда дело доходит до JavaScript таск-менеджеров, Grunt — царь. Ну, по крайней 
мере, раньше было так… Ранее, в этом году, команда на [Fractal][1] 
[выразила свое отношение][2] к Grunt, и выступила с идеей взять все великие идеи
и преимущества Grunt, и воссоздать их. Они назвали свой проект [gulp][3](глоток).
И, хотя, он решает те же проблемы, что и Grunt, под капотом у них очень большие
различия. И так…


## Что такое «таск-менеджер»

Некоторые из вас могут быть знакомы с Grunt, некоторые — нет. Для несведущих,
давайте немного проясним, что же такое JavaScript таск-менеджер.

Таск-менеджер — это небольшое приложение, которое используется для автоматизации
занудных, отнимающих время задач, которые приходится делать в процессе разработки
проекта. Такие задачи включают в себя запуск тестов, конкатенацию файлов, 
минификацию, препроцессинг CSS. Просто создав таск-файл, вы можете
проинструктировать таск-менеджер, как выполнить практически любую задачу. После
этого вы можете заняться делом. Это очень простая идея, которая позволяет
сохранить очень много времени, и помогает держать фокус на разработке.


## Различия

Теперь, когда мы находимся на одном уровне, вы можете спросить: «Чем
gulp отличается от Grunt, и почему это меня должно меня беспокоить?»


### Потоки

Gulp *потоковая* система сборки. Здесь я хотел бы углубиться объяснение устройства
потоков, но [вот замечательный источник][4], где, если вам интересно, вы можете
выяснить что такое потоки.

Если не усложнять, потоки дают вам больше контроля над происходящим и избавляют
вас от промежуточных папок и файлов. Вы передаете файл в gulp, а затем сохраняете
результат. Это очень просто.


### Плагины

Когда дело доходит до расширения функциональности, gulp верит, что каждый 
плагин должен выполнять только *одно простое действие*. Gulp же просто
соединяет и организует их. Здесь нет общих плагинов или плагинов, конфликтующих
с другими плагинами или ядром.

### Code Not Config

Больше всего лично мне нравится, что gulpfile — это код, а не *конфиг*. С тех
пор как gulp следует спецификации CommonJS, если вы знакомы с Node, вы будете
чувствовать себя как дома. Все это выглядит аккуратно и читаемо, и, потому как
все структурированно знакомым способом, то написать gulpfile не составит никаких
затруднений.


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