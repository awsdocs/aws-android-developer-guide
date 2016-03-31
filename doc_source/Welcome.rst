.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

================================
What is the AWS SDK for Android?
================================

The AWS SDK for Android is an open-source software development kit, distributed under an Apache Open Source license. The SDK for Android provides libraries, code samples, and documentation to help developers build connected mobile applications using AWS. Supported AWS services currently include:

* `Amazon Cognito Identity`_
* `Amazon Cognito Sync`_
* `Amazon Mobile Analytics`_
* `Amazon S3`_
* `Amazon DynamoDB`_
* `Amazon Kinesis`_
* `AWS Lambda`_

Compatability
=============

The AWS Mobile SDK for Android is compatible with Android 2.3.3 (API Level 10) or higher. For more information about the Android platform, see `Android Developers <http://developer.android.com/index.html>`_.

Download the AWS Mobile SDK for Android
=======================================

* `Download AWS Mobile SDK for Android <http://sdk-for-android.amazonwebservices.com/latest/aws-android-sdk.zip>`_ (zip file)
* `Source on Github <https://github.com/aws/aws-sdk-android>`_

About the AWS Mobile Services
=============================

The AWS Mobile SDKs include client-side libraries for working with Amazon Web Services. These client libraries provide high-level, mobile-optimized interfaces to services such as Amazon DynamoDB, Amazon Simple Storage Service, and Amazon Kinesis.

The Mobile SDKs also include clients for Amazon Cognito and Amazon Mobile Analytics |mdash| web services designed specifically for use by mobile developers.

Amazon Cognito
--------------

Amazon Cognito facilitates the delivery of scoped, temporary credentials to mobile
devices or other untrusted environments, and it uniquely identifies a device or user and
supplies the user with a consistent identity throughout the lifetime of an application.
The Amazon Cognito Sync service enables cross-device syncing of application-related user data.
Cognito also persists data locally, so that it's available even if the device is
offline.

After you set up the SDK, you can start using Amazon Cognito by following the
instructions at :doc:`cognito-auth` and :doc:`cognito-sync`.

Amazon Mobile Analytics
-----------------------

Amazon Mobile Analytics lets you collect, visualize, and understand app usage for your
mobile apps. Reports are available for metrics on active users, sessions, retention,
in-app revenue, and custom events, and can be filtered by platform and date
range.

After you set up the SDK, you can start using Amazon Mobile Analytics by following
the instructions at :doc:`analytics`.

Amazon Simple Storage Service (S3)
----------------------------------

Amazon Simple Storage Service (S3) provides secure, durable, highly-scalable object storage in the cloud. Using the AWS Mobile SDK, you can directly access Amazon S3 from your mobile app.

After you set up the SDK, you can start using Amazon S3 by following
the instructions at :doc:`s3transferutility`.

Amazon DynamoDB
---------------

Amazon DynamoDB is a fast, highly scalable, highly available, cost-effective, nonrelational database service. DynamoDB removes traditional scalability limitations on data storage while maintaining low latency and predictable performance.

After you set up the SDK, you can start using Amazon DynamoDB by following
the instructions at :doc:`dynamodb_om`.

Amazon Kinesis
--------------

Amazon Kinesis is a fully managed service for real-time processing of streaming data at massive scale.

After you set up the SDK, you can start using Amazon Kinesis by following
the instructions at :doc:`kinesis`.

AWS Lambda
----------

AWS Lambda is a compute service that runs your code in response to requests or events and automatically manages the compute resources for you, making it easy to build applications that respond quickly to new information.

After you set up the SDK, you can start using AWS Lambda by following
the instructions at :doc:`lambda`.

What’s included in the AWS Mobile SDK for Android?
==================================================

The AWS SDK for Android includes the following:

- *Class libraries* - Classes that hide much of
  the lower-level plumbing of the web service interface, including authentication,
  request retries, and error handling. Each service has its own library, so you can
  include class libraries for only the services you need and keep your application as
  small as possible.

- *Code samples* - Practical examples of using the
  class libraries to build applications.

- *Documentation* - Reference documentation for
  the AWS SDK for Android.

The SDK is distributed as a :file:`.zip` file containing the following assets:

- :file:`License.txt`
- :file:`Notice.txt`
- :file:`Readme.txt`
- lib/ |mdash| Contains Java archive files (:file:`.jar`) that include AWS class libraries. To manage the size of your application, you can include only the files that you need for the services your application is using.
- documentation/ |mdash| Includes Javadoc files and other documentation for using the AWS SDK for Android.
- samples/ |mdash| Contains an HTML document with links to samples on GitHub. Samples are named based on the services they demonstrate.
- src/ |mdash| Contains an HTML document with links to source on GitHub. Contains the original source files for the class libraries.
- third-party/ |mdash| Contains third-party libraries that the SDK depends on.

.. important:: The SDK for Android no longer includes a separate JAR for AWS Security Token Service.
   STS is now bundled with the core, and including STS as a separate JAR will result
   in a compile-time error.

.. _Amazon Cognito Identity: http://aws.amazon.com/cognito
.. _Amazon Cognito Sync: http://aws.amazon.com/cognito
.. _Amazon S3: http://aws.amazon.com/s3/
.. _Amazon DynamoDB: http://aws.amazon.com/dynamodb/
.. _Amazon Mobile Analytics: http://aws.amazon.com/mobileanalytics/
.. _Amazon Simple Notification Service: http://aws.amazon.com/sns/
.. _Amazon Kinesis: http://aws.amazon.com/kinesis
.. _AWS Lambda: http://aws.amazon.com/lambda
