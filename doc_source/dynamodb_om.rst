.. Copyright 2010-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. highlight:: java

##############################################
Store and Retrieve App Data in Amazon DynamoDB
##############################################

What is Amazon DynamoDB?
========================

|DDB|_ is a fast, highly scalable, highly available, cost-effective, nonrelational database service.
|DDB| removes traditional scalability limitations on data storage while maintaining low latency and
predictable performance.

The |sdk-android| provides a high-level library for working with |DDB|. The library includes the
|DDB| Object Mapper, which lets you map client-side classes to |DDB| tables; perform various
create, read, update, and delete (CRUD) operations; and execute queries. Using the |DDB| Object
Mapper, you can write simple, readable code that stores objects in the cloud.

For information about |DDB| Region availability, see `AWS Service Region Availability
<http://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/>`_.

Getting Started
===============

This section provides a step-by-step guide for getting started with |DDB| using the |sdk-android|.
You can also try out the `DynamoDB sample
<https://github.com/awslabs/aws-sdk-android-samples/tree/master/DynamoDBMapper_UserPreference_Cognito>`_.

Include the JAR Files in Your Project
-------------------------------------

Follow the instructions on the :doc:`setup` page to include the proper JAR files for this service
and set the appropriate permissions.

Add Import Statements
---------------------

Add the following imports to the main activity of your app::

    import com.amazonaws.auth.CognitoCachingCredentialsProvider;
    import com.amazonaws.regions.Regions;
    import com.amazonaws.services.dynamodbv2.*;
    import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.*;
    import com.amazonaws.services.dynamodbv2.model.*;

Set Permissions in Your Android Manifest
----------------------------------------

In :file:`AndroidManifest.xml`, set the following permission, if it's not already present

.. code-block:: xml

    <uses-permission android:name="android.permission.INTERNET" />

Create an Identity Pool
-----------------------

To use AWS services in your mobile application, you must obtain AWS Credentials using Amazon Cognito
Identity as your credential provider. Using a credentials provider allows you to access AWS services
without having to embed your private credentials in your application. This also allows you to set
permissions to control which AWS services your users have access to.

The identities of your application's users are stored and managed by an identity pool, which is a
store of user identity data specific to your account. Every identity pool has roles that specify
which AWS resources your users can access. Typically, a developer will use one identity pool per
application. For more information on identity pools, see the `Cognito Developer Guide
<http://docs.aws.amazon.com/cognito/devguide/identity/identity-pools/>`_.

To create an identity pool for your application:

#. Log in to the `Cognito Console <https://console.aws.amazon.com/cognito/home>`_ and click
   :guilabel:`Manage Federated Identities`, then :guilabel:`Create new identity pool`.

#. Enter a name for your Identity Pool and check the checkbox to enable access to unauthenticated
   identities. Click :guilabel:`Create Pool` to create your identity pool.

#. Click :guilabel:`Allow` to create the roles with access to your new identity pool.

The next page displays code that creates a credentials provider so you can easily integrate Cognito
Identity in your Android application.

For more information about |COG| Identity, see :doc:`cognito-auth`.

Create a DynamoDB Table
-----------------------

Let's assume we're building a bookstore app. The app will need to keep track of books available in a
bookstore, and we can create a DynamoDB table to do so.

To create the Books table:

#. Log in to the `DynamoDB Console <https://console.aws.amazon.com/dynamodb/home>`_.
#. Click :guilabel:`Create Table`.
#. Enter :command:`Books` as the name of the table.
#. Enter :command:`ISBN` in the :guilabel:`Partition key` field of the :guilabel:`Primary key` with :guilabel:`String` as its type.
#. Uncheck the :guilabel:`Use default settings` checkbox and click :guilabel:`+ Add Index`.
#. In the :guilabel:`Add Index` dialog enter :command:`Author` with :guilabel:`String` as its type.
#. Check the :guilabel:`Add sort key` checkbox and enter :command:`Title` as the sort key value, with :guilabel:`String` as its type.
#. Leave the other values at their defaults and click :guilabel:`Add index` to add the :command:`Author-Title-index` index.
#. Set the read capacity to ``10`` and the write capacity to ``5``.
#. Click :guilabel:`Create`. DynamoDB will create your database.
#. Refresh the console and select your Books table from the list of tables.
#. Open the :guilabel:`Overview` tab and copy or note the Amazon Resource Name (ARN). You'll need
   this in a moment.

Set Permissions
---------------

To use |DDB| in an application, you must set the correct permissions. The following IAM policy
allows the user to perform the actions shown in this tutorial on two resources (a table and an
index) identified by `ARN
<http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html>`_.

.. code-block:: json

    {
        "Statement": [{
            "Effect": "Allow",
            "Action": [
                "dynamodb:DeleteItem",
                "dynamodb:GetItem",
                "dynamodb:PutItem",
                "dynamodb:Scan",
                "dynamodb:Query",
                "dynamodb:UpdateItem",
                "dynamodb:BatchWriteItem"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-west-2:123456789012:table/Books",
                "arn:aws:dynamodb:us-west-2:123456789012:table/Books/index/*"
            ]
        }]
    }

Apply this policy to the unauthenticated role assigned to your Cognito identity pool, replacing the
``Resource`` values with the correct ARN for your DynamoDB table:

#. Log in to the `IAM Console <https://console.aws.amazon.com/iam/home>`_.

#. Select :guilabel:`Roles` and select the "Unauth" role that Cognito created for you.

#. Click :guilabel:`Attach Role Policy`.

#. Select :guilabel:`Custom Policy` and click :guilabel:`Select`.

#. Enter a name for your policy and paste in the policy document shown above, replacing the
   ``Resource`` values with the ARNs for your table and index. (You can retrieve the table ARN from
   the :guilabel:`Details` tab of database; then append :file:`/index/*` to obtain the value for the
   index ARN.

#. Click :guilabel:`Apply Policy`.

To learn more about IAM policies, see `Overview of IAM Policies <http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html>`_. To learn more about DynamoDB-specific policies, see |ddb-dg| `Authentication and Access Control <http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/authentication-and-access-control.html>`_.

Create a DynamoDB Client and Object Mapper
==========================================

We're going to use the DynamoDB Object Mapper to map a client-side class to our database. To use the
Object Mapper, we first have to instantiate a DynamoDB client.

When we created an identity pool, we copied the Cognito client initialization code into our app.
Assuming that we have a ``credentialsProvider`` variable holding a reference to our Cognito
credential provider, we can create a DynamoDB client as follows::

    AmazonDynamoDBClient ddbClient = new AmazonDynamoDBClient(credentialsProvider);

Then we can use our DynamoDB client to create an Object Mapper::

    DynamoDBMapper mapper = new DynamoDBMapper(ddbClient);

Now we're ready to map a class to our database.


Define a Mapping Class
======================

In DynamoDB, a database is a collection of tables. A table can be described as follows:

* A table is a collection of items.
* Each item is a collection of attributes.
* Each attribute has a name and a value.

For our bookstore app, each item in the table will represent a book, and each item will have five
attributes: :dfn:`Title`, :dfn:`Author`, :dfn:`Price`, :dfn:`ISBN`, and :dfn:`Hardcover`.

Each item (Book) in the table will have a hash key |mdash| in this case, ISBN |mdash| which is the primary key for the table.

We're going to map each item in the Book table to a ``Book`` object in the Java code, so that we can
directly manipulate the database item through its object representation.

To establish mappings, DynamoDB defines annotations, including the following:

- :command:`@DynamoDBTable` |mdash| Identifies the target table in
  DynamoDB.

- :command:`@DynamoDBHashKey` |mdash| Maps a class property to the hash
  attribute of the table.

- :command:`@DynamoDBAttribute` |mdash| Maps a class property to an
  item attribute.

For a complete list of the annotations that the Object Mapper offers, see `Java Annotations for
DynamoDB
<http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/JavaDeclarativeTagsList.html>`_.

Let's create a ``Book`` mapping class::

    import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.*;

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

Note that ``hardCover`` is a nullable type. With the DynamoDB Object Mapper, primitives and nullable
types behave differently. On a ``save()``, an unset nullable type is not sent to DynamoDB; an unset
primitive is sent as its default value.

Interact with Stored Objects
============================

Now that we have a database, a mapping class, and an Object Mapper client, we can start interacting
with objects in the cloud. These calls are synchronous and must be taken off of the main thread. You can wrap the code with::
   Runnable runnable = new Runnable() {
        public void run() {
        //DynamoDB calls go here
   };
   Thread mythread = new Thread(runnable);
   mythread.start();


Save an Item
------------

To save an object, first create it and set the appropriate fields::

    Book book = new Book();
    book.setTitle("Great Expectations");
    book.setAuthor("Charles Dickens");
    book.setPrice(1299);
    book.setIsbn("1234567890");
    book.setHardCover(false);

Then use the Object Mapper client to write the object to a corresponding item in the table. In this
case, we'll call ``save()`` on the client and pass in our ``book`` object::

    mapper.save(book);

Except for the primary key (here "ISBN"), there is no predefined schema for the items in a table. We
can update our mapping class and add or remove attributes at will. An item can have any number of
attributes, although there is a limit of 400 KB on the item size.

Retrieve an Item
----------------

Using an object's primary key (in this case, the hash attribute "ISBN"), we can load the
corresponding item from the database. The following code snippet returns the Book item with an ISBN
of "1234567890"::

    Book selectedBook = mapper.load(Book.class, "1234567890");

Update an Item
--------------

To update an item in the database, just set new attributes and save the object again. For example,
we could update the price of a Book instance as follows::

    Book selectedBook = mapper.load(Book.class, "1234567890");
    selectedBook.setPrice(1199);
    mapper.save(selectedBook);

Note that setting a new hash key creates a new item in the database, even though it doesn't create a
new object on the client side. Consider the following example::

    Book selectedBook = mapper.load(Book.class, "1234567890");
    selectedBook.setIsbn("0987654321");
    mapper.save(selectedBook);

The result is a new item in the database, identical to the loaded item but with the new ISBN. The
reference ``selectedBook`` now maps to this new item in the database, but the old item also exists.


Delete an Item
--------------

To delete an item from the database, use the ``delete()`` method and pass in the object to be
deleted::

    mapper.delete(selectedBook);


Perform a Scan
==============

With a scan operation, we can retrieve all items from a given table. A scan examines every item in
the table and returns the results in an undetermined order::

    DynamoDBScanExpression scanExpression = new DynamoDBScanExpression();
    PaginatedScanList<Book> result = mapper.scan(Book.class, scanExpression);
    // Do something with result.

The returned list of items is lazily loaded when possible, so calls to DynamoDB are made only as
needed. When you need to download an entire dataset in advance, you can call the ``size()`` method
on the list to fetch the entire list.

The list returned by the Object Mapper can't be modified, and an attempt to do so results in an
exception. If you want to use the result of a scan as a data source for a modifiable user interface
component (for example, an editable ``ListActivity``), you'll need to create a modifiable list
object and move all of the data to it.

Scan is an expensive operation and should be used with care to avoid disrupting higher priority
traffic on the table. The *Amazon DynamoDB Developer Guide* has `Guidelines for Query and Scan
<http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html>`_ that explain
best  practices for scan operations.

Perform a Query
===============

A query operation lets us find items in a table using both hash and range key attributes. The
primary key for our Books table doesn't have a range key. However, when we created the table, we
specified a global secondary index, and that secondary index does have a range key attribute. We'll
perform a query against the hash key and the range key of our secondary index.

Secondary Indexes
-----------------

A secondary index is a data structure that contains a subset of attributes from a table, along with
an alternate key to support query operations. With a secondary index, queries are no longer
restricted to the table primary key; we can retrieve data using the alternate key, too.

The data in a secondary index consists of attributes that are projected, or copied, from the table
into the index. Every secondary index is automatically maintained by DynamoDB. When we add, modify,
or delete items in the table, any indexes on the table are also updated to reflect these changes.

To learn more about secondary indexes, see `Improving Data Access with Secondary Indexes
<http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html>`_.

Query Example
-------------

The following example performs a query for books by the author "Charles Dickens" with a title
beginning with "Great"::

    Book bookToFind = new Book();
    bookToFind.setAuthor("Charles Dickens");

    String queryString = "Great";

    Condition rangeKeyCondition = new Condition()
            .withComparisonOperator(ComparisonOperator.BEGINS_WITH.toString())
            .withAttributeValueList(new AttributeValue().withS(queryString.toString()));

    DynamoDBQueryExpression queryExpression = new DynamoDBQueryExpression()
            .withHashKeyValues(bookToFind)
            .withRangeKeyCondition("Title", rangeKeyCondition)
            .withConsistentRead(false);

    PaginatedQueryList<Book> result = mapper.query(Book.class, queryExpression);
    // Do something with result.

We begin by creating a book object and setting the hash key attribute that we want to query against.
The global secondary index for our Books table uses Author as a hash key, so we set the Author
attribute for the Book item we're looking for.

Then we create a range key condition, which represents the selection criteria for our query. In this
case, we want to select attribute values beginning with the string "Great".

When we create ``DynamoDBQueryExpression``, we set the hash key value and the range key condition
for the query. Note that the first parameter to ``withRangeKeyCondition`` is the range key attribute
name.

Finally, we create a ``PaginatedQueryList<T>`` to represent the results from the query. Like the
scan result list, the query result list can't be modified.

Conditional Writes
==================

In a multi-user environment, multiple clients can access the same item and attempt to modify its
attribute values at the same time. To help clients coordinate writes to data items, the DynamoDB
low-level client supports conditional writes for ``PutItem``, ``DeleteItem``, and ``UpdateItem``
operations. With a conditional write, an operation succeeds only if the item attributes meet one or
more expected conditions; otherwise, it returns an error.

In the following example, we update the price of an item in the Books table *if* the item has a
"Price" value of "1299"::

    try {
        HashMap<String, AttributeValue> primaryKey = new HashMap<>();
        AttributeValue isbn = new AttributeValue()
                .withS("1234567890");
        primaryKey.put("ISBN", isbn);

        UpdateItemRequest request = new UpdateItemRequest()
                .withTableName("Books")
                .withKey(primaryKey)
                .addAttributeUpdatesEntry(
                        "Price", new AttributeValueUpdate()
                                .withValue(new AttributeValue().withN("1199"))
                                .withAction(AttributeAction.PUT))
                .addExpectedEntry(
                        "Price", new ExpectedAttributeValue()
                                .withValue(new AttributeValue().withN("1299"))
                                .withComparisonOperator(ComparisonOperator.EQ));

        ddbClient.updateItem(request);

    }
    catch (ConditionalCheckFailedException e) {
        // The conditional check failed.
    }

In this example, we construct an `UpdateItemRequest
<http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/services/dynamodbv2/model/UpdateItemRequest.html>`_
to pass to ``updateItem()`` on the DynamoDB client. The ``UpdateItemRequest`` object calls
``addAttributeUpdatesEntry``, which specifies the name of the attribute to update, the new value for
the attribute, and the action to perform on the attribute. To add a condition, we also call
``addExpectedEntry``, which is the conditional block for the operation. In this case, the
``ComparisonOperator`` is checking that the price of the item equals (``EQ``) "1299". If this is not
the case, the update fails.

Note that conditional writes are idempotent. This means that you can send the same conditional write
request multiple times, but it will have no further effect on the item after the first time DynamoDB
performs the specified update.

Batch Operations
================

The DynamoDB Object Mapper provides batch write operations to put items in the database and delete
items from the database. The following example illustrates a batch put operation using the
``batchSave`` method::

    Book book1 = new Book();
    book1.setTitle("Moby-Dick; or, The Whale");
    book1.setAuthor("Herman Melville");
    book1.setPrice(999);
    book1.setIsbn("7654321098");
    book1.setHardCover(false);

    Book book2 = new Book();
    book2.setTitle("Madame Bovary");
    book2.setAuthor("Gustave Flaubert");
    book2.setPrice(1099);
    book2.setIsbn("6543210987");
    book2.setHardCover(true);

    Book book3 = new Book();
    book3.setTitle("The Brothers Karamazov");
    book3.setAuthor("Fyodor Dostoyevsky");
    book3.setPrice(1399);
    book3.setIsbn("5432109876");
    book3.setHardCover(false);

    mapper.batchSave(Arrays.asList(book1, book2, book3));

The ``batchSave`` method saves items into the database. We can use ``batchDelete`` to delete items
from the database and ``batchWrite`` to either save or delete items.
