---
layout: single
title: Mandelbrot Code in Java and Kotlin
except: Taking a look at a JavaFX application that suffers from some very common beginner mistakes, converting it to Kotlin and sorting out the issues.
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /pages/mandelbrot_code/
skip_link: true
mandelbrot: /assets/posts/Mandelbrot.png
---
 <script src="/assets/js/playground.js" data-selector=".kotlin-code"></script>
# Introduction


# The Java Code

``` java
public class myMandelbrot extends Application {

    /* =========================================Variables================================================ */

    double width = 800;
    double height = 600;
    double maximumIterations = 50;
    Canvas canvas = new Canvas(width, height);
    WritableImage actualImage;
    double zoom = 250.0;
    double xPos = -470; //add 0 on both of the coordinates for the accurate plane
    double yPos = 30;
    double hue = 264.0;
    double saturation = maximumIterations;
    double brightness = 0.9;
    int R = 60;
    int G = 0;
    int B = 60;

    /* =========================================MainMethod================================================ */

    public static void main(String[] args) {
        launch(args);
    }

    /* =========================================MainStage================================================= */

    @Override
    public void start(Stage stage) {
        Group group = new Group(canvas);

        Scene scene = new Scene(group, width, height);
        scene.setOnKeyPressed(event -> {
            switch (event.getCode()) {
                case W, UP -> {
                    if (event.isShiftDown()) {
                        up(10);
                    } else {
                        up(100);
                    }
                }
                case A, LEFT -> {
                    if (event.isShiftDown()) {
                        left(10);
                    } else {
                        left(100);
                    }
                }
                case S, DOWN -> {
                    if (event.isShiftDown()) {
                        down(10);
                    } else {
                        down(100);
                    }
                }
                case D, RIGHT -> {
                    if (event.isShiftDown()) {
                        right(10);
                    } else {
                        right(100);
                    }
                }
                case EQUALS -> zoomIn();
                case MINUS -> zoomOut();
                case SPACE -> reset();
                case ESCAPE -> Platform.exit();
            }
        });     //key listener
        scene.setOnMouseClicked(event -> {
            switch (event.getButton()) {
                case PRIMARY -> {

                    zoom /= 0.7;

                    MandelbrotSet();
                }
                case SECONDARY -> {
                    zoom *= 0.7;
                    MandelbrotSet();
                }
            }
        });   //mouse listener for easier zoom

        TextField typeHeight = new TextField();
        TextField typeWidth = new TextField();
        Button button1 = new Button("Save");

        TextField typeIter = new TextField();
        Button button2 = new Button("Refresh");

        mainMenuBar(stage, group, typeHeight, typeWidth, typeIter, button1, button2);

        textFieldsImg(group, typeHeight, typeWidth, button1);

        textFieldIter(group, typeIter, button2);

        stage.widthProperty().addListener((obs, oldVal, newVal) -> {
            canvas.widthProperty().bind(stage.widthProperty());
            MandelbrotSet();
        });

        stage.heightProperty().addListener((obs, oldVal, newVal) -> {
            canvas.heightProperty().bind(stage.heightProperty());
            MandelbrotSet();
        });

        stage.setScene(scene);

        long start = System.currentTimeMillis();
        //MandelbrotSet();
        long end = System.currentTimeMillis();
        long result = (end - start);
        runTime(group, stage, result);

        stage.setTitle("Mandelbrot Set");
        stage.show();
    }

    /* =========================================Iterations================================================ */

    public int iterationChecker(double cr, double ci) {
        int iterationsOfZ = 0;
        double zr = 0.0;
        double zi = 0.0;

        while (iterationsOfZ < maximumIterations && (zr * zr) + (zi * zi) < 4) {
            double oldZr = zr;
            zr = (zr * zr) - (zi * zi) + cr;
            zi = 2 * (oldZr * zi) + ci;
            iterationsOfZ++;
        }
        return iterationsOfZ;
    }

    /* ========================================MandelbrotSet============================================== */

    public void MandelbrotSet() {
        WritableImage image = new WritableImage((int) canvas.getWidth(), (int) canvas.getHeight());
        actualImage = image;
        double centerY = canvas.getWidth() / 2.0;
        double centerX = canvas.getHeight() / 2.0;
        for (int x = 0; x < canvas.getWidth(); x++) {
            for (int y = 0; y < canvas.getHeight(); y++) {
                double cr = xPos / width + (x - centerY) / zoom;
                double ci = yPos / height + (y - centerX) / zoom;       //getting position of the points on the canvas

                int iterations = iterationChecker(cr, ci);

                if (iterations == maximumIterations) {  //inside the set
                    image.getPixelWriter().setColor(x, y, Color.rgb(R, G, B));
                } else if (brightness == 0.9) {  //white background
                    image.getPixelWriter().setColor(x, y, Color.hsb(hue, iterations / maximumIterations, brightness));
                } else if (hue == 300) {  //colorful background
                    image.getPixelWriter().setColor(x, y, Color.hsb(hue * iterations / maximumIterations, saturation, brightness));
                } else if (hue == 0 && saturation == 0 && brightness == 1) {
                    image.getPixelWriter().setColor(x, y, Color.hsb(hue, saturation, brightness));
                } else {   //black background
                    image.getPixelWriter().setColor(x, y, Color.hsb(hue, saturation, iterations / brightness));
                }
            }
            canvas.getGraphicsContext2D().drawImage(image, 0, 0); //x and y coordinates of the image.
        }
    }

    /* ===========================================Colors================================================== */

    public void colorLight() {
        hue = 246.0;
        saturation = maximumIterations;
        brightness = 0.9;
        R = 60;
        G = 0;
        B = 60;
        MandelbrotSet();
    }

    public void colorDark() {
        hue = 0;
        saturation = 0;
        brightness = maximumIterations;
        R = 15;
        G = 15;
        B = 15;
        MandelbrotSet();
    }

    public void colorHue() {
        hue = 300.0;
        saturation = 1.0;
        brightness = 1.0;
        R = 35;
        G = 0;
        B = 35;
        MandelbrotSet();
    }

    public void colorWhite() {
        hue = 0.0;
        saturation = 0.0;
        brightness = 1.0;
        R = 0;
        G = 0;
        B = 0;
        MandelbrotSet();
    }

    /* ==========================================Position================================================= */

    public void up(int number) {
        yPos -= (height / zoom) * number;
        MandelbrotSet();
    }

    public void down(int number) {
        yPos += (height / zoom) * number;
        MandelbrotSet();
    }

    public void left(int number) {
        xPos -= (width / zoom) * number;
        MandelbrotSet();
    }

    public void right(int number) {
        xPos += (width / zoom) * number;
        MandelbrotSet();
    }

    public void zoomIn() {
        zoom /= 0.7;
        MandelbrotSet();
    }

    public void zoomOut() {
        zoom *= 0.7;
        MandelbrotSet();
    }

    public void reset() {
        zoom = 250.0;
        xPos = -470;
        yPos = 30;
        MandelbrotSet();
    }

    /* ==========================================SaveImage================================================ */

    public void saveImage(Stage stage, WritableImage image) {
        FileChooser fc = new FileChooser();
        fc.setTitle("Save File");
        FileChooser.ExtensionFilter extensions = new FileChooser.ExtensionFilter("Images *.jpg, *.png", "*.jpg", "*.png");

        fc.getExtensionFilters().add(extensions);

        File file = fc.showSaveDialog(stage);
        if (file != null) {
            try {
                canvas.snapshot(null, image);
                RenderedImage renderedImage = SwingFXUtils.fromFXImage(image, null);
                ImageIO.write(renderedImage, "png", file);
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }

    /* ============================================MenuBar================================================ */

    public void mainMenuBar(Stage stage, Group group, TextField typeHeight, TextField typeWidth, TextField typeIter, Button button1, Button button2) {
        MenuBar menubar = new MenuBar();
        menu1(menubar, group);
        menu2(menubar);
        menu3(stage, menubar, typeHeight, typeWidth, typeIter, button1, button2);

    }

    public void menu1(MenuBar menubar, Group group) {
        Menu menu1 = new Menu("Color");
        menubar.getMenus().add(menu1);
        group.getChildren().add(menubar);
        menubar.setPrefWidth(176);
        CheckMenuItem m1i1 = new CheckMenuItem("Light");
        CheckMenuItem m1i2 = new CheckMenuItem("Dark");
        CheckMenuItem m1i3 = new CheckMenuItem("Colorful");
        CheckMenuItem m1i4 = new CheckMenuItem("Solid White");
        m1i1.setOnAction(e -> {
            if (m1i1.isSelected()) {
                colorLight();
                m1i2.setSelected(false);
                m1i3.setSelected(false);
                m1i4.setSelected(false);
            }
        });
        m1i2.setOnAction(e -> {
            if (m1i2.isSelected()) {
                colorDark();
                m1i1.setSelected(false);
                m1i3.setSelected(false);
                m1i4.setSelected(false);
            }
        });
        m1i3.setOnAction(e -> {
            if (m1i3.isSelected()) {
                colorHue();
                m1i1.setSelected(false);
                m1i2.setSelected(false);
                m1i4.setSelected(false);
            }
        });
        m1i4.setOnAction(e -> {
            if (m1i4.isSelected()) {
                colorWhite();
                m1i1.setSelected(false);
                m1i2.setSelected(false);
                m1i3.setSelected(false);
            }
        });
        m1i1.setAccelerator(KeyCombination.keyCombination("1"));
        m1i2.setAccelerator(KeyCombination.keyCombination("2"));
        m1i3.setAccelerator(KeyCombination.keyCombination("3"));
        m1i4.setAccelerator(KeyCombination.keyCombination("4"));
        menu1.getItems().addAll(m1i1, m1i2, m1i3, m1i4);
    }

    public void menu2(MenuBar menubar) {
        Menu menu2 = new Menu("View");
        menubar.getMenus().add(menu2);
        MenuItem m2i1 = new MenuItem("Zoom in");
        MenuItem m2i2 = new MenuItem("Zoom out");
        MenuItem m2i3 = new MenuItem("Reset");
        MenuItem m2i4 = new MenuItem("Move Up");
        MenuItem m2i5 = new MenuItem("Move Down");
        MenuItem m2i6 = new MenuItem("Move Left");
        MenuItem m2i7 = new MenuItem("Move Right");

        m2i1.setAccelerator(KeyCombination.keyCombination("Plus"));
        m2i2.setAccelerator(KeyCombination.keyCombination("Minus"));
        m2i3.setAccelerator(KeyCombination.keyCombination("Space"));
        m2i4.setAccelerator(KeyCombination.keyCombination("UP"));
        m2i5.setAccelerator(KeyCombination.keyCombination("DOWN"));
        m2i6.setAccelerator(KeyCombination.keyCombination("LEFT"));
        m2i7.setAccelerator(KeyCombination.keyCombination("RIGHT"));

        m2i1.setOnAction(t -> zoomIn());
        m2i2.setOnAction(t -> zoomOut());
        m2i3.setOnAction(t -> reset());
        m2i4.setOnAction(t -> up(100));
        m2i5.setOnAction(t -> down(100));      //menubar location
        m2i6.setOnAction(t -> left(100));
        m2i7.setOnAction(t -> right(100));
        menu2.getItems().addAll(m2i1, m2i2, m2i3, new SeparatorMenuItem(), m2i4, m2i5, m2i6, m2i7);
    }

    public void menu3(Stage stage, MenuBar menubar, TextField typeHeight, TextField typeWidth, TextField typeIter, Button button1, Button button2) {
        Menu menu3 = new Menu("Image");
        menubar.getMenus().add(menu3);

        MenuItem m3i1 = new MenuItem("Set Iterations");
        MenuItem m3i2 = new MenuItem("Save Image");

        menu3.getItems().addAll(m3i1, m3i2);

        m3i2.setOnAction(e -> {
            typeHeight.setVisible(true);
            typeWidth.setVisible(true);
            button1.setVisible(true);
            button2.setVisible(false);
            typeIter.setVisible(false);
        });
        button1.setOnAction(e -> {       //setting image resolution and saving
            canvas.widthProperty().unbind();
            canvas.heightProperty().unbind();
            canvas.setWidth(Integer.parseInt(typeHeight.getText()));
            canvas.setHeight(Integer.parseInt(typeWidth.getText()));
            MandelbrotSet();

            saveImage(stage, actualImage);
            canvas.widthProperty().bind(stage.widthProperty());
            canvas.heightProperty().bind(stage.heightProperty());
            MandelbrotSet();

            typeHeight.setVisible(false);
            typeWidth.setVisible(false);
            button1.setVisible(false);
        });
        m3i1.setAccelerator(KeyCombination.keyCombination("CTRL + C"));
        m3i1.setOnAction(e -> {
            typeIter.setVisible(true);
            button2.setVisible(true);
            typeHeight.setVisible(false);
            typeWidth.setVisible(false);
            button1.setVisible(false);
        });
        button2.setOnAction(e -> {
            typeIter.setVisible(false);
            button2.setVisible(false);
            maximumIterations = Double.parseDouble(typeIter.getText());
            MandelbrotSet();
        });
        m3i2.setAccelerator(KeyCombination.keyCombination("CTRL + V"));
    }


    /* ========================================TextFields================================================= */

    public void textFieldsImg(Group group, TextField typeHeight, TextField typeWidth, Button button) {

        button.setLayoutX(120);
        button.setLayoutY(33);
        button.setPrefWidth(50);

        typeHeight.setLayoutX(7);
        typeHeight.setLayoutY(33);
        typeHeight.setPrefWidth(50);
        typeWidth.setLayoutX(63);
        typeWidth.setLayoutY(33);
        typeWidth.setPrefWidth(50);

        onlyNumbers(typeHeight);
        onlyNumbers(typeWidth);

        group.getChildren().add(typeHeight);
        group.getChildren().add(typeWidth);
        group.getChildren().add(button);
        typeHeight.setVisible(false);
        typeWidth.setVisible(false);
        button.setVisible(false);
    }

    public void textFieldIter(Group group, TextField typeIter, Button button) {
        typeIter.setLayoutY(200);
        button.setLayoutX(92);
        button.setLayoutY(33);
        button.setPrefWidth(78);

        typeIter.setLayoutX(7);
        typeIter.setLayoutY(33);
        typeIter.setPrefWidth(78);

        onlyNumbers(typeIter);
        typeIter.setVisible(false);
        button.setVisible(false);

        group.getChildren().add(typeIter);
        group.getChildren().add(button);
    }

    public void onlyNumbers(TextField text) {
        text.textProperty().addListener((observable, oldNum, newNum) -> {
            if (!newNum.matches("\\d*")) text.setText(newNum.replaceAll("[^\\d]", ""));
        });
    }

    /* ==========================================CalculateRunTime============================================ */

    public void runTime(Group group, Stage stage, long result) {
        Label label = new Label((" Time to compile: " + (result / 1000.0) + " sec.   (" + (int) maximumIterations + "  iterations)  " + (int) width + " x " + (int) height));
        label.layoutXProperty().bind(stage.widthProperty().subtract(stage.getWidth()));   //Should align label to horizontal center, but it is off
        label.layoutYProperty().bind(stage.heightProperty().subtract(label.getHeight() + 45));
        group.getChildren().add(label);
    }

    /* ==========================================CalculateRunTime============================================ */

    private static final Executor executor = Executors.newSingleThreadExecutor();

    /* ============================================ParallelMode============================================== */

}
```

# The Kotlin Code

```kotlin
class MyMandelbrot : Application() {
   private val width = 800.0
   private val height = 600.0
   private var maximumIterations = 50.0
   private var canvas = Canvas(width, height)
   private val zoom: DoubleProperty = SimpleDoubleProperty(250.0)
   private var xPos = -470.0
   private var yPos = 30.0
   private var hue = 264.0
   private var saturation = maximumIterations
   private var brightness = 0.9
   private var red = 60
   private var green = 0
   private var blue = 60
   private val setProgress: DoubleProperty = SimpleDoubleProperty(0.0)
   private val zoomIn: () -> Unit = { mandelbrotSet { zoom.value /= 0.4 } }
   private val zoomOut: () -> Unit = { mandelbrotSet { zoom.value *= 0.4 } }
   private val up: (Int) -> Unit = { mandelbrotSet { yPos -= height / zoom.value * it } }
   private val down: (Int) -> Unit = { mandelbrotSet { yPos += height / zoom.value * it } }
   private val left: (Int) -> Unit = { mandelbrotSet { xPos -= width / zoom.value * it } }
   private val right: (Int) -> Unit = { mandelbrotSet { xPos += width / zoom.value * it } }
   private val reset: () -> Unit = {
      mandelbrotSet {
         zoom.value = 250.0
         xPos = -470.0
         yPos = 30.0
      }
   }

   override fun start(stage: Stage) {
      stage.scene = Scene(createContent(), width, height)
      stage.title = "Mandelbrot Set"
      stage.show()
   }

   private fun createContent() = BorderPane().apply {
      top = mainMenuBar()
      center = canvas
      bottom = createBottom()
      minWidth = this@MyMandelbrot.width
      minHeight = this@MyMandelbrot.height
      addEventFilter(KeyEvent.KEY_PRESSED) {
         if (processKeystroke(it.code, it.isShiftDown)) {
            it.consume()
         }
      }
      with(canvas) {
         widthProperty().also {
            it.bind(this@apply.widthProperty())
            it.addListener(InvalidationListener {
               mandelbrotSet()
            })
         }
         heightProperty().also {
            it.bind(this@apply.heightProperty().add(-60.0))
            it.addListener(InvalidationListener {
               mandelbrotSet()
            })
         }
         onMouseClicked = EventHandler {
            when (it.button) {
               MouseButton.PRIMARY -> zoomIn()
               MouseButton.SECONDARY -> zoomOut()
               else -> {}
            }
         }
      }
   }

   private fun processKeystroke(keyCode: KeyCode, isShiftDown: Boolean): Boolean {
      var didSomething = true
      when (keyCode) {
         KeyCode.W, KeyCode.UP -> up(if (isShiftDown) 10 else 100)
         KeyCode.A, KeyCode.LEFT -> left(if (isShiftDown) 10 else 100)
         KeyCode.S, KeyCode.DOWN -> down(if (isShiftDown) 10 else 100)
         KeyCode.D, KeyCode.RIGHT -> right(if (isShiftDown) 10 else 100)
         KeyCode.EQUALS -> zoomIn()
         KeyCode.MINUS -> zoomOut()
         KeyCode.SPACE -> reset()
         KeyCode.ESCAPE -> Platform.exit()
         else -> {
            didSomething = false
         }
      }
      return didSomething
   }

   private fun createBottom() = HBox(6.0).apply {
      children += listOf(Label("Iterations: "),
                         iterationsBox(),
                         ProgressBar().apply {
                            progressProperty().bind(setProgress)
                            prefWidth = 200.0
                         },
                         Label("      Pending: "),
                         boundLabel(Bindings.createStringBinding({ pendingSets.value.toString() }, pendingSets)),
                         Label("    Zoom:"),
                         boundLabel(Bindings.createStringBinding({ zoom.value.roundToInt().toString() }, zoom)))
      padding = Insets(6.0)
   }

   private fun iterationsBox() = HBox(6.0).apply {
      val iterationsTextField = TextField()
      val textFormatter: TextFormatter<Int> = TextFormatter(IntegerStringConverter())
      iterationsTextField.textFormatter = textFormatter
      val refreshButton = Button("Refresh").apply {
         onAction = EventHandler {
            canvas.requestFocus()
            isDisable = true
            mandelbrotSet { maximumIterations = textFormatter.value?.toDouble() ?: 50.0 }
            isDisable = false
         }
         defaultButtonProperty().bind(iterationsTextField.focusedProperty())
      }
      children += listOf(iterationsTextField, refreshButton)
   }

   private fun boundLabel(contents: ObservableStringValue) = Label().apply { textProperty().bind(contents) }

   private fun iterationChecker(cr: Double, ci: Double): Int {
      var iterationsOfZ = 0
      var zr = 0.0
      var zi = 0.0
      while (iterationsOfZ < maximumIterations && ((zr.pow(2) + zi.pow(2)) < 4)) {
         val oldZr = zr
         zr = zr * zr - zi * zi + cr
         zi = 2 * (oldZr * zi) + ci
         iterationsOfZ++
      }
      return iterationsOfZ
   }

   private var taskRunning = false
   private val preProcesses = mutableListOf<() -> Unit>()
   private val pendingSets: IntegerProperty = SimpleIntegerProperty(0)

   private fun mandelbrotSet(preProcessor: () -> Unit = {}) {
      if ((canvas.width > 0) && (canvas.height > 0)) {
         preProcesses += preProcessor
         pendingSets.value = preProcesses.size
         println("In mandelbrot ${preProcesses.size} - already running?  $taskRunning")
         if (!taskRunning) {
            val startTime = System.currentTimeMillis()
            taskRunning = true
            preProcesses.forEach { it.invoke() }
            preProcesses.clear()
            pendingSets.value = 0
            val task = object : Task<WritableImage>() {
               override fun call(): WritableImage = performMandelbrot { workDone, maxWork -> updateProgress(workDone, maxWork) }
            }
            task.setOnSucceeded {
               canvas.graphicsContext2D.drawImage(task.value, 0.0, 0.0)
               taskRunning = false
               if (preProcesses.isNotEmpty()) {
                  mandelbrotSet()
               }
            }
            task.setOnFailed {
               println("Failed! ${task.exception.stackTraceToString()}")
               taskRunning = false
            }
            setProgress.unbind()
            setProgress.bind(task.progressProperty())
            Thread(task).start()
         }
      }
   }

   private fun performMandelbrot(progressConsumer: (workDone: Long, maxWork: Long) -> Unit) = WritableImage(canvas.width.toInt(), canvas.height.toInt()).apply {
      println("Starting mandelbrot")
      val centerY = width / 2.0
      val centerX = height / 2.0
      val counter = AtomicLong(0)
      (0 until width.roundToInt()).toList().parallelStream().forEach { x ->
         progressConsumer.invoke(counter.incrementAndGet(), width.roundToLong())
         for (y: Int in 0 until height.roundToInt()) {
            val cr = xPos / width + (x - centerY) / zoom.value
            val ci = yPos / height + (y - centerX) / zoom.value
            pixelWriter.setColor(x, y, determineColour(iterationChecker(cr, ci)))
         }
      }
      println("Done")
   }

   private fun determineColour(iterations: Int): Color {
      if (iterations.toDouble() == maximumIterations) {  //inside the set
         return Color.rgb(red, green, blue)
      }
      if (brightness == 0.9) {  //white background
         return Color.hsb(hue, iterations / maximumIterations, brightness)
      }
      if (hue == 300.0) {  //colorful background
         return Color.hsb(hue * iterations / maximumIterations, saturation, brightness)
      }
      if (hue == 0.0 && saturation == 0.0 && brightness == 1.0) {
         return Color.hsb(hue, saturation, brightness)
      }
      return Color.hsb(hue, saturation, iterations / brightness)
   }

   private fun mainMenuBar() = MenuBar().apply {
      prefWidth = 176.0
      menus.addAll(menu1(), menu2())
   }

   @Suppress("UNCHECKED_CAST")
   private fun menu1() = Menu("Color").also {
      ToggleGroup().apply {
         toggles += listOf(makeColourMenuItem("Light", "1") { mandelbrotSet { newSettings(246.0, maximumIterations, 0.9, 60, 0, 60) } },
                           makeColourMenuItem("Dark", "2") { mandelbrotSet { newSettings(0.0, 0.0, maximumIterations, 15, 15, 15) } },
                           makeColourMenuItem("Colourful", "3") { mandelbrotSet { newSettings(300.0, 1.0, 1.0, 35, 0, 35) } },
                           makeColourMenuItem("Solid White", "4") { mandelbrotSet { newSettings(0.0, 0.0, 1.0, 0, 0, 0) } })
         it.items += toggles as List<MenuItem>
      }
   }

   private fun makeColourMenuItem(name: String, accel: String, func: () -> Unit) = RadioMenuItem(name).apply {
      selectedProperty().addListener { ob: Observable -> if ((ob as ObservableBooleanValue).value) func.invoke() }
      accelerator = KeyCombination.keyCombination(accel)
   }

   private fun menu2() = Menu("View").apply {
      items += listOf(makeMenuItem("Zoom In", "Plus") { zoomIn() },
                      makeMenuItem("Zoom Out", "Minus") { zoomOut() },
                      makeMenuItem("Reset", "Space") { reset() },
                      makeMenuItem("Move Up", "UP") { up(100) },
                      makeMenuItem("Move Down", "DOWN") { down(100) },
                      makeMenuItem("Move Left", "LEFT") { left(100) },
                      makeMenuItem("Move Right", "RIGHT") { right(100) })
   }

   private fun makeMenuItem(name: String, accel: String, eventHandler: EventHandler<ActionEvent>) = MenuItem(name).apply {
      accelerator = KeyCombination.keyCombination(accel)
      onAction = eventHandler
   }

   private fun newSettings(newHue: Double, newSat: Double, newBright: Double, newRed: Int, newGreen: Int, newBlue: Int) {
      hue = newHue
      saturation = newSat
      brightness = newBright
      red = newRed
      green = newGreen
      blue = newBlue
   }
}

fun main() {
   Application.launch(MyMandelbrot::class.java)
}
```
