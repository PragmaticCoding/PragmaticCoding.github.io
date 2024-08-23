---
title:  "Guide To the Observable Classes - Part IV"
date:   2024-08-03 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/observable-classes-custom
ScreenSnap1: /assets/elements/ListObservables1.png
ScreenSnap2: /assets/elements/ListObservables2.png
ScreenSnap3: /assets/elements/ListObservables3.png
ScreenSnap4: /assets/elements/ListObservables4.png
Diagram: /assets/elements/ListProperties.png
part1: /javafx/elements/observable-classes-generics
part2: /javafx/elements/observable-classes-typed

excerpt: A look at the Observable classes that wrap ObservableList
---

# Introduction

In [Part 1]({{page.part1}}) of this series, we looked at the `Observable` classes designed to wrap aribitrary `Object` values, and in [Part II]({{page.part2}}) we looked at the typed `Observable` classes.  In this article we are going to look at the `Observables` designed to hold `ObservableLists`.

This is a subject that always confused me.  Aren't `ObserableLists` already `Observable`?  And doesn't `ObseravbleList` already implement the `Obseravble` interface?  So why wrap an `ObservableList` inside an `Observable` wrapper?

The whole thing, at first glance, seems very circular.  We have an `Observable` that wraps an `ObservableList` and supports these same methods as `ObservableList`.  What's the point?

The truth is, you've probably already used a `Property` that wraps an `ObseravbleList` if you have ever used `TableView`.  That's because the `items` field in `TableView` is an `ObjectProperty<ObservableList<S>>`.  But it's highly unlikely that you've ever used this as a `Property`, meaning that you've probably never put a `Listener` on it or bound it to or from anything.

We should have a closer look at what this does.

# Why Do This?

I really, really like the idea of getting the "plumbing" out of your layout code.  As much as possible, layout code should be "do this, do that", without giving details about *how* to do this or that.  And when you have a `Property` that wraps an object that has `Observable` fields of its own, you can spend a lot of time writing code in your layout to implement `ChangeListeners` that bind and unbind fields as the object in a `Property` changes.  

If you have a really big project, maybe multiple applications in the same domain, that are using some standard elements as part of a Presentation Model, then it's possible that creating some custom `Properties` for them is a worthwhile exercise.  That way that you move the common plumbing that you'd otherwise be repeating over and over in your layout code (and violating DRY) into an encapsulated area inside your custom `Property`.

# An Example - A "Customer" Object

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
