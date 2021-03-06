<!-- markdownlint-disable MD002 MD041 -->

在此部分中，您将添加在用户日历上创建事件的能力。

1. 打开 **GraphManager.h，** 在声明上方添加以下 `@interface` 代码。

    ```objc
    typedef void (^CreateEventCompletionBlock)(MSGraphEvent* _Nullable event,
                                               NSError* _Nullable error);
    ```

1. 将以下代码添加到 `@interface` 声明。

    ```objc
    - (void) createEventWithSubject: (NSString*) subject
                           andStart: (NSDate*) start
                             andEnd: (NSDate*) end
                       andAttendees: (NSArray<NSString*>* _Nullable) attendees
                            andBody: (NSString* _Nullable) body
                 andCompletionBlock: (CreateEventCompletionBlock) completion;
    ```

1. 打开 **GraphManager.m** 并添加以下函数以在用户日历上创建新事件。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="CreateEventSnippet":::

1. 在 **GraphTu一** l 文件夹中 **新建一** 个名为 Cocoa Touch 类的文件 `NewEventViewController` 。 在 **字段的子类中选择 UIViewController。** 
1. 打开 **NewEventViewController.h，** 在声明中添加以下 `@interface` 代码。

    ```objectivec
    @property (nonatomic) IBOutlet UITextField* subject;
    @property (nonatomic) IBOutlet UITextField* attendees;
    @property (nonatomic) IBOutlet UIDatePicker* start;
    @property (nonatomic) IBOutlet UIDatePicker* end;
    @property (nonatomic) IBOutlet UITextView* body;
    ```

1. 打开 **NewEventController.m，** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/NewEventViewController.m" id="NewEventViewControllerSnippet":::

1. 打开 **Main.storyboard**。 使用 **库** 将视图 **控制器拖动** 到情节提要上。
1. 使用 **库**，将 **导航栏** 添加到视图控制器。
1. 双击导航 **栏中** 的标题，并更新为 `New Event` 。
1. 使用 **库**，将 **栏按钮项** 添加到导航栏的左侧。
1. 选择新的栏按钮，然后选择 **属性检查器**。 将 **标题** 更改为 `Cancel` 。
1. 使用 **库**，将 **栏按钮项** 添加到导航栏的右侧。
1. 选择新的栏按钮，然后选择 **属性检查器**。 将 **标题更改为** `Create` 。
1. 选择视图控制器，然后选择 **标识检查器**。 将 **类** 更改为 **NewEventViewController**。
1. 将以下控件从 **库** 添加到视图中。

    - 在导航 **栏** 下添加标签。 将文本设置为 `Subject` 。
    - 在标签 **下添加** 文本字段。 将占位符 **属性** 设置为 `Subject` 。
    - 在文本 **字段** 下添加标签。 将文本设置为 `Attendees` 。
    - 在标签 **下添加** 文本字段。 将占位符 **属性** 设置为 `Separate multiple entries with ;` 。
    - 在文本 **字段** 下添加标签。 将文本设置为 `Start` 。
    - 在标签 **下添加** 日期选取器。 将首选 **样式** 设置为 **"压缩**"，**将间隔** 设置为 **15 分钟**，将高度设置为 **35。**
    - 在日期 **选取** 器下添加标签。 将文本设置为 `End` 。
    - 在标签 **下添加** 日期选取器。 将首选 **样式** 设置为 **"压缩**"，**将间隔** 设置为 **15 分钟**，将高度设置为 **35。**
    - 在日期 **选取器** 下添加文本视图。

1. 选择 **"新建事件视图控制器** "，并使用 **连接检查** 器建立以下连接。

    - 将 **取消** 接收的操作连接到"取消栏 **"** 按钮。
    - 将 **createEvent** 接收的操作连接到 **"创建栏"** 按钮。
    - 将 **主题出口** 连接到第一个文本字段。
    - 将 **与会者出口** 连接到第二个文本字段。
    - 将 **开始出口** 连接到第一个日期选取器。
    - 将 **结束出口** 连接到第二个日期选取器。
    - 将 **正文出口** 连接到文本视图。

1. 添加以下约束。

    - **导航栏**
        - 安全区域前导空格，值：0
        - 安全区域尾部空间，值：0
        - 安全区域的顶部空间，值：0
        - 高度，值：44
    - **主题标签**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 导航栏的顶部空间，值：20
    - **主题文本字段**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 主题标签的上空间，值：Standard
    - **与会者标签**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 主题文本字段的上空间，值：Standard
    - **与会者文本字段**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 与会者标签的上空间，值：Standard
    - **开始标签**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 主题文本字段的上空间，值：Standard
    - **开始日期选取器**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 与会者标签的上空间，值：Standard
        - 高度，值：35
    - **结束标签**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 开始日期选取器的顶部空间，值：Standard
    - **结束日期选取器**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 结束标签的上空间，值：Standard
        - 高度：35
    - **正文文本视图**
        - 视图边距前导空格，值：0
        - 查看边距的尾部空间，值：0
        - 结束日期选取器的顶部空间，值：Standard
        - 查看边距的底部空间，值：0

    ![情节提要上新事件表单的屏幕截图](images/new-event-form.png)

1. 选择 **日历场景**，然后选择连接 **检查器**。
1. 在 **"触发的 Segues"** 下，将手动旁的未填充圆圈拖到情节提要上的"新建事件视图控制器"上。 在 **弹出菜单中** 选择"以模式方式显示"。
1. 选择刚添加的 segue，然后选择 **属性检查器**。 将 **Identifier 字段** 设置为 `showEventForm` 。
1. 将 **showNewEventForm** 接收的操作连接到 **+** 导航栏按钮。
1. 保存更改并重新启动该应用。 转到日历页面，然后点击 **+** 该按钮。 填写表单并点击 **"创建** "创建新事件。

    ![新事件表单的屏幕截图](images/create-event.png)
