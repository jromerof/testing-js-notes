# Testing Javascript Notes

## Configure Jest for testing

1) Add Jest with ```yarn add jest -D```
2) Set the Jest script in package.json
```javascript
    ...
    "test": "jest"
    ...
```
It automatically search for files in \_\_test\_\_ folders or filenames like filename.test.js  

3) For using with Modules ( ```import``` statements), since it runs in node you have to have babel parse them. For that you can use this config
```javascript
const isTest = String(process.env.NODE_ENV) === "test";

module.exports = {
  presets: [
    ["@babel/preset-env", { modules: isTest ? "commonjs" : false }],
    "@babel/preset-react",
  ],
  plugins: [["@babel/transform-runtime"]],
};
```
Jest automatticaly pass the NODE_ENV = test
In babel preset if the enviroment is test you use the commonjs modules

4) For using node or jsdom enviroments you can make a ``jest.config.js``file like this 
```javascript
module.exports = {
  testEnvironment: "jest-environment-jsdom", // Or jest-environment-node
};
```
When you are using only node you get a performance boost in the start up of jest because it doesn't need to load jsdom. If you're doing browser code you need jsdom to use window or other browser functions

5) Test that import styles will fail because jest doesn't known how to "read" css. If you import a ```.css```file inside your test.js you need to mock them up. You can create a .js file that exports an empty object like
```javascript
module.exports = {}
```
And declare inside the ```jest.config.js```that every ```.css``` import will be mapped to that mock file
```javascript
module.exports = {
  ...
  moduleNameMapper: {
    '\\.css$': require.resolve('./test/style-mock.js'),
  },
  ...
}
```
If you want to test css properties you will want to use some other tool like cypress

6) You can generate snapshots ([Snapshot Testing · Jest](https://jestjs.io/docs/en/snapshot-testing.html#snapshot-testing-with-jest))
Snapshots are serializable values of a variable passed to jest. You can use them to assert changes in the values returned. Some use cases of snapshots are asserting returned values from functions that returns objects or arrays, or using them to serialize dom nodes to check how your React components will render.
If a snapshots changes between test Jest will throw an error. It's a good practice to check snapshots changes.

You can use 
```javascript
test('makes a snapshot', () => {
  const someArrayOfObjects = getAnArrayOfObjects()
  
  expect(someArrayOfObjects).toMatchSnapshot()
})
```
In this case  ```toMatchSnapshot()``` writes an snapshots to a ```.snap``` file. You can also use ```toMathInlineSnapshot()``` for Jest to write the snapshot inside the function (requires prettier).

If you checked the new snapshot value and there is no error you can use the `-u` flag to update the snapshots. 

7) Jest doesn't known when you do a Webpack resolve.modules with your utils to share them in your code, so it will thrown an error if you have them imported in the files that you're testing.
You can configure Jest to read the modules the same way that your Webpack config adding a ``moduleDirectories`` to the config
```javascript
const path = require('path')
module.exports {
  ...
  moduleDirectories: ['node_modules', path.join(__dirname, 'src'), 'yourModuleFolder'],
  ...
}
```
8) If you want to make an import boilerplate in all your test you can use ```setupFilesAfterEnv``` in the jest config
```javascript
module.exports = {
  ...
setupFilesAfterEnv: ['@some-import'],
  ...
```
That way Jest will load the boilerplate after the setting up of the testing enviroment

9) If you're using custom module imports for jest in the ``jest.config.js`` file eslint will throw error on the imports. 
You can use an eslint-resolver with
```
npm install --save-dev eslint-import-resolver-jest
```
And then add to your .eslintrc an override for \_\_test__ folders
```javascript
  overrides: [
    ...
    {
      files: ['**/__test__/**'],
      settings: {
        'import/resolver': {
          jest: {
            jestConfigFile: path.join(__dirname, './jest.config)
        },
    }},
  ],
```

10) Jest has a watch mode to run tests autommatically. The watch mode checks the last git commit and run the test for files that changed.
Watch mode is interactive so check the console.

```javascript
"scripts": {
    ...
    "test:watch": "jest --watch",
    ...
    }
```

11) You can debug your test with chrome. Write ``debugger`` at the place you want your code to stop and then run the debugger. You can add jest to the debugger with the following script.

```javascript
"scripts": {
    ...
    "test:debug": "node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand --watch",
    ...
    }
```

12) You can make test coverage reports adding a ``--coverage``flag to your jest script.
To decide what folders do you want to read to create the coverage add  an array of directories (or a expresion) to ``collectCoverageFrom``on your jest config.
```javascript
collectCoverageFrom: ['**/src/**/*.js']
```
To make a threshold for testing coverage you can add a ``coverageThreshold``object to your jest.config. You can add other objects with the directory names to make separated thresholds.

```javascript
coverageThreshold: {
    global: {
      statements: 34,
      branches: 24,
      functions: 34,
      lines: 29,
    },
    './src/shared/utils.js': {
      statements: 100,
      branches: 80,
      functions: 100,
      lines: 100,
    },
  },
```

13) You can use multiple config passing the config as a parameter with the ``--config``flag

You can use the global configs to run multiple config adding an array of ``projects`` to the jest global config. The array must have the path to the other configs.

(It seems a good practice to have a common config and import it to the other ones)

14) You can run eslint as a jest task, adding it to jest watch mode.

Install ``jest-runner-eslint`` as a dev dependency.
Then create a jest.lint.js like this:
```javascript
const path = require('path')

module.exports = {
  rootDir: path.join(__dirname, '..'),
  displayName: 'lint',
  runner: 'jest-runner-eslint',
  testMatch: ['<rootDir>/**/*.js'],
}
```
To ignore the .gitignore files add to your ``package.json``a config for jest-runner-eslint
```json
"jest-runner-eslint": {
  "clipOptions": {
    "ignorePath": "./.gitignore
  }
```
Add the ``jest.lint.js``to the projects array in the ``jest.config.js`` and you're ready

15) You can add the ``jest-watch-select-projects`` plugin to select an especific project to run test in watch mode.
Add it to your jest config file
```javascript
...
watchPlugins: ['jest-watch-select-projects'],
...
```
16) To filter more easily wich file you want to test in watch mode you can install the ``jest-watch-typeahead``plugin to help you preview what files you're getting with  your patterns.

```javascript
watchPlugins: [
  'jest-watch-select-projects',
  'jest-watch-typeahead/filename',
  'jest-watch-typeahead/testname',
]
```

17) To run test on pre-commit only on the files changed you can use ``husky``and ``lint-staged``

```
npm install --save-dev husky lint-staged
```
```json
//In package.json
"husky": {
    "hooks": {
      "pre-commit": "lint-staged && npm run build"
    }
  },
  "lint-staged": {
    "**/*.+(js|json|css|html|md)": [
      "prettier",
      "jest --findRelatedTests",
      "git add"
    ]
  },
```

\* There is some other lessons on emotion and css modules that I didn't write because I don't particulary use them in my personal stack or at work

\* Tools mentioned not on the notes: 
* Codecov
* is-ci package
