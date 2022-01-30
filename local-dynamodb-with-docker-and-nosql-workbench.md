# Setup a local DynamoDB with Docker and NoSQL Workbench
Tags: *AWS* | *Serverless* | *DynamoDB* | *NoSQL Workbench*

In this mini tutorial I want to show you how you can setup a local DynamoDB with Docker and connect your database with NoSQL Workbench. This post is an overview where I 
will not deep-dive into the details. However if you have any questions just let me know.

## Why NoSQL Workbench
It's totally fine to use AWS CLI to perform database actions. However sometimes having some visuals and data modeling tools around comes in handy. Workbench helps
with creating and designing data models and queries. For me this had proven to be very helpful, as desiging data models and access patterns for this key/value database
is not always one of the easiest things to do.

## Prerequisites
* Docker installed
* NoSQL Workbench installed
* AWC CLI installed
* Some understanding of above tools

## Setup DynamoDB in a local network with Docker
* First setup a local network using Docker. From your terminal type `docker network create [NETWORK]`
* Create a DynamoDB container with `docker run -d -p 8000:8000 --network=[NETWORK] --name [NETWORK_NAME] amazon/dynamodb-local`

The steps above create a local network with a container for DynamoDB based on Amazon's DynamoDB image. You now have a working DynamoDB locally you can access using port 8000.

Test if your setup is working by typing `aws dynamodb list-tables --endpoint-url http://localhost:8000`. If everything works ok, you will get following output:

```json
{
    "TableNames": []
}
```

## Setup NoSQL Workbench
*"NoSQL Workbench is a unified visual IDE tool that provides data modelling, data visualisation, and query development features to help you design, create, query, and manage DynamoDB tables."*

The introduction from AWS says it all. After using it for a while I can appreciate the visual editor. It makes life a bit easier for local development. It can also connect to remote databases.
More information and download here: [https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html)

### Connect NoSQL Workbench to your local DynamoDB
If your local network on Docker is up and running we can go ahead and connect Workbench to our network. 

* From the *Operations Builder* menu, select *Add connection*
* Select *DynamoDB local* in the popup screen
* Name your connection (anything you want)
* Select the port (default 8000)
* Hit *connect*

You now have to add a datamodel with a table and some data to your database. For this mini tutorial we will be working with the samples provided by AWS in Workbench.

* From the *Amazon DynamoDB* tab, select a data model and hit *import*
* In the next screen (*Data modeller* tab), click *Visualise datamodel*
* You can now click *Commit to Amazon DynamoDB*

These steps add a table and some data to your database.

After this, when you type `aws dynamodb list-tables --endpoint-url http://localhost:8000` in your terminal, your tables are... still empty! To save you the Googling, 
we need to add our credentials and region in the mix so the CLI has the correct rights to access our local tables.

There are a couple of ways to do this. You can add them to your terminal command like this: `AWS_DEFAULT_REGION=localhost AWS_ACCESS_KEY_ID=id AWS_SECRET_ACCESS_KEY=secret aws dynamodb list-tables --endpoint-url http://localhost:8000`.
You can find your access keys in Workbench. In the *Operation builder* tab (the overview page of all of your connections), you can click on the three dots just next to
the "open" button of your connection.

Another way is to add your credentials and region in your AWS credentials file (default profile). Use "localhost" as your region and add your keys accordingly. You can also put them
in an env file of some sort. Pick whatever suits your project. As long as you add them.

Now when you list your tables via terminal you should see your tables in the output.

Now you should be able to perform actions on your local DynamoDB!

*More on how to connect your Lambda code to your local DynamoDB in the next post.*

## Conclusion
Although most developers will use the command line to perform database actions, having some visuals when working with DynamoDB does make developer life easier.
Setting up a DynamoDB locally speeds up development and testing. Even when your internet connection is down. 
Having Workbench help you to build data models and queries and even generate code for you has proven to be very helpful in speeding up things. Hope this post has helped
you, if any questions let me know.






