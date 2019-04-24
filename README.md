### SalesforceREST

[Credera Post](https://www.credera.com/blog/technology-solutions/how-to-use-the-salesforce-rest-api/)

Software applications that can integrate with Salesforce have become an essential part of many business processes. On a recent client project, we developed software that could upload data to a Salesforce server and then regularly synchronize it with an outside database. As it happens, Salesforce uses a database system of its own, and we found its APIs for remotely reading, writing, and updating information to be tremendously helpful.

In this blog post we will discuss how to call Salesforce’s APIs to operate on its data. We will begin with a brief introduction to the query language SOQL, followed by an overview of the Salesforce REST API.

SOQL
Salesforce’s web interface provides a developer console that we can use to test different queries. The developer console link can be found in a dropdown menu in the top-right of the web interface (under your user’s name in the classic view or the gear icon if you are using the newer Lightning view).

Click the Query Editor tab at the bottom of the console window, and then you can write and execute SOQL queries using the text box that appears below.



The query above is identical in nature to SQL. In this case, we are requesting all Ids and Names from the Contact table (or “object” to use the Salesforce term). The Id field is a primary key for the record, distinct across the entirety of the Salesforce system.

The Salesforce documentation provides a list of standard Salesforce objects and their fields. Depending on your Salesforce configuration, there may be other custom objects and fields that are defined in the setup section of the web user interface.

SOQL has several differences from normal SQL. One of the first you will encounter is a lack of support for field wildcards: you cannot request all fields from an object using a “SELECT * FROM …” query, and must instead specify each field you need. More notably, you will find that joins in SOQL work very differently from SQL.

You can perform limited subqueries in SOQL. For example, the following query will return Salesforce IDs for all Accounts tied to Contacts with names that include “david”:

Select Id
from Account
where Id in (Select AccountId
            from Contact
            where Name like '%david%')
Sadly, we cannot use subqueries to return fields from multiple objects. In the query above, we cannot extract fields from the Contact object to return as part of our results.

However, Salesforce does provide a mechanism to retrieve fields from multiple objects through relational fields. Any foreign key field (in this case, AccountId) can generate a reference to the record associated with its ID. For standard object fields, simply remove the “Id” suffix from the field name and then reference fields using dot notation. For example, to get the names of the Accounts associated with all Davids in our Contact object, we can use the following query:

Select AccountId, Name, Account.Name
from Contact
where Name like '%david%'
When using custom objects and fields, Salesforce requires you to use a “__r” suffix to indicate that you are using a relational field. You can get more information from the Salesforce documentation.

API
Salesforce provides REST APIs for operating on its data. In this section we will discuss examples of API calls for reading, writing, and updating records.

Before making any request through the REST API, you must be able to retrieve OAuth tokens to put in your request headers. Configuring OAuth client authorization is outside the scope of this blog post, but more information may be found in the Salesforce developer documentation. Once you have an authorized client, you can retrieve tokens from /services/oauth2/token.

Read API
To query for Salesforce data, we can send a GET request to the following URL:

https://<yourInstance>.salesforce.com/services/data/v<version>/query/?q=<soqlQuery>

Replace the <version> with the version number of the API you are using. For these examples we used v38.0.

The SOQL query associated with the q parameter should be HTML encoded.

Salesforce returns query responses using pagination, producing different URLs to call for each successive page of results. This means that any application that calls the search API should be able to recursively call each page URL until it has the entire result set.

The response JSON is in the following format:

{
    "totalSize": <totalResults>,
    "done": <isDone>,
    "nextRecordsUrl": "/services/data/v38.0/query/<nextPageId>",
    "records": [
        {
            "attributes": {
                "type": "<objectName>",
                "url": "/services/data/v38.0/sobjects/<objectName>/<objectId>"
            },
            <fieldName>: <fieldValue>,
            ...
        },
        ...
    ]
}
If the “done” field is set to false, the “nextRecordsUrl” field will be populated with the path to the next page of results. On the final page, “done” will be set to true and no “nextRecordsUrl” field will be provided.

Write API
To write a new Salesforce record, we use a POST request:

https://<yourInstance>.salesforce.com/services/data/v<version>/sobjects/<objectName>/

In the body of the POST request, provide the record data in application/json format:

{
    <field1Name>: <field1Value>,
    <field2Name>: <field2Value>,
    ...
}
You do not need to provide an ID field to Salesforce when creating records, as Salesforce will generate a unique ID value and return it in the response.

Update API
To update an existing Salesforce record, we use a PATCH request:

https://<yourInstance>.salesforce.com/services/data/v<version>/sobjects/<objectName>/<salesforceId>

Replace <salesforceId> with the unique Salesforce ID of the record you wish to update.

As with the write API, we must provide the record data in the request body using application/json format. However, for an update, we need only provide the specific fields we wish to update.

CONCLUSION
Salesforce’s SOQL query language and data access APIs are extraordinarily useful in software development, allowing us to use Salesforce as a writable, searchable database. Other APIs we did not cover allow record deletion, inserting blob data, and batch request processing.

Here we have focused on interacting with Salesforce from code bases that exist outside of the Salesforce system. For more information on development within Salesforce, see our blog series about the new Salesforce DX toolset.

To speak with Credera consultants about Salesforce development, please reach out to us at findoutmore@credera.com. We look forward to chatting with you!
