---
layout: post
title: "Getting Started with FakeXrmEasy"
description: "How to use FakeXrmEasy to make unit testing your D365 CE easier and faster."
thumb_image: ""
tags: [dynamics]
---

Hat tip to [Dini Ghanda](https://www.linkedin.com/in/dini-ganda-23b28376/?originalSubdomain=nz) for pointing out the [Fake Xrm Easy framework](https://dynamicsvalue.com/home).

Here's all you need to do to unit test your D365 CE code.

## Step 1: Add Nuget Packages
1. Create a Unit Test Project if you don't have one.
2. Add the FakeXrmEasy.9 package (or FakeXrmEasy.365/2016 etc for earlier versions of CRM).
3. Optionally, add your choice of test framework/runner packages (like NUnit or XUnit).

## Step 2: Create a Test Method
1. First, some using statements
{% highlight cs %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;

using Microsoft.VisualStudio.TestTools.UnitTesting;
using Microsoft.Xrm.Sdk;
using FakeXrmEasy;
{% endhighlight %}
{:start="2"}
2. Then, the test method (with explanatory comments)
{% highlight cs %}
[TestMethod]
public void UpdateLockValueTest()
{
    var context = new XrmFakedContext
    {
        // Tell the context to use the assembly with our early-bound classes
        // for typed entities.
        ProxyTypesAssembly = Assembly.GetAssembly(typeof(Contact))
    };
    // Add the objects/state we need for queries and initialise the context.
    var contactId = Guid.NewGuid();
    var contact = new Contact
    {
        Id = contactId,
        ContactId = contactId,
        StateCode = ContactState.Active
    };
    context.Initialize(new List<Entity>() { contact });

    // All the mocks are belong to us.
    var service = context.GetOrganizationService();
    var trace = context.GetFakeTracingService();
    var target = new OrganisationIndividualHelper(service, trace, null);
    //var dataContext = new XrmServiceContext(service); // Ideally, in a using statement.

    // Execute our code under test.
    // FakeXrmEasy's in-memory database and query execution engine
    // means we can test everying.
    target.UpdateLockValue(contactId);

    // Check the expected stuff happenedf
    var actual = context.CreateQuery<Contact>()
        .Where(c => c.Id == contactId)
        .Select(c => c.fus_LockValue).FirstOrDefault();

    Assert.IsTrue(!string.IsNullOrWhiteSpace(actual));
}
{% endhighlight %}

It's that easy. Well, almost. One quirk I found was an unhandled exception in a LINQ query with a condition to check an OptionSetValueCollection wasn't null. It worked fine after I removed that condition (and checked it in the code that processed the query results).

We also need to provide a bit more information to FakeXrmEasy for it to deal with m:n relationships. For example:
{% highlight cs %}
context.AddRelationship("systemuserroles_association", new XrmFakedRelationship
{
    IntersectEntity = "systemuserroles",
    Entity1LogicalName = SystemUser.EntityLogicalName,
    Entity1Attribute = "systemuserid",
    Entity2LogicalName = Role.EntityLogicalName,
    Entity2Attribute = "roleid"
});
{% endhighlight %}

And we can provide more information to FakeXrmEasy for it to deal with requests to retrieve attribute metadata. For example:
{% highlight cs %}
var accountMetadata = new EntityMetadata { LogicalName = Account.EntityLogicalName };
accountMetadata.SetAttribute(new PicklistAttributeMetadata
{
    SchemaName = "statecode",
    LogicalName = "statecode",
    DisplayName = new Label("Status", 1033),
    RequiredLevel = new AttributeRequiredLevelManagedProperty(
        AttributeRequiredLevel.SystemRequired),
    OptionSet = new OptionSetMetadata
    {
        IsGlobal = false,
        OptionSetType = OptionSetType.State,
        Options =
        {
            new OptionMetadata(new Label(
                new LocalizedLabel("Active", 1033), null), 0),
            // We can also do this if the code we're testing doesn't
            // rely on user localised labels.
            new OptionMetadata(new Label("Inactive", 1033), 1)
        }
    }
});
fakeContext.InitializeMetadata(accountMetadata);{% endhighlight %}

FakeXrmEasy also provides methods to test plug-in/workflow execution context. The simplest is ExecutePluginWithContext, but you can use GetDefaultPluginContext() to provide images etc. See [https://dynamicsvalue.com/get-started/plugins](https://dynamicsvalue.com/get-started/plugins).
