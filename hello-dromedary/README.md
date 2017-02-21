# hello-dromedary <img align="right" src="../img/liatrio.png">
Chico Tech. Talk 2017

  Overview:

  1. Hello Dromedary
  2. Run Locally
  3. Test the Application
  4. Integrate Travis CI
  5. Integrate Heroku

## 1. Hello Dromedary

#### Make a personal fork of dromedary
Create a fork of the [liatrio/dromedary repo](http://github.com/liatrio/dromedary.git) on github, and clone it to your local machine.

#### Set up your environment
Java must be installed on your machine to run DynamoDB Local can run. This is the database that our app relies upon.  
  
Ensure that you have [Node JS](https://nodejs.org/en/download/) installed as well.

Install Dependencies by running `npm install` in project's root directory.

Install gulp with `npm install -g gulp` If you already have gulp installed ensure that `./node_modules/.bin/` is in your PATH

#### Install Ruby
Ruby is a requirement for Travis CLI, which we will use later in the lab. To install Ruby

- For Mac using Homebrew [http://brew.sh/](http://brew.sh/)

```bash
brew install ruby
```

- For Ubuntu using RVM [https://rvm.io/](https://rvm.io/)

```bash
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update && sudo apt-get upgrade
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
rvm install 2.3
```

## 2. Run Locally
To start dromedary, simply run `gulp` in the project's root directory.

You can access dromedary at [localhost:8080](http://localhost:8080)

To supress output to the terminal you can run dromedary n the background with `nohup gulp &`

You can specify the port dromedary runs on with the following optional parameter `PORT=<port_number> gulp` to run on port 1337 simply enter `PORT=1337 gulp`

## 3. Test the Application
Unit tests in the `test/` directory are written using Mocha and Chai.

To run unit tests, execute the test task: `gulp test`

## 4. Integrate Travis CI

#### Register for Travis CI
  Travis CI is a continuous integration service that is free for open source GitHub repositories. If you don’t already have an account, go to [https://travis-ci.org/](https://travis-ci.org/) and register with your GitHub account. Travis CI will watch your GitHub repositories and can be configured to build/deploy your projects when they recieve new commits. After registering, head to travis-ci.org/profile and toggle on your fork of the Dromedary repository. If you don't see your repository, trying clicking "Sync Account"


#### Install Travis CLI
  The following will install Travis Client, a command line utility for Travis CI (requires Ruby, see Ch. 2). This client provides many tools for monitoring and managing Travis CI jobs. Refer to [https://github.com/travis-ci/travis.rb#installation](https://github.com/travis-ci/travis.rb#installation) for more help with installing Travis CLI.

  ```
  gem install travis -v 1.8.5 --no-rdoc --no-ri
  ```

#### Create a `.travis.yml` Configuration File
  To instruct Travis CI on how to build, test, and deploy this application, we need to place a `.travis.yml` configuration file in the base directory of the dromedary project.

  ```
  language: node_js
  node_js:
    - "7.2"
  beforescript:
    - npm install -g gulp
    - npm install
  script:
    gulp package-app
  ```

  This will specify the language, version, and setup commands to build this project. Dromedary uses Gulp and has conveniently defined a gulp task `package-app` to build and test itself. Commit `.travis.yml` and push to remote. This will trigger Travis CI to run the build. For more information on creating a `.travis.yml` for other types of projects, refer to the Travis CI documentation: [https://docs.travis-ci.com/user/customizing-the-build](https://docs.travis-ci.com/user/customizing-the-build).

  Commit and push .travis.yml to GitHub

## 4. Integrate Heroku
#### Register on Heroku
  Heroku is a service used to deploy and maintain applications. If you don't already have an account, register for one at [https://signup.heroku.com](https://signup.heroku.com).

#### Install Heroku CLI
  Run these commands to install Heroku Command Line Interface. This will be used to create and deploy applications from the Ubuntu instance. Refer to [https://devcenter.heroku.com/articles/heroku-cli#download-and-install] (https://devcenter.heroku.com/articles/heroku-cli#download-and-install) for further information.
  ```
  sudo add-apt-repository "deb https://cli-assets.heroku.com/branches/stable/apt ./"
  curl -L https://cli-assets.heroku.com/apt/release.key | sudo apt-key add -
  sudo apt-get update
  sudo apt-get install heroku
```

#### Create a Heroku App
    Run the following to create a heroku app through the Heroku CLI, where `app_name` is a unique name of your choosing. Further information on creating apps is available in the [official Heroku documentation](https://devcenter.heroku.com/articles/creating-apps).
    ```bash
    heroku create app_name
    ```
 If an `app_name` isn't specified, one will be generated. It will also output the url at which the application will be hosted.

#### Configure Travis CI Deployment
  Add the following `deploy` configuration to your `.travis.yml` file where `app_name` is the name of your application.

  ```
  language: node_js
  node_js:
    - "7.2"
  beforescript:
    - npm install -g gulp
    - npm install
  script:
    gulp package-app
  deploy:
    provider: heroku
    app: <app_name>
  ```

  Then generate a Heroku auth token and encrypt it using the Travis CLI:

  ```sh
  travis encrypt $(heroku auth:token) --add deploy.api_key
  ```

  This semi-secure auth token will allow Travis to deploy this project to the heroku app created in the previous step.

#### Create A Procfile
  Lastly, create a Procfile that Heroku will use to execute the application. This will instruct Heroku to execute `npm start` on a free-tier web dyno.

  *Procfile*
  ```Procfile
  web: npm start
  ```

#### Commit Changes
  Committing and pushing these changes to GitHub will trigger a build job on Travis CI that will deploy to Heroku on success.

  ```sh
  git add .
  git commit -m "Setup CICD"
  git push origin master
  ```

## 5. Confirm that it worked
  Pushing this commit to GitHub should trigger the travis-ci dromedary job. Confirm that the job is running by navigating to the Dromedary job a http://travis-ci.org. After a few minutes the job should successfully complete which means that Dromedary was built, tested, and deployed to a Heroku dyno without error. Navigate to your Heroku app's url to confirm that Dromedary is live (if you forgot it, it can be found in the "Settings" tab of your app on the Heroku dashboard).


#### See it in Action
  Let's make a change to Dromedary and watch it make its way through the pipeline. Edit *lib/inMemoryStorage.js* and update `chartData` to add new color choices:

  ```
  var chartData = {
    values: {
      darkblue: {
        label: 'DarkBlue',
        value: 10,
        color:'#000066',
        highlight: '#6F6F6F'
      },
      red: {
        label: 'Red',
        value: 10,
        color: '#CC0000',
        highlight: '#C9DF6E'
      },
      yellow: {
        label: 'Yellow',
        value: 10,
        color:'#FF9900',
        highlight: '#FFB75E'
      }
    }
  };
    ```
  Now commit and push the changes so Travis will build and deploy the update.
