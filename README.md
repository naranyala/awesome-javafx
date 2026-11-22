# Awesome JavaFX

A focused guide to **JavaFX** desktop development: its UI model, modules, styling, architecture, packaging, testing, native integration, ecosystem, and long-term maintenance.

JavaFX is a Java-based desktop toolkit for building rich cross-platform applications. It combines a retained-mode scene graph, controls, CSS, property binding, FXML, charts, media, WebView, animation, and graphics APIs in one coherent platform.

## Contents

- [Why JavaFX?](#why-javafx)
- [Status and maturity](#status-and-maturity)
- [Core mental model](#core-mental-model)
- [Minimal implementation examples](#minimal-implementation-examples)
- [JavaFX modules](#javafx-modules)
- [Controls and layout](#controls-and-layout)
- [Properties, bindings, and observable collections](#properties-bindings-and-observable-collections)
- [FXML and controllers](#fxml-and-controllers)
- [CSS and theming](#css-and-theming)
- [Concurrency and background work](#concurrency-and-background-work)
- [Packaging and deployment](#packaging-and-deployment)
- [Testing and diagnostics](#testing-and-diagnostics)
- [Ecosystem and library ideas](#ecosystem)
- [Practical checklist](#practical-checklist)

## Why JavaFX?

JavaFX is a strong choice when an application needs:

- a modern Java-first desktop UI;
- consistent rendering across Windows, macOS, and Linux;
- CSS styling and declarative FXML views;
- observable state, properties, and binding;
- tables, trees, forms, charts, animation, media, or WebView;
- a deployable application with a bundled Java runtime;
- a UI that does not require a browser stack for every screen.

It is not a native-widget toolkit in the SWT sense, and it is not a web application framework. JavaFX renders its own scene graph while integrating with the desktop through Java and platform services.

## Status and maturity

- **OpenJFX:** the open-source upstream implementation developed in the OpenJDK ecosystem.
- **Separate distribution:** JavaFX is no longer bundled with the JDK. Applications must declare JavaFX modules and package them with the application or runtime image.
- **Active maintenance:** the project receives fixes and improvements for rendering, HiDPI, accessibility, controls, media, WebView, input, and supported operating systems.
- **Commercial ecosystem:** vendors such as Gluon provide builds, tooling, support, mobile/embedded solutions, and deployment services.
- **Maturity:** JavaFX is mature enough for production desktop software, but its ecosystem is smaller than web UI ecosystems and it requires deliberate runtime packaging.

Always verify the compatibility matrix between the JDK, OpenJFX distribution, operating system, architecture, and build plugins before upgrading.

## Core mental model

A JavaFX application is built around an **application thread**, a **scene graph**, and **observable state**:

```text
Application
  └── Stage              top-level window
        └── Scene        content and root node
              └── Node   controls, layouts, shapes, images, media, and custom views
```

- `Application` owns startup and shutdown.
- `Stage` represents a window.
- `Scene` contains the visual tree and input context.
- `Node` is the base type for controls, layouts, shapes, images, and custom content.
- `Parent` nodes contain children.
- `Control` nodes provide user-facing widgets with skins and behaviors.
- `ObservableValue`, `Property`, and `Binding` connect state to the scene graph.

JavaFX UI objects must be accessed on the **JavaFX Application Thread**. Long-running work must run elsewhere and publish results back to that thread.

## Minimal implementation examples

### A window with a button

```java
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.stage.Stage;

public class HelloApp extends Application {
    @Override
    public void start(Stage stage) {
        Button button = new Button("Click me");
        button.setOnAction(event -> button.setText("Clicked"));

        stage.setScene(new Scene(button, 300, 180));
        stage.setTitle("Hello JavaFX");
        stage.show();
    }

    public static void main(String[] args) {
        launch(args);
    }
}
```

### A small form and layout

```java
VBox form = new VBox(10,
    new Label("Name"),
    new TextField(),
    new Button("Save")
);
form.setPadding(new Insets(16));

Scene scene = new Scene(form, 320, 180);
stage.setScene(scene);
stage.show();
```

The imports for this example are `javafx.geometry.Insets`, `javafx.scene.control.*`, and `javafx.scene.layout.VBox`.

### Binding a label to input

```java
TextField input = new TextField();
Label output = new Label();
output.textProperty().bind(input.textProperty());

VBox root = new VBox(8, input, output);
```

### Running work in the background

```java
Task<String> task = new Task<>() {
    @Override
    protected String call() throws Exception {
        return loadData(); // slow I/O or computation
    }
};

task.setOnSucceeded(event -> resultLabel.setText(task.getValue()));
task.setOnFailed(event -> resultLabel.setText("Could not load data"));
new Thread(task, "loader").start();
```

Here, `resultLabel` is updated on the JavaFX Application Thread by the task event handler. In production, prefer a managed executor and handle cancellation and exceptions explicitly.

### Showing an alert

```java
Alert alert = new Alert(Alert.AlertType.INFORMATION);
alert.setTitle("Saved");
alert.setHeaderText(null);
alert.setContentText("The file was saved successfully.");
alert.showAndWait();
```

### Displaying a list

```java
ListView<String> list = new ListView<>();
list.getItems().addAll("One", "Two", "Three");
list.setOnMouseClicked(event ->
    System.out.println(list.getSelectionModel().getSelectedItem())
);
```

### Styling a control with CSS

```java
Button save = new Button("Save");
save.getStyleClass().add("primary-button");
scene.getStylesheets().add(
    getClass().getResource("/app.css").toExternalForm()
);
```

```css
.primary-button {
    -fx-background-color: #3478f6;
    -fx-text-fill: white;
}
```

### Switching scenes

```java
Button next = new Button("Next");
next.setOnAction(event -> stage.setScene(new Scene(new Label("Second view"), 300, 180)));
```

### Using a checkbox and slider

```java
CheckBox enabled = new CheckBox("Enabled");
Slider volume = new Slider(0, 100, 50);
Label value = new Label();
value.textProperty().bind(volume.valueProperty().asString("Volume: %.0f"));
```

### Playing a simple animation

```java
TranslateTransition animation = new TranslateTransition(Duration.seconds(1), button);
animation.setByX(100);
animation.setAutoReverse(true);
animation.setCycleCount(Animation.INDEFINITE);
animation.play();
```

The imports are `javafx.animation.*` and `javafx.util.Duration`.

### Loading an image

```java
Image image = new Image(
    Objects.requireNonNull(getClass().getResourceAsStream("/logo.png"))
);
ImageView imageView = new ImageView(image);
imageView.setFitWidth(120);
imageView.setPreserveRatio(true);
```

### A minimal FXML view

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?import javafx.scene.control.Button?>
<?import javafx.scene.layout.StackPane?>

<StackPane xmlns:fx="http://javafx.com/fxml" fx:controller="com.example.MainController">
    <Button text="Hello" onAction="#sayHello" />
</StackPane>
```

Load it from Java with:

```java
Parent root = FXMLLoader.load(getClass().getResource("/main.fxml"));
```

### A minimal table

```java
TableView<String> table = new TableView<>();
TableColumn<String, String> column = new TableColumn<>("Items");
column.setCellValueFactory(cell -> new ReadOnlyStringWrapper(cell.getValue()));
table.getColumns().add(column);
table.getItems().addAll("First", "Second");
```

### A menu bar

```java
MenuItem quit = new MenuItem("Quit");
quit.setOnAction(event -> Platform.exit());

Menu file = new Menu("File");
file.getItems().add(quit);
MenuBar menuBar = new MenuBar(file);
```

### Radio buttons

```java
ToggleGroup colors = new ToggleGroup();
RadioButton red = new RadioButton("Red");
RadioButton blue = new RadioButton("Blue");
red.setToggleGroup(colors);
blue.setToggleGroup(colors);
```

### A date picker

```java
DatePicker datePicker = new DatePicker();
datePicker.setOnAction(event ->
    System.out.println(datePicker.getValue())
);
```

### Progress feedback

```java
ProgressBar progress = new ProgressBar();
progress.progressProperty().bind(task.progressProperty());
progress.visibleProperty().bind(task.runningProperty());
```

### Tabs

```java
TabPane tabs = new TabPane(
    new Tab("Home", new Label("Home content")),
    new Tab("Settings", new Label("Settings content"))
);
tabs.getTabs().forEach(tab -> tab.setClosable(false));
```

### Choosing a file

```java
FileChooser chooser = new FileChooser();
chooser.setTitle("Open file");
File selected = chooser.showOpenDialog(stage);
if (selected != null) {
    System.out.println(selected.getAbsolutePath());
}
```

## Getting started: project setup

JavaFX is delivered separately from the JDK. Use a build tool to resolve platform-specific JavaFX artifacts rather than copying SDK jars into a project manually. Pin compatible JDK, OpenJFX, and plugin versions in the build, and verify the combination on each target OS and CPU architecture.

### Gradle (Groovy DSL)

A minimal `build.gradle` using the OpenJFX Gradle plugin:

```groovy
plugins {
    id 'application'
    id 'org.openjfx.javafxplugin' version '0.1.0'
}

repositories { mavenCentral() }

dependencies {
    // Add libraries used by the application here.
}

javafx {
    version = '21.0.2'
    modules = [ 'javafx.controls' ]
}

application {
    mainClass = 'com.example.app/com.example.app.HelloApp'
}
```

Use versions appropriate to your project; this is an illustrative configuration, not a claim that these are the latest releases. Run with `./gradlew run` after creating the Gradle wrapper. Include `javafx.fxml` when using FXML and declare other modules only when needed.

A modular project should include `module-info.java` with the modules it uses and the package containing the application entry point exported or opened as required. For FXML reflection, `opens com.example.app to javafx.fxml;` is commonly needed for controller packages.

### Maven

For Maven, use the OpenJFX Maven plugin and declare the JavaFX modules as dependencies. OpenJFX artifacts are platform-aware; follow the plugin's current documentation for the target platform and module configuration. Typical launch command: `mvn clean javafx:run`.

Keep build instructions synchronized with the actual project and plugin versions. Avoid presenting copied commands as universal: platform classifiers, module paths, and native packaging differ by JDK, OS, and architecture.

## JavaFX modules

Common modules include:

| Module | Purpose |
|---|---|
| `javafx.base` | Properties, bindings, collections, events, and foundational APIs |
| `javafx.graphics` | `Application`, `Stage`, `Scene`, nodes, layout, rendering, animation, input, and windows |
| `javafx.controls` | Buttons, fields, tables, trees, lists, menus, dialogs, charts, and skins |
| `javafx.fxml` | FXML loading and controller integration |
| `javafx.media` | Audio and video playback |
| `javafx.web` | WebView and embedded web content |
| `javafx.swing` | Swing interoperability through `JFXPanel` and `SwingNode` |
| `javafx.swt` | SWT interoperability where supplied by the distribution |

Use only the modules needed by the application when creating a custom runtime with `jlink`.

## Controls and layout

The following bare-minimum layouts are intended to be placed inside `Application.start(Stage stage)`. They use standard controls and layout panes; imports are listed after the examples.

### Vertical form

```java
var name = new TextField();
name.setPromptText("Your name");

var message = new Label("Enter your name");
var greet = new Button("Greet");
greet.setOnAction(event -> message.setText("Hello, " + name.getText()));

var root = new VBox(10, name, greet, message);
root.setPadding(new Insets(16));
stage.setScene(new Scene(root, 300, 180));
stage.show();
```

### Header, content, and footer

```java
var root = new BorderPane();
root.setTop(new Label("My App"));
root.setCenter(new TextArea());
root.setBottom(new Button("Save"));
BorderPane.setMargin(root.getTop(), new Insets(12));
BorderPane.setMargin(root.getBottom(), new Insets(12));
stage.setScene(new Scene(root, 400, 260));
stage.show();
```

### Grid of labeled inputs

```java
var grid = new GridPane();
grid.setHgap(10);
grid.setVgap(10);
grid.setPadding(new Insets(16));
grid.addRow(0, new Label("Email"), new TextField());
grid.addRow(1, new Label("Password"), new PasswordField());
grid.add(new Button("Sign in"), 1, 2);
stage.setScene(new Scene(grid, 360, 180));
stage.show();
```

### List with selection

```java
var list = new ListView<String>();
list.getItems().addAll("Java", "JavaFX", "Kotlin");
var selected = new Label("Choose an item");
list.getSelectionModel().selectedItemProperty().addListener(
    (observable, oldValue, newValue) -> selected.setText("Selected: " + newValue)
);
stage.setScene(new Scene(new VBox(8, list, selected), 280, 240));
stage.show();
```

### Tabs

```java
var tabs = new TabPane(
    new Tab("Home", new Label("Welcome")),
    new Tab("Settings", new CheckBox("Enable feature"))
);
tabs.getTabs().forEach(tab -> tab.setClosable(false));
stage.setScene(new Scene(tabs, 360, 220));
stage.show();
```

These snippets use `javafx.scene.Scene`, `javafx.stage.Stage`, `javafx.scene.control.*`, `javafx.scene.layout.*`, and `javafx.geometry.Insets`. They assume a recent Java version supporting local-variable type inference (`var`).

### Toolbar and split work area

```java
var toolbar = new ToolBar(new Button("Open"), new Button("Save"));
var navigation = new ListView<String>();
navigation.getItems().addAll("Overview", "Documents", "Settings");
var workArea = new TextArea("Work area");
var split = new SplitPane(navigation, workArea);
split.setDividerPositions(0.25);
var root = new BorderPane(split);
root.setTop(toolbar);
stage.setScene(new Scene(root, 640, 400));
stage.show();
```

### Master–detail view

```java
record Person(String name, String role) {}
var people = FXCollections.observableArrayList(
    new Person("Ada", "Engineer"), new Person("Grace", "Scientist"));
var list = new ListView<>(people);
list.setCellFactory(view -> new ListCell<>() {
    @Override protected void updateItem(Person person, boolean empty) {
        super.updateItem(person, empty);
        setText(empty || person == null ? null : person.name());
    }
});
var detail = new Label("Select a person");
list.getSelectionModel().selectedItemProperty().addListener((obs, old, person) ->
    detail.setText(person == null ? "Select a person" : person.name() + " — " + person.role()));
stage.setScene(new Scene(new SplitPane(list, detail), 500, 280));
stage.show();
```

### Searchable list

```java
var source = FXCollections.observableArrayList("Java", "JavaFX", "Kotlin", "Scala");
var filtered = new FilteredList<>(source, value -> true);
var search = new TextField();
search.setPromptText("Filter…");
search.textProperty().addListener((obs, old, text) -> {
    var query = text == null ? "" : text.trim().toLowerCase(Locale.ROOT);
    filtered.setPredicate(value -> value.toLowerCase(Locale.ROOT).contains(query));
});
var list = new ListView<String>(filtered);
stage.setScene(new Scene(new VBox(8, search, list), 300, 280));
stage.show();
```

### Content with a status bar

```java
var status = new Label("Ready");
var progress = new ProgressIndicator();
progress.setVisible(false);
var footer = new HBox(8, status, new Region(), progress);
HBox.setHgrow(footer.getChildren().get(1), Priority.ALWAYS);
footer.setPadding(new Insets(8));
var root = new BorderPane(new TextArea("Content"));
root.setBottom(footer);
stage.setScene(new Scene(root, 500, 320));
stage.show();
```

### Dashboard cards

```java
var cards = new TilePane(12, 12);
cards.setPadding(new Insets(16));
cards.setPrefColumns(2);
for (var title : List.of("Revenue", "Users", "Orders", "Health")) {
    var card = new VBox(8, new Label(title), new Label("—"));
    card.setPadding(new Insets(16));
    card.setStyle("-fx-border-color: lightgray; -fx-border-radius: 6;");
    cards.getChildren().add(card);
}
stage.setScene(new Scene(new ScrollPane(cards), 420, 300));
stage.show();
```

### Validated dialog-style form

```java
var email = new TextField();
email.setPromptText("name@example.com");
var submit = new Button("Submit");
submit.disableProperty().bind(email.textProperty().isEmpty());
var form = new VBox(10, new Label("Email"), email, submit);
form.setPadding(new Insets(18));
var dialog = new Dialog<Void>();
dialog.setTitle("Contact details");
dialog.getDialogPane().setContent(form);
dialog.getDialogPane().getButtonTypes().add(ButtonType.CLOSE);
submit.setOnAction(event -> dialog.close());
dialog.showAndWait();
```

This is only a required-field check; real applications should validate format and show an accessible explanation of errors.

### Menu-driven app with shortcut

```java
var open = new MenuItem("Open");
open.setAccelerator(new KeyCodeCombination(KeyCode.O, KeyCombination.SHORTCUT_DOWN));
open.setOnAction(event -> System.out.println("Open requested"));
var menuBar = new MenuBar(new Menu("File", open, new SeparatorMenuItem(), new MenuItem("Exit")));
var root = new BorderPane(new Label("Main content"));
root.setTop(menuBar);
stage.setScene(new Scene(root, 480, 300));
stage.show();
```

### Scrollable settings page

```java
var settings = new VBox(12,
    new Label("Preferences"), new CheckBox("Launch at startup"),
    new CheckBox("Show notifications"), new Label("Text size"),
    new Slider(10, 24, 14));
settings.setPadding(new Insets(16));
stage.setScene(new Scene(new ScrollPane(settings), 360, 280));
stage.show();
```

### Resize-aware layout

```java
var left = new Label("Navigation");
var content = new TextArea("Resize the window");
var root = new BorderPane(content);
root.setLeft(left);
BorderPane.setMargin(left, new Insets(12));
// BorderPane and resizable controls allocate space as the window changes size.
stage.setMinWidth(320);
stage.setMinHeight(220);
stage.setScene(new Scene(root, 640, 400));
stage.show();
```

Prefer resizable layout panes and constraints over manually setting node coordinates or repeatedly rebuilding the scene when the window resizes.

### Custom-drawn graphic with Canvas

```java
var canvas = new Canvas(320, 180);
var g = canvas.getGraphicsContext2D();
g.setFill(Color.LIGHTSKYBLUE);
g.fillRect(0, 0, canvas.getWidth(), canvas.getHeight());
g.setFill(Color.DARKBLUE);
g.fillOval(120, 45, 80, 80);
var root = new StackPane(canvas);
stage.setScene(new Scene(root, 360, 220));
stage.show();
```

Canvas is immediate-mode drawing: redraw when content changes. Use scene-graph nodes instead when individual shapes need built-in picking, accessibility, or independent layout. These examples additionally use `javafx.collections.*`, `java.util.*`, and `javafx.scene.canvas.Canvas` / `javafx.scene.paint.Color` as applicable.

JavaFX includes controls for most conventional desktop applications:

- buttons, toggle buttons, check boxes, radio buttons, sliders, spinners, and date pickers;
- text fields, password fields, formatted fields, text areas, and rich text building blocks;
- list views, tree views, table views, tree tables, and editable cells;
- menus, toolbars, context menus, dialogs, alerts, pagination, tabs, accordions, and split panes;
- charts including line, area, bar, bubble, scatter, pie, and stacked variants;
- progress indicators, tooltips, separators, scroll panes, and custom controls.

Common layout containers include `BorderPane`, `HBox`, `VBox`, `GridPane`, `FlowPane`, `TilePane`, `StackPane`, `AnchorPane`, and `ScrollPane`. Prefer layout constraints over fixed coordinates, and test resizing, localization, accessibility, and HiDPI.

### Compact reusable interaction patterns

**Confirmation before a destructive action:**

```java
var remove = new Button("Delete");
remove.setOnAction(event -> {
    var answer = new Alert(Alert.AlertType.CONFIRMATION,
        "Delete this item?", ButtonType.CANCEL, ButtonType.OK).showAndWait();
    if (answer.orElse(ButtonType.CANCEL) == ButtonType.OK) deleteItem();
});
```

**Show validation state without a modal dialog:**

```java
var input = new TextField();
var error = new Label();
var save = new Button("Save");
var valid = Bindings.createBooleanBinding(
    () -> input.getText() != null && input.getText().trim().length() >= 3,
    input.textProperty());
save.disableProperty().bind(valid.not());
input.textProperty().addListener((obs, old, value) ->
    error.setText(valid.get() ? "" : "Enter at least 3 characters"));
```

**Accessible icon button:**

```java
var refresh = new Button("⟳");
refresh.setAccessibleText("Refresh data");
refresh.setTooltip(new Tooltip("Refresh data"));
refresh.setOnAction(event -> refreshData());
```

Give controls meaningful accessible text, preserve keyboard focus visibility, and do not communicate errors by color alone.

### Debounced search

A pause before filtering keeps expensive predicates from running on every keystroke. This example owns and stops its JavaFX `PauseTransition` with the view.

```java
var source = FXCollections.observableArrayList("Java", "JavaFX", "Kotlin");
var filtered = new FilteredList<>(source, item -> true);
var search = new TextField();
var debounce = new PauseTransition(Duration.millis(250));
search.textProperty().addListener((obs, old, value) -> debounce.playFromStart());
debounce.setOnFinished(event -> {
    var query = search.getText().strip().toLowerCase(Locale.ROOT);
    filtered.setPredicate(item -> item.toLowerCase(Locale.ROOT).contains(query));
});
var list = new ListView<String>(filtered);
```

### File open with `FileChooser`

```java
var open = new Button("Open…");
var text = new TextArea();
open.setOnAction(event -> {
    var file = new FileChooser().showOpenDialog(open.getScene().getWindow());
    if (file == null) return; // user cancelled
    try {
        text.setText(Files.readString(file.toPath(), StandardCharsets.UTF_8));
    } catch (IOException ex) {
        new Alert(Alert.AlertType.ERROR, "Could not open file: " + ex.getMessage()).showAndWait();
    }
});
```

For large files, do the read in a background task. Consider encoding, file size, and whether the application should preserve unsaved changes.

## Properties, bindings, and observable collections

JavaFX properties are one of its defining features:

```java
Label status = new Label();
StringProperty message = new SimpleStringProperty("Ready");
status.textProperty().bind(message);

BooleanProperty busy = new SimpleBooleanProperty();
button.disableProperty().bind(busy);
```

Useful types include:

- `StringProperty`, `BooleanProperty`, `IntegerProperty`, `DoubleProperty`, and object properties;
- `ObservableList`, `ObservableSet`, and `ObservableMap`;
- `Bindings` and fluent binding expressions;
- `ObjectProperty`, `ReadOnlyProperty`, and `ReadOnlyObjectWrapper`;
- `CellValueFactory` and `CellFactory` for tables and lists.

Use properties at UI boundaries, not necessarily throughout the entire domain model. A separate domain model and view model usually makes persistence, validation, and testing easier.

Avoid creating unnecessary bindings for large or frequently changing datasets. Profile listeners and collection updates when tables or charts become slow.

### Editable table with typed columns

```java
record Product(StringProperty name, DoubleProperty price) {
    Product(String name, double price) {
        this(new SimpleStringProperty(name), new SimpleDoubleProperty(price));
    }
}
var products = FXCollections.observableArrayList(
    new Product("Notebook", 4.50), new Product("Pen", 1.25));
var table = new TableView<Product>(products);
var nameColumn = new TableColumn<Product, String>("Name");
nameColumn.setCellValueFactory(cell -> cell.getValue().name());
nameColumn.setCellFactory(TextFieldTableCell.forTableColumn());
nameColumn.setOnEditCommit(event -> event.getRowValue().name().set(event.getNewValue()));
var priceColumn = new TableColumn<Product, Number>("Price");
priceColumn.setCellValueFactory(cell -> cell.getValue().price());
table.getColumns().addAll(nameColumn, priceColumn);
table.setEditable(true);
```

This uses mutable properties because cells need observable editable values. For larger datasets, page or load data incrementally instead of materializing everything in the table.

### FXML loading with a controller

```java
// In Application.start:
var loader = new FXMLLoader(getClass().getResource("/views/main.fxml"));
var scene = new Scene(loader.load());
stage.setScene(scene);
stage.show();
```

```xml
<!-- /views/main.fxml -->
<VBox xmlns:fx="http://javafx.com/fxml" fx:controller="com.example.MainController" spacing="8">
    <Label fx:id="message" text="Ready" />
    <Button text="Run" onAction="#run" />
</VBox>
```

```java
public class MainController {
    @FXML private Label message;
    @FXML private void run() { message.setText("Running"); }
}
```

In a named module, open the controller package to `javafx.fxml` in `module-info.java`. Handle loader exceptions as startup failures and avoid placing business logic in controllers.

## FXML and controllers

FXML is an XML-based declarative format for describing a scene graph. It can make views easier to inspect, style, and edit separately from application logic.

Recommended practices:

- keep controllers thin and delegate business logic to services or view models;
- use `fx:id` only for elements that need controller access;
- use `FXMLLoader` with an explicit controller factory when dependency injection is needed;
- keep resource loading relative to the module/classpath rather than the current working directory;
- handle `FXMLLoader` failures as startup/configuration errors with useful diagnostics;
- avoid putting database, network, or complex domain logic in controller methods;
- use typed events and commands rather than making controllers depend on the entire scene.

FXML is optional. Programmatic JavaFX views are often better for highly dynamic interfaces, generated UIs, or teams that prefer compile-time refactoring.

## CSS and theming

### Runtime stylesheet switching

```java
var lightCss = Objects.requireNonNull(getClass().getResource("/light.css")).toExternalForm();
var darkCss = Objects.requireNonNull(getClass().getResource("/dark.css")).toExternalForm();
var scene = new Scene(new BorderPane(new Label("Theme demo")), 420, 260);
scene.getStylesheets().add(lightCss);
var toggle = new Button("Switch theme");
toggle.setOnAction(event -> {
    var styles = scene.getStylesheets();
    styles.setAll(styles.contains(darkCss) ? lightCss : darkCss);
});
((BorderPane) scene.getRoot()).setTop(toggle);
stage.setScene(scene);
stage.show();
```

Define matching semantic style classes in both stylesheets; avoid scattering inline styles through application code.

JavaFX CSS resembles web CSS but is not browser CSS. Selectors and properties are JavaFX-specific, and not every web CSS feature exists.

```css
.root {
    -fx-font-family: "Inter";
    -fx-font-size: 13px;
    -fx-base: #20242b;
}

.primary-button {
    -fx-background-color: #3478f6;
    -fx-text-fill: white;
}
```

```java
button.getStyleClass().add("primary-button");
scene.getStylesheets().add(
    Objects.requireNonNull(getClass().getResource("/theme.css")).toExternalForm()
);
```

Good styling practices:

- use semantic style classes rather than styling every control by ID;
- keep colors, spacing, typography, and control states consistent;
- support focused, disabled, hovered, pressed, selected, and validation states;
- test dark mode, high contrast, font scaling, and localization;
- avoid relying on undocumented skin structure;
- treat CSS resources as versioned application assets.

Third-party themes such as **AtlantaFX**, **JMetro**, and **Ikonli** can accelerate application styling. Confirm their JavaFX and JDK compatibility before adopting them.

## Concurrency and background work

Never block the JavaFX Application Thread with network calls, database queries, filesystem operations, process execution, media preparation, or expensive computation.

JavaFX provides `Task` and `Service` for common background workflows:

```java
Task<String> loadTask = new Task<>() {
    @Override
    protected String call() throws Exception {
        return service.loadText();
    }
};

loadTask.setOnSucceeded(event -> resultLabel.setText(loadTask.getValue()));
loadTask.setOnFailed(event -> showError(loadTask.getException()));
new Thread(loadTask, "data-loader").start();
```

For production applications:

- use a managed `ExecutorService` rather than creating unbounded threads;
- support cancellation and interruption;
- update controls only on the JavaFX Application Thread;
- avoid holding UI references in long-lived background tasks;
- show progress and disable only the controls affected by the operation;
- centralize error reporting and logging;
- shut down executors, media, services, and native resources during application exit.

A `Service` is reusable for repeated requests and exposes observable progress/state:

```java
var loader = new Service<String>() {
    @Override protected Task<String> createTask() {
        return new Task<>() {
            @Override protected String call() throws Exception {
                updateMessage("Loading…");
                return loadData(); // blocking work; check isCancelled() when appropriate
            }
        };
    }
};
var status = new Label();
status.textProperty().bind(loader.messageProperty());
var progress = new ProgressIndicator();
progress.progressProperty().bind(loader.progressProperty());
var cancel = new Button("Cancel");
cancel.setOnAction(event -> loader.cancel());
loader.setOnSucceeded(event -> showData(loader.getValue()));
loader.setOnFailed(event -> reportError(loader.getException()));
loader.start();
```

Configure an executor for the service as appropriate, and call `shutdown()` on application-owned executors during `stop()`. Task handlers run on the FX thread; `Task.call()` runs in the background.

`CompletableFuture`, virtual threads, reactive libraries, and structured concurrency can be used, but their completion callbacks must be marshalled back to the JavaFX thread before touching UI state.

## Modern Java and experimental JavaFX

JavaFX is a useful playground for modern Java—not just a way to place controls on a window. Keep experiments in small, reversible slices, measure before optimizing, and distinguish stable Java SE features from preview or incubator APIs. Preview features require explicit compiler/runtime flags and can change between JDK releases; do not make them a production dependency without a deliberate upgrade plan.

### Language and API experiments

- **Records** make concise immutable view models and table-row values. Keep mutable JavaFX properties in presentation models when binding/editing is required; records are not observable by themselves.
- **Sealed types and pattern matching** can model navigation commands, dialog results, and domain outcomes as closed sets. Convert them to UI state at the presentation boundary.
- **`var`, switch expressions, and text blocks** can reduce ceremony in controllers, test fixtures, and SQL/JSON templates. Prefer explicit types where JavaFX generic inference or event-handler types would otherwise be unclear.
- **Virtual threads** (JDK 21+) can simplify blocking I/O workflows. They do not make CPU-bound work faster, remove cancellation requirements, or permit touching controls off the FX thread. Bound resource-heavy operations and marshal results to the UI thread.
- **Structured concurrency and scoped values** are evolving across JDK releases. Check their status in the exact JDK you target; APIs may be preview/incubator and should be isolated behind application-owned abstractions.
- **Foreign Function & Memory API** (final in JDK 22) enables safer native interop than JNI for many use cases, but native ABI, lifetime, threading, and platform packaging remain your responsibility.

### UI experiments worth building

Try a command palette, keyboard-first navigation, live theme editor, CSS inspector, drag-and-drop workflow canvas, node-graph editor, timeline, data explorer, or a custom control with a documented skin. Use `Canvas` for immediate-mode drawing and `Node`/`Shape` for scene-graph interaction; benchmark the choice with realistic workloads.

For high-throughput views, experiment with virtualized controls, paging, incremental loading, immutable snapshots, and batched observable-list updates. Avoid rebuilding the entire scene graph on every event. Profile pulse/layout/render behavior and memory use before introducing custom rendering or caches.

### Data, AI, and live systems

JavaFX can host dashboards and interactive clients for local models, remote AI services, telemetry, simulations, and streaming data. Keep network/model work in services, expose progress and cancellation, debounce expensive searches, and avoid placing secrets in the client. Treat streamed output as untrusted input, and use bounded queues or coalescing so producers cannot overwhelm the UI event thread.

### A safe experimentation loop

1. State the user-visible hypothesis and define a measurable success criterion.
2. Build a small vertical slice using public JavaFX APIs.
3. Test keyboard, accessibility, resizing, HiDPI, and target operating systems early.
4. Measure startup, responsiveness, memory, and rendering with representative data.
5. Check licensing, maintenance, security, and runtime packaging before adopting dependencies.
6. Record JDK/OpenJFX versions and flags, especially for preview APIs, and keep a fallback path.

Experimental interfaces still need focus handling, accessible names, predictable cancellation, and recoverable errors. Novelty is not a substitute for usability or maintainability.

## Tables, trees, and large data

Tables and trees are common performance hotspots.

- use stable, lightweight cell value factories;
- avoid rebuilding the entire `ObservableList` for small changes;
- use pagination or lazy loading for remote/large datasets;
- avoid expensive work in `updateItem` and cell factories;
- debounce filtering and search fields;
- do not create one heavyweight node per domain object unless required;
- keep formatting and database access out of cell rendering;
- measure sorting, filtering, layout, and CSS costs with realistic data.

For very large or specialized datasets, consider virtualized controls, a custom `VirtualFlow`-based control, or a specialized third-party table.

## Graphics, animation, charts, and media

JavaFX includes:

- 2D shapes, paths, gradients, images, canvas drawing, effects, and transforms;
- transitions, timelines, keyframes, animation timers, and interpolators;
- charts suitable for many business dashboards and monitoring views;
- audio and video playback through `javafx.media`;
- 3D primitives, cameras, lights, materials, meshes, and subscenes.

JavaFX is not a replacement for a dedicated game engine or professional video editor. For intensive rendering, benchmark the target hardware, test GPU fallback behavior, and keep high-frequency updates away from unnecessary layout and CSS recalculation.

## WebView

`javafx.web` embeds a WebKit-based browser component. It is useful for:

- help pages and documentation;
- controlled HTML reports;
- authentication or web-based workflows where embedding is acceptable;
- gradual migration of a legacy web screen.

Consider JCEF, JxBrowser, or an external browser architecture when the application needs a newer browser engine, modern web platform compatibility, developer tools, multiple browser processes, or stronger isolation.

Treat WebView content as untrusted unless proven otherwise. Restrict navigation, validate URLs, control JavaScript bridges, avoid exposing powerful Java objects, and define an update policy for the embedded engine.

## Swing and SWT interoperability

JavaFX can coexist with older desktop code:

- `JFXPanel` embeds JavaFX content in Swing.
- `SwingNode` embeds Swing content in a JavaFX scene.
- SWT interoperability is available through the appropriate JavaFX SWT module/distribution.

Interop introduces multiple event threads and lifecycle rules. Keep the boundary narrow, marshal calls to the correct UI thread, and avoid deeply nesting toolkits. Migration is safer when screens are moved one boundary at a time.

## Native integration and FFI

Most applications should use JavaFX and Java APIs first. Native access is justified for OS-specific features, hardware, high-performance codecs, existing C libraries, or functionality not exposed by JavaFX.

### Java access mechanisms

| Mechanism | Best use | Trade-off |
|---|---|---|
| JavaFX/AWT/Desktop APIs | Windows, files, clipboard, fonts, desktop actions, basic integration | Portable abstraction; not every OS feature is exposed |
| **JNA** | Small and stable C APIs, prototypes, low-frequency calls | Easy to start; callback, layout, and overhead concerns |
| **JNI** | High-throughput calls, existing native code, precise lifecycle control | More boilerplate and difficult debugging |
| **FFM API** | Modern typed access to native symbols and memory segments | Requires a suitable JDK and still needs ABI discipline |
| SWT or maintained bindings | Native widgets or established C/C++ ecosystems | Native artifacts and toolkit-specific packaging |

### FFI rules for JavaFX applications

- isolate FFI behind a service interface; never expose native pointers to controllers or FXML;
- keep native calls off the JavaFX Application Thread unless they are guaranteed non-blocking;
- define ownership for every handle, buffer, callback, and string;
- match C layout, alignment, calling convention, encoding, and architecture exactly;
- keep callbacks alive for as long as native code can call them;
- marshal native callbacks back to the JavaFX thread before changing controls;
- convert native error codes into domain exceptions or result types with context;
- close native resources deterministically with `AutoCloseable` and `try`/`finally`;
- package libraries for every OS/CPU target and verify loading from an installed application;
- test shutdown, cancellation, device removal, library-not-found, and native-crash paths;
- track native licenses, transitive dependencies, CVEs, and update responsibilities.

A C library wrapped by a Rust crate may also be usable from Java, but the Rust wrapper itself is not automatically a Java binding. Java can call the underlying C ABI directly, call a Rust `cdylib` with a stable C interface, or use a higher-level service boundary. Do not depend on Rust-specific ABI or Rust memory layouts across FFI.

### Calling a Rust library from Java

If a Rust implementation is required, expose a small C-compatible API:

```rust
#[no_mangle]
pub extern "C" fn calculate(value: i32) -> i32 {
    value * 2
}
```

Then distribute a platform-specific shared library and declare the ABI through FFM, JNA, or JNI. Prefer opaque handles and explicit `create`/`destroy` functions over sharing Rust structs, `String`, `Vec`, trait objects, or Rust-owned pointers with Java. Define who allocates and frees memory, how strings are encoded, how errors are returned, and whether calls are thread-safe.

For larger integrations, consider a command/service boundary or a generated protocol instead of embedding native code in the UI process. A process boundary can contain crashes and simplify upgrades, at the cost of IPC latency and deployment complexity.

## C/C++ libraries relevant to desktop GUI applications

JavaFX is often sufficient for the UI, while a C or C++ library supplies a specialized native capability. The library may be used directly through FFI, through an existing Java binding, or behind a small native helper process.

| Library/ecosystem | Language | Desktop use | JavaFX integration path |
|---|---|---|---|
| **Qt / Qt Widgets / Qt Quick** | C++ | Full widget toolkit, model/view, graphics, networking, multimedia, and desktop utilities | Qt Jambi for Java Qt applications; otherwise keep Qt as a separate UI or expose selected services through JNI/FFM. Mixing a Qt event loop directly into JavaFX is complex. |
| **GTK 3/4** | C | Native Linux/GNOME widgets, dialogs, accessibility, and desktop integration | GTK Java bindings, SWT, or native FFI. Use JavaFX for the main UI unless native GTK behavior is a requirement. |
| **libadwaita** | C | GNOME design system and adaptive GTK controls | GTK bindings or a native helper; target Linux/GNOME explicitly. |
| **wxWidgets** | C++ | Cross-platform native-looking widgets | wxWidgets Java bindings are uncommon and maintenance varies; use JNI/FFM only when its native behavior is essential. |
| **FLTK** | C++ | Small, portable traditional widget toolkit | FLTK Java bindings or JNA/JNI/FFM; usually choose JavaFX instead for a Java-first application. |
| **JUCE** | C++ | Audio tools, plugins, media applications, custom controls, and cross-platform windows | JavaFX can host the application shell while JUCE runs as a native component/service, but view embedding and licensing require careful review. |
| **Dear ImGui** | C++ | Immediate-mode tools, editors, debug panels, and graphics utilities | Java bindings exist; integrate through LWJGL/JavaCPP or a native surface. It is not a replacement for JavaFX’s accessible business controls. |
| **Nuklear** | C | Small immediate-mode GUI | Native binding or generated FFI; useful for specialized tools, not a general JavaFX control replacement. |
| **SDL2 / SDL3** | C | Windows, input, audio, game controllers, OpenGL/Vulkan/Metal contexts | LWJGL, JavaCPP, JNA/JNI/FFM, or a separate native rendering surface. SDL is not a widget toolkit. |
| **GLFW** | C | Windows, input, and graphics contexts | LWJGL or native FFI. Combine with OpenGL/Vulkan/wgpu-style rendering rather than JavaFX controls. |
| **raylib** | C | Lightweight graphics, games, simulations, and interactive tools | Java bindings or native FFI; use a separate rendering window or embed only with platform-specific work. |
| **Skia** | C++ | 2D rendering, text, images, and GPU-backed drawing | Skija, JavaCPP, LWJGL, or JNI/FFM. Useful for custom JavaFX-adjacent rendering, but it does not provide JavaFX controls. |
| **Cairo / Pango / HarfBuzz** | C | 2D drawing, text layout, shaping, and font processing | Java2D/JavaFX cover common needs; use JavaCPP/JNA/JNI/FFM for exact native behavior. |
| **OpenGL / Vulkan / DirectX / Metal** | C APIs / C++ ecosystems | GPU rendering and compute | LWJGL, JOGL, JavaCPP, or FFM. Keep rendering isolated from ordinary JavaFX controls and benchmark synchronization costs. |
| **WebView2 / WebKitGTK / Chromium / CEF** | C/C++ APIs | Embedded browser engines and web content | JavaFX WebView, JCEF, JxBrowser, SWT Browser, or native FFI. Engine version, sandboxing, licensing, and bundle size matter. |
| **FFmpeg / GStreamer** | C | Audio/video codecs, demuxing, capture, streaming, and transcoding | JavaCV, vlcj, GStreamer Java bindings, JavaCPP, or JNI/FFM. Prefer JavaFX Media only when its supported formats and controls are sufficient. |
| **OpenAL / PortAudio / RtAudio** | C/C++ | Low-level and cross-platform audio | Java Sound or JavaFX Media for ordinary playback; LWJGL/JavaCPP/native bindings for low latency or device control. |
| **libusb / HIDAPI** | C | USB, HID, controllers, scanners, and custom hardware | usb4java, hid4java, JavaCPP, or JNI/FFM; isolate device I/O from the JavaFX thread. |
| **SQLite / PostgreSQL libpq / MySQL client** | C | Embedded or native database access | Prefer JDBC drivers; use native bindings only for a specific extension, performance requirement, or existing native component. |
| **OpenSSL / mbedTLS / libsodium** | C | TLS, cryptography, certificates, and secure primitives | Prefer JSSE, Bouncy Castle, or a maintained Java provider; use native FFI only when compliance or a required primitive demands it. |

### Choosing how to use a C/C++ GUI library

- **Use a Java binding** when it is maintained, supports the target JDK and platforms, and provides the required widget or native API.
- **Use JavaFX plus native services** when JavaFX should own the UI and the C/C++ library only provides codecs, hardware, rendering, or computation.
- **Use a native child process** when crash containment, an existing C++ application, or incompatible event loops make in-process FFI risky.
- **Use a custom JavaFX `Node` or `Canvas` bridge** only when input, focus, resizing, GPU context ownership, and accessibility are fully defined.
- **Avoid mixing event loops casually:** Qt, GTK, SDL, JUCE, WebKit, and JavaFX each have lifecycle and thread-affinity rules.
- **Check licenses and redistribution:** Qt, JUCE, Chromium/CEF, FFmpeg, codecs, GPU drivers, and platform SDKs can impose materially different obligations.

A C/C++ library is not automatically a JavaFX competitor. Qt, GTK, wxWidgets, FLTK, and JUCE are GUI platforms; SDL, GLFW, Skia, Cairo, OpenGL, Vulkan, FFmpeg, and libusb are supporting technologies. JavaFX can replace the widget layer while using the latter group behind carefully designed Java services.

## Packaging and deployment

JavaFX is a runtime dependency and must be included in the delivered application.

### Recommended build flow

1. Pin a supported JDK and JavaFX version.
2. Declare JavaFX modules with Maven or Gradle.
3. Run tests on every target operating system and architecture.
4. Build a runtime image with `jlink` when a reduced runtime is useful.
5. Create installers with `jpackage` or a maintained packaging tool.
6. Sign binaries and installers; notarize macOS applications.
7. Test the installed application, not only the IDE run configuration.
8. Provide crash reporting, logging, update, rollback, and uninstall behavior.

A typical Maven dependency pattern is:

```xml
<properties>
    <javafx.version>YOUR_JAVAFX_VERSION</javafx.version>
</properties>

<dependency>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-controls</artifactId>
    <version>${javafx.version}</version>
</dependency>
```

Add `javafx-fxml`, `javafx-web`, `javafx-media`, or other modules only when needed. Maven and Gradle plugins can select platform-specific artifacts; keep the exact plugin configuration aligned with the selected JDK and JavaFX release.

### JPMS and runtime images

Modules are optional for JavaFX projects but useful for strong encapsulation and smaller runtime images. A modular application typically needs:

- `requires javafx.controls;`
- `requires javafx.fxml;` when using FXML;
- `opens package.name to javafx.fxml;` for reflective controller and field access;
- `exports` only for packages that form a public module API.

Use `jdeps`, `jlink`, and `jpackage` as part of release automation. Verify reflection, FXML resources, service loading, fonts, native libraries, and WebView/media modules in the final image.

## Testing and diagnostics

### Test application logic separately from the UI

Keep business logic in ordinary Java classes so it can be tested without starting the JavaFX toolkit:

```java
final class NameValidator {
    boolean isValid(String name) {
        return name != null && name.strip().length() >= 3;
    }
}
```

```java
@Test
void rejectsShortNames() {
    var validator = new NameValidator();
    assertFalse(validator.isValid("Al"));
    assertTrue(validator.isValid("Ada"));
}
```

For tests that construct controls, initialize JavaFX once using the test framework's supported JavaFX launcher/extension, and perform UI access on the FX thread. Avoid arbitrary sleeps; wait for explicit conditions and close windows/resources after each test. Headless execution may require a virtual display depending on the platform and test tooling.

### Minimal scene navigation

For a small application, replace the root of one `Scene` rather than creating an unmanaged window for every view:

```java
var scene = new Scene(new Label("Home"), 480, 300);
var home = new Button("Home");
var settings = new Button("Settings");
var navigation = new HBox(8, home, settings);
var root = new BorderPane(new Label("Home"));
root.setTop(navigation);
scene.setRoot(root);
home.setOnAction(event -> root.setCenter(new Label("Home")));
settings.setOnAction(event -> root.setCenter(new CheckBox("Enable option")));
stage.setScene(scene);
stage.show();
```

For larger apps, use a navigation service or view factory and define how view state, focus, and resources are preserved or disposed.

### Save a file safely

```java
var save = new Button("Save…");
save.setOnAction(event -> {
    var file = new FileChooser().showSaveDialog(save.getScene().getWindow());
    if (file == null) return;
    try {
        Files.writeString(file.toPath(), textArea.getText(), StandardCharsets.UTF_8);
    } catch (IOException ex) {
        new Alert(Alert.AlertType.ERROR, "Could not save file: " + ex.getMessage()).showAndWait();
    }
});
```

For important data, consider writing to a temporary file and moving it into place, handling overwrite confirmation, and running the operation off the FX thread for large content.


Keep most application logic testable without starting JavaFX. Then add focused UI and packaging tests.

- **Unit tests:** domain logic, validation, view models, converters, bindings, and services.
- **JavaFX-thread tests:** use a framework such as TestFX or Jemmy and respect toolkit startup rules.
- **Visual tests:** screenshots or golden images for important themes and layouts, with controlled fonts and DPI.
- **Accessibility tests:** keyboard-only navigation, focus order, screen readers, scaling, contrast, and accessible text.
- **Integration tests:** files, databases, HTTP, WebView bridges, media, native devices, and shutdown.
- **Release tests:** installers, clean machines, offline mode, upgrades, signing, and missing optional libraries.

Use Java Flight Recorder, VisualVM, async-profiler, and OS-level profilers to investigate startup, memory, blocked threads, rendering, and native calls. Do not infer JavaFX performance from a debug build or an empty scene.

## Recommended project structure

```text
src/main/java/com/example/app/
  App.java                 # Application lifecycle
  ui/                      # FXML/controllers or programmatic views
  viewmodel/               # JavaFX-facing observable state
  domain/                  # Toolkit-independent business model
  service/                 # I/O, database, HTTP, and background work
  platform/                # OS and FFI adapters
src/main/resources/
  com/example/app/
    views/
    styles/
    images/
    i18n/
```

Keep `domain` independent of JavaFX where practical. Put JavaFX properties and bindings in view models, and keep platform-specific code behind interfaces.

## Ecosystem

Useful categories to evaluate:

- **Themes:** AtlantaFX, JMetro, Ikonli.
- **Layout and forms:** MigLayout JavaFX and application-specific layout components.
- **Charts and dashboards:** ControlsFX, Medusa, ChartsFX, and specialized chart libraries.
- **Rich text and editors:** RichTextFX and domain-specific editors.
- **Docking and workbenches:** DockFX and commercial docking suites.
- **Testing:** TestFX, Jemmy, JUnit, AssertJ, and screenshot-test tooling.
- **Interop and graphics:** LWJGL, JOGL, JNA, JNI, FFM, and JavaFX Swing/SWT bridges.
- **Deployment:** `jlink`, `jpackage`, Maven/Gradle plugins, signing tools, and vendor support.

Evaluate maintenance, license, JavaFX compatibility, accessibility, native classifiers, documentation, and issue activity before adding a library.

## Potential ideas for a reusable JavaFX library

A reusable JavaFX library can focus on common application problems rather than only adding isolated controls:

### UI and layout

- reusable cards, badges, banners, empty states, loading views, and error views;
- responsive layouts, form layouts, sidebars, navigation bars, split views, and docking panels;
- enhanced tables and lists with filtering, sorting, pagination, inline editing, and export;
- wizards, settings screens, dashboards, galleries, and application shells.

### Forms and state

- form validation, validation summaries, masked inputs, converters, and undo/redo;
- view-model base classes, observable state utilities, commands, actions, and navigation/routing;
- event buses, dependency-injection helpers, preferences, and persisted application state;
- background-task helpers with progress, cancellation, retry, and JavaFX-thread dispatching.

### Reactive state and signals

A reactive module could provide a small, JavaFX-friendly abstraction for state, derived values, effects, and event streams:

- writable signals for local and shared application state;
- computed signals for derived values and memoized calculations;
- effects that react to changes and clean themselves up with a view lifecycle;
- bindings between signals and JavaFX properties in both directions;
- observable collections with incremental list, map, and set updates;
- event streams for commands, user actions, notifications, and application events;
- operators for mapping, filtering, combining, debouncing, throttling, buffering, and error recovery;
- asynchronous signals for loading, success, error, cancellation, and retry states;
- scheduling on the JavaFX Application Thread, background executors, or virtual threads;
- lifecycle-aware subscriptions that prevent listeners and tasks from leaking;
- adapters for `ObservableValue`, `Property`, `ObservableList`, `Task`, `Service`, and `CompletableFuture`;
- optional integrations for Reactor, RxJava, Mutiny, or Project Loom without making them mandatory dependencies.

A minimal API might look like this:

```java
Signal<String> name = Signals.state("");
Signal<String> greeting = Signals.computed(
    () -> "Hello, " + name.get()
);

Subscription subscription = greeting.subscribe(label::setText);
name.set("Ada");
subscription.close();
```

The library should define clear rules for equality, scheduling, errors, re-entrant updates, lazy versus eager computation, and subscription disposal. UI effects must run on the JavaFX Application Thread, while expensive work should remain outside it.

### Interaction with JavaFX and the Java standard library

The reactive layer should complement JavaFX primitives and standard Java APIs instead of replacing them:

| Library concept | JavaFX companion | Java standard-library companion |
|---|---|---|
| Signal | `Property`, `ReadOnlyProperty`, `ObservableValue`, and `Binding` | ordinary fields, records, and immutable value objects |
| Reactive collection | `ObservableList`, `ObservableSet`, and `ObservableMap` | `List`, `Set`, `Map`, streams, and collection views |
| Event stream | `EventHandler`, `Event`, and control events | `Flow.Publisher`, `Consumer`, and `Function` |
| Async state | `Task`, `Service`, and progress properties | `CompletableFuture`, `Executor`, `ExecutorService`, and virtual threads |
| Subscription cleanup | scene/control lifecycle and `Node` disposal hooks | `AutoCloseable`, `Cleaner`, and structured task scopes |
| Scheduling | `Platform.runLater` and the JavaFX Application Thread | `ScheduledExecutorService` and `Executor` |
| Time and cancellation | animation and task APIs | `Duration`, `Instant`, `Clock`, and `Future.cancel` |
| Errors | alert, validation, and error views | exceptions, result types, and structured logging |

Useful adapters could make the boundary explicit:

```java
Signal<String> name = Signals.state("");
StringProperty nameProperty = FxSignals.asStringProperty(name);

ObservableList<Item> items = FxSignals.asObservableList(itemSignal);
CompletableFuture<Result> future = service.loadAsync();
Signal<AsyncState<Result>> result = Signals.fromFuture(future);
```

Recommended rules:

- expose JavaFX properties at the UI boundary and plain Java types in the core;
- provide adapters rather than silently wrapping or duplicating JavaFX state;
- guarantee that UI updates are marshalled to the JavaFX Application Thread;
- allow background work to use standard executors, `CompletableFuture`, or virtual threads;
- make subscription, task, and listener ownership explicit with `AutoCloseable` scopes;
- preserve JavaFX observable semantics such as invalidation, change notifications, and list diffs;
- avoid forcing applications to adopt a third-party reactive framework for basic signals;
- support interoperation with `Flow.Publisher`, Reactor, and RxJava through optional modules.

### Companions for the other library areas

The same boundary should apply to the rest of the library, not only reactive state:

| Library area | JavaFX primitives to integrate with | Java standard-library companions |
|---|---|---|
| UI components | `Node`, `Parent`, `Region`, `Control`, `Skin`, and `PseudoClass` | records for immutable configuration, collections, and `Objects` validation |
| Layout and navigation | `Pane`, `VBox`, `GridPane`, `BorderPane`, `Scene`, `Stage`, and `TabPane` | `URI`, `Optional`, and application-defined navigation commands |
| Forms and validation | `TextInputControl`, `ComboBox`, `DatePicker`, `ValidationSupport`, and properties | `Predicate`, `Function`, `Pattern`, `Locale`, `Formatter`, and `Optional` |
| Tables and data | `TableView`, `TreeView`, `ListView`, `Cell`, and `ObservableList` | `List`, `Map`, streams, `Comparator`, pagination services, and immutable DTOs |
| Styling and themes | CSS, `Styleable`, style classes, pseudo-classes, and `Scene` stylesheets | resource bundles, `Resource`, `Color`, and configuration properties |
| Dialogs and notifications | `Alert`, `Dialog`, `Popup`, `Tooltip`, and `Notification` views | exceptions, result types, `Logger`, and `ResourceBundle` |
| Charts and visualization | `Chart`, `Axis`, `XYChart`, `Canvas`, `Shape`, and animation APIs | numeric types, streams, `DoubleSummaryStatistics`, and time-series models |
| Media and content | `ImageView`, `MediaPlayer`, `WebView`, and `WebEngine` | `Path`, `Files`, `URI`, `InputStream`, and `Duration` |
| Platform integration | `Clipboard`, `FileChooser`, `DirectoryChooser`, `Robot`, and `HostServices` | `Path`, `Files`, `Desktop`, `ProcessBuilder`, environment properties, and `Preferences` |
| Background work | `Task`, `Service`, `Worker`, and progress properties | `Executor`, `CompletableFuture`, `Future`, `Flow`, and virtual threads |
| Testing and diagnostics | TestFX/Jemmy fixtures, scenes, nodes, and CSS state | JUnit, assertions, `Logger`, JFR, and standard test resources |
| Native integration | `SwingNode`, `JFXPanel`, `Canvas`, and custom `Node` bridges | FFM, `AutoCloseable`, `ByteBuffer`, `MemorySegment`, and `ServiceLoader` |
| Packaging and configuration | module resources, FXML, CSS, images, and services | JPMS, `ResourceBundle`, `Properties`, `ServiceLoader`, `jlink`, and `jpackage` |

Design each feature with three layers:

```text
core       # Java types, services, validation, persistence, and business rules
javafx     # Controls, views, properties, CSS, dialogs, and thread adapters
integrations # Optional charts, media, native, RxJava, database, or platform modules
```

For example, a reusable file-picker service can return a `Path` from the core-facing API, use `FileChooser` in its JavaFX adapter, and use `Files` for reading and writing. A table component can accept plain immutable records while exposing JavaFX cell factories and observable updates at the UI boundary.

### Styling and user experience

- light and dark themes, design tokens, spacing and typography utilities;
- runtime theme switching and custom CSS pseudo-classes;
- toast notifications, confirmation dialogs, progress dialogs, and exception views;
- focus management, keyboard navigation, accessible labels, and high-contrast support;
- localization, RTL layouts, and date, number, and currency formatting.

### Data, media, and visualization

- KPI cards, sparklines, gauges, timelines, heat maps, and real-time dashboards;
- Markdown, rich-text, PDF, image-gallery, audio, video, and WebView components;
- REST, WebSocket, database, file, clipboard, printing, and drag-and-drop helpers;
- serial, USB, system-information, and other platform-service adapters.

### Testing, tooling, and deployment

- TestFX fixtures, JavaFX-thread utilities, screenshot testing, and component test helpers;
- CSS inspection, debug overlays, performance metrics, and diagnostic views;
- logging, crash reporting, configuration, update, `jlink`, and `jpackage` helpers.

A practical library structure is:

```text
your-library-core       # Java-only utilities and abstractions
your-library-javafx     # Controls, bindings, views, and CSS
your-library-icons      # Optional icons and assets
your-library-charts     # Optional visualization features
your-library-platform   # Optional OS/native integrations
your-library-test       # Testing utilities
```

Keep `core` independent of JavaFX where practical. Make JavaFX, charts, native integrations, and testing support optional modules so applications can depend only on the capabilities they need.

## Strengths and limitations

### Strengths

- coherent scene graph, controls, styling, animation, charts, media, and WebView;
- strong property/binding model for state-driven desktop interfaces;
- Java tooling, libraries, profiling, and enterprise integration;
- cross-platform rendering without requiring a full browser for native screens;
- practical packaging through custom runtimes and native installers;
- suitable for forms, dashboards, editors, internal tools, and rich client software.

### Limitations

- JavaFX is distributed separately from the JDK;
- the third-party ecosystem is smaller than web, Swing, or Android ecosystems;
- native controls and platform conventions are not identical across operating systems;
- accessibility, IME, printing, WebView, and media behavior still require target-platform testing;
- large tables, embedded browsers, and custom rendering need profiling;
- deployment must include compatible JavaFX modules and, where applicable, native libraries.

## Practical checklist

Before choosing JavaFX for a long-lived application, verify:

- required controls, charts, editors, printing, and docking support;
- target JDK, operating systems, CPU architectures, and GPU requirements;
- keyboard, screen-reader, high-contrast, RTL, IME, and font-scaling behavior;
- background task, cancellation, offline, and crash-recovery design;
- WebView engine requirements and security policy;
- native libraries, FFI ownership, licenses, and update responsibility;
- clean installation, signing, notarization, updates, rollback, and uninstall;
- team familiarity with JavaFX properties, CSS, FXML, modules, and the Application Thread.

The most useful proof of suitability is a vertical slice of the hardest screen, running from a packaged installer on every target platform.

## When JavaFX is not the right fit

Consider another approach when:

- the application must use fully native platform widgets and behavior;
- the product is primarily a web application and already has a mature web frontend;
- the UI is a game or high-end 3D engine workload;
- a vendor-supported Qt, GTK, or platform-native stack is a hard requirement;
- an extremely small native binary or no JVM runtime is mandatory.

For those cases, evaluate the alternative on the exact requirements rather than assuming that a different language automatically provides a better desktop experience.
