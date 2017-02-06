[![Build Status](https://travis-ci.com/shanemacbride/techTalk17.svg?token=vjqBpF5Ke4q3CszpjJsP&branch=master)](https://travis-ci.com/shanemacbride/techTalk17)  
# hello-ci
Chico Tech. Talk 2017
  
  Overview:  
  
  1. Hello Ruby
  2. Run Locally
  3. Test the Application
  4. Integrate Travis CI
  5. Integrate Heroku
  
## 1. Hello Ruby  
  
### Initialize a GitHub Repository
Create a public GitHub repository for this exercise and name it helloCI. Clone this repository and navigate to it on your local machine.  
  
### Install Ruby
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
 
### Create Files
  1. Create `app.rb`.
  
  ```ruby
  require 'sinatra'
  class HelloWorld < Sinatra::Base
    get '/' do
      "Hello, world!"
    end
    get '/:name' do
      "Hello, #{params[:name]}!"
    end
  end
  ```  
 
  2. Create `config.ru`.
 
  ```ruby
  require './app'
  run HelloWorld
  ```  
 
### Manage Dependencies with Bundler
  1. Install bundler.  
  
  ```bash
  gem install bundler
  ```
  
  2. Create `Gemfile`.  
  
  ```ruby
  source 'https://rubygems.org'
  gem 'sinatra'
  gem 'minitest'
  gem 'rack-test'
  ```
  
  3. Install from the Gemfile with `bundle install`.
  
## 2. Run Locally  
1. Enter the following command to test the application using rackup.  
  
  ```bash
  rackup -o 0.0.0.0
  ```  
  
2. Using a web browser, navigate to [http://localhost:9292](http://localhost:9292). "Hello, world!" should be displayed in the browser. 
   
3. [http://localhost:9292/foo](http://localhost:9292) can also be used. “Hello, foo” should be output to the page; the application takes whatever is in the first directory in the path component and appends that to the “Hello” statement. We will use this for testing.  
  
## 3. Test the Application  
  1. Create `test.rb`.
  
  ```ruby
  require './app.rb'
  require 'minitest/autorun'
  require 'rack/test'
  
  class MyAppTest < Minitest::Test
    include Rack::Test::Methods
    
    def app
      HelloWorld
    end
    
    def test_my_default
      get '/'
      assert_equal 'Hello, world!', last_response.body
    end
    
    def test_with_params
      get '/Frank'
      assert_equal 'Hello, Frank!', last_response.body
    end
  
  end
  ```  
  
  2. Run `ruby test.rb`.
  3. The output should be similar to the following.
  
  ```bash
  Run options: --seed 13471
  # Running:
  ..
  Finished in 0.015424s, 129.6701 runs/s, 129.6701 assertions/s.
  2 runs, 2 assertions, 0 failures, 0 errors, 0 skips
  ```
  
  4. Finally, commit the code and push it up to the GitHub repository.  
  
  ```bash
  git add .
  git commit -m “Initial app commit”
  git push origin master
  ```
  
## 4. Integrate Travis CI  
Travis CI is a continuous integration service that is free for open source GitHub repositories.  
  
  1. Register for Travis CI.  
If you don’t already have an account, go to [https://travis-ci.org/](https://travis-ci.org) and register with your GitHub account. Travis CI will sync your GitHub repositories and can be configured to build/deploy your projects when they recieve new commits.  
  
  2. Enable helloCI in Travis CI.  
After registering, head to [https://travis-ci.org/profile](https://travis-ci.org/profile) and toggle on your helloCI repository. If you don't see your repository, try clicking "Sync Account."  
  
  3. Install Travis CLI.  
The following will install Travis Client, a command line utility for Travis CI. This client provides many tools for monitoring and managing Travis CI jobs. Refer to [https://github.com/travis-ci/travis.rb#installation](https://github.com/travis-ci/travis.rb#installation) for more help with installing Travis CLI.  
  
  ```bash
  gem install travis -v 1.8.5 --no-rdoc --no-ri
  ```  
  
  4. Create a `.travis.yml` file.  
  To instruct Travis CI on how to build, test, and deploy this application, we need to place a `.travis.yml` configuration file in the base directory of the helloCI project.  
  
  ```yml
  notifications:                                                                     
    email:                                                                           
      on_success: always # default: change                                           
      on_failure: never  # default: always
  language: ruby                                                                     
  rvm:                                                                               
    - 2.3.2                                                                          
  script:                                                                            
    - bundle exec ruby test.rb 
  ```  
  
  This will specify the notifications, language, version, and setup commands to build this project. Commit `.travis.yml` and push to GitHub. This will trigger Travis CI to run the build.  
  
  5. Navigate to the build page on Travis CI and click the "Build Passing" to reveal options for pasting a Build icon. Select "Markdown" and paste the snippet into your README.md in the helloCI repository. Commit and push the code to GitHub.  
  
## 5. Integrate Heroku  
Heroku is a service used to deploy and maintain applications.  
  
  1. Register on Heroku.  
  If you don't already have an account, register for one at [https://signup.heroku.com](https://signup.heroku.com).
  2. Install Heroku CLI.  
  On Mac, download the installer from [https://cli-assets.heroku.com/branches/stable/heroku-osx.pkg](https://cli-assets.heroku.com/branches/stable/heroku-osx.pkg). On Ubuntu, run these commands to install Heroku Command Line Interface. This will be used to create and deploy applications from the Ubuntu instance. Refer to [https://devcenter.heroku.com/articles/heroku-cli#download-and-install](https://devcenter.heroku.com/articles/heroku-cli#download-and-install) for further information.  
  
  ```bash
  sudo add-apt-repository "deb https://cli-assets.heroku.com/branches/stable/apt ./"  
  curl -L https://cli-assets.heroku.com/apt/release.key | sudo apt-key add -  
  sudo apt-get update
  sudo apt-get install heroku
  ```
  
  3. Create a Heroku Application.  
  Run the following to create a Heroku Application through the Heroku CLI. `app_name` is a unique name of your choosing.  
  
  ```bash
  heroku create app_name
  ```
  
  If an `app_name` isn't specified, one will be generated. It will also output the URL at which the application will be hosted.  
  
  4. Configure Travis CI Deployment.  
  Add the following _deploy_ configuration to your `.travis.yml` file where `app_name` is the name of your application.  
  
  ```yml
  notifications:                                                                     
    email:                                                                           
      on_success: always # default: change                                           
      on_failure: never  # default: always
  language: ruby                                                                     
  rvm:                                                                               
    - 2.3.2                                                                          
  script:                                                                            
    - bundle exec ruby test.rb 
  deploy:
    provider: heroku
    app: <app_name>
  ```
  
  Then, generate a Heroku auth. token and encrypt it using the Travis CLI.  
  
  ```bash
  travis encrypt $(heroku auth:token) --add deploy.api_key
  ```
  
  This semi-secure auth. token will allow Travis to deploy this project to the Heroku app. created in the previous step.
  
  5. Create A Procfile.  
  Lastly, create a Procfile that Heroku will use to execute the application. This will instruct Heroku to execute `rackup` on a free-tier web dyno.  
  
  Create a file named `Procfile` and input:  
  
  ```bash
  rackup
  ```  
  
  6. Commit Changes.  
  Committing and pushing these changes to GitHub will trigger a build job on Travis CI that will deploy to Heroku on success.  
  
  ```bash
  git add .
  git commit -m "Setup CI"
  git push origin master
  ```
  
  7. Confirm that it worked.  
  Pushing this commit to GitHub should trigger the Travis CI helloCI job. Confirm that the job is running by navigating to the helloCI job a [http://travis-ci.org](http://travis-ci.org). After a a minute or so, the job should successfully complete which means that helloCI was built, tested, and deployed to a Heroku dyno without error. Navigate to your Heroku app's URL, [https://app-name.herokuapp.com/](https://app-name.herokuapp.com/), to confirm that helloCI is live (if you forgot it, it can be found in the "Settings" tab of your app on the Heroku dashboard).
