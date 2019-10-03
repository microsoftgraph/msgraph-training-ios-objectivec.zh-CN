<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="85aa5-101">在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。</span><span class="sxs-lookup"><span data-stu-id="85aa5-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="85aa5-102">若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。</span><span class="sxs-lookup"><span data-stu-id="85aa5-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="85aa5-103">为此，需要将[iOS 的 Microsoft 身份验证库（MSAL）](https://github.com/AzureAD/microsoft-authentication-library-for-objc)集成到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="85aa5-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="85aa5-104">在名为**AuthSettings**的**GraphTutorial**项目中创建一个新的**属性列表**文件。</span><span class="sxs-lookup"><span data-stu-id="85aa5-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="85aa5-105">将以下项添加到**根**字典中的文件中。</span><span class="sxs-lookup"><span data-stu-id="85aa5-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="85aa5-106">Key</span><span class="sxs-lookup"><span data-stu-id="85aa5-106">Key</span></span> | <span data-ttu-id="85aa5-107">类型</span><span class="sxs-lookup"><span data-stu-id="85aa5-107">Type</span></span> | <span data-ttu-id="85aa5-108">值</span><span class="sxs-lookup"><span data-stu-id="85aa5-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="85aa5-109">String</span><span class="sxs-lookup"><span data-stu-id="85aa5-109">String</span></span> | <span data-ttu-id="85aa5-110">Azure 门户中的应用程序 ID</span><span class="sxs-lookup"><span data-stu-id="85aa5-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="85aa5-111">数组</span><span class="sxs-lookup"><span data-stu-id="85aa5-111">Array</span></span> | <span data-ttu-id="85aa5-112">两个字符串值`User.Read` ：和`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="85aa5-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Xcode 中的 plist 文件的屏幕截图 AuthSettings](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="85aa5-114">如果您使用的是源代码管理（如 git），现在是从源代码控制中排除**AuthSettings**文件以避免无意中泄漏您的应用程序 ID 的最佳时机。</span><span class="sxs-lookup"><span data-stu-id="85aa5-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="85aa5-115">实施登录</span><span class="sxs-lookup"><span data-stu-id="85aa5-115">Implement sign-in</span></span>

<span data-ttu-id="85aa5-116">在本节中，您将为 MSAL 配置项目，创建身份验证管理器类，并更新应用程序以进行登录和注销。</span><span class="sxs-lookup"><span data-stu-id="85aa5-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="85aa5-117">为 MSAL 配置项目</span><span class="sxs-lookup"><span data-stu-id="85aa5-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="85aa5-118">控件单击 " **plist** "，然后选择 "**打开方式**"，然后选择 "**源代码**"。</span><span class="sxs-lookup"><span data-stu-id="85aa5-118">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="85aa5-119">在`<dict>`元素内添加以下项。</span><span class="sxs-lookup"><span data-stu-id="85aa5-119">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="85aa5-120">打开**AppDelegate** ，并在文件顶部添加以下导入语句。</span><span class="sxs-lookup"><span data-stu-id="85aa5-120">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="85aa5-121">将以下函数添加到 `AppDelegate` 类。</span><span class="sxs-lookup"><span data-stu-id="85aa5-121">Add the following function to the `AppDelegate` class.</span></span>

    ```objc
    - (BOOL)application:(UIApplication *)app
                openURL:(NSURL *)url
                options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
    {
        return [MSALPublicClientApplication handleMSALResponse:url
                                             sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```

### <a name="create-authentication-manager"></a><span data-ttu-id="85aa5-122">创建身份验证管理器</span><span class="sxs-lookup"><span data-stu-id="85aa5-122">Create authentication manager</span></span>

1. <span data-ttu-id="85aa5-123">在名为**authenticationmanager.java**的**GraphTutorial**项目中创建一个新的**Cocoa Touch 类**。</span><span class="sxs-lookup"><span data-stu-id="85aa5-123">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="85aa5-124">在字段的 "**子类**" 中选择 " **NSObject** "。</span><span class="sxs-lookup"><span data-stu-id="85aa5-124">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="85aa5-125">打开**authenticationmanager.java** ，并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="85aa5-125">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSAL/MSAL.h>

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetTokenCompletionBlock)(NSString* _Nullable accessToken, NSError* _Nullable error);

    @interface AuthenticationManager : NSObject

    + (id) instance;
    - (void) getTokenInteractivelyWithCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) getTokenSilentlyWithCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) signOut;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="85aa5-126">打开**authenticationmanager.java**并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="85aa5-126">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "AuthenticationManager.h"

    @interface AuthenticationManager()

    @property NSString* appId;
    @property NSArray<NSString*>* graphScopes;
    @property MSALPublicClientApplication* publicClient;

    @end

    @implementation AuthenticationManager

    + (id) instance {
        static AuthenticationManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {
            // Get app ID and scopes from AuthSettings.plist
            NSString* authConfigPath =
            [NSBundle.mainBundle pathForResource:@"AuthSettings" ofType:@"plist"];
            NSString* bundleId = NSBundle.mainBundle.bundleIdentifier;
            NSDictionary* authConfig = [NSDictionary dictionaryWithContentsOfFile:authConfigPath];

            self.appId = authConfig[@"AppId"];
            self.graphScopes = authConfig[@"GraphScopes"];

            // Create the MSAL client
            self.publicClient = [[MSALPublicClientApplication alloc] initWithClientId:self.appId
                                                                        keychainGroup:bundleId
                                                                                error:nil];
        }

        return self;
    }

    - (void) getTokenInteractivelyWithCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        // Call acquireToken to open a browser so the user can sign in
        [self.publicClient
         acquireTokenForScopes:self.graphScopes
         completionBlock:^(MSALResult * _Nullable result, NSError * _Nullable error) {

            // Check error
            if (error) {
                completionBlock(nil, error);
                return;
            }

            // Check result
            if (!result) {
                NSMutableDictionary* details = [NSMutableDictionary dictionary];
                [details setValue:@"No result was returned" forKey:NSDebugDescriptionErrorKey];
                completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
                return;
            }

            NSLog(@"Got token interactively: %@", result.accessToken);
            completionBlock(result.accessToken, nil);
        }];
    }

    - (void) getTokenSilentlyWithCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        // Check if there is an account in the cache
        NSError* msalError;
        MSALAccount* account = [self.publicClient allAccounts:&msalError].firstObject;

        if (msalError || !account) {
            NSMutableDictionary* details = [NSMutableDictionary dictionary];
            [details setValue:@"Could not retrieve account from cache" forKey:NSDebugDescriptionErrorKey];
            completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
            return;
        }

        // Attempt to get token silently
        [self.publicClient
         acquireTokenSilentForScopes:self.graphScopes account:account
         completionBlock:^(MSALResult * _Nullable result, NSError * _Nullable error) {
             // Check error
             if (error) {
                 completionBlock(nil, error);
                 return;
             }

             // Check result
             if (!result) {
                 NSMutableDictionary* details = [NSMutableDictionary dictionary];
                 [details setValue:@"No result was returned" forKey:NSDebugDescriptionErrorKey];
                 completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
                 return;
             }

             NSLog(@"Got token silently: %@", result.accessToken);
             completionBlock(result.accessToken, nil);
         }];
    }

    - (void) signOut {
        NSError* msalError;
        NSArray* accounts = [self.publicClient allAccounts:&msalError];

        if (msalError) {
            NSLog(@"Error getting accounts from cache: %@", msalError.debugDescription);
            return;
        }

        for (id account in accounts) {
            [self.publicClient removeAccount:account error:nil];
        }
    }

    @end
    ```

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="85aa5-127">添加登录和注销</span><span class="sxs-lookup"><span data-stu-id="85aa5-127">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="85aa5-128">打开**SignInViewController**文件并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="85aa5-128">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

    ```objc
    #import "SignInViewController.h"
    #import "SpinnerViewController.h"
    #import "AuthenticationManager.h"

    @interface SignInViewController ()
    @property SpinnerViewController* spinner;
    @end

    @implementation SignInViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [AuthenticationManager.instance
         getTokenSilentlyWithCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error || !accessToken) {
                     // If there is no token or if there's an error,
                     // no user is signed in, so stay here
                     return;
                 }

                 // Since we got a token, user is signed in
                 // Go to welcome page
                 [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
             });
        }];
    }

    - (IBAction)signIn {
        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [AuthenticationManager.instance
         getTokenInteractivelyWithCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error || !accessToken) {
                     // Show the error and stay on the sign-in page
                     UIAlertController* alert = [UIAlertController
                                                 alertControllerWithTitle:@"Error signing in"
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

                 // Since we got a token, user is signed in
                 // Go to welcome page
                 [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
             });
         }];
    }
    @end
    ```

1. <span data-ttu-id="85aa5-129">打开**WelcomeViewController** ，并将现有`signOut`函数替换为以下项。</span><span class="sxs-lookup"><span data-stu-id="85aa5-129">Open **WelcomeViewController.m** and replace the existing `signOut` function with the following.</span></span>

    ```objc
    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }
    ```

1. <span data-ttu-id="85aa5-130">保存所做的更改，并在模拟器中重新启动应用程序。</span><span class="sxs-lookup"><span data-stu-id="85aa5-130">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="85aa5-131">如果登录到应用，您应该会看到在 Xcode 的输出窗口中显示的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="85aa5-131">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![显示访问令牌的 Xcode 中的输出窗口的屏幕截图](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="85aa5-133">获取用户详细信息</span><span class="sxs-lookup"><span data-stu-id="85aa5-133">Get user details</span></span>

<span data-ttu-id="85aa5-134">在本节中，您将创建一个帮助程序类，以保存对 Microsoft Graph 的所有调用并`WelcomeViewController`更新，以使用此新类获取已登录的用户。</span><span class="sxs-lookup"><span data-stu-id="85aa5-134">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="85aa5-135">打开**authenticationmanager.java** ，并在文件顶部添加`#import`以下语句。</span><span class="sxs-lookup"><span data-stu-id="85aa5-135">Open **AuthenticationManager.h** and add the following `#import` statement at the top of the file.</span></span>

    ```objc
    #import <MSGraphMSALAuthProvider/MSGraphMSALAuthProvider.h>
    ```

1. <span data-ttu-id="85aa5-136">在`@interface`声明中添加以下行。</span><span class="sxs-lookup"><span data-stu-id="85aa5-136">Add the following line inside the `@interface` declaration.</span></span>

    ```objc
    - (MSALAuthenticationProvider*) getGraphAuthProvider;
    ```

1. <span data-ttu-id="85aa5-137">打开**authenticationmanager.java** ，并将以下函数添加到`AuthenticationManager`类中。</span><span class="sxs-lookup"><span data-stu-id="85aa5-137">Open **AuthenticationManager.m** and add the following function to the `AuthenticationManager` class.</span></span>

    ```objc
    - (MSALAuthenticationProvider*) getGraphAuthProvider {
        // Create an MSAL auth provider for use with the Graph client
        return [[MSALAuthenticationProvider alloc]
                initWithPublicClientApplication:self.publicClient
                andScopes:self.graphScopes];
    }
    ```

1. <span data-ttu-id="85aa5-138">在名为**GraphManager**的**GraphTutorial**项目中创建一个新的**Cocoa Touch 类**。</span><span class="sxs-lookup"><span data-stu-id="85aa5-138">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="85aa5-139">在字段的 "**子类**" 中选择 " **NSObject** "。</span><span class="sxs-lookup"><span data-stu-id="85aa5-139">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="85aa5-140">打开**GraphManager** ，并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="85aa5-140">Open **GraphManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user, NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock)completionBlock;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="85aa5-141">打开**GraphManager**并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="85aa5-141">Open **GraphManager.m** and replace its contents with the following code.</span></span>

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

            MSALAuthenticationProvider* authProvider =
            [AuthenticationManager.instance getGraphAuthProvider];

            // Create the Graph client
            self.graphClient = [MSClientFactory
                                createHTTPClientWithAuthenticationProvider:authProvider];
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

1. <span data-ttu-id="85aa5-142">打开**WelcomeViewController** ，并在文件顶部添加`#import`以下语句。</span><span class="sxs-lookup"><span data-stu-id="85aa5-142">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="85aa5-143">将以下属性添加到`WelcomeViewController`接口声明中。</span><span class="sxs-lookup"><span data-stu-id="85aa5-143">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="85aa5-144">将现有`viewDidLoad`替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="85aa5-144">Replace the existing `viewDidLoad` with the following code.</span></span>

    ```objc
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
    ```

<span data-ttu-id="85aa5-145">如果现在保存更改并重新启动应用程序，则在使用用户的显示名称和电子邮件地址更新 UI 后，登录 UI。</span><span class="sxs-lookup"><span data-stu-id="85aa5-145">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
