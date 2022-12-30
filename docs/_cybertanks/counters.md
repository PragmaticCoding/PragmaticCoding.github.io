---
excerpt: Taking a first look at how we'll display the various counters used for the defending units.
CpAlpha: /assets/cybertanks/CpAlpha.png
HeavyTank: /assets/cybertanks/HeavyTank.png
LightTank: /assets/cybertanks/LightTank.png
MslTank: /assets/cybertanks/MslTank.png
Gev: /assets/cybertanks/Gev.png
Howitzer: /assets/cybertanks/CpAlpha.png
Infantry1: /assets/cybertanks/Infantry1.png
Infantry2: /assets/cybertanks/Infantry2.png
permalink: /javafx/cybertanks/counters
---

![CyberTank]({{page.banner}})

# Counters for the Defenders

Now we're going to look at the `Image` and `ImageView` setup for the counters that represent the defending units.

# The Counters

CheckPoint Alpha

![CheckPoint Alpha]({{page.CpAlpha}})

Heavy Tank

![Heavy Tank]({{page.HeavyTank}})

Light Tank

![Light Tank]({{page.LightTank}})

Missile Tank

![Missile Tank]({{page.MslTank}})

Ground Effect Vehicle (GEV)

![GEV]({{page.Gev}})

Howitzer

![Howitzer]({{page.Howitzer}})

Infantry (Single Unit)

![Single Infantry]({{page.Infantry1}})

Infantry (Two Units)

![Double Infantry]({{page.Infantry2}})

Infantry are the only units where two are allowed to occupy the same `Tile`, so we need to have a counter for each situation.

# Linking the Images to Data

We need a class to be able to treat these `Images` more like data.  Since it's a set of discrete values, an `Enum` makes sense:

``` kotlin
enum class CounterType(val label: String, private val resourceName: String) {
   NONE("None", ""),
   HVYTK("Heavy Tank", "HeavyTank"),
   LGHTK("Light Tank", "LightTank"),
   HWZR("Howitzer", "Howitzer"),
   INFTY1("Infantry 1", "Infantry1"),
   INFTY2("Infantry 2", "Infantry2"),
   GEV("Gev", "Gev"),
   MSLTK("Missile Tank", "MslTank"),
   CPA("CP Alpha", "CpAlpha"),
   CT1("CyberTank1", "cybertank1");

   companion object {
      private val images = mutableMapOf<CounterType, Image?>()

      private fun createImage(resourceName: String) =
         CounterType::class.java.getResource("/counters/black/${resourceName}.png")?.toExternalForm()?.let { Image(it) }

      private fun getImage(counterType: CounterType): Image? {
         if (!images.contains(counterType)) {
            images[counterType] = createImage(counterType.resourceName)
            println("Creating image: ${counterType.name} -> ${images[counterType]}")
         }
         return images[counterType]
      }

      fun byLabel(label: String) = values().find { it.label == label } ?: NONE
   }

   val image: Image? by lazy { getImage(this) }
}
```

Now we can refer to the counters by their `Enum` value, and pull the image via the `getImage()` function.  

For now, that's all we need.
