![Picture](http://cwrc.ca/logos/CWRC_logos_2016_versions/CWRCLogo-Horz-FullColour.png)

CWRC-Writer-Dev-Docs
====================

Describes the overall organization of the CWRC-Writer code, how we use NPM, and one approach to development for CWRC-Writer coding.

1. [Overview](#overview)
1. [Editor](#editor)
1. [Server](#server)
1. [How to Develop with CWRC packages](#how-to-develop-with-cwrc-packages)
1. [How to Create a CWRC package](#how-to-create-a-cwrc-package)

## Overview

The CWRC-Writer is an in-browser WYSIWYG XML text editor that also enables standoff RDF annotation to mark references to named entities in the text, and to make textual annotations.  There are two main parts to a CWRC-Writer installation that run more or less independently:  the CWRC-Writer editor itself that runs in the web browser, and the complementary backend services that run on a server and provide document storage, XML validation, and entity lookup.  The best example of how to put together a full CWRC-Writer installation is our sandbox version, which is running here: [http://208.75.74.217](http://208.75.74.217) and whose code is available here:  [CWRC-GitWriter](https://github.com/cwrc/CWRC-GitWriter).

## Editor

An instance of the CWRC-Writer web editor is built around the [CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base), a heavily customized version of the [TinyMCE](https://www.tinymce.com) web editor.  The CWRC-WriterBase is used in conjunction with a few other CWRC modules:

#### Storage Dialogs

The CWRC-Writer editor ([CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base) running in the web browser) can be used with different backends for storage.  Your storage choice will likely require specific interactions with the end user, and so we've isolated the dialogs for loading and saving documents, allowing you to substitute your own dialogs.  The default dialogs for CWRC are in the [cwrc-git-dialogs](https://www.npmjs.com/package/cwrc-git-dialogs) and handle lists, loads, saves, and authentication to the default [CWRC-GitServer](https://www.npmjs.com/package/cwrc-git-server), which in turn makes calls to GitHub itself.

#### Delegator

Other backend services are called through a javascript object that we've called the 'Delegator' (because the editor 'delegates' server side requests to it).  The delegator currently handles requests to validate XML documents.  The delegator essentially forwards requests on to the server to carry out the actual request.  NOTE:  we will likely soon rename the delegator as Validator since that's all it now does.  At one point it handled calls to all backend services, but now only to the XML Validator.  Moving towards more modular code, we've separated out the original delegator into smaller packages.

#### Entity Lookup

The editor also allows users to lookup references to named entities (people, places, organizations) and so another javascript object, much like the delegator, is also packaged in with the [CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base), and handles entity lookup. The default entity lookup package is [cwrc-public-entity-dialogs](https://www.npmjs.com/package/cwrc-public-entity-dialogs) which looks up named entities in [VIAF](https://viaf.org) and returns unique URIs for the selected entity.

#### NPM packages and browserify

The [CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base), the delegator, the entity lookups, and a few other components are organized as [NPM](https://www.npmjs.com) packages (and published to the [public npm repository](https://www.npmjs.com/search?q=cwrc) for use by anyone).  

The NPM packages that contribute to the browser part of the CWRC-Writer are combined together using the node.js module loading system and with [Browserify](https://browserify.org).  We write code like we would for a node.js application (that would normally run on the server), using the node.js 'require' statements to import packages.  Browserify bundles all the code together (both our packages and all other packages we've included like jquery, bootstrap, and so on) into a single javascript file that can then be brought into the index.html page of our web app:

```<script type="text/javascript" src="js/app.js"></script>```

The best example of how the NPM packages are combined and browserified is in the [CWRC-GitWriter](https://github.com/cwrc/CWRC-GitWriter) repository.  Specifically take a look at the [app.js](https://github.com/cwrc/CWRC-GitWriter/blob/master/src/js/app.js) file, the so-called 'entry point' into the application, which is where Browserify starts and then 'crawls' the dependency tree to pull in all dependencies (as defined by 'require' statements).  The [app.js](https://github.com/cwrc/CWRC-GitWriter/blob/master/src/js/app.js) file also shows how the 'require' statements are used to combine the CWRC-Writer javascript packages together, by passing them into the [CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base).

The CWRC NPM packages that are used in the browser:

###### CWRC-WriterBase
The base class for the cwrc-writer.

* in NPM: [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base)
* in GitHub: [CWRC-WriterBase](https://github.com/cwrc/CWRC-WriterBase)

###### cwrc-git-dialogs
Used by the [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) to make calls to [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer).

* in NPM: [cwrc-git-dialogs](https://www.npmjs.com/package/cwrc-git-dialogs)
* in GitHub: [cwrc-git-dialogs](https://github.com/cwrc/cwrc-git-dialogs)

###### CWRCGitDelegator
Delegator to which [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) delegates server-side calls.  Used by the [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) to make calls to [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer).
NOTE:  THIS PACKAGE IS DEPRECATED AND SHOULD NOT BE USED.  PARTS HAve BEEN INCORPORATED DIRECTLY INTO THE CWRC-WRITER-BASE, and parts have been moved to [cwrc-git-dialogs](https://github.com/cwrc/cwrc-git-dialogs).

* in NPM: [cwrc-git-delegator](https://www.npmjs.com/package/cwrc-git-delegator)
* in GitHub: [CWRC-GitDelegator](https://github.com/cwrc/CWRC-GitDelegator)

###### CWRC-GitServerClient
Client for calls to the [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer) from the [cwrc-git-delegator](https://www.npmjs.com/package/cwrc-git-delegator).

* in NPM: [cwrc-git-server-client](https://www.npmjs.com/package/cwrc-git-server-client)
* in GitHub: [CWRC-GitServerClient](https://github.com/cwrc/CWRC-GitServerClient)

###### CWRCPublicEntityDialogs
Dialogs for the [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) that lookup people, places, organizations, and publications in public authority files.

* in NPM: [cwrc-public-entity-dialogs](https://www.npmjs.com/package/cwrc-public-entity-dialogs)
* in GitHub: [CWRC-PublicEntityDialogs](https://github.com/cwrc/CWRC-PublicEntityDialogs)

###### CWRCWriterLayout
Components for customizing the CWRC-Writer layout.  This package is used by the layout-config.js file in an instance of the CWRC-Writer.  See [CWRC-GitWriter](https://github.com/cwrc/CWRC-GitWriter) for an example.  
NOTE:  THIS PACKAGE IS DEPRECATED AND SHOULD NOT BE USED.  THE LAYOUT CODE HAS BEEN INCORPORATED DIRECTLY INTO THE CWRC-WRITER-BASE.

* in NPM: [cwrc-writer-layout](https://www.npmjs.com/package/cwrc-writer-layout)
* in GitHub: [CWRC-WriterLayout](https://github.com/cwrc/CWRC-WriterLayout)

###### CWRCBasicDelegator

Delegator to which the [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) delegates server side calls for file creation in the file system on the server; entity lookups; schema retrieval; xml validation; template loading.  
NOTE: THIS PACKAGE IS DEPRECATED AND SHOULD NOT BE USED.

* in NPM: [cwrc-basic-delegator](https://www.npmjs.com/package/cwrc-basic-delegator)
* in GitHub: [CWRC-BasicDelegator](https://github.com/cwrc/CWRC-BasicDelegator)

Typical development on the browser part of the CWRC-Writer will therefore be changes to the above packages.  Each package has it's own GitHub repository, listed above, with specifics about how to work with it.  General development practices are also listed below in [How to Work with CWRC packages](#how-to-work-with-cwrc-packages).

## Server

The server (or servers) provides services for storing documents, XML validation, and entity lookup.  The server(s) can be implemented however one would like, and in particular, can be a more general server, or set of servers, used by other applications besides the CWRC-Writer.  Some of the backend services could even be provided by a third party, like our default public entity lookup, provided by VIAF.  In any case the services needed are:

#### Entity Lookup

The default lookup for the CWRC-Writer is an example of a general service that isn't specific to the CWRC-Writer.  It uses the VIAF servers directly.  The [cwrc-public-entity-dialogs (NPM)](https://www.npmjs.com/package/cwrc-public-entity-dialogs) package running in the browser makes calls directly to VIAF.  You could alternatively implement your own server side entity management system and follow the example of the [cwrc-public-entity-dialogs (NPM)](https://www.npmjs.com/package/cwrc-public-entity-dialogs) to make calls to your own system or to any other system, for example to another lookup service like say WorldCat.

#### XML Validation

The default XML validator is a public server supplied by CWRC.  The call to it is in the default delegator: [cwrc-git-delegator (NPM)](https://www.npmjs.com/package/cwrc-git-delegator).  If you are implementing your own delegator, you'll probably want to use the same call to the CWRC validation server.  The XML validator is another example of a general service that isn't specific to CWRC-Writer.

#### Document Storage

The default backend server we use for storage is the [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer), an Express.js server that in turn uses GitHub for storage.  We don't make direct calls to GitHub from the browser because we use GitHub's OAuth, which requires that we run a server to recieve the OAuth callback from GitHub.

The [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer) uses one other CWRC NPM package:

* in NPM: [cwrcgit](https://www.npmjs.com/package/cwrcgit)
* in GitHub: [CWRC-Git](https://github.com/cwrc/CWRC-Git)
Client for creating and updating CWRC XML documents in GitHub through the GitHub API.  Used by the [CWRC-GitServer](jchartrand/CWRC-GitServer).

Typical development on the server part of the CWRC-Writer will therefore be changes to the [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer) (probably to change [routes](https://expressjs.com/en/guide/routing.html)) and to [cwrcgit](https://github.com/cwrc/CWRCGit).  Both have their own GitHub repository (hyperlinked in the prior sentence) with specifics about how to work with it.  General development practices are also listed below in [How to Develop with CWRC packages](#how-to-work-with-develop-packages).

## How to Develop with CWRC Packages

There are two types of package:  those that interact with the DOM and are intended only to run in a web browser, and those that don't interact with the DOM and might run either in the web browser or on the server in node.js.  Both types of package are fundamentally the same, but we test the web packages differently.  The following steps apply to both types of package, with the different testing steps for the web packages marked accordingly.

#### Basic Development Setup

* Fork or clone (depending on your role in the project) the relevant repo (i.e., one of the CWRC repos: CWRC-WRiterBase, CWRC-Git, etc.) to your local machine.

* `npm install` to install the node.js dependencies 
	
	NOTE:  we use `npm set save-exact true` to save dependencies as exact version numbers, so NPM should install exact versions when you run install

* If the repository has a config.js file with passwords or tokens, you'll have to set these values appropriately in your cloned repo.  Tell git to ignore the file completely (so that you don't inadvertently commit the file and push it to the public repo thereby exposing the passwords):

`git update-index --skip-worktree config.js`

Note that .gitignore doesn't ignore files that have been comitted, and the config.js file likely has been commited to allow the Travis build tool to run, albeit with dummy values.

#### Test and code

* write a test (or two) for your new feature 

* `npm test` to run the tests

* write some code to satisfy new test

Depending on the type of package, different tests are used:

###### no DOM no browswer no problem

Testing of non-DOM (no interaction with the browser DOM, so can run either in browser or on server in node.js) packages is described in the [cwrc-git](https://github.com/cwrc/CWRC-Git) package.

###### DOM

Testing of packages that interact with the browser DOM is described in the [cwrc-git-dialogs](https://github.com/cwrc/cwrc-git-dialogs) package.

###### REST API

Testing of REST API calls is described in the [CWRC-GitServer](https://github.com/cwrc/CWRC-GitServer) package.


#### Commit to Github / Build in Travis / Release to NPM

We use [commitizen](https://www.npmjs.com/package/commitizen), [Travis](https://travis-ci.org), [semantic-release](https://www.npmjs.com/package/semantic-release), [Istanbul](https://www.npmjs.com/package/istanbul) (although Istanbul has just been subsumed into NYC so we'll soon have to update), and [codecov.io](https://codecov.io) for our commits, builds, NPM releases, code coverage, and code coverage reporting.  This should all be mostly setup when you clone the repository, but you may have to rerun some portions on your own machine.  For a full description of the setup see below [How to Create a CWRC package](#how-to-create-a-cwrc-package).

Semantic-release-cli configures the corresponding Travis build (on the Travis web site in the Travis account associated with the given Github username) so that when the Travis build is triggered (whenever you push a change to the GitHub repo), Travis will run semantic-release, which will in turn:

- write a new version number to package.json
- deploy a new version to the NPM registry if the commited change is either a new feature or a breaking change.
- generate a changelog
- create a release in the Github project

A full description of what semantic-release-cli does is [here](https://github.com/semantic-release/cli#what-it-does).
A full description of what semantic-release itself does is [here](https://github.com/semantic-release/semantic-release#how-does-it-work)

##### Commits

To submit a commit, stage your changes (e.g., git add -A) then instead of using git's commit command, use `npm run cm` (or in some cases the script name may be `commit` so run `npm run commit`.  Just check the scripts property of the package.json to confirm.) which uses commitizen to create commits that adhere to the semantic-release conventions (the same conventions as those used by Google: https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#commit )

The NPM `ghooks` package (ghooks has just been replaced by [husky](https://github.com/typicode/husky) so we'll soon have to upgrade) is used to add two pre-commit git hooks that will check that all tests pass and that code coverage is 100% (as caluclated by istanbul) before allowing a commit to proceed.  The hooks are set in package.json:

```
"config": {
    "ghooks": {
      "pre-commit": "npm run test:single && npm run check-coverage"
    }
  }
```

After the commit has succeeded then `git push` it all up to github, which will trigger the Travis build.  The Travis build is also set to confirm that all tests pass and that code coverage is 100%.  This is set in the `.travis.yml` file like so:

```
script:
  - npm run test:single
  - npm run check-coverage
```

Of course, if the githooks that check tests and code coverage themselves passed, then the Travis check for tests and code coverage should also be fine (but a second check is well worth running).

##### Travis

Results of the travis build are accessed using the github owner's name, and the repository name, like so:

`https://travis-ci.org/cwrc/CWRC-Git` 

##### Code Coverage

After the Travis build finishes (successfully), Travis triggers two of our NPM scripts:  to report coverage to codecov.io and to run semantic release to publish to NPM.  These triggers are defined in `.travis.yml`:

```
after_success:
  - npm run report-coverage
  - npm run semantic-release
```

report-coverage publishes the code coverage statistics to codecov.io where the coverage can be viewed:

`https://codecov.io/gh/cwrc/CWRC-Git/`

You can also browse the code coverage reports locally by opening:

`coverage/lcov-report/index.html`

in the project directory.

codecov.io also provides us with the code coverage badge at the top of this README.

##### NPM Publishing

Finally the `npm run semantic-release` script, listed in the Travis after_success section, is triggered.

The [semantic-release](https://github.com/semantic-release/semantic-release) script follows the [SemVer](http://semver.org/) spec, and:
 
- updates our version number in package.json, following the SemVer spec.
- publishes a new version to NPM
- generates a changeling and tags our github repository with a new release

Read more about semantic-release and how it works here:  [How Semantic Release works](https://github.com/semantic-release/semantic-release#how-does-it-work)

## How to Create a new CWRC Package

There are two types of package:  those that interact with the DOM and are intended only to run in a web browser, and those that don't interact with the DOM and might run either in the web browser or on the server in node.js.  Both types of package are fundamentally the same, but we test them differently.  The following steps apply to both types of package, with different testing steps outlined accordingly.

##### Create Github repo

create a new github repository in the cwrc account (from the cwrc github web page)
	- as of this writing, choose GPL2.0 for the licence, and node for the .gitignore 
	- choose to include a README or not - doesn't really matter, you can create one later

clone the repository to your local machine.  From the directory in which you'd like the new directory created:

```git clone git@github.com:cwrc/cwrc-somepackage.git```

##### Initialize as NPM package

switch into the newly created directory and initialize it as an NPM package:

```
cd cwrc-somepackage
npm init
```

NPM will prompt you for a few things.  

For 'entry point: (index.js)' switch it to `src/index.js`.  For 'license' specify GPL-2.0

Otherwise answer appropriately. 

##### Install dependencies

Install whatever NPM packages you need.  External NPM packages (from the NPM registry) can be used as:

- tools during development (typically in build scripts or during development or testing, .g., a test runner, a lint tool, a watch) 
- as part of your own package (a dependency)
- globally during development.  

When installing an NPM package indicate where it should go with either -D (development) -S(standard dependencies, i.e,. packages used by your new package), -g(install globally, usually to use as a command line tool).  

For CWRC, we typically install the following development tools:

```
npm i -D tape commitizen cz-conventional-changelog husky semantic-release codecov.io nyc watch faucet
```

and for packages that are to be run on the browser also install:

```
npm i -D babel-preset-es2015 babelify browserify browserify-istanbul watchify concat-stream
```

If the package makes http requests then you'll probably want to mock those calls to keep tests fast.  We've used [nock](https://github.com/node-nock/nock) for node.js:

npm i -D nock

and [sinon](http://sinonjs.org) for mocking in the browser:

npm i -D sinon

You’d also install whatever packages will be used by your new package (either to run on the server in Express.js or to be bundled up by browserify into the bundle that is sent down to the browser) but saving them as standard dependencies like so (substitute whatever packages you’ll use, but you can install them anytime):

```
npm i -S jquery bootstrap  
```

And finally install semantic-release-cli as a global, so it can be run from the command line:

```
npm i -g semantic-release-cli 
```

##### Configure NPM settings

Now make sure you've got NPM configured to publish to the NPM registry:

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

Semantic release will:

- create .travis.yml (which tells Travis what NPM scripts to run as part of the build, which versions of node.js to use, etc.)
- login to travis and set github and npm tokens that allow semantic release to later tag a github release, and to publish to npm.

Modify the .travis.yml that semantic-release-cli created so that 'script' and 'after_success' look like:

```
script:
  - npm run test:single
  - npm run check-coverage
after_success:
  - npm run report-coverage
  - npm run semantic-release
```

The entries in ’script’ are run first.  If they both pass, then the 'after_success' scripts are run.  Report-coverage sends our coverage information to codecov.io.  The [semantic-release](https://github.com/semantic-release/semantic-release) script follows the [SemVer](http://semver.org/) spec, and:
 
- updates our version number in package.json, following the SemVer spec.
- publishes a new version to NPM
- generates a changelog and tags our github repository with a new release

Read more about semantic-release and how it works here:  [How Semantic Release works](https://github.com/semantic-release/semantic-release#how-does-it-work)

Everything that semantic-release-cli does is described here:  https://github.com/semantic-release/cli#what-it-does

Note the difference between semantic-release-cli and semantic-release.  semantic-release-cli is run once on the command line to setup travis, etc.  semantic-release is run everytime a build is invoked on Travis.

##### Configure commitizen

Add a script to package.json 'scripts' property:

```
"commit": "git-cz",
```

 Add a commitizen property to the package.json config property:

```
"config": {
    "commitizen": {
      "path": "node_modules/cz-conventional-changelog"
    }
  }
```

[cz-conventional-changelog](https://github.com/commitizen/cz-conventional-changelog) tells commitizen to structure our commits according to [AngularJS's commit message convention](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#-git-commit-guidelines) also called the [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog).

##### Configure Husky

When installed [Husky](https://github.com/typicode/husky) overwrites certain [github hooks](https://git-scm.com/docs/githooks) in the .git/hooks directory to trigger prenamed NPM scripts.  We use the 'precommit' hook. Add a precommit script to ‘scripts’:

```
“scripts”: {
	”precommit": "npm run test:single && npm run check-coverage"
}
```

[Husky](https://github.com/typicode/husky) triggers this script whenever a commit is made to git.  As you can see, it will run our tests and verify our test coverage.

##### Add test and coverage scripts

All tests, both in node.js and in the browser, are [TAPE](https://github.com/substack/tape).

Add the following to the package.json 'scripts' property.

###### non-DOM 

```
"test:single": "nyc tape test/main.js | tap-spec",
"check-coverage": "nyc check-coverage --statements 0 --branches 0 --functions 0 --lines 0",
"report-coverage": "nyc report --reporter=text-lcov > coverage.lcov && codecov"
```

[tap-spec](https://www.npmjs.com/package/tap-spec) formats the [tap](https://testanything.org) output from [TAPE](https://github.com/substack/tape).  Could also use [faucet](https://www.npmjs.com/package/faucet) to format the output.

[nyc](https://github.com/istanbuljs/nyc) is the new incarnation of [istanbul](https://github.com/gotwarlost/istanbul) and calclates test coverage.

###### DOM

```
"test:single": "npm run test:electron && npm generate-coverage",
"test:browser": "browserify -t browserify-istanbul test/browser.js | browser-run  -p 2222 --static .  | node test/extract-coverage.js | faucet",
"test:electron": "browserify -t browserify-istanbul test/browser.js | browser-run --static . | node test/extract-coverage.js | faucet ",
"test:chrome": "browserify -t browserify-istanbul test/browser.js | browser-run --static . -b chrome | node test/extract-coverage.js | faucet ",
"check-coverage": "nyc check-coverage --statements 0 --branches 0 --functions 0 --lines 0",
"report-coverage": "nyc report --reporter=text-lcov > coverage.lcov && codecov"
```

Create a file test/extract-coverage.js containing:

```
'use strict'

// from https://github.com/davidguttman/cssify/blob/master/test/extract-coverage.js

var fs = require('fs')
var path = require('path')
var concatStream = require('concat-stream')

var covPath = path.join(__dirname, '..', 'coverage', 'coverage.json')

process.stdin.pipe(concatStream(function (input) {

  input = input.toString('utf-8')
  var sp = input.split('# coverage: ')
  var output = sp[0]
  var coverage = sp[1]
  console.log(output)

  fs.writeFile(covPath, coverage, function (err) {
    if (err) console.error(err)
  })
}))
```

A complete explantation of how we test in the browser and generate test coverage statistics (tricky!) is [here](https://github.com/cwrc/cwrc-git-dialogs#testing)

##### Setup browser development

If the package is intended to run in the web browser, then we'd like to run the code while developing to see the effect of changes. So, we add an NPM script to browserify the code and thereby allow manually testing it directly in a web browser: 

```
"browserify": "browserify test/manual.js -o build/test.js --debug -t [ babelify --presets [ es2015 ] ]",
```

Couple this with a watch (using watchify, which is basically browserify with a watch), and it becomes that little bit easier to makes changes to the source and see the result immediately in the browser (with a refresh - command-R):

```
 "watch": "watchify test/manual.js -o build/test.js --debug --verbose -t [ babelify --presets [ es2015 ] ]",
```

The build/test.js file can now be linked into an html file to allow us to play with the running code in a browser.  An example index.html file is in the [cwrc-git-dialogs](https://github.com/cwrc/cwrc-git-dialogs) project.

##### Source

Create a 'src' folder.  The 'entry point' into your new package is src/index.js  (as specified during 'npm init' and in the package.json for 'main').  This is where your code goes.  You can of course split your code out into other files that are required into index.js, but ideally the package is small enough that the source fits comfortably within the single index.js file.  If it gets unwieldy, consider splitting out a new package.

##### Tests

As mentioned earlier all tests are [TAPE](https://github.com/substack/tape).  

##### Commit

Okay, we’re pretty much setup.  Now commit to github, but follow this slightly different approach:

Instead of:

```
git commit -m “some message”
```
use:
```
npm run commit
```
which will prompt for various things with which to write the changelog, and will then try to commit.  This will trigger the Git precommit hook, which in turn runs the husky scripts in .git/hooks/precommit, which will run our tests and confirm that our test coverage meets our set limit.  If all's well, the commit is made.

##### Push to GitHub

After successfully commiting:

```
git push
```

it all up to GitHub which will trigger Travis and hence run the tests and coverage checks again, and then run semantic release as described earlier.







