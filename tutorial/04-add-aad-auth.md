<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a8c9f-101">在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="a8c9f-102">这是获取调用 Microsoft Graph 所需的 OAuth 访问令牌所必需的。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="a8c9f-103">为此，您需要将 Microsoft 身份验证库 [ (MSAL) iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) 集成到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="a8c9f-104">在 **名为\*\*\*\*AuthSettings.plist\*\*\*\*的 GraphTu一l** 项目中创建新的属性列表文件。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="a8c9f-105">将以下项添加到根字典 **中的文件** 。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="a8c9f-106">键</span><span class="sxs-lookup"><span data-stu-id="a8c9f-106">Key</span></span> | <span data-ttu-id="a8c9f-107">类型</span><span class="sxs-lookup"><span data-stu-id="a8c9f-107">Type</span></span> | <span data-ttu-id="a8c9f-108">值</span><span class="sxs-lookup"><span data-stu-id="a8c9f-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="a8c9f-109">String</span><span class="sxs-lookup"><span data-stu-id="a8c9f-109">String</span></span> | <span data-ttu-id="a8c9f-110">Azure 门户中的应用程序 ID</span><span class="sxs-lookup"><span data-stu-id="a8c9f-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="a8c9f-111">数组</span><span class="sxs-lookup"><span data-stu-id="a8c9f-111">Array</span></span> | <span data-ttu-id="a8c9f-112">三个字符串值：、 `User.Read` `MailboxSettings.Read` 和 `Calendars.ReadWrite`</span><span class="sxs-lookup"><span data-stu-id="a8c9f-112">Three String values: `User.Read`, `MailboxSettings.Read`, and `Calendars.ReadWrite`</span></span> |

    ![Xcode 中的 AuthSettings.plist 文件的屏幕截图](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="a8c9f-114">如果你使用的是源代码管理（如 git），那么现在应该从源代码管理中排除 **AuthSettings.plist** 文件，以避免意外泄露应用 ID。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="a8c9f-115">实现登录</span><span class="sxs-lookup"><span data-stu-id="a8c9f-115">Implement sign-in</span></span>

<span data-ttu-id="a8c9f-116">在此部分中，你将为 MSAL 配置项目，创建身份验证管理器类，并更新应用以登录和注销。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="a8c9f-117">为 MSAL 配置项目</span><span class="sxs-lookup"><span data-stu-id="a8c9f-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="a8c9f-118">向项目功能添加新的钥匙链组。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="a8c9f-119">选择 **GraphTu一l** 项目，然后签署 **&功能**。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="a8c9f-120">选择 **+ 功能**，然后双击 **钥匙链共享**。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="a8c9f-121">添加带值的钥匙链组 `com.microsoft.adalcache` 。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="a8c9f-122">控件单击 **Info.plist，** 然后选择 **"打开为**"，然后选择 **"源代码"。**</span><span class="sxs-lookup"><span data-stu-id="a8c9f-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="a8c9f-123">在元素中添加 `<dict>` 以下内容。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="a8c9f-124">打开 **AppDelegate.m，** 在文件顶部添加以下导入语句。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-124">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="a8c9f-125">将以下函数添加到 `AppDelegate` 类。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.m" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="a8c9f-126">创建身份验证管理器</span><span class="sxs-lookup"><span data-stu-id="a8c9f-126">Create authentication manager</span></span>

1. <span data-ttu-id="a8c9f-127">在 **GraphTu一** l 项目中 **新建一个名为** **AuthenticationManager** 的 Cocoa Touch 类。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-127">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="a8c9f-128">在 **字段的子类中选择 NSObject。** </span><span class="sxs-lookup"><span data-stu-id="a8c9f-128">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="a8c9f-129">打开 **AuthenticationManager.h，** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-129">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.h" id="AuthManagerSnippet":::

1. <span data-ttu-id="a8c9f-130">打开 **AuthenticationManager.m，** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-130">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.m" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="a8c9f-131">添加登录和注销</span><span class="sxs-lookup"><span data-stu-id="a8c9f-131">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="a8c9f-132">打开 **SignInViewController.m** 文件，并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-132">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.m" id="SignInViewSnippet":::

1. <span data-ttu-id="a8c9f-133">打开 **WelcomeViewController.m，** 将以下 `import` 语句添加到文件顶部。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-133">Open **WelcomeViewController.m** and add the following `import` statement to the top of the file.</span></span>

    ```objc
    #import "AuthenticationManager.h"
    ```

1. <span data-ttu-id="a8c9f-134">将现有的 `signOut` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-134">Replace the existing `signOut` function with the following.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="SignOutSnippet":::

1. <span data-ttu-id="a8c9f-135">保存更改，并重启模拟器中的应用程序。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-135">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="a8c9f-136">如果登录应用，应该会看到访问令牌显示在 Xcode 的输出窗口中。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-136">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Xcode 中输出窗口的屏幕截图，显示访问令牌](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="a8c9f-138">获取用户详细信息</span><span class="sxs-lookup"><span data-stu-id="a8c9f-138">Get user details</span></span>

<span data-ttu-id="a8c9f-139">在此部分中，你将创建一个帮助程序类来保存对 Microsoft Graph 的所有调用，并更新为使用此新类获取 `WelcomeViewController` 登录用户。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-139">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="a8c9f-140">在 **GraphTu一** l 项目中 **新建一** 个名为 **GraphManager** 的 Cocoa Touch 类。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-140">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="a8c9f-141">在 **字段的子类中选择 NSObject。** </span><span class="sxs-lookup"><span data-stu-id="a8c9f-141">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="a8c9f-142">打开 **GraphManager.h** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-142">Open **GraphManager.h** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="a8c9f-143">打开 **GraphManager.m** 并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-143">Open **GraphManager.m** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="a8c9f-144">打开 **WelcomeViewController.m，** 在文件顶部 `#import` 添加以下语句。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-144">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="a8c9f-145">将以下属性添加到 `WelcomeViewController` 接口声明。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-145">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="a8c9f-146">将现有 `viewDidLoad` 代码替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-146">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="objc" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.m" id="ViewDidLoadSnippet":::

<span data-ttu-id="a8c9f-147">如果保存更改并立即重新启动应用，则登录 UI 后，会使用用户的 显示名称 和电子邮件地址进行更新。</span><span class="sxs-lookup"><span data-stu-id="a8c9f-147">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
