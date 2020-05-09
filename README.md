# Jekyll S3 Test

This project demonstrates how to continuously deploy a Jekyll site to S3 via CircleCI.

## Setup

1. Initialize a new GitHub repo containing a Jekyll site, like this repo
2. Add `.circleci/config.yml` to the repo (see below for example)
3. (Optional but recommended) Create a separate AWS IAM account for hosting the S3 site
4. Obtain an AWS access key via IAM
5. Create a new S3 bucket to host the site
6. Configure the S3 bucket properties to enable static website hosting
7. Add an AWS policy to the bucket allowing public `s3:GetObject` access (see below for example)
8. Add a new project in CircleCI and select your GitHub repo
9. Configure the CircleCI project environment variables to add:
    * `AWS_ACCESS_KEY_ID` - from IAM
    * `AWS_SECRET_ACCESS_KEY` - from IAM
    * `AWS_REGION` - your default AWS region

At this point, CircleCI should automatically build and deploy any changes you push to the `master` branch.

### CircleCI configuration example

```
version: 2.1
orbs:
  ruby: circleci/ruby@0.2.2
  s3: circleci/aws-s3@1.0.15

jobs:
  build:
    docker:
      - image: circleci/ruby:2.6.5
    steps:
      - checkout
      - run:
          name: Install bundler
          command: gem install bundler
      - ruby/load-cache
      - ruby/install-deps
      - ruby/save-cache
      - run:
          name: Jekyll build
          command: BUNDLE_PATH=vendor/bundle bundle exec jekyll build
      - persist_to_workspace:
          root: .
          paths:
            - _site
  deploy:
    docker:
      - image: circleci/python:3.6
    steps:
      - attach_workspace:
          at: .
      - s3/sync:
          from: _site
          overwrite: true
          to: 's3://YOUR_BUCKET_NAME_HERE/'

workflows:
  continuous_deployment:
    jobs:
      - build
      - deploy:
          requires:
            - build

```

### S3 bucket policy example

```
{
    "Version": "2012-10-17",
    "Id": "s3BucketPolicy",
    "Statement": [
        {
            "Sid": "allowPublicAccess",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME_HERE/*"
        }
    ]
}
```
