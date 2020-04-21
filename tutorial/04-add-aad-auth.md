<!-- markdownlint-disable MD002 MD041 -->

在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。 若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。 为此，需要将[iOS 的 Microsoft 身份验证库（MSAL）](https://github.com/AzureAD/microsoft-authentication-library-for-objc)集成到应用程序中。

1. 在名为**AuthSettings**的**GraphTutorial**项目中创建一个新的**属性列表**文件。
1. 将以下项添加到**根**字典中的文件中。

    | 键 | 类型 | 值 |
    |-----|------|-------|
    | `AppId` | String | Azure 门户中的应用程序 ID |
    | `GraphScopes` | 数组 | 两个字符串值`User.Read` ：和`Calendars.Read` |

    ![Xcode 中的 plist 文件的屏幕截图 AuthSettings](./images/auth-settings.png)

> [!IMPORTANT]
> 如果您使用的是源代码管理（如 git），现在是从源代码控制中排除**AuthSettings**文件以避免无意中泄漏您的应用程序 ID 的最佳时机。

## <a name="implement-sign-in"></a>实施登录

在本节中，您将为 MSAL 配置项目，创建身份验证管理器类，并更新应用程序以进行登录和注销。

### <a name="configure-project-for-msal"></a>为 MSAL 配置项目

1. 将新的密钥链组添加到项目的功能中。
    1. 依次选择 " **GraphTutorial** " 项目、"**签名 & 功能**"。
    1. 选择 " **+ 功能**"，然后双击 "**密钥链共享**"。
    1. 添加具有值`com.microsoft.adalcache`的密钥链组。

1. 控件单击 " **plist** "，然后选择 "**打开方式**"，然后选择 "**源代码**"。
1. 在`<dict>`元素内添加以下项。

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
      </dict>
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. 打开**AppDelegate** ，并在文件顶部添加以下导入语句。

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. 将以下函数添加到 `AppDelegate` 类。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>创建身份验证管理器

1. 在名为**authenticationmanager.java**的**GraphTutorial**项目中创建一个新的**Cocoa Touch 类**。 在字段的 "**子类**" 中选择 " **NSObject** "。
1. 打开**authenticationmanager.java** ，并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. 打开**authenticationmanager.java**并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>添加登录和注销

1. 打开**SignInViewController**文件并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. 打开**WelcomeViewController** ，并将以下`import`语句添加到文件顶部。

    ```objc
    #import "AuthenticationManager.h"
    ```

1. 将现有的 `signOut` 函数替换为以下内容。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. 保存所做的更改，并在模拟器中重新启动应用程序。

如果登录到应用，您应该会看到在 Xcode 的输出窗口中显示的访问令牌。

![显示访问令牌的 Xcode 中的输出窗口的屏幕截图](./images/access-token-output.png)

## <a name="get-user-details"></a>获取用户详细信息

在本节中，您将创建一个帮助程序类，以保存对 Microsoft Graph 的所有调用并`WelcomeViewController`更新，以使用此新类获取已登录的用户。

1. 在名为**GraphManager**的**GraphTutorial**项目中创建一个新的**Cocoa Touch 类**。 在字段的 "**子类**" 中选择 " **NSObject** "。
1. 打开**GraphManager** ，并将其内容替换为以下代码。

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import <MSGraphClientModels/MSCollection.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user, NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock)completionBlock;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. 打开**GraphManager**并将其内容替换为以下代码。

    ```objc
    #import "GraphManager.h"

    @interface GraphManager()

    @property MSHTTPClient* graphClient;

    @end

    @implementation GraphManager

    + (id) instance {
        static GraphManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {
            // Create the Graph client
            self.graphClient = [MSClientFactory
                                createHTTPClientWithAuthenticationProvider:AuthenticationManager.instance];
        }

        return self;
    }

    - (void) getMeWithCompletionBlock:(GetMeCompletionBlock)completionBlock {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me", MSGraphBaseURL];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completionBlock(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completionBlock(nil, graphError);
                } else {
                    completionBlock(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. 打开**WelcomeViewController** ，并在文件顶部添加`#import`以下语句。

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. 将以下属性添加到`WelcomeViewController`接口声明中。

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. 将现有`viewDidLoad`替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

如果现在保存更改并重新启动应用程序，则在使用用户的显示名称和电子邮件地址更新 UI 后，登录 UI。
