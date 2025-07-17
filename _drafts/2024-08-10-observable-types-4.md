---
title:  "Guide To the Observable Classes - Part IV"
date:   2025-07-09 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/observable-classes-custom
ScreenSnap0: /assets/elements/CustomProperty0.png
ScreenSnap1: /assets/elements/CustomProperty1.png
ScreenSnap2: /assets/elements/CustomProperty2.png
ScreenSnap3: /assets/elements/CustomProperty3.png
ScreenSnap4: /assets/elements/CustomProperty4.png
ScreenSnap5: /assets/elements/CustomProperty5.png
ScreenSnap6: /assets/elements/CustomProperty6.png
ScreenSnap7: /assets/elements/CustomProperty7.png
ScreenSnap8: /assets/elements/CustomProperty8.png
ScreenSnap9: /assets/elements/CustomProperty9.png
Diagram: /assets/elements/ListProperties.png
part1: /javafx/elements/observable-classes-generics
part2: /javafx/elements/observable-classes-typed
part3: /javafx/elements/observable-classes-lists
JavaDocs : https://openjfx.io/javadoc/23/javafx.base/javafx/beans/value/ObservableListValue.html

excerpt: A look at how to create custom Propery classes.
---

# Introduction

In [Part 1]({{page.part1}}) of this series, we looked at the `Observable` classes designed to wrap aribitrary `Object` values, and in [Part II]({{page.part2}}) we looked at the typed `Observable` classes.  In [Part III]({{page.part3}}) we took a deep dive into the `Observables` designed to hold `Lists`, and the `Properties` designed to hold `ObservableLists`.

This last item always confused me.  Aren't `ObserableLists` already `Observable`?  And doesn't `ObseravbleList` already implement the `Obseravble` interface?  So why wrap an `ObservableList` inside an `Observable` wrapper?

The whole thing, at first glance, seems very circular.  We have an `Observable` that wraps an `ObservableList` and supports these same methods as `ObservableList`.  What's the point?

The "point" it turns out is very cool.  You end up with a `Property` that behaves like a `Property` but also behaves like the `ObservableList` it contains.  Now, if you generalize this:  

You have a `Property` that behaves like a `Property` but also behaves exactly like the object it contains.

# Is this a Big Deal?

If you think about it, most of the other "typed" `Properties` do this to some degree.  `IntegerProperty` has methods for adding and subtracting, and `StringProperty` can do things like concatenation. But not quite.  For instance, you can't do this:

``` kotlin
val someIntegerProperty = firstIntegerProperty + secondIntegerProperty
```
but you can do the `Binding`:

``` kotlin
val someIntegerObservable = firstIntegerProperty.add(secondIntegerProperty)
```

The only `Observable` class that actually works like it's backing type is `ObseravbleList`.  It wraps a `List` of some sort, and any operation that you can do to a `List`, you can do to its enclosing `ObservableList`.  

I think it's undeniable that being able to `add()`, `setAll()` or `remove()` directly from an `ObservableList`, without having to reach in to the backing `List` object is worthwhile.

# Why Do This?

I really, really like the idea of getting the "plumbing" out of your layout code.  As much as possible, layout code should be "do this, do that", without giving details about *how* to do this or that.  And when you have a `Property` that wraps an object that has `Observable` fields of its own, you can spend a lot of time writing code in your layout to implement `ChangeListeners` that bind and unbind fields as the object in a `Property` changes.  

If you have a really big project, maybe multiple applications in the same domain, that are using some standard elements as part of a Presentation Model, then it's possible that creating some custom `Properties` for them is a worthwhile exercise.  That way that you move the common plumbing that you'd otherwise be repeating over and over in your layout code (and violating DRY) into an encapsulated area inside your custom `Property`.

What we are going to look at first, is the idea wrapping a POJO inside a `Property` that then acts just like that POJO that it wraps.

# Wrapping a Plain Java Object

This first case that we will look at is analagous to `ObservableList` in that it will just wrap an object that doesn't have any observable characteristics of its own.  

Just like `ObservableList`, that wraps an `Interface` we'll follow the same pattern.  To illustrate this, we'll take a fairly common example that a lot of business applications have: a "customer" object:

``` kotlin
interface Customer {
    fun getAccount(): String?
    fun setAccount(p0: String)

    fun getFName(): String?
    fun setFName(p0: String)

    fun getLName(): String?
    fun setLName(p0: String)

    fun getAge(): Int
    fun setAge(p0: Int)

    fun fullName(): String
}
```
Here's our POJO implementation, `CustomerImpl`:

``` kotlin
class CustomerImpl : Customer {

    private var account: String? = null
    private var fName: String? = null
    private var lName: String? = null
    private var age: Int = 0

    override fun getAccount(): String? = account

    override fun setAccount(p0: String) {
        account = p0
    }

    override fun getFName(): String? = fName

    override fun setFName(p0: String) {
        fName = p0
    }

    override fun getLName(): String? = lName

    override fun setLName(p0: String) {
        lName = p0
    }

    override fun getAge(): Int = age

    override fun setAge(p0: Int) {
        age = p0
    }

    override fun fullName(): String = "$fName $lName"
}
```
Now we can think about creating a `CustomerProperty` that both contains a `Customer` as an `Observable` and implements `Customer` itself.  We'll follow the same pattern that `SimpleObjectProperty` uses by implementing `getBean()` and `getName()` and supplying the constructors to initialize them:

``` kotlin
class CustomerProperty(
    private val bean: Any?,
    private val name: String,
    initialValue: Customer? = null
) : ObjectPropertyBase<Customer>(), Customer {

    override fun getName() = name
    override fun getBean() = bean

    constructor() : this(null, "", null)
    constructor(initialValue: Customer) : this(null, "", initialValue)

    val fred: ListProperty<String> = SimpleListProperty(FXCollections.observableArrayList())

    init {
        initialValue?.let { super.setValue(it) }
    }

    override fun getAccount(): String? = value?.getAccount()

    override fun setAccount(p0: String) {
        value?.setAccount(p0)
    }

    override fun getFName(): String? = value?.getFName()

    override fun setFName(p0: String) {
        value?.setFName(p0)
    }

    override fun getLName(): String? = value?.getLName()

    override fun setLName(p0: String) {
        value?.setLName(p0)
    }

    override fun getAge(): Int = value?.getAge() ?: 0

    override fun setAge(p0: Int) {
        value?.setAge(p0)
    }

    override fun fullName(): String = value?.let { "${it.getFName()} ${it.getLName()}" } ?: ""
}
```
All the rest of the `Property` stuff inherits from `ObjectPropertyBase` and we don't need to worry about it.  We provide implementation methods for all of the getters and setters from `CustomerInterface` that simply delegate to the enclosed `Customer` value.  We deal with a `null` in the `value` field by just returning `null` for the getters that support it, and a "0" in the age getter.  For the setters, we only attempt to delegate if the `value` in the `Property` is not `null`.  In real life, you'd probably want to handle that better.

Now we have a `Property` that wraps a `CustomerInterface` value and implements `CustomerInterface` itself, delegating to the enclosed value.  

We can use it like this:

``` kotlin
class CustomPropertyExample0() : Application() {
    val customer = CustomerProperty(CustomerImpl().apply {
        setFName("Fred")
        setLName("Brown")
        setAge(3)
    })

    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(customer))
        stage.show()
    }

    private fun createContent(customerProperty: CustomerProperty): Region = BorderPane().apply {
        center = VBox(10.0).apply {
            children += Label().apply {
                textProperty().bind(customerProperty.map { "First Name: ${it.getFName()}" })
            }
            children += Label().apply {
                textProperty().bind(customerProperty.map { "Last Name: ${it.getLName()}" })
            }
            children += Label().apply {
                textProperty().bind(customerProperty.map { "Age: ${it.getAge()}" })
            }
            children += Button("Increment Age").apply {
                setOnAction { evt -> customerProperty.setAge(customerProperty.getAge() + 1) }
            }
        }
        padWith(20.0)
    }
}

fun main() {
    Application.launch(CustomPropertyExample0::class.java)
}
```
Here we use `CustomerProperty` in two ways.  In the `Button` action, we use it just like an implementation of `Customer` to increment the age.  In the "Age" `Label` we treat it like a `Property` and use `Property.map{}` to bind it to the `Label`.  However, inside the `map{}` we treat it like `Customer` again.

This is what it looks like:

![Screen Snap]({{page.ScreenSnap0}})

But there is a problem!  If you click on the `Button`, nothing happens.

That's because we haven't changed the value in the `Property`, we changed the value of one of the fields *inside* the `Property`.  One small change will fix this, in `CustomerProperty.setAge()`:

``` kotlin
override fun setAge(p0: Int) {
    value?.setAge(p0)
    fireValueChangedEvent()
}
```
That one call to `fireValueChangedEvent()` will cause the `Property` to invalidate.  Then the `Label`, which is bound to the `Property` will re-evaluate and call the `Property.map{}` again.  Now it looks like this after the `Button` has been clicked a few times:

![Screen Snap]({{page.ScreenSnap1}})

This is pretty cool.  We've effectively changed a `POJO` object into something internally observable by forcing it to invalidate via the setters implemented in the `Property` implementation.  

## What About the Backing Object?

One caveat with the approach is that you won't trigger invalidation if you directly invoke the setters of the backing object.  Let's make a tiny change to how the `Button` works:

``` kotlin
children += Button("Increment Age").apply {
    setOnAction { evt -> customerProperty.value.setAge(customerProperty.getAge() + 1) }
}
```
Instead of treating `customerProperty` like `Customer` and calling `Customer.setAge()`, we are calling `customerProperty.value.setAge()`.  This calls `CustomerImpl.setAge()`.

The same thing would happen if you had a reference to that `CustomerImpl` and called one of its setters directly.  You won't see that change until your `CustomerProperty` invalidates.  

This isn't really anything new.  You wouldn't expect this...

``` kotlin
var someInteger = 5
val someIntProperty = SimpleIntegerProperty(someInteger)
someInteger = 20
```
...to change the value in `someIntProperty` to 20.  It just gets a bit more complicated when you are thinking about composed objects as `Property` values.

This is also the way that `ObservableList` works.  If you hang on to a reference to the backing `List` and update it directly, then you won't trigger `Listeners` on the `ObservableList`.

But you can change the value in the `Property` itself.  Let's try this:

``` kotlin
class CustomPropertyExample1() : Application() {
    val customer1 = CustomerImpl().apply {
        setFName("Fred")
        setLName("Brown")
        setAge(3)
    }
    val customer2 = CustomerImpl().apply {
        setFName("George")
        setLName("Smith")
        setAge(15)
    }
    val customer = CustomerProperty(customer1)

    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(customer))
        stage.show()
    }

    private fun createContent(customerProperty: CustomerProperty): Region = BorderPane().apply {
        center = VBox(10.0).apply {
            children += Label().apply {
                textProperty().bind(customerProperty.map { "First Name: ${it.getFName()}" })
            }
            children += Label().apply {
                textProperty().bind(customerProperty.map { "Last Name: ${it.getLName()}" })
            }
            children += Label().apply {
                textProperty().bind(customerProperty.map { "Age: ${it.getAge()}" })
            }
            children += Button("Increment Age").apply {
                setOnAction { evt -> customerProperty.setAge(customerProperty.getAge() + 1) }
            }
            children += Button("Customer 2").apply {
                setOnAction { evt -> customerProperty.value = customer2 }
            }
            children += Button("Customer 1").apply {
                setOnAction { evt -> customerProperty.value = customer1 }
            }
        }
        padWith(20.0)
    }
}

fun main() {
    Application.launch(CustomPropertyExample1::class.java)
}
```
We now have two `CustomerImpl`, "Fred" and "George" with different ages.  Two more `Buttons` have been added, each of which swaps the value in our `CustomerProperty` to one of either "Fred" or "George".  Now it look like this when started up:

![Screen Snap]({{page.ScreenSnap2}})

When we click the "Increment Age" `Button` a few times:

![Screen Snap]({{page.ScreenSnap3}})

And then click "Customer 2":

![Screen Snap]({{page.ScreenSnap4}})

Then some more "Increment Age":

![Screen Snap]({{page.ScreenSnap5}})

Let's take one more look at what you can do:

``` kotlin
class CustomPropertyExample3() : Application() {
    val customer1 = CustomerImpl().apply {
        setFName("Fred")
        setLName("Brown")
        setAge(3)
    }
    val customer2 = CustomerImpl().apply {
        setFName("George")
        setLName("Smith")
        setAge(15)
    }
    val customer = CustomerProperty(customer1)
    val custList: ObservableList<Customer> = FXCollections.observableArrayList(listOf(customer1, customer2))

    override fun start(stage: Stage) {
        stage.scene = Scene(createContent(customer))
        stage.show()
    }

    private fun createContent(customerProperty: CustomerProperty): Region = BorderPane().apply {
        center = VBox(10.0).apply {
            children += Label().apply {
                textProperty().bind(customerProperty.map { "First Name: ${it.getFName()}" })
            }
            children += Label().apply {
                textProperty().bind(customerProperty.map { "Last Name: ${it.getLName()}" })
            }
            children += Label().apply {
                textProperty().bind(customerProperty.map { "Age: ${it.getAge()}" })
            }
            children += Button("Increment Age").apply {
                setOnAction { evt -> customerProperty.setAge(customerProperty.getAge() + 1) }
            }
        }
        right = ListView<Customer>().apply {
            items = custList
            customer.bind(this.selectionModel.selectedItemProperty())
        }

        padWith(20.0)
    }
}

fun main() {
    Application.launch(CustomPropertyExample3::class.java)
}
```
Here, I've added an `ObservableList<Customer>` and populated it with `customer1` and `customer2`.  Then I've added a `ListView` to the layout and set its `items` property to that `ObservableList<Customer>`.  Finally, I've bound our `CustomerProperty` to the `selectedItemProperty` of the `ListView`.  

Let's see what it looks like:

![Screen Snap]({{page.ScreenSnap6}})

Then, after clicking on one of the items in the `ListView`

![Screen Snap]({{page.ScreenSnap7}})

And then clicking on the `Button` a few times:

![Screen Snap]({{page.ScreenSnap8}})

One thing to keep in mind is that even though the `ListView` returns a `ReadOnlyObjectProperty` as its `selectedItemProperty`, that doesn't make the `value` held in the `CustomerProperty` immutable.  As you can see in the last screen snap, the age of Fred has been incremented a few times to 8.

Also note that the `selectedItemProperty` is `ReadOnlyObjectProperty<Customer>` and `CustomerProperty` extends `ObjectPropertyBase<Customer>` which means that the binding is type compatible.  However, you could not call `ListView.selectionModel.selectedItemProperty().setAge(20)` because `ReadOnlyObjectProperty<Customer>` doesn't implement `Customer`.  It just wraps `Customer` as a value in an `Observable`.


{% include notice_question type="primary" content = "What if we wanted to implement Customer in an ObservableValue instead of a Property?<br><br>" %}

# Creating a Hierachy

We should go back and look at the hierarchy tree for `ObservableList` again, because it can help:

![Diagram]({{page.Diagram}})

For us, the most important element is `ObservableListValue`.  This is were we integrate `ObservableList` to `ObservableValue<ObservableList>`.  If we look at this in the [JavaDocs]({{page.JavaDocs}}), we find this:

>All Superinterfaces:
>
>    Collection<E>, Iterable<E>, List<E>, Observable, ObservableList<E>, ObservableObjectValue<ObservableList<E>>, ObservableValue<ObservableList<E>>, SequencedCollection<E>

Look at that third item.  `List`!  And the fifth item.  `ObservableList`.

But this is just an `Interface`.  It doesn't implement any of the methods from these other classes, just forces some subclass to do so.  

It turns out that the class that does much of the implementation is `ListExpression`.  If we look at the source for `ListExpression.add()`, this is what we see:

``` java
public boolean add(E element) {
    return getNonNull().add(element);
}
   .
   .
   .
private ObservableList<E> getNonNull() {
   ObservableList<E> list = get();
   return list == null ? FXCollections.emptyObservableList() : list;
}
```
This is just a delegate to the backing `List`, with a provision to avoid having to deal a `null` backing `List`.

Clearly this is where we need to insert our top level classes:  `ObservableCustomerValue` and `CustomerExpression`.  So let's go to it:

``` kotlin


```



# More


To illustrate this, we'll take a fairly common example that a lot of business applications have: a "customer" object.  We're interested here, not in the domain object, but the Presentation Model, meaning an object that is composed of `Observable` type fields.

## CustomerModel and a Basic Implementation

Since we have to deal with inheritance here, we're going to start out with a interface, and we'll call it `CustomerModel`:

``` kotlin
interface CustomerModel {
    fun accountProperty(): StringProperty
    fun getAccount(): String?
    fun setAccount(p0: String)

    fun fNameProperty(): StringProperty
    fun getFName(): String?
    fun setFName(p0: String)

    fun lNameProperty(): StringProperty
    fun getLName(): String?
    fun setLName(p0: String)

    fun getAge(): Int
    fun setAge(p0: Int)

    fun fullName(): ObservableStringValue
}
```
This is obviously just a minimal example.  We have an "account" `Property` because we can use that for equality, and separate first and last name `Properties` so that we can do a little `String` manipulation.  Finally, we have an "age" field, which isn't a property at all.  This last one is just to give an example of working with something that isn't an `Observable` field.  In the interface, though, it just looks like we're missing the `ageProperty()` method.  

Since we're actually going to use this pretty early on, here's a basic implementation of our `CustomerModel`

``` kotlin
class CustomerImpl : CustomerModel {
    private val fNameProp: StringProperty = SimpleStringProperty("")
    private val lNameProp: StringProperty = SimpleStringProperty("")
    private val accountProp: StringProperty = SimpleStringProperty("")
    private var age: Int = 0

    override fun accountProperty(): StringProperty = accountProp
    override fun getAccount(): String = accountProp.get()
    override fun setAccount(p0: String) {
        accountProp.set(p0)
    }

    override fun fNameProperty(): StringProperty = fNameProp
    override fun getFName(): String = fNameProp.get()
    override fun setFName(p0: String) {
        fNameProp.set(p0)
    }

    override fun lNameProperty(): StringProperty = lNameProp
    override fun getLName(): String = lNameProp.get()
    override fun setLName(p0: String) {
        lNameProp.set(p0)
    }

    override fun getAge() = age
    override fun setAge(p0: Int) {
        age = p0
    }

    override fun fullName(): ObservableStringValue = lNameProp.concat(", ").concat(fNameProp)
}
```

## Building the Property Hierachy

We'll start out by looking at the classes and interfaces needed to get us down to creating a custom `Property` that can wrap our `CustomerModel`.  Then we'll keep going and look at how to create the `Binding` elements.

### ObservableCustomerValue

Just a note on the naming:  I felt it would get tedious adding "Model" into every name and it wouldn't actually add any clarity to anything.  So instead of calling this `ObservableCustomerModelValue`, I went with `ObservableCustomerValue`.  We know that, since it's an `Observable` that we're dealing with Presentation Model stuff here, and there's little possibility of confusion with Domain Objects.  

That said, `ObservableCustomerValue` is not very exciting in itself:

``` kotlin
interface ObservableCustomerValue : ObservableObjectValue<CustomerModel>, CustomerModel
```
On the other hand, this **is** critical.  This is the place where we say that all of the rest of the `Observables` in our hierarchy are going to implement not just `ObservableObjectValue<CustomerModel>`, but they are going to implement `CustomerModel` itself!  This means that we can treat any of these `Observables` just like the object that they are wrapping.  

### CustomerExpression

As you will remember from the other articles in this series, the top level "Expression" class is the root from which all of the `Properties` inherit.  Since it inherits from `ObservableCustomerValue`, it's a good place to define not just the "Expression" methods themselves, but also all of the methods defined in `CustomerModel`.  In fact, at this stage, this is all that we're going to define:

``` kotlin
abstract class CustomerExpression : ObservableCustomerValue {
    protected val delegate = CustomerImpl()

    override fun accountProperty() = delegate.accountProperty()
    override fun getAccount(): String? = delegate.getAccount()
    override fun setAccount(p0: String) {
        delegate.setAccount(p0)
    }

    override fun fNameProperty(): StringProperty = delegate.fNameProperty()
    override fun getFName(): String? = delegate.getFName()
    override fun setFName(p0: String) {
        delegate.setFName(p0)
    }

    override fun lNameProperty(): StringProperty = delegate.lNameProperty()
    override fun getLName(): String? = delegate.getLName()
    override fun setLName(p0: String) {
        delegate.setLName(p0)
    }

    override fun getAge(): Int = delegate.getAge()
    override fun setAge(p0: Int) {
        delegate.setAge(p0)
    }

    override fun fullName(): ObservableStringValue = delegate.fullName()
}
```
Originally, I was going to try to implement a lot of these methods using exactly the same techniques used internally in the JavaFX code.  However, it makes extensive use of an object called `ExpressionHelper` which, while public, is not exposed to outside modules.  I got this compiler error when attempting to implement some of the `Listener` code when I got to `CustomerPropertyBase`:

> Symbol is declared in module 'javafx. base' which does not export package 'com. sun. javafx. binding'

It may be possible to get around this, but my experience is that when you get these kinds of errors you're most likely barking up the wrong tree.  So I took a different approach.  We'll see that in detail in a minute, but the approach involves encapsulation and delegation - as you can see in the code for `CustomerExpression`.

At this point, we have a field called `delegate` which is instantiated as our basic `CustomerImpl`.  Then we're just delegating all of our `CustomerModel` methods to that field.  This allows us to avoid recreating all of the functionality that's already inside `CustomerImpl`.  

### Basic Property Classes

We need to construct the hierarchy down to `SimpleCustomerProperty`, and there are two classes and one interface we need to define.  None of them have any methods or implementations of their own, but they do connect everything together via inheritance:

``` kotlin
abstract class ReadOnlyCustomerProperty : CustomerExpression(), ReadOnlyProperty<CustomerModel>

interface WritableCustomerValue : WritableObjectValue<CustomerModel>, CustomerModel

abstract class CustomerProperty : ReadOnlyCustomerProperty(), Property<CustomerModel>, WritableCustomerValue
```

### CustomerPropertyBase

This is the place where I actually ran into issues with `ExpressionHelper`, and you can see the delegation approach come into play again:

``` kotlin
abstract class CustomerPropertyBase : CustomerProperty() {
    private var objectProperty = SimpleObjectProperty<CustomerModel>()
    private val changeListeners: MutableList<ChangeListener<in CustomerModel>?> = mutableListOf()
    private val invalidationListeners: MutableList<InvalidationListener?> = mutableListOf()
    private var fNameSubscription: Subscription? = null
    private var lNameSubscription: Subscription? = null
    private var accountSubscription: Subscription? = null
    private var inUpdate: Boolean = false

    init {
        delegate.accountProperty()
            .subscribe { newVal -> objectProperty.value?.let { updateProperty(it.accountProperty(), newVal) } }
        delegate.fNameProperty()
            .subscribe { newVal -> objectProperty.value?.let { updateProperty(it.fNameProperty(), newVal) } }
        delegate.lNameProperty()
            .subscribe { newVal -> objectProperty.value?.let { updateProperty(it.lNameProperty(), newVal) } }
        objectProperty.subscribe { newObject ->
            accountSubscription?.unsubscribe()
            fNameSubscription?.unsubscribe()
            lNameSubscription?.unsubscribe()
            newObject?.let {
                accountSubscription = it.accountProperty()
                    .subscribe { newVal -> updateProperty(delegate.accountProperty(), newVal) }
                fNameSubscription = it.fNameProperty()
                    .subscribe { newVal -> updateProperty(delegate.fNameProperty(), newVal) }
                lNameSubscription = it.lNameProperty()
                    .subscribe { newVal -> updateProperty(delegate.lNameProperty(), newVal) }
            }
        }
    }

    private fun <T : Any> updateProperty(property: Property<T>, newVal: T) {
        if (!inUpdate) {
            inUpdate = true
            if (!property.isBound) property.value = newVal
            inUpdate = false
        }
    }

    override fun addListener(p0: ChangeListener<in CustomerModel>?) {
        objectProperty.addListener(p0)
        changeListeners += p0
    }

    override fun addListener(p0: InvalidationListener?) {
        objectProperty.addListener(p0)
        invalidationListeners += p0
    }

    override fun removeListener(p0: ChangeListener<in CustomerModel>?) {
        objectProperty.removeListener(p0)
        changeListeners -= p0
    }

    override fun removeListener(p0: InvalidationListener?) {
        objectProperty.removeListener(p0)
        invalidationListeners -= p0
    }

    override fun getValue(): CustomerModel = objectProperty.get()

    override fun get(): CustomerModel = objectProperty.get()

    override fun setValue(p0: CustomerModel?) {
        objectProperty.value = p0
    }

    override fun bind(p0: ObservableValue<out CustomerModel>?) {
        objectProperty.bind(p0)
    }

    override fun unbind() {
        objectProperty.unbind()
    }

    override fun isBound(): Boolean = objectProperty.isBound

    override fun bindBidirectional(p0: Property<CustomerModel>?) {
        objectProperty.bindBidirectional(p0)
    }

    override fun unbindBidirectional(p0: Property<CustomerModel>?) {
        objectProperty.unbindBidirectional(p0)
    }

    override fun set(p0: CustomerModel?) {
        objectProperty.set(p0)
    }
}
```

The net result of this design is that we have two delegation patterns in play...

The first is that we realize that `CustomerProperty` encapsulates `Property<CustomerModel>`, so we can can create a field which implements that and then just delegate all of the `Property` methods to that field.  So we have a field called `objectProperty` and it now handles all of the `Listeners` and `Binding` operations of the `Property` itself.

The second delegation pattern is the one to handle the `CustomerModel` methods.  We've seen that we have our `delegate` field, but now we need to keep it connected to our encapsulated `Property<CustomerModel>`.  This is where it gets a little messy.

What I was hoping to do was to bi-directionally bind the `Properties` in `delegate` to the `Properties` held in the `objectProperty` field.  This would be much in the same vein as `IntegerProperty.asObject()` does.  However, `IntegerProperty.asObject()` also uses magical binding stuff that isn't available to us.  This becomes important when `Properties` in either the `delegate` or `objectProperty` have been bound to other, external `Properties`.  

So bidirectional binding was out.

But we can do it manually.  

In reality, this is very much the same way that bi-directional binding works under the hood, although this uses `Subscriptions`, which has an advantage.  From the `delegate` end - which would be changes coming directly to our `CustomerProperty` we need a `Subscription` on each `Property` in `CustomerModel`.  We can copy any changes that happen to any of the corresponding `Properties` in `objectProperty` as long as those corresponding `Properties` have not been bound to something else.  

Since those `Properties` in `objectProperty` have `Subscriptions` of their own, we want to avoid triggering them when we're in the process of doing internal updates.  So I've added a flag called `inUpdate` which gets turned on and then off before and after making the updates.  The update changes are in their own method to avoid a lot of code repetitions.  

When the `CustomerModel` inside `objectProperty` is changed, then all of the `Subscriptions` on the various `Properties` inside of it need to be cancelled, and new `Subscriptions` put on the `Properties` inside the new value.  Here's where `Subscription` comes in handy, because `ObservableValue.subscribe()` returns a reference to the `Subscription` that it creates.  So we can put those references inside fields, and then call `Subscription.unsubscribe()` on them.  

And, of course, all of this stuff needs to be done with `Null` safety, because `objectProperty` could be empty at any time.

### SimpleCustomerProperty

There's only one thing left to complete the hierarchy, `SimpleCustomerProperty`:

``` kotlin
class SimpleCustomerProperty(initialValue: CustomerModel? = null) : CustomerPropertyBase() {

    init {
        set(initialValue)
    }

    override fun getBean() = "Bean!"
    override fun getName() = "A Customer"
}
```
We're ignoring the Java Bean stuff as much as possible, and providing only one constructor which optionally allows an initial value to be passed.  

## Basic Testing

Now we have something that will actually compile without any red squigglies for missing methods.  Let's see if it actually works!

``` kotlin
class TestApplication1 : Application() {
    override fun start(stage: Stage) {
        with(stage) {
            scene = Scene(TestScreenBuilder().build(), 400.0, 400.0)
            show()
            centerOnScreen()
        }
    }
}

class TestScreenBuilder : Builder<Region> {
    override fun build(): Region = VBox(10.0).apply {
        val customer1 = CustomerImpl().apply {
            setFName("Fred")
            setLName("Smith")
        }
        val customer2 = CustomerImpl().apply {
            setFName("George")
            setLName("Jones")
        }
        val customerProperty: CustomerProperty = SimpleCustomerProperty()
        customerProperty.set(customer1)
        children += Label().apply {
            textProperty().bind(customerProperty.fNameProperty().map { "Current Customer FName: $it" })
        }
        children += Label().apply {
            textProperty().bind(customerProperty.fullName().map { "Current Customer Full Name: $it" })
        }
        children += HBox(10.0).apply {
            children += Label("Customer1: ")
            children += Label().apply {
                textProperty().bind(customer1.fNameProperty())
            }
        }
        children += HBox(10.0).apply {
            children += Label("Customer2: ")
            children += Label().apply {
                textProperty().bind(customer2.fNameProperty())
            }
        }
        var swapper = true
        children += Button("Swap Customer").apply {
            setOnAction {
                customerProperty.set(if (swapper) customer2 else customer1)
                swapper = swapper.not()
            }
        }
        val nameList = listOf("Albert", "Vinny", "Bob", "Aloysius")
        var counter = 0
        children += Button("Change Name").apply {
            setOnAction { customerProperty.setFName(nameList[counter++]) }
        }
        padding = Insets(30.0)
    }

}


fun main() {
    Application.launch(TestApplication1::class.java)
}
```
We have two `CustomerImpl`, one that starts with "George Jones" and the other that starts with "Fred Smith".  Then we have a single `CustomerProperty` that we'll start off with holding `customer1`.  There are some `Labels` that show us the first names of `customer1` and `customer2` and the current occupant of our `CustomerProperty`.  We also have a `Label` that shows the `fullName()` from our `CustomerProperty`.

Then we have two `Buttons`.  The first one swaps the contents of our `CustomerProperty` between `customer1` and `customer2`.  The second changes the `fName` of `CustomerModel` currently in our `CustomerProperty`.  

## Binding Classes
