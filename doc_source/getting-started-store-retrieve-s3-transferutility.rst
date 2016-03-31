.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. highlight:: java

Store and Retrieve Files with Amazon S3 Transfer Utility
========================================================

Amazon Simple Storage Service (Amazon S3) provides mobile developers with secure, durable, highly-scalable object storage. Amazon S3 is easy to use, with a simple web services interface to store and retrieve any amount of data from anywhere on the web.

The AWS Mobile SDK allows you to consume the S3 service in your mobile application via the S3 Transfer Utility, which is replacing the S3 Transfer Manager as of AWS Mobile SDK for Android v 2.2.4. For information on migrating from the S3 Transfer Manger to the S3 Transfer Utility, see `Migrating from the Transfer Manager to the Transfer Utility <https://mobile.awsblog.com/post/Tx2KF0YUQITA164/AWS-SDK-for-Android-Transfer-Manager-to-Transfer-Utility-Migration-Guide>`_ on the AWS Blog.

The tutorial below explains how to integrate the S3 TransferUtility, a high-level utility for using S3 with your app. This tutorial assumes you have already created an S3 bucket. To create an S3 bucket, visit the `S3 AWS Console <https://console.aws.amazon.com/s3/home>`_.

Project Setup
-------------

Prerequisites
~~~~~~~~~~~~~

You must complete all of the instructions on the `Set Up the SDK for Android <http://docs.aws.amazon.com/mobile/sdkforandroid/developerguide/setup.html>`_ page before beginning this tutorial.

Grant Access to Your S3 Resources
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The default IAM role policy grants your application access to Amazon Mobile Analytics and Amazon Cognito Sync. In order for your Cognito identity pool to access Amazon S3, you must modify the identity pool's roles.

#. Navigate to the `Identity and Access Management Console`_ and click :guilabel:`Roles` in the left-hand pane.
#. Type your identity pool name into the search box. Two roles will be listed: one for unauthenticated users and one for authenticated users.
#. Click the role for unauthenticated users (it will have unauth appended to your Identity Pool name).
#. Click the :guilabel:`Create Role Policy` button, select :guilabel:`Policy Generator`, and then click the :guilabel:`Select` button.
#. On the Edit Permissions page, enter the settings shown in the following image. The Amazon Resource Name (ARN) of an S3 bucket looks like :code:`arn:aws:s3:::examplebucket/*` and is composed of the region in which the bucket is located and the name of the bucket. The settings shown below will give your identity pool full to access to all actions for the specified bucket.

    .. image:: images/edit-permissions.png

6. Click the :guilabel:`Add Statement` button and then the :guilabel:`Next Step` button.
7. The Wizard will show you the configuration that you generated. Click the :guilabel:`Apply Policy` button.

For more information on granting access to S3, see `Granting Access to an Amazon S3 Bucket`_.

Declare the Service in AndroidManifest.xml
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add the following declaration to your :file:`AndroidManifest.xml`:
::

    <service
        android:name="com.amazonaws.mobileconnectors.s3.transferutility.TransferService"
        android:enabled="true" />

Initialize the S3 TransferUtility
---------------------------------

First, pass your Amazon Cognito credentials provider to the S3 client constructor. Then, pass the client to the TransferUtility constructor along with the application context:
::

    AmazonS3 s3 = new AmazonS3Client(credentialsProvider);
    TransferUtility transferUtility = new TransferUtility(s3, getApplicationContext());

Upload a File to Amazon S3
--------------------------

To upload a file to S3, instantiate a :code:`TransferObserver` object. Call :code:`upload()` on your :code:`TransferUtility` object and assign it to the observer, passing the following parameters:

- :code:`bucket_name` - Name of the S3 bucket to store the file
- :code:`key` - Name of the file, once stored in S3
- :code:`file` - java.io.File object to upload

::

  TransferObserver observer = transferUtility.upload(
    MY_BUCKET,     /* The bucket to upload to */
    OBJECT_KEY,    /* The key for the uploaded object */
    MY_FILE        /* The file where the data to upload exists */
  );

Uploads automatically use S3's multi-part upload functionality on large files to enhance throughput.

Download a File from Amazon S3
------------------------------

To download a file from S3, instantiate a :code:`TransferObserver` object. Call :code:`download()` on your :code:`TransferUtility` object and assign it to the observer, passing the following parameters:

- :code:`bucket_name` - A string representing the name of the S3 bucket where the file is stored
- :code:`key` - A string representing the name of the S3 object (a file in this case) to download
- :code:`file` - the java.io.File object where the downloaded file will be written

::

  TransferObserver observer = transferUtility.download(
    MY_BUCKET,     /* The bucket to download from */
    OBJECT_KEY,    /* The key for the object to download */
    MY_FILE        /* The file to download the object to */
  );

For more information on accessing Amazon S3 from an Android application, see :doc:`s3transferutility`.

.. _Identity and Access Management Console: https://console.aws.amazon.com/iam/home
.. _Granting Access to an Amazon S3 Bucket: http://blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket
