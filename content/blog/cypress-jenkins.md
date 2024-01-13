---
title: "Cypress Parallelization on Jenkins using Sorry-Cypress"
date: 2021-11-28
cascade:
  showReadingTime: false
---
<img src="imgg/001-uml-describe.png"/>
UML describing the flow

My favorite way of writing an e2e test for my UI is using cypress. Cypress makes things very simple and easy to write tests with.

The problem I found with writing tests with cypress is that it takes too long for the test to complete. This in turn creates friction for new developers to do a full-blown e2e test run.
<img src="https://miro.medium.com/max/700/1*ubufLTQ-htrHFWu2CSWiRQ.png"/>
Cypress running for 40 mins

The fix to ensure we run all our tests regularly to run the test in parallel. Thankfully Jenkins (or any modern CI ) supports running tasks in parallel (https://www.jenkins.io/blog/2017/09/25/declarative-1/).
```
stage('run test') {
  parallel {
   stage('tester A') {
    steps {
     sh "npm run cy:run:parallel"
    }
   }
   // second tester runs the same command
   stage('tester B') {
    steps {
     sh "npm run cy:run:parallel"
    }
   }
   // third tester runs the same command
   stage('tester C') {
    steps {
     sh "npm run cy:run:parallel"
    }
   }
   // fourth tester runs the same command
   stage('tester D') {
    steps {
     sh "npm run cy:run:parallel"
    }
   }
  }
```
The code above will ensure the tests are run in parallel. Cypress io offers a bunch of plans including a free plan to record the test. The reason why we need to record the test is that cypress takes over the process of dividing the test for each machine.

After doing this change, our tests now run in less than 12mins (from earlier 40mins).

Due to security concerns, I decided that it’s better to host the dashboard instead of using cypress io. The dashboard will help us run tests in parallel.

I came across this project Sorry Cypress project which is an open-source alternative to cypress.io paid dashboard.
The project was simple to set up since all it involves is running docker-compose up ( Since we are on Kubernetes I had to use kompose to convert docker compose to a helm chart)

Once I had it up and running, all I had to do is to change cypress API path to point to the sorry cypress instance
<image src="https://miro.medium.com/max/700/1*ZRR4qC7UXfokegqkH_kMnw.png"/>
Sorry Cypress dashboard
<image src="https://miro.medium.com/max/700/1*xL2nuFfKwPlt2cFJl-NUrA.png"/>
Failed Test screenshot
```
def CYPRESS_VERSION="\$(npm show cypress version)"def CYPRESS_CONFIG_FILE_PATH="\$(npx cypress cache path)/${CYPRESS_VERSION}/Cypress/resources/app/packages/server/config/app.yml"
          
sh "sed -i 's/https:\\/\\/api.cypress.io\\//https:\\/\\/sorry-cy.internal.adityat.com/g' $CYPRESS_CONFIG_FILE_PATH"
```
I used this as step 1 of the Build process, right after I do npm install. This will point cypress instance running on Jenkins to the internally hosted cypress dashboard.

To group your tests you can change the CI build command to below, so your test will appear as “master-1”, “master-2”
```
sh "npm run cy:run:parallel -- --ci-build-id ${env.BRANCH_NAME}-${env.BUILD_ID}"
```
One thing that may not work with parallelization is the ability to report code coverage.

