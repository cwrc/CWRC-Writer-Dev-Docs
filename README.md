![Picture](http://www.cwrc.ca/wp-content/uploads/2010/12/CWRC_Dec-2-10_smaller.png)

CWRC-Writer-Dev-Docs
====================

Describes the overall organization of the CWRC-Writer code as NPM packages, and the development procedures for CWRC-Writer coding.

1. [Overview](#overview)
1. [CWRC-Writer](#cwrcwriter)
1. [Server](#server)
1. [Development Process](#development)

## Overview

The CWRC-Writer is an in-browser WYSIWYG XML text editor that also supports standoff RDF annotation to mark references to named entities in the text.  There are two main parts to a CWRC-Writer installation that run more or less independently:  the CWRC-Writer editor itself that runs in the web browser, and the complementary backend services for document storage, XML validation, and entity lookup, that run on a server.  

## CWRC-Writer

An instance of the CWRC-Writer web editor is built around the [CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base), a heavily customized version of the [TinyMCE](https://www.tinymce.com) web editor.

#### Delegator

The CWRC-Writer editor ([CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base) running in the web browser) communicates with the backend services throug a javascript object that we've called the 'Delegator' (because the editor 'delegates' server side requests to it).  The delegator handles requests to list, create, or save documents, or to validate XML documents.  The delegator essentially forwards the requests on to the server that carries out the actual request.  The default delegator for CWRC is the [CWRC-GitDelegator](https://www.npmjs.com/package/cwrc-git-delegator) that handles requests to the default [CWRC-GitServer](https://www.npmjs.com/package/cwrc-git-server), which in turn makes calls to GitHub itself to create documents, list documents, save documents, retrieve documents, etc.)

#### Entity Lookup

The editor also allows users to lookup references to named entities (people, places, organizations) and so another javascript object, much like the delegator, is also packaged in with the [CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base), and handles entity lookup. The default entity lookup package is [cwrc-public-entity-dialogs](https://www.npmjs.com/package/cwrc-public-entity-dialogs) which looks up named entities in [VIAF](https://viaf.org) and returns unique URIs for the selected entity.

#### NPM packages and browserify

The [CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base), the delegator, the entity lookups, and a few other components are organized as [NPM](https://www.npmjs.com) packages (and published to the [public npm repository](https://www.npmjs.com/search?q=cwrc) for use by anyone).  

The NPM packages that contribute to the browser part of the CWRC-Writer are combined together using the node.js module loading system and with [Browserify](https://browserify.org).  We write code like we would for a node.js application (that would normally run on the server), using the node.js 'require' statements to import packages, and Browserify bundles all the code together (both our packages and all other packages we've included like jquery, bootstrap, and so on) into a single javascript file that can then be referenced in the index.html page of our web app.

The best example of how the NPM packages are combined and browserified is in the [CWRC-GitWriter](https://github.com/jchartrand/CWRC-GitWriter) repository.  Specifically take a look at the [app.js](https://github.com/jchartrand/CWRC-GitWriter/blob/master/src/js/app.js) file, the so-called 'entry point' into the application, which is from where Browserify starts and then 'crawls' the dependency tree to pull in all dependencies (as defined by 'require' statements).  The [app.js](https://github.com/jchartrand/CWRC-GitWriter/blob/master/src/js/app.js) file also shows how the 'require' statements are used to combine the CWRC-Writer javascript packages together, by passing them into the [CWRC-WriterBase](https://www.npmjs.com/package/cwrc-writer-base).

The CWRC NPM packages that are used in the browser:

###### CWRC-WriterBase
The base class for the cwrc-writer.

* in NPM: [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base)
* in GitHub: [CWRC-WriterBase](https://github.com/jchartrand/CWRC-WriterBase)

###### CWRCGitDelegator
Delegator to which [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) delegates server-side calls.  Used by the [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) to make calls to [CWRC-GitServer](https://github.com/jchartrand/CWRC-GitServer).

* in NPM: [cwrc-git-delegator](https://www.npmjs.com/package/cwrc-git-delegator)
* in GitHub: [CWRC-GitDelegator](https://github.com/jchartrand/CWRC-GitDelegator)

###### CWRC-GitServerClient
Client for calls to the [CWRC-GitServer](https://github.com/jchartrand/CWRC-GitServer) from the [cwrc-git-delegator](https://www.npmjs.com/package/cwrc-git-delegator).

* in NPM: [cwrc-git-server-client](https://www.npmjs.com/package/cwrc-git-server-client)
* in GitHub: [CWRC-GitServerClient](https://github.com/jchartrand/CWRC-GitServerClient)

###### CWRCPublicEntityDialogs
Dialogs for the [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) that lookup people, places, organizations, and publications in public authority files.

* in NPM: [cwrc-public-entity-dialogs](https://www.npmjs.com/package/cwrc-public-entity-dialogs)
* in GitHub: [CWRC-PublicEntityDialogs](https://github.com/jchartrand/CWRC-PublicEntityDialogs)

###### CWRCWriterLayout
Components for customizing the CWRC-Writer layout.  This package is used by the layout-config.js file in an instance of the CWRC-Writer.  See (CWRC-GitWriter)[https://github.com/jchartrand/CWRC-GitWriter] for an example.  THIS PACKAGE IS DEPRECATED AND SHOULD NOT BE USED.  THE LAYOUT CODE HAS BEEN INCORPORATED DIRECTLY INTO THE CWRC-WRITER-BASE.

* in NPM: [cwrc-writer-layout](https://www.npmjs.com/package/cwrc-writer-layout)
* in GitHub: [CWRC-WriterLayout](https://github.com/jchartrand/CWRC-WriterLayout)

###### CWRCBasicDelegator

Delegator to which the [cwrc-writer-base](https://www.npmjs.com/package/cwrc-writer-base) delegates server side calls for file creation in the file system on the server; entity lookups; schema retrieval; xml validation; template loading.  THIS PACKAGE IS DEPRECATED AND SHOULD NOT BE USED.

* in NPM: [cwrc-basic-delegator](https://www.npmjs.com/package/cwrc-basic-delegator)
* in GitHub: [CWRC-BasicDelegator](https://github.com/jchartrand/CWRC-BasicDelegator)

Typical development on the browser part of the CWRC-Writer will therefore be changes to the above packages.  Each package has it's own GitHub repository, listed above, with specifics about how to work with it.  General development practices are also listed below in [Development Process](#development).

## Server

The server provides services for storing documents, XML validation, and entity lookup.  The server can be implemented however one would like, and in particular, can be a more general server used by other applications besides the CWRC-Writer.

#### Entity Lookup

The default lookup for the CWRC-Writer is an example of a general service that isn't specific to the CWRC-Writer.  It uses the VIAF servers directly.  The [cwrc-public-entity-dialogs (NPM)](https://www.npmjs.com/package/cwrc-public-entity-dialogs) package running in the browser makes calls directly to VIAF.  You could alternatively implement your own server side entity management system and follow the example of the [cwrc-public-entity-dialogs (NPM)](https://www.npmjs.com/package/cwrc-public-entity-dialogs) to make calls to your own system or to any other system, for example to another lookup service like say WorldCat.

#### XML Validation

The default XML validator is a public server supplied by CWRC.  The call to it is in the default delegator: [cwrc-git-delegator (NPM)](https://www.npmjs.com/package/cwrc-git-delegator).  If you are implementing your own delegator, you'll probably want to use the same call to the CWRC validation server.  The XML validator is another example of a general service that isn't specific to CWRC-Writer.

#### Document Storage

The default backend server we use for storage is the [CWRC-GitServer](https://github.com/jchartrand/CWRC-GitServer), an Express.js server that in turn uses GitHub for storage.

There is one NPM CWRC package used by the [CWRC-GitServer](https://github.com/jchartrand/CWRC-GitServer):

* in NPM: [cwrcgit](https://www.npmjs.com/package/cwrcgit)
* in GitHub: [CWRC-Git](https://github.com/jchartrand/CWRC-Git)
Client for creating and updating CWRC XML documents in GitHub through the GitHub API.  Used by the [CWRC-GitServer](jchartrand/CWRC-GitServer).

Typical development on the server part of the CWRC-Writer will therefore be changes to the [CWRC-GitServer](https://github.com/jchartrand/CWRC-GitServer) (probably to change [routes](https://expressjs.com/en/guide/routing.html)) and to[cwrcgit (NPM)](https://www.npmjs.com/package/cwrcgit).  Both have their own GitHub repository, listed above, with specifics about how to work with it.  General development practices are also listed below in [Development Process](#development).

## General Development Practices

#### Basic Setup

* Fork or clone (depending on your role in the project) the relevant repo to your local machine.

* `npm install` to install the node.js dependencies 
	
	NOTE:  we use `npm set save-exact true` to save dependencies as exact version numbers, so NPM should install exact versions when you run install

* If the repository has a config.js file with passwords or tokens, you'll have to set these values appropriately in your cloned repo.  To tell git to ignore the file completely (so that you don't inadvertently commit the file and push it to the public repo thereby exposing the passwords) use:

`git update-index --skip-worktree config.js`

Note that .gitignore doesn't ignore files that have been comitted, and the config.js file likely has, to allow the Travis build tool to run, but with dummy values.

#### Test and code

* write a test (or two) for your new feature (in 'spec' directory)

* `npm test` to start mocha and automatically rerun the tests whenever you change a file

* write some code to satisfy new test

#### Commit to Github / Build in Travis / Release to NPM

Setup automatic semantic release through continuous integration using [semantic-release-cli](https://www.npmjs.com/package/semantic-release-cli) and [commitizen](https://www.npmjs.com/package/commitizen): 

First, make sure you've got NPM configured to publish to the NPM registry:

```
npm set init.author.name "James Chartrand"
npm set init.author.email "jc.chartrand@gmail.com"
npm set init.author.url "http://openskysolutions.ca"
npm login  (answer prompts approriately)
```
then install semantic-release-cli globally:

`npm install -g semantic-release-cli`

Run semantic release from the command line:

`semantic-release-cli setup`

which will ask you a series of questions, which at the time of writing this were:

```
semantic-release-cli setup
? What is your npm registry? https://registry.npmjs.org/
? What is your npm username? jchartrand
? What is your npm password? *******
? What is your GitHub username? jchartrand
? What is your GitHub password? ********
? What CI are you using? Travis CI
```

Semantic-release sets up a Travis build (on the Travis web site in the Travis account associated with the given Github username) and a trigger in GitHub to run the Travis build on the Travis site whenever you push a change to the GitHub repo.  The Travis build will also deploy a new version to the NPM registry if the commited change is either a new feature or a breaking change.

To submit a commit, stage your changes (e.g., git add -A) then instead of using git's commit command, use `npm run commit` which uses commitizen to create commits that are structured to adhere to the semantic-release conventions (which are the same as those used by Google: https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#commit )

The NPM `ghooks` package is used to add two pre-commit git hooks that will check that all mocha tests pass and that code coverage is 100% (as caluclated by istanbul) before allowing a commit to proceed.  The hooks are set in package.json:

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

`https://travis-ci.org/jchartrand/CWRC-Git` 

##### Code Coverage

The Travis build also publishes the code coverage statistics to codecov.io where the coverage can be viewed:

`https://codecov.io/gh/jchartrand/CWRC-Git/`

You can also browse the code coverage reports locally by opening:

`coverage/lcov-report/index.html`

in the project directory.

codecov.io also provides us with the code coverage badge at the top of this README.

##### NPM Publishing

Finally the Travis build publishes a new version (if the commit was designated as a new feature or breaking change) to NPM:

https://www.npmjs.com/package/cwrcgit

##### Testing

Testing uses mocha and chai.  Tests are in the `spec` directory. 

For modules that make http calls (e.g., the cwrcgit package, which makes calls to the GitHub API, including calls to create new repositories), rather than make those calls for every test, [nock](https://github.com/node-nock/nock) instead mocks the calls to GitHub (intercepts the calls and instead returns pre-recorded data).

Testing of changes to DOM elements is done with [https://www.npmjs.com/package/jsdom](https://www.npmjs.com/package/jsdom) within standard mocha tests.

