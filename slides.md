class: slide-home
![Grunt](./img/grunt-logo.svg)
# The Good Parts


---
# Experience Grunt

![Canal+](./img/logo-canal.png)
![ING Direct](./img/logo-ing.gif)
![Meetic](./img/logo-meetic.jpg)

Limites atteintes, lessons apprises.

---
# Refonte appli mobile @Meetic

![Meetic](./img/webapp-mobile.jpg)

- 6 devs front, 2 backs
- Intégration continue
- Jenkins, Gitlab, Bamboo, Hipchat

---
# Pas Glop

![Pas glop pas glop](./img/pas-glop-pas-glop.png)

- Configuration over configuration
- Gruntfile énorme
- Trop de plugins
- Modification sur disque

---
## Charger les plugins automatiquement

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
require('load-grunt-tasks')(grunt);
```

---
## Charger la config automatiquement

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
### Small Gruntfile

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
background-image: url(./img/configuration-over-configuration.jpg)
class: full-page-title

# Configuration over Configuration

---
## Recherche étendue

```javascript
options: {
  files: {
    'dist/css/main.css': 'app/css/main.css',
    'dist/css/print.css': 'app/css/print.css',
    'dist/css/misc.css': 'app/css/misc.css'
  }
}
```

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
## Nommer vos target avec des noms explicites

`grunt rsync:devJsCoreToDist`

```javascript
[...]
devJsCoreToDist: {...},
devJsComponentsToDist: {...},
prodCssDependenciesToTmp: {...},
prodTemplatesAppToTmp: {...},
[...]
```

- __environment__*Type*Action
- __dev__*JsCore*ToDist
- __prod__*CssDependencies*ToTmp

---
## Utiliser les valeurs par défaut

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
# Make it faster


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
## Rsync

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
## Local build



## Build et compile

Build construit tout
example typique de taches BUILD
compression images -> révisions
sass -> css -> concaténation -> modification assets -> autoprefixer -> minification -> révision
suppression de devcode -> injection html dans templates -> injection variables
de conf / env -> récriture angular -> annotate -> concaténation -> uglification
-> révisions

COMPILE
minimum, pour un débug plus facile.
Sass -> CSS, mais pas de concat ni de minification

## My pick
Meilleurs plugins front.
Pas usemin.
fileblocks pour intégrer fichier générés
concat, etc (dresser la liste)

## Livereload

Watch des fichiers, rejoue les taches, et mets à jour le browser. Gain de temps
énorme. Sur CSS assez malin pour mettre à jour seulement les styles.
Peut faire la même chose sur les tests (si fichier de test change, on rejoue)

syntaxe spéciale pour trouver dans sous dossiers

on déplace dans tmp/src, on convertit dans tmp/output, et on déplace ensuite
dans dist

## Serve apimock

taper sur des versions statiques de l'API pour quand pas développé


# Integration Continue

## Gitlab, Jenkins (et Bamboo)

On travail sur un fork local de l'app. On créé des branches pour chaque
features. On push la branche sur notre fork. On créé une MR de notre branche
vers l'upstream.

Code review de tous, commentaires, une fois que 1 ok, le deuxième ok valide.
Mergé automatiquement dans develop. Styleguide pour les remarques régulières.

## Build incassable local

Checkstyle avant de commit. Check test avant de push. Check merge markeurs.

Test
Tests unitaires sous karma/phantom. Lancé avant de push. Build "incassable".
Test end-to-end "BDD-style", lance un firefox, avec des assertions cucumber,
mais va vérifier dans les bases "de prod" en scrappant que c'est là.

## Badges

Gitlab <-> Jenkins. Si nouvelle MR ou nouveau commit sur une MR, Jenkins tente
de merge la MR sur develop et lance les checks et les tests. Si passe, badge
vert, si foire, badge rouge. Si instable, badge orange.

## Build instable

On mesure plein de choses, metrics créées par grunt, sorties dans des xml
lisibles par jenkins. jshint, jscs (mais toujours ok), complexité cyclomatique,
code coverage, copié-collé.

## Release

BUMPVERSION
équivalent maven
On bump la version dans package.json et autres fichiers de conf, on commit, on
tag, on push sur master. Lancé par jenkins manuellement quand on release

CHANGELOG
Génération de changelog en markdown depuis l'historique git


