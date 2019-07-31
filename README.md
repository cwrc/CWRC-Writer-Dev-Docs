![Picture](http://cwrc.ca/logos/CWRC_logos_2016_versions/CWRCLogo-Horz-FullColour.png)

CWRC-Writer-Dev-Docs
====================

Describes the overall organization of the CWRC-Writer code, how we use NPM, and one approach to development for CWRC-Writer coding.

1. [Overview](#overview)
1. [Editor](#editor)
1. [Server](#server)
1. [How to Develop with CWRC packages](#how-to-develop-with-cwrc-packages)
1. [How to Create a New CWRC package](#how-to-create-a-new-cwrc-package)
1. [Sponsors](#Sponsors)

## Overview

The CWRC-Writer is an in-browser WYSIWYG XML text editor that also enables stand-off RDF annotation to mark references to named entities in the text, and to make textual annotations. There are two main parts to a CWRC-Writer installation that run more or less independently: the CWRC-Writer editor itself that runs in the web browser, and the complementary backend services that run on a server and provide document storage, XML validation, and entity lookup. The best example of how to put together a full CWRC-Writer installation is our sandbox version, which is useable at [https://cwrc-writer.cwrc.ca/](https://cwrc-writer.cwrc.ca/) and whose code is available at [CWRC-GitWriter](https://github.com/cwrc/CWRC-GitWriter).

![High Level Overview](/images/cwrc-gitwriter-overview.svg)

## Editor

An instance of the CWRC-Writer web editor is built around the [CWRC-WriterBase](https://github.com/cwrc/CWRC-WriterBase), a heavily customized version of the [TinyMCE](https://www.tiny.cloud/) text editor.  The CWRC-WriterBase is used in conjunction with a few other CWRC modules:

#### Storage Dialogs

The CWRC-Writer editor ([CWRC-WriterBase](https://github.com/cwrc/CWRC-WriterBase) running in the web browser) can be used with different backends for storage.  Your storage choice will likely require specific interactions with the end user, and so we've isolated the dialogs for loading and saving documents, allowing you to substitute your own dialogs.  The default dialogs for CWRC-Writer are in the [cwrc-git-dialogs](https://github.com/cwrc/cwrc-git-dialogs) and handle lists, loads, saves, and authentication to the default [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer), which in turn makes calls to GitHub itself.

#### Entity Lookup

The editor also allows users to lookup references to named entities (e.g. people, places, organizations). The default entity lookup package is [CWRC-PublicEntityDialogs](https://github.com/cwrc/CWRC-PublicEntityDialogs) which looks up named entities and returns unique URIs for the selected entity. This package uses a modular approach so that lookup sources can be easily added. Some of the available sources are [VIAF](https://viaf.org), [Wikidata](https://www.wikidata.org), and [GeoNames](http://www.geonames.org/). You can find the modules for these sources here: [viaf-entity-lookup](https://github.com/cwrc/viaf-entity-lookup), [wikidata-entity-lookup](https://github.com/cwrc/wikidata-entity-lookup), [geonames-entity-lookup](https://github.com/cwrc/geonames-entity-lookup).

#### npm packages and Browserify

The [CWRC-WriterBase](https://github.com/cwrc/CWRC-WriterBase), the entity lookups, and a few other components are organized as [npm](https://www.npmjs.com) packages (and published to the [public npm repository](https://www.npmjs.com/search?q=cwrc) for use by anyone).  
The npm packages that contribute to the browser part of the CWRC-Writer are combined together using the node.js module loading system and with [Browserify](https://browserify.org).  We write code like we would for a node.js application (that would normally run on the server), using the node.js `require` statements to import packages.  Browserify bundles all the code together (both our packages and all other packages we've included like jquery, bootstrap, and so on) into a single javascript file that can then be brought into the index.html page of our web app:

```<script type="text/javascript" src="js/app.js"></script>```

The best example of how the npm packages are combined and browserified is in the [CWRC-GitWriter](https://github.com/cwrc/CWRC-GitWriter) repository.  Specifically take a look at the [app.js](https://github.com/cwrc/CWRC-GitWriter/blob/master/src/js/app.js) file, the so-called "entry point" into the application, which is where Browserify starts and then "crawls" the dependency tree to pull in all dependencies (as defined by `require` statements).  The [app.js](https://github.com/cwrc/CWRC-GitWriter/blob/master/src/js/app.js) file also shows how the `require` statements are used to combine the CWRC-Writer javascript packages together, by passing them into the [CWRC-WriterBase](https://github.com/cwrc/CWRC-WriterBase).

The CWRC npm packages that are used in the browser:

###### CWRC-WriterBase
The base class for the CWRC-Writer.

* on npm: [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base)
* on GitHub: [CWRC-WriterBase](https://github.com/cwrc/CWRC-WriterBase)

###### cwrc-git-dialogs
Dialogs for loading, saving, and authenticating.

* on npm: [cwrc-git-dialogs](https://www.npmjs.com/package/cwrc-git-dialogs)
* on GitHub: [cwrc-git-dialogs](https://github.com/cwrc/cwrc-git-dialogs)

###### CWRC-PublicEntityDialogs
Dialogs for looking up named entities (e.g. people, places, organizations, and publications) in public authority files.

* on npm: [cwrc-public-entity-dialogs](https://www.npmjs.com/package/cwrc-public-entity-dialogs)
* on GitHub: [CWRC-PublicEntityDialogs](https://github.com/cwrc/CWRC-PublicEntityDialogs)

## Server

The server provides services for storing documents, XML validation, and entity lookup proxying.  The server can be implemented however one would like, and can be a more general server, or set of servers, used by other applications besides the CWRC-Writer.

#### Entity Lookup Proxy

Certain lookup services don't provide HTTPS endpoints for querying. Since CWRC-GitWriter is located at an HTTPS URL, a proxy service is required in order to avoid [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) errors. The CWRC-GitWriter uses a CWRC-hosted proxy for Getty and DBPedia lookups.

#### XML Validation

The [default XML validator](https://validator.services.cwrc.ca/validator/) is a public service supplied by CWRC, and the call to it is handled by the CWRC-WriterBase.  If you are implementing your own validator, you can specify the URL for it using the `validationUrl` property in the CWRC-WriterBase validation module config. However, you will also need ensure that your service returns the same XML that's expected by the [validation module](https://github.com/cwrc/CWRC-WriterBase/blob/master/src/js/layout/modules/validation/validation.js), or re-implement it yourself.

#### Document Storage

The default backend server we use for storage is the [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer), an Express.js server that in turn uses GitHub for storage.  We don't make direct calls to GitHub from the browser because we use GitHub's OAuth, which requires that we run a server to receive the OAuth callback from GitHub.

The [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer) uses one other CWRC npm package:

###### CWRC-Git
A wrapper for [@octokit/rest](https://www.npmjs.com/package/@octokit/rest) that takes care of searching, loading and saving GitHub documents.

* on npm: [cwrcgit](https://www.npmjs.com/package/cwrcgit)
* on GitHub: [CWRC-Git](https://github.com/cwrc/CWRC-Git)

Typical development on the server part of the CWRC-Writer will therefore require changes to both the [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer) (probably to change [routes](https://expressjs.com/en/guide/routing.html)) and to [CWRC-Git](https://github.com/cwrc/CWRC-Git).

## How to Develop with CWRC Packages

There are two types of package: those that interact with the DOM and are intended only to run in a web browser, and those that don't interact with the DOM and might run either in the web browser or on the server in node.js. Both types of package are fundamentally the same, but we test the web packages differently. The following steps apply to both types of package, with the different testing steps for the web packages marked accordingly.

#### Basic Development Setup

* Fork or clone (depending on your role in the project) the relevant repository to your local machine.

* `npm install` to install the node.js dependencies.

* If the repository has a `config.js` file with passwords or tokens, you'll have to set these values appropriately in your cloned repo.  Tell git to ignore the file completely (so that you don't inadvertently commit the file and push it to the public repo thereby exposing the passwords) using `git update-index --skip-worktree config.js`

	Note that `.gitignore` doesn't ignore files that have been comitted, and the `config.js` file likely has been commited to allow the Travis build tool to run, albeit with dummy values.

#### Test and Code

* write a test (or two) for your new feature 

* `npm run test` to run the tests

* write some code to satisfy new test

See the test files for each CWRC package to get an idea of how the actual tests are written.

Depending on the type (and age) of the package, different testing libraries are used. Older packages use [tape](https://www.npmjs.com/package/tape) as a testing framework, whereas newer packages use [mocha](https://www.npmjs.com/package/mocha) and [chai](https://www.npmjs.com/package/chai).

#### Commit to GitHub / Build in Travis / Release to npm

We use [commitizen](https://www.npmjs.com/package/commitizen), [Travis](https://travis-ci.org), [semantic-release](https://www.npmjs.com/package/semantic-release), [Istanbul](https://www.npmjs.com/package/istanbul) / [nyc](https://www.npmjs.com/package/nyc), and [codecov.io](https://codecov.io) for our commits, builds, npm releases, code coverage, and code coverage reporting.  This should all be mostly setup when you clone the repository, but you may have to re-run some portions on your own machine.  For a full description of the setup see [How to Create a New CWRC package](#how-to-create-a-new-cwrc-package) below.

##### Commits

To submit a commit:
1. stage your changes (e.g. `git add -A`)
2. instead of using git's commit command, use `npm run cm`, which uses commitizen to create commits that adhere to the semantic-release conventions

The [husky](https://www.npmjs.com/package/husky) package is used to add two pre-commit git hooks that will check that all tests pass and that code coverage (as calculated by Istanbul) is above the specified minimum, before allowing a commit to proceed.

After the commit has succeeded then `git push` it all up to GitHub, which will trigger the Travis build. The Travis build is also set to confirm that all tests pass and that the code coverage is adequate.  This is set in the `.travis.yml` file like so:

```
script:
  - npm run test
  - npm run check-coverage
```

Of course, if the husky hooks that check tests and code coverage themselves passed, then the Travis check for tests and code coverage should also be fine (but a second check is well worth running).

##### Travis

Results of the Travis build are accessed using the GitHub repo owner's name and the repository name, e.g. https://travis-ci.org/cwrc/CWRC-Git

##### Code Coverage

After the Travis build finishes (successfully), Travis triggers scripts to report coverage to codecov.io and to run semantic release to publish to npm. This is set in the `.travis.yml` file like so:

```
after_success:
  - npm run report-coverage
  - npm run travis-deploy-once "npm run semantic-release"
```

report-coverage publishes the code coverage statistics to codecov.io where the coverage can be viewed: https://codecov.io/gh/cwrc/CWRC-Git/

You can also browse the code coverage reports locally by opening `coverage/lcov-report/index.html` in the project directory.

codecov.io also provides us with the code coverage badge at the top of each repo's README.

##### npm Publishing

When the `npm run semantic-release` script (listed in the Travis after_success section) is triggered, the [semantic-release](https://github.com/semantic-release/semantic-release) package performs the following actions:
 
- updates our version number and publishes a new version to npm
- generates a changelog and tags our GitHub repository with a new release

Version numbers follow the [SemVer](http://semver.org/) spec.

Read more about semantic-release and how it works here: https://github.com/semantic-release/semantic-release#how-does-it-work

## How to Create a New CWRC Package

There are two types of package:
* those that interact with the DOM and are intended only to run in a web browser
* those that don't interact with the DOM and might run either in the web browser or on the server in node.js

Both types of package are fundamentally the same, but we test them differently.  The following steps apply to both types of package, with different testing steps outlined accordingly.

##### Create GitHub repo

From the [CWRC GitHub account](https://github.com/cwrc/), create a new GitHub repository. Choose `Node` for the .gitignore, and `GNU General Public License v2.0` for the license.

Clone the repository to your local machine. From the directory in which you'd like the new directory created:

```git clone git@github.com:cwrc/cwrc-somepackage.git```

##### Initialize Repo as npm Package

Switch into the newly created directory and initialize it as an npm package:

```
cd cwrc-somepackage
npm init
```

npm will prompt you for a few details. Most of these will be specfic to your repo, however:

* for `entry point: (index.js)`, use `src/index.js`
* for `license`, use `GPL-2.0`

##### Install Dependencies

Install whatever npm packages you need.  External npm packages (from the npm registry) can be used as:

- part of your own package (a dependency)
- development tools (e.g. a test runner, or a linting tool) 
- package-independent tools (globals)

When installing an npm package, if it is not a standard dependency then indicate where it should go with either `-D` (development), or `-g` (install globally, usually to use as a command line tool).  

For CWRC, we typically install the following development tools:

```
npm i -D commitizen cz-conventional-changelog husky semantic-release travis-deploy-once codecov.io nyc mocha chai watch faucet
```

and for packages that are to be run on the browser also install:

```
npm i -D babel-core babel-plugin-istanbul babel-preset-env babelify browserify browserify-istanbul watchify concat-stream
```

If the package makes http requests then you'll probably want to mock those calls to keep tests fast.  We've used [nock](https://www.npmjs.com/package/nock) for node.js:

```
npm i -D nock
```

and [sinon](https://www.npmjs.com/package/sinon) for mocking in the browser:

```
npm i -D sinon
```

You'd also install whatever packages will be used by your new package (either to run on the server in Express.js or to be bundled up by browserify into the bundle that is sent down to the browser) but saving them as standard dependencies like so (substitute whatever packages you'll use, but you can install them anytime):

```
npm i jquery bootstrap  
```

And finally install semantic-release-cli as a global, so it can be run from the command line:

```
npm i -g semantic-release-cli 
```

##### Configure npm Settings

Now make sure you've got npm configured to publish to the npm registry:

```
npm set init.author.name "James Chartrand"
npm set init.author.email "jc.chartrand@gmail.com"
npm set init.author.url "http://openskysolutions.ca"
npm login  (answer prompts approriately)
```

##### Run semantic-release-cli

```semantic-release-cli setup```

which will ask you a series of questions, which at the time of writing were:

```
semantic-release-cli setup
? What is your npm registry? https://registry.npmjs.org/
? What is your npm username? jchartrand
? What is your npm password? *******
? What is your GitHub username? jchartrand
? What is your GitHub password? ********
? What CI are you using? Travis CI
```

semantic-release will:

- create `.travis.yml` (which tells Travis what npm scripts to run as part of the build, which versions of node.js to use, etc.)
- login to Travis and set GitHub and npm tokens that allow semantic-release to later tag a GitHub release, and to publish to npm.

Modify the `.travis.yml` that semantic-release-cli created so that `script` and `after_success` look like:

```
script:
  - npm run test
  - npm run check-coverage
after_success:
  - npm run report-coverage
  - npm run travis-deploy-once "npm run semantic-release"
```

The entries in `script` are run first.  If they both pass, then the `after_success` scripts are run.  Report-coverage sends our coverage information to codecov.io.  The [semantic-release](https://github.com/semantic-release/semantic-release) script follows the [SemVer](http://semver.org/) spec, and:
 
- updates our version number in package.json, following the SemVer spec.
- publishes a new version to NPM
- generates a changelog and tags our github repository with a new release

Read more about semantic-release and how it works here: https://github.com/semantic-release/semantic-release#how-does-it-work

Everything that semantic-release-cli does is described here:  https://github.com/semantic-release/cli#what-it-does

Note the difference between semantic-release-cli and semantic-release.  semantic-release-cli is run once on the command line to configure Travis, the npm user, etc..  semantic-release is run everytime a build is invoked on Travis.

##### Configure commitizen

Add a script to package.json `scripts` property:

```
"cm": "git-cz",
```

 Add a commitizen property to the package.json `config` property:

```
"config": {
  "commitizen": {
    "path": "node_modules/cz-conventional-changelog"
  }
}
```

[cz-conventional-changelog](https://github.com/commitizen/cz-conventional-changelog) tells commitizen to structure our commits according to [AngularJS's commit message convention](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#commits), also called the [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog).

##### Configure Husky

When installed, [Husky](https://www.npmjs.com/package/husky) overwrites certain [git hooks](https://git-scm.com/docs/githooks) in the .git/hooks directory to trigger prenamed npm scripts.  We use the `precommit` hook. Add a precommit script to `scripts`:

```
"scripts": {
  "precommit": "npm run test && npm run check-coverage"
}
```

[Husky](https://www.npmjs.com/package/husky) triggers this script whenever a commit is made to GitHub.  As you can see, it will run our tests and verify our test coverage.

##### Add Test and Coverage Scripts

Depending on the type (and age) of the package, different testing libraries are used. Older packages use [tape](https://www.npmjs.com/package/tape) as a testing framework, whereas newer packages use [mocha](https://www.npmjs.com/package/mocha) and [chai](https://www.npmjs.com/package/chai). All packages use [nyc](https://www.npmjs.com/package/nyc) to report coverage.

Add the following to the package.json `scripts` property.

###### non-DOM 

```
"test": "nyc mocha",
"check-coverage": "nyc check-coverage --statements 0 --branches 0 --functions 0 --lines 0",
"report-coverage": "nyc report --reporter=text-lcov > coverage.lcov && codecov"
```

###### DOM

```
"test": "npm run test:electron && npm generate-coverage",
"test:browser": "browserify -t browserify-istanbul test/browser.js | browser-run -p 2222 --static . | node test/extract-coverage.js | faucet",
"test:electron": "browserify -t browserify-istanbul test/browser.js | browser-run --static . | node test/extract-coverage.js | faucet",
"check-coverage": "nyc check-coverage --statements 0 --branches 0 --functions 0 --lines 0",
"report-coverage": "nyc report --reporter=text-lcov > coverage.lcov && codecov"
```

NB: you need to add the `extract-coverage.js` file.

For a complete explanation of how we test in the browser and generate test coverage statistics (including extract-coverage.js), see [DOM Testing](https://github.com/cwrc/CWRC-Writer-Dev-Docs/blob/master/DOM_TESTING.md).

##### Setup Browser Development

If the package is intended to run in the web browser, then we'd like to build the code while developing to see the effect of changes. So, we add an npm script to browserify the code and thereby allow manually testing it directly in a web browser: 

```
"test:browserify": "browserify test/manual.js -o build/test.js --debug -t [ babelify --presets [ es2015 ] ]",
```

Couple this with a watch (using [watchify](https://www.npmjs.com/package/watchify), which is basically browserify with a watch), and it becomes quicker and easier to makes changes to the source and see the result immediately in the browser:

```
 "test:watch": "watchify test/manual.js -o build/test.js --debug --verbose -t [ babelify --presets [ es2015 ] ]",
```

The `build/test.js` file can now be linked into an HTML file to allow us to play with the running code in a browser.

##### Source

Create a `src` folder.  The "entry point" into your new package is `src/index.js` (as specified during `npm init` and in the `package.json` "main" entry).  This is where your code goes.  You can of course split your code out into other files that are required by `index.js`, but ideally the package is small enough that the source fits comfortably within the single `index.js` file.  If it gets unwieldy, consider splitting out a new package.

##### Commit Changes and Push to GitHub

The package has now been set up. Commit the changes to GitHub, but use the previously added script:

```
npm run cm
```

instead of the usual:

```
git commit
```

After successfully commiting:

```
git push
```

it all up to GitHub which will trigger Travis and hence run the tests and coverage checks again, and then run semantic release as described earlier.


## Sponsors
Funding for generalization of CWRC-Writer has been provided by [CANARIE](https://www.canarie.ca/?referral=home) under a Research Software Program grant in 2015.
