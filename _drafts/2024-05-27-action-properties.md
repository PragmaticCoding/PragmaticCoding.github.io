---
title:  "Dealing With Modena"
date:   2024-03-18 00:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/action-properties
Diagram: /assets/posts/MVCI.png
Modena: /assets/elements/modena.css
excerpt: If your are going to do any custom styling in JavaFX, you need to understand at least the basics about the Modena stylesheet and how it works with the standard JavaFX Nodes.
---

# Introduction


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
