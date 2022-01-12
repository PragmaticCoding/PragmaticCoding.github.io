---
title:  "JavaFX: Stop Using PropertyValueFactory"
date:   2021-06-24 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
excerpt: PropertyValueFactory is an obsolete convenience method designed to eliminate boilerplate.  We don't don't need it any more now that we have Lambda expressions.
---

PropertyValueFactory is an obsolete convenience method designed to eliminate boilerplate.  We don't don't need it any more now that we have Lambda expressions.

In JavaFX, the TableView class is largely customized by adding TableColumn's to an instance.  TableColumn is where you define both the TableCell's that will be used, and manner in which data will be extracted from each row model to populate each the TableCell.

### The Full Boilerplate Approach

The JavaDoc's for TableColumn document the full boilerplate necessary to define how the data will be populated into the cells:

{% highlight java %}
ObservableList<Person> data = ...
TableView<Person> tableView = new TableView<Person>(data);

TableColumn<Person,String> firstNameCol = new TableColumn<Person,String>("First Name");
firstNameCol.setCellValueFactory(new Callback<CellDataFeatures<Person, String>, ObservableValue<String>>() {
     public ObservableValue<String> call(CellDataFeatures<Person, String> p) {
         return p.getValue().firstNameProperty();
     }
});
{% endhighlight %}

Yikes!  That's a lot of "<>" to wrap your brain around.

What does this do?  

Well first, a `Callback` is very similiar to a `Function`.  It takes a value of some specific type and returns a value of some other, specific type.  While a `Function` uses the `apply()` method, a `Callback` uses a `call()` method.

In this case, we're going to pass a `TableColumn.CellDataFeatures` object to the `call()` method, and it will return an `ObservableValue` back.  Both of these classes are generics, so we need specify the class of the data model for the table, and the class of the TableColumn data.  In the example above, the table model is `Person` and the `Observable` returned will contain a `String`.

It's not really important to know, but `TableColumn.CellDataFeatures` is a wrapper class which holds a reference to the `TableView`, the `TableColumn`, and has a function to return an instance of the TableView data model (in other words, the data for the row).  Remember that this `Callback` is invoked from deep inside the `TableView` code, and the purpose of this `Callback` is to provide a "hook" to allow that code to extract the data that it needs from the TableView data model to populate the cell.

Technically, it might be possible that you'd need some properties of the `TableView` or the `TableColumn` to determine how to extract the data from the data model.  Generally speaking, though, you just need the data model to extract the data.

### The Original Convenience Method

Even when you do understand what that boilerplate code is doing, it's still a lot of code to write for something that almost always boils down to one line very similar to this:

{% highlight java %}
return p.getValue().firstNameProperty();
{% endhighlight %}

Because that's what most CellValueFactories do, they extract a single `ObservableValue` from an object composed of a number of `ObservableValue`'s.  Furthermore, the best practice is to implement the fields in the model as JavaFX Property Beans, like this:

{% highlight java %}
public class Model {
    private StringProperty firstName = new SimpleStringProperty("");

    public String getFirstName() {
        return firstName.get();
    }

    public void setFirstName(String firstName) {
        this.firstName.set(firstName);
    }

    public StringProperty firstNameProperty() {
        return firstName;
    }
}
{% endhighlight %}

For each field in the Model, there's a `getter` and a `setter` which delegate to the field's `get()` and `set()` methods.  Then there's a method called `{Field Name}Property()`, which returns the property itself.  

This last piece is the most important for the TableView because, **if this pattern is followed**, you can use reflection to access the property through this method.

Returning to the JavaDoc's for `TableColumn`, we see this:

> It is hoped that over time there will be convenience cell value factories developed and made available to developers. As of the JavaFX 2.0 release, there is one such convenience class: `PropertyValueFactory`. This class removes the need to write the code above, instead relying on reflection to look up a given property from a String.

Curiously, this hasn't been updated, even as of JavaFX 15 - which is a shame.

This meant that you could replace the boilerplate described above with this:

{% highlight java %}
firstNameCol.setCellValueFactory(new PropertyValueFactory("firstName");
{% endhighlight %}

Which is clearly a lot easier to both write and read than the full boilerplate.  In fact, most programmers learn this method without ever understanding the `Callback` code that lies behind it.

### The Problem with "PropertyValueFactory"

The problem is that "firstName" is just a `String`.  There's no way that your IDE or a compiler is going to detect that "firstName" means that there needs to be a method called `firstNameProperty()` in your model.  So if you misspell "firstName", perhaps as "FirstName", your code will fail at runtime and you'll have to chase it down.

Even worse, should you decide to refactor your model and change the name of the field and the methods associated with it, your IDE won't be able to track down the reference in the `String` and update it as well.  Which, of course, won't generate any compiler errors but will mysteriously cause your `TableView` to stop working.

### Lambda to the Rescue

Since Java 8, there is a better way to reduce the boilerplate which avoids the use of reflection and makes `PropertyValueFactory` obsolete.

Remember that the `Callback` interface is a Functional Interface.  Which means that it can be the target of a lambda expression.  Also, lambda expressions can infer the class of the input parameters based on the context in which it is declared.  So the boilerplate listed above can be replaced with this:

{% highlight java %}
ObservableList<Person> data = ...
TableView<Person> tableView = new TableView<Person>(data);

TableColumn<Person,String> firstNameCol = new TableColumn<Person,String>("First Name");
firstNameCol.setCellValueFactory(p -> p.getValue().firstNameProperty());
{% endhighlight %}

In this example `p` is the `CellDataFeatures<Person, String>` object generated from deep inside the `TableView` code, and the return value is an `ObservableValue<String>`.  But there's no need to declare these types in the lambda since they are all inferred from the context.  

Clearly, this is about the same amount of code as using `PropertyValueFactory`, but it avoids the use of reflection.  If you misspell something, you will get a compiler error and a red squiggly line in your IDE.  If you refactor `firstNameProperty()` to a new name, your IDE will adjust it automatically and the compiler will complain if something goes wrong.

So that's it.  

Stop using `PropertyValueFactory` right now.  It does nothing for you, and can cause you trouble in the future.  Use lambda expressions instead.
