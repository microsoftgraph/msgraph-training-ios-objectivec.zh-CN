<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="af4f6-101">首先创建一个新的 Swift 项目。</span><span class="sxs-lookup"><span data-stu-id="af4f6-101">Begin by creating a new Swift project.</span></span>

1. <span data-ttu-id="af4f6-102">打开 Xcode。</span><span class="sxs-lookup"><span data-stu-id="af4f6-102">Open Xcode.</span></span> <span data-ttu-id="af4f6-103">在 "**文件**" 菜单上，依次选择 "**新建**"、"**项目**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-103">On the **File** menu, select **New**, then **Project**.</span></span>
1. <span data-ttu-id="af4f6-104">选择 "**单个视图应用程序**" 模板，然后选择 "**下一步**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-104">Choose the **Single View App** template and select **Next**.</span></span>

    ![Xcode 模板选择对话框的屏幕截图](./images/xcode-select-template.png)

1. <span data-ttu-id="af4f6-106">将 "**产品名称**" `GraphTutorial`和 "**语言**" 设置为 "**目标-C**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-106">Set the **Product Name** to `GraphTutorial` and the **Language** to **Objective-C**.</span></span>
1. <span data-ttu-id="af4f6-107">填写其余字段，然后选择 "**下一步**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-107">Fill in the remaining fields and select **Next**.</span></span>
1. <span data-ttu-id="af4f6-108">选择项目的位置并选择 "**创建**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-108">Choose a location for the project and select **Create**.</span></span>

## <a name="install-dependencies"></a><span data-ttu-id="af4f6-109">安装依赖项</span><span class="sxs-lookup"><span data-stu-id="af4f6-109">Install dependencies</span></span>

<span data-ttu-id="af4f6-110">在继续之前，请安装稍后将使用的一些其他依赖项。</span><span class="sxs-lookup"><span data-stu-id="af4f6-110">Before moving on, install some additional dependencies that you will use later.</span></span>

- <span data-ttu-id="af4f6-111">适用于[iOS 的 Microsoft 身份验证库（MSAL）](https://github.com/AzureAD/microsoft-authentication-library-for-objc) ，用于向 Azure AD 进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="af4f6-111">[Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) for authenticating to with Azure AD.</span></span>
- <span data-ttu-id="af4f6-112">[Microsoft GRAPH SDK For 客观 C](https://github.com/microsoftgraph/msgraph-sdk-objc) ，用于调用 Microsoft graph。</span><span class="sxs-lookup"><span data-stu-id="af4f6-112">[Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="af4f6-113">[Microsoft Graph 模型 SDK （针对目标 C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) ）对于强类型的对象，表示 Microsoft Graph 资源，如用户或事件。</span><span class="sxs-lookup"><span data-stu-id="af4f6-113">[Microsoft Graph Models SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) for strongly-typed objects representing Microsoft Graph resources like users or events.</span></span>

1. <span data-ttu-id="af4f6-114">退出 Xcode。</span><span class="sxs-lookup"><span data-stu-id="af4f6-114">Quit Xcode.</span></span>
1. <span data-ttu-id="af4f6-115">打开 "终端" 并将目录更改为**GraphTutorial**项目的位置。</span><span class="sxs-lookup"><span data-stu-id="af4f6-115">Open Terminal and change the directory to the location of your **GraphTutorial** project.</span></span>
1. <span data-ttu-id="af4f6-116">运行以下命令以创建 Podfile。</span><span class="sxs-lookup"><span data-stu-id="af4f6-116">Run the following command to create a Podfile.</span></span>

    ```Shell
    pod init
    ```

1. <span data-ttu-id="af4f6-117">打开 Podfile，并在行的`use_frameworks!`后面添加以下行。</span><span class="sxs-lookup"><span data-stu-id="af4f6-117">Open the Podfile and add the following lines just after the `use_frameworks!` line.</span></span>

    ```Ruby
    pod 'MSAL', '~> 1.0.2'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. <span data-ttu-id="af4f6-118">保存 Podfile，然后运行以下命令来安装依赖项。</span><span class="sxs-lookup"><span data-stu-id="af4f6-118">Save the Podfile, then run the following command to install the dependencies.</span></span>

    ```Shell
    pod install
    ```

1. <span data-ttu-id="af4f6-119">命令完成后，在 Xcode 中打开新创建的**GraphTutorial graph-ios-swift-connect.xcworkspace** 。</span><span class="sxs-lookup"><span data-stu-id="af4f6-119">Once the command completes, open the newly created **GraphTutorial.xcworkspace** in Xcode.</span></span>

## <a name="design-the-app"></a><span data-ttu-id="af4f6-120">设计应用程序</span><span class="sxs-lookup"><span data-stu-id="af4f6-120">Design the app</span></span>

<span data-ttu-id="af4f6-121">在本节中，您将为应用程序创建视图：登录页面、选项卡栏导航器、欢迎页面和日历页面。</span><span class="sxs-lookup"><span data-stu-id="af4f6-121">In this section you will create the views for the app: a sign in page, a tab bar navigator, a welcome page, and a calendar page.</span></span> <span data-ttu-id="af4f6-122">您还将创建活动指示器覆盖。</span><span class="sxs-lookup"><span data-stu-id="af4f6-122">You'll also create an activity indicator overlay.</span></span>

### <a name="create-sign-in-page"></a><span data-ttu-id="af4f6-123">创建登录页</span><span class="sxs-lookup"><span data-stu-id="af4f6-123">Create sign in page</span></span>

1. <span data-ttu-id="af4f6-124">展开 Xcode 中的 " **GraphTutorial** " 文件夹，然后选择**viewcontroller.m**文件。</span><span class="sxs-lookup"><span data-stu-id="af4f6-124">Expand the **GraphTutorial** folder in Xcode, then select the **ViewController.m** file.</span></span>
1. <span data-ttu-id="af4f6-125">在**文件检查器**中，将文件的**名称**更改为`SignInViewController.m`。</span><span class="sxs-lookup"><span data-stu-id="af4f6-125">In the **File Inspector**, change the **Name** of the file to `SignInViewController.m`.</span></span>

    ![文件检查器的屏幕截图](./images/rename-file.png)

1. <span data-ttu-id="af4f6-127">打开**SignInViewController**并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="af4f6-127">Open **SignInViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "SignInViewController.h"

    @interface SignInViewController ()

    @end

    @implementation SignInViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.
    }

    - (IBAction)signIn {
        [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
    }
    @end
    ```

1. <span data-ttu-id="af4f6-128">选择 " **viewcontroller.m** " 文件。</span><span class="sxs-lookup"><span data-stu-id="af4f6-128">Select the **ViewController.h** file.</span></span>
1. <span data-ttu-id="af4f6-129">在**文件检查器**中，将文件的**名称**更改为`SignInViewController.h`。</span><span class="sxs-lookup"><span data-stu-id="af4f6-129">In the **File Inspector**, change the **Name** of the file to `SignInViewController.h`.</span></span>
1. <span data-ttu-id="af4f6-130">打开**SignInViewController** ，并将的`ViewController`所有实例更改为`SignInViewController`。</span><span class="sxs-lookup"><span data-stu-id="af4f6-130">Open **SignInViewController.h** and change all instances of `ViewController` to `SignInViewController`.</span></span>

1. <span data-ttu-id="af4f6-131">打开 " **storyboard** " 文件。</span><span class="sxs-lookup"><span data-stu-id="af4f6-131">Open the **Main.storyboard** file.</span></span>
1. <span data-ttu-id="af4f6-132">展开 "**查看控制器场景**"，然后选择 "**查看控制器**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-132">Expand **View Controller Scene**, then select **View Controller**.</span></span>

    ![选定了视图控制器的 Xcode 的屏幕截图](./images/storyboard-select-view-controller.png)

1. <span data-ttu-id="af4f6-134">选择 "**标识" 检查器**，然后将 "**类**" 下拉列表更改为 " **SignInViewController**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-134">Select the **Identity Inspector**, then change the **Class** dropdown to **SignInViewController**.</span></span>

    ![标识检查器的屏幕截图](./images/change-class.png)

1. <span data-ttu-id="af4f6-136">选择**库**，然后将**按钮**拖到 "**登录视图" 控制器**上。</span><span class="sxs-lookup"><span data-stu-id="af4f6-136">Select the **Library**, then drag a **Button** onto the **Sign In View Controller**.</span></span>

    ![Xcode 中库的屏幕截图](./images/add-button-to-view.png)

1. <span data-ttu-id="af4f6-138">选择该按钮后，选择 "**属性" 检查器**， \*\*\*\* 并将按钮的标题`Sign In`更改为。</span><span class="sxs-lookup"><span data-stu-id="af4f6-138">With the button selected, select the **Attributes Inspector** and change the **Title** of the button to `Sign In`.</span></span>

    ![Xcode 中的 "属性" 检查器中 "标题" 字段的屏幕截图](./images/set-button-title.png)

1. <span data-ttu-id="af4f6-140">选择 "**登录视图控制器**"，然后选择 "**连接检查器**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-140">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="af4f6-141">在 "**接收的操作**" 下，将 "**登录**" 旁边的未填充圆圈拖到按钮上。</span><span class="sxs-lookup"><span data-stu-id="af4f6-141">Under **Received Actions**, drag the unfilled circle next to **signIn** onto the button.</span></span> <span data-ttu-id="af4f6-142">在弹出菜单上选择 "**向上触摸**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-142">Select **Touch Up Inside** on the pop-up menu.</span></span>

    ![将登录操作拖到 Xcode 中的按钮的屏幕截图](./images/connect-sign-in-button.png)

1. <span data-ttu-id="af4f6-144">在**编辑器**菜单上，选择 "**解决自动布局问题**"，然后**在 "登录视图控制器" 中选择 "在所有视图的**下方**添加缺少的约束**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-144">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Sign In View Controller**.</span></span>

### <a name="create-tab-bar"></a><span data-ttu-id="af4f6-145">创建选项卡栏</span><span class="sxs-lookup"><span data-stu-id="af4f6-145">Create tab bar</span></span>

1. <span data-ttu-id="af4f6-146">选择**库**，然后将**选项卡栏控制器**拖到情节提要上。</span><span class="sxs-lookup"><span data-stu-id="af4f6-146">Select the **Library**, then drag a **Tab Bar Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="af4f6-147">选择 "**登录视图控制器**"，然后选择 "**连接检查器**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-147">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="af4f6-148">在 "已**触发 Segues**" 下，将 "**手动**" 旁边的未填充圆圈拖到情节提要上的**选项卡栏控制器**上。</span><span class="sxs-lookup"><span data-stu-id="af4f6-148">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Tab Bar Controller** on the storyboard.</span></span> <span data-ttu-id="af4f6-149">在弹出菜单中选择 "**模式显示**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-149">Select **Present Modally** in the pop-up menu.</span></span>

    ![将手动 segue 拖动到 Xcode 中新选项卡栏控制器中的屏幕截图](./images/add-segue.png)

1. <span data-ttu-id="af4f6-151">选择您刚刚添加的 segue，然后选择 "**属性" 检查器**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-151">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="af4f6-152">将 "**标识符**" 字段`userSignedIn`**设置为，** 并将 "**演示文稿**" 设置为全屏。</span><span class="sxs-lookup"><span data-stu-id="af4f6-152">Set the **Identifier** field to `userSignedIn`, and set **Presentation** to **Full Screen**.</span></span>

    ![Xcode 中的 "属性" 检查器中的 "标识符" 字段的屏幕截图](./images/set-segue-identifier.png)

1. <span data-ttu-id="af4f6-154">选择 "**项目1场景**"，然后选择 "**连接检查器**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-154">Select the **Item 1 Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="af4f6-155">在 "已**触发 Segues**" 下，将 "**手动**" 旁边的未填充圆圈拖到情节提要上的 "**登录视图控制器" 中**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-155">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Sign In View Controller** on the storyboard.</span></span> <span data-ttu-id="af4f6-156">在弹出菜单中选择 "**模式显示**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-156">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="af4f6-157">选择您刚刚添加的 segue，然后选择 "**属性" 检查器**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-157">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="af4f6-158">将 "**标识符**" 字段`userSignedOut`**设置为，** 并将 "**演示文稿**" 设置为全屏。</span><span class="sxs-lookup"><span data-stu-id="af4f6-158">Set the **Identifier** field to `userSignedOut`, and set **Presentation** to **Full Screen**.</span></span>

### <a name="create-welcome-page"></a><span data-ttu-id="af4f6-159">创建欢迎页面</span><span class="sxs-lookup"><span data-stu-id="af4f6-159">Create welcome page</span></span>

1. <span data-ttu-id="af4f6-160">选择 " **xcassets** " 文件。</span><span class="sxs-lookup"><span data-stu-id="af4f6-160">Select the **Assets.xcassets** file.</span></span>
1. <span data-ttu-id="af4f6-161">在**编辑器**菜单上，选择 "**添加资源**"，然后选择 "**新建图像集**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-161">On the **Editor** menu, select **Add Assets**, then **New Image Set**.</span></span>
1. <span data-ttu-id="af4f6-162">选择新**图像**资产，并使用 "**属性" 检查器**将其**名称**设置为`DefaultUserPhoto`。</span><span class="sxs-lookup"><span data-stu-id="af4f6-162">Select the new **Image** asset and use the **Attribute Inspector** to set its **Name** to `DefaultUserPhoto`.</span></span>
1. <span data-ttu-id="af4f6-163">添加您希望用作默认用户配置文件照片的任何图像。</span><span class="sxs-lookup"><span data-stu-id="af4f6-163">Add any image you like to serve as a default user profile photo.</span></span>

    ![图像在 Xcode 中设置资产视图的屏幕截图](./images/add-default-user-photo.png)

1. <span data-ttu-id="af4f6-165">在名为`WelcomeViewController`的**GraphTutorial**文件夹中创建一个新的**Cocoa Touch 类**文件。</span><span class="sxs-lookup"><span data-stu-id="af4f6-165">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `WelcomeViewController`.</span></span> <span data-ttu-id="af4f6-166">在字段的 "**子类**" 中选择 " **UIViewController** "。</span><span class="sxs-lookup"><span data-stu-id="af4f6-166">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="af4f6-167">打开**WelcomeViewController** ，并将以下代码添加到`@interface`声明中。</span><span class="sxs-lookup"><span data-stu-id="af4f6-167">Open **WelcomeViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    @property (nonatomic) IBOutlet UIImageView *userProfilePhoto;
    @property (nonatomic) IBOutlet UILabel *userDisplayName;
    @property (nonatomic) IBOutlet UILabel *userEmail;
    ```

1. <span data-ttu-id="af4f6-168">打开**WelcomeViewController**并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="af4f6-168">Open **WelcomeViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "WelcomeViewController.h"

    @interface WelcomeViewController ()

    @end

    @implementation WelcomeViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        // TEMPORARY
        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];
        self.userDisplayName.text = @"Default User";
        [self.userDisplayName sizeToFit];
        self.userEmail.text = @"default@contoso.com";
        [self.userEmail sizeToFit];
    }

    - (IBAction)signOut {
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }

    @end
    ```

1. <span data-ttu-id="af4f6-169">打开 "**情节提要**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-169">Open **Main.storyboard**.</span></span> <span data-ttu-id="af4f6-170">选择 "**项目1场景**"，然后选择 "**标识检查器**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-170">Select the **Item 1 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="af4f6-171">将 "**类**" 值更改为**WelcomeViewController**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-171">Change the **Class** value to **WelcomeViewController**.</span></span>
1. <span data-ttu-id="af4f6-172">使用**库**将以下项添加到**Item 1 场景**中。</span><span class="sxs-lookup"><span data-stu-id="af4f6-172">Using the **Library**, add the following items to the **Item 1 Scene**.</span></span>

    - <span data-ttu-id="af4f6-173">一个**图像视图**</span><span class="sxs-lookup"><span data-stu-id="af4f6-173">One **Image View**</span></span>
    - <span data-ttu-id="af4f6-174">两个**标签**</span><span class="sxs-lookup"><span data-stu-id="af4f6-174">Two **Labels**</span></span>
    - <span data-ttu-id="af4f6-175">一个**按钮**</span><span class="sxs-lookup"><span data-stu-id="af4f6-175">One **Button**</span></span>

1. <span data-ttu-id="af4f6-176">选择 "图像" 视图，然后选择 "**大小" 检查器**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-176">Select the image view, then select the **Size Inspector**.</span></span>
1. <span data-ttu-id="af4f6-177">将 "**宽度**" 和 "**高度**" 设置为196。</span><span class="sxs-lookup"><span data-stu-id="af4f6-177">Set the **Width** and **Height** to 196.</span></span>
1. <span data-ttu-id="af4f6-178">选择第二个标签，然后选择 "**属性" 检查器**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-178">Select the second label, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="af4f6-179">将**颜色**更改为**深灰色**的颜色，并将**字体**更改为**系统 12.0**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-179">Change the **Color** to **Dark Gray Color**, and change the **Font** to **System 12.0**.</span></span>
1. <span data-ttu-id="af4f6-180">选择该按钮，然后选择 "**属性" 检查器**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-180">Select the button, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="af4f6-181">将 "**标题**" `Sign Out`更改为。</span><span class="sxs-lookup"><span data-stu-id="af4f6-181">Change the **Title** to `Sign Out`.</span></span>
1. <span data-ttu-id="af4f6-182">使用 "**连接检查器**" 进行以下连接。</span><span class="sxs-lookup"><span data-stu-id="af4f6-182">Using the **Connections Inspector**, make the following connections.</span></span>

    - <span data-ttu-id="af4f6-183">将**userDisplayName**插座链接到第一个标签。</span><span class="sxs-lookup"><span data-stu-id="af4f6-183">Link the **userDisplayName** outlet to the first label.</span></span>
    - <span data-ttu-id="af4f6-184">将**userEmail**插座链接到第二个标签。</span><span class="sxs-lookup"><span data-stu-id="af4f6-184">Link the **userEmail** outlet to the second label.</span></span>
    - <span data-ttu-id="af4f6-185">将**userProfilePhoto**插座链接到图像视图。</span><span class="sxs-lookup"><span data-stu-id="af4f6-185">Link the **userProfilePhoto** outlet to the image view.</span></span>
    - <span data-ttu-id="af4f6-186">将**signOut**接收的操作链接到按钮的**内部**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-186">Link the **signOut** received action to the button's **Touch Up Inside**.</span></span>

1. <span data-ttu-id="af4f6-187">选择场景底部的 "选项卡栏" 项，然后选择 "**属性" 检查器**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-187">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="af4f6-188">将 "**标题**" `Me`更改为。</span><span class="sxs-lookup"><span data-stu-id="af4f6-188">Change the **Title** to `Me`.</span></span>
1. <span data-ttu-id="af4f6-189">在**编辑器**菜单上，选择 "**解决自动布局问题**"，然后选择 **"欢迎视图控制器中的所有视图"** 下的 "**添加缺少约束**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-189">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>

<span data-ttu-id="af4f6-190">完成后，欢迎场景应如下所示。</span><span class="sxs-lookup"><span data-stu-id="af4f6-190">The welcome scene should look similar to this once you're done.</span></span>

![欢迎场景布局的屏幕截图](./images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a><span data-ttu-id="af4f6-192">创建日历页</span><span class="sxs-lookup"><span data-stu-id="af4f6-192">Create calendar page</span></span>

1. <span data-ttu-id="af4f6-193">在名为`CalendarViewController`的**GraphTutorial**文件夹中创建一个新的**Cocoa Touch 类**文件。</span><span class="sxs-lookup"><span data-stu-id="af4f6-193">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `CalendarViewController`.</span></span> <span data-ttu-id="af4f6-194">在字段的 "**子类**" 中选择 " **UIViewController** "。</span><span class="sxs-lookup"><span data-stu-id="af4f6-194">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="af4f6-195">打开**CalendarViewController** ，并将以下代码添加到`@interface`声明中。</span><span class="sxs-lookup"><span data-stu-id="af4f6-195">Open **CalendarViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    @property (nonatomic) IBOutlet UITextView *calendarJSON;
    ```

1. <span data-ttu-id="af4f6-196">打开**CalendarViewController**并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="af4f6-196">Open **CalendarViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "CalendarViewController.h"

    @interface CalendarViewController ()

    @end

    @implementation CalendarViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        // TEMPORARY
        self.calendarJSON.text = @"Calendar";
        [self.calendarJSON sizeToFit];
    }

    @end
    ```

1. <span data-ttu-id="af4f6-197">打开 "**情节提要**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-197">Open **Main.storyboard**.</span></span> <span data-ttu-id="af4f6-198">选择 "**项目2场景**"，然后选择 "**标识检查器**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-198">Select the **Item 2 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="af4f6-199">将 "**类**" 值更改为**CalendarViewController**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-199">Change the **Class** value to **CalendarViewController**.</span></span>
1. <span data-ttu-id="af4f6-200">使用**库**向**项目2场景**添加**文本视图**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-200">Using the **Library**, add a **Text View** to the **Item 2 Scene**.</span></span>
1. <span data-ttu-id="af4f6-201">选择您刚添加的文本视图。</span><span class="sxs-lookup"><span data-stu-id="af4f6-201">Select the text view you just added.</span></span> <span data-ttu-id="af4f6-202">在**编辑器**中，依次选择 "**嵌入入**" 和 "**滚动视图**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-202">On the **Editor**, choose **Embed In**, then **Scroll View**.</span></span>
1. <span data-ttu-id="af4f6-203">使用 "**连接" 检查器**，将**calendarJSON**插座连接到文本视图。</span><span class="sxs-lookup"><span data-stu-id="af4f6-203">Using the **Connections Inspector**, connect the **calendarJSON** outlet to the text view.</span></span>
1. 1. <span data-ttu-id="af4f6-204">选择场景底部的 "选项卡栏" 项，然后选择 "**属性" 检查器**。</span><span class="sxs-lookup"><span data-stu-id="af4f6-204">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="af4f6-205">将 "**标题**" `Calendar`更改为。</span><span class="sxs-lookup"><span data-stu-id="af4f6-205">Change the **Title** to `Calendar`.</span></span>
1. <span data-ttu-id="af4f6-206">在**编辑器**菜单上，选择 "**解决自动布局问题**"，然后选择 **"欢迎视图控制器中的所有视图"** 下的 "**添加缺少约束**"。</span><span class="sxs-lookup"><span data-stu-id="af4f6-206">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Welcome View Controller**.</span></span>

<span data-ttu-id="af4f6-207">完成后，日历场景应如下所示。</span><span class="sxs-lookup"><span data-stu-id="af4f6-207">The calendar scene should look similar to this once you're done.</span></span>

![日历场景布局的屏幕截图](./images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a><span data-ttu-id="af4f6-209">创建活动指示器</span><span class="sxs-lookup"><span data-stu-id="af4f6-209">Create activity indicator</span></span>

1. <span data-ttu-id="af4f6-210">在名为`SpinnerViewController`的**GraphTutorial**文件夹中创建一个新的**Cocoa Touch 类**文件。</span><span class="sxs-lookup"><span data-stu-id="af4f6-210">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `SpinnerViewController`.</span></span> <span data-ttu-id="af4f6-211">在字段的 "**子类**" 中选择 " **UIViewController** "。</span><span class="sxs-lookup"><span data-stu-id="af4f6-211">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="af4f6-212">打开**SpinnerViewController** ，并将以下代码添加到`@interface`声明中。</span><span class="sxs-lookup"><span data-stu-id="af4f6-212">Open **SpinnerViewController.h** and add the following code inside the `@interface` declaration.</span></span>

    ```objc
    - (void) startWithContainer:(UIViewController*) container;
    - (void) stop;
    ```

1. <span data-ttu-id="af4f6-213">打开**SpinnerViewController**并将其内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="af4f6-213">Open **SpinnerViewController.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "SpinnerViewController.h"

    @interface SpinnerViewController ()
    @property (nonatomic) UIActivityIndicatorView* spinner;
    @end

    @implementation SpinnerViewController

    - (void)viewDidLoad {
        [super viewDidLoad];

        _spinner = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:
                    UIActivityIndicatorViewStyleWhiteLarge];

        self.view.backgroundColor = [UIColor colorWithWhite:0 alpha:0.7];
        [self.view addSubview:_spinner];

        _spinner.translatesAutoresizingMaskIntoConstraints = false;
        [_spinner startAnimating];

        [_spinner.centerXAnchor constraintEqualToAnchor:self.view.centerXAnchor].active = true;
        [_spinner.centerYAnchor constraintEqualToAnchor:self.view.centerYAnchor].active = true;
    }

    - (void) startWithContainer:(UIViewController *)container {
        [container addChildViewController:self];
        self.view.frame = container.view.frame;
        [container.view addSubview:self.view];
        [self didMoveToParentViewController:container];
    }

    - (void) stop {
        [self willMoveToParentViewController:nil];
        [self.view removeFromSuperview];
        [self removeFromParentViewController];
    }

    @end
    ```

## <a name="test-the-app"></a><span data-ttu-id="af4f6-214">测试应用程序</span><span class="sxs-lookup"><span data-stu-id="af4f6-214">Test the app</span></span>

<span data-ttu-id="af4f6-215">保存所做的更改并启动应用程序。</span><span class="sxs-lookup"><span data-stu-id="af4f6-215">Save your changes and launch the app.</span></span> <span data-ttu-id="af4f6-216">您应该能够使用 "**登录**" 和 "**注销**" 按钮和选项卡栏在屏幕之间移动。</span><span class="sxs-lookup"><span data-stu-id="af4f6-216">You should be able to move between the screens using the **Sign In** and **Sign Out** buttons and the tab bar.</span></span>

![应用程序的屏幕截图](./images/app-screens.png)
