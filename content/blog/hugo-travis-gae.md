+++
description = "Setup instructions on to host your hugo site on Google App Engine via Travis CI"
title = 'Host your Hugo site on Google App Engine via Travis CI'
date = 2018-12-23T19:59:05+05:30
draft = false
tags = ["Hugo", "Travis", "GAE"]
toc = true
+++

# Setup Hugo Site

 Hugo has excellent documentation on how to setup up a site on your local machine. You can quickly get a basic website running locally by following their [instructions](https://gohugo.io/getting-started/quick-start/).

 Note: I use the [cocoa-eh](https://github.com/mtn/cocoa-eh-hugo-theme) theme. It has an excellent [wiki](https://github.com/mtn/cocoa-eh-hugo-theme/wiki) page on github. This is how my [config.toml](https://github.com/rahulmanthena/blog/blob/master/config.toml) looks like if your are interested.

# Setup Github repo

Initialize git in your root directory and make your initial commit . Now create a new repo for your website on github and push your local code to github.

# Setup Google Appengine
Follow the [instructions](https://cloud.google.com/resource-manager/docs/creating-managing-projects) on Appengine documentation website and create a new standard google appengine project. Get the gcloud cli [here](https://cloud.google.com/sdk/docs/#install_the_latest_cloud_tools_version_cloudsdk_current_version) for testing locally(This is entirely optional but I recommend it so you have instant feedback instead of waiting for Travis build to complete.)

Add an ```app.yaml``` and ```app.go``` files to your root dir. They should look something like this. We are serving the ```public``` directory created by hugo.

  ```app.yaml```

````yaml
runtime: go
api_version: go1

handlers:
- url: /.*
  script: _go_app
  secure: always
````

  ```app.go```

````go
package blog

import "net/http"

func init() {
  http.Handle("/", http.FileServer(http.Dir("public")))
}
````

# Setup TravisCI

Sign in to TravisCI with your Github account and enable Travis for your repo. Now create  ```.travis.yml``` in your root directory and configure it as below.

[TravisCI's documentation](https://docs.travis-ci.com/user/deployment/google-app-engine/) on how to deploy to Appengine are good but I needed to change a one command to make it work. When encrypting the ```client-secret.json``` file, just add ```--com``` to the command so it should look like this.

```` bash

$ travis encrypt-file client-secret.json --add --com

````
This will add the decryption step to ```.travis.yml``` file in the ```before_install``` section.

````yaml
- openssl aes-256-cbc -K $encrypted_5e8948859fb9_key 
  -iv $encrypted_5e8948859fb9_iv 
  -in client-secret.json.enc 
  -out client-secret.json -d
````

Your completed .travis.yml should look like this.
````yml
language: go
go:
- 1.11.x

before_install:
- openssl aes-256-cbc -K $encrypted_5e8948859fb9_key 
  -iv $encrypted_5e8948859fb9_iv
  -in client-secret.json.enc 
  -out client-secret.json -d
- mkdir $HOME/src
- cd $HOME/src
- git clone https://github.com/gohugoio/hugo.git
- cd hugo
- go install --tags extended
- cd ${TRAVIS_HOME}/gopath/src/github.com/rahulmanthena/blog

deploy:
  provider: gae
  keyfile: client-secret.json
  project: rahulmanthena
  skip_cleanup: true
````

Add a makefile to your root dir to run Hugo in verbose mode. For good measure, add a clean step too. The clean step will allow you to run make clean and remove your local generated site.

Try running ```make``` from the root dir of your repo. You’ll see that hugo generates a public dir with all your static files.

````make
run:
  hugo --verbose
clean:
  rm -rf public
````

The clean step will allow you to run make clean and remove your local generated site.

Try running make from the root dir of your repo. You’ll see that hugo generates a public dir with all your static files.

# Congrats

Push your local repo to github and a Travis build should kickoff automatically. Once the build is successful your app should be running on Google App Engine.Your site should be redeployed by TravisCI every time you push new commits to Github.

> Huge thanks to this [guide](https://taymor.io/posts/deploying-a-blog-with-hugo-travis-and-gae/) which helped me in setting up this site.
  
  
If you find any trouble following along just leave a comment below.