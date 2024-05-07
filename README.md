# Golden Owl DevOps Internship - Technical Test
At Golden Owl, we believe in treating infrastructure as code and automating resource provisioning to the fullest extent possible. 

In this technical test, we challenge you to create a robust CI build pipeline using GitHub Actions. You have the freedom to complete this test in your local environment.

## Your Mission 🌟
Your mission, should you choose to accept it, is to craft a CI job that:
1. Forks this repository to your personal GitHub account.
2. Dockerizes a Node.js application.
3. Establishes an automated CI/CD build process using GitHub Actions workflow and a container registry service such as DockerHub or Amazon Elastic Container Registry (ECR) or similar services.
4. Initiates CI tests automatically when changes are pushed to the feature branch on GitHub.
5. Utilizes GitHub Actions for Continuous Deployment (CD) to deploy the application to major cloud providers like AWS EC2, AWS ECS or Google Cloud (please submit the deployment link).
## Nice to have 🎨
We would be genuinely delighted if you could complement your submission with a `visual flow diagram`, illustrating the sequence of tasks you performed, including the implementation of a `load balancer` and `auto scaling` for the deployed application. This additional touch would greatly enhance our understanding and appreciation of your work.

Reference tools for creating visual flow diagrams:
- https://www.drawio.com/
- https://excalidraw.com/
- https://www.eraser.io/
  
Including a visual representation of your workflow will provide valuable insights into your approach and make your submission stand out. Thank you for considering this enhancement! 
## The Bigger Picture 🌏
This test is designed to evaluate your ability to implement modern automated infrastructure practices while demonstrating a basic understanding of Docker containers. In your solution, we encourage you to prioritize readability, maintainability, and the principles of DevOps.

 ## Submission Guidelines 📬
Your solution should be showcased in a public GitHub repository. We encourage you to commit early and often. We prefer to see a history of iterative progress rather than a single massive push. When you've completed the assignment, kindly share the URL of your repository with us.

 ## Running the Node.js Application Locally  🏃‍♂️
 This is a Node.js application, and running it locally is straightforward:
- Navigate to the `src` directory by executing `cd src`.
- Install the project's dependencies listed in the package.json file by running `npm i`.
- Execute `npm test` to run the application's tests.
- Start the HTTP server with `npm start`.

You can test it using the following command:
  
```shell
curl localhost:3000
```
You should receive the following response:
```json
{"message":"Welcome warriors to Golden Owl!"}
```

Are you ready to embark on this DevOps journey with us? 🚀 Best of luck with your assignment! 🌟


## 1. Forks this repository to your personal GitHub account

Link : [GitHub - gangi2424/goldenowl-devops-internship-challenge](https://github.com/giang2424/goldenowl-devops-internship-challenge)

## 2. Dockerizes a Node.js application

The Dockerfile utilizes the previously built base image to build a new Node.js image.

```Dockerfile
#### Docker file ####
FROM node:10-alpine

RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app
WORKDIR /home/node/app
COPY package*.json ./
USER node
RUN npm install
COPY --chown=node:node . .

EXPOSE 3000
CMD [ "npm", "start" ]
```

## 3. Establishes an automated CI/CD build process using GitHub Actions workflow and a container registry service DockerHub

### Step 1: Add secret variables for git action

We add the following variables secret:
```
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

### Step 2: Config .yml workflow file

```yaml
name: CI/CD build

on:
  workflow_dispatch:
  push:
    branches:
      - "*"
    paths:
      - 'src/**'
      - '.github/workflows/main.yml'
      
jobs:
  build:
    runs-on: ubuntu-latest
    # needs: test

    steps:
      - name: Check out
        uses: actions/checkout@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push image
        env:
          DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker pull $DOCKERHUB_USERNAME/go-devops-test:latest
          cd src
          docker build -t "$DOCKERHUB_USERNAME/go-devops-test:latest" .
          docker push "$DOCKERHUB_USERNAME/go-devops-test:latest"
```

Here, the workflow will be triggered when there is a code push event on any branch. It will build the code and push it to Docker Hub.


## 4. Initiates CI tests automatically when changes are pushed to the feature branch on GitHub.

We add the following job to the cicd.yml file.

```yaml
  test:
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - name: Check out
        uses: actions/checkout@v3
      
      - name: Set up Node
        uses: actions/setup-node@v3
        with:
          node-version: 20
      
      - name: Install dependencies
        run: npm install
        working-directory: src
        
      - name: Test
        run: |
          if npm test; then
            echo "test success"
          else
            echo "test failure"
            exit 1
          fi
        working-directory: src
```

This job is executed after the build job is successful.

## 5. Utilizes GitHub Actions for Continuous Deployment (CD) to deploy the application to major cloud providers GCP

### Step 1: Setup GCP

We create a service account and get private the key as a .json file. Besides, we proceed to enable APIs and Services.

### Step 2: Add secret variables

We add the following variables secret:
```
GCP_PROJECT_ID
GCP_SA_KEY_JSON
```

### Step 3: Config .yml workflow file

We add the following job to the cicd.yml file.

```yaml
  deploy:
      runs-on: ubuntu-latest
      needs: [build, test]
      
      steps:
        - name: Deploy to Cloud Run
          uses: actions/checkout@v3
        
        - name: Set up GCLoud CLI
          uses: google-github-actions/auth@v2
          with:
            credentials_json: "${{ secrets.GCP_SA_KEY_JSON }}"
            version: "290.0.1"
            SA_KEY_JSON: ${{ secrets.GCP_SA_KEY_JSON }}
            PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
            
        - name: Build
          run: echo  $PROJECT_ID && gcloud builds submit --tag gcr.io/$PROJECT_ID/go-devops-test:latest
          working-directory: src
          
        - name: Deploy
          run: gcloud run deploy $PROJECT_ID --image gcr.io/$PROJECT_ID/go-devops-test:latest --platform managed --region us-west4
          working-directory: src
```

The "Deploy" job will only run when the two preceding jobs have completed successfully.

Link deploy: https://console.cloud.google.com/run/detail/us-west4/golden-owl-project/metrics?authuser=1&project=golden-owl-project&supportedpurview=project
## Related
[Dockerizing a Node.js web app | Node.js](https://golden-owl-project-kave3j76aq-wn.a.run.app/)
