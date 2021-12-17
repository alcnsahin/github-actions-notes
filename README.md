# Github Actions

__.github/workflows/test.yml__

```
name: shell commands
on: [push]
jobs:
  run-shell-command:
    runs-on: ubuntu-latest
    steps:
      - name: echo a string
        run: echo "test command"
      - name: multiline script
        run: | 
          node -v
          npm -v
      - name: python command
        run: |
          import platform
          print(platform.processor())
        shell: python     -> komutları python ile çalıştır demek
  run-windows-commands:
    runs-on: windows-latest
    needs: [run-shell-command] -> run-shell-command bittikten sonra "run-windows-commands" Job'ını çalıştır demek. 
    steps:
      - name: Directory Powershell
        run: Get-Location
      - name: Directory bashAdministra
        run: pwd
        shell: bash
```

__.github/workflows/actions.yml__
```
name: Actions Workflow
on: [push]
jobs:
  run-github-actions:
    runs-on: ubuntu-latest
    steps:
      - name: List files
        run: |
          pwd
          ls
          echo $GITHUB_SHA --> commit id
          echo $GITHUB_REPOSITORY --> repository name
          echo $GITHUB_WORKSPACE -> workspace path in actions runner
          echo "${{ github.token }}" --> auth token
          # git clone git@github:$GITHUB_REPOSITORY
          # git checkout $GITHUB_SHA
      - name: Checkout
        uses: actions/checkout@v1 -> projeni clone'lar
      - name: List files
        run: |
          pwd
          ls
      - name: Simple JS Action
        id: greet
        uses: alcnsahin/drmon-api@4378922b26a633b3b1cb613cab21d4adb7c6eb94 commit id'si | master | yada release kısmından version bilgisi girilir. version tercih ediliyor.
        with: 
          who-to-greet: John
      - name: Log greeting Time
        run: echo "${{ steps.greet.outputs.time }}" 
```

uses: kısmında action belirtilir. bu localde (./test-action.yml) bulunan bir action da olabilir github'da önceden tanımlanmış -bir başkasının önceden geliştirmiş olduğu- bir action da olabilir. 


### DEBUG
Projenin settings bölümünden secrets eklenebilir.
Eğer debug etmek istiyorsa aşağıdaki 2 key'i secret'a eklemeliyiz.

```
ACTIONS_RUNNER_DEBUG=true
ACTIONS_STEP_DEBUG=true
```

### Kaynaklar
- https://github.com/marketplace?type=actions
- https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions#using-a-specific-shell

### Managing workflow runs
You can re-run or cancel a workflow, review deployments, view billable job execution minutes, and download artifacts.
https://docs.github.com/en/actions/managing-workflow-runs#enabling-debug-logging


## Triggering a Workflow with Github Events & Activity Types


__actions.yml:__
```
name: Actions Workflow
```
1-
```
on: [push, pull_request]
```
2-
```
on:
  push:
    pull_request:
      types: [closed, assigned, opened, reopened]
```

# Setting a Schedule to Trigger Workflows

__actions.yml:__
```
name: Actions Workflow
on: 
  schedule:
    - cron: "0/5 * * * *" --> every 5 minutes
    - cron: "0/6 * * * *" --> every 6 minutes
  pull_request:
    types: [closed, assigned, opened, reopened]
```
__crontab.guru__'dan schedule ayarlayabilirsin.


## Triggering Workflows Manually Using the Repository Dispatch Event
```
on: 
  repository_dispatch:
    types: [build]
  pull_request:
    types: [closed, assigned, opened, reopened]
steps:
  - name: payload
    run: echo ${{  github.event.client_payload.env }}
```

__HTTP METHOD:__
```
[POST]
https://api.github.com/repos/alcnsahin/drmon-frontend/dispatch
Headers:
Accept: application/vnd.github.everest-preview+json... bakarsın
Content-Type: application/json
Body:
{
  "event_type": "build"
  "client_payload": { --> pass an extra variables with this keyword
    "env": "production"
  }
}
Authorization: Basic -> just set the Github Token to password area on postman

response: 204
```

## Filtering Workflows by Branches, Tags & Paths

__actions.yml:__
```
name: Actions Workflow
on:
  push:
    branches:
      - master
      - 'feature/some-new-feature'
      - 'feature/*' --> matches feature/featA, feature/featB, doesn't match feature/feat/a
      - 'feature/**'
      - '!feature/featc' -> ignores
    branches-ignore:
      - null
    tags: 
      - v1.*
    tags-ignore:
      - v2.*
    paths:
      - '**.js'
      - '!filename.js'
    paths-ignore:
      - 'docs/**'
```

## Bölüm 3 - Environment Variables, Encryption, Expression & Context

### Default & Custom Environment Variables
- https://docs.github.com/en/actions/learn-github-actions/environment-variables

__.github/workflows/env.yml:__
```
name: ENV Vars
on: push
env:
  WF_ENV: Available to all jobs

jobs:
  log-env: 
    runs-on: ubuntu-latest
    env:
      JOB_ENV: Available to all steps in log-env job
    steps:
      - name: Log ENV Vars
        env: 
          STEP_ENV: Available to only this step
        run: |
          echo "WF_ENV: ${WF_ENV}"
          echo "JOB_ENV: ${JOB_ENV}"
          echo "STEP_ENV: ${STEP_ENV}"
      - name: Log ENV 2
          run: |
          echo "WF_ENV: ${WF_ENV}"
          echo "JOB_ENV: ${JOB_ENV}"
          echo "STEP_ENV: ${STEP_ENV}"
  log-default-env:
    runs-on: ubuntu-latest
    steps:
      - name: Default ENV Vars
        run: |
          echo "HOME: ${HOME}"
          echo "GITHUB_WORKFLOW: ${GITHUB_WORKFLOW}"
          echo "GITHUB_ACTION: ${GITHUB_ACTION}"
          echo "GITHUB_ACTIONS: ${GITHUB_ACTIONS}"
          echo "GITHUB_ACTOR: ${GITHUB_ACTOR}"
          echo "GITHUB_REPOSITORY: ${GITHUB_REPOSITORY}"
          echo "GITHUB_EVENT_NAME: ${GITHUB_EVENT_NAME}"
          echo "GITHUB_WORKSPACE: ${GITHUB_WORKSPACE}"
          echo "GITHUB_SHA: ${GITHUB_SHA}"
          echo "GITHUB_REF: ${GITHUB_REF}"
          echo "WF_ENV: ${WF_ENV}"
          echo "JOB_ENV: ${JOB_ENV}"
          echo "STEP_ENV: ${STEP_ENV}"
```

### Encrypting Env Vars
go to github repo -> settings -> secrets -> add a new secret -> for example: WF_ENV: "a secret token"

__.github/workflows/env.yml:__
```
name: ENV Vars
on: push
env:
  WF_ENV: ${{ secrets.WF_ENV }}
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Using the GITHUB_TOKEN Secret for Authentication
- https://docs.github.com/en/actions/security-guides/automatic-token-authentication
```
steps:
  - name: push a file to your repository
    run: |
      pwd
      ls -a
      git remote add origin "https://$GITHUB_ACTOR:${{ secrets.GITHUB_TOKEN }}@github.com/$GITHUB_REPOSITORY.git"
      git config --global user.email "bot@bot.com"
      git config --global user.name "bot"
      git fetch
      git checkout master
      git branch --set-upstream-to=origin/master
      git pull
      ls -a
      echo $RANDOM >> random.txt
      ls -a
      git add -A
      git commit -m"Random file"
      git push
```
### Encrypting & Decrypting Files
- https://docs.github.com/en/actions/security-guides/encrypted-secrets
- https://www.gnupg.org/

### Expression & Contexts
- https://docs.github.com/en/actions/learn-github-actions/contexts

__.github/workflows/main.yml__
```
on: push
jobs:
  one:
    runs-on: ubuntu-latest
    steps:
      - name: Dump GitHub context
        env:
          GITHUB_CONTEXT: ${{ toJSON(github) }}
        run: echo "$GITHUB_CONTEXT"
      - name: Dump job context
        env:
          JOB_CONTEXT: ${{ toJSON(job) }}
        run: echo "$JOB_CONTEXT"
      - name: Dump steps context
        env:
          STEPS_CONTEXT: ${{ toJSON(steps) }}
        run: echo "$STEPS_CONTEXT"
      - name: Dump runner context
        env:
          RUNNER_CONTEXT: ${{ toJSON(runner) }}
        run: echo "$RUNNER_CONTEXT"
      - name: Dump strategy context
        env:
          STRATEGY_CONTEXT: ${{ toJSON(strategy) }}
        run: echo "$STRATEGY_CONTEXT"
      - name: Dump matrix context
        env:
          MATRIX_CONTEXT: ${{ toJSON(matrix) }}
        run: echo "$MATRIX_CONTEXT"
```

### Using Functions in Expressions
- https://docs.github.com/en/actions/learn-github-actions/contexts#functions
- toJson()
- contains()
- startsWith()
- endsWith()
- format()

```
jobs:
  functions:
    runs-on: ubuntu-16.04
    steps:
      - name: dump
        run: |
          echo ${{ contains('hello', 'll') }}
          echo ${{ startsWith('hello', 'he') }}
          echo ${{ endsWith('hello', 'lo') }}
          echo ${{ format('hello {0} {1} {2}', 'world', '!', '!') }}
      - name: Dump GitHub context
        env:
          GITHUB_CONTEXT: ${{ toJSON(github) }}
```

### The if key & Job Status Check Functions
- https://docs.github.com/en/actions/learn-github-actions/contexts#job-status-check-functions

```
name: if
on: [push, pull_request]
jobs:
  functions:
    runs-on: ubuntu-16.04
    if: github.event_name == 'push'
    steps:
      - name: step 1
        run:
          ecco "$GITHUB_CONTEXT"
      - name: Dump job context
        if: failure() --> bu adım hata alırsa devamındaki adımlar çalışmaz.
        run: echo "something necessary"
      - name: Dump strategy context
        if: always() -> önceki adımlarda hata alan olsa bile çalışır.
        run: echo "$STRATEGY_CONTEXT"

```
## Bölüm 4 - Using Strategy, Matrix & Docker Containers in Jobs

### Continue on Error & Timeout Minutes
```
name: if
on: [push, pull_request]
jobs:
  functions:
    runs-on: ubuntu-latest
    timeout-minutes: 360 (360 default değer. 6 saat)
    steps:
      - name: echo a string
        run: echo "hello"
        continue-on-error: true
```

### Using the setup-node Action
- https://github.com/actions/setup-node

```
name: Matrix
on: push

jobs: 
  node-version:
    runs-on: ubuntu-latest
    steps:
      - name: Log node version
        run: node -v
      - uses: actions/setup-node@v1
        with:
          node-version: 10
      - name: Log node version
        run: node -v
```

### Creating a Matrix for Running a Job with Different Environments

```
name: Matrix
on: push

jobs: 
  node-version:
    strategy:
      matrix:
        os: [macos-latest, ubuntu-latest, windows-latest]
        node_version: [6,8,10]

      #fail-fast: true -> versiyonlardan biri hata alırsa doğrudan diğer versiyona geçsin
      #max-parallel: 0 -> eğer 3 node versiyonu aynı anda çalışsın istemiyorsan buraya paralel job sayısını belirtebilirsin. Bu parametre setlenmezse github actions hepsini paralel çalıştıracaktır.
    runs-on: ${{ matrix.os }}
    steps:
      - name: Log node version
        run: node -v
      - uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node_version }}
      - name: Log node version
        run: node -v
```

### Including & Excluding Matrix Configurations
```
jobs: 
  node-version:
    strategy:
      matrix:
        os: [macos-latest, ubuntu-latest, windows-latest]
        node_version: [6,8,10]
        include:
          - os: ubuntu-latest
            node_version: 8
            is_ubuntu_8: "true"
        exclude: 
          - os: ubuntu-latest
            node_version: 6
          - os: macos-latest
            node_version: 8
    runs-on: ${{ matrix.os }}
    env:
      IS_UBUNTU_8: ${{ matrix.is_ubuntu_8 }}
    steps:
      - name: Log node version
        run: node -v
      - uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node_version }}
      - name: Log node version
        run: | 
          node -v
          echo $IS_UBUNTU_8
```

### Using Docker Container in Jobs

__.github/workflows/container.yml__
```
name: Container
on: push

jobs: 
  node-docker:
    runs-on: ubuntu-latest
    container: 
      image: node:13.5.0-alpine3.10
      #env: 
      #ports:
      #volumes:
      #options: --cpus 1 --host ...
      steps:
        - name: Log node version
          run: |
            node -v
            cat /etc/os-release
```



### An Overview of a Simple Dockerized NodeJS API
- https://github.com/alialaa/simple-docker-nodejs-api


### Running Multiple Docker Services in Our Workflows

__.github/workflows/container.yml__
```
name: Container
on: push

jobs: 
  node-docker:
    runs-on: ubuntu-latest
    services: 
      app:
        image: alialaa17/node-api --> docker hub
        ports:
          - 3001:3000
      mongo:
        image: mongo
        ports:
          - "27017:27017"
    steps:
      - name: Post a user
        run: "curl -X POST http://localhost:3001/api/user -H 'Content-Type: application/json' -d '{\"username\": \"hello\", \"address\": \"asdasd\"}'"
      - name: get users
        run: curl http://localhost:3001/api/users
```

### Running Docker Containers in Individual Steps

__.github/workflows/container.yml__
```
name: Container
on: push

jobs:
  docker-steps:
    runs-on: ubuntu-latest
    container:
      image: node:10.18.0-jessie
    steps:
      - name: log node version
        run: node -v
      - name: step with docker
        uses: docker://node:12.14.1-alpine3.10
        with: 
          entrypoint: '/bin/echo'
          args: ['Hello World']
      - name: log node version
        uses: docker://node:12.14.1-alpine3.10
        with: 
          entrypoint: '/usr/local/bin/node'
          args: '-v'
  node-docker:
    runs-on: ubuntu-latest
    services: 
      app:
        image: alialaa17/node-api --> docker hub
        ports:
          - 3001:3000
      mongo:
        image: mongo
        ports:
          - "27017:27017"
    steps:
      - name: Post a user
        run: "curl -X POST http://localhost:3001/api/user -H 'Content-Type: application/json' -d '{\"username\": \"hello\", \"address\": \"asdasd\"}'"
      - name: get users
        run: curl http://localhost:3001/api/users
```


### Creating our Own Executable File and Running it in our Steps

__script.sh:__
```
#!/bin/sh
echo $1 $2
echo "Hello World"
```

before to push the script --> chmod +x script.sh  

__.github/workflows/container.yml__
```
name: Container
on: push

jobs:
  docker-steps:
    runs-on: ubuntu-latest
    container:
      image: node:10.18.0-jessie
    steps:
      - name: log node version
        run: node -v
      - name: step with docker
        uses: docker://node:12.14.1-alpine3.10
        with: 
          entrypoint: '/bin/echo'
          args: ['Hello World']
      - name: log node version
        uses: docker://node:12.14.1-alpine3.10
        with: 
          entrypoint: '/usr/local/bin/node'
          args: '-v'
      - uses: actions/checkout@1
      - name: run a script
        uses: docker://node:12.14.1-alpine3.10
        with: 
          entrypoint: './script.sh'
          args: 'some string'
  node-docker:
    runs-on: ubuntu-latest
    services: 
      app:
        image: alialaa17/node-api --> docker hub
        ports:
          - 3001:3000
      mongo:
        image: mongo
        ports:
          - "27017:27017"
    steps:
      - name: Post a user
        run: "curl -X POST http://localhost:3001/api/user -H 'Content-Type: application/json' -d '{\"username\": \"hello\", \"address\": \"asdasd\"}'"
      - name: get users
        run: curl http://localhost:3001/api/users
```

### Sending a Slack Message Using a Docker Container
- https://hub.docker.com/r/technosophos/slack-notify/

__.github/workflows/container.yml__
```
name: Container
on: push

jobs:
  docker-steps:
    runs-on: ubuntu-latest
    container:
      image: node:10.18.0-jessie
    steps:
      - name: log node version
        run: node -v
      - name: step with docker
        uses: docker://node:12.14.1-alpine3.10
        with: 
          entrypoint: '/bin/echo'
          args: ['Hello World']
      - name: log node version
        uses: docker://node:12.14.1-alpine3.10
        with: 
          entrypoint: '/usr/local/bin/node'
          args: '-v'
      - uses: actions/checkout@1
      - name: run a script
        uses: docker://node:12.14.1-alpine3.10
        with: 
          entrypoint: './script.sh'
          args: 'some string'
      - name: send a slack message
        uses: docker://technosophos/slack-notify
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }} --> webhook secret olmalı yoksa slack durumu anlayıp mesajı göndermiyor.
          SLACK_MESSAGE: "Hello Slack"
  node-docker:
    runs-on: ubuntu-latest
    services: 
      app:
        image: alialaa17/node-api --> docker hub
        ports:
          - 3001:3000
      mongo:
        image: mongo
        ports:
          - "27017:27017"
    steps:
      - name: Post a user
        run: "curl -X POST http://localhost:3001/api/user -H 'Content-Type: application/json' -d '{\"username\": \"hello\", \"address\": \"asdasd\"}'"
      - name: get users
        run: curl http://localhost:3001/api/users
```


## Bölüm 5 - Creating a CI/CD Workflow to Automate Testing and Deployment

### Creating a ReactJS Boilerplate Application
- https://create-react-app.dev/

### Creating a TypeScript App
``` npx create-reactt-app my-app --template typescript```

### Selecting a package manager
``` npx create-react-app my-app --use-npm ```

### Building & Testing the Application Locally

```
CI=true npm run test
CI=true npm run test -- --coverage 
CI=true npm run build
```

### Deploying the Application using Surge

Surge: Static web publishing for front-end developers

``` npm install --global surge ```

In your project root directory, just run the following command;

``` surge ```

### Using Prettier to Check for Code Formatting Rules
- https://prettier.io/

```
npm install --save-dev --save-exact prettier
npx prettier --check "**/*.js"
npx prettier --write "**/*.js"
```

### Workflow Plan

- Feature 1 -> Pull Request -> Run Workflow 1 -> OK? then Merge to Dev
  -> Develop -> Pull Request -> Run Workflow 2 -> OK? then Merge to Master -> Master -> Run Workflow 3
- Job Failure -> Create Issue -> Issue Created -> Send a Slack Message


__Workflow 1 Steps__
- Install Dependencies
- Check Code Formatting
- Run Automated Tests
- Upload Code Coverage as an Artifact
- Cache Dependencies

__Workflow 2 Steps__
- Install Dependencies
- Check Code Formatting
- Run Automated Tests
- Upload Code Coverage as an Artifact
- Build Project
- Upload Build as an Artifact
- Deploy to Staging Server
- Cache Dependencies

__Workflow 3 Steps__
- Install Dependencies
- Check Code Formatting
- Run Automated Tests
- Upload Code Coverage as an Artifact
- Build Project
- Upload Build as an Artifact
- Create a Release
- Deploy to Production Server
- Upload Coverage to Codecov

### Setting Up Our Repository

```
git init
git add -A
git commit -m"init"
git remote add origin git@github.com:alcnsahin/drmon-frontend.git
git push -u origin master
```

- create develop branch
- set branch rules:
- settings -> branches -> branch protection rules -> add rule -> branch name pattern=master, require pull request reviews before merging, Dismiss stale pull request approvals when new commits are pushed, Require review from Code Owners, Require status checks to pass before merging, Require branches to be up to date before merging, Include administrators -> save --> add the same rule for develop

npm install yerine npm ci komutu da kullanılabilir. Aşağıdaki dökümana bak;<br>
https://docs.npmjs.com/cli/v8/commands/npm-ci

```
git fetch
```

### create a new branch named workflow
```
git checkout -b workflow
```

__.github/workflows/ci.yml:__
```
  name: CI
  on:
    pull_request:
      branches:
        - develop
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: Use NodeJS
          uses: actions/setup-node@v1
          with:
            node-version: 14.x
        - name: run npm install
          run: npm install
          # run: npm run <code prettier lib>
          # run: npm run test
          # env:
          #   CI: true
```

```
git add -A
git commit -m"wf"
git push --set-upstream origin workflow
```

### Creating the Develop Merge Pull Request Workflow

```
  name: CI
  on:
    pull_request:
      branches:
        - develop
    push:
      branches:
        - develop
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: Use NodeJS
          uses: actions/setup-node@v1
          with:
            node-version: 14.x
        - name: run npm install
          run: npm install
          # run: npm run <code prettier lib>
          # run: npm run test
          # env:
          #   CI: true
        - name: build project
          if: github.event_name == 'push'
          run: npm run build
        #- name: deploy to staging server
        #  if: github.event_name == 'push'
        # run: deploy to test server / kubernetes or etc...
```

yukarıdaki ci.yml'da 2 tür requestte workflow tetiklenir. Bunlar pull_request ve push requesttir.

__Özetle;__
- Developer bir feature geliştirir.
- Bu feature'ı develop branch'i ile merge etmek isterse pull request girer.
- Pull request girdiği anda ci.yml devreye girer ve yukarıdaki adımlar sırayla işlenir ancak pull req olduğu için "build project" ve deploy to staging server adımları çalışmaz. Çünkü if condition eklenmiş ve yapılmış olan event "push" değil ise bu adımları çalıştırma denilmiş.
- Pull request onaylanıp feature develop ile merge edildiğinde ci.yml tekrar devreye girer ve bu sefer if condition sağlandığı için ilgili stepler de çalışır.

### Caching NPM Dependencies
__Caching:__
- https://docs.github.com/en/actions/advanced-guides/caching-dependencies-to-speed-up-workflows

__Cache Action:__
- https://github.com/actions/cache

```
  name: CI
  on:
    pull_request:
      branches:
        - develop
    push:
      branches:
        - develop
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: Cache node_modules
          uses: actions/cache@v1
          with:
            path: ~/.npm
            key: node-${{ runner.os }}-{{ hashFiles('**/package-lock.json') }}
            restore-key: |
              node-${{ runner.os }}
        - name: Use NodeJS
          uses: actions/setup-node@v1
          with:
            node-version: 14.x
        - name: run npm install
          run: npm ci
          # run: npm run <code prettier lib>
          # run: npm run test
          # env:
          #   CI: true
        - name: build project
          if: github.event_name == 'push'
          run: npm run build
        #- name: deploy to staging server
        #  if: github.event_name == 'push'
        # run: deploy to test server / kubernetes or etc...
```

### Uploading Artifacts in Our Workflows
Eğer test çalıştırılıyorsa ve bu test sonucunda istatistikler üretiildiyse yada build sonucunda çıkan dosyalar varsa ki olacak işte bu veriler artifact olarak geçer. Bu artifactleri indirebilmek için test ve/veya build sonrasında aşağıdaki gibi iki step eklemek gerekiyor.

__Özetle;__
- Proje build edilir
- Artifact upload edilir
- Artifact download edilir
- Bu işlemler github actions libs ile yapldı.

```
        - name: build project
          if: github.event_name == 'push'
          run: npm run build
        - name: upload Build(.output) Folder
          if: github.event_name == 'push'
          uses: actions/upload-artifact@v2
          with:
            name: build-artifacts
            path: .output
        - uses: actions/download-artifact@v2
          if: github.event_name == 'push'
```

### Semantic Versioning & Conventional Commits

- Semantic Versioning: __x.y.z__ (örn: 2.1.3) (https://semver.org/)
```
  x: Major Version (breaking changes)
  y: Minor Version (new features, non-breaking functionality)
  z: Patch Version (bug fixes)
```

- Conventional Commits: 
```
  <type>[optional scope]:<description>
  (https://www.conventionalcommits.org/en/v1.0.0/)
  [optional body]
  [optional footer]
```

Examples:
```
1-
fix(card):change card endpoint
BREAKNG CHANGE: changed card endpoint
closes issue #12

2-
feat(auth): added Facebook Authentication
docs(auth): added auth docs
ci: added a new workflow
style: updated documentation styles
```

### Installing semantic-release in Our Project
- https://github.com/semantic-release/semantic-release
- https://semantic-release.gitbook.io/semantic-release/

```
npm install --save-dev semantic-release
```

__release.config.js:__
```
module.exports = {
    branches: "master",
    repositoryUrl: "https://github.com/alcnsahin/endikasyonlar-frontend",
    plugins: [
        "@semantic-release/commit-analyzer",
        "@semantic-release/release-notes-generator",
        "@semantic-release/github"
    ]
}
```

Semantic versiyonlama sadece master'a merge edildiğinde çalışacak. Bu sebeple ci.yml aşağıdaki şekilde güncellendi.

__.github/workflows/ci.yml:__
```
  name: CI
  on:
    pull_request:
      branches: [develop, master]
    push:
      branches: [develop, master]
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: Cache node_modules
          uses: actions/cache@v1
          with:
            path: ~/.npm
            key: node-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
            restore-keys: node-${{ runner.os }}
        - name: Use NodeJS
          uses: actions/setup-node@v1
          with:
            node-version: 16.x
        - name: run npm install / ci
          run: npm ci
          # run: npm run <code prettier lib>
          # run: npm run test
          # env:
          #   CI: true
        - name: build project
          if: github.event_name == 'push'
          run: npm run build
        - name: upload Build(.output) Folder
          if: github.event_name == 'push'
          uses: actions/upload-artifact@v2
          with:
            name: build-artifacts
            path: .output
        - name: Create a Release
          if: github.event_name == 'push' && github.ref == 'refs/heads/master'
          run: npx semantic-release
          env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        - uses: actions/download-artifact@v2
          if: github.event_name == 'push'
        #- name: deploy to staging server
        #  if: github.event_name == 'push'
        # run: deploy to test server / kubernetes or etc...
```

```
$ git commit -m"asd"
$ #git reset HEAD~
$ #git rm -r --cached .husky
$ git checkout master
$ git merge workflow
$ git push
```


### Uploading Release Assets

__release.config.js:__
```
module.exports = {
    branches: "master",
    repositoryUrl: "https://github.com/alcnsahin/endikasyonlar-frontend",
    plugins: [
        "@semantic-release/commit-analyzer",
        "@semantic-release/release-notes-generator",
        ["@semantic-release/github", {
            assets: [
                { path: "build.zip", label: "Build" }
            ]
        }]
    ]
}
```

__ci.yml:__
```
- name: ZIP Artifacts
  if: github.event_name == 'push' && github.ref == 'refs/heads/master'
  run: |
    pwd
    ls -la
    zip -r build.zip ./.output
- name: Create a Release
  if: github.event_name == 'push' && github.ref == 'refs/heads/master'
  run: npx semantic-release
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```


### Deploying to Production when Pushing to Master
```
if: github.event_name == 'push' && github.ref == 'refs/heads/master' run: deploy to kubernetes or etc...(https://surge.sh/)
```

### Uploading Code Coverage Reports to Codecov
- https://about.codecov.io/

Test sonucunda çıkan istatistik ve raporlarını codecov'a gönderip analiz edilmesini sağlayabilirsin.

### Validating Our Commit Messages with Commitlint & Commitizen

- https://github.com/conventional-changelog/commitlint
- https://github.com/commitizen/cz-cli
- https://typicode.github.io/husky/#/

CommitLint ve Husky kullanacağız:

Kurulum:
```
$ npm install --save-dev @commitlint/{config-conventional,cli} husky
$ echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
$ npx husky install
$ npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
```

### Send Events to Slack
- https://api.slack.com/
get webhook

### Adding a Status Badge in README.md
```
![](https://github.com/alcnsahin/drmon-frontend/workflows/CI/badge.svg?branch=develop&event=push)
```

## Bölüm 6 - Creating Our Own Github Actions

### Creating a Simple Javascript Action

__Installation:__
- https://github.com/actions/toolkit
```
$ npm install @actions/core
$ npm install @actions/github
```

__.github/actions/hello/action.yml:__
```
name: Hello World
author: Alican Sahin
description: Some description
inputs:
  who-to-greet:
    description: 'who to greet'
    required: true
    default: Alican
outputs:
  time:
    description: 'The greeting time'
runs:
  using: "node16"
  main: "dist/index.js"
```

__.github/actions/hello/index.js:__
```
const core = require('@actions/core');
const github = require('@actions/github');

try{
// throw new Error("Some Error Message!");
const name = core.getInput('who-to-greet')
console.log(`Hello ${name}`)

const time = new Date();
core.setOutput("time", time.toTimeString());

console.log(JSON.stringify(github, null, '\t'));
}catch(error){
  core.setFailed("Error Message");
}
```

__.github/workflow/custom-action.yml:__
```
on: push
jobs: 
  testing-action:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: ./.github/actions/hello
        id: hello
        with:
          who-to-greet: "World"
      - run: |
          echo "Time: ${{ steps.hello.outputs.time }}"
```

### Bundling Our Javascript Code into 1 File
__Installation:__
```
npm i -D @zeit/ncc
```

__Compile:__
```
npx ncc build .github/actions/hello/index.js -o .github/actions/hello/dist/
```

### Let's Discover More About the @github/core Package
- https://github.com/actions/toolkit/tree/master/packages/core

```
const core = require("@actions/core");

core.debug("Debug message");
core.warning("Warning message");
core.error("Error message");

const name = core.getInput("who-to-greet");
core.setSecret(name) // won't show on the logs ******
console.log(`Hello ${name}`);

// start a logging group
core.startGroup("Logging github object");
console.log(JSON.stringify(github, null, "\t"));
core.endGroup()

// set env var
core.exportVariable('HELLO', 'Hello World');

/*
env setledikten sonra custom-action.yml içerisinde aşağıdaki şekilde okunabilir.
on: push
jobs: 
  testing-action:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: ./.github/actions/hello
        id: hello
        with:
          who-to-greet: "World"
      - run: |
          echo "Time: ${{ steps.hello.outputs.time }}"
          echo $HELLO
*/
```


### Creating a Javascript Action for Opening Github Issues
- https://octokit.github.io/rest.js/v18#issues

__.github/actions/issues/action.yml:__
```
name: Open Github Issues
author: alcnsahin
description: Opens a github issue
inputs:
  token:
    description: 'Github Token'
    required: true
  title:
    description: Issue Title
    required: true
  body: 
    description: Issue Body
  assignees:
    description: Issue Assignees
outputs:
  issue:
    description: 'The issue object as a json string'
runs:
  using: "node16"
  main: "dist/index.js"
```

__.github/actions/issues/index.js:__
```
const core = require('@actions/core');
const github = require('@actions/github');

async function run(){
  try{
    const token = core.getInput('token');
    const title = core.getInput('title');
    const body = core.getInput('body');
    const assignees = core.getInput('assignees');
    
    const octokit = new github.Github(token);
    const response = await octokit.rest.issues.addAssignees({
      // owner: github.context.repo.owner,
      // repo: github.context.repo.owner,
      ...github.context.repo,
      title,
      body,
      assignees
    });

    core.setOutput('issue', JSON.stringify(response.data));

  }catch(error){
    core.setFailed("Error Message");
  }
}

run();
```


__.github/workflow/issues-action.yml:__
```
on: push
jobs: 
  testing-action:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: ./.github/actions/issues
        id: issue
        with: 
          token: ${{ secrets.GITHUB_TOKEN }}
          title: title
          body: body
          assignees: ['alcnsahin','velican']
      - run: |
          echo ${{ steps.issue.outputs.issue }}
      - uses: ./.github/actions/hello
        id: hello
        with:
          who-to-greet: "World"
      - run: |
          echo "Time: ${{ steps.hello.outputs.time }}"
```

```
npx ncc build .github/actions/issue/index.js -o .github/actions/issue/dist
```


### Creating a Simple Docker Action
- https://docs.github.com/en/actions/learn-github-actions/workflow-commands-for-github-actions

__.github/actions/hello-docker/action.yml:__
```
name: Open Github Issues
author: alcnsahin
description: Opens a github issue
inputs:
  token:
    description: 'Github Token'
    required: true
  title:
    description: Issue Title
    required: true
  body: 
    description: Issue Body
  assignees:
    description: Issue Assignees
outputs:
  issue:
    description: 'The issue object as a json string'
runs:
  using: "docker"
  main: "dockerfile"
  args:
    - ${{inputs.who-to-greet}}
```


__.github/actions/hello-docker/Dockerfile__
```
FROM alpine:3.11
COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ['']
```

__.github/actions/hello-docker/entrypoint.sh__
```
#!/bin/sh -l

#if [ true ]
#then
#  echo 'error'
#  exit 1
#fi

echo "::debug ::Debug Message"
echo "::warning ::Warning Message"
echo "::error ::Error Message"
echo "::add-mask::$1"
echo "Hello $1"
time=$(date)
echo "::set-output name=time::$time"

echo "::group::Some expandable logs"
echo "some stuff"
echo "some stuff"
echo "some stuff"
echo "some stuff"
echo "::endgroup::"

echo "HELLO=hello" >> $GITHUB_ENV
```

__.github/actions/hello-docker/index.js:__
```
const core = require("@actions/core");

core.debug("Debug message");
core.warning("Warning message");
core.error("Error message");

const name = core.getInput("who-to-greet");
core.setSecret(name) // won't show on the logs ******
console.log(`Hello ${name}`);

// start a logging group
core.startGroup("Logging github object");
console.log(JSON.stringify(github, null, "\t"));
core.endGroup()

// set env var
core.exportVariable('HELLO', 'Hello World');
```


.github/workflow/hello-docker.yml:
```
on: push
jobs: 
  testing-action:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: ./.github/actions/hello-docker
        id: hello
        with:
          who-to-greet: "World"
      - run: |
          echo "Time: ${{ steps.hello.outputs.time }}"
          echo $HELLO
```

```
$ chmod +x .github/actions/hello-docker/entrypoint.sh
$ npx ncc build .github/actions/hello-docker/index.js -o .github/actions/hello-docker/dist
```

### Publishing Github Actions into the Marketplace
- __Example:__ https://github.com/alialaa/open-issue
- __Action Versioning:__ https://github.com/actions/toolkit/blob/master/docs/action-versioning.md
- __icons:__ https://feathericons.com/

__Steps__:
- go to https://github.com/actions/javascript-action
- click "use this template" button
- set repo name and fill the others
- git clone your new repo to local machine
- open the project in IDE
```
$ npm install
```

Action'a icon eklemek için feathericons'dan icon seç ve adını branding'e yaz. Aşağıdaki yaml'da en alt satırı incele.

__.github/actions/issues/action.yml:__
```
name: Open Github Issues
author: alcnsahin
description: Opens a github issue
inputs:
  token:
    description: 'Github Token'
    required: true
  title:
    description: Issue Title
    required: true
  body: 
    description: Issue Body
  assignees:
    description: Issue Assignees
outputs:
  issue:
    description: 'The issue object as a json string'
runs:
  using: "node16"
  main: "dist/index.js"
branding:
  icon: 'alert-octagon'
  color: 'purple'
```

- kodu düzenleyip pushladıktan sonra repoya girdiğinde karşına "Publish this Action to Marketplace" mesajını göreceksin.
- "Draft a Release" butonuna bas
- Hata varsa uyarılar gelecek, düzenle!
- "primary category" seç
- "another category" seç
- versiyon gir
- publish butonuna bas.
- versiyonlarken v1.0.0 verildiyse action çağırılırken aynı şekilde tag girilmeli aksi taktirde workflow hata alır
- bunu önlemek için aşağıdaki adımlarla v1.0.0 versiyonu v1 şeklinde tekrar taglenir.

```
git tag -fa -v1 -m "Update v1 tag"
git push origin v1 --force
```

### Slack Message Builder
- https://app.slack.com/block-kit-builder