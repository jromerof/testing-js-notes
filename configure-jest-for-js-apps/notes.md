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


