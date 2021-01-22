<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。 这是获取调用 Microsoft Graph 所需的 OAuth 访问令牌所必需的。 为此，您需要将 Microsoft 身份验证库 [ (MSAL) iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) 集成到应用程序中。

1. 在 **名为****AuthSettings.plist****的 GraphTu一l** 项目中创建新的属性列表文件。
1. 将以下项添加到根字典 **中的文件** 。

    | 键 | 类型 | 值 |
    |-----|------|-------|
    | `AppId` | String | Azure 门户中的应用程序 ID |
    | `GraphScopes` | 数组 | 三个字符串值：、 `User.Read` `MailboxSettings.Read` 和 `Calendars.ReadWrite` |

    ![Xcode 中的 AuthSettings.plist 文件的屏幕截图](./images/auth-settings.png)

> [!IMPORTANT]
> 如果你使用的是源代码管理（如 git），那么现在应该从源代码管理中排除 **AuthSettings.plist** 文件，以避免意外泄露应用 ID。

## <a name="implement-sign-in"></a>实现登录

在此部分中，你将为 MSAL 配置项目，创建身份验证管理器类，并更新应用以登录和注销。

### <a name="configure-project-for-msal"></a>为 MSAL 配置项目

1. 向项目功能添加新的钥匙链组。
    1. 选择 **GraphTu一l** 项目，然后签署 **&功能**。
    1. 选择 **+ 功能**，然后双击 **钥匙链共享**。
    1. 添加带值的钥匙链组 `com.microsoft.adalcache` 。

1. 控件单击 **Info.plist，** 然后选择 **"打开为**"，然后选择 **"源代码"。**
1. 在元素中添加 `<dict>` 以下内容。

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

1. 打开 **AppDelegate.m，** 在文件顶部添加以下导入语句。

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. 将以下函数添加到 `AppDelegate` 类。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>创建身份验证管理器

1. 在 **GraphTu一** l 项目中 **新建一个名为** **AuthenticationManager** 的 Cocoa Touch 类。 在 **字段的子类中选择 NSObject。** 
1. 打开 **AuthenticationManager.h，** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. 打开 **AuthenticationManager.m，** 并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>添加登录和注销

1. 打开 **SignInViewController.m** 文件，并将其内容替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. 打开 **WelcomeViewController.m，** 将以下 `import` 语句添加到文件顶部。

    ```objc
    #import "AuthenticationManager.h"
    ```

1. 将现有的 `signOut` 函数替换为以下内容。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. 保存更改，并重启模拟器中的应用程序。

如果登录应用，应该会看到访问令牌显示在 Xcode 的输出窗口中。

![Xcode 中输出窗口的屏幕截图，显示访问令牌](./images/access-token-output.png)

## <a name="get-user-details"></a>获取用户详细信息

在此部分中，你将创建一个帮助程序类来保存对 Microsoft Graph 的所有调用，并更新为使用此新类获取 `WelcomeViewController` 登录用户。

1. 在 **GraphTu一** l 项目中 **新建一** 个名为 **GraphManager** 的 Cocoa Touch 类。 在 **字段的子类中选择 NSObject。** 
1. 打开 **GraphManager.h** 并将其内容替换为以下代码。

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import <MSGraphClientModels/MSCollection.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user,
                                         NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock) completion;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. 打开 **GraphManager.m** 并将其内容替换为以下代码。

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

    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock) completion {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me?%@",
                                 MSGraphBaseURL,
                                 @"$select=displayName,mail,mailboxSettings,userPrincipalName"];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completion(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completion(nil, graphError);
                } else {
                    completion(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. 打开 **WelcomeViewController.m，** 在文件顶部 `#import` 添加以下语句。

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. 将以下属性添加到 `WelcomeViewController` 接口声明。

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. 将现有 `viewDidLoad` 代码替换为以下代码。

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

如果保存更改并立即重新启动应用，则登录 UI 后，会使用用户的 显示名称 和电子邮件地址进行更新。
