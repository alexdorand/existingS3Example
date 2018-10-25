# Steps to reproduce:
## 1- clone the project
clone the project using the following command:
```commandLine
git clone https://github.com/alexdorand/existingS3Example.git
```

## 2- install the plugins
```commandLine
npm install
```

you can also run (but not necessary) the following commands:
```commandLine
npm install serverless-plugin-existing-s3
npm install serverless-pseudo-parameters
```


## 3- deploy

Run the following commands to deploy to the server:

```commandLine
sls deploy
```

followed by

```commandLine
sls s3Deploy
```