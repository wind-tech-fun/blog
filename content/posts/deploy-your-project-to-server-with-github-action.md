+++
title = 'Deploy Your Project to Server With Github Action'
date = 2024-07-20T10:25:21Z
draft = false
+++

## First 

- Install ```git``` in your server  

- cd /path/to/workspace && git clone https://github.com/username/your_project.git

- cd ~/.ssh  

- ssh-keygen -y -f yourkey.pem > yourkey.pub  

- cat yourkey.pub >> ./authorized_keys  

- chmod 700 ~/.ssh  

- chmod 640 ~/.ssh/authorized_keys  

- cat ~/.ssh/id_rsa  

```
-----BEGIN OPENSSH PRIVATE KEY-----
your SECRET_KEY
-----END OPENSSH PRIVATE KEY-----
```

## Second 

- vi /path/to/workspace/your_project/.github/workflows/deploy.yml

```
# This is a basic workflow to help you get started with Actions

name: DEPLOY

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the "main" branch
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v4

      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo Hello, world!

      # Runs a set of commands using the runners shell
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.
          
  deploy:
    runs-on: ubuntu-latest
    needs: build 
    steps:
      - uses: actions/checkout@v2
      - name: pull code
        uses: appleboy/ssh-action@master
        env:
          SERVER_WORKDIR: ${{ secrets.SERVER_WORKDIR }} 
        with:
          personal_token: ${{ secrets.PUBLISH_TOKEN }}
          external_repository: username/project_name
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USERNAME }}
          key: ${{ secrets.SERVER_KEY }}
          envs: SERVER_WORKDIR
          script: |
            cd $SERVER_WORKDIR 
            git checkout . 
            git pull 
          
```

## Last 

- Go to your deploy project page in github, then go to ```Settings```, then go to ```Secrets and variables```, then go to ```Actions```, then ```New repository secret```.  

- ```secrets.SERVER_WORKDIR``` /path/to/workspace/your_project

- ```secrets.PUBLISH_TOKEN``` https://github.com/settings/tokens?type=beta set it

- ```secrets.SERVER_HOST```  Your server ip [For example ```39.16.110.35```]

- ```secrets.SERVER_USERNAME```  [For example ```root```]

- ```secrets.SECRET_KEY```  ```cat ~/.ssh/id_rsa```
