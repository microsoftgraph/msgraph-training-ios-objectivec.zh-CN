<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将 Microsoft Graph 合并到应用程序中。 对于此应用程序，你将使用 [适用于目标 C](https://github.com/microsoftgraph/msgraph-sdk-objc) 的 Microsoft Graph SDK 调用 Microsoft Graph。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

在此部分中，您将扩展类以添加一个函数，获取当前一周的用户事件，并 `GraphManager` 更新为 `CalendarViewController` 使用此新函数。

1. 打开 **GraphManager.h，** 在声明上方添加以下 `@interface` 代码。

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSData* _Nullable data,
                                                   NSError* _Nullable error);
    ```

1. 将以下代码添加到 `@interface` 声明。

    ```objc
    - (void) getCalendarViewStartingAt: (NSString*) viewStart
                              endingAt: (NSString*) viewEnd
                   withCompletionBlock: (GetCalendarViewCompletionBlock) completion;
    ```

1. 打开 **GraphManager.m，** 将以下函数添加到 `GraphManager` 类。

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
    > 考虑代码正在 `getCalendarViewStartingAt` 执行哪些工作。
    >
    > - 将调用的 URL 为 `/v1.0/me/calendarview` 。
    >   - 和 `startDateTime` `endDateTime` 查询参数定义日历视图的开始和结束。
    >   - 查询 `select` 参数将每个事件返回的字段限制为视图将实际使用的字段。
    >   - 查询 `orderby` 参数按开始时间对结果进行排序。
    >   - 查询 `top` 参数请求每页 25 个结果。
    >   - 标头使 Microsoft Graph 返回用户时区中每个事件的开始时间 `Prefer: outlook.timezone` 和结束时间。

1. 在 **GraphTu一** l 项目中 **新建一** 个名为 **GraphToIana** 的 Cocoa Touch 类。 在 **字段的子类中选择 NSObject。** 
1. 打开 **GraphToIana.h** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.h" id="GraphToIanaSnippet":::

1. 打开 **GraphToIana.m** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.m" id="GraphToIanaSnippet":::

    这将进行简单的查找，以根据 Microsoft Graph 返回的时区名称查找 IANA 时区标识符。

1. 打开 **CalendarViewController.m，** 并将其全部内容替换为以下代码。

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

1. 运行应用、登录，然后点击菜单中的 **"** 日历"导航项。 你应该会看到应用中事件的 JSON 转储。

## <a name="display-the-results"></a>显示结果

现在，可以将 JSON 转储替换为某些内容，以用户友好的方式显示结果。 在此部分中，您将修改函数以返回强类型对象，并修改为使用表视图 `getCalendarViewStartingAt` `CalendarViewController` 呈现事件。

### <a name="update-getcalendarviewstartingat"></a>更新 getCalendarViewStartingAt

1. 打开 **GraphManager.h**。 将 `GetCalendarViewCompletionBlock` 类型定义更改为以下内容。

    ```objc
    typedef void (^GetCalendarViewCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. 打开 **GraphManager.m**。 将现有的 `getCalendarViewStartingAt` 函数替换成以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/GraphManager.m" id="GetCalendarViewSnippet" highlight="42-61":::

### <a name="update-calendarviewcontroller"></a>更新 CalendarViewController

1. 在 **GraphTu一** l 项目中 **新建一** 个名为 Cocoa Touch 类的文件 `CalendarTableViewCell` 。 在字段的子类 **中选择** **UITableViewCell。**
1. 打开 **CalendarTableViewCell.h** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.h" id="CalendarTableCellSnippet":::

1. 打开 **CalendarTableViewCell.m，** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.m" id="CalendarTableCellSnippet":::

1. 在 **GraphTu一** l 项目中 **新建一** 个名为 Cocoa Touch 类的文件 `CalendarTableViewController` 。 在 **字段的子类中选择 UITableViewController。** 
1. 打开 **CalendarTableViewController.h，** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.h" id="CalendarTableViewControllerSnippet":::

1. 打开 **CalendarTableViewController.m，** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.m" id="CalendarTableViewControllerSnippet":::

1. 打开 **Main.storyboard** 并找到 **日历场景**。 从根视图中删除滚动视图。
1. 使用 **库**，将 **导航栏** 添加到视图的顶部。
1. 双击导航 **栏中** 的标题，并更新为 `Calendar` 。
1. 使用 **库**，将 **栏按钮项** 添加到导航栏的右侧。
1. 选择新的栏按钮，然后选择 **属性检查器**。 将 **图像更改为****加** 号。
1. 将 **"库"中的** 容器 **视图** 添加到导航栏下的视图中。 调整容器视图的大小以占用视图中的所有剩余空间。
1. 设置导航栏和容器视图的约束，如下所示。

    - **导航栏**
        - 添加约束：高度，值：44
        - 添加约束：安全区域前导空格，值：0
        - 向安全区域添加约束：尾随空格，值：0
        - 添加约束：安全区域的顶部空间，值：0
    - **容器视图**
        - 添加约束：安全区域前导空格，值：0
        - 向安全区域添加约束：尾随空格，值：0
        - 添加约束：导航栏底部的顶部空间，值：0
        - 添加约束：安全区域的底部空间，值：0

1. 在添加容器视图时，找到添加到情节提要的第二个视图控制器。 它通过嵌入 segue **连接到日历** 场景。 选择此控制器并使用 **标识检查** 器将 **类** 更改为 **CalendarTableViewController。**
1. 从 **日历表** 视图 **控制器中删除视图**。
1. 将库 **的表** 视图 **添加到****日历表视图控制器**。
1. 选择表视图，然后选择 **属性检查器**。 将 **原型单元格设置为** **1。**
1. 拖动原型单元格的底部边缘，以为您提供一个更大的区域。
1. 使用 **库向** 原型单元格 **添加** 三个标签。
1. 选择原型单元格，然后选择 **标识检查器**。 将 **类** 更改为 **CalendarTableViewCell**。
1. 选择 **属性检查器，** 将 **标识符设置为** `EventCell` 。
1. 选中 **EventCell** 后，选择 **连接** 检查器并连接 ，并连接到添加到情节提要上的单元格 `durationLabel` `organizerLabel` `subjectLabel` 的标签。
1. 按如下所示设置三个标签的属性和约束。

    - **主题标签**
        - 添加约束：将空格前导到内容视图前导边距，值：0
        - 添加约束：内容视图尾部边距的尾部空格，值：0
        - 添加约束：内容视图上边距的上边距空间，值：0
    - **组织者标签**
        - 字体：System 12.0
        - 添加约束：高度，值：15
        - 添加约束：将空格前导到内容视图前导边距，值：0
        - 添加约束：内容视图尾部边距的尾部空格，值：0
        - 添加约束：主题标签底部的顶部空间，值：标准
    - **持续时间标签**
        - 字体：System 12.0
        - 颜色：深灰色
        - 添加约束：高度，值：15
        - 添加约束：将空格前导到内容视图前导边距，值：0
        - 添加约束：内容视图尾部边距的尾部空格，值：0
        - 添加约束：将顶部空间添加到组织者标签底部，值：标准
        - 添加约束：内容视图下边距的底部空间，值：0

1. 选择 **EventCell，** 然后选择 **大小检查器**。 启用 **自动** 行 **高**。

    ![日历和日历表视图控制器的屏幕截图](images/calendar-table-storyboard.png)

1. 打开 **CalendarViewController.h** 并删除 `calendarJSON` 该属性。

1. 打开 **CalendarViewController.m，** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.m" id="CalendarViewControllerSnippet":::

1. 运行应用、登录并点击 **"日历"** 选项卡。应看到事件列表。

    ![事件表的屏幕截图](./images/calendar-list.png)
