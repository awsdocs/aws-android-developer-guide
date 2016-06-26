.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. highlight:: java


####################################
Store and Query App Data in DynamoDB
####################################

`Amazon DynamoDB <http://aws.amazon.com/dynamodb/>`_ is a fast, highly scalable, highly available,
cost-effective, non-relational database service. DynamoDB removes traditional scalability
limitations on data storage while maintaining low latency and predictable performance.

The tutorial below explains how to integrate the DynamoDB ObjectMapper with your app, which stores
Java objects in DynamoDB.

Project Setup
=============

Prerequisites
-------------

You must complete all of the instructions on the `Set Up the SDK for Android
<http://docs.aws.amazon.com/mobile/sdkforandroid/developerguide/setup.html>`_ page before beginning
this tutorial.


Create a DynamoDB Table
=======================

Before you can read and write data to a DynamoDB database, you must create a table. When creating a
table you must specify the primary key.  The primary key is composed of a hash attribute and an
optional range attribute. For more information on how primary and range attributes are used, see
`Working With Tables`_.

#. Browse to the `DynamoDB Console`_ and click the :guilabel:`Create Table` button. The Create Table
   wizard appears.

#. Specify your table name, primary key type, hash attribute name, and range attribute name as shown
   below, and then click :guilabel:`Continue`:

   .. image:: ./images/create-table.png

#. Leave the edit fields in the next screen empty and click the :guilabel:`Continue` button.

#. Accept the default values for :guilabel:`Read Capacity Units` and :guilabel:`Write Capacity
   Units` and click the :guilabel:`Continue` button.

#. On the next screen enter your email address in the :guilabel:`Send notification to:` text box and
   click the :guilabel:`Continue` button.

#. On the next screen you will see a review of your settings, click the :guilabel:`Create` button.
   It may take a few minutes for your table to be created.


Update IAM Roles
================

In order for your Cognito identity pool to access Amazon DynamoDB, you must modify the identity
pool's roles.

#. Navigate to the `Identity and Access Management Console`_ and click :guilabel:`Roles` in the
   left-hand pane and search for your Identity Pool name - two roles will be listed one for
   unauthenticated users and one for authenticated users.

#. Click the role for unauthenticated users (it will have "unauth" appended to your Identity Pool
   name) and click the :guilabel:`Create Role Policy` button.

#. Select :guilabel:`Policy Generator` and click the :guilabel:`Select` button.

#. In the Edit Permissions page enter the settings shown in the following image. The Amazon Resource
   Name (ARN) of a DynamoDB table looks like
   :code:`arn:aws:dynamodb:us-west-2:123456789012:table/my-table-name` and is composed of the region
   in which the table is located, the owner's AWS account number, and the name of the table in the
   format :file:`table/my-table-name`. For more information about specifying ARNs, see `Amazon
   Resource Names for DynamoDB`_.

   .. image:: images/edit-permissions-dynamodb.png

#. Click the :guilabel:`Add Statement` button, click the :guilabel:`Next Step` button and the Wizard
   will show you the configuration generated.

#. Click the :guilabel:`Apply Policy` button.


Add Import Statements
---------------------

Add the following imports to the main activity of your app::

    import com.amazonaws.auth.CognitoCachingCredentialsProvider;
    import com.amazonaws.regions.Regions;
    import com.amazonaws.services.dynamodbv2.*;
    import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.*;


Initialize AmazonDynamoDBClient
===============================

Pass your initialized Amazon Cognito credentials provider to the :code:`AmazonDynamoDB`
constructor::

    AmazonDynamoDBClient ddbClient = new AmazonDynamoDBClient(credentialsProvider);


Initialize DynamoDBMapper
=========================

Pass your initialized DynamoDB client to the :code:`DynamoDBMapper` constructor::

    DynamoDBMapper mapper = new DynamoDBMapper(ddbClient);


Write a Row
===========

To write a row to the table, define a class to hold your row data. This class must be derived from
:code:`AWSDynamoDBModel` and implement the :code:`AWSDynamoDBModel` interface. The class should also
contain properties that hold the attribute data for the row.  The following class declaration
illustrates such a class::

    @DynamoDBTable(tableName = "Books")
        public class Book {
            private String title;
            private String author;
            private int price;
            private String isbn;
            private Boolean hardCover;

            @DynamoDBIndexRangeKey(attributeName = "Title")
            public String getTitle() {
                return title;
            }

            public void setTitle(String title) {
                this.title = title;
            }

            @DynamoDBIndexHashKey(attributeName = "Author")
            public String getAuthor() {
                return author;
            }

            public void setAuthor(String author) {
                this.author = author;
            }

            @DynamoDBAttribute(attributeName = "Price")
            public int getPrice() {
                return price;
            }

            public void setPrice(int price) {
                this.price = price;
            }

            @DynamoDBHashKey(attributeName = "ISBN")
            public String getIsbn() {
                return isbn;
            }

            public void setIsbn(String isbn) {
                this.isbn = isbn;
            }

            @DynamoDBAttribute(attributeName = "Hardcover")
            public Boolean getHardCover() {
                return hardCover;
            }

            public void setHardCover(Boolean hardCover) {
                this.hardCover = hardCover;
            }
        }

To save an object, first create it and set the appropriate fields::

	Book book = new Book();
	book.setTitle("Great Expectations");
	book.setAuthor("Charles Dickens");
	book.setPrice(1299);
	book.setIsbn("1234567890");
	book.setHardCover(false);

Then save the object::

    mapper.save(book);

To update a row, modify the instance of the :code:`DDTableRow` class and call
:code:`AWSDynamoObjectMapper.save()` as shown above.


Retrieve a Row
==============

Retrieve an object using a primary key::

    Book selectedBook = mapper.load(Book.class, "1234567890");

For more information on accessing DynamoDB from an Android application, see `Amazon Dynamo DB
<http://docs.aws.amazon.com/mobile/sdkforandroid/developerguide/dynamodb_om.html>`_.

.. _DynamoDB Console: https://console.aws.amazon.com/dynamodb/home
.. _Cognito Console: https://console.aws.amazon.com/cognito/home
.. _Identity and Access Management Console: https://console.aws.amazon.com/iam/home
.. _Amazon Resource Names for DynamoDB: http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/UsingIAMWithDDB.html#ARN_Format
.. _Working With Tables: http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithTables.html
