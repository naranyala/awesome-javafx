# Awesome JavaFX

A focused guide to **JavaFX** desktop development: its UI model, modules, styling, architecture, packaging, testing, native integration, ecosystem, and long-term maintenance.

JavaFX is a Java-based desktop toolkit for building rich cross-platform applications. It combines a retained-mode scene graph, controls, CSS, property binding, FXML, charts, media, WebView, animation, and graphics APIs in one coherent platform.

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

JavaFX includes controls for most conventional desktop applications:

- buttons, toggle buttons, check boxes, radio buttons, sliders, spinners, and date pickers;
- text fields, password fields, formatted fields, text areas, and rich text building blocks;
- list views, tree views, table views, tree tables, and editable cells;
- menus, toolbars, context menus, dialogs, alerts, pagination, tabs, accordions, and split panes;
- charts including line, area, bar, bubble, scatter, pie, and stacked variants;
- progress indicators, tooltips, separators, scroll panes, and custom controls.

Common layout containers include `BorderPane`, `HBox`, `VBox`, `GridPane`, `FlowPane`, `TilePane`, `StackPane`, `AnchorPane`, `ScrollPane`, and `GridPane`-based form layouts.

Prefer layout panes and constraints over fixed coordinates. Test with resizing, font scaling, localization, accessibility settings, and different DPI factors.

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

`CompletableFuture`, virtual threads, reactive libraries, and structured concurrency can be used, but their completion callbacks must be marshalled back to the JavaFX thread before touching UI state.

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
