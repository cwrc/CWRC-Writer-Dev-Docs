
## Testing with DOM

There are [TAPE](https://github.com/substack/tape) tests in the test directory that can be browserified and run in a spawned web browser via [browser-run](https://github.com/juliangruber/browser-run).  The following npm script, defined in package.json, will run browser-run with the Electron headless web browser that is packaged with browser-run:

```
npm run test:browser
```

or to rebuild the browserify build when source files change:

```
npm run watch
```

You can force browser-run to use specific browsers (chrome, firefox, ie, phantom, safari  [default: "electron"]) with the b switch, like in the test:firefox script in package.json with:

```
npm run test:chrome
```

 Or have browser-run start listening on a given port so that you can then open any web browser you like to that port (e.g., http://localhost:2222):

```
npm run test:browser
```
## Code Coverage

We generate code coverage statistics with [Istanbul](https://www.npmjs.com/package/istanbul) and publish them to [codecov.io] when our Travis build runs.  Hopefully you won't need to get into the mechanics of the code coverage generation, but if ever you do, read on...

Generating code coverage reports when running tests in the browser is slightly tricky.  Take a look at test:browser script in package.json:

"test:browser": "browserify -t browserify-istanbul test/browser.js | browser-run  -p 2222 --static .  | node test/extract-coverage.js | faucet",

You can see that we've inserted a browserify transform called [browserify-istanbul](https://www.npmjs.com/package/browserify-istanbul) which invokes the code coverage tool [Istanbul](https://www.npmjs.com/package/istanbul) to 'instrument' the code we're browserifying.  [Instrumentation](https://en.wikipedia.org/wiki/Instrumentation_(computer_programming)) adds instcructions to the original source code. In this case to allow the code coverage tool to determine which parts of the original source code are called by the tests, and which aren't (which leds us then calculate percentages of lines covered by tests).

After the tests run (in a browser via [browser-run](https://github.com/juliangruber/browser-run)), browserify-istanbul has put the code coverage information in a property of the global window object of the browser:

``` window.__coverage__```

To get that out after the tests have finished, we use the onFinish event provided by TAPE, which is run after ALL tests have run:

test.onFinish(()=>{
        console.log('# coverage:', JSON.stringify(window.__coverage__))
        window.close()
    })

By sending the coverage data to the console, the coverage data is attached to the output from our TAPE tests, but delineated with '# coverage:'  so that we can pull it out later.

Just after logging the coverage, we close the browser window (window.close()), which takes us back to our test:browser script, just after the browser-run command, where we now pipe the output to 'node test/extract-coverage.js'.  extract-coverage is gratefully borrowed from https://github.com/davidguttman/cssify/blob/master/test/extract-coverage.js.  It simply separates the code coverage information from the TAPE output (using the '# coverage;' marker we inserted earlier), writes the code coverage data to coverage/coverage.json, and sends the TAPE output along the pipe.

extract-coverage.js contents:
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

Now we've got code coverage information, but we have one more step to convert it to the lcov format that we can send to codecov.io (and that we can also browse in our own local web browser using the nice formatted version in coverage/lcov/index.html).  We convert with:

``` 
npm generate-coverage 
```

which for now is invoked in test:single after running test:electron

```
"test": "npm run test:electron && npm generate-coverage",
```

`test` is what we ask Travis to run when checking our build.  Note that Travis can only run the test against the headless browser Electron.

Finally we publish the coverage at codecov.io:

```
"report-coverage": "cat ./coverage/lcov.info | codecov"
```

