---
title:  "Action Properties"
date:   2024-03-18 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/action-properties
Diagram: /assets/posts/MVCI.png
Modena: /assets/elements/modena.css
excerpt: There a little known, but very useful function call in most Properties.  This can be overridden to turn a Property into a class that transforms a value change into an action.
---

# Introduction

I'm not a big fan of the way that the JavaFX JavaDocs are written, especially the introductory sections.  My biggest complaint is that they are relatively opaque to a reader with little understanding of the specific topic.  Quite ofen, I'll go back once I have a fuller understanding of a concept and I'll see that the introductory discussion actually directly addressed whatever issue I was having, just not in a way that I had any hope of picking up on at the beginning.

This example starts with the code that was included in the introductory discussion for PseudoClass:

``` java
public boolean isMagic() {
       return magic.get();
   }

   public BooleanProperty magicProperty() {
       return magic;
   }

   public BooleanProperty magic =
       new BooleanPropertyBase(false) {

       @Override protected void invalidated() {
           pseudoClassStateChanged(MAGIC_PSEUDO_CLASS, get());
       }

       @Override public Object getBean() {
           return MyControl.this;
       }

       @Override public String getName() {
           return "magic";
       }
   }

   private static final PseudoClass
       MAGIC_PSEUDO_CLASS = PseudoClass.getPseudoClass("xyzzy");
```

Right off the bat, this is probably the most complicated way that you could implement a new PseudoClass, and written under the unspoken assumption that this code would be inside some custom `Node` class that implements the PseudoClass.  The only way that you can determine that last assumption is because the call to `pseudoClassStateChanged()` has an implied `this` and that `pseudoClassStateChanged()` is a member method of `Node`.  So `this` has to refer to `Node` which means that this anonymous inner class has to be defined inside a custom class that extends `Node`.

But, putting that aside, the really interesting thing here is this part:

``` java
@Override protected void invalidated() {
    pseudoClassStateChanged(MAGIC_PSEUDO_CLASS, get());
}
```

Woa!  What's this?  

Well, it's in the JavaDocs:

> The method invalidated() can be overridden to receive invalidation notifications. This is the preferred option in Objects defining the property, because it requires less memory. The default implementation is empty.

That doesn't help much, to be honest.  The memory stuff is confusing too.  

The other thing to notice is that this is in `BooleanPropertyBase`.  It's also in `IntegerPropertyBase`, `ObjectPropertyBase`, and, I assume, all the `...PropertyBase` classes.  But what are these classes?

To understand this, let's first look at `SimpleBooleanProperty`, the one we're all familiar with...

## SimpleBooleanProperty

Here's the source code for `SimpleBooleanProperty`:

``` java
public class SimpleBooleanProperty extends BooleanPropertyBase {
    private static final Object DEFAULT_BEAN = null;
    private static final String DEFAULT_NAME = "";
    private final Object bean;
    private final String name;

    public Object getBean() {
        return this.bean;
    }

    public String getName() {
        return this.name;
    }

    public SimpleBooleanProperty() {
        this(DEFAULT_BEAN, "");
    }

    public SimpleBooleanProperty(boolean var1) {
        this(DEFAULT_BEAN, "", var1);
    }

    public SimpleBooleanProperty(Object var1, String var2) {
        this.bean = var1;
        this.name = var2 == null ? "" : var2;
    }

    public SimpleBooleanProperty(Object var1, String var2, boolean var3) {
        super(var3);
        this.bean = var1;
        this.name = var2 == null ? "" : var2;
    }
}
```
Really what we have here is 4 different constructors, all designed to handle various cases of defining, or not defining, the Java Bean, name and initial value of the `Property`.  Other than that, it doesn't do much, and it delegates any initialization of the boolean value to its parent, `BooleanPropertyBase`.

If you look at the abstract `BooleanPropertyBase` class, you'll find that it does 100% of the actual `Property` stuff you need, but none of the Java Bean handling at all.  Those functions are left to whatever concrete class extends from it.  So `SimpleBooleanProperty` only introduces enough code to support the Java Bean functions `getName()` and `getBean()` required by the interface `ReadOnlyProperty` that is implemented by `BooleanProperty`.

I'll admit that the Java Bean stuff mystifies me a little bit.  I've always assumed that it was used for serialization and de-serialization, but I've yet to see a real case for serializing JavaFX properties.  I've certainly never seen any project where the Java Bean values were actually used for JavaFX `Properties`.  

But still, this is what `SimpleBooleanProperty` (and all the other "Simple...Property" classes) add to `BooleanProperty`.

And that means that there's nothing wonderful or mysterious or essential about the `Simple...Property` classes at all.  They just add some simple stuff to `...PropertyBase`.  There's no inherent advantage to extending `Simple...Property` classes instead than ditching them and extending directly from `...PropertyBase` - as the example from the PseudoClass JavaDocs does.

## Creating a Custom PseudoClassProperty

Let's look at creating a custom `Property` to handle `PseudoClass` changes that we can bind to other `Properties`.  We'll build it as an extension of `BooleanPropertyBase`.  It's going to be a `BooleanProperty` because that's what we need for PseudoClasses, they're either off or on, true or false.

We'll call our new class: `PseudoClassProperty`.

The only things that we are obligated to handle, because they are defined in the interfaces but not implemented in `BooleanPropertyBase` or any class it inherits from, are `getBean()` and `getName()`.  Remember, we're not obligated to use any structure or particular values under the hood, we just have to implement these two methods.  

It turns out that there are two pieces of information we'll need to supply when we instantiate our `PseudoClassProperty`: The `Node` to which it is associated, and the `PseudoClass` that it implements.  It's not a stretch to view the `Node` as a good candidate for the `Bean`, since it kind of "owns" the `Property`.  The `PseudoClass` also has a name, which would work as the `PseudoClassProperty` name as well.  Maybe, if the `Node` also has an id defined, we would want to combine it with the `PseudoClass` name to became the `Property` name.  

That being said, I don't actually see any practical use to calling either `getBean()` or `getName()` in the real world.  So this exercise is a little bit like "going through the motions", but at least we'll have return values that are sensible.  

This solves all of our constructor issues.  We only need one kind of constructor since we always need to supply the `Node` and the `PseudoClass`, and we can let Kotlin handle a default value for our initial value of the `Property`, making it optional.  In truth, any instance of `PseudoClassProperty` would almost inevitably be bound to some other `Property` right away, and the `initialValue` parameter could probably just be taken out of the constructor.  But with Kolin, it doesn't hurt to leave it in (Yeah! Kotlin!!!)

This gives us this:

``` kotlin
class PseudoClassProperty(private val node: Node, private val pseudoClass: PseudoClass,
                          initialValue: Boolean = false) : BooleanPropertyBase(initialValue) {
    override fun getBean() = node
    override fun getName(): String = pseudoClass.pseudoClassName

    override fun invalidated() {
        node.pseudoClassStateChanged(pseudoClass, value)
    }
}
```
This is geunuinely, generically useful.

There is virtually no way to implement a PseudoClass that's going to be linked to a `Property` that isn't going to execute `node.pseudoClassStateChanged(pseudoClass, value)` in some way.  It's total "boilerplate", and code that you want out of your layout code.

There's a little bit of added Kotlin trickery that can be used here to:

``` kotlin
infix fun Node.addPseudoClass(pseudoClass: PseudoClass) = PseudoClassProperty(this, pseudoClass)
```

What does this do?  It adds a function to virtually every JavaFX component that connects a `PseudoClassProperty` to it.  Now we can do this:

``` kotlin
private fun createSampleRegion() = HBox(10.0).apply {
   children += Label("Some Text").apply {
      addPseudoClass(somePseudoClass).bind(model.someBooleanProperty)
  }
}
```

# The Action Property

The whole point of `PseudoClassProperty` is that it connects something that doesn't know about `Observables` into something that does know about `Observables` through an "action".  In this case, it's the PseudoClass state change that now responds to a state change in an `ObservableBooleanValue`.  Generally speaking this is an "adaptor".  

It's what I'm calling an "Action Property".  

And it's possible because of `...PropertyBase.invalidated()`.  But what is that?

## ...PropertyBase.invalidated()

Let's look at the source code for `BooleanPropertyBase`.  We find this:

``` java
private void markInvalid() {
    if (this.valid) {
        this.valid = false;
        this.invalidated();
        this.fireValueChangedEvent();
    }

}

protected void invalidated() {
}

public void set(boolean var1) {
    if (!this.isBound()) {
        if (this.value != var1) {
            this.value = var1;
            this.markInvalid();
        }

    } else {
        String var10002 = this.getBean() != null && this.getName() != null ? this.getBean().getClass().getSimpleName() + "." + this.getName() + " : " : "";
        throw new RuntimeException(var10002 + "A bound value cannot be set.");
    }
}

```

First off, we see that, just as described in the JavaDocs, `invalidated()` is empty.  So we'll never need to call `super.invalidated()`.  

More importantly:  Since it's empty, that means that it's only reason for existance is to be overriden in extending classes!  Keep that in mind.

We can also see that `invalidated()` is called by `markInvalid()`.  Looking a little bit closer, we can see that `markInvalid()` is called by the public method `set()` when the value actually changes.  There are a few other places that `markInvalid()` is called from which have to do with handling when it is bound to another `Property`.  But you should be able to get the idea from `set()`.

Back in days long gone by, this is something that we would have called a "hook".  It's an empty method that is called in certain circumstances with the expectation that you can, if you want, supply some code to run there in an extending class.  Its only purpose for existance is to provide that facility.

## Advantages of invalidated() Over addListener(InvalidationListener)

Let's be fair.  There's nothing you can do with overriding `invalidated()` that you can't do with `addListener()`.  There are no other `protected` methods or fields all the way up the hierarchy that you could only access through `invalidated()`.  So no special status is awarded to code running in `invalidated()`.

The difference is boilerplate code in your layout.  

# Animations

``` kotlin
class FadeActionProperty(private val node: Node, private val duration: Duration) : BooleanPropertyBase() {
    override fun getBean() = node
    override fun getName() = "Fade Transition Property"

    private val transition = FadeTransition(duration, node)

    override fun invalidated() {
        if (value) node.isVisible = true
        transition.interpolator = Interpolator.EASE_IN
        transition.toValue = if (value) 1.0 else 0.0
        transition.setOnFinished { node.isVisible = value }
        transition.playFromStart()
    }
}

class ActionPropertyViewBuilder1() : Builder<Region> {
    private val nodeVisible: BooleanProperty = SimpleBooleanProperty(true)

    override fun build(): Region = VBox(10.0).apply {
        children += VBox(
            10.0,
            Label("Line 1"),
            Label("Line 2").apply {
                FadeActionProperty(this, Duration.millis(700.0)).bind(nodeVisible)
                managedProperty().bind(visibleProperty())
            },
            Label("Line 3"),
        )
        children += ToggleButton("Visible").apply {
            nodeVisible.bind(selectedProperty())
            isSelected = true
        }
        minWidth = 300.0
    } padWith 20.0
}

class ActionPropertyApplication1 : Application() {
    override fun start(stage: Stage) {
        with(stage) {
            scene = Scene(ActionPropertyViewBuilder1().build())
            show()
        }
    }
}

fun main() {
    Application.launch(ActionPropertyApplication1::class.java)
}
```
