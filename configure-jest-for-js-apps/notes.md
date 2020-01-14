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

\* There is some other lessons on emotion and css modules that I didn't write because I don't particulary use them in my personal stack or at work

