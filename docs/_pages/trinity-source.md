---
layout: single
title: Trinity Project Source Code
toc: false
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4
sidebar:
    nav: "master"
permalink: /pages/trinity-source
brain: /assets/logos/brain.png
---
# Trinity Source

I was concerned that the source code that I used for my article about converting FXML to Kotlin would someday become out of date, or removed from GitHub, so I thought I would preserve a copy for permanent reference on this website.

# The FXML File

``` xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>

<AnchorPane fx:id="root" style="-fx-background-color: #00000000;" xmlns="http://javafx.com/javafx" xmlns:fx="http://javafx.com/fxml"
            fx:controller="edu.jhuapl.trinity.javafx.controllers.ManifoldControlController">
    <children>
        <TabPane fx:id="tabPane" tabClosingPolicy="UNAVAILABLE">
            <tabs>
                <Tab closable="false" text="UMAP">
                    <content>
                        <BorderPane fx:id="majorPane" minHeight="200.0" minWidth="400.0">
                            <children>
                            </children>
                            <top>
                            </top>
                            <center>
                                <GridPane hgap="10.0" vgap="5.0" BorderPane.alignment="CENTER">
                                    <columnConstraints>
                                        <ColumnConstraints hgrow="SOMETIMES" maxWidth="288.0" minWidth="10.0" prefWidth="206.0"/>
                                        <ColumnConstraints hgrow="SOMETIMES" maxWidth="380.0" minWidth="10.0" prefWidth="380.0"/>
                                    </columnConstraints>
                                    <rowConstraints>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                    </rowConstraints>
                                    <children>
                                        <Label text="Number of Components"/>
                                        <Spinner fx:id="numComponentsSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="1"/>
                                        <Label text="Number of Epochs" GridPane.rowIndex="2"/>
                                        <Spinner fx:id="numEpochsSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="3"/>
                                        <Label text="Nearest Neighbors" GridPane.rowIndex="4"/>
                                        <Label text="Negative Sample Rate" GridPane.rowIndex="6"/>
                                        <Label text="Local Connectivity" GridPane.rowIndex="8"/>
                                        <Spinner fx:id="nearestNeighborsSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="5"/>
                                        <Spinner fx:id="negativeSampleRateSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="7"/>
                                        <Spinner fx:id="localConnectivitySpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="9"/>
                                        <HBox spacing="10.0" GridPane.columnIndex="1" GridPane.rowIndex="10" GridPane.rowSpan="2">
                                            <children>
                                                <VBox alignment="CENTER_LEFT" spacing="10.0">
                                                    <children>
                                                        <Label text="Distance Metric"/>
                                                        <ChoiceBox fx:id="metricChoiceBox" prefWidth="200.0"/>
                                                    </children>
                                                </VBox>
                                                <VBox alignment="CENTER_LEFT" layoutX="10.0" layoutY="10.0" spacing="10.0">
                                                    <children>
                                                        <Label text="Threshold (if applicable)"/>
                                                        <Spinner fx:id="thresholdSpinner" prefWidth="150.0">
                                                            <editable>true</editable>
                                                            <valueFactory>
                                                                <SpinnerValueFactory.DoubleSpinnerValueFactory min="0.01" max="1.0" initialValue="0.1"
                                                                                                               amountToStepBy="0.01"/>
                                                            </valueFactory>
                                                        </Spinner>
                                                    </children>
                                                </VBox>
                                            </children>
                                            <padding>
                                                <Insets top="5.0"/>
                                            </padding>
                                        </HBox>
                                        <Label text="Repulsion Strength" GridPane.columnIndex="1"/>
                                        <Label text="Minimum Distance" GridPane.columnIndex="1" GridPane.rowIndex="2"/>
                                        <Label text="Spread" GridPane.columnIndex="1" GridPane.rowIndex="4"/>
                                        <Label text="Op Mix Ratio" GridPane.columnIndex="1" GridPane.rowIndex="6"/>
                                        <Slider fx:id="repulsionSlider" blockIncrement="0.1" majorTickUnit="0.1" max="2.0" showTickLabels="true"
                                                showTickMarks="true" snapToTicks="true" value="1.0" GridPane.columnIndex="1" GridPane.rowIndex="1"/>
                                        <Slider fx:id="minDistanceSlider" blockIncrement="0.1" majorTickUnit="0.1" max="0.6" showTickLabels="true"
                                                showTickMarks="true" snapToTicks="true" value="0.1" GridPane.columnIndex="1" GridPane.rowIndex="3"/>
                                        <Slider fx:id="spreadSlider" blockIncrement="0.1" majorTickUnit="0.1" max="1.5" min="0.5" showTickLabels="true"
                                                showTickMarks="true" snapToTicks="true" value="1.0" GridPane.columnIndex="1" GridPane.rowIndex="5"/>
                                        <Slider fx:id="opMixSlider" blockIncrement="0.1" majorTickUnit="0.1" max="1.0" showTickLabels="true"
                                                showTickMarks="true" snapToTicks="true" value="0.5" GridPane.columnIndex="1" GridPane.rowIndex="7"/>
                                        <HBox alignment="CENTER" spacing="15.0" GridPane.columnSpan="2147483647" GridPane.halignment="CENTER"
                                              GridPane.rowIndex="12" GridPane.rowSpan="2" GridPane.valignment="CENTER">
                                            <children>
                                                <VBox alignment="CENTER" spacing="10.0">
                                                    <children>
                                                        <Button mnemonicParsing="false" onAction="#loadUmapConfig" prefWidth="200.0" text="Load New Config"/>
                                                        <Button mnemonicParsing="false" onAction="#saveUmapConfig" prefWidth="200.0"
                                                                text="Save Current Config"/>
                                                    </children>
                                                </VBox>
                                                <VBox alignment="CENTER_LEFT" spacing="10.0">
                                                    <children>
                                                        <RadioButton fx:id="useHypersurfaceButton" mnemonicParsing="false" text="Use Hypersurface"/>
                                                        <RadioButton fx:id="useHyperspaceButton" mnemonicParsing="false" selected="true" text="Use Hyperspace"/>
                                                        <CheckBox fx:id="verboseCheckBox" mnemonicParsing="false" selected="true" text="Progress Output"/>
                                                    </children>
                                                    <padding>
                                                        <Insets bottom="5.0" left="5.0" right="5.0" top="5.0"/>
                                                    </padding>
                                                </VBox>
                                                <VBox alignment="CENTER" spacing="10.0">
                                                    <children>
                                                        <Button defaultButton="true" mnemonicParsing="false" onAction="#project" prefWidth="200.0"
                                                                text="Run UMAP"/>
                                                        <Button mnemonicParsing="false" onAction="#exportMatrix" prefWidth="200.0" text="Export TMatrix"/>
                                                        <Button mnemonicParsing="false" onAction="#saveProjections" prefWidth="200.0"
                                                                text="Export Projections"/>
                                                    </children>
                                                </VBox>
                                            </children>
                                            <padding>
                                                <Insets bottom="2.0" left="2.0" right="2.0" top="2.0"/>
                                            </padding>
                                        </HBox>
                                        <Label text="Target Weight" GridPane.columnIndex="1" GridPane.rowIndex="8"/>
                                        <Slider fx:id="targetWeightSlider" blockIncrement="0.1" majorTickUnit="0.1" max="1.0" showTickLabels="true"
                                                showTickMarks="true" snapToTicks="true" value="0.5" GridPane.columnIndex="1" GridPane.rowIndex="9"/>
                                    </children>
                                    <padding>
                                        <Insets bottom="25.0" left="25.0" right="25.0" top="25.0"/>
                                    </padding>
                                </GridPane>
                            </center>
                        </BorderPane>
                    </content>
                </Tab>
                <Tab closable="false" text="PCA">
                    <content>
                        <BorderPane fx:id="majorPane" minHeight="200.0" minWidth="400.0">
                            <children>
                            </children>
                            <top>
                            </top>
                            <center>
                                <GridPane hgap="10.0" vgap="5.0" BorderPane.alignment="CENTER">
                                    <columnConstraints>
                                        <ColumnConstraints hgrow="SOMETIMES" maxWidth="288.0" minWidth="10.0" prefWidth="200.0"/>
                                        <ColumnConstraints hgrow="SOMETIMES" maxWidth="380.0" minWidth="10.0" prefWidth="200.0"/>
                                    </columnConstraints>
                                    <rowConstraints>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                    </rowConstraints>
                                    <children>
                                        <Label text="Number of Components"/>
                                        <Spinner fx:id="numPcaComponentsSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="1"/>
                                        <HBox spacing="20.0" GridPane.rowIndex="3" GridPane.rowSpan="2">
                                            <children>
                                                <VBox spacing="15.0">
                                                    <children>
                                                        <Label text="Fit Start Index"/>
                                                        <Spinner fx:id="fitStartIndexSpinner" editable="true" prefWidth="100.0"/>
                                                    </children>
                                                </VBox>
                                                <VBox spacing="15.0">
                                                    <children>
                                                        <Label text="Fit End Index"/>
                                                        <Spinner fx:id="fitEndIndexSpinner" editable="true" prefWidth="100.0"/>
                                                    </children>
                                                </VBox>
                                            </children>
                                        </HBox>
                                        <RadioButton fx:id="pcaRadioButton" mnemonicParsing="false" selected="true" text="PCA (EigenValue)"
                                                     GridPane.columnIndex="1" GridPane.rowIndex="1"/>
                                        <Label text="Component Analysis Type" GridPane.columnIndex="1"/>
                                        <RadioButton fx:id="svdRadioButton" mnemonicParsing="false" text="Singular Value Decomposition" GridPane.columnIndex="1"
                                                     GridPane.rowIndex="2"/>
                                        <RadioButton fx:id="pcaUseHypersurfaceButton" mnemonicParsing="false" text="Use Hypersurface" GridPane.columnIndex="1"
                                                     GridPane.rowIndex="4"/>
                                        <RadioButton fx:id="pcaUseHyperspaceButton" mnemonicParsing="false" selected="true" text="Use Hyperspace"
                                                     GridPane.columnIndex="1" GridPane.rowIndex="5"/>
                                        <Label text="Output Scaling Factor" GridPane.rowIndex="5"/>
                                        <Spinner fx:id="pcaScalingSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="6"/>
                                        <HBox alignment="CENTER" spacing="10.0" GridPane.columnSpan="2147483647" GridPane.rowIndex="8">
                                            <children>
                                                <Button defaultButton="true" mnemonicParsing="false" onAction="#runPCA" prefWidth="175.0" text="Project Data"/>
                                                <Button mnemonicParsing="false" onAction="#saveProjections" prefWidth="175.0" text="Export Projections"/>
                                            </children>
                                            <padding>
                                                <Insets bottom="5.0" left="5.0" right="5.0" top="5.0"/>
                                            </padding>
                                        </HBox>
                                        <Label text="Input Data Source" GridPane.columnIndex="1" GridPane.rowIndex="3"/>
                                        <CheckBox fx:id="rangedFittingCheckBox" mnemonicParsing="false" text="Enabled Ranged Fitting (Experimental)"
                                                  GridPane.rowIndex="2"/>
                                    </children>
                                    <padding>
                                        <Insets bottom="15.0" left="15.0" right="15.0" top="15.0"/>
                                    </padding>
                                </GridPane>
                            </center>
                        </BorderPane>
                    </content>
                </Tab>
                <Tab closable="false" text="Distances">
                    <content>
                        <BorderPane fx:id="majorPane" minHeight="200.0" minWidth="400.0">
                            <children>
                            </children>
                            <top>
                            </top>
                            <center>
                                <GridPane hgap="10.0" vgap="5.0" BorderPane.alignment="CENTER">
                                    <columnConstraints>
                                        <ColumnConstraints hgrow="SOMETIMES" maxWidth="288.0" minWidth="10.0" prefWidth="206.0"/>
                                        <ColumnConstraints fillWidth="false" hgrow="NEVER"/>
                                        <ColumnConstraints hgrow="SOMETIMES" maxWidth="380.0" minWidth="10.0" prefWidth="380.0"/>
                                    </columnConstraints>
                                    <rowConstraints>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="NEVER"/>
                                    </rowConstraints>
                                    <children>
                                        <Label text="Connector Thickness" GridPane.rowIndex="5"/>
                                        <VBox alignment="CENTER_LEFT" spacing="5.0" GridPane.rowIndex="7" GridPane.rowSpan="2">
                                            <children>
                                                <Label text="Connector Color"/>
                                                <ColorPicker fx:id="connectorColorPicker" editable="true" minHeight="40.0" prefWidth="200.0"
                                                             promptText="Change the color of the 3D connector"/>
                                            </children>
                                            <padding>
                                                <Insets bottom="5.0" left="5.0" right="5.0" top="5.0"/>
                                            </padding>
                                        </VBox>
                                        <Spinner fx:id="connectorThicknessSpinner" editable="true" prefWidth="100.0" GridPane.rowIndex="6"/>
                                        <HBox alignment="CENTER" spacing="20.0" GridPane.columnIndex="2">
                                            <children>
                                                <Label text="Collected Distances"/>
                                                <Button mnemonicParsing="false" onAction="#clearAllDistances" prefWidth="150.0" text="Clear All"/>
                                            </children>
                                        </HBox>
                                        <VBox spacing="5.0" GridPane.rowSpan="2">
                                            <children>
                                                <Label text="Distance Metric"/>
                                                <TextField fx:id="distanceMetricTextField" minHeight="40.0" promptText="Select Distance From ListView"/>
                                            </children>
                                            <padding>
                                                <Insets bottom="5.0" left="5.0" right="5.0" top="5.0"/>
                                            </padding>
                                        </VBox>
                                        <ScrollPane fitToHeight="true" fitToWidth="true" pannable="true" GridPane.columnIndex="2" GridPane.rowIndex="1"
                                                    GridPane.rowSpan="2147483647">
                                            <content>
                                                <ListView fx:id="distancesListView" prefHeight="200.0" prefWidth="200.0">
                                                    <padding>
                                                        <Insets bottom="5.0" left="5.0" right="5.0" top="5.0"/>
                                                    </padding>
                                                </ListView>
                                            </content>
                                        </ScrollPane>
                                        <Separator orientation="VERTICAL" prefHeight="200.0" GridPane.columnIndex="1" GridPane.rowSpan="2147483647"/>
                                        <RadioButton fx:id="pointToGroupRadioButton" disable="true" mnemonicParsing="false" text="Point to Group"
                                                     GridPane.rowIndex="3"/>
                                        <RadioButton fx:id="pointToPointRadioButton" mnemonicParsing="false" selected="true" text="Point to Point"
                                                     GridPane.rowIndex="2"/>
                                    </children>
                                    <padding>
                                        <Insets bottom="10.0" left="10.0" right="10.0" top="10.0"/>
                                    </padding>
                                </GridPane>
                            </center>
                        </BorderPane>
                    </content>
                </Tab>
                <Tab closable="false" text="Hull Geometry">
                    <content>
                        <BorderPane>
                            <center>
                                <VBox spacing="5.0" BorderPane.alignment="CENTER">
                                    <children>
                                        <HBox alignment="CENTER_LEFT" spacing="20.0">
                                            <children>
                                                <Label text="Generated Manifolds"/>
                                            </children>
                                        </HBox>
                                        <HBox alignment="CENTER" layoutX="15.0" layoutY="15.0" spacing="20.0">
                                            <children>
                                                <Button mnemonicParsing="false" onAction="#clearAll" prefWidth="125.0" text="Clear All"/>
                                                <Button layoutX="100.0" layoutY="10.0" mnemonicParsing="false" onAction="#exportAll" prefWidth="125.0"
                                                        text="Export All"/>
                                            </children>
                                        </HBox>
                                        <ScrollPane fitToHeight="true" fitToWidth="true" pannable="true">
                                            <content>
                                                <ListView fx:id="manifoldsListView">
                                                    <padding>
                                                        <Insets bottom="5.0" left="5.0" right="5.0" top="5.0"/>
                                                    </padding>
                                                </ListView>
                                            </content>
                                        </ScrollPane>
                                    </children>
                                    <padding>
                                        <Insets bottom="5.0" left="5.0" right="5.0" top="5.0"/>
                                    </padding>
                                </VBox>
                            </center>
                            <top>
                                <HBox alignment="CENTER" BorderPane.alignment="CENTER">
                                    <children>
                                        <GridPane maxHeight="1.7976931348623157E308" maxWidth="1.7976931348623157E308">
                                            <columnConstraints>
                                                <ColumnConstraints halignment="CENTER" hgrow="ALWAYS" maxWidth="-Infinity" minWidth="10.0" prefWidth="150.0"/>
                                                <ColumnConstraints halignment="CENTER" hgrow="ALWAYS" maxWidth="216.0" minWidth="10.0" prefWidth="214.0"/>
                                                <ColumnConstraints halignment="CENTER" hgrow="ALWAYS" maxWidth="145.0" minWidth="10.0" prefWidth="118.0"/>
                                                <ColumnConstraints halignment="CENTER" hgrow="ALWAYS" maxWidth="145.0" minWidth="10.0" prefWidth="118.0"/>
                                            </columnConstraints>
                                            <rowConstraints>
                                                <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="ALWAYS"/>
                                                <RowConstraints minHeight="10.0" prefHeight="40.0" vgrow="ALWAYS"/>
                                                <RowConstraints minHeight="10.0" prefHeight="40.0" vgrow="ALWAYS"/>
                                                <RowConstraints minHeight="10.0" vgrow="ALWAYS"/>
                                            </rowConstraints>
                                            <children>
                                                <Label layoutX="15.0" layoutY="62.0" text="Point Set"/>
                                                <HBox alignment="CENTER" GridPane.columnIndex="1" GridPane.valignment="CENTER">
                                                    <children>
                                                        <RadioButton fx:id="useVisibleRadioButton" mnemonicParsing="false" prefWidth="75.0" selected="true"
                                                                     text="Visible"/>
                                                        <RadioButton fx:id="useAllRadioButton" mnemonicParsing="false" prefWidth="75.0" text="All"/>
                                                    </children>
                                                </HBox>
                                                <Label text="Distance Tolerance" textAlignment="CENTER" GridPane.rowIndex="1"/>
                                                <HBox alignment="CENTER" spacing="10.0" GridPane.columnIndex="1" GridPane.rowIndex="1">
                                                    <children>
                                                        <CheckBox fx:id="automaticCheckBox" mnemonicParsing="false" selected="true" text="Auto"/>
                                                        <Spinner fx:id="manualSpinner" editable="true" prefWidth="75.0"/>
                                                    </children>
                                                </HBox>
                                                <Label text="Find by Label" GridPane.rowIndex="2"/>
                                                <ChoiceBox fx:id="labelChoiceBox" maxWidth="200.0" prefWidth="150.0" GridPane.columnIndex="1"
                                                           GridPane.rowIndex="2"/>
                                                <Separator prefWidth="200.0" GridPane.columnSpan="2147483647" GridPane.rowIndex="3"/>
                                                <VBox alignment="CENTER" spacing="20.0" GridPane.columnIndex="2" GridPane.columnSpan="2"
                                                      GridPane.halignment="CENTER" GridPane.rowSpan="3" GridPane.valignment="CENTER">
                                                    <children>
                                                        <Button mnemonicParsing="false" onAction="#generate" prefWidth="175.0" text="Generate"/>
                                                        <Button mnemonicParsing="false" onAction="#clusterBuilder" prefWidth="175.0" text="Cluster Tools"/>
                                                    </children>
                                                    <padding>
                                                        <Insets bottom="10.0" left="10.0" right="10.0" top="10.0"/>
                                                    </padding>
                                                </VBox>
                                            </children>
                                        </GridPane>
                                    </children>
                                </HBox>
                            </top>
                            <left>
                                <VBox spacing="10.0" BorderPane.alignment="CENTER">
                                    <children>
                                        <Label text="Selected Manifold Properties"/>
                                        <TitledPane collapsible="false" text="Material" VBox.vgrow="ALWAYS">
                                            <content>
                                                <VBox spacing="5.0">
                                                    <children>
                                                        <HBox alignment="CENTER_LEFT" spacing="10.0">
                                                            <children>
                                                                <Label prefWidth="125.0" text="Diffuse Color"/>
                                                                <ColorPicker fx:id="manifoldDiffuseColorPicker" editable="true" prefHeight="50.0"
                                                                             prefWidth="150.0"/>
                                                            </children>
                                                        </HBox>
                                                        <HBox alignment="CENTER_LEFT" spacing="10.0">
                                                            <children>
                                                                <Label prefWidth="125.0" text="Wire Mesh Color"/>
                                                                <ColorPicker fx:id="manifoldWireMeshColorPicker" editable="true" prefHeight="50.0"
                                                                             prefWidth="150.0"/>
                                                            </children>
                                                        </HBox>
                                                        <HBox alignment="CENTER_LEFT" spacing="10.0">
                                                            <children>
                                                                <Label prefWidth="125.0" text="Specular Color"/>
                                                                <ColorPicker fx:id="manifoldSpecularColorPicker" editable="true" prefHeight="50.0"
                                                                             prefWidth="150.0"/>
                                                            </children>
                                                        </HBox>
                                                    </children>
                                                </VBox>
                                            </content>
                                        </TitledPane>
                                        <TitledPane collapsible="false" layoutX="10.0" layoutY="396.0" text="MeshView" VBox.vgrow="ALWAYS">
                                            <content>
                                                <GridPane>
                                                    <columnConstraints>
                                                        <ColumnConstraints hgrow="SOMETIMES" maxWidth="-Infinity" minWidth="10.0" prefWidth="300.0"/>
                                                    </columnConstraints>
                                                    <rowConstraints>
                                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES"/>
                                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES"/>
                                                        <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES"/>
                                                    </rowConstraints>
                                                    <children>
                                                        <HBox alignment="CENTER_LEFT" spacing="15.0" GridPane.halignment="CENTER" GridPane.valignment="CENTER">
                                                            <children>
                                                                <Label prefWidth="75.0" text="Cull Face"/>
                                                                <RadioButton fx:id="frontCullFaceRadioButton" mnemonicParsing="false" prefWidth="70.0"
                                                                             selected="true" text="Front"/>
                                                                <RadioButton fx:id="backCullFaceRadioButton" mnemonicParsing="false" text="Back"/>
                                                                <RadioButton fx:id="noneCullFaceRadioButton" mnemonicParsing="false" text="None"/>
                                                            </children>
                                                        </HBox>
                                                        <HBox alignment="CENTER_LEFT" spacing="15.0" GridPane.halignment="CENTER" GridPane.rowIndex="1"
                                                              GridPane.valignment="CENTER">
                                                            <children>
                                                                <Label prefWidth="75.0" text="Draw Mode"/>
                                                                <RadioButton fx:id="fillDrawModeRadioButton" mnemonicParsing="false" prefWidth="70.0"
                                                                             selected="true" text="Fill"/>
                                                                <RadioButton fx:id="linesDrawModeRadioButton" mnemonicParsing="false" text="Lines"/>
                                                            </children>
                                                        </HBox>
                                                        <HBox alignment="CENTER" spacing="10.0" GridPane.rowIndex="2">
                                                            <children>
                                                                <CheckBox fx:id="showWireframeCheckBox" mnemonicParsing="false" selected="true"
                                                                          text="Show Wire Frame"/>
                                                                <CheckBox fx:id="showControlPointsCheckBox" mnemonicParsing="false" selected="true"
                                                                          text="Show Control Points"/>
                                                            </children>
                                                        </HBox>
                                                    </children>
                                                </GridPane>
                                            </content>
                                        </TitledPane>
                                    </children>
                                    <padding>
                                        <Insets bottom="5.0" left="5.0" right="5.0" top="5.0"/>
                                    </padding>
                                </VBox>
                            </left>
                        </BorderPane>
                    </content>
                </Tab>
            </tabs>
        </TabPane>

    </children>
</AnchorPane>
```

# The FXML Controller File

``` java
/* Copyright (C) 2021 - 2023 The Johns Hopkins University Applied Physics Laboratory LLC */

/**
 * FXML Controller class
 *
 * @author Sean Phillips
 */
public class ManifoldControlController implements Initializable {
    private static final Logger LOG = LoggerFactory.getLogger(ManifoldControlController.class);

    @FXML
    private Node root;

    //Geometry Tab
    @FXML
    private ListView<ManifoldListItem> manifoldsListView;
    private Manifold3D activeManifold3D = null;
    @FXML
    private RadioButton useVisibleRadioButton;
    @FXML
    private RadioButton useAllRadioButton;
    ToggleGroup pointsToggleGroup;
    @FXML
    private ChoiceBox labelChoiceBox;
    @FXML
    private CheckBox automaticCheckBox;
    @FXML
    private Spinner manualSpinner;
    @FXML
    private ColorPicker manifoldDiffuseColorPicker;
    @FXML
    private ColorPicker manifoldSpecularColorPicker;
    @FXML
    private ColorPicker manifoldWireMeshColorPicker;
    @FXML
    private RadioButton frontCullFaceRadioButton;
    @FXML
    private RadioButton backCullFaceRadioButton;
    @FXML
    private RadioButton noneCullFaceRadioButton;
    ToggleGroup cullfaceToggleGroup;
    @FXML
    private RadioButton fillDrawModeRadioButton;
    @FXML
    private RadioButton linesDrawModeRadioButton;
    ToggleGroup drawModeToggleGroup;
    @FXML
    private CheckBox showWireframeCheckBox;
    @FXML
    private CheckBox showControlPointsCheckBox;

    //PCA Tab
    @FXML
    private RadioButton pcaRadioButton;
    @FXML
    private RadioButton svdRadioButton;
    ToggleGroup componentAnalysisToggleGroup;
    @FXML
    private Spinner numPcaComponentsSpinner;
    @FXML
    private Spinner fitStartIndexSpinner;
    @FXML
    private Spinner fitEndIndexSpinner;
    @FXML
    private CheckBox rangedFittingCheckBox;

    @FXML
    private Spinner pcaScalingSpinner;

    @FXML
    private RadioButton pcaUseHyperspaceButton;
    @FXML
    private RadioButton pcaUseHypersurfaceButton;
    ToggleGroup pcahyperSourceGroup;

    //UMAP tab
    @FXML
    private Slider targetWeightSlider;
    @FXML
    private Slider repulsionSlider;
    @FXML
    private Slider minDistanceSlider;
    @FXML
    private Slider spreadSlider;
    @FXML
    private Slider opMixSlider;
    @FXML
    private Spinner numComponentsSpinner;
    @FXML
    private Spinner numEpochsSpinner;
    @FXML
    private Spinner nearestNeighborsSpinner;
    @FXML
    private Spinner negativeSampleRateSpinner;
    @FXML
    private Spinner localConnectivitySpinner;
    @FXML
    private Spinner<Double> thresholdSpinner;
    @FXML
    private ChoiceBox metricChoiceBox;
    @FXML
    private CheckBox verboseCheckBox;
    @FXML
    private RadioButton useHyperspaceButton;
    @FXML
    private RadioButton useHypersurfaceButton;
    ToggleGroup hyperSourceGroup;


    //Distances Tab
    @FXML
    private ListView<DistanceListItem> distancesListView;
    @FXML
    private RadioButton pointToPointRadioButton;
    @FXML
    private RadioButton pointToGroupRadioButton;
    ToggleGroup pointModeToggleGroup;
    @FXML
    private TextField distanceMetricTextField;
    @FXML
    private ColorPicker connectorColorPicker;
    @FXML
    private Spinner connectorThicknessSpinner;

    Scene scene;
    private final String ALL = "ALL";
    boolean reactive = true;
    Umap latestUmapObject = null;
    File latestDir = new File(".");

    /**
     * Initializes the controller class.
     *
     * @param url
     * @param rb
     */
    @Override
    public void initialize(URL url, ResourceBundle rb) {
        scene = App.getAppScene();
        setupPcaControls();
        setupHullControls();
        setupUmapControls();
        setupDistanceControls();
        if (null != root) {
            root.addEventHandler(DragEvent.DRAG_OVER, event -> {
                event.acceptTransferModes(TransferMode.COPY);
            });
            root.addEventHandler(DragEvent.DRAG_DROPPED, e -> {
                Dragboard db = e.getDragboard();
                if (db.hasFiles()) {
                    File file = db.getFiles().get(0);
                    try {

                        if (UmapConfig.isUmapConfig(Files.readString(file.toPath()))) {
                            ObjectMapper mapper = new ObjectMapper();
                            UmapConfig uc = mapper.readValue(file, UmapConfig.class);
                            setUmapConfig(uc);
                            sendUmapConfig();
                        }
                    } catch (IOException ex) {
                        LOG.error(null, ex);
                    }
                }
            });
            scene.addEventHandler(ManifoldEvent.NEW_UMAP_CONFIG, event -> {
                UmapConfig config = (UmapConfig) event.object1;
                setUmapConfig(config);
            });
        }
    }

    private void setupDistanceControls() {
        distanceMetricTextField.setText("Select Distance Object");
        distanceMetricTextField.setEditable(false);
        connectorThicknessSpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(1, 50, 5, 1));
        connectorThicknessSpinner.setEditable(true);
        connectorThicknessSpinner.valueProperty().addListener(e -> {
            DistanceListItem item = distancesListView.getSelectionModel().getSelectedItem();
            if (null != item) {
                Integer width = (Integer) connectorThicknessSpinner.getValue();
                item.getDistance().setWidth(width);
                scene.getRoot().fireEvent(
                    new ManifoldEvent(ManifoldEvent.DISTANCE_CONNECTOR_WIDTH, item.getDistance()));
            }
        });
        connectorThicknessSpinner.setInitialDelay(Duration.millis(500));
        connectorThicknessSpinner.setRepeatDelay(Duration.millis(500));

        connectorColorPicker.valueProperty().addListener(cl -> {
            DistanceListItem item = distancesListView.getSelectionModel().getSelectedItem();
            if (null != item) {
                item.getDistance().setColor(connectorColorPicker.getValue());
                scene.getRoot().fireEvent(
                    new ManifoldEvent(ManifoldEvent.DISTANCE_CONNECTOR_COLOR, item.getDistance()));
            }
        });
        //Get a reference to any Distances already collected
        List<DistanceListItem> existingItems = new ArrayList<>();
        for (Distance d : Distance.getDistances()) {
            DistanceListItem item = new DistanceListItem(d);
            existingItems.add(item);
        }
        //add them all in one shot
        distancesListView.getItems().addAll(existingItems);
        ImageView iv = ResourceUtils.loadIcon("metric", 200);
        VBox placeholder = new VBox(10, iv, new Label("No Distances Acquired"));
        placeholder.setAlignment(Pos.CENTER);
        distancesListView.setPlaceholder(placeholder);

        //Bind disable properties so that controls only active when item is selected
        distanceMetricTextField.disableProperty().bind(
            distancesListView.getSelectionModel().selectedIndexProperty().lessThan(0));
        connectorThicknessSpinner.disableProperty().bind(
            distancesListView.getSelectionModel().selectedIndexProperty().lessThan(0));
        connectorColorPicker.disableProperty().bind(
            distancesListView.getSelectionModel().selectedIndexProperty().lessThan(0));

        pointModeToggleGroup = new ToggleGroup();
        pointToPointRadioButton.setToggleGroup(pointModeToggleGroup);
        pointToGroupRadioButton.setToggleGroup(pointModeToggleGroup);
        scene.addEventHandler(ManifoldEvent.DISTANCE_CONNECTOR_SELECTED, e -> {
            Distance distance = (Distance) e.object1;
            for (DistanceListItem item : distancesListView.getItems()) {
                if (item.getDistance() == distance) {
                    distancesListView.getSelectionModel().select(item);
                    distanceMetricTextField.setText(distance.getMetric());
                    connectorColorPicker.setValue(distance.getColor());
                    connectorThicknessSpinner.getValueFactory().setValue(distance.getWidth());
                    return; //break out early
                }
            }
            //if we get here its because that Distance object wasn't in the list
            scene.getRoot().fireEvent(new CommandTerminalEvent(
                "Distance object not found!", new Font("Consolas", 20), Color.YELLOW));
        });
        scene.addEventHandler(ManifoldEvent.DISTANCE_OBJECT_SELECTED, e -> {
            Distance distance = (Distance) e.object1;
            distanceMetricTextField.setText(distance.getMetric());
            connectorColorPicker.setValue(distance.getColor());
            connectorThicknessSpinner.getValueFactory().setValue(distance.getWidth());
        });
        scene.addEventHandler(ManifoldEvent.CREATE_NEW_DISTANCE, e -> {
            Distance distance = (Distance) e.object1;
            DistanceListItem distanceListItem = new DistanceListItem(distance);
            Distance.addDistance(distance);
            distancesListView.getItems().add(distanceListItem);
        });
    }

    private void getCurrentLabels() {
        labelChoiceBox.getItems().clear();
        labelChoiceBox.getItems().add(ALL);
        labelChoiceBox.getItems().addAll(
            FactorLabel.getFactorLabels().stream()
                .map(f -> f.getLabel()).sorted().toList());
    }

    private void setupPcaControls() {
        numPcaComponentsSpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(2, 100, 3, 1));
        fitStartIndexSpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(0, 500, 0, 5));
        fitEndIndexSpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(5, 2000, 50, 5));
        //"min","max","initialValue","amountToStepBy"
        fitStartIndexSpinner.disableProperty().bind(rangedFittingCheckBox.selectedProperty().not());
        fitEndIndexSpinner.disableProperty().bind(rangedFittingCheckBox.selectedProperty().not());

        pcaScalingSpinner.setValueFactory(
            new SpinnerValueFactory.DoubleSpinnerValueFactory(1, 1000, 100, 10));

        componentAnalysisToggleGroup = new ToggleGroup();
        pcaRadioButton.setToggleGroup(componentAnalysisToggleGroup);
        svdRadioButton.setToggleGroup(componentAnalysisToggleGroup);

        pcahyperSourceGroup = new ToggleGroup();
        pcaUseHyperspaceButton.setToggleGroup(pcahyperSourceGroup);
        pcaUseHypersurfaceButton.setToggleGroup(pcahyperSourceGroup);
    }

    private void setupUmapControls() {
        hyperSourceGroup = new ToggleGroup();
        useHyperspaceButton.setToggleGroup(hyperSourceGroup);
        useHypersurfaceButton.setToggleGroup(hyperSourceGroup);

        metricChoiceBox.getItems().addAll(Metric.getMetricNames());
        metricChoiceBox.getSelectionModel().selectFirst();

        numComponentsSpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(2, 50, 3, 1));
        numEpochsSpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(25, 500, 200, 25));
        nearestNeighborsSpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(5, 500, 15, 5));
        negativeSampleRateSpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(1, 250, 5, 1));
        localConnectivitySpinner.setValueFactory(
            new SpinnerValueFactory.IntegerSpinnerValueFactory(1, 250, 1, 1));
//        thresholdSpinner.setValueFactory(
//            new SpinnerValueFactory.DoubleSpinnerValueFactory(0.001, 1.0, 0.001, 0.001));
//        thresholdSpinner.setEditable(true);

        hyperSourceGroup = new ToggleGroup();
        useHyperspaceButton.setToggleGroup(hyperSourceGroup);
        useHypersurfaceButton.setToggleGroup(hyperSourceGroup);
    }

    private void setupHullControls() {
        //Get a reference to any Distances already collected
        List<ManifoldListItem> existingItems = new ArrayList<>();
        for (Manifold m : Manifold.getManifolds()) {
            ManifoldListItem item = new ManifoldListItem(m);
            existingItems.add(item);
        }
        //add them all in one shot
        manifoldsListView.getItems().addAll(existingItems);
        ImageView iv = ResourceUtils.loadIcon("manifold", 200);
        VBox placeholder = new VBox(10, iv, new Label("No Manifolds Acquired"));
        placeholder.setAlignment(Pos.CENTER);
        manifoldsListView.setPlaceholder(placeholder);

        getCurrentLabels();
        labelChoiceBox.getSelectionModel().selectFirst();
        labelChoiceBox.setOnShown(e -> getCurrentLabels());
        manualSpinner.setValueFactory(
            new SpinnerValueFactory.DoubleSpinnerValueFactory(0.1, 1, 0.5, 0.1));
        manualSpinner.setEditable(true);
        //whenever the spinner value is changed...
        manualSpinner.valueProperty().addListener(e -> {
            scene.getRoot().fireEvent(
                new ManifoldEvent(ManifoldEvent.SET_DISTANCE_TOLERANCE,
                    (Double) manualSpinner.getValue()));
        });
        manualSpinner.disableProperty().bind(automaticCheckBox.selectedProperty());

        manifoldDiffuseColorPicker.setValue(Color.CYAN);
        manifoldDiffuseColorPicker.valueProperty().addListener(cl -> {
            if (!reactive) return;
            ManifoldListItem item = manifoldsListView.getSelectionModel().getSelectedItem();
            if (null != item) {
                Manifold m = item.getManifold();
                if (null != m) {
                    m.setColor(manifoldDiffuseColorPicker.getValue());
                    scene.getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_DIFFUSE_COLOR,
                        manifoldDiffuseColorPicker.getValue(), m));
                }
            }
        });
        manifoldSpecularColorPicker.setValue(Color.RED);
        manifoldSpecularColorPicker.valueProperty().addListener(cl -> {
            if (!reactive) return;
            ManifoldListItem item = manifoldsListView.getSelectionModel().getSelectedItem();
            if (null != item) {
                Manifold m = item.getManifold();
                if (null != m)
                    scene.getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_SPECULAR_COLOR,
                        manifoldSpecularColorPicker.getValue(), m));
            }
        });
        manifoldWireMeshColorPicker.setValue(Color.BLUE);
        manifoldWireMeshColorPicker.valueProperty().addListener(cl -> {
            if (!reactive) return;
            ManifoldListItem item = manifoldsListView.getSelectionModel().getSelectedItem();
            if (null != item) {
                Manifold m = item.getManifold();
                if (null != m)
                    scene.getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_WIREFRAME_COLOR,
                        manifoldWireMeshColorPicker.getValue(), m));
            }
        });

        pointsToggleGroup = new ToggleGroup();
        useVisibleRadioButton.setToggleGroup(pointsToggleGroup);
        useAllRadioButton.setToggleGroup(pointsToggleGroup);
        pointsToggleGroup.selectedToggleProperty().addListener(cl -> {
            if (useVisibleRadioButton.isSelected())
                scene.getRoot().fireEvent(new ManifoldEvent(
                    ManifoldEvent.USE_VISIBLE_POINTS, true));
            else
                scene.getRoot().fireEvent(new ManifoldEvent(
                    ManifoldEvent.USE_ALL_POINTS, true));
        });

        cullfaceToggleGroup = new ToggleGroup();
        frontCullFaceRadioButton.setToggleGroup(cullfaceToggleGroup);
        backCullFaceRadioButton.setToggleGroup(cullfaceToggleGroup);
        noneCullFaceRadioButton.setToggleGroup(cullfaceToggleGroup);
        cullfaceToggleGroup.selectedToggleProperty().addListener(cl -> {
            Manifold m = manifoldsListView.getSelectionModel().getSelectedItem().getManifold();
            if (null != m)
                if (frontCullFaceRadioButton.isSelected())
                    scene.getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_FRONT_CULLFACE, true, m));
                else if (backCullFaceRadioButton.isSelected())
                    scene.getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_BACK_CULLFACE, true, m));
                else
                    scene.getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_NONE_CULLFACE, true, m));
        });
        drawModeToggleGroup = new ToggleGroup();
        fillDrawModeRadioButton.setToggleGroup(drawModeToggleGroup);
        linesDrawModeRadioButton.setToggleGroup(drawModeToggleGroup);
        drawModeToggleGroup.selectedToggleProperty().addListener(cl -> {
            Manifold m = manifoldsListView.getSelectionModel().getSelectedItem().getManifold();
            if (null != m)
                if (fillDrawModeRadioButton.isSelected())
                    scene.getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_FILL_DRAWMODE, true, m));
                else
                    scene.getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_LINE_DRAWMODE, true, m));
        });

        showWireframeCheckBox.selectedProperty().addListener(cl -> {
            Manifold m = manifoldsListView.getSelectionModel().getSelectedItem().getManifold();
            if (null != m)
                scene.getRoot().fireEvent(new ManifoldEvent(
                    ManifoldEvent.MANIFOLD_SHOW_WIREFRAME,
                    showWireframeCheckBox.isSelected(), m));
        });
        showControlPointsCheckBox.selectedProperty().addListener(cl -> {
            Manifold m = manifoldsListView.getSelectionModel().getSelectedItem().getManifold();
            if (null != m)
                scene.getRoot().fireEvent(new ManifoldEvent(
                    ManifoldEvent.MANIFOLD_SHOW_CONTROL,
                    showControlPointsCheckBox.isSelected(), m));
        });

        scene.addEventHandler(ManifoldEvent.MANIFOLD_3D_SELECTED, e -> {
            Manifold manifold = (Manifold) e.object1;
            for (ManifoldListItem item : manifoldsListView.getItems()) {
                if (item.getManifold() == manifold) {
                    manifoldsListView.getSelectionModel().select(item);
                    Manifold3D manifold3D = Manifold.globalManifoldToManifold3DMap.get(manifold);
                    updateActiveManifold3D(manifold3D);
                    return; //break out early
                }
            }
            //if we get here its because that Distance object wasn't in the list
            scene.getRoot().fireEvent(new CommandTerminalEvent(
                "Manifold object not found!", new Font("Consolas", 20), Color.YELLOW));
        });
        scene.addEventHandler(ManifoldEvent.MANIFOLD_OBJECT_SELECTED, e -> {
            Manifold manifold = (Manifold) e.object1;
            Manifold3D manifold3D = Manifold.globalManifoldToManifold3DMap.get(manifold);
            updateActiveManifold3D(manifold3D);
        });
        scene.addEventHandler(ManifoldEvent.MANIFOLD3D_OBJECT_GENERATED, e -> {
            Manifold manifold = (Manifold) e.object1;
            updateActiveManifold3D((Manifold3D) e.object2);
            ManifoldListItem manifoldListItem = new ManifoldListItem(manifold);
            manifoldsListView.getItems().add(manifoldListItem);
            manifoldsListView.getSelectionModel().selectLast();
        });
    }

    private void updateActiveManifold3D(Manifold3D manifold3D) {
        reactive = false;
        activeManifold3D = manifold3D;
        if (null != manifold3D) {
            PhongMaterial phong = (PhongMaterial) manifold3D.quickhullMeshView.getMaterial();
            manifoldDiffuseColorPicker.setValue(phong.getDiffuseColor());
            manifoldSpecularColorPicker.setValue(phong.getSpecularColor());
            manifoldWireMeshColorPicker.setValue(((PhongMaterial)
                manifold3D.quickhullLinesMeshView.getMaterial()).getDiffuseColor());
        }
        reactive = true;
    }

    @FXML
    public void exportMatrix() {
        if (null != latestUmapObject) {
            if (null != latestUmapObject.getmEmbedding()) {
                LOG.info("latestUmapObject: {}", latestUmapObject.getmEmbedding().toStringNumpy());
            } else {
                LOG.info("UMAP Embeddings not generated yet.");
            }
        } else {
            LOG.info("UMAP object not yet established.");
        }
    }

    @FXML
    public void loadUmapConfig() {
        FileChooser fc = new FileChooser();
        fc.setTitle("Choose UMAP Config to load...");
        if (!latestDir.isDirectory())
            latestDir = new File(".");
        fc.setInitialDirectory(latestDir);
        File file = fc.showOpenDialog(scene.getWindow());
        if (null != file) {
            if (file.getParentFile().isDirectory())
                latestDir = file;
            ObjectMapper mapper = new ObjectMapper();
            UmapConfig uc;
            try {
                uc = mapper.readValue(file, UmapConfig.class);
                setUmapConfig(uc);
            } catch (IOException ex) {
                LOG.error(null, ex);
            }
        }
    }

    public void setUmapConfig(UmapConfig uc) {
        if (null != uc.getTargetWeight())
            targetWeightSlider.setValue(uc.getTargetWeight());
        repulsionSlider.setValue(uc.getRepulsionStrength());
        minDistanceSlider.setValue(uc.getMinDist());
        spreadSlider.setValue(uc.getSpread());
        opMixSlider.setValue(uc.getOpMixRatio());
        numComponentsSpinner.getValueFactory().setValue(uc.getNumberComponents());
        numEpochsSpinner.getValueFactory().setValue(uc.getNumberEpochs());
        nearestNeighborsSpinner.getValueFactory().setValue(uc.getNumberNearestNeighbours());
        negativeSampleRateSpinner.getValueFactory().setValue(uc.getNegativeSampleRate());
        localConnectivitySpinner.getValueFactory().setValue(uc.getLocalConnectivity());
        if (null != uc.getThreshold())
            thresholdSpinner.getValueFactory().setValue(uc.getThreshold());
        metricChoiceBox.getSelectionModel().select(uc.getMetric());
        verboseCheckBox.setSelected(uc.getVerbose());

    }

    @FXML
    public void saveUmapConfig() {
        FileChooser fc = new FileChooser();
        fc.setTitle("Choose UMAP Config file output...");
        fc.setInitialFileName(UmapConfig.configToFilename(getCurrentUmapConfig()).concat(".json"));
        if (!latestDir.isDirectory())
            latestDir = new File(".");
        fc.setInitialDirectory(latestDir);
        File file = fc.showSaveDialog(scene.getWindow());
        if (null != file) {
            if (file.getParentFile().isDirectory())
                latestDir = file;
            writeConfigFile(file);
        }
    }

    public UmapConfig getCurrentUmapConfig() {
        UmapConfig uc = new UmapConfig();
        uc.setTargetWeight((float) targetWeightSlider.getValue());
        uc.setRepulsionStrength((float) repulsionSlider.getValue());
        uc.setMinDist((float) minDistanceSlider.getValue());
        uc.setSpread((float) spreadSlider.getValue());
        uc.setOpMixRatio((float) opMixSlider.getValue());
        uc.setNumberComponents((int) numComponentsSpinner.getValue());
        uc.setNumberEpochs((int) numEpochsSpinner.getValue());
        uc.setNumberNearestNeighbours((int) nearestNeighborsSpinner.getValue());
        uc.setNegativeSampleRate((int) negativeSampleRateSpinner.getValue());
        uc.setLocalConnectivity((int) localConnectivitySpinner.getValue());
        uc.setThreshold((double) thresholdSpinner.getValue());
        uc.setMetric((String) metricChoiceBox.getValue());
        uc.setVerbose(verboseCheckBox.isSelected());
        return uc;
    }

    private void writeConfigFile(File file) {
        UmapConfig uc = getCurrentUmapConfig();
        ObjectMapper mapper = new ObjectMapper();
        try {
            mapper.writeValue(file, uc);
        } catch (IOException ex) {
            LOG.error(null, ex);
        }
    }
//    private String configToFilename(){
//        NumberFormat format = new DecimalFormat("0.00");
//        StringBuilder sb = new StringBuilder("UmapConfig-");
////        sb.append(targetWeightSlider.getValue()).append("-");
//        sb.append((String) metricChoiceBox.getValue()).append("-");
//        sb.append("R").append(format.format(repulsionSlider.getValue())).append("-");
//        sb.append("MD").append(format.format(minDistanceSlider.getValue())).append("-");
//        sb.append("S").append(format.format(spreadSlider.getValue())).append("-");
//        sb.append("OPM").append(format.format(opMixSlider.getValue())).append("-");
////        uc.setNumberComponents((int) numComponentsSpinner.getValue());
////        uc.setNumberEpochs((int) numEpochsSpinner.getValue());
//        sb.append("NN").append(nearestNeighborsSpinner.getValue()).append("-");
//        sb.append("NSR").append(negativeSampleRateSpinner.getValue()).append("-");
//        sb.append("LC").append(localConnectivitySpinner.getValue());

    /// /        uc.setThreshold((double) thresholdSpinner.getValue());
    /// /        uc.setVerbose(verboseCheckBox.isSelected());
//        return sb.toString();
//    }
    private void sendUmapConfig() {
        String name = latestDir.getAbsolutePath() + File.separator
            + UmapConfig.configToFilename(getCurrentUmapConfig()).concat(".json");
        scene.getRoot().fireEvent(new ManifoldEvent(
            ManifoldEvent.NEW_UMAP_CONFIG, getCurrentUmapConfig(), name));


    }

    @FXML
    public void exportScene() {
        FileChooser fc = new FileChooser();
        fc.setTitle("Choose Location file output...");
        fc.setInitialFileName(UmapConfig.configToFilename(getCurrentUmapConfig()).concat(".json"));
        if (!latestDir.isDirectory())
            latestDir = new File(".");
        fc.setInitialDirectory(latestDir);
        File file = fc.showSaveDialog(scene.getWindow());
        if (null != file) {
            if (file.getParentFile().isDirectory())
                latestDir = file.getParentFile();
            writeConfigFile(file);
            scene.getRoot().fireEvent(new ManifoldEvent(
                ManifoldEvent.EXPORT_PROJECTION_SCENE, file));
        }
    }

    @FXML
    public void saveProjections() {
        FileChooser fc = new FileChooser();
        fc.setTitle("Choose Projection file output...");
        fc.setInitialFileName("ProjectionData.json");
        if (!latestDir.isDirectory())
            latestDir = new File(".");
        fc.setInitialDirectory(latestDir);
        File file = fc.showSaveDialog(scene.getWindow());
        if (null != file) {
            if (file.getParentFile().isDirectory())
                latestDir = file;
            scene.getRoot().fireEvent(new ManifoldEvent(
                ManifoldEvent.SAVE_PROJECTION_DATA, file));
        }
    }

    @FXML
    public void runPCA() {
        AnalysisUtils.SOURCE source = useHypersurfaceButton.isSelected() ?
            AnalysisUtils.SOURCE.HYPERSURFACE : AnalysisUtils.SOURCE.HYPERSPACE;
        AnalysisUtils.ANALYSIS_METHOD method = svdRadioButton.isSelected() ?
            AnalysisUtils.ANALYSIS_METHOD.SVD : AnalysisUtils.ANALYSIS_METHOD.PCA;

        int startIndex = 0;
        int endIndex = -1; //use max indicator
        if (rangedFittingCheckBox.isSelected()) {
            startIndex = (int) fitStartIndexSpinner.getValue();
            endIndex = (int) fitEndIndexSpinner.getValue();
            if (endIndex <= startIndex)
                endIndex = -1;
        }

        PCAConfig config = new PCAConfig(source, method,
            (int) numPcaComponentsSpinner.getValue(), (double) pcaScalingSpinner.getValue(),
            startIndex, endIndex);
        ManifoldEvent.POINT_SOURCE pointSource = useHypersurfaceButton.isSelected() ?
            ManifoldEvent.POINT_SOURCE.HYPERSURFACE : ManifoldEvent.POINT_SOURCE.HYPERSPACE;
        scene.getRoot().fireEvent(new ManifoldEvent(
            ManifoldEvent.GENERATE_NEW_PCA, config, pointSource));
    }

    @FXML
    public void project() {
        Umap umap = new Umap();
        umap.setTargetWeight((float) targetWeightSlider.getValue());
        umap.setRepulsionStrength((float) repulsionSlider.getValue());
        umap.setMinDist((float) minDistanceSlider.getValue());
        umap.setSpread((float) spreadSlider.getValue());
        umap.setSetOpMixRatio((float) opMixSlider.getValue());
        umap.setNumberComponents((int) numComponentsSpinner.getValue());
        umap.setNumberEpochs((int) numEpochsSpinner.getValue());
        umap.setNumberNearestNeighbours((int) nearestNeighborsSpinner.getValue());
        umap.setNegativeSampleRate((int) negativeSampleRateSpinner.getValue());
        umap.setLocalConnectivity((int) localConnectivitySpinner.getValue());
        umap.setThreshold((double) thresholdSpinner.getValue());
        umap.setMetric((String) metricChoiceBox.getValue());
        umap.setVerbose(verboseCheckBox.isSelected());
        ManifoldEvent.POINT_SOURCE pointSource = useHypersurfaceButton.isSelected() ?
            ManifoldEvent.POINT_SOURCE.HYPERSURFACE : ManifoldEvent.POINT_SOURCE.HYPERSPACE;
        latestUmapObject = umap;
        scene.getRoot().fireEvent(new ManifoldEvent(
            ManifoldEvent.GENERATE_NEW_UMAP, umap, pointSource));
    }

    @FXML
    public void generate() {
        Double tolerance = null;
        if (!automaticCheckBox.isSelected())
            tolerance = (Double) manualSpinner.getValue();
        scene.getRoot().fireEvent(new ManifoldEvent(
            ManifoldEvent.GENERATE_PROJECTION_MANIFOLD,
            useVisibleRadioButton.isSelected(),
            (String) labelChoiceBox.getValue(), tolerance
        ));
    }

    @FXML
    public void clearAll() {
        scene.getRoot().fireEvent(new ManifoldEvent(
            ManifoldEvent.CLEAR_ALL_MANIFOLDS));
        //add them all in one shot
        manifoldsListView.getItems().clear();
    }

    @FXML
    public void exportAll() {
        DirectoryChooser fc = new DirectoryChooser();
        fc.setTitle("Choose Directory to export ManifoldData files...");
        if (!latestDir.isDirectory())
            latestDir = new File(".");
        fc.setInitialDirectory(latestDir);
        File file = fc.showDialog(scene.getWindow());
        if (null != file) {
            if (file.getParentFile().isDirectory()) {
                latestDir = file;
                int sequence = 0;
                for (ManifoldListItem m : manifoldsListView.getItems()) {
                    Manifold3D m3D = Manifold.globalManifoldToManifold3DMap.get(m.getManifold());
                    if (null != m3D) {
                        String fileName = "ManifoldData_" + sequence + "_" + m.getManifold().getLabel() + ".json";
                        File exportFile = new File(latestDir.getPath() + fileName);
                        scene.getRoot().fireEvent(new ManifoldEvent(
                            ManifoldEvent.EXPORT_MANIFOLD_DATA, exportFile, m3D));
                    }
                    sequence++;
                }
            }
        }
    }

    @FXML
    public void clusterBuilder() {
        //fire event to put projection view into cluster selection mode
        scene.getRoot().fireEvent(new ApplicationEvent(
            ApplicationEvent.SHOW_SHAPE3D_CONTROLS));
    }

    @FXML
    public void startConnector() {
        //fire event to put projection view into connector mode
        //@TODO SMP Hardcoded for now to Euclidean
        if (pointToGroupRadioButton.isSelected())
            scene.getRoot().fireEvent(new ManifoldEvent(
                ManifoldEvent.DISTANCE_MODE_POINTGROUP, pointToGroupRadioButton.isSelected(), "euclidean"));
        else
            scene.getRoot().fireEvent(new ManifoldEvent(
                ManifoldEvent.DISTANCE_MODE_POINTPOINT, pointToPointRadioButton.isSelected(), "euclidean"));
    }

    @FXML
    public void clearAllDistances() {
        distancesListView.getItems().clear();
        Distance.removeAllDistances(); //will fire event notifying scene
        scene.getRoot().fireEvent(
            new ManifoldEvent(ManifoldEvent.CLEAR_DISTANCE_CONNECTORS));
    }
}
```

# Secondary Files

There are a number of other files that you'll need to look at to understand the code in the FXML Controller.  Here they are:

## ManifoldListItem

``` java
/* Copyright (C) 2021 - 2024 Sean Phillips */

package edu.jhuapl.trinity.javafx.components.listviews;


import edu.jhuapl.trinity.data.Manifold;
import edu.jhuapl.trinity.javafx.events.ManifoldEvent;
import edu.jhuapl.trinity.javafx.javafx3d.Manifold3D;
import edu.jhuapl.trinity.utils.ResourceUtils;
import javafx.scene.control.CheckBox;
import javafx.scene.control.TextField;
import javafx.scene.image.ImageView;
import javafx.scene.input.MouseEvent;
import javafx.scene.layout.HBox;
import javafx.scene.layout.VBox;

import static edu.jhuapl.trinity.data.Manifold.globalManifoldToManifold3DMap;

/**
 * @author Sean Phillips
 */
public class ManifoldListItem extends VBox {

    private String labelString;
    private CheckBox visibleCheckBox;
    private TextField manifoldLabelTextField;
    private Manifold manifold;
    public boolean reactive = true;

    public ManifoldListItem(Manifold manifold) {
        this.manifold = manifold;
        labelString = manifold.getLabel();
        manifoldLabelTextField = new TextField(labelString);
        manifoldLabelTextField.setPrefWidth(200);
        visibleCheckBox = new CheckBox("Show");
        visibleCheckBox.setSelected(true);

        ImageView manifoldIcon = ResourceUtils.loadIcon("manifold", 32);

        HBox topHBox = new HBox(5, manifoldIcon, visibleCheckBox, manifoldLabelTextField);

        getChildren().addAll(topHBox);//, bottomHBox);
        setSpacing(2);
        visibleCheckBox.selectedProperty().addListener(cl -> {
            if (null != visibleCheckBox.getScene()) {
                this.manifold.setVisible(visibleCheckBox.isSelected());
                if (reactive) {
                    Manifold.updateManifold(this.manifold.getLabel(), this.manifold);
                    Manifold3D m3D = globalManifoldToManifold3DMap.get(this.manifold);
                    m3D.setVisible(manifold.getVisible());
                    getScene().getRoot().fireEvent(new ManifoldEvent(
                        ManifoldEvent.MANIFOLD_3D_VISIBLE, manifold.getVisible()));
                }
            }
        });
        manifoldIcon.addEventHandler(MouseEvent.MOUSE_CLICKED, e -> {
            if (e.getClickCount() > 1) {
                //Let application know this distance object has been selected
                getScene().getRoot().fireEvent(new ManifoldEvent(
                    ManifoldEvent.MANIFOLD_OBJECT_SELECTED, this.manifold));
                getScene().getRoot().fireEvent(
                    new ManifoldEvent(ManifoldEvent.MANIFOLD_3D_SELECTED, this.manifold));
            }
        });
        setOnMouseClicked(e -> {
            //Let application know this distance object has been selected
            getScene().getRoot().fireEvent(new ManifoldEvent(
                ManifoldEvent.MANIFOLD_OBJECT_SELECTED, this.getManifold()));
        });
        manifoldLabelTextField.textProperty().addListener(e -> {
            labelString = manifoldLabelTextField.getText();
            this.manifold.setLabel(labelString);
        });
    }

    public boolean getDataVisible() {
        return visibleCheckBox.isSelected();
    }

    public void setDataVisible(boolean visible) {
        visibleCheckBox.setSelected(visible);
    }

    /**
     * @return the manifold
     */
    public Manifold getManifold() {
        return manifold;
    }

    /**
     * @param manifold the manifold to set
     */
    public void setManifold(Manifold manifold) {
        this.manifold = manifold;
    }

}
```


## Manifold

``` java
/* Copyright (C) 2021 - 2024 Sean Phillips */

package edu.jhuapl.trinity.data;

import edu.jhuapl.trinity.javafx.javafx3d.Manifold3D;
import javafx.beans.property.SimpleBooleanProperty;
import javafx.geometry.Point3D;
import javafx.scene.paint.Color;

import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.List;

/**
 * @author Sean Phillips
 */
public class Manifold {
    private ArrayList<Point3D> points;
    private String label; //common data label of manifold (from input data)
    private Color color; //color of representative object
    public SimpleBooleanProperty visible;

    public Manifold(ArrayList<Point3D> points, String label, String name, Color color) {
        this.points = new ArrayList<>(points.size());
        this.points.addAll(points);
        this.label = label;
        this.color = color;
        visible = new SimpleBooleanProperty(true);
    }

    /**
     * Provides lookup mechanism to find any object model that is currently
     * anchored in the system.
     */
    private static HashMap<String, Manifold> globalManifoldMap = new HashMap<>();
    public static HashMap<Manifold, Manifold3D> globalManifoldToManifold3DMap = new HashMap<>();

    public static Collection<Manifold> getManifolds() {
        return globalManifoldMap.values();
    }

    public static Manifold getManifold(String label) {
        return globalManifoldMap.get(label);
    }

    public static void addManifold(Manifold manifold) {
        globalManifoldMap.put(manifold.getLabel(), manifold);
    }

    public static void addAllManifolds(List<Manifold> manifolds) {
        manifolds.forEach(d -> {
            globalManifoldMap.put(d.getLabel(), d);
        });
    }

    public static void removeAllManifolds() {
        globalManifoldMap.clear();
        globalManifoldToManifold3DMap.clear();
    }

    public static Manifold removeManifold(String label) {
        Manifold removed = globalManifoldMap.remove(label);
        return removed;
    }

    public static void updateManifold(String label, Manifold manifold) {
        globalManifoldMap.put(label, manifold);
    }

    public static Color getColorByLabel(String label) {
        Manifold fl = Manifold.getManifold(label);
        if (null == fl)
            return Color.ALICEBLUE;
        return fl.getColor();
    }

    public static boolean visibilityByLabel(String label) {
        Manifold fl = Manifold.getManifold(label);
        if (null == fl)
            return true;
        return fl.getVisible();
    }

    public static void setAllVisible(boolean visible) {
        globalManifoldMap.forEach((s, fl) -> {
            fl.setVisible(visible);
        });
    }

    /**
     * @return the label
     */
    public String getLabel() {
        return label;
    }

    /**
     * @param label the label to set
     */
    public void setLabel(String label) {
        this.label = label;
    }

    /**
     * @return the color
     */
    public Color getColor() {
        return color;
    }

    /**
     * @param color the color to set
     */
    public void setColor(Color color) {
        this.color = color;
    }

    public SimpleBooleanProperty visibleProperty() {
        return this.visible;
    }

    public java.lang.Boolean getVisible() {
        return this.visibleProperty().get();
    }

    public void setVisible(final java.lang.Boolean visible) {
        this.visibleProperty().set(visible);
    }

    /**
     * @return the points
     */
    public ArrayList<Point3D> getPoints() {
        return points;
    }

    /**
     * @param points the points to set
     */
    public void setPoints(ArrayList<Point3D> points) {
        this.points = points;
    }
}
```



## DistanceListItem

``` java
/* Copyright (C) 2021 - 2024 Sean Phillips */

package edu.jhuapl.trinity.javafx.components.listviews;

import edu.jhuapl.trinity.data.Distance;
import edu.jhuapl.trinity.javafx.events.ManifoldEvent;
import edu.jhuapl.trinity.utils.PrecisionConverter;
import javafx.scene.control.CheckBox;
import javafx.scene.control.Label;
import javafx.scene.layout.HBox;

/**
 * @author Sean Phillips
 */
public class DistanceListItem extends HBox {
    private String labelString;
    private CheckBox visibleCheckBox;
    private Label label;
    private Label distanceValueLabel;
    private Distance distance;
    public boolean reactive = true;

    public DistanceListItem(Distance distance) {
        this.distance = distance;
        this.labelString = distance.getLabel();
        visibleCheckBox = new CheckBox("Visible");
        visibleCheckBox.setSelected(true);
        label = new Label(labelString);
        PrecisionConverter pc = new PrecisionConverter(7);
        String distanceLabel = distance.getMetric() + ":" + pc.toString(distance.getValue());
        distanceValueLabel = new Label(distanceLabel);
        getChildren().addAll(visibleCheckBox, label, distanceValueLabel);
        setSpacing(5);
        visibleCheckBox.selectedProperty().addListener(cl -> {
            if (null != visibleCheckBox.getScene()) {
                distance.setVisible(visibleCheckBox.isSelected());
                if (reactive)
                    Distance.updateDistance(distance.getLabel(), distance);
            }
        });
        setOnMouseClicked(e -> {
            //Let application know this distance object has been selected
            getScene().getRoot().fireEvent(new ManifoldEvent(
                ManifoldEvent.DISTANCE_OBJECT_SELECTED, distance));
        });
    }

    public boolean getDataVisible() {
        return visibleCheckBox.isSelected();
    }

    public void setDataVisible(boolean visible) {
        visibleCheckBox.setSelected(visible);
    }

    /**
     * @return the distance
     */
    public Distance getDistance() {
        return distance;
    }

    /**
     * @param distance the distance to set
     */
    public void setDistance(Distance distance) {
        this.distance = distance;
    }
}
```


## Distance

``` java
/* Copyright (C) 2021 - 2024 Sean Phillips */

package edu.jhuapl.trinity.data;

import javafx.beans.property.SimpleBooleanProperty;
import javafx.geometry.Point3D;
import javafx.scene.paint.Color;

import java.util.Collection;
import java.util.HashMap;
import java.util.List;

/**
 * @author Sean Phillips
 */
public class Distance {
    private String metric; //Used to lookup into Metric enum later
    private double value; //the computed value
    private int width; //width of the connector
    private Point3D point1;
    private Point3D point2;
    private String label;
    private Color color;
    public SimpleBooleanProperty visible;

    public Distance(String label, Color color, String metric, Integer width) {
        this.label = label;
        this.color = color;
        this.width = width;
        if (null == metric)
            metric = "euclidean";
        else
            this.metric = metric;
        visible = new SimpleBooleanProperty(true);
    }

    /**
     * Provides lookup mechanism to find any object model that is currently
     * anchored in the system.
     */
    private static HashMap<String, Distance> globalDistanceMap = new HashMap<>();

    public static Collection<Distance> getDistances() {
        return globalDistanceMap.values();
    }

    public static Distance getDistance(String label) {
        return globalDistanceMap.get(label);
    }

    public static void addDistance(Distance distance) {
        globalDistanceMap.put(distance.getLabel(), distance);
    }

    public static void addAllDistances(List<Distance> distances) {
        distances.forEach(d -> {
            globalDistanceMap.put(d.getLabel(), d);
        });
    }

    public static void removeAllDistances() {
        globalDistanceMap.clear();
    }

    public static Distance removeDistance(String label) {
        Distance removed = globalDistanceMap.remove(label);
        return removed;
    }

    public static void updateDistance(String label, Distance distance) {
        globalDistanceMap.put(label, distance);
    }

    public static Color getColorByLabel(String label) {
        Distance fl = Distance.getDistance(label);
        if (null == fl)
            return Color.ALICEBLUE;
        return fl.getColor();
    }

    public static boolean visibilityByLabel(String label) {
        Distance fl = Distance.getDistance(label);
        if (null == fl)
            return true;
        return fl.getVisible();
    }

    public static void setAllVisible(boolean visible) {
        globalDistanceMap.forEach((s, fl) -> {
            fl.setVisible(visible);
        });
    }

    /**
     * @return the value
     */
    public double getValue() {
        return value;
    }

    /**
     * @param value the value to set
     */
    public void setValue(double value) {
        this.value = value;
    }

    /**
     * @return the point1
     */
    public Point3D getPoint1() {
        return point1;
    }

    /**
     * @param point1 the point1 to set
     */
    public void setPoint1(Point3D point1) {
        this.point1 = point1;
    }

    /**
     * @return the point2
     */
    public Point3D getPoint2() {
        return point2;
    }

    /**
     * @param point2 the point2 to set
     */
    public void setPoint2(Point3D point2) {
        this.point2 = point2;
    }

    /**
     * @return the label
     */
    public String getLabel() {
        return label;
    }

    /**
     * @param label the label to set
     */
    public void setLabel(String label) {
        this.label = label;
    }

    /**
     * @return the color
     */
    public Color getColor() {
        return color;
    }

    /**
     * @param color the color to set
     */
    public void setColor(Color color) {
        this.color = color;
    }

    public SimpleBooleanProperty visibleProperty() {
        return this.visible;
    }

    public java.lang.Boolean getVisible() {
        return this.visibleProperty().get();
    }

    public void setVisible(final java.lang.Boolean visible) {
        this.visibleProperty().set(visible);
    }

    /**
     * @return the metric
     */
    public String getMetric() {
        return metric;
    }

    /**
     * @param metric the metric to set
     */
    public void setMetric(String metric) {
        this.metric = metric;
    }

    /**
     * @return the width
     */
    public int getWidth() {
        return width;
    }

    /**
     * @param width the width to set
     */
    public void setWidth(int width) {
        this.width = width;
    }
}
```
