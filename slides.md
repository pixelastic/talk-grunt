.center[
  ![Grunt](./img/grunt-logo.svg)
  # The Good Parts
]
---

# Grunt @Meetic

.center[
  ![Meetic](./img/webapp-mobile.jpg)

  ![Canal+](./img/logo-canal.png)
  ![ING Direct](./img/logo-ing.gif)
]
---

# Sommaire

.big-list[
1. Maintenabilité
2. Lenteur
3. Tips & Tricks
]
---

class: full-page slide-maintain
# Maintenabilité
---

## Charger les plugins

### Avant
```javascript
grunt.loadNpmTasks('grunt-contrib-clean');
grunt.loadNpmTasks('grunt-contrib-concat');
grunt.loadNpmTasks('grunt-contrib-connect');
grunt.loadNpmTasks('grunt-contrib-cssmin');
grunt.loadNpmTasks('grunt-contrib-htmlmin');
grunt.loadNpmTasks('grunt-contrib-imagemin');
grunt.loadNpmTasks('grunt-contrib-jshint');
grunt.loadNpmTasks('grunt-contrib-sass');
grunt.loadNpmTasks('grunt-contrib-uglify');
grunt.loadNpmTasks('grunt-contrib-watch');
```
### Après
```javascript
require('matchdep').filterDev('grunt-*').forEach(grunt.loadNpmTasks);
```
---

## Charger la configuration

### Avant
```javascript
// Gruntfile.js
watch: {
  [...]
},
connect: {
  [...]
},
jshint: {
  [...]
},
clean: {
  [...]
},
autoprefixer: {
  [...]
},
[...]
```
---

### Découpage en plusieurs fichiers

```javascript
var fs = require('fs');
function loadConfig(path) {
  var config = {};
  fs.readdirSync(path).forEach(function(file) {
    config[file.replace(/\.js$/, '')] = require(path + file);
  });
  return config;
}

module.exports = function(grunt) {
  var config = {
    [...]
  }
  grunt.util._.extend(
    config, 
    loadConfig(__dirname + '/tasks/options/')
  );

  require('load-grunt-tasks')(grunt);

  grunt.loadTasks('tasks');
  grunt.config.init(config);
};
```
---

### Après
```shell
$ tree ./tasks/options/
options
├── autoprefixer.js
├── bump.js
├── cssmin.js
[...]
├── rsync.js
├── sass.js
├── uglify.js
├── version.js
└── watch.js
```

```javascript
// ./tasks/options/cssmin.js
module.exports = {
  options: {
    root: 'app',
    noAdvanced: true
  },
  prodMinifyStyleFile: {
    files: {
      '.tmp/css/output/style.css': '.tmp/css/output/style.css'
    }
  }
};
```
---

## Compact & Files Array syntax

### Compact
```javascript
options: {
  files: {
    'dist/css/main.css': 'app/css/main.css',
    'dist/css/print.css': 'app/css/print.css',
    'dist/css/misc.css': 'app/css/misc.css'
  }
}
```
### Files Array
```javascript
options: {
  files: [{
    expand: true,
    cwd: 'app/css/',
    src: '*.css',
    dest: 'dist/css/'
  }]
}
```
---

## Pipeline

### `./app` > `./tmp` > `./dist`


.big-list[
- `./app`: Sources
- `./tmp`: Transformations
- `./dist`: Final
]
---

## Targets explicites & Conventions

<code>grunt task:<strong>target</strong></code>

```javascript
[...]
devCssDependenciesToDist: {...},
devHtmlAppToDist: {...},
devTemplatesAppToDist: {...},
devJsBowerTopToDist: {...},
prodJsCoreToTmp: {...},
prodJsComponentsToTmp: {...},
prodJsScriptFileToDist: {...},
[...]
```

.big-list[
- __environment__*Type*Action
- __dev__*JsCore*ToDist
- __prod__*CssDependencies*ToTmp
]
---

## Valeurs par défaut

```javascript
options: {
  recursive: true,
  args: [
    '--archive',
    '--prune-empty-dirs'
  ]
},
devTemplatesAppToDist: {
  options: {
    src: 'app/',
    dest: 'dist/',
    args: [
      '--archive',
      '--recursive',
      '--delete',
      '--prune-empty-dirs',
      '--exclude=/*.html',
      '--include=*/',
      '--include=*.html',
      '--exclude=*'
    ]
  }
}
```
---

class: full-page slide-slow
# Lenteur
---

## Build, Measure, Learn

`npm install time-grunt`

```shell
$ grunt build
[...]
Execution Time (2015-02-24 17:07:22 UTC)
imagemin:prodMinifyTmpFiles      317ms  ▇ 1%
sass:prodAppToTmp                962ms  ▇▇▇▇ 4%
ngAnnotate:prodAnnotateTmpFiles   1.2s  ▇▇▇▇▇ 5%
concat:prodJsBottomTmpFile       11.5s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 53%
uglify:prodMinifyFile             5.9s  ▇▇▇▇▇▇▇▇▇▇▇▇ 27%
Total 21.7s
```
---

## grunt newer

`npm install grunt-newer`

```javascript
$ grunt jshint:dev
jshint:dev     2.3s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 98%
Total 2.4s
```
```javascript
$ grunt newer:jshint:dev
newer:jshint:dev  418ms  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 92%
Total 453ms
```
---

## grunt rsync

`npm install grunt-rsync`

```javascript
$ grunt copy
[...]
copy  119ms  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 65%
Total 182ms
```

```javascript
$ grunt rsync
[...]
rsync 66ms  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 61%
Total 108ms
```

```javascript
src: 'app/components/',
dest: 'dist/js/components',
args: [
  '--archive',
  '--delete',
  '--prune-empty-dirs',
  '--exclude=*_test.js',
  '--include=*/',
  '--include=*.js',
  '--exclude=*'
]
```
---
## Build faster

      | Compile | Build
------|---------|-------
Sass | ✔ | ✔
Auto-prefixeur | ✔ | ✔
Concaténation | ✘ | ✔
Minification | ✘ | ✔
File versionning | ✘ | ✔
---

class: full-page slide-plugins
# Plugins
---
## <strike>grunt usemin</strike>

.big-list[
- Concaténation
  - `grunt-contrib-concat`
- Compression
  - `grunt-contrib-cssmin`
  - `grunt-contrib-htmlmin`
  - `grunt-contrib-uglify`
  - `grunt-contrib-imagemin`
- Cache
  - `grunt-filerev`
- Injection
  - `grunt-devcode`
  - `grunt-file-blocks`
]

---
## Livereload

`npm install grunt-contrib-watch`

```javascript
watch: {
  files: [
    'app/core/**/*.scss'
  ],
  tasks: [
    'sass:devAppToTmp',
    'newer:autoprefixer:devTmpToDist'
  ]
}
```
.big-list[
- Watch fichiers sur disque
- Rejoue tâches
- Recharge navigateur
- Recharge tests
]
---

## Apimock

`grunt serve:dev --apimock`

```shell
./apimock
├── accounts
│   ├── me.GET.json
│   ├── me.OPTIONS.json
│   └── validations.POST.json
├── accounts.POST.json
└── members
    ├── 50397963.GET.json
    ├── 530176615.GET.json
    ├── 590042344.OPTIONS.json
    ├── 590042344.PATCH.json
    └── 660624580.GET.json
```
.big-list[
- Option custom
- Mock l'API
- Retourne fichiers statiques dans `./apimock`
]

---
## Build incassable

.big-list[
- `pre-commit`
  - `grunt jshint`
  - `grunt jscs`
  - `grunt merge-conflict`
- `pre-push`
  - `grunt test `
- Manuel
  - Cucumber + Firefox + Scrapping
- Jenkins
  - `grunt test:coverage`
  - `grunt jscpd`
  - `grunt complexity-report`
]
---

class: full-page slide-questions
# Questions ?

---
# Sources
- [Slow](https://www.flickr.com/photos/zivkovic/5850008238/in/photolist-9UWPD1-c1B9xf-9KjprJ-oeayt6-5wBH6k-LhX2t-cLYPbJ-DsWKL-8iDc53-8rwSDR-5p3vQ7-8oKLsw-HT3yu-p8Bsj1-hgJz2H-6Q7xE8-7VB5sf-vmhA6-3gW3DE-6pdSx2-d2CnFA-cuvg83-oEH5dm-fKtXLR-6s6Q8E-dv26Jo-9suTuN-ciofxq-3gRFNH-6mp5NP-8r4t5s-4Kc33f-8VYhLT-8RfWxc-8jD1oY-6Awwkj-e4Lxa-bEPjxE-8bwd4m-7XMF3J-8jyxkN-5wT9i5-3gJHim-7Gf8M-doN72m-oB1m5G-5iih9A-6sfQbk-bNi5pP-e8bpW5)
  by Zdenko Zivkovic
