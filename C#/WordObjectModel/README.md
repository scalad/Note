### Word对象模型 ###

#### 一、开发环境布置 ####
C#中添加对Word的支持，只需添加对Microsoft.Office.Interop.Word的命名空间，如下图所示，右键点击“引用”，在弹出的“添加引用”对话框中选中COM标签页，找到“Microsoft Word 12.0 Object Library”。

![](https://github.com/scalad/Note/blob/master/C%23/WordObjectModel/image/20161109112442139.png)

点击确定按钮后，可在引用中添加显示名称为Microsoft.Office.Interop.Word的引用：

![](https://github.com/scalad/Note/blob/master/C%23/WordObjectModel/image/20161109112637339.png)

#### 二、Word的对象模型介绍 ####
Word中共有5种常用的对象模型：应用程序对象Application、文档对象Document、Selection对象、Range对象和Bookmark对象。
下图显示了 Word 对象模型层次结构中这些对象的一个视图。

![](https://github.com/scalad/Note/blob/master/C%23/WordObjectModel/image/20161109112800937.png)

初看起来，对象似乎重叠在一起。 例如，Document 和 Selection 对象都是 Application 对象的成员，但 Document 对象也是 Selection 对象的成员。 Document 和 Selection 对象都包含 Bookmark 和 Range 对象。 因为有多种方法可以访问相同类型的对象，所以存在重叠。 例如，你将格式设置应用于 Range 对象；但你可能想要访问当前选定内容、某一特定段落，某一节或整个文档的范围。

下面分别介绍五种模型对象的含义和作用。

#### 2.1 Applicatin对象。 ####
Application 对象表示 Word 应用程序，并且是所有其他对象的父级。 其成员通常作为一个整体应用于 Word。 你可以使用其属性和方法来控制 Word 环境。

在文档级项目中，可以通过使用 ThisDocument 类的 Application 属性来访问 Application 对象。

#### 2.2 Document对象 ####
`Microsoft.Office.Interop.Word.Document` 对象是 Word 编程的中心。 它表示一个文档及其所有内容。 当你打开文档或创建新文档时，将创建新的 `Microsoft.Office.Interop.Word.Document` 对象，并将其添加到 Application 对象的 T:`Microsoft.Office.Interop.Word.Documents` 集合。 具有焦点的文档被称为活动文档。 它由 Application 对象的 P:`Microsoft.Office.Interop.Word._Application.ActiveDocument` 属性表示。

#### 2.3 Selection对象 ####
Selection 对象表示当前所选的区域。 在 Word 用户界面中执行操作（如文本加粗）时，可以选择或突出显示文本，然后应用格式设置。 文档中始终存在 Selection 对象。 如果未选中任何内容，则它表示插入点。 此外，选定内容可包含多个不相邻的文本块。

#### 2.4 Range对象 ####
Range 对象表示文档中的相邻区域，并由起始字符位置和结束字符位置进行定义。 并不仅限于单个 Range 对象。 你可以在同一文档中定义多个 Range 对象。 Range 对象具有以下特性：

* 它可以只包含单独的插入点，也可包含一个文本范围或整个文档。
* 它包括非打印字符，如空格、制表符和段落标记。
* 它可以是当前选定内容所表示的区域，也可以表示不同于此内容的区域。
* 它在文档中不可见，这与选定内容不同，后者总是可见。
* 它不随文档一起保存，且仅在代码运行时才存在。
*  当在某个范围的末尾插入文本时，Word 会自动扩展该范围以包括插入的文本。

#### 2.5 Bookmark对象 ####
Microsoft.Office.Interop.Word.Bookmark 对象表示文档中的相邻区域，同时具有起始位置和结束位置。 你可以使用书签标记文档中的某个位置，也可将其作为文档中文本的容器。 Microsoft.Office.Interop.Word.Bookmark 对象可以包含插入点，也可以与整个文档一样大。Microsoft.Office.Interop.Word.Bookmark 具有下列特征，以将其与 Range 对象区别开来：
* 你可以在设计时命名书签。
* Microsoft.Office.Interop.Word.Bookmark 对象随文档一起保存，因此在代码停止运行或文档关闭时不会被删除。
* 通过将 T:Microsoft.Office.Interop.Word.View 对象的 P:Microsoft.Office.Interop.Word.View.ShowBookmarks 属性设置为 false 或 true，可以隐藏或显示书签。