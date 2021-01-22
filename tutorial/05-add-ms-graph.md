<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="fa03c-101">在此练习中，你将 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="fa03c-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="fa03c-102">对于此应用程序，你将使用 [适用于目标 C](https://github.com/microsoftgraph/msgraph-sdk-objc) 的 Microsoft Graph SDK 调用 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="fa03c-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="fa03c-103">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="fa03c-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="fa03c-104">在此部分中，您将扩展类以添加一个函数，获取当前一周的用户事件，并 `GraphManager` 更新为 `CalendarViewController` 使用此新函数。</span><span class="sxs-lookup"><span data-stu-id="fa03c-104">In this section you will extend the `GraphManager` class to add a function to get the user's events for the current week and update `CalendarViewController` to use this new function.</span></span>

1. <span data-ttu-id="fa03c-105">打开 **GraphManager.h，** 在声明上方添加以下 `@interface` 代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSData* _Nullable data,
                                                   NSError* _Nullable error);
    ```

1. <span data-ttu-id="fa03c-106">将以下代码添加到 `@interface` 声明。</span><span class="sxs-lookup"><span data-stu-id="fa03c-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getCalendarViewStartingAt: (NSString*) viewStart
                              endingAt: (NSString*) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion;
    ```

1. <span data-ttu-id="fa03c-107">打开 **GraphManager.m，** 将以下函数添加到 `GraphManager` 类。</span><span class="sxs-lookup"><span data-stu-id="fa03c-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

    ```objc
    - (void) getCalendarViewStartingAt: (NSString *) viewStart
                              endingAt: (NSString *) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion {
        // Set calendar view start and end parameters
        NSString* viewStartEndString =
        [NSString stringWithFormat:@"startDateTime=%@&endDateTime=%@",
         viewStart,
         viewEnd];

        // GET /me/calendarview
        NSString* eventsUrlString =
        [NSString stringWithFormat:@"%@/me/calendarview?%@&%@&%@&%@",
         MSGraphBaseURL,
         viewStartEndString,
         // Only return these fields in results
         @"$select=subject,organizer,start,end",
         // Sort results by start time
         @"$orderby=start/dateTime",
         // Request at most 25 results
         @"$top=25"];

        NSURL* eventsUrl = [[NSURL alloc] initWithString:eventsUrlString];
        NSMutableURLRequest* eventsRequest = [[NSMutableURLRequest alloc] initWithURL:eventsUrl];

        // Add the Prefer: outlook.timezone header to get start and end times
        // in user's time zone
        NSString* preferHeader =
        [NSString stringWithFormat:@"outlook.timezone=\"%@\"",
         self.graphTimeZone];
        [eventsRequest addValue:preferHeader forHTTPHeaderField:@"Prefer"];

        MSURLSessionDataTask* eventsDataTask =
        [[MSURLSessionDataTask alloc]
         initWithRequest:eventsRequest
         client:self.graphClient
         completion:^(NSData *data, NSURLResponse *response, NSError *error) {
             if (error) {
                 completion(nil, error);
                 return;
             }

             // TEMPORARY
             completion(data, nil);
         }];

        // Execute the request
        [eventsDataTask execute];
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="fa03c-108">考虑代码正在 `getCalendarViewStartingAt` 执行哪些工作。</span><span class="sxs-lookup"><span data-stu-id="fa03c-108">Consider what the code in `getCalendarViewStartingAt` is doing.</span></span>
    >
    > - <span data-ttu-id="fa03c-109">将调用的 URL 为 `/v1.0/me/calendarview` 。</span><span class="sxs-lookup"><span data-stu-id="fa03c-109">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
    >   - <span data-ttu-id="fa03c-110">和 `startDateTime` `endDateTime` 查询参数定义日历视图的开始和结束。</span><span class="sxs-lookup"><span data-stu-id="fa03c-110">The `startDateTime` and `endDateTime` query parameters define the start and end of the calendar view.</span></span>
    >   - <span data-ttu-id="fa03c-111">查询 `select` 参数将每个事件返回的字段限制为视图将实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="fa03c-111">The `select` query parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="fa03c-112">查询 `orderby` 参数按开始时间对结果进行排序。</span><span class="sxs-lookup"><span data-stu-id="fa03c-112">The `orderby` query parameter sorts the results by start time.</span></span>
    >   - <span data-ttu-id="fa03c-113">查询 `top` 参数请求每页 25 个结果。</span><span class="sxs-lookup"><span data-stu-id="fa03c-113">The `top` query parameter requests 25 results per page.</span></span>
    >   - <span data-ttu-id="fa03c-114">标头使 Microsoft Graph 返回用户时区中每个事件的开始时间 `Prefer: outlook.timezone` 和结束时间。</span><span class="sxs-lookup"><span data-stu-id="fa03c-114">the `Prefer: outlook.timezone` header causes the Microsoft Graph to return the start and end times of each event in the user's time zone.</span></span>

1. <span data-ttu-id="fa03c-115">在 **GraphTu一** l 项目中 **新建一** 个名为 **GraphToIana** 的 Cocoa Touch 类。</span><span class="sxs-lookup"><span data-stu-id="fa03c-115">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphToIana**.</span></span> <span data-ttu-id="fa03c-116">在 **字段的子类中选择 NSObject。** </span><span class="sxs-lookup"><span data-stu-id="fa03c-116">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="fa03c-117">打开 **GraphToIana.h** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-117">Open **GraphToIana.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.h" id="GraphToIanaSnippet":::

1. <span data-ttu-id="fa03c-118">打开 **GraphToIana.m** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-118">Open **GraphToIana.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.m" id="GraphToIanaSnippet":::

    <span data-ttu-id="fa03c-119">这将进行简单的查找，以根据 Microsoft Graph 返回的时区名称查找 IANA 时区标识符。</span><span class="sxs-lookup"><span data-stu-id="fa03c-119">This does a simple lookup to find an IANA time zone identifier based on the time zone name returned by Microsoft Graph.</span></span>

1. <span data-ttu-id="fa03c-120">打开 **CalendarViewController.m，** 并将其全部内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-120">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import "GraphToIana.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>

    @interface CalendarViewController ()

    @property SpinnerViewController* spinner;

    @end

    @implementation CalendarViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        // Calculate the start and end of the current week
        NSString* timeZoneId = [GraphToIana
                                getIanaIdentifierFromGraphIdentifier:
                                [GraphManager.instance graphTimeZone]];

        NSDate* now = [NSDate date];
        NSCalendar* calendar = [NSCalendar calendarWithIdentifier:NSCalendarIdentifierGregorian];
        NSTimeZone* timeZone = [NSTimeZone timeZoneWithName:timeZoneId];
        [calendar setTimeZone:timeZone];

        NSDateComponents* startOfWeekComponents = [calendar
                                                   components:NSCalendarUnitCalendar |
                                                   NSCalendarUnitYearForWeekOfYear |
                                                   NSCalendarUnitWeekOfYear
                                                   fromDate:now];
        NSDate* startOfWeek = [startOfWeekComponents date];
        NSDate* endOfWeek = [calendar dateByAddingUnit:NSCalendarUnitDay
                                                 value:7
                                                toDate:startOfWeek
                                               options:0];

        // Convert start and end to ISO 8601 strings
        NSISO8601DateFormatter* isoFormatter = [[NSISO8601DateFormatter alloc] init];
        NSString* viewStart = [isoFormatter stringFromDate:startOfWeek];
        NSString* viewEnd = [isoFormatter stringFromDate:endOfWeek];

        [GraphManager.instance
         getCalendarViewStartingAt:viewStart
         endingAt:viewEnd
         withCompletionBlock:^(NSData * _Nullable data, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error) {
                     // Show the error
                     UIAlertController* alert = [UIAlertController
                                                 alertControllerWithTitle:@"Error getting events"
                                                 message:error.debugDescription
                                                 preferredStyle:UIAlertControllerStyleAlert];

                     UIAlertAction* okButton = [UIAlertAction
                                                actionWithTitle:@"OK"
                                                style:UIAlertActionStyleDefault
                                                handler:nil];

                     [alert addAction:okButton];
                     [self presentViewController:alert animated:true completion:nil];
                     return;
                 }

                 // TEMPORARY
                 self.calendarJSON.text = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
                 [self.calendarJSON sizeToFit];
             });
         }];
    }

    @end
    ```

1. <span data-ttu-id="fa03c-121">运行应用、登录，然后点击菜单中的 **"** 日历"导航项。</span><span class="sxs-lookup"><span data-stu-id="fa03c-121">Run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="fa03c-122">你应该会看到应用中事件的 JSON 转储。</span><span class="sxs-lookup"><span data-stu-id="fa03c-122">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="fa03c-123">显示结果</span><span class="sxs-lookup"><span data-stu-id="fa03c-123">Display the results</span></span>

<span data-ttu-id="fa03c-124">现在，可以将 JSON 转储替换为某些内容，以用户友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="fa03c-124">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="fa03c-125">在此部分中，您将修改函数以返回强类型对象，并修改为使用表视图 `getCalendarViewStartingAt` `CalendarViewController` 呈现事件。</span><span class="sxs-lookup"><span data-stu-id="fa03c-125">In this section, you will modify the `getCalendarViewStartingAt` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getcalendarviewstartingat"></a><span data-ttu-id="fa03c-126">更新 getCalendarViewStartingAt</span><span class="sxs-lookup"><span data-stu-id="fa03c-126">Update getCalendarViewStartingAt</span></span>

1. <span data-ttu-id="fa03c-127">打开 **GraphManager.h**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-127">Open **GraphManager.h**.</span></span> <span data-ttu-id="fa03c-128">将 `GetCalendarViewCompletionBlock` 类型定义更改为以下内容。</span><span class="sxs-lookup"><span data-stu-id="fa03c-128">Change the `GetCalendarViewCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="fa03c-129">打开 **GraphManager.m**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-129">Open **GraphManager.m**.</span></span> <span data-ttu-id="fa03c-130">将现有的 `getCalendarViewStartingAt` 函数替换成以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-130">Replace the existing `getCalendarViewStartingAt` function with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetCalendarViewSnippet" highlight="42-61":::

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="fa03c-131">更新 CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="fa03c-131">Update CalendarViewController</span></span>

1. <span data-ttu-id="fa03c-132">在 **GraphTu一** l 项目中 **新建一** 个名为 Cocoa Touch 类的文件 `CalendarTableViewCell` 。</span><span class="sxs-lookup"><span data-stu-id="fa03c-132">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="fa03c-133">在字段的子类 **中选择** **UITableViewCell。**</span><span class="sxs-lookup"><span data-stu-id="fa03c-133">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="fa03c-134">打开 **CalendarTableViewCell.h** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-134">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="fa03c-135">打开 **CalendarTableViewCell.m，** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-135">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. <span data-ttu-id="fa03c-136">在 **GraphTu一** l 项目中 **新建一** 个名为 Cocoa Touch 类的文件 `CalendarTableViewController` 。</span><span class="sxs-lookup"><span data-stu-id="fa03c-136">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewController`.</span></span> <span data-ttu-id="fa03c-137">在 **字段的子类中选择 UITableViewController。** </span><span class="sxs-lookup"><span data-stu-id="fa03c-137">Choose **UITableViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="fa03c-138">打开 **CalendarTableViewController.h，** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-138">Open **CalendarTableViewController.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.h" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="fa03c-139">打开 **CalendarTableViewController.m，** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-139">Open **CalendarTableViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.m" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="fa03c-140">打开 **Main.storyboard** 并找到 **日历场景**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-140">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="fa03c-141">从根视图中删除滚动视图。</span><span class="sxs-lookup"><span data-stu-id="fa03c-141">Delete the scroll view from the root view.</span></span>
1. <span data-ttu-id="fa03c-142">使用 **库**，将 **导航栏** 添加到视图的顶部。</span><span class="sxs-lookup"><span data-stu-id="fa03c-142">Using the **Library**, add a **Navigation Bar** to the top of the view.</span></span>
1. <span data-ttu-id="fa03c-143">双击导航 **栏中** 的标题，并更新为 `Calendar` 。</span><span class="sxs-lookup"><span data-stu-id="fa03c-143">Double-click the **Title** in the navigation bar and update it to `Calendar`.</span></span>
1. <span data-ttu-id="fa03c-144">使用 **库**，将 **栏按钮项** 添加到导航栏的右侧。</span><span class="sxs-lookup"><span data-stu-id="fa03c-144">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="fa03c-145">选择新的栏按钮，然后选择 **属性检查器**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-145">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="fa03c-146">将 **图像更改为\*\*\*\*加** 号。</span><span class="sxs-lookup"><span data-stu-id="fa03c-146">Change **Image** to **plus**.</span></span>
1. <span data-ttu-id="fa03c-147">将 **"库"中的** 容器 **视图** 添加到导航栏下的视图中。</span><span class="sxs-lookup"><span data-stu-id="fa03c-147">Add a **Container View** from the **Library** to the view under the navigation bar.</span></span> <span data-ttu-id="fa03c-148">调整容器视图的大小以占用视图中的所有剩余空间。</span><span class="sxs-lookup"><span data-stu-id="fa03c-148">Resize the container view to take all of the remaining space in the view.</span></span>
1. <span data-ttu-id="fa03c-149">设置导航栏和容器视图的约束，如下所示。</span><span class="sxs-lookup"><span data-stu-id="fa03c-149">Set constraints on the navigation bar and container view as follows.</span></span>

    - <span data-ttu-id="fa03c-150">**导航栏**</span><span class="sxs-lookup"><span data-stu-id="fa03c-150">**Navigation Bar**</span></span>
        - <span data-ttu-id="fa03c-151">添加约束：高度，值：44</span><span class="sxs-lookup"><span data-stu-id="fa03c-151">Add constraint: Height, value: 44</span></span>
        - <span data-ttu-id="fa03c-152">添加约束：安全区域前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-152">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="fa03c-153">向安全区域添加约束：尾随空格，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-153">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="fa03c-154">添加约束：安全区域的顶部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-154">Add constraint: Top space to Safe Area, value: 0</span></span>
    - <span data-ttu-id="fa03c-155">**容器视图**</span><span class="sxs-lookup"><span data-stu-id="fa03c-155">**Container View**</span></span>
        - <span data-ttu-id="fa03c-156">添加约束：安全区域前导空格，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-156">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="fa03c-157">向安全区域添加约束：尾随空格，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-157">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="fa03c-158">添加约束：导航栏底部的顶部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-158">Add constraint: Top space to Navigation Bar Bottom, value: 0</span></span>
        - <span data-ttu-id="fa03c-159">添加约束：安全区域的底部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-159">Add constraint: Bottom space to Safe Area, value: 0</span></span>

1. <span data-ttu-id="fa03c-160">在添加容器视图时，找到添加到情节提要的第二个视图控制器。</span><span class="sxs-lookup"><span data-stu-id="fa03c-160">Locate the second view controller added to the storyboard when you added the container view.</span></span> <span data-ttu-id="fa03c-161">它通过嵌入 segue **连接到日历** 场景。</span><span class="sxs-lookup"><span data-stu-id="fa03c-161">It is connected to the **Calendar Scene** by an embed segue.</span></span> <span data-ttu-id="fa03c-162">选择此控制器并使用 **标识检查** 器将 **类** 更改为 **CalendarTableViewController。**</span><span class="sxs-lookup"><span data-stu-id="fa03c-162">Select this controller and use the **Identity Inspector** to change **Class** to **CalendarTableViewController**.</span></span>
1. <span data-ttu-id="fa03c-163">从 **日历表** 视图 **控制器中删除视图**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-163">Delete the **View** from the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="fa03c-164">将库 **的表** 视图 **添加到\*\*\*\*日历表视图控制器**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-164">Add a **Table View** from the **Library** to the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="fa03c-165">选择表视图，然后选择 **属性检查器**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-165">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="fa03c-166">将 **原型单元格设置为** **1。**</span><span class="sxs-lookup"><span data-stu-id="fa03c-166">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="fa03c-167">拖动原型单元格的底部边缘，以为您提供一个更大的区域。</span><span class="sxs-lookup"><span data-stu-id="fa03c-167">Drag the bottom edge of the prototype cell to give you a larger area to work with.</span></span>
1. <span data-ttu-id="fa03c-168">使用 **库向** 原型单元格 **添加** 三个标签。</span><span class="sxs-lookup"><span data-stu-id="fa03c-168">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="fa03c-169">选择原型单元格，然后选择 **标识检查器**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-169">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="fa03c-170">将 **类** 更改为 **CalendarTableViewCell**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-170">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="fa03c-171">选择 **属性检查器，** 将 **标识符设置为** `EventCell` 。</span><span class="sxs-lookup"><span data-stu-id="fa03c-171">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="fa03c-172">选中 **EventCell** 后，选择 **连接** 检查器并连接 ，并连接到添加到情节提要上的单元格 `durationLabel` `organizerLabel` `subjectLabel` 的标签。</span><span class="sxs-lookup"><span data-stu-id="fa03c-172">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="fa03c-173">按如下所示设置三个标签的属性和约束。</span><span class="sxs-lookup"><span data-stu-id="fa03c-173">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="fa03c-174">**主题标签**</span><span class="sxs-lookup"><span data-stu-id="fa03c-174">**Subject Label**</span></span>
        - <span data-ttu-id="fa03c-175">添加约束：将空格前导到内容视图前导边距，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-175">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="fa03c-176">添加约束：内容视图尾部边距的尾部空格，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-176">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="fa03c-177">添加约束：内容视图上边距的上边距空间，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-177">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="fa03c-178">**组织者标签**</span><span class="sxs-lookup"><span data-stu-id="fa03c-178">**Organizer Label**</span></span>
        - <span data-ttu-id="fa03c-179">字体：System 12.0</span><span class="sxs-lookup"><span data-stu-id="fa03c-179">Font: System 12.0</span></span>
        - <span data-ttu-id="fa03c-180">添加约束：高度，值：15</span><span class="sxs-lookup"><span data-stu-id="fa03c-180">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="fa03c-181">添加约束：将空格前导到内容视图前导边距，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-181">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="fa03c-182">添加约束：内容视图尾部边距的尾部空格，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-182">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="fa03c-183">添加约束：主题标签底部的顶部空间，值：标准</span><span class="sxs-lookup"><span data-stu-id="fa03c-183">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="fa03c-184">**持续时间标签**</span><span class="sxs-lookup"><span data-stu-id="fa03c-184">**Duration Label**</span></span>
        - <span data-ttu-id="fa03c-185">字体：System 12.0</span><span class="sxs-lookup"><span data-stu-id="fa03c-185">Font: System 12.0</span></span>
        - <span data-ttu-id="fa03c-186">颜色：深灰色</span><span class="sxs-lookup"><span data-stu-id="fa03c-186">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="fa03c-187">添加约束：高度，值：15</span><span class="sxs-lookup"><span data-stu-id="fa03c-187">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="fa03c-188">添加约束：将空格前导到内容视图前导边距，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-188">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="fa03c-189">添加约束：内容视图尾部边距的尾部空格，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-189">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="fa03c-190">添加约束：将顶部空间添加到组织者标签底部，值：标准</span><span class="sxs-lookup"><span data-stu-id="fa03c-190">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="fa03c-191">添加约束：内容视图下边距的底部空间，值：0</span><span class="sxs-lookup"><span data-stu-id="fa03c-191">Add constraint: Bottom space to Content View Bottom Margin, value: 0</span></span>

1. <span data-ttu-id="fa03c-192">选择 **EventCell，** 然后选择 **大小检查器**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-192">Select the **EventCell**, then select the **Size Inspector**.</span></span> <span data-ttu-id="fa03c-193">启用 **自动** 行 **高**。</span><span class="sxs-lookup"><span data-stu-id="fa03c-193">Enable **Automatic** for **Row Height**.</span></span>

    ![日历和日历表视图控制器的屏幕截图](images/calendar-table-storyboard.png)

1. <span data-ttu-id="fa03c-195">打开 **CalendarViewController.h** 并删除 `calendarJSON` 该属性。</span><span class="sxs-lookup"><span data-stu-id="fa03c-195">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>

1. <span data-ttu-id="fa03c-196">打开 **CalendarViewController.m，** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="fa03c-196">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewControllerSnippet":::

1. <span data-ttu-id="fa03c-197">运行应用、登录并点击 **"日历"** 选项卡。应看到事件列表。</span><span class="sxs-lookup"><span data-stu-id="fa03c-197">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![事件表的屏幕截图](./images/calendar-list.png)
