# Angular Github Actions

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/7e17715d9aec43e58f18a331fac56823)](https://app.codacy.com/manual/JakobVesely/angular-github-actions?utm_source=github.com&utm_medium=referral&utm_content=JakobVesely/angular-github-actions&utm_campaign=Badge_Grade_Dashboard)

## Prepare a CI Pipeline

### 1. Install Dev Dependencies

```sh
npm install ci --save
npm install puppeteer --save-dev
```

jasmine and karma packages comes automatically via `ng cli` (starting new angular project with ng new)!


### 2. Add Karma Config

File: `/karma.conf.js`

```js
config.set({
    /* Insert Start */
    restartOnFileChange: true,
    browsers: ['Chrome', 'ChromeHeadlessCustom'],
    customLaunchers: {
        ChromeHeadlessCustom: {
            base: 'ChromeHeadless',
            flags: ['--no-sandbox', '--disable-gpu']
        }
    },
    /* Insert End */
    ...
});
```


### 3. Setup npm scripts for CI

Add `ci:clean`, `ci:test` and `ci:build` scripts to `/package.json` File!

```json
"scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test",
    "lint": "ng lint",
    "e2e": "ng e2e",
    "ci:clean": "rimraf ./dist",
    "ci:test": "ng test --watch=false --browsers=ChromeHeadlessCustom --code-coverage",
    "ci:build": "ng build --prod"
}
```

### 4. Create Github Action

File: `pipeline-buidl-test.yml`

```yml
name: Build and Test

on:
  push:
    branches: [master]

jobs:
  build:
    name: Build & Test
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [12.x]

    steps:
      - uses: actions/checkout@v2

      - name: Cache node modules
        uses: actions/cache@v1
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
            
      - name: Setup Node.js environment
        uses: actions/setup-node@v1.4.2
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install Dependencies
        run: npm ci
          
      - name: Run Unit Tests
        run: npm run ci:test

      - name: Clean Distribution Directory
        run: npm run ci:clean
        
      - name: Build Application
        run: npm run ci:build
      
      - name: List Files in Distribution Directory
        run: ls -R ./dist
```