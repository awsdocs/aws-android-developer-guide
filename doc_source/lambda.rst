.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. highlight:: java

AWS Lambda
==========

What is AWS Lambda?
-------------------

AWS Lambda is a compute service that runs your code in response to requests or events and
automatically manages the compute resources for you, making it easy to build applications that
respond quickly to new information. AWS Lambda functions can be called directly from mobile, IoT,
and Web apps and sends a response back synchronously, making it easy to create scalable, secure, and
highly available backends for your mobile apps without the need to provision or manage
infrastructure. For more information, see `AWS Lambda
<http://docs.aws.amazon.com/lambda/latest/dg/welcome.html>`_.

The Mobile SDK for Android enables you to call Lambda functions from your Android mobile apps, using
the :code:`lambdainvoker` library. When invoked from the Mobile SDK, Lambda functions receive data
about the device, app, and end user identity through a client and identity context object, making it
easy to create rich, and personalized app experiences.

The getting started section for AWS Lambda, To use AWS Lambda, :doc:`getting-started-lambda`,
provides step by step instructions for integrating the AWS Mobile SDK for Android into your app.

The AWS Mobile SDK for Android enables you to easily map client methods to Lambda function
executions, using the @LambdaFunction annotation. For example, the code below will result in the
"echo" Lambda function being executed every time the echo method is called inside the application
code::

   @LambdaFunction
   NameInfo echo(NameInfo nameInfo);

The @LambdaFunction annotation can take 3 optional parameters:

functionName
    Allows you to specify the name of the Lambda function to call when the method is executed, by
    default the name of the method is used. For example, the code below will result in the "echo"
    Lambda function being executed every time the noEcho method is called inside the application
    code::

       @LambdaFunction(functionName = "echo")
       NameInfo noEcho(NameInfo nameInfo);

invocationType
    Specifies how the Lambda function will be invoked. Can be one of the following values:

    * Event |ndash| calls are one-way and asynchronous
    * RequestResponse |ndash| calls take input and return output and are executed synchronously
    * DryRun |ndash| allows you to validate access to a Lambda Function without executing it

LogType
    This parameter is valid only when invocationType is set to "RequestResponse". Valid values are
    "None" or "Tail". If set to "Tail" AWS Lambda will return the last 4KB of log data produced by
    your Lambda Function in the x-amz-log-results header, which is base64-encoded.

The class containing methods with the @LambdaFunction annotation must be instantiated using the
:code:`lambdainvoker`. :code:`LambdaInvokerFactory.Build()` method. At the time of creating the
:code:`lambdainvoker`. :code:`LambdaInvokerFactory` object, you will also be able to specify the
region in which the Lambda function exists, and the Cognito Credentials provider to use to verify
permissions to invoke the Lambda function.

Client and Identity Information
===============================

When invoked through the SDK, Lambda functions automatically have access to the data about the
device and the app. It also has access to the end user identity by virtue of using Amazon Cognito as
the credential provider. These are available in form of context.clientContext and context.identity.
To learn more about AWS Lambda’s programming model, see `AWS Lambda
<http://docs.aws.amazon.com/lambda/latest/dg/welcome.html>`_.

You can access the client context within your lambda function as follows::

    exports.handler = function(event, context) {
        console.log("installation_id = " + context.clientContext.client.installation_id);
        console.log("app_version_code = " + context.clientContext.client.app_version_code);
        console.log("app_version_name = " + context.clientContext.client.app_version_name);
        console.log("app_package_name = " + context.clientContext.client.app_package_name);
        console.log("app_title = " + context.clientContext.client.app_title);
        console.log("platform_version = " + context.clientContext.env.platform_versioin);
        console.log("platform = " + context.clientContext.env.platform);
        console.log("make = " + context.clientContext.env.make);
        console.log("model = " + context.clientContext.env.model);
        console.log("locale = " + context.clientContext.env.locale);

        context.succeed("Your platform is " + context.clientContext.env.platform;
    }


Client Context
==============

client.installation_id
    Auto-generated UUID that is created the first time the app is launched. This is stored in shared
    preferences on the device. In case the shared preferences is wiped a new appId will be
    generated.

client.app_version_code
    `versionCode <http://developer.android.com/guide/topics/manifest/manifest-element.html#vcode>`_
    from the Android Manifest

client.app_version_name
    `versionName <http://developer.android.com/guide/topics/manifest/manifest-element.html#vname>`_
    from the Android Manifest

client.app_package_name
    `package <http://developer.android.com/guide/topics/manifest/manifest-element.html#package>`_
    name from the Android Manifest file

client.app_title
    Text defined as `Android:label
    <http://developer.android.com/guide/topics/manifest/application-element.html#label>`_

env.platform_version
    `Build.VERSION.RELEASE <http://developer.android.com/reference/android/os/Build.VERSION.html>`_

env.platform
	Hardcoded as "Android"

env.make
    `Build.MANUFACTURER
    <http://developer.android.com/reference/android/os/Build.html#MANUFACTURER>`_

env.model
    `Build.MODEL <http://developer.android.com/reference/android/os/Build.html#MODEL>`_

env.locale
    `Locale.getDefault()
    <http://developer.android.com/reference/java/util/Locale.html#getDefault()>`_


Identity Context
================

To invoke a Lambda function from your mobile app, you can use Cognito as the credential provider.
For more information, see `Amazon Cognito <http://aws.amazon.com/cognito/>`_. Cognito assigns each
user a unique Identity ID. This Identity ID is available to you in the Lambda functions invoked
through the AWS Mobile SDK. You can access the Identity ID as follows::

    exports.handler = function(payload, context) {
        console.log("clientID = " + context.identity.cognitoIdentityId);
        context.succeed("Your client pool ID is " + context.identity.cognitoIdentityIdPoolId);
    }


Data Types
==========

A method, annotated with LambdaFunction, can have at most one argument. When invoked, its argument
is serialized into JSON. The invocation is translated to an AWS request and is sent to AWS Lambda
service. After excution, Lambda returns a JSON encoded response which is deserialized into an object
whose type matches the return type of the method. The (de)serialization is handled by
LambdaDataBinder. The default implementation is LambdaJsonBinder backed by Gson.

::

   public interface MyInterface {
      /*
       * String[] words = {"Hello", "world", "!"} is serialized as
       * ["Hello", "world", "!"]
       */

      @LambdaFunction
      String echo(String[] words);

      /*
       * NameInfo nameInfo = new NameInfo();
       * nameInfo.firstName = "John";
       * nameInfo.lastName = "Doe";
       * Then nameInfo is serialized as
       * {"firstName":"John","lastName":"Doe"}
       */
      @LambdaFunction
      String echo(NameInfo nameInfo);

      class NameInfo {
         String firstName;
         String lastName;
      }
   }

In case you need to customize LambdaJsonBinder, you have the option to provide your implementation
with LambdaInvokerFactory.build(Class<T>, LambdaDataBinder).

::

    public class JacksonDataBinder implements LambdaDataBinder {
        private final ObjectMapper mapper;

        public JacksonDataBinder() {
            mapper = new ObjectMapper();
            mapper.setPropertyNamingStrategy(
                PropertyNamingStrategy.CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES);
        }

        @Override
        public <T> T deserialize(byte[] content, Class<T> clazz) {
            try {
                return mapper.readValue(content, clazz);
            }
            catch (IOException e) {
                throw new AmazonClientException("Failed to deserialize content", e);
            }
        }

        @Override
        public byte[] serialize(Object object) {
            try {
                return mapper.writeValueAsBytes(object);
            }
            catch (IOException e) {
                throw new AmazonClientException("Failed to serialize object", e);
            }
        }
    }

    // create a Lambda proxied object
    MyInterface myInterface = lambdaInvokerFactory.build(
        MyInterface.class, new JacksonDataBinder());


Error Handling
==============

When you invoke a method annotated with LambdaFunction and it results in an error on the server
side, a LambdaFunctionException, subclass of RuntimeException, will be thrown. You can get the error
message and the invocation result from the exception object.

Note that the method can fail due to other reasons, such as invalid credentials, network problem, or
(de)serialization issue. These errors won't be turned into LambdaFunctionException.

::

    // suppose echo(String) is an annotated Lambda function
    try {
        String result = myInterface.echo("Hello world!");
    }
    catch (LambdaFunctionException lfe) {
        // Lambda code has error.
        Log.e(TAG, String.format(
            "echo method failed: error [%s], payload [%s].",
            lfe.getMessage(), lfe.getPayload());
    }
    catch (AmazonServiceException ase) {
        // invalid credentials, incorrect AWS signature, etc
    }
    catch (AmazonClientException ace) {
        // Network issue
    }

For more information about Identity ID, see `Cognito Identity
<http://docs.aws.amazon.com/mobile/sdkforandroid/developerguide/cognito-auth.html>`_.
