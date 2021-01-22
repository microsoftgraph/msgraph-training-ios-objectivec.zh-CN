<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c23fd-101">在此部分中，您将添加在用户日历上创建事件的能力。</span><span class="sxs-lookup"><span data-stu-id="c23fd-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="c23fd-102">打开 **GraphManager.h，** 在声明上方添加以下 `@interface` 代码。</span><span class="sxs-lookup"><span data-stu-id="c23fd-102">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^CreateEventCompletionBlock)(MSGraphEvent* _Nullable event,
                                               NSError* _Nullable error);
    ```

1. <span data-ttu-id="c23fd-103">将以下代码添加到 `@interface` 声明。</span><span class="sxs-lookup"><span data-stu-id="c23fd-103">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) createEventWithSubject: (NSString*) subject
                           andStart: (NSDate*) start
                             andEnd: (NSDate*) end
                       andAttendees: (NSArray<NSString*>* _Nullable) attendees
                            andBody: (NSString* _Nullable) body
                 andCompletionBlock: (CreateEventCompletionBlock) completion;
    ```

1. <span data-ttu-id="c23fd-104">打开 **GraphManager.m** 并添加以下函数以在用户日历上创建新事件。</span><span class="sxs-lookup"><span data-stu-id="c23fd-104">Open **GraphManager.m** and add the following function to create a new event on the user's calendar.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="CreateEventSnippet":::

1. <span data-ttu-id="c23fd-105">在 **GraphTu一** l 文件夹中 **新建一** 个名为 Cocoa Touch 类的文件 `NewEventViewController` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-105">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `NewEventViewController`.</span></span> <span data-ttu-id="c23fd-106">在 **字段的子类中选择 UIViewController。** </span><span class="sxs-lookup"><span data-stu-id="c23fd-106">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="c23fd-107">打开 **NewEventViewController.h，** 在声明中添加以下 `@interface` 代码。</span><span class="sxs-lookup"><span data-stu-id="c23fd-107">Open **NewEventViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objectivec
    @property (nonatomic) IBOutlet UITextField* subject;
    @property (nonatomic) IBOutlet UITextField* attendees;
    @property (nonatomic) IBOutlet UIDatePicker* start;
    @property (nonatomic) IBOutlet UIDatePicker* end;
    @property (nonatomic) IBOutlet UITextView* body;
    ```

1. <span data-ttu-id="c23fd-108">打开 **NewEventController.m，** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="c23fd-108">Open **NewEventController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/NewEventViewController.m" id="NewEventViewControllerSnippet":::

1. <span data-ttu-id="c23fd-109">打开 **Main.storyboard**。</span><span class="sxs-lookup"><span data-stu-id="c23fd-109">Open **Main.storyboard**.</span></span> <span data-ttu-id="c23fd-110">使用 **库** 将视图 **控制器拖动** 到情节提要上。</span><span class="sxs-lookup"><span data-stu-id="c23fd-110">Use the **Library** to drag a **View Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="c23fd-111">使用 **库**，将 **导航栏** 添加到视图控制器。</span><span class="sxs-lookup"><span data-stu-id="c23fd-111">Using the **Library**, add a **Navigation Bar** to the view controller.</span></span>
1. <span data-ttu-id="c23fd-112">双击导航 **栏中** 的标题，并更新为 `New Event` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-112">Double-click the **Title** in the navigation bar and update it to `New Event`.</span></span>
1. <span data-ttu-id="c23fd-113">使用 **库**，将 **栏按钮项** 添加到导航栏的左侧。</span><span class="sxs-lookup"><span data-stu-id="c23fd-113">Using the **Library**, add a **Bar Button Item** to the left-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="c23fd-114">选择新的栏按钮，然后选择 **属性检查器**。</span><span class="sxs-lookup"><span data-stu-id="c23fd-114">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="c23fd-115">将 **标题** 更改为 `Cancel` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-115">Change **Title** to `Cancel`.</span></span>
1. <span data-ttu-id="c23fd-116">使用 **库**，将 **栏按钮项** 添加到导航栏的右侧。</span><span class="sxs-lookup"><span data-stu-id="c23fd-116">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="c23fd-117">选择新的栏按钮，然后选择 **属性检查器**。</span><span class="sxs-lookup"><span data-stu-id="c23fd-117">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="c23fd-118">将 **标题更改为** `Create` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-118">Change **Title** to `Create`.</span></span>
1. <span data-ttu-id="c23fd-119">选择视图控制器，然后选择 **标识检查器**。</span><span class="sxs-lookup"><span data-stu-id="c23fd-119">Select the view controller, then select the **Identity Inspector**.</span></span> <span data-ttu-id="c23fd-120">将 **类** 更改为 **NewEventViewController**。</span><span class="sxs-lookup"><span data-stu-id="c23fd-120">Change **Class** to **NewEventViewController**.</span></span>
1. <span data-ttu-id="c23fd-121">将以下控件从 **库** 添加到视图中。</span><span class="sxs-lookup"><span data-stu-id="c23fd-121">Add the following controls from the **Library** to the view.</span></span>

    - <span data-ttu-id="c23fd-122">在导航 **栏** 下添加标签。</span><span class="sxs-lookup"><span data-stu-id="c23fd-122">Add a **Label** under the navigation bar.</span></span> <span data-ttu-id="c23fd-123">将文本设置为 `Subject` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-123">Set its text to `Subject`.</span></span>
    - <span data-ttu-id="c23fd-124">在标签 **下添加** 文本字段。</span><span class="sxs-lookup"><span data-stu-id="c23fd-124">Add a **Text Field** under the label.</span></span> <span data-ttu-id="c23fd-125">将占位符 **属性** 设置为 `Subject` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-125">Set its **Placeholder** attribute to `Subject`.</span></span>
    - <span data-ttu-id="c23fd-126">在文本 **字段** 下添加标签。</span><span class="sxs-lookup"><span data-stu-id="c23fd-126">Add a **Label** under the text field.</span></span> <span data-ttu-id="c23fd-127">将文本设置为 `Attendees` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-127">Set its text to `Attendees`.</span></span>
    - <span data-ttu-id="c23fd-128">在标签 **下添加** 文本字段。</span><span class="sxs-lookup"><span data-stu-id="c23fd-128">Add a **Text Field** under the label.</span></span> <span data-ttu-id="c23fd-129">将占位符 **属性** 设置为 `Separate multiple entries with ;` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-129">Set its **Placeholder** attribute to `Separate multiple entries with ;`.</span></span>
    - <span data-ttu-id="c23fd-130">在文本 **字段** 下添加标签。</span><span class="sxs-lookup"><span data-stu-id="c23fd-130">Add a **Label** under the text field.</span></span> <span data-ttu-id="c23fd-131">将文本设置为 `Start` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-131">Set its text to `Start`.</span></span>
    - <span data-ttu-id="c23fd-132">在标签 **下添加** 日期选取器。</span><span class="sxs-lookup"><span data-stu-id="c23fd-132">Add a **Date Picker** under the label.</span></span> <span data-ttu-id="c23fd-133">将首选 **样式** 设置为 **"压缩**"，**将间隔** 设置为 **15 分钟**，将高度设置为 **35。**</span><span class="sxs-lookup"><span data-stu-id="c23fd-133">Set its **Preferred Style** to **Compact**, its **Interval** to **15 minutes**, and its height to **35**.</span></span>
    - <span data-ttu-id="c23fd-134">在日期 **选取** 器下添加标签。</span><span class="sxs-lookup"><span data-stu-id="c23fd-134">Add a **Label** under the date picker.</span></span> <span data-ttu-id="c23fd-135">将文本设置为 `End` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-135">Set its text to `End`.</span></span>
    - <span data-ttu-id="c23fd-136">在标签 **下添加** 日期选取器。</span><span class="sxs-lookup"><span data-stu-id="c23fd-136">Add a **Date Picker** under the label.</span></span> <span data-ttu-id="c23fd-137">将首选 **样式** 设置为 **"压缩**"，**将间隔** 设置为 **15 分钟**，将高度设置为 **35。**</span><span class="sxs-lookup"><span data-stu-id="c23fd-137">Set its **Preferred Style** to **Compact**, its **Interval** to **15 minutes**, and its height to **35**.</span></span>
    - <span data-ttu-id="c23fd-138">在日期 **选取器** 下添加文本视图。</span><span class="sxs-lookup"><span data-stu-id="c23fd-138">Add a **Text View** under the date picker.</span></span>

1. <span data-ttu-id="c23fd-139">选择 **"新建事件视图控制器** "，并使用 **连接检查** 器建立以下连接。</span><span class="sxs-lookup"><span data-stu-id="c23fd-139">Select the **New Event View Controller** and use the **Connection Inspector** to make the following connections.</span></span>

    - <span data-ttu-id="c23fd-140">将 **取消** 接收的操作连接到"取消栏 **"** 按钮。</span><span class="sxs-lookup"><span data-stu-id="c23fd-140">Connect the **cancel** received action to the **Cancel** bar button.</span></span>
    - <span data-ttu-id="c23fd-141">将 **createEvent** 接收的操作连接到 **"创建栏"** 按钮。</span><span class="sxs-lookup"><span data-stu-id="c23fd-141">Connect the **createEvent** received action to the **Create** bar button.</span></span>
    - <span data-ttu-id="c23fd-142">将 **主题出口** 连接到第一个文本字段。</span><span class="sxs-lookup"><span data-stu-id="c23fd-142">Connect the **subject** outlet to the first text field.</span></span>
    - <span data-ttu-id="c23fd-143">将 **与会者出口** 连接到第二个文本字段。</span><span class="sxs-lookup"><span data-stu-id="c23fd-143">Connect the **attendees** outlet to the second text field.</span></span>
    - <span data-ttu-id="c23fd-144">将 **开始出口** 连接到第一个日期选取器。</span><span class="sxs-lookup"><span data-stu-id="c23fd-144">Connect the **start** outlet to the first date picker.</span></span>
    - <span data-ttu-id="c23fd-145">将 **结束出口** 连接到第二个日期选取器。</span><span class="sxs-lookup"><span data-stu-id="c23fd-145">Connect the **end** outlet to the second date picker.</span></span>
    - <span data-ttu-id="c23fd-146">将 **正文出口** 连接到文本视图。</span><span class="sxs-lookup"><span data-stu-id="c23fd-146">Connect the **body** outlet to the text view.</span></span>

1. <span data-ttu-id="c23fd-147">添加以下约束。</span><span class="sxs-lookup"><span data-stu-id="c23fd-147">Add the following constraints.</span></span>

    - <span data-ttu-id="c23fd-148">**导航栏**</span><span class="sxs-lookup"><span data-stu-id="c23fd-148">**Navigation Bar**</span></span>
        - <span data-ttu-id="c23fd-149">安全区域前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-149">Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="c23fd-150">安全区域尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-150">Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="c23fd-151">安全区域的顶部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-151">Top space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="c23fd-152">高度，值：44</span><span class="sxs-lookup"><span data-stu-id="c23fd-152">Height, value: 44</span></span>
    - <span data-ttu-id="c23fd-153">**主题标签**</span><span class="sxs-lookup"><span data-stu-id="c23fd-153">**Subject Label**</span></span>
        - <span data-ttu-id="c23fd-154">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-154">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-155">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-155">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-156">导航栏的顶部空间，值：20</span><span class="sxs-lookup"><span data-stu-id="c23fd-156">Top space to Navigation Bar, value: 20</span></span>
    - <span data-ttu-id="c23fd-157">**主题文本字段**</span><span class="sxs-lookup"><span data-stu-id="c23fd-157">**Subject Text Field**</span></span>
        - <span data-ttu-id="c23fd-158">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-158">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-159">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-159">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-160">主题标签的上空间，值：Standard</span><span class="sxs-lookup"><span data-stu-id="c23fd-160">Top space to Subject Label, value: Standard</span></span>
    - <span data-ttu-id="c23fd-161">**与会者标签**</span><span class="sxs-lookup"><span data-stu-id="c23fd-161">**Attendees Label**</span></span>
        - <span data-ttu-id="c23fd-162">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-162">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-163">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-163">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-164">主题文本字段的上空间，值：Standard</span><span class="sxs-lookup"><span data-stu-id="c23fd-164">Top space to Subject Text Field, value: Standard</span></span>
    - <span data-ttu-id="c23fd-165">**与会者文本字段**</span><span class="sxs-lookup"><span data-stu-id="c23fd-165">**Attendees Text Field**</span></span>
        - <span data-ttu-id="c23fd-166">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-166">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-167">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-167">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-168">与会者标签的上空间，值：Standard</span><span class="sxs-lookup"><span data-stu-id="c23fd-168">Top space to Attendees Label, value: Standard</span></span>
    - <span data-ttu-id="c23fd-169">**开始标签**</span><span class="sxs-lookup"><span data-stu-id="c23fd-169">**Start Label**</span></span>
        - <span data-ttu-id="c23fd-170">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-170">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-171">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-171">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-172">主题文本字段的上空间，值：Standard</span><span class="sxs-lookup"><span data-stu-id="c23fd-172">Top space to Subject Text Field, value: Standard</span></span>
    - <span data-ttu-id="c23fd-173">**开始日期选取器**</span><span class="sxs-lookup"><span data-stu-id="c23fd-173">**Start Date Picker**</span></span>
        - <span data-ttu-id="c23fd-174">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-174">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-175">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-175">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-176">与会者标签的上空间，值：Standard</span><span class="sxs-lookup"><span data-stu-id="c23fd-176">Top space to Attendees Label, value: Standard</span></span>
        - <span data-ttu-id="c23fd-177">高度，值：35</span><span class="sxs-lookup"><span data-stu-id="c23fd-177">Height, value: 35</span></span>
    - <span data-ttu-id="c23fd-178">**结束标签**</span><span class="sxs-lookup"><span data-stu-id="c23fd-178">**End Label**</span></span>
        - <span data-ttu-id="c23fd-179">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-179">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-180">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-180">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-181">开始日期选取器的顶部空间，值：Standard</span><span class="sxs-lookup"><span data-stu-id="c23fd-181">Top space to Start Date Picker, value: Standard</span></span>
    - <span data-ttu-id="c23fd-182">**结束日期选取器**</span><span class="sxs-lookup"><span data-stu-id="c23fd-182">**End Date Picker**</span></span>
        - <span data-ttu-id="c23fd-183">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-183">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-184">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-184">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-185">结束标签的上空间，值：Standard</span><span class="sxs-lookup"><span data-stu-id="c23fd-185">Top space to End Label, value: Standard</span></span>
        - <span data-ttu-id="c23fd-186">高度：35</span><span class="sxs-lookup"><span data-stu-id="c23fd-186">Height: 35</span></span>
    - <span data-ttu-id="c23fd-187">**正文文本视图**</span><span class="sxs-lookup"><span data-stu-id="c23fd-187">**Body Text View**</span></span>
        - <span data-ttu-id="c23fd-188">视图边距前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-188">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-189">查看边距的尾部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-189">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="c23fd-190">结束日期选取器的顶部空间，值：Standard</span><span class="sxs-lookup"><span data-stu-id="c23fd-190">Top space to End Date Picker, value: Standard</span></span>
        - <span data-ttu-id="c23fd-191">查看边距的底部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="c23fd-191">Bottom space to View margin, value: 0</span></span>

    ![情节提要上新事件表单的屏幕截图](images/new-event-form.png)

1. <span data-ttu-id="c23fd-193">选择 **日历场景**，然后选择连接 **检查器**。</span><span class="sxs-lookup"><span data-stu-id="c23fd-193">Select the **Calendar Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="c23fd-194">在 **"触发的 Segues"** 下，将手动旁的未填充圆圈拖到情节提要上的"新建事件视图控制器"上。</span><span class="sxs-lookup"><span data-stu-id="c23fd-194">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **New Event View Controller** on the storyboard.</span></span> <span data-ttu-id="c23fd-195">在 **弹出菜单中** 选择"以模式方式显示"。</span><span class="sxs-lookup"><span data-stu-id="c23fd-195">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="c23fd-196">选择刚添加的 segue，然后选择 **属性检查器**。</span><span class="sxs-lookup"><span data-stu-id="c23fd-196">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="c23fd-197">将 **Identifier 字段** 设置为 `showEventForm` 。</span><span class="sxs-lookup"><span data-stu-id="c23fd-197">Set the **Identifier** field to `showEventForm`.</span></span>
1. <span data-ttu-id="c23fd-198">将 **showNewEventForm** 接收的操作连接到 **+** 导航栏按钮。</span><span class="sxs-lookup"><span data-stu-id="c23fd-198">Connect the **showNewEventForm** received action to the **+** navigation bar button.</span></span>
1. <span data-ttu-id="c23fd-199">保存更改并重新启动该应用。</span><span class="sxs-lookup"><span data-stu-id="c23fd-199">Save your changes and restart the app.</span></span> <span data-ttu-id="c23fd-200">转到日历页面，然后点击 **+** 该按钮。</span><span class="sxs-lookup"><span data-stu-id="c23fd-200">Go to the calendar page and tap the **+** button.</span></span> <span data-ttu-id="c23fd-201">填写表单并点击 **"创建** "创建新事件。</span><span class="sxs-lookup"><span data-stu-id="c23fd-201">Fill in the form and tap **Create** to create a new event.</span></span>

    ![新事件表单的屏幕截图](images/create-event.png)
