---
title:  "Beginner Mistakes"
date:   2024-03-09 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/beginner_mistakes
ScreenSnap1: /assets/posts/Right_Wrong2.jpeg
ScreenSnap2: /assets/posts/StarterFX.png
ScreenSnap3: /assets/posts/StarterFX.png

excerpt: Dealing with the first level of the hierarchy of competence
---

# The Hierarchy of Competence

The "Hierarchy of Competence" is a well known concept that speaks about the stages that people go through when they learn a skill.  There's a diagram, and it looks like this:

![Hierarchy of Competence]({{page.ScreenSnap1}})

In this article, we're going to talk about what it's like to be at the first stage, "Unconscious Incompetence".

I know that "incompetence" is a loaded word, and it's likely to trigger some people, but in this case it's just a accurate description of someone's lack of skill when they first start out learning something.  It's not a commentary on their value as a person, or their ability to learn, but just a statement that they lack competence.

"Unconscious Incompetence" means that not only do they lack competence, they don't know what they need to learn to gain competence.  In other words, "They don't know what they don't know".  As you move through the stages, eventually you do come to understand how much you don't know, but that takes time.  

# Learning How Much You Don't Know

I had the experience of learning JavaFX when it first came out around 2014, and while I was still a relative beginner to Java.  This was while we were engaged in building real software that was going into production to be used to run a business.  We had to learn, but our focus was producing working code that could be used by real users as soon possible.  

Because of this, I was doomed to have to deal with the code that I wrote for years after I wrote it.  This meant that I forced, over and over again, to face the horrors that a less competent me created, and to realize exactly how incompetent I was just a few months earlier.  

There were a number of occassions where we reached the point that something that we had written a year or two earlier was just too painful to deal with any more.  There was no alternative except to take the time to do a re-write of that code.  In every case that I can recall, it took me about 1/5 as much time to do the re-write than it did to produce the original version.  Additionally, I found that the amount of code in the new version was less than 20% of the amount in the original.  Not "shrunk by 20%", but "reduced down to 20%".  

One time I actually counted lines of code.  The original version had something like 3500 lines.  The new version had only about 500-600! I think that the original took us more than a month to write, while I did the re-write by myself in less than a week.

My recollection of these re-writes was that this was a typical result.  Not an outlier.

The experience was humbling.  I'm pretty sure during the entire span of 7 years that I worked on this system I was pretty stoked about how much I had learnt, how well I was doing, and how much I was improving.  And yet, time and time again I'd look at code I had written only a few months earlier and shake my head.  I'd ask myself, "How could I have thought that was right?"

# What Beginner Code Looks Like

We're going to look at some beginner code, once again from [StackOverflow](https://stackoverflow.com/questions/78152795/how-can-i-make-a-custom-tablecolumn-display-cell-observable).  

Before we get started: I want to emphasize that this is NOT about making fun of the OP in this example, or pointing out their mistakes in order to belittle them.  They are a beginner, and being a beginner is all about making beginner mistakes.  I'm sure that I did stuff just as horrible as the code that we are about to look at when I was a beginner.  Probably even worse.

This is about the code, not the coder.

That being said, nothing is going to be sugar-coated either...

This example was a problem that the OP had with getting `Properties` inside an object to update properly in a `TableView` when those `Properties` were updated from outside the `TableView`.  It's a fairly common issue when dealing with `Properties` composed of `Properties`.

The original code, in Java, uses FXML.  Let's look at that first:

```
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.*?>
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>

<VBox alignment="CENTER" prefHeight="473.0" prefWidth="923.0" spacing="20.0" xmlns="http://javafx.com/javafx/17.0.2-ea" xmlns:fx="http://javafx.com/fxml/1" fx:controller="org.test.tableview.HelloController">
    <padding>
        <Insets bottom="20.0" left="20.0" right="20.0" top="20.0" />
    </padding>
   <TableView fx:id="TableView1" editable="true" prefHeight="200.0" prefWidth="400.0">
     <columns>
        <TableColumn fx:id="col_1" prefWidth="200.0" text="C1" />
        <TableColumn fx:id="col_2" prefWidth="200.0" text="C2" />
        <TableColumn fx:id="col_3" prefWidth="200.0" text="C3" />
        <TableColumn fx:id="col_4" prefWidth="200.0" text="C4" />
     </columns>
   </TableView>

    <Button onAction="#onsetup" text="setup" />
    <Button onAction="#ongoClick" text="GO" />
    <Button onAction="#onbreak" text="break" />
</VBox>
```

This is straight forward, we have a `VBox` with a `TableView` of four columns and three `Buttons`.

``` java
public class myclass
{
    private IntegerProperty fld1 = new SimpleIntegerProperty();
    private StringProperty fld2 = new SimpleStringProperty();
    private StringProperty fld3 = new SimpleStringProperty();
    private StringProperty fld4 = new SimpleStringProperty();

    public myclass(Integer fld1,String fld2,String fld3,String fld4)
    {
        this.fld1.set(fld1);
        this.fld2.set(fld2);
        this.fld3.set(fld3);
        this.fld4.set(fld4);
    }

    // Getter methods
    public IntegerProperty fld1Property() {
        return fld1;
    }

    public StringProperty fld2Property() {
        return fld2;
    }

    public StringProperty fld3Property() {
        return fld3;
    }

    public StringProperty fld4Property() {
        return fld4;
    }
}
```
This is a fairly standard Model class which is a POJO composed of `Properties`.  We don't have the delegated setters and getters for the values, but we do have the `Property` retrieval methods.

``` java
public class HelloController
{
   public TableView<myclass> TableView1;
   public TableColumn<myclass,Integer> col_1;
   public TableColumn<myclass,myclass>  col_2;
   public TableColumn<myclass,String>  col_3;
   public TableColumn<myclass,String>  col_4;

   List<myclass> origlist = new ArrayList<>();


   public HelloController()
   {
       init();
   }

   private void init()
   {
       origlist.clear();
       myclass m = new myclass(1,"A1","B1","C1");
       origlist.add(m);
       m= new myclass(2,"A2","B2","C2");
       origlist.add(m);
       m= new myclass(3,"A3","B3","C3");
       origlist.add(m);
       m= new myclass(4,"A4","B4","C4");
       origlist.add(m);
   }

   @FXML
   public void onsetup(ActionEvent actionEvent)
   {
       init();

       col_1.setCellValueFactory(new PropertyValueFactory<>("fld1"));

       col_2.setCellValueFactory(data -> new ObservableValue<>()
       {
           @Override
           public void addListener(ChangeListener<? super myclass> listener) {}

           @Override
           public void removeListener(ChangeListener<? super myclass> listener) {}

           @Override
           public myclass getValue()
           {
               return data.getValue();
           }

           @Override
           public void addListener(InvalidationListener listener) {}

           @Override
           public void removeListener(InvalidationListener listener) {

           }
       });

       col_3.setCellValueFactory(new PropertyValueFactory<>("fld3"));
       col_4.setCellValueFactory(new PropertyValueFactory<>("fld4"));

       col_2.setCellFactory(p ->
       {
           TableCell<myclass,myclass> cell = new TableCell<>()
           {
               @Override
               protected void updateItem(myclass item, boolean empty)
               {
                   if (item != null)
                   {
                       Label l = new Label();
                       l.setText("combination display " + item.fld2Property().getValue() + " " + item.fld4Property().getValue());
                       setGraphic(l);
                   }
                   else
                   {
                       setGraphic(null);
                       setText(null);
                   }
               }
           };
           return cell;
       });

       final ObservableList<myclass> olpc = FXCollections.observableArrayList();

       olpc.addAll(origlist.stream().toList());

       TableView1.setItems(olpc);
   }

   @FXML
   public void ongoClick(ActionEvent actionEvent)
   {
       origlist.get(0).fld2Property().setValue("Z");
       origlist.get(0).fld4Property().setValue("ZZZ");
   }

   @FXML
   public void onbreak(ActionEvent actionEvent)
   {
       System.out.println(origlist.get(0).fld2Property().get());
   }
}
```
This is the FXML Controller for this application.  We'll look at it a bit closer later on.

This is about 140 lines of code and 20 lines of FXML, and it isn't actually runnable because it doesn't have the Application class and the code that would run the `FXMLLoader`.

## Getting Rid of the FXML

The first issue I have is that we have 20+ lines of FXML for a `VBox` with a `TableView` and three `Buttons`.  The FXML adds nothing here, and it just complicates everything.  You cannot see what the `Buttons` do without looking at the FXML Controller code, and the same applies to the `TableView` columns.  

I've converted this into Kotlin without the FXML, but tried to leave everything as it was originally designed - at least in approach:  

``` kotlin
class BeginnerApproach : Application() {

    private var origlist: MutableList<MyClass> = mutableListOf()
    override fun start(stage: Stage) {
        origlist.clear()
        var m = MyClass(1, "A1", "B1", "C1")
        origlist.add(m)
        m = MyClass(2, "A2", "B2", "C2")
        origlist.add(m)
        m = MyClass(3, "A3", "B3", "C3")
        origlist.add(m)
        m = MyClass(4, "A4", "B4", "C4")
        origlist.add(m)
        stage.scene = Scene(createContent())
        stage.show()
    }

    private fun createContent(): Region = VBox().apply {
        padding = Insets(20.0)
        children += createTable()
        children += Button("setup")
        children += Button("GO").apply {
            setOnAction {
                origlist[0].fld2.value = "Z";
                origlist[0].fld4.value = "ZZZ";
            }
        }
        children += Button("break").apply {
            setOnAction { println(origlist[0].fld2.value); }
        }
    }

    private fun createTable(): Region = TableView<MyClass>().apply {
        columns += TableColumn<MyClass, Int>("C1").apply {
            cellValueFactory = PropertyValueFactory("fld1")
            prefWidth = 200.0
        }
        columns += TableColumn<MyClass, MyClass>("C2").apply {
            setCellValueFactory { data ->
                object : ObservableValue<MyClass> {
                    override fun addListener(listener: ChangeListener<in MyClass?>?) {}

                    override fun removeListener(listener: ChangeListener<in MyClass?>?) {}

                    override fun getValue(): MyClass = data.value

                    override fun addListener(listener: InvalidationListener) {}

                    override fun removeListener(listener: InvalidationListener) {
                    }
                }
            }
            setCellFactory {
                object : TableCell<MyClass, MyClass>() {
                    override fun updateItem(item: MyClass?, empty: Boolean) {
                        if (item != null) {
                            val l = Label()
                            l.text = ("combination display " + item.fld2.value) + " " + item.fld4.value
                            graphic = l
                        } else {
                            graphic = null
                            text = null
                        }
                    }
                }
            }
        }
        columns += TableColumn<MyClass, Int>("C3").apply {
            setCellValueFactory(PropertyValueFactory("fld3"))
            prefWidth = 200.0
        }
        columns += TableColumn<MyClass, Int>("C4").apply {
            setCellValueFactory(PropertyValueFactory("fld4"))
            prefWidth = 200.0
        }
        val olpc: ObservableList<MyClass> = FXCollections.observableArrayList<MyClass>()
        olpc.addAll(origlist.stream().toList())
        setItems(olpc)
    }
}

class MyClass(v1: Int, v2: String, v3: String, v4: String) {
    private val fld1: IntegerProperty = SimpleIntegerProperty(v1)
    val fld2: StringProperty = SimpleStringProperty(v2)
    val fld3: StringProperty = SimpleStringProperty(v3)
    val fld4: StringProperty = SimpleStringProperty(v4)

    fun fld1Property() = fld1
    fun fld2Property() = fld2
    fun fld3Property() = fld3
    fun fld4Property() = fld4
}


fun main() = Application.launch(BeginnerApproach::class.java)
```
It's now a shade under 100 lines of code in total, has no FXML, and it will run because it has the Application class and a `main()`.  Kotlin shrinks it a little bit, so it would probably take a few extra lines to do this in Java.

Why count lines?  Because less code is (usually) better code.  Less to read, less to understand, less opportunities to write bugs.

## Reworking the Approach

``` kotlin
class NonBeginnerApproach : Application() {

    private val origList = listOf(
        MyClass2(1, "A1", "B1", "C1"),
        MyClass2(2, "A2", "B2", "C2"),
        MyClass2(3, "A3", "B3", "C3"),
        MyClass2(4, "A4", "B4", "C4")
    )

    override fun start(stage: Stage) {
        stage.scene = Scene(createContent())
        stage.show()
    }

    private fun createContent(): Region = VBox().apply {
        padding = Insets(20.0)
        children += createTable()
        children += Button("GO").apply {
            setOnAction {
                origList[0].fld2.value = "Z";
                origList[0].fld4.value = "ZZZ";
            }
        }
        children += Button("break").apply {
            setOnAction { println(origList[0].fld2.value); }
        }
    }

    private fun createTable(): Region = TableView<MyClass2>().apply {
        columns += createColumn<Int>("C1") { it.value.fld1 }
        columns += createColumn<String>("C2") {
            Bindings.concat(it.value.fld2.map { ob -> "combination display $ob " }, it.value.fld4)
        }
        columns += createColumn<String>("C3") { it.value.fld3 }
        columns += createColumn<String>("C4") { it.value.fld4 }
        items += origList
    }

    private fun <T : Any?> createColumn(
        name: String,
        valueCallback: Callback<TableColumn.CellDataFeatures<MyClass2, T>, ObservableValue<T>>
    ) = TableColumn<MyClass2, T>(name).apply {
        cellValueFactory = valueCallback
        prefWidth = 200.0
    }
}

class MyClass2(v1: Int, v2: String, v3: String, v4: String) {
    val fld1: ObjectProperty<Int> = SimpleObjectProperty(v1)
    val fld2: StringProperty = SimpleStringProperty(v2)
    val fld3: StringProperty = SimpleStringProperty(v3)
    val fld4: StringProperty = SimpleStringProperty(v4)
}


fun main() = Application.launch(NonBeginnerApproach::class.java)

```

We are now down to 55 lines of code!  It almost all fits on the screen at once.

## Looking at the Issues





```

The observable value you're returning in your cell value factory does not ever fire any invalidation events, so the column has no idea anything has changed. The best solution is probably to create your observable list with an extractor. That way changes to the properties returned by the extractor will cause a change event to be fired by the observable list, ultimately leading to the updateItem method of the cell being invoked so that the UI can be updated. –
Slaw
6 hours ago
Related: How to get row value inside updateItem() of CellFactory. –
jewelsea
5 hours ago
Also: Prefer lambdas over PropertyValueFactories. For instance, col2.setCellValueFactory(data -> new SimpleObjectProperty<MyClass>(data.getValue()));. (I used java naming conventions). –
jewelsea
5 hours ago
1
Maybe I am not understanding your issue, but try final ObservableList<myclass> olpc = FXCollections.observableArrayList(model -> new Observable[]{model.fld2Property(),model.fld4Property()}); –
SedJ601
2 hours ago
1
Either, as previously suggested, create the list with an extractor, or bind/unbind the label's text property in the updateItem() method. Note you should not be creating a new label every time in the updateItem() method; just create one label per cell. –
James_D
2 hours ago
You are better off not passing the whole object to the TableColumn, but rather using a Binding to create an ObservableValue for CellValueFactory. Something like cdf -> cdf.value.fld2Property().concat(" ").concat(cdf.value.fld4Property()). Then you're done. –
DaveB
55 mins ago

I cover this stuff in my articles on TableView. This specific situation is talked about here: pragmaticcoding.ca/javafx/elements/… –
DaveB
54 mins ago

@DaveB won't that throw null pointer exceptions for empty cells? –
James_D
36 mins ago

```
