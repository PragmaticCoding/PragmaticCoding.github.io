---
title: Part 10 - Handling Database Errors
short_title: Part 10
permalink: /beginners/part10
screenshot_1: /assets/beginners/part10_1.png
excerpt: The last problem with our application is that it can save duplicate customer records, corrupting our database.  This means adding some rules to our database and telling our GUI when those rules have been broken.  We'll see how to handle exceptions from the back-end in our application.
---

# What You'll Learn

1. Dealing with external API failures
1. The difference between checked and unchecked exceptions
1. How and where to handle exceptions
1. Translating exceptions into Task results
1. Communicating errors to the users through Alerts


# The Last Functional Issue

The last problem we have with out application is that it is possible to save customer records with duplicate account numbers.  This is different from the last problem because we cannot know that there is an issue until *after* we attempt to save it in the database.  Even if we wanted to build a retrieve function to check first, we still have to go to the database to determine if there is a problem.  

In a lot of databases, you'd define the account number that we're using as a "unique index".  This would mean that it was defined in our database as a field that we wish to do quick look-ups on, and that it will always have unique values.  Attempting to save a new record with a value in a unique index field would generate an error.

It's not really reasonable to take all of the rules in our database and bake them into our front end.  At some point, we have to cope with the fact that our database might reject our update and somehow handle that situation and notify the user through the GUI.  That's what we are going to do here.

# Defining the Rule in the Database

The bulk of the work in this section is actual about how to get our database to simulate a real-world database that enforces rules about how data can be saved.  So let's get to it...

In order to enforce this rule in the database, we're going to need a way to report that it has been broken.  This will be with a Checked Exception:

``` java
public class CustomerAlreadyOnFileException extends Exception {

    public CustomerAlreadyOnFileException(String accountNumber) {
        super("Customer account: " + accountNumber + " is already on file");
    }
}
```

That's pretty simple.  Now, we are going to need to put some code in our database to establish "account_number" as a unique index in our database:

``` java
public class CustomerDatabase {

    private final Map<Integer, Map<String, String>> data = new HashMap<>();
    private Integer nextKey = 0;

    public int saveNewCustomer(Map<String, String> customerRecord) throws CustomerAlreadyOnFileException, IllegalArgumentException {
        Optional<String> accountNumberOptional = getAccount(customerRecord);
        if (accountNumberOptional.isPresent()) {
            String accountNumber = accountNumberOptional.get();
            if (!isAccountOnFile(accountNumber)) {
                return saveCustomer(customerRecord);
            } else {
                throw new CustomerAlreadyOnFileException(accountNumber);
            }
        } else {
            throw new IllegalArgumentException("Account number must be included in Customer Record");
        }
    }

    int saveCustomer(Map<String, String> customerRecord) {
        customerRecord.put("_id", (++nextKey).toString());
        data.put(nextKey, customerRecord);
        return nextKey;
    }

    boolean isAccountOnFile(String accountNumber) {
        return data.values().stream().anyMatch(record -> getAccount(record).map(value -> value.equals(accountNumber)).orElse(false));
    }

    private Optional<String> getAccount(Map<String, String> customerRecord) {
        return Optional.ofNullable(customerRecord.get("account_number"));
    }

    Map<Integer, Map<String, String>> getData() {
        return data;
    }
}
```

We've now wrapped the call to `saveCustomer` in a new method called `saveNewCustomer` that has the logic to ensure that duplicate account numbers are not saved.  This new method is now the public method that will be called from the DAO.  The names accurately represent what they do.

In order to enforce the unique account numbers, we'll need to search through the existing records, extract the "account_number" field from each one and check if it matches our new account number, which we'll also have to extract from the new customer record.  

A method called `getAccount()` was created to do the extraction of the account number from a customer record.  At this point we need some further explanation:

Because "account_number" is now an index value, it's not allowed - at the database level - to be empty.  Ignore the fact that our front-end doesn't allow empty, that doesn't matter this far down as we cannot know what the GUI is doing.  This is now a database rule and needs to be enforced here.  So `getAccount()` now returns an `Optional`, and it's up to `saveNewCustomer()` to deal with the `Optional.isEmpty()` situation - which is to throw an exception and refuse to save the record.

## Checked vs Unchecked Exceptions

A little diversion here to explain why only one of these exceptions is checked...

Checked exceptions are used when there's a possibility of a method failing for unavoidable reasons that can occur at runtime.  In this case, we have a database that has no retrieve function, so there's no way for our DAO to check that the record isn't on file before attempting to save it.

But even if we did have a retrieve function there, it still might be reasonable to have a duplicate record.  Some other user, or some other process or thread might sneak in and create the record in between our call to do a test retrieve and our call to save the record.  It's extremely unlikely, but it's not *impossible*, and that means we have to have some way to deal with it in our code.

On the other hand, the requirement to have an "account_number" field in our new customer record is something that the DAO can check before calling `CustomerDatabase.saveNewCustomer()`.  We need to check for it in our database because allowing this to happen would corrupt our database, but it should never occur in production because the DAO should prevent it from happening.

Checked Exception
: An issue that cannot be avoided by writing good code, but must be handled by the code when it occurs in production.

Unchecked Exception
: An issue that needs to guarded against but can absolutely be avoided by writing good code in the calling method.  An unchecked exception usually indicates a program bug.

## Testing the Database

This is going to add a whole new set of testing requirements to our database.  So clean-up and reorganization of the test class has been done, as well as to add some new test cases:

``` java
class CustomerDatabaseTest {

    CustomerDatabase dataBase;

    @BeforeEach
    void init() {
        dataBase = new CustomerDatabase();
    }

    @Test
    void saveCustomerIdIncrementTest() {
        assertEquals(1, dataBase.saveCustomer(createFred()), "Id is incremented");
    }

    @Test
    void saveCustomerIdIncrementTwiceTest() {
        dataBase.saveCustomer(createFred());
        assertEquals(2, dataBase.saveCustomer(createGeorge()), "Id is incremented");
    }

    @Test
    void saveCustomerIdInsertionTest() {
        dataBase.saveCustomer(createFred());
        assertEquals("1", dataBase.getData().get(1).get("_id"), "Id is inserted");
    }

    @Test
    void saveCustomerRecordCorrectTest() {
        dataBase.saveCustomer(createFred());
        assertEquals("Fred", dataBase.getData().get(1).get("name"), "Id is inserted");
    }

    @Test
    void findAccountTest_Found() {
        dataBase.saveCustomer(createFred());
        dataBase.saveCustomer(createGeorge());
        assertTrue(dataBase.isAccountOnFile("123"), "Lookup Fred");
    }

    @Test
    void findAccountTest_NotFound() {
        dataBase.saveCustomer(createFred());
        dataBase.saveCustomer(createGeorge());
        assertFalse(dataBase.isAccountOnFile("7777"), "Lookup someone not present");
    }

    @Test
    void saveWithoutAccountNumber() {
        HashMap<String, String> customerRecord = new HashMap<>();
        customerRecord.put("name", "Fred");
        assertThrows(IllegalArgumentException.class, () -> dataBase.saveNewCustomer(customerRecord));
    }

    @Test
    void saveRecordTwiceTest() {
        dataBase.saveCustomer(createFred());
        assertThrows(CustomerAlreadyOnFileException.class, () -> dataBase.saveNewCustomer(createFred()));
    }

    private Map<String, String> createFred() {
        HashMap<String, String> customerRecord = new HashMap<>();
        customerRecord.put("name", "Fred");
        customerRecord.put("account_number", "123");
        return customerRecord;
    }

    private Map<String, String> createGeorge() {
        HashMap<String, String> customerRecord = new HashMap<>();
        customerRecord.put("name", "George");
        customerRecord.put("account_number", "567");
        return customerRecord;
    }
}
```
Added here are some tests for `isAccountOnFile()`, as well as for the two exceptions that might be thrown.


# Handling the Exception

The situation where you have a method that calls another method that, in turn, calls another method, and so on - and the method all the way at the bottom throws a checked exception happens a lot.  Your code needs to work backwards through the method chain, deciding what to do with exception in each method.  There are three options available in each method. Here they are, in order of preference:

1. Handle it
1. Transform it into a more meaningful exception and throw that.
1. Throw it to the calling method.

Obviously, if a method can effectively deal with an exception, then that's end of it and the method that calls it need not know that it happened.  If your method can't deal with it, then it's possible that your method can add better context to what has happened, in which case it should throw a new exception.  

Otherwise, just specify that the method can throw that exception type in it's signature and let it move on up to the previous method in the call chain.  

Let's look at the DAO and the Broker.  

There's nothing the DAO can do about this error, and there's no context that the DAO can meaningfully add to the error, either.  So that DAO should just let it propagate up to the calling method in the Broker.

The Broker can't do anything with it either.  Once again, there's no context that it can add.  So it just throws it again and it goes to the Interactor.

## Handling the Exception in the Interactor

The Interactor is the lowest class that can actually do something about this error, as deciding what should happen next is part of the business logic.  So, what should it do?

The main thing that needs to happen is that we notify the user that it couldn't be saved and tell them why.  This is something that the Interactor cannot do by itself.  We'll need to get the Controller involved.  We also need to transform this exception into some sort of data that we can communicate back to the Controller.  

We only have two possible results:  the save worked, or the saved failed because of a duplicate account.  This means we can get away with a simple Boolean result.  

Here's the new Interactor code:

``` java
public boolean saveCustomer() {
    String customerName = model.getCustomerName();
    String account = model.getAccountNumber();
    try {
        int recordId = broker.saveCustomer(createCustomerFromModel());
        System.out.println("Saving account: " + account + " Name: " + customerName + " Result: " + recordId);
        return true;
    } catch (CustomerAlreadyOnFileException e) {
        return false;
    }
}
```

We've changed the return value from `void` to `boolean`.  Then we return `true` unless we catch the `CustomerAlreadyOnFileException` where we return `false`.

# Communicating With the User

The Controller is going to be responsible for making sure that the user is told about this issue.  We're going to use an `Alert` to do this.

We need to transform our `Task` from one that returns `Void` to one the returns `Boolean`.  Then we can test for this in our `onSuccess` `EventHandler`.  `Task` always completes with `success` unless its call generates an uncaught exception.  Since we're catching the exception in the Interactor it will still complete successfully, even though the save itself failed.

Here's what our method looks like now:

``` java
private void saveCustomer(Runnable postTaskGuiActions) {
    Task<Boolean> saveTask = new Task<>() {
        @Override
        protected Boolean call() {
            return interactor.saveCustomer();

        }
    };
    saveTask.setOnSucceeded(evt -> {Alert
        postTaskGuiActions.run();
        if (!saveTask.getValue()) {
            Alert alert = new Alert(Alert.AlertType.ERROR);
            alert.setContentText("This customer is already on file, cannot save.");
            alert.show();
        }
    });
    Thread saveThread = new Thread(saveTask);
    saveThread.start();
}
```

And this is what it looks like:

![Screen shot]({{page.screenshot_1}})

We still need to run the `postTaskGuiActions`, but now we need to check the results of the `Task`'s `call()` method to see if the save was successful.  If it wasn't then we'll show an `Alert` with an appropriate message.
