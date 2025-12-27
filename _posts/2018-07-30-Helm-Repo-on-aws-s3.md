---
layout: post
title:  "Helm Chart Repo on Amazon S3"
date:   2018-07-30 05:55:10 +0100
---

## Prerequisites

* Technical IAM user with programmatic access only, used to push new Helm Charts to the repo, e.g. ci-user-accesskey
* New S3 bucket, in this how to using my-helm-repo
* [awscli](https://github.com/aws/aws-cli), [helm](https://github.com/helm/helm), [helm-s3 plugin](https://github.com/hypnoglow/helm-s3)

## Get started

* Create local awscli profiles, allowing to test the S3 bucket policy later on:
{% highlight bash linenos %}
# default AWS user (enter AWS Access Key ID, Key, region,...)
aws configure
# CI user
aws configure --profile ci-user
{% endhighlight %}
* Go to the AWS console > Amazon S3 > my-helm-chart-repo and add the following Bucket policy (adjust as needed):
{% highlight bash linenos %}
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowListObjects",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::101010101:root"
            },
            "Action": [
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::my-helm-repo",
                "arn:aws:s3:::my-helm-repo/*"
            ]
        },
        {
            "Sid": "AllowObjectsFetchAndCreate",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::101010101:user/ci-user-accesskey"
            },
            "Action": [
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:ListBucketMultipartUploads",
                "s3:ListMultipartUploadParts",
                "s3:AbortMultipartUpload",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::my-helm-repo",
                "arn:aws:s3:::my-helm-repo/*"
            ]
        }
    ]
}
{% endhighlight %}
* The idea here is to allow any user on behalf of the main account to read the repository's charts, where only the continuous integration user is allowed to push new Helm Charts to the repo
* Initilaize and add the Helm repo:
{% highlight bash linenos %}
helm S3 init s3://my-helm-repo/charts
helm repo add my-helm-repo s3://my-helm-repo/charts
# List repos
helm repo list
{% endhighlight %}
* Perform a first test in order to verify if access to the S3 bucket works as expected:
{% highlight bash linenos %}
# Switch awscli profile and list bucket content as the CI-user
export AWS_PROFILE=ci-user
aws s3 ls s3://my-helm-repo/charts/
# unset
unset AWS_PROFILE
{% endhighlight %}
* Create and upload a test helm chart:
{% highlight bash linenos %}
helm create test-chart
rm -rf test-chart/templates/*.*
cat >test-chart/templates/configmap.yaml <<EOL
apiVersion: v1  
kind: ConfigMap  
metadata:  
  name: test-chart-configmap
data:  
  myvalue: "Hello World"
EOL
helm package ./test-chart
helm s3 push ./test-chart-0.1.0.tgz my-helm-repo
{% endhighlight %}

## CI pipeline integration

* Depending on the CI-product being used (e.g. Jenkins, Gitlab), add the following variables to your pipeline configuration in order to have them being passed into the build runner or build job as environment variables:
{% highlight linenos %}
AWS_ACCESS_KEY_ID = "ci-user key id"
AWS_SECRET_ACCESS_KEY = "ci-user key"
AWS_DEFAULT_REGION = "s3 bucket region"
{% endhighlight %}
* As soon as a new Helm Chart is being created and pushed to the repo, this way awscli driven by the helm-s3 plugin will use these env vars, respectively these credentials from within the build runner