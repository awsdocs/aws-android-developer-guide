.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. highlight:: java

#####################################
Tracking App Usage Data with |MAlong|
#####################################

|MAlong| allows you to measure app usage and app revenue. By tracking key trends such as new vs.
returning users, app revenue, user retention, and custom in-app behavior events, you can make
data-driven decisions to increase engagement and monetization for your app.

The tutorial below explains how to integrate |MA| with your app.

Project Setup
=============

Prerequisites
-------------

You must complete all of the instructions on the :doc:`setup` page before beginning this tutorial.


Create an App in the |MA| Console
---------------------------------

Go to the :console:`Mobile Analytics Console <mobileanalytics>` and create an
app. Note the :code:`appId` value, as you'll need it later.

.. note:: To learn more about working in the console, see the |MA-ug|_.

Set Permissions in Your Android Manifest
----------------------------------------

In :file:`AndroidManifest.xml`, set the following permissions::

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

Initialize MobileAnalyticsManager
=================================

Define a static reference to the :code:`MobileAnalyticsManager` in the :code:`onCreate()` method of
your main activity::

    private static MobileAnalyticsManager analytics;

For this particular example, let's also create two constants that we'll use later in a custom
event::

    private static final int STATE_LOSE = 0;
    private static final int STATE_WIN = 1;

In the activity’s onCreate() method, create an instance of MobileAnalyticsManager. You’ll need to
replace "cognitoId" and "appId" to their respective values as shown from the |MA| console. The appId
is used to group your data in the console.

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
====================

Override the activity’s :code:`onPause()` and :code:`onResume()` methods to record session events::

    /**
     * Invoked when the Activity loses user focus.
     */
    @Override
    protected void onPause() {
        super.onPause();
        if(analytics != null) {
            analytics.getSessionClient().pauseSession();
            //Attempt to send any events that have been recorded to the Mobile Analytics service
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
-----------------------

The |sdk-android| provides a :code:`MonetizationEventBuilder` that lets you create events for Amazon
purchases, Google Play purchases, and virtual store purchases. The :code:`MonetizationEventBuilder`
class can be extended if you need to record monetization events from other purchase frameworks.

To learn more about adding monetization events, see the API reference guide for
`MonetizationEventBuilder
<http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/index.html?com/amazonaws/mobileconnectors/amazonmobileanalytics/monetization/MonetizationEventBuilder.html>`_.

Record Custom Events
--------------------

The |MA| client lets you create and record custom events. For example, if our app were a game, we
might create a custom event to be submitted when the user completes a level. In your main activity,
add the following method, which creates and records a custom event.

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

