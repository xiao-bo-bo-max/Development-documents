## QT多个ui文件使用
https://blog.csdn.net/liangyunshan123/article/details/134020052
这段代码需不需要delete 其他ui？
有一个Qt项目，存在一个UI文件，最顶层QWidget的objectName为MainWindow。现在我新增一个UI文件，修改最顶层QWidget的objectName为SessionList。如何定义SessionList类，并将其加入到MainWindow布局中
## 前向声明
在Qt中，当你使用Qt Designer设计界面并生成对应的UI类时，通常会在主窗口的头文件中看到这样的代码：

```cpp
namespace Ui {
    class MainWindow;
}
```

这段代码的作用是声明一个命名空间`Ui`，并在其中声明一个前向声明的类指针`MainWindow`。这个类是自动生成的，它包含了所有UI组件的信息和行为。这样做有几个目的：

1. **前向声明**：这个声明允许你在包含 `ui_MainWindow.h` 之前使用 `Ui::MainWindow*` 类型的指针。这在某些情况下非常有用，比如当你需要在主窗口类中声明一个指向UI类的指针，但你还不想立即包含整个UI类的定义，因为这可能会导致不必要的编译依赖和编译时间增加。

2. **代码清晰和组织**：通过在主窗口头文件中声明这个命名空间和类，你可以清晰地表明这个类是由UI工具自动生成的，并且与主窗口类紧密相关。这有助于其他开发者理解代码结构和组织。

3. **编译效率**：如果你在主窗口类中直接包含 `ui_MainWindow.h`，那么每次主窗口类被包含时，都会强制编译器处理整个UI类的定义。通过使用前向声明，你可以减少这种依赖，只有在真正需要UI类定义的地方（如在主窗口类的实现文件中）才包含 `ui_MainWindow.h`。

4. **避免循环依赖**：在某些复杂的项目中，可能会有多个类相互依赖。通过使用前向声明，你可以减少或避免循环依赖的问题，这在大型项目中尤为重要。

5. **灵活性**：有时候，你可能需要在不同的上下文中使用UI类，而不是直接在主窗口类中使用。通过在主窗口头文件中声明这个类，你可以在不改变UI类定义的情况下，灵活地在不同的类或函数中使用UI类。

总之，虽然直接包含 `ui_MainWindow.h` 可以工作，但使用前向声明提供了更多的灵活性和控制，有助于提高代码的可维护性和编译效率。
# ui->setupUi(this)解析
在Qt框架中，`ui->setupUi(this)` 是一个非常重要的调用，它的作用是将由Qt Designer设计的用户界面(UI)布局和控件映射到你的应用程序窗口中。具体来说，这个函数执行以下几个关键步骤：

1. **初始化UI组件**：`setupUi` 函数会根据 `ui_MainWindow.h` 文件中定义的 `Ui::MainWindow` 类的布局和控件，初始化你的主窗口中的所有UI组件。这包括创建按钮、文本框、标签等，并按照设计时的布局将它们放置在窗口中。

2. **设置父对象**：`this` 参数是指向当前窗口对象的指针。通过传递 `this` 给 `setupUi` 函数，你告诉所有的UI组件它们的父对象是谁。在Qt中，控件的父对象负责管理其子控件的生命周期，包括内存管理、事件处理等。

3. **连接信号和槽**：虽然 `setupUi` 函数本身不直接连接信号和槽，但它会设置好UI组件，使得你可以在代码中轻松地连接这些组件的信号和槽。例如，你可以连接按钮的 `clicked` 信号到一个槽函数，当按钮被点击时，槽函数会被调用。

4. **设置初始属性**：`setupUi` 还会设置UI组件的初始属性，如文本、图标、大小等，这些都是你在Qt Designer中设置的。

5. **启用动态特性**：对于某些控件，`setupUi` 还会启用一些动态特性，比如工具栏的可停靠性、窗口的可拖动性等。

通常，`ui->setupUi(this)` 这行代码会在主窗口类的构造函数中被调用，确保在窗口对象被创建时，UI组件也被正确初始化。例如：

```cpp
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    ui = new Ui::MainWindow();
    ui->setupUi(this);
    // 其他初始化代码...
}
```
在这个例子中，`ui` 是一个指向 `Ui::MainWindow` 类实例的指针，它在构造函数中被创建，并调用 `setupUi` 函数来初始化UI。这样，当主窗口显示时，用户就会看到由Qt Designer设计的用户界面。
## delete ui解析
在Qt中，`ui->setupUi(this);` 确实设置了UI组件的父对象，使得这些组件在逻辑上属于主窗口，并且它们的生命周期与主窗口的生命周期绑定。然而，`delete ui;` 在析构函数中的使用并不是为了删除UI组件，而是为了删除 `Ui::MainWindow` 类的实例本身。

这里是详细解释：

1. **UI组件的父对象**：当你调用 `ui->setupUi(this);` 时，所有的UI组件（如按钮、标签等）都会被设置为主窗口 (`this`) 的子对象。这意味着这些组件的内存管理是由主窗口负责的，当主窗口被销毁时，这些子组件也会自动被销毁。

2. **`Ui::MainWindow` 实例**：`ui` 指针指向的是一个 `Ui::MainWindow` 类的实例。这个实例包含了所有UI组件的数据和方法，但它本身并不是一个QWidget，而是一个普通的C++对象。因此，它的生命周期并不自动与主窗口的生命周期绑定。

3. **资源回收**：当主窗口被销毁时，由于UI组件是主窗口的子对象，它们会被自动销毁。但是，`Ui::MainWindow` 类的实例本身并不是主窗口的子对象，所以它不会被自动销毁。因此，你需要在主窗口的析构函数中手动删除这个实例，以避免内存泄漏。

例如：

```cpp
MainWindow::~MainWindow()
{
    delete ui;  // 删除Ui::MainWindow实例
}
```

这里，`delete ui;` 会释放 `Ui::MainWindow` 实例所占用的内存。如果不这样做，当主窗口被销毁时，`Ui::MainWindow` 实例所占用的内存不会被释放，从而导致内存泄漏。

总结来说，`ui->setupUi(this);` 设置了UI组件的父对象，确保了这些组件的生命周期与主窗口的生命周期绑定。但是，`Ui::MainWindow` 实例本身并不是一个UI组件，也不是主窗口的子对象，所以它的生命周期需要手动管理。这就是为什么需要在析构函数中使用 `delete ui;` 的原因。
## [ Qt项目打包](https://blog.csdn.net/m0_73633088/article/details/143131805)

# Inno Setup

教程：

[Inno Setup脚本语法大全 - littlejoean - 博客园](https://www.cnblogs.com/joean/p/4842707.html)

[Inno Setup 3 ：语法解析（二）_开源，分享，进步的技术博客_51CTO博客](https://blog.51cto.com/weiyuqingcheng/2132799)

## 安装程序的多语言支持

```ini
[Languages]
Name: "english"; MessagesFile: "compiler:Default.isl"
Name: "chinesesimplified"; MessagesFile: "compiler:Languages\ChineseSimplified.isl"
```

`.iss` 文件是 Inno Setup 脚本文件，Inno Setup 是一个免费的安装程序制作工具，用于创建 Windows 应用程序的安装程序。在 `.iss` 文件中，`Name` 和 `MessagesFile` 是两个参数，它们通常用于定义安装程序的多语言支持。

- `Name: "english";` 表示这是英语版本的安装程序。`Name` 参数用于标识语言包的名称。

- `MessagesFile: "compiler:Default.isl";` 表示安装程序将使用 `Default.isl` 文件中定义的消息和文本。`MessagesFile` 参数指定了一个 ` isl` 文件，这个文件包含了安装过程中显示的所有文本消息。

**compiler前缀：**

在 Inno Setup 的 `.iss` 文件中，`compiler:` 是一个特殊的前缀，用于指示 Inno Setup 编译器将指定的资源直接嵌入到生成的安装程序文件中，而不是作为外部文件引用。

当使用 `compiler:` 前缀时，Inno Setup 会将指定的文件（如 `MessagesFile` 参数中的 `Default.isl`）的内容直接编译到安装程序的可执行文件中。这样做的好处是：

1. **减少外部依赖**：安装程序不需要依赖外部的 ` isl` 文件，因为所有必要的文本和消息都已经包含在安装程序内部。
2. **便于分发**：用户在安装时不需要额外下载或寻找语言文件，因为所有内容都已经集成在安装程序中。
3. **提高安全性**：通过将文件编译到安装程序中，可以减少外部文件被篡改的风险。

**MessagesFile查找路径：**

在 Inno Setup 的 `.iss` 文件中，`MessagesFile` 的路径是相对于 Inno Setup 的安装目录来查找的。当你使用 `compiler:` 前缀时，Inno Setup 编译器会在编译过程中将指定的 `.isl` 文件嵌入到生成的安装程序中。

例如，如果你的 Inno Setup 安装在默认路径 `C:\Program Files (x86)\Inno Setup 6`，并且你的 `.iss` 文件中有这样的配置：

```ini
[Languages]
Name: "english"; MessagesFile: "compiler:Default.isl"
Name: "chinesesimplified"; MessagesFile: "compiler:Languages\ChineseSimplified.isl"
```

Inno Setup 编译器会从 `C:\Program Files (x86)\Inno Setup 6\Languages` 目录开始查找 `Default.isl` 和 `ChineseSimplified.isl` 文件。

如果你没有使用 `compiler:` 前缀，而是提供了一个完整的文件路径，那么 Inno Setup 会直接从那个指定的路径查找文件。但是，通常情况下，我们使用 `compiler:` 前缀来确保语言文件被嵌入到安装程序中，这样安装程序就不需要依赖于外部的语言文件。

如果你添加了自定义语言文件，你需要将这些文件放在 Inno Setup 安装目录下的 `Languages` 文件夹中，或者在 `.iss` 文件中提供这些文件的完整路径。然后，通过 `MessagesFile` 指定这些文件，Inno Setup 编译器在编译安装程序时会将它们嵌入进去。

### 添加语言选择功能

要在Inno Setup的安装包中添加语言选择功能，并确保安装后软件显示对应的语言，你需要在.iss脚本文件中进行以下配置：

1. **启用语言选择对话框**：在`[Setup]`段中添加`ShowLanguageDialog=yes`指令。这会显示一个语言选择对话框，让用户在安装开始前选择语言。这是确保用户可以选择安装语言的关键配置。
   
   ```ini
   [Setup]
   ShowLanguageDialog=yes
   ```

2. **定义支持的语言和对应的消息文件**：在`[Languages]`段中定义你希望支持的语言及其对应的`.isl`消息文件。每个语言条目都由`Name:`和`MessagesFile:`组成，分别指定语言代码和消息文件的路径。
   
   ```ini
   [Languages]
   Name: "english"; MessagesFile: "compiler:Default.isl"
   Name: "chinesesimplified"; MessagesFile: "compiler:Languages\ChineseSimplified.isl"
   ```
   
   在这个例子中，`Default.isl`是英文消息文件，而`ChineseSimplified.isl`是简体中文消息文件。`compiler:`前缀表示文件被编译到安装程序中。

3. **保存并编译脚本**：保存你的.iss脚本文件，然后使用Inno Setup编译器编译它以生成安装程序。

4. **测试安装程序**：运行生成的安装程序以测试语言选择功能是否正常工作。

通过以上步骤，你可以让用户在安装过程中选择语言，并且确保安装后的软件界面显示为所选语言。这样，你就可以为用户提供多语言支持，提升用户体验。

## Build & Run

Inno Setup 提供了两种主要的操作来处理 `.iss` 文件：编译（Build）和运行（Run）。这两种操作的区别在于它们的执行环境和目的：

1. **编译（Build）**：
   
   - **目的**：编译（或构建）`.iss` 文件是为了生成一个安装程序（通常是一个 `.exe` 文件），这个安装程序可以在任何目标计算机上运行以安装软件。
   - **执行环境**：当你编译 `.iss` 文件时，Inno Setup 编译器读取脚本文件，将其转换为一个独立的可执行安装程序。这个过程中，编译器会处理 `.iss` 文件中的所有指令，包括文件复制、注册表操作、创建快捷方式等，并将其打包成一个单一的可执行文件。
   - **结果**：编译操作完成后，你会得到一个 `.exe` 文件，这个文件包含了安装程序的所有内容和指令，可以分发给用户进行安装。

2. **运行（Run）**：
   
   - **目的**：运行 `.iss` 文件通常是为了在当前计算机上直接执行安装程序，而不是编译生成一个新的安装程序。
   - **执行环境**：当你运行 `.iss` 文件时，Inno Setup 编译器会直接执行 `.iss` 文件中的脚本，而不是生成一个新的 `.exe` 文件。这意味着所有的安装操作（如文件复制、注册表修改等）都会立即在当前系统上执行。
   - **结果**：运行操作完成后，如果 `.iss` 文件中包含了安装指令，那么这些指令会被执行，软件会被安装到你的系统上。这通常用于测试安装脚本，确保一切按预期工作。

总结来说，编译 `.iss` 文件是为了生成一个可分发的安装程序，而运行 `.iss` 文件则是直接执行安装操作。在实际使用中，你会先编译 `.iss` 文件生成安装程序，然后分发给用户；在开发和测试阶段，你可能会直接运行 `.iss` 文件来测试安装过程是否正确。

## MsgBox

```pascal
MsgBox(ExpandConstant('{cm:AlreadyInstalled}')+ #10#10, mbConfirmation, MB_YESNO)
```

在 Inno Setup 的 `MsgBox` 函数中：

- `#10` 表示换行符，它在 Pascal 字符串中用来插入一个新行。这与许多编程语言中的 `\n` 相似。

- `mbConfirmation` 是一个消息框类型常量，它指定消息框的风格。`mbConfirmation` 表示消息框将显示“是（Y）”和“否（N）”按钮，让用户进行确认选择。

- `MB_YESNO` 是一个按钮常量，它指定消息框中显示哪些按钮。`MB_YESNO` 表示将显示“是（Yes）”和“否（No）”两个按钮。

- `ExpandConstant` 函数用于展开常量，这些常量可以是 Inno Setup 预定义的常量，也可以是自定义常量。`'{cm:...}'` 是一种特殊格式，用于引用编译时或运行时的消息，这些消息通常用于界面元素，如按钮文本或提示信息。

## .isl常量定义

在 Inno Setup 的 `.isl` 文件中，如果你想要为消息添加换行，可以使用 `%n` 来表示一个新行。`%n` 是一个特殊的序列，Inno Setup 在处理消息时会将其替换为当前操作系统的换行符。

```ini
[CustomMessages]
AlreadyInstalled=安装程序检测到 HATWebCtrlPlugin 已经安装。%n%n单击“是(Y)”重新安装程序%n单击“否(N)”退出安装。
RunningProgram=程序正在运行中。%n%n单击“是(Y)”程序退出，继续卸载%n单击“否(N)”退出卸载。
```

在上面的例子中，`%n%n` 被用来在消息中插入两个换行符，这样在显示消息时，警告或提示信息会在指定的位置换行。当 Inno Setup 显示这些消息时，`%n` 会被转换成适当的换行符，从而在用户界面上正确地显示多行文本。

## 常见设置

### 开启日志

```pascal
[setup]
//打开日志功能
SetupLogging=yes
```

开启后日志将默认生成在C:\Users\用户名\AppData\Local\Temp目录下并以类似Setup Log 2024-10-29 #001.txt命名。
