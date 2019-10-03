<!-- markdownlint-disable MD002 MD041 -->

在本练习中，将把 Microsoft Graph 合并到应用程序中。 对于此应用程序，您将使用[Microsoft GRAPH SDK For 目标 C](https://github.com/microsoftgraph/msgraph-sdk-objc)以调用 microsoft graph。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

在本节中，您将扩展`GraphManager`类以添加一个函数，以获取用户的事件并更新`CalendarViewController`以使用这些新函数。

1. 打开**GraphManager** ，并在`@interface`声明上方添加以下代码。

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSData* _Nullable data, NSError* _Nullable error);
    ```

1. 将以下代码添加到`@interface`声明中。

    ```objc
    - (void) getEventsWithCompletionBlock: (GetEventsCompletionBlock)completionBlock;
    ```

1. 打开**GraphManager** ，并将以下函数添加到`GraphManager`类中。

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
    > 考虑中`getEventsWithCompletionBlock`的代码执行的操作。
    >
    > - 将调用的 URL 为`/v1.0/me/events`。
    > - `select`查询参数将为每个事件返回的字段限制为仅应用程序实际使用的字段。
    > - `orderby`查询参数按其创建日期和时间对结果进行排序，最新项目最先开始。

1. 打开**CalendarViewController** ，并将其全部内容替换为以下代码。

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

您现在可以运行应用程序，登录，然后点击菜单中的 "**日历**" 导航项。 您应该会看到应用程序中的事件的 JSON 转储。

## <a name="display-the-results"></a>显示结果

现在，您可以将 JSON 转储替换为以用户友好的方式显示结果的内容。 在本节中，您将修改`getEventsWithCompletionBlock`函数以返回强类型的对象，并进行修改`CalendarViewController`以使用表视图呈现事件。

### <a name="update-getevents"></a>更新 getEvents

1. 打开**GraphManager**。 将`GetEventsCompletionBlock`类型定义更改为以下项。

    ```objc
    typedef void (^GetEventsCompletionBlock)(NSArray<MSGraphEvent*>* _Nullable events, NSError* _Nullable error);
    ```

1. 打开**GraphManager**。 将`getEventsWithCompletionBlock`函数`completionBlock(data, nil);`中的行替换为以下代码。

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

### <a name="update-calendarviewcontroller"></a>更新 CalendarViewController

1. 在名为`CalendarTableViewCell`的**GraphTutorial**项目中创建一个新的**Cocoa Touch 类**文件。 在字段的 "**子类**" 中选择 " **UITableViewCell** "。
1. 打开**CalendarTableViewCell** ，并将其内容替换为以下代码。

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

1. 打开**CalendarTableViewCell**并将其内容替换为以下代码。

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

1. 打开 "**情节提要**" 并找到**日历场景**。 在**日历场景**中选择**视图**并将其删除。

    ![日历场景中的视图的屏幕截图](./images/view-in-calendar-scene.png)

1. 将**库**中的**表视图**添加到**日历场景**中。
1. 选择表格视图，然后选择 "**属性" 检查器**。 将 "**原型" 单元格**设置为**1**。
1. 使用**库**向原型单元格添加三个**标签**。
1. 选择 "原型" 单元格，然后选择 "**标识检查器**"。 将**类**更改为**CalendarTableViewCell**。
1. 选择 "**属性" 检查器**并将`EventCell`"**标识符**" 设置为。
1. 在**编辑器**菜单上，选择 "**解决自动布局问题**"，然后选择 **"欢迎视图控制器中的所有视图"** 下的 "**添加缺少约束**"。
1. 选择**EventCell**后，选择 "**连接检查器**" 和`durationLabel`" `organizerLabel`连接" `subjectLabel` ，以及添加到情节提要上的单元格的标签。

    ![原型单元格布局的屏幕截图](./images/prototype-cell-layout.png)

1. 打开**CalendarViewController**并删除`calendarJSON`属性。
1. 将`@interface`声明更改为以下项。

    ```objc
    @interface CalendarViewController : UITableViewController
    ```

1. 打开**CalendarViewController**并将其内容替换为以下代码。

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

1. 运行应用程序，登录，然后点击 "**日历**" 选项卡。您应该会看到事件列表。

    ![事件表的屏幕截图](./images/calendar-list.png)
