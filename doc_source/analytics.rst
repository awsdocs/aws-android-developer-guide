.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-Sha$
   International License (the "License"). You may not use this file except in c$
   License. A copy of the License is located at http://creativecommons.org/lice$

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIO$
   either express or implied. See the License for the specific language governi$
   limitations under the License.

.. highlight:: java

.. _analytics:

#################################################
Track App Usage Data with Amazon Mobile Analytics
#################################################

.. contents::
    :local:
    :depth: 1

What is Amazon Mobile Analytics?
================================

Amazon Mobile Analytics lets you collect, visualize, and understand app usage for your Android and
Fire OS apps. Reports are available for metrics on active users, sessions, retention, in-app
revenue, and custom events, and can be filtered by platform and date range. Amazon Mobile Analytics
is built to scale with your business and can collect and process billions of events from many
millions of endpoints.

Using Amazon Mobile Analytics, you can track customer behaviors, aggregate metrics, generate data
visualizations, and identify meaningful patterns. The AWS SDK for Android provides integration with
the Mobile Analytics service.

For information about Mobile Analytics Region availability, see `AWS Service Region Availability
<http://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/>`_.

The sections below explain how to integrate Mobile Analytics with your app.


Getting Started
===============

Create an App in the Mobile Analytics Console
---------------------------------------------

Go to the `Amazon Mobile Analytics Console <https://console.aws.amazon.com/mobileanalytics/home>`_
and create an app. Note the :code:`appId` value, as you'll need it later.

To learn more about working in the console, see the `Amazon Mobile Analytics User Guide
<http://docs.aws.amazon.com/mobileanalytics/latest/ug/>`_.


Create an Identity Pool
-----------------------

To use AWS services in your mobile application, you must obtain AWS credentials using |COG| Identity
as your credential provider. Using a credentials provider allows you to access AWS services without
having to embed your private credentials in your application. This also allows you to set
permissions to control which AWS services your users have access to.

The identities of your application's users are stored and managed by an identity pool, which is a
store of user identity data specific to your account. Every identity pool has roles that specify
which AWS resources your users can access. Typically, a developer will use one identity pool per
application. For more information on identity pools, see the `Cognito Developer Guide
<http://docs.aws.amazon.com/cognito/devguide/identity/identity-pools/>`_.

To create an identity pool for your application:

#. Log in to the `Cognito Console <https://console.aws.amazon.com/cognito/home>`_ and click
   :guilabel:`Create new identity pool`.

#. Enter a name for your Identity Pool and check the checkbox to enable access to unauthenticated
   identities. Click :guilabel:`Create Pool` to create your identity pool.

#. Click :guilabel:`Allow` to create the roles associated with your identity pool.

The next page displays code that creates a credentials provider so you can easily integrate Cognito
Identity in your Android application.

For more information on Cognito Identity, see :doc:`cognito-auth`.


IAM Policy for Amazon Mobile Analytics
--------------------------------------

To use Mobile Analytics, AWS users must have the correct permissions. The following IAM policy
allows the user to submit events to Mobile Analytics:

.. code-block:: json

    {
        "Statement": [{
            "Effect": "Allow",
            "Action": "mobileanalytics:PutEvents",
            "Resource": "*"
        }]
    }

This policy should be assigned to roles associated with the Cognito identity pool for your app. The
policy allows clients to record events with the Mobile Analytics service. Amazon Cognito will set
this policy for you, if you let it create new roles. Other policies are required to allow IAM users
to view reports.

You can set permissions at the `IAM Console <https://console.aws.amazon.com/iam/>`_. To learn more
about IAM policies, see `Using IAM
<http://docs.aws.amazon.com/IAM/latest/UserGuide/IAM_Introduction.html>`_.


Include the SDK in Your Project
-------------------------------

Follow the instructions on the :doc:`setup` page to include the proper JAR files for this service
and set the appropriate permissions.


Set Permissions in Your Android Manifest
----------------------------------------

In :file:`AndroidManifest.xml`, set the following permissions, if they're not already present:

.. code-block:: xml

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />


Initialize MobileAnalyticsManager
---------------------------------

Define a static reference to the :code:`MobileAnalyticsManager` in the :code:`onCreate()` method of
your main activity::

    private static MobileAnalyticsManager analytics;

For this particular example, let's also create two constants that we'll use later in a custom
event::

    private static final int STATE_LOSE = 0;
    private static final int STATE_WIN = 1;

In the activity’s onCreate() method, create an instance of MobileAnalyticsManager. You’ll need to
replace "cognitoId" and "appId" to their respective values as shown from the Mobile Analytics
console. The appId is used to group your data in the Mobile Analytics console.

::

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        try {
            analytics = MobileAnalyticsManager.getOrCreateInstance(
                        this.getApplicationContext(),
                        "appId",
                        "identityPoolId"
            );
        } catch(InitializationException ex) {
                Log.e(this.getClass().getName(), "Failed to initialize Amazon Mobile Analytics", ex);
        }
    }

By default, the MobileAnalyticsManager client initializes with WAN delivery enabled.


Track Session Events
--------------------

Override the activity’s :code:`onPause()` and :code:`onResume()` methods to record session events.

::

    /**
     * Invoked when the Activity loses user focus
     */
    @Override
    protected void onPause() {
        super.onPause();
        if(analytics != null) {
            analytics.getSessionClient().pauseSession();
            //Attempt to send any events that have been recorded to the Mobile Analytics service.
            analytics.getEventClient().submitEvents();
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        if(analytics != null)  {
            analytics.getSessionClient().resumeSession();
        }
    }

For each activity in your application, you will need to record session events in the
:code:`onPause()` and :code:`onResume()` methods.

Add Monetization Events
~~~~~~~~~~~~~~~~~~~~~~~

The SDK for Android provides a :code:`MonetizationEventBuilder` that lets you create events for
Amazon purchases, Google Play purchases, and virtual store purchases. The
:code:`MonetizationEventBuilder` class can be extended if you need to record monetization events
from other purchase frameworks.

To learn more about adding monetization events, see the API reference guide for
`MonetizationEventBuilder
<http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/index.html?com/amazonaws/mobileconnectors/amazonmobileanalytics/monetization/MonetizationEventBuilder.html>`_.

Record Custom Events
~~~~~~~~~~~~~~~~~~~~

The Mobile Analytics client lets you create and record custom events. For example, if our app were a
game, we might create a custom event to be submitted when the user completes a level. In your main
activity, add the following method, which creates and records a custom event.

::

    /**
    * This method gets called when the player completes a level
    * @param levelName the name of the level
    * @param difficulty the difficulty setting
    * @param timeToComplete the time to complete the level in seconds
    * @param playerState the winning/losing state of the player
    */
    public void onLevelComplete(String levelName, String difficulty, double timeToComplete, int playerState) {

        //Create a Level Complete event with some attributes and metrics(measurements)
        //Attributes and metrics can be added using with statements
        AnalyticsEvent levelCompleteEvent = analytics.getEventClient().createEvent("LevelComplete")
                .withAttribute("LevelName", levelName)
                .withAttribute("Difficulty", difficulty)
                .withMetric("TimeToComplete", timeToComplete);

        //attributes and metrics can also be added using add statements
        if (playerState == STATE_LOSE)
            levelCompleteEvent.addAttribute("EndState", "Lose");
        else if (playerState == STATE_WIN)
            levelCompleteEvent.addAttribute("EndState", "Win");

        //Record the Level Complete event
        analytics.getEventClient().recordEvent(levelCompleteEvent);
    }

Test this custom event by calling it at the end of the :code:`onCreate()` method::

    this.onLevelComplete("Lower Dungeon", "Very Difficult", 2734, STATE_WIN);

Launch and test your app.

Navigate to the :console:`Mobile Analytics Console <mobileanalytics>`. Your app should appear in the
drop-down list, though it may take a few minutes for a new app to appear in the list. You should see
session-related data for your app in the graphs.

