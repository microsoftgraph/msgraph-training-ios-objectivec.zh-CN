<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c9715-101">在本练习中，将把 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="c9715-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="c9715-102">对于此应用程序，您将使用[Microsoft GRAPH SDK For 目标 C](https://github.com/microsoftgraph/msgraph-sdk-objc)以调用 microsoft graph。</span><span class="sxs-lookup"><span data-stu-id="c9715-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="c9715-103">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="c9715-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="c9715-104">在本节中，您将扩展`GraphManager`类以添加一个函数，以获取用户的事件并更新`CalendarViewController`以使用这些新函数。</span><span class="sxs-lookup"><span data-stu-id="c9715-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="c9715-105">打开**GraphManager** ，并在`@interface`声明上方添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="c9715-105">Open **GraphManager.h** and add the following code above the `@interface` declaration.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. <span data-ttu-id="c9715-106">将以下代码添加到`@interface`声明中。</span><span class="sxs-lookup"><span data-stu-id="c9715-106">Add the following code to the `@interface` declaration.</span></span>

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. <span data-ttu-id="c9715-107">打开**GraphManager** ，并将以下函数添加到`GraphManager`类中。</span><span class="sxs-lookup"><span data-stu-id="c9715-107">Open **GraphManager.m** and add the following function to the `GraphManager` class.</span></span>

    ```objc
    - (void) getEventsWithCompletionBlock:(GetEventsCompletionBlock)completionBlock {
        // GET /me/events?$select='subject,organizer,start,end'$orderby=createdDateTime DESC
        NSString* eventsUrlString =
        [NSString stringWithFormat:@"%@/me/events?%@&%@",
         MSGraphBaseURL,
         // Only return these fields in results
         @"$select=subject,organizer,start,end",
         // Sort results by when they were created, newest first
         @"$orderby=createdDateTime+DESC"];

        NSURL* eventsUrl = [[NSURL alloc] initWithString:eventsUrlString];
        NSMutableURLRequest* eventsRequest = [[NSMutableURLRequest alloc] initWithURL:eventsUrl];

        MSURLSessionDataTask* eventsDataTask =
        [[MSURLSessionDataTask alloc]
         initWithRequest:eventsRequest
         client:self.graphClient
         completion:^(NSData *data, NSURLResponse *response, NSError *error) {
             if (error) {
                 completionBlock(nil, error);
                 return;
             }

             // TEMPORARY
             completionBlock(data, nil);
         }];

        // Execute the request
        [eventsDataTask execute];
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="c9715-108">考虑中`getEventsWithCompletionBlock`的代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="c9715-108">Consider what the code in `getEventsWithCompletionBlock` is doing.</span></span>
    >
    > - <span data-ttu-id="c9715-109">将调用的 URL 为`/v1.0/me/events`。</span><span class="sxs-lookup"><span data-stu-id="c9715-109">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="c9715-110">`select`查询参数将为每个事件返回的字段限制为仅应用程序实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="c9715-110">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="c9715-111">`orderby`查询参数按其创建日期和时间对结果进行排序，最新项目最先开始。</span><span class="sxs-lookup"><span data-stu-id="c9715-111">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="c9715-112">打开**CalendarViewController** ，并将其全部内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="c9715-112">Open **CalendarViewController.m** and replace its entire contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
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

        [GraphManager.instance
         getEventsWithCompletionBlock:^(NSData * _Nullable data, NSError * _Nullable error) {
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

<span data-ttu-id="c9715-113">您现在可以运行应用程序，登录，然后点击菜单中的 "**日历**" 导航项。</span><span class="sxs-lookup"><span data-stu-id="c9715-113">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="c9715-114">您应该会看到应用程序中的事件的 JSON 转储。</span><span class="sxs-lookup"><span data-stu-id="c9715-114">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="c9715-115">显示结果</span><span class="sxs-lookup"><span data-stu-id="c9715-115">Display the results</span></span>

<span data-ttu-id="c9715-116">现在，您可以将 JSON 转储替换为以用户友好的方式显示结果的内容。</span><span class="sxs-lookup"><span data-stu-id="c9715-116">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="c9715-117">在本节中，您将修改`getEventsWithCompletionBlock`函数以返回强类型的对象，并进行修改`CalendarViewController`以使用表视图呈现事件。</span><span class="sxs-lookup"><span data-stu-id="c9715-117">In this section, you will modify the `getEventsWithCompletionBlock` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getevents"></a><span data-ttu-id="c9715-118">更新 getEvents</span><span class="sxs-lookup"><span data-stu-id="c9715-118">Update getEvents</span></span>

1. <span data-ttu-id="c9715-119">打开**GraphManager**。</span><span class="sxs-lookup"><span data-stu-id="c9715-119">Open **GraphManager.h**.</span></span> <span data-ttu-id="c9715-120">将`GetEventsCompletionBlock`类型定义更改为以下项。</span><span class="sxs-lookup"><span data-stu-id="c9715-120">Change the `GetEventsCompletionBlock` type definition to the following.</span></span>

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. <span data-ttu-id="c9715-121">打开**GraphManager**。</span><span class="sxs-lookup"><span data-stu-id="c9715-121">Open **GraphManager.m**.</span></span> <span data-ttu-id="c9715-122">将`getEventsWithCompletionBlock`函数`completionBlock(data, nil);`中的行替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="c9715-122">Replace the `completionBlock(data, nil);` line in the `getEventsWithCompletionBlock` function with the following code.</span></span>

    ```objc
    NSError* graphError;

    // Deserialize to an events collection
    MSCollection* eventsCollection = [[MSCollection alloc] initWithData:data error:&graphError];
    if (graphError) {
        completionBlock(nil, graphError);
        return;
    }

    // Create an array to return
    NSMutableArray* eventsArray = [[NSMutableArray alloc]
                                initWithCapacity:eventsCollection.value.count];

    for (id event in eventsCollection.value) {
        // Deserialize the event and add to the array
        MSGraphEvent* graphEvent = [[MSGraphEvent alloc] initWithDictionary:event];
        [eventsArray addObject:graphEvent];
    }

    completionBlock(eventsArray, nil);
    ```

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="c9715-123">更新 CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="c9715-123">Update CalendarViewController</span></span>

1. <span data-ttu-id="c9715-124">在名为`CalendarTableViewCell`的**GraphTutorial**项目中创建一个新的**Cocoa Touch 类**文件。</span><span class="sxs-lookup"><span data-stu-id="c9715-124">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell`.</span></span> <span data-ttu-id="c9715-125">在字段的 "**子类**" 中选择 " **UITableViewCell** "。</span><span class="sxs-lookup"><span data-stu-id="c9715-125">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="c9715-126">打开**CalendarTableViewCell** ，并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="c9715-126">Open **CalendarTableViewCell.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <UIKit/UIKit.h>

    NS_ASSUME_NONNULL_BEGIN

    @interface CalendarTableViewCell : UITableViewCell

    @property (nonatomic) NSString* subject;
    @property (nonatomic) NSString* organizer;
    @property (nonatomic) NSString* duration;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="c9715-127">打开**CalendarTableViewCell**并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="c9715-127">Open **CalendarTableViewCell.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "CalendarTableViewCell.h"

    @interface CalendarTableViewCell()

    @property (nonatomic) IBOutlet UILabel *subjectLabel;
    @property (nonatomic) IBOutlet UILabel *organizerLabel;
    @property (nonatomic) IBOutlet UILabel *durationLabel;

    @end

    @implementation CalendarTableViewCell

    - (void)awakeFromNib {
        [super awakeFromNib];
        // Initialization code
    }

    - (void)setSelected:(BOOL)selected animated:(BOOL)animated {
        [super setSelected:selected animated:animated];

        // Configure the view for the selected state
    }

    - (void) setSubject:(NSString *)subject {
        _subject = subject;
        self.subjectLabel.text = subject;
        [self.subjectLabel sizeToFit];
    }

    - (void) setOrganizer:(NSString *)organizer {
        _organizer = organizer;
        self.organizerLabel.text = organizer;
        [self.organizerLabel sizeToFit];
    }

    - (void) setDuration:(NSString *)duration {
        _duration = duration;
        self.durationLabel.text = duration;
        [self.durationLabel sizeToFit];
    }

    @end
    ```

1. <span data-ttu-id="c9715-128">打开 "**情节提要**" 并找到**日历场景**。</span><span class="sxs-lookup"><span data-stu-id="c9715-128">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="c9715-129">在**日历场景**中选择**视图**并将其删除。</span><span class="sxs-lookup"><span data-stu-id="c9715-129">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![日历场景中的视图的屏幕截图](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="c9715-131">将**库**中的**表视图**添加到**日历场景**中。</span><span class="sxs-lookup"><span data-stu-id="c9715-131">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="c9715-132">选择表格视图，然后选择 "**属性" 检查器**。</span><span class="sxs-lookup"><span data-stu-id="c9715-132">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="c9715-133">将 "**原型" 单元格**设置为**1**。</span><span class="sxs-lookup"><span data-stu-id="c9715-133">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="c9715-134">使用**库**向原型单元格添加三个**标签**。</span><span class="sxs-lookup"><span data-stu-id="c9715-134">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="c9715-135">选择 "原型" 单元格，然后选择 "**标识检查器**"。</span><span class="sxs-lookup"><span data-stu-id="c9715-135">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="c9715-136">将**类**更改为**CalendarTableViewCell**。</span><span class="sxs-lookup"><span data-stu-id="c9715-136">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="c9715-137">选择 "**属性" 检查器**并将`EventCell`"**标识符**" 设置为。</span><span class="sxs-lookup"><span data-stu-id="c9715-137">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="c9715-138">在**编辑器**菜单上，选择 "**解决自动布局问题**"，然后选择 **"欢迎视图控制器中的所有视图"** 下的 "**添加缺少约束**"。</span><span class="sxs-lookup"><span data-stu-id="c9715-138">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>
1. <span data-ttu-id="c9715-139">选择**EventCell**后，选择 "**连接检查器**" 和`durationLabel`" `organizerLabel`连接" `subjectLabel` ，以及添加到情节提要上的单元格的标签。</span><span class="sxs-lookup"><span data-stu-id="c9715-139">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>

    ![原型单元格布局的屏幕截图](./images/prototype-cell-layout.png)

1. <span data-ttu-id="c9715-141">打开**CalendarViewController**并删除`calendarJSON`属性。</span><span class="sxs-lookup"><span data-stu-id="c9715-141">Open **CalendarViewController.h** and remove the `calendarJSON` property.</span></span>
1. <span data-ttu-id="c9715-142">将`@interface`声明更改为以下项。</span><span class="sxs-lookup"><span data-stu-id="c9715-142">Change the `@interface` declaration to the following.</span></span>

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. <span data-ttu-id="c9715-143">打开**CalendarViewController**并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="c9715-143">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "WelcomeViewController.h"
    #import "SpinnerViewController.h"
    #import "AuthenticationManager.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>

    @interface WelcomeViewController ()

    @property SpinnerViewController* spinner;

    @end

    @implementation WelcomeViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];

        [GraphManager.instance
         getMeWithCompletionBlock:^(MSGraphUser * _Nullable user, NSError * _Nullable error) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [self.spinner stop];

                if (error) {
                    // Show the error
                    UIAlertController* alert = [UIAlertController
                                                alertControllerWithTitle:@"Error getting user profile"
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

                // Set display name
                self.userDisplayName.text = user.displayName ? : @"Mysterious Stranger";
                [self.userDisplayName sizeToFit];

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = user.mail ? : user.userPrincipalName;
                [self.userEmail sizeToFit];
            });
         }];
    }

    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }

    @end
    ```

1. <span data-ttu-id="c9715-144">运行应用程序，登录，然后点击 "**日历**" 选项卡。您应该会看到事件列表。</span><span class="sxs-lookup"><span data-stu-id="c9715-144">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![事件表的屏幕截图](./images/calendar-list.png)
