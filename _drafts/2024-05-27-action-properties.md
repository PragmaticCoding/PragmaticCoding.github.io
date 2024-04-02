---
title:  "Dealing With Modena"
date:   2024-03-18 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/action-properties
Diagram: /assets/posts/MVCI.png
Modena: /assets/elements/modena.css
excerpt: If you're are going to do any custom styling in JavaFX, you need to understand at least the basics about the Modena stylesheet and how it works with the standard JavaFX Nodes.
---

# Introduction

I'm not a big fan of the way that the JavaFX JavaDocs are written, especially the introductory sections.  

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


## SimpleBooleanProperty

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

If you look at abstract `BooleanPropertyBase` class, you'll find that it does 100% of the actual `Property` stuff you need, but none of the Java Bean handling at all.  Those functions are left to whatever concrete class extends from it.  So `SimpleBooleanProperty` only introduces the Java Bean functions required by the interface `ReadOnlyProperty` that is implemented by `BooleanProperty`.

I'll admit that the Java Bean stuff mystifies me a little bit.  I've always assumed that it was used for serialization and de-serialization, but I've yet to see a real case for serializing JavaFX properties.  I've certainly never seen any project where the Java Bean values were actually used for JavaFX `Properties`.  

But still, this is what `SimpleBooleanProperty` (and all the other "Simple...Property" classes) add to the `BooleanProperty`.
