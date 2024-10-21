---
title:  "EdenCoding's Guide to TableView Styling"
date:   2024-08-03 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/elements/eden-tableView
ScreenSnap1: /assets/elements/ListObservables1.png
ScreenSnap2: /assets/elements/ListObservables2.png
ScreenSnap3: /assets/elements/ListObservables3.png
ScreenSnap4: /assets/elements/ListObservables4.png
Diagram: /assets/elements/ListProperties.png
part1: /javafx/elements/observable-classes-generics
part2: /javafx/elements/observable-classes-typed

excerpt: A look at the Observable classes that wrap ObservableList
---

# What we’ll achieve in this tutorial

In this tutorial, I’ll go through which selectors are available to you as a developer, which I’ll split into four categories:

* Headings
* Cells and Rows
* Scrollbars
* Pseudoclasses, based on cell values

In each category, we’ll go through the selectors and how we can use them. In the Header’s section, we’ll also use the .table-view .corner selector, which is the frustrating rectangle above the scrollbar most people miss!

By the end of the tutorial, we’ll have a fully styled TableView that looks like this:
EdenCode simple UI with a TableView that has not been styled using CSS. Row highlighting has been completed with pseudoclasses
# Selectors

Knowing the selectors for a JavaFX TableView can be challenging and the JavaFX CSS Reference Guide can seem like a bit like a maze. Starting with the TableView, you’re directed to Controls, Nodes, StackPanes and Labels and that’s before we’ve got to scrolling.

A completely unstyled table, which we’ll start with, actually has styles from the default JavaFX CSS file, Modena.css
EdenCode simple UI with a TableView that has not been styled using CSS

In this case, the table we’ll use is just a summary of orders, and not particularly useful summary either. For each Order, we’ll record the id, the state, and the city to which the order’s going to be delivered. Once Janet from Sales asks us to post them out, everyone’s going to find out we forgot to record the majority of the address. Until then, let’s make the table look really good and add it to our CV.

# Styling a TableView

Selectors in the JavaFX TableView can be broken down into three basic groups – headers, rows, and scrollbars.

The TableView itself has a selector – .table-view, which is rendered right at the bottom underneath everything else. This is a Region, and so can have background colour or image, border colour, border stroke types, and a whole lot more. To see all the options for styling a Region, check out the CSS style guide for Region.

.table-view {
    -fx-background-color: transparent;
}

Before we jump into the rest of the table, we’ll just cover two awkward selectors that most people find really hard to style.


# Awkward Selectors

There are a two selectors that are almost impossible to find unless you know what you’re looking for.

## The corner of the header section

The top right corner of the header section above the scroll bars can be styled with the .table-view .column-header-background .filler.

Again, this is a Region, so it can have a lot of customisation. Because I was looking for quite a minimal styling when I designed this table, I chose to make this transparent, but you can access a world of possibilities.

.table-view .column-header-background .filler {
    -fx-background-color: transparent;
}

## The bottom right corner between the scrollbars

The bottom right corner between the scrollbars can be styled using the selector .table-view .corner.

Again, also a Region. In fact, most of the selectors across JavaFX tend to be a Region – although many extend Region (or Pane, which is a subclass of Region) and have some extra selectors stacked on top of that.

You could even make one of them a particular shape using the -fx-shape selector if you wanted some custom iconography on your table. But, for now, I’ll make it transparent.

.table-view .corner {
    -fx-background-color: transparent;
}

# Headers

The header of a TableView has two main sections:

    The Global header section, which is rendered underneath all of the other headers and extends across the entire header area
    The header of each column itself

# Global header background

For a smooth appearance, we’re going to give the entire header bar a consistent colour (a slight gradient), using the Global header section selector .table-view .column-header-background.

This area is also a Region, which comes with all the customisation options available to a Region. In this case, we’ll use four selectors to style the background area:

    -fx-background-color
    -fx-background-radius
    -fx-background-insets
    -fx-padding

This will allow us to create a coloured background, with rounded corners at the top left and top right. We will inset the background by the width of the scrollbar, so that the right edge of the header is flush with the table. Finally, we’ll use some padding to make the bottom headers a little further from the table contents.

.table-view .column-header-background{
    -fx-background-color: linear-gradient(to bottom, #1dbbdd44, #93f9b944);
    -fx-background-radius: 7px 7px 0px 0px;
    -fx-background-insets: 0 11px 0 0;
    -fx-padding: 0 0 5px 0;
}

# Individual Column Headers

Individual column headers are rendered on top of the column background, and so we need to either style them consistently or make them transparent.We can access them using the.table-view .column-header selector.

Giving headers a vertical gradient won’t stretch across a nested column. So in Location -> State and Location -> City, we’d have one gradient going vertically in the Location header, and one in the State/City header. I don’t particularly like that effect, so I’ve opted to make them transparent.

.table-view .column-header {
    -fx-background-color: transparent;
}

By setting this, we can give the header section a clean, linear gradient. We’ve set the background radius to give the top corners a curved look, leaving the bottom flush with the table. The background insets ensure the right of the header is flush with the right of the table. Ordinarily, the header will extend to the outside of the scrollbar.
EdenCode simple UI with a TableView where the column headers have been styled using CSS

# Cells and Rows

Next, we’ll style the rows so they have a similar style and a semi-transparent fill. We can access separate styles for odd and even rows with pseudo-classes, and we can greater improve readability by highlighting cells as the mouse hovers over them.

.table-view .table-cell{
    -fx-border-color: transparent;
    -fx-padding: 2 0 2 10px;
}
.table-row-cell: hover {
    -fx-background-color: #93f9b911;
    -fx-text-background-color: red;
}
.table-row-cell: odd{
    -fx-background-color: #93f9b911;
    -fx-background-insets: 0, 0 0 1 0;
    -fx-padding: 0.0em;
}
.table-row-cell: even{
    -fx-background-color: #1dbbdd11;
    -fx-background-insets: 0, 0 0 1 0;
    -fx-padding: 0.0em;
}

EdenCode simple UI with a TableView where the column headers, cells and scrollpanes have been styled using CSS

The table is almost complete, but as we add more orders, JavaFX tries to compensate by adding scrollbars – horizontal and vertical.

# Scrollbars

To style these, we need to access them through the .virtual-flow .scroll-bar selector. The .track and .track-background selectors correspond to the space along which the scrollbar moves. We’ll set these to transparent, because we want to keep the same consistent, clean feel.

.table-view .virtual-flow .scroll-bar:vertical,
.table-view .virtual-flow .scroll-bar:vertical .track,
.table-view .virtual-flow .scroll-bar:vertical .track-background,
.table-view .virtual-flow .scroll-bar:horizontal,
.table-view .virtual-flow .scroll-bar:horizontal .track,
.table-view .virtual-flow .scroll-bar:horizontal .track-background {
    -fx-background-color: transparent;
}

The arrows on either side of the scrollbar are accessed through .increment-button and .decrement-button. As with any node, they can be customized using the -fx-shape attribute, which gives you complete control over their shape. For a clean look, we’ll make them transparent by setting their opacity to zero.

.table-view .virtual-flow .scroll-bar .increment-button,
.table-view .virtual-flow .scroll-bar .decrement-button {
    -fx-opacity: 0;
    -fx-padding: 2;
}

The scrollbar itself is referred to using the .thumb selector. Because we’re setting a gradient, and we want the look to be consistent, we’ll set the vertical bar’s gradient from top to bottom, and the horizontal bar’s gradient from left to right.

.table-view .virtual-flow .scroll-bar:vertical .thumb{
    -fx-background-color:  linear-gradient(to bottom, #1dbbdd44, #93f9b944);
}
.table-view .virtual-flow .scroll-bar:horizontal .thumb {
    -fx-background-color:  linear-gradient(to right, #1dbbdd44, #93f9b944);
}

EdenCode simple UI with a TableView that has been styled using CSS

We’ve now set all of the default styles for the TableView. For even more customisation we’ll explore how to set custom pseudo-classes, similar to ‘odd’ and ‘even’ above. With these in hand, we can use them to trigger custom styling based on the contents of a row.
# Value-specific Styling

JavaFX provides plenty of functionality for predicates, that is styling something based on a condition. Here, we might want to highlight important orders – those we’ll deliver to the biggest cities. Maybe our marketing specialists have told us that we’ll increase revenue if we send promotional material to the people in densely populated areas. So, we duly implement it in the code.

To start, we need to define a special case for a table row, which we’ll call highlighted. This is called a pseudo-class, because highlighted cannot exist without another selector. Just like odd and even before, which wouldn’t have made sense on their own.

Because they’re represent an important demographic, we’ll color them green using the hex code #81c483, and we’ll round the corners to increase the effect.

.table-row-cell: highlighted {
    -fx-background-color: #81c483;
    -fx-background-radius: 15px;
}

To determine which rows get highlighted, we can specify a condition based on the attributes of the Order class. To do that, we need to understand a little about how JavaFX reads our Orders into rows.
# Cell Values

You may have noticed that when we defined the TableView, we parameterised it with the type Order.

public TableView<Order> exampleTable;

Then, when we define how to populate a column, we define it through a CellValueFactory.

stateColumn.setCellValueFactory(cellData -> cellData.getValue().stateProperty());

To populaye the table, we pass an ObservableList of Orders into the TableView. Then, for every Order in the ObservableList, JavaFX finds the State property and uses it to get generate the data for that cell in the stateColumn.

As well as defining how JavaFX generates the values relative to the columns, we can do the same thing for rows, which it generates using a RowFactory.
# Row Factories

For each Order in the ObservableList, JavaFX uses the RowFactory to create a TableRow. We can hook into that process to change the styling on the row, depending on the value of the Order we’re currently at.

exampleTable.setRowFactory(tableView -> {
    TableRow<Order> row = new TableRow<>();
    return row;
});

So, assuming an Order has access to the population of the city to which it’s delivered, in between defining our row and returning it, we test whether that number is greater than 4 million. In the case of a positive result, we set the pseudo-class for additional styling.

row.pseudoClassStateChanged(PseudoClass.getPseudoClass("highlighted"),
                    newOrder.getCityPopulation() > 4e6));

EdenCode simple UI with a TableView that has not been styled using CSS. Row highlighting has been completed with pseudoclasses

And that’s it!
# Conclusions

So, we can customise the JavaFX TableView by identifying and styling each of the TableView’s CSS selectors. These fell into three main groups:

    Headers
    Cells / Rows
    Scrollbars

In cases where we need to highlight rows to improve the user experience, we can do so by creating and applying pseudo-classes.

The benefit of achieving this with CSS is that, once we’ve set the style, it can be used throughout our program. There’s no need to repeat code or create TableViewFactory classes to churn out pre-styled content.

As always, you’re welcome to use the code to dig around in and help you design your own user interfaces. It can be found in the edencoding GitHub repo here.
cssstyling
You may also like
Interacting with nodes in a GridPane layout – JavaFX
December 11, 2020
How to use JavaFX Layouts
September 17, 2020
How to add an image to a button (and position it) in JavaFX
July 2, 2021

    Privacy Policy

2020 © EdenCoding
Back to top
Close

    Home
    JavaFX
        Core JavaFX Tips
        Animations
        UI Elements
        Game Development
    Resources (new)
    Latest Posts!
    About Me
