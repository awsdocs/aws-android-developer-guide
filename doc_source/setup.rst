.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. highlight:: java

#####################################
Set Up the AWS Mobile SDK for Android
#####################################

To get started with the |sdk-android|, you can set up the SDK and start building a new project, or
you can integrate the SDK with an existing project. You can also clone and run the `samples
<https://github.com/awslabs/aws-sdk-android-samples>`_ to get a sense of how the SDK works.

Prerequisites
=============

Before you can use the |sdk-android|, you will need the following:

- An `AWS Account <http://aws.amazon.com>`_

- Android 2.3.3 (API Level 10) or higher (for more information about the Android platform, see
  `Android Developers <http://developer.android.com/index.html>`_)

- `Android Studio <https://developer.android.com/sdk/index.html>`_ or `Android Development Tools for
  Eclipse <http://developer.android.com/sdk/eclipse-adt.html>`_

After completing the prerequisites, you will need to do the following to get started:

#. Get the |sdk-android|.
#. Set permissions in your :file:`AndroidManifest.xml` file.
#. Obtain AWS credentials using Amazon Cognito.

Step 1: Get the |sdk-android|
=============================

There are three ways to get the |sdk-android|.

Option 1: Using Gradle with Android Studio
------------------------------------------

If you are using Android Studio, add the :file:`aws-android-sdk-core` dependency to your
:file:`app/build.gradle` file, along with the dependencies for the individual services
that your project will use, as shown below.

.. code-block:: groovy

    dependencies {
        compile 'com.amazonaws:aws-android-sdk-core:2.2.+'
        compile 'com.amazonaws:aws-android-sdk-s3:2.2.+'
        compile 'com.amazonaws:aws-android-sdk-ddb:2.2.+'
    }

A full list of dependencies are listed below.

====================================== =======================================
Dependency                             Build.gradle Value
====================================== =======================================
AWS Mobile SDK core                    com.amazonaws:aws-android-sdk-core:2.2.+
Amazon API Gateway                     com.amazonaws:aws-android-sdk-apigateway-core:2.2.+
Auto Scaling                           com.amazonaws:aws-android-sdk-autoscaling:2.2.+
Amazon Cloud Watch                     com.amazonaws:aws-android-sdk-cloudwatch:2.2.+
Amazon Cognito Sync                    com.amazonaws:aws-android-sdk-cognito:2.2.+
Amazon Cognito Identity Provider       com.amazonaws:aws-android-sdk-cognitoidentityprovider:2.2.+
Amazon DynamoDB                        com.amazonaws:aws-android-sdk-ddb:2.2.+
Amazon DynamoDB Object Mapper          com.amazonaws:aws-android-sdk-ddb-mapper:2.2.+
Amazon EC2                             com.amazonaws:aws-android-sdk-ec2:2.2.+
Elastic Load Balancing                 com.amazonaws:aws-android-sdk-elb:2.2.+
AWS IoT                                com.amazonaws:aws-android-sdk-iot:2.2.+
Amazon Kinesis                         com.amazonaws:aws-android-sdk-kinesis:2.2.+
AWS Key Management Service (KMS)       com.amazonaws:aws-android-sdk-kms:2.2.+
Amazon Lex                             com.amazonaws:aws-android-sdk-lex:2.3.4@aar
AWS Lambda                             com.amazonaws:aws-android-sdk-lambda:2.2.+
Amazon Machine Learning                com.amazonaws:aws-android-sdk-machinelearning:2.2.+
Amazon Mobile Analytics                com.amazonaws:aws-android-sdk-mobileanalytics:2.2.+
Amazon Pinpoint                        com.amazonaws:aws-android-sdk-pinpoint:2.3.5
Amazon Polly                           com.amazonaws:aws-android-sdk-polly:2.3.4
Amazon S3                              com.amazonaws:aws-android-sdk-s3:2.2.+
Amazon Simple DB                       com.amazonaws:aws-android-sdk-sdb:2.2.+
Amazon SES                             com.amazonaws:aws-android-sdk-ses:2.2.+
Amazon SNS                             com.amazonaws:aws-android-sdk-sns:2.2.+
Amazon SQS                             com.amazonaws:aws-android-sdk-sqs:2.2.+
====================================== =======================================

Option 2: Import the JAR Files
------------------------------

To obtain the JAR files, download the SDK from http://aws.amazon.com/mobile/sdk. The SDK is stored
in a compressed file named :file:`aws-android-sdk-#-#-#`, where #-#-# represents the version number.
Source code is available on `GitHub <https://github.com/aws/aws-sdk-android>`_.

**If using Android Studio:**

In the Project view, drag :file:`aws-android-sdk-#-#-#-core.jar` plus the :file:`.jar` files for the individual services
your project will use into the :file:`apps/libs` folder. They'll be included on the build path
automatically. Then, sync your project with the Gradle file.

**If using Eclipse:**

Drag the :file:`aws-android-sdk-#-#-#-core.jar` file
plus the :file:`.jar` files for the individual services your project will use, into the :file:`libs`
folder. They'll be included on the build path automatically.

Option 3: Using Maven
---------------------

The |sdk-android| supports Apache Maven, a dependency management and build automation tool. A Maven
project contains a :file:`pom.xml` file where you can specify the Amazon Web Services that you want
to use in your app. Maven then includes the services in your project, so that you don't have to
download the entire AWS Mobile SDK and manually include JAR files.

Maven is supported in |sdk-android| v. 2.1.3 and onward. Older versions of the SDK are not available
via Maven. If you're new to Maven and you'd like to learn more about it, see the `Maven
documentation <http://maven.apache.org/what-is-maven.html>`_.


pom.xml Example
~~~~~~~~~~~~~~~

Here's an example of how you can add `Amazon Cognito Identity <http://aws.amazon.com/cognito/>`_,
`Amazon S3 <http://aws.amazon.com/s3/>`_, and `Amazon Mobile Analytics
<http://aws.amazon.com/mobileanalytics/>`_ to your project:

.. code-block:: xml

    <dependencies>
        <dependency>
            <groupid>com.amazonaws</groupid>
            <artifactid>aws-android-sdk-core</artifactid>
            <version>[2.2.0, 2.3)</version>
        </dependency>
        <dependency>
            <groupid>com.amazonaws</groupid>
            <artifactid>aws-android-sdk-s3</artifactid>
            <version>[2.2.0, 2.3)</version>
        </dependency>
        <dependency>
            <groupid>com.amazonaws</groupid>
            <artifactid>aws-android-sdk-mobileanalytics</artifactid>
            <version>[2.2.0, 2.3)</version>
        </dependency>
    </dependencies>

As shown above, the groupId for the |sdk-android| is ``com.amazonaws``. For each additional service,
include a ``<dependency>`` element following the model above, and use the appropriate artifactID
from the table below. The ``<version>`` element specifies the version of the |sdk-android|. The
example above demonstrate's Maven's ability to use a range of acceptable versions for a given
dependency. To review available versions of the SDK for Android, see the `Release Notes
<https://aws.amazon.com/releasenotes/Android>`_.

The AWS Mobile :code:`artifactId` values are as follows:

====================================== =======================================
Service/Feature                        artifactID
====================================== =======================================
AWS Mobile SDK Core [#f1]_             aws-android-sdk-core
Amazon API Gateway                     aws-android-sdk-apigateway-core
Auto Scaling                           aws-android-sdk-autoscaling
Amazon Cloud Watch                     aws-android-sdk-cloudwatch
Amazon Cognito Sync                    aws-android-sdk-cognito
Amazon Cognito Identity Provider       aws-android-sdk-cognitoidentityprovider
Amazon DynamoDB                        aws-android-sdk-ddb
Amazon DynamoDB Object Mapper          aws-android-sdk-ddb-mapper
Amazon EC2                             aws-android-sdk-ec2
Elastic Load Balancing                 aws-android-sdk-elb
AWS IoT                                aws-android-sdk-iot
Amazon Kinesis                         aws-android-sdk-kinesis
AWS Key Management Service (KMS)       aws-android-sdk-kms
AWS Lambda                             aws-android-sdk-lambda
AWmazon Lex                            aws-android-sdk-lex
Amazon Machine Learning                aws-android-sdk-machinelearning
Amazon Mobile Analytics                aws-android-sdk-mobileanalytics
Amazon Pinpoint                        aws-android-sdk-pinpoint
Amazon Polly                           aws-android-sdk-polly
Amazon S3                              aws-android-sdk-s3
Amazon Simple DB                       aws-android-sdk-sdb
Amazon SES                             aws-android-sdk-ses
Amazon SNS                             aws-android-sdk-sns
Amazon SQS                             aws-android-sdk-sqs
====================================== =======================================

.. rubric:: Footnotes

.. [#f1] AWS Mobile SDK Core includes Amazon Cognito Identity and AWS Simple Token Service (STS).

Step 2: Set Permissions in Your Manifest
========================================

Add the following permission to your :file:`AndroidManifest.xml`

.. code-block:: xml

    <uses-permission android:name="android.permission.INTERNET" />

Step 3: Get AWS Credentials
===========================

To use AWS services in your mobile application, you must obtain AWS Credentials using Amazon Cognito
Identity as your credential provider. Using a credentials provider allows your app to access AWS
services without having to embed your private credentials in your application. This also allows you
to set permissions to control which AWS services your users have access to.

To get started with Amazon Cognito, you must create an identity pool. An identity pool is a store of
user identity data specific to your account. Every identity pool has configurable IAM roles that
allow you to specify which AWS services your application's users can access. Typically, a developer
will use one identity pool per application. For more information on identity pools, see the `Amazon
Cognito Developer Guide <http://docs.aws.amazon.com/cognito/devguide/identity/identity-pools/>`_.

To create an identity pool for your application:

#. Log in to the `Amazon Cognito Console <https://console.aws.amazon.com/cognito/home>`_ and click
   :guilabel:`Manage Federated Identities`, then :guilabel:`Create new identity pool`.

#. Enter a name for your Identity Pool and check the checkbox to enable access to unauthenticated
   identities. Click :guilabel:`Create Pool` to create your identity pool.

#. Click :guilabel:`Allow` to create the two default roles associated with your identity pool
   |mdash| one for unauthenticated users and one for authenticated users. These default roles
   provide your identity pool access to Cognito Sync and Mobile Analytics.

The next page displays code that creates a credentials provider so you can easily integrate Cognito
Identity with your Android application. You pass the credentials provider object to the constructor
of the AWS client you are using. The credentials provider looks like this::

    CognitoCachingCredentialsProvider credentialsProvider = new CognitoCachingCredentialsProvider(
        getApplicationContext(),    /* get the context for the application */
        "COGNITO_IDENTITY_POOL",    /* Identity Pool ID */
        Regions.MY_REGION           /* Region for your identity pool--US_EAST_1 or EU_WEST_1*/
    );

Next Steps
==========

- **Get Started**: View one of our step-by-step `Getting Started Guides
  <http://docs.aws.amazon.com/mobile/sdkforandroid/developerguide/getting-started-android.html>`_.

- **Run the demos**: View our `sample Android apps
  <https://github.com/awslabs/aws-sdk-android-samples>`_ that demonstrate common use cases. To run
  the sample apps, set up the SDK for Android as described above, and then follow the instructions
  contained in the README files of the individual samples.

- **Read the API Reference**: View the `API Reference
  <https://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/>`_ for the AWS Mobile SDK for Android.

- **Try AWS Mobile Hub**: Quickly configure and provision an AWS cloud backend for many common mobile
  app features, and download end to end working Android demonstration projects, SDK, and helper code, all
  generated based on your choices. These are accompanied by detailed integration guidance for your mobile app.

- **Ask questions**: Post questions on the :forum:`AWS Mobile SDK Forums <88>`.

