# Creating new React Project with Linter and Prettier

- Create new React project:
    - `npx create-react-app my-app`
- Run Project:
    - `cd my-app`
    - `npm start`
- ESLint and Prettier install
    - In project root run:
    - `npm install --save-dev eslint-config-react-app eslint@^8.0.0`
    - `npm install --save-dev eslint-config-prettier`
    - `npm install eslint-plugin-react@latest --save-dev`
    - `npm init @eslint/config`
- Configure exlintrc.js:
    - To make sure Prettier overrides ESLint rules, in `eslintrc.js` :
        - Last statement in extends: `"prettier"`
    - Make sure `"react-app"` and `"plugin:react-hooks/recommended"` are also extended
- To disable prop types error, add to top of file: `/* eslint-disable react/prop-types */`
- To disable require react: [[]]#configuration)
    
    ```json
    "react/jsx-uses-react": "off",
    "react/react-in-jsx-scope": "off"
    ```
    

### .eslintrc.js

```json
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
		"react-app",
    "prettier",
  ],
  overrides: [],
  parserOptions: {
    ecmaVersion: "latest",
    sourceType: "module",
  },
  plugins: ["react"],
  rules: {},
};
```