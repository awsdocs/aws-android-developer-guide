.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. highlight:: java


#######################################
Store and Retrieve Files with Amazon S3
#######################################

What is Amazon S3?
==================

|S3long|_ provides secure, durable, highly-scalable object storage in the cloud. Using the AWS
Mobile SDK, you can directly access Amazon S3 from your mobile app. For information about S3 Region
availability, see `AWS Service Region Availability
<http://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/>`_.

The AWS Mobile SDK allows you to consume the S3 service in your mobile application via the S3
Transfer Utility, which is replacing the S3 Transfer Manager as of AWS Mobile SDK for Android v
2.2.4.

For information on migrating from the S3 Transfer Manager to the S3 Transfer Utility, see `Migrating
from the Transfer Manager to the Transfer Utility <http://mobile.awsblog.com>`_ on the AWS Blog.

For a complete sample that shows how to use the TransferUtility class to perform download and upload tasks, and manage the tasks, see `Running S3TransferUtility Sample <https://github.com/awslabs/aws-sdk-android-samples/tree/master/S3TransferUtilitySample>`_

Setup
=====

Prerequisites
-------------

You must complete all of the instructions on the :doc:`setup` page before beginning this tutorial.

Create and Configure an S3 Bucket
---------------------------------

|S3| stores your application's resources in buckets |mdash| cloud storage containers that live in a
specific `region <regions-and-endpoints>`_.

Create a Bucket
~~~~~~~~~~~~~~~

#. Sign in to the :console:`S3 Console <s3>` and click :guilabel:`Create Bucket`.

#. Enter a bucket name, select a region, and click :guilabel:`Create`. Each S3 bucket must have a
   globally unique name.

Grant Access to Your S3 Resources
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The default IAM role policy grants your application access to Amazon Mobile Analytics and Amazon
Cognito Sync. In order for your Cognito identity pool to access Amazon S3, you must modify the
identity pool's roles.

#. Navigate to the :console:`Identity and Access Management Console <iam>` and click
   :guilabel:`Roles` in the left-hand pane.

#. Type your identity pool name into the search box. Two roles will be listed: one for
   unauthenticated users and one for authenticated users.

#. Click the role for unauthenticated users (it will have unauth appended to your Identity Pool
   name).

#. Click the :guilabel:`Create Role Policy` button, select :guilabel:`Policy Generator`, and then
   click the :guilabel:`Select` button.

#. On the Edit Permissions page, enter the settings shown in the following image. The Amazon
   Resource Name (ARN) of an S3 bucket looks like :code:`arn:aws:s3:::examplebucket/*` and is
   composed of the region in which the bucket is located and the name of the bucket. The settings
   shown below will give your identity pool full to access to all actions for the specified bucket.

   .. image:: images/edit-permissions.png

#. Click the :guilabel:`Add Statement` button and then the :guilabel:`Next Step` button.

#. The Wizard will show you the configuration that you generated. Click the :guilabel:`Apply Policy`
   button.

For more information on granting access to S3, see `Granting Access to an Amazon S3 Bucket`_.


Configure Your Environment
--------------------------

To start using S3 in your application, you need to do the following:

- Add the correct import statements
- Declare the S3 TransferUtility service in your manifest files
- Instantiate a Cognito Caching Credentials Provider, an Amazon S3 client, and a Transfer Utility

Declare the Service in AndroidManifest.xml
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add the following declaration to your :file:`AndroidManifest.xml`:

.. code-block:: xml

    <service
      android:name="com.amazonaws.mobileconnectors.s3.transferutility.TransferService"
      android:enabled="true" />

Instantiate an S3 Client
~~~~~~~~~~~~~~~~~~~~~~~~

Pass your credentials provider to the S3 client constructor, like so::

  // Create an S3 client
  AmazonS3 s3 = new AmazonS3Client(credentialsProvider);

  

Instantiate TransferUtility
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You will use the :code:`TransferUtility` class to upload and download files from S3. Pass the S3
client and the application context to the Transfer Utility, like so::

  TransferUtility transferUtility = new TransferUtility(s3, APPLICATION_CONTEXT);

Operations
==========


Upload an Object to S3
----------------------

To upload a file::

  TransferObserver observer = transferUtility.upload(
    MY_BUCKET,     /* The bucket to upload to */
    OBJECT_KEY,    /* The key for the uploaded object */
    MY_FILE        /* The file where the data to upload exists */
  );

Uploads automatically use S3's multi-part upload functionality on large files to enhance throughput.

Upload an Object to S3 with Metadata
------------------------------------

Create a :code:`ObjectMetadata` object::

  ObjectMetadata myObjectMetadata = new ObjectMetadata();

  //create a map to store user metadata
  Map<String, String> userMetadata = new HashMap<String,String>();
  userMetadata.put(“myKey”,”myVal”);

  //call setUserMetadata on our ObjectMetadata object, passing it our map
  myObjectMetadata.setUserMetadata(userMetadata);

Then, upload an object along with its metadata::

  TransferObserver observer = transferUtility.upload(
    MY_BUCKET,        /* The bucket to upload to */
    OBJECT_KEY,       /* The key for the uploaded object */
    MY_FILE,          /* The file where the data to upload exists */
    myObjectMetadata  /* The ObjectMetadata associated with the object*/
  );

To download the meta, use the low-level S3 :code:`getObjectMetadata` method. For more information,
see the `API Reference
<http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/s3/AmazonS3Client.html#getObjectMetadata%28com.amazonaws.services.s3.model.GetObjectMetadataRequest%29>`_.

Download an Object from S3
--------------------------

To download a file::

  TransferObserver observer = transferUtility.download(
    MY_BUCKET,     /* The bucket to download from */
    OBJECT_KEY,    /* The key for the object to download */
    MY_FILE        /* The file to download the object to */
  );

Tracking S3 Transfer Progress
-----------------------------

With the Transfer Utility, the :code:`download()` and :code:`upload()` methods return a
:code:`TransferObserver` object. This object gives access to:

- The state (now specified as an enum)
- The total bytes transferred thus far
- The total bytes to transfer (for easily calculating progress bars)
- A unique ID that you can use to keep track of distinct transfers

Given the transfer ID, this :code:`TransferObserver` object can be retrieved from anywhere in your
app, including if the app is killed. It also lets you create a :code:`TransferListener`, which will
be updated on state or progress change, as well as when an error occurs.

To get the progress of a download or upload, call :code:`setTransferListener()` on your
:code:`TransferObserver`. This requires you to implement :code:`onStateChanged`,
:code:`onProgressChanged`, and :code:`onError`. For example::

  TransferObserver transferObserver = download(MY_BUCKET, OBJECT_KEY, MY_FILE);
  transferObserver.setTransferListener(new TransferListener(){

      @Override
      public void onStateChanged(int id, TransferState state) {
          // do something
      }

      @Override
      public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
          int percentage = (int) (bytesCurrent/bytesTotal * 100);
          //Display percentage transfered to user
      }

      @Override
      public void onError(int id, Exception ex) {
          // do something
      }

  });

Pause an S3 Transfer
--------------------

If an app is killed, crashes, or loses Internet connectivity, transfers are automatically paused. If
the device running your app loses network connectivity, paused transfers will automatically resume
when the network is available again. If the transfer was manually paused, or the app was killed,
transfers can be resumed with the :code:`resume(transferId)` method.

To pause a single transfer::

  transferUtility.pause(idOfTransferToBePaused);

To pause all uploads::

  transferUtility.pauseAllWithType(TransferType.UPLOAD);

To pause all downloads::

  transferUtility.pauseAllWithType(TransferType.DOWNLOAD);

To pause all transfers of any type::

  transferUtility.pauseAllWithType(TransferType.ANY);

You can also query for :code:`TransferObservers`, which contain the transfer ID, with either
:code:`getTransfersWithType(transferType)` or :code:`getTransfersWithTypeAndState(transferType,
transferState)`. This means that if your app is killed or crashes during a transfer, you can
manually determine if there are any paused transfers when the app resumes and handle those as you
see fit.

Resume a Transfer
-----------------

To resume a single transfer::

  transferUtility.resume(idOfTransferToBeResumed);

Cancel a Transfer
-----------------

Canceling an upload or download is simple. Just call :code:`cancel()` or :code:`cancelAllWithType()`
on the Transfer Utility object.

To cancel a single transfer::

  transferUtility.cancel(idToBeCancelled);

To cancel all transfers of a certain type::

  transferUtility.cancelAllWithType(TransferType.DOWNLOAD);

.. _Granting Access to an Amazon S3 Bucket: http://blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket
