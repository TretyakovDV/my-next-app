[APP LINK](http://my-next-app.s3-website.us-east-2.amazonaws.com/)

# Deploy to AWS S3 checklist

1. In the `package.json` file, change build script
```
"build": "next build && next export",
```

2. Create S3 bucket
    - go to aws management console
    - in the search field to find `s3`
    - create a new bucket
    - in the properties tab enable `static web site hosting`
    - in the permissions tab `Block all public access` must be off
    - in the `bucket policy` add 
   ```
   {
      "Version":"2012-10-17",
      "Statement":[
         {
            "Sid":"PublicReadGetObject",
            "Effect":"Allow",
            "Principal": "*",
            "Action":["s3:GetObject"],
            "Resource":"arn:aws:s3:::<bucket name>/*"
         }
      ]
   }
   ```

2. GitLab CI
   - create file with name `.gitlab-ci.yml`
   - add
   ```
   variables:
      S3_BUCKET_NAME: "my-next-app"

   stages:
   - build
   - deploy

   build:
   stage: build
   image: node:14.16.0
   script:
      - yarn install --frozen-lockfile
      - yarn build
   artifacts:
      paths:
      - ./out
      
   deploy:
   stage: deploy
   dependencies: 
      - build
   image: registry.gitlab.com/gitlab-org/cloud-deploy/aws-base:latest
   script:
      - aws configure set region us-east-2
      - aws s3 sync ./out s3://$S3_BUCKET_NAME
   only:
      - main

   ```

3. Create security credentials
   - in the AWS Management Console in a header, click on your username
   - go to `My Security Credentials`
   - in a `Access keys` section click on `Create New Access Key`
   
     IMPORTANT!!! Download Key File
   
4. Add CI/CD variables
   - go to a Settings > CI/CD
   - expand `Variables` block and add variables
     
     `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` with value from downloaded file