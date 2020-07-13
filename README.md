# Angular Github Actions

Simple Angular App (Default Starting App) automated with Github Action CI Pipeline for testing and building, connected to Codacy with a coverage badge and a CD Pipeline deploying the Application with FTP, also via Github Action.

![Build, Test & Deploy](https://github.com/JakobVesely/angular-github-actions/workflows/Build,%20Test%20&%20Deploy/badge.svg)
[![Codacy Badge](https://app.codacy.com/project/badge/Grade/413a9957f4784296a40e889235c20d4d)](https://www.codacy.com/manual/JakobVesely/angular-github-actions?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=JakobVesely/angular-github-actions&amp;utm_campaign=Badge_Grade)
[![Codacy Badge](https://app.codacy.com/project/badge/Coverage/413a9957f4784296a40e889235c20d4d)](https://www.codacy.com/manual/JakobVesely/angular-github-actions?utm_source=github.com&utm_medium=referral&utm_content=JakobVesely/angular-github-actions&utm_campaign=Badge_Coverage)

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

Also adapt Instanbul Reporter path to directory `./converage`, if neccessary!

```js
coverageIstanbulReporter: {
    dir: require('path').join(__dirname, './coverage'),
    reports: ['html', 'lcovonly', 'text-summary'],
    fixWebpackSourcePaths: true
},
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

File: `pipeline-build-test.yml`

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

### 5. Connect Codacy Code Coverage

Adding a secret named `CODACY_CODE_COVERAGE_TOKEN` in GitHub under { Repository Name } > Settings > Secrets.

```yml
      - name: Run codacy-coverage-reporter
        uses: codacy/codacy-coverage-reporter-action@master
        with:
          project-token: ${{ secrets.CODACY_CODE_COVERAGE_TOKEN }}
          coverage-reports: coverage/lcov.info
```

## Prepare a CI/CD Pipeline

### 1. Adding `.git-ftp-include`

We need to add a `.git-ftp-include` in our root directory, because it is ignored by git (.gitignore) and won´t be uploaded with git ftp.
```sh
!dist/
```

### 2. Add Credential Secrets

Adding the three credential secrets under { Repository Name } > Settings > Secrets:
-   `CD_FTP_SERVER`
-   `CD_FTP_USERNAME`
-   `CD_FTP_PASSWORD`

### 3. Extend the CI Pipeline for Deployment with FTP

We are using the Pipeline from before and adding one last step, the git ftp action, to it:

```yml
      - name: FTP Deploy
        uses: SamKirkland/FTP-Deploy-Action@3.1.1
        with:
          ftp-server: "${{ secrets.CD_FTP_SERVER }}"
          ftp-username: "${{ secrets.CD_FTP_USERNAME }}"
          ftp-password: "${{ secrets.CD_FTP_PASSWORD }}"
          local-dir: "dist/"
```
