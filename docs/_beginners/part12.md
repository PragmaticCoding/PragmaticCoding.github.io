---
title: "Part 12 - Adding a New Field: eMail"
short_title: Part 12
permalink: /beginners/part12
screenshot_1: /assets/beginners/part10_1.png
excerpt: It's finally time to add a third field to our screen!  In this article we're going to add just a single new field, for email, to our screen.  We're going to see how the process goes from the View all the way down to the Broker.  Most importantly, we going to see how we need to be vigilant in ensuring that our code stays clean as we add more features.
---

# What You'll Learn

1. How easy it is to expand the application by a single field
1. Some more DRY
1. How to keeping code clean as we expand functionality

# Adding a Field

At this point we're going to add a single field to our screen.  We're going to do just one field because that will clearly show the process for extending the scope of our application, in this respect, down through the entire application.  Later on, we'll add a bunch of fields at once, but it will be virtually the same operation, but just with more than one field.

As we do this, we're going to see the potential for this process to corrupt our nice clean code.  We vigilant about how our code follows the two key priciples:  Dry and the Single Responsibility Principle.

# Start with the Data

We have three kinds of data in our application:

1. The Model - which represents State
1. Customer - which is a "Domain Object"
1. Database record - which is the external data format of our database

Since our database records are essentially free-form, there won't be any need to make any changes there, nor will we need to write any new tests at that layer of our application.  

Let's look at the new Model:

``` java
public class CustomerModel {

    private final StringProperty accountNumber = new SimpleStringProperty("");
    private final StringProperty customerName = new SimpleStringProperty("");
    private final StringProperty email = new SimpleStringProperty("");
    private final BooleanProperty okToSave = new SimpleBooleanProperty(false);
}
```

and now, the Customer:

``` java
public class Customer {

    private String accountNumber = "";
    private String name = "";
    private String email = "";
}

```

All the getters and setters have been left out - you should know how that stuff works.  

These changes are pretty trivial, we've simply added a new String-based field to each class.

# The Broker

The job of the Broker is to turn the Customer object into a database record.  So we need to account for the new field:

``` java
Map<String, String> createCustomerRecord(Customer customer) {
    Map<String, String> customerRecord = new HashMap<>();
    customerRecord.put("name", customer.getName());
    customerRecord.put("account_number", customer.getAccountNumber());
    customerRecord.put("email", customer.getEmail());
    return customerRecord;
}
```

Of course, new functionality means a new test:

``` java
class CustomerBrokerTest {

    @Test
    void createCustomerRecord_AccountNumberTest() {
        CustomerBroker broker = new CustomerBroker();
        Customer customer = createCustomer();
        assertEquals("1234", broker.createCustomerRecord(customer).get("account_number"), "Account number check");
    }

    @Test
    void createCustomerRecord_CustomerNameTest() {
        CustomerBroker broker = new CustomerBroker();
        Customer customer = createCustomer();
        assertEquals("Fred", broker.createCustomerRecord(customer).get("name"), "Customer name check");
    }

    @Test
    void createCustomerRecord_EmailTest() {
        CustomerBroker broker = new CustomerBroker();
        Customer customer = createCustomer();
        assertEquals("abc@def.com", broker.createCustomerRecord(customer).get("email"), "Customer email check");
    }

    private Customer createCustomer() {
        Customer customer = new Customer();
        customer.setAccountNumber("1234");
        customer.setName("Fred");
        customer.setEmail("abc@def.com");
        return customer;
    }
}
```

It's become obvious that we were getting into some repeated code here, and it was only going to get worse as we added more fields.  So DRY kicks in and we've put the repeated code into a method, `createCustomer()`.  Other than that, we've just added a single test case to handle the email address.

# The Interactor

This change affects the Interactor in one spot - when it creates the `Customer` from the Model.  So we'll update that method and then add a test case:

``` java
Customer createCustomerFromModel() {
    Customer customer = new Customer();
    customer.setAccountNumber(model.getAccountNumber());
    customer.setName(model.getCustomerName());
    customer.setEmail(model.getEmail());
    return customer;
}
```

and the test class:

``` java
class CustomerInteractorTest {

    @Test
    void createCustomer_NameTest() {
        CustomerModel model = createCustomerModel();
        CustomerInteractor interactor = new CustomerInteractor(model);
        assertEquals("Fred", interactor.createCustomerFromModel().getName(), "Check customer name");
    }

    @Test
    void createCustomer_AccountTest() {
        CustomerModel model = createCustomerModel();
        CustomerInteractor interactor = new CustomerInteractor(model);
        assertEquals("ABCDE", interactor.createCustomerFromModel().getAccountNumber(), "Check customer name");
    }

    @Test
    void createCustomer_EmailTest() {
        CustomerModel model = createCustomerModel();
        CustomerInteractor interactor = new CustomerInteractor(model);
        assertEquals("abc@def.com", interactor.createCustomerFromModel().getEmail(), "Check customer email");
    }

    private CustomerModel createCustomerModel() {
        CustomerModel model = new CustomerModel();
        model.setCustomerName("Fred");
        model.setAccountNumber("ABCDE");
        model.setEmail("abc@def.com");
        return model;
    }
}
```

You'll see that we've applied DRY again here, to keep our code as simple as possible.

# Finally, The View

At this point we have a back-end that will handle our new field; it's tested and ready to go.  All we need to do now is to add it to our View:

``` java
private Node createCentre() {
    VBox results = new VBox(6, accountBox(), nameBox(), emailBox());
    results.setPadding(new Insets(20));
    return results;
}

private Node accountBox() {
    return new HBox(6, promptLabel("Account #:"), boundTextField(model.accountNumberProperty()));
}

private Node nameBox() {
    return new HBox(6, promptLabel("Name:"), boundTextField(model.customerNameProperty()));
}

private Node emailBox() {
    return new HBox(6, promptLabel("eMail:"), boundTextField(model.emailProperty()));
}
```

Do you see how simple that is?  We just add a new builder method to create the `HBox`, then call it from the constructor of the Centre's `VBox`.  

However, we do have some repeated code that's starting to get ugly.  This is a little bit harder to see, since we've already applied DRY to pull the `HBox` creation out into separate methods.  But all three of these methods are pretty much the same, we just put in different strings and Properties.  So let's fix that:

Let's try it:

``` java
private Node createCentre() {
    VBox results = new VBox(6, accountBox(), nameBox(), emailBox());
    results.setPadding(new Insets(20));
    return results;
}

private Node accountBox() {
    return rowBox("Account #:", model.accountNumberProperty());
}

private Node nameBox() {
    return rowBox("Name:", model.customerNameProperty());
}

private Node emailBox() {
    return rowBox("eMail:", model.emailProperty());
}

private Node rowBox(String prompt, StringProperty boundProperty) {
    return new HBox(6, promptLabel(prompt), boundTextField(boundProperty));
}
```

That's removed the repeated code, as much as we can.  We could stop at this point, but it's probably better to refactor out those three methods since they probably don't add much value any more.

The question here is:  Which is more clear?  Having methods with names that describe what they are doing, or having less code?  Personally, I lean towards less code, following the idea that, "less code is better code."

Let's see what it looks like:

``` java
private Node createCentre() {
    VBox results = new VBox(6,
            rowBox("Account #:", model.accountNumberProperty()),
            rowBox("Name:", model.customerNameProperty()),
            rowBox("eMail:", model.emailProperty()));
    results.setPadding(new Insets(20));
    return results;
}

private Node rowBox(String prompt, StringProperty boundProperty) {
    return new HBox(6, promptLabel(prompt), boundTextField(boundProperty));
}
```

Honestly, I think this *is* clearer.  The prompt text is what you'll see on the screen, so it's what you'd be looking for in the code.  The method names are OK, but this is much more compact, and the intent of the code is obvious.  If you want to know the gory details about `rowBox()`, then go look at it.  But you probably won't.

And if there were 30 rows of prompt/`TextFields`, this code would still be clear.  You can also see that adding a new field just adds another parameter to the `VBox` constructor.  We'll stick with this.

# Wrapping it Up

At this point it should be clear that adding a new field, or a dozen new fields, is just a matter of adding a row to the GUI and then following the data down to the Broker.  It's rinse and repeat, "paint by numbers" for each field.

What's far more important is that you keep your code clean as do this.  Things that seemed reasonable with just one or two fields become obviously bad as you add more and more to your application.  If not actually "bad", then certainly "sub-optimal".  Most of the things we fixed in the lesson were already issues with the code; things that, to be honest, we should have already fixed.  They really jumped out as problems once we added that third field.

You need to constantly take a step back and look at your code with a critical eye and make sure that you're keeping on top of emerging issues.  Make sure that you're following DRY and the Single Responsibility Principle.  Make sure that in those cases where you are not, that it's because you think the code is cleaner in its current state.  

And don't forget, as we've seen here, all of this goes for your JUnit test classes as well.  
