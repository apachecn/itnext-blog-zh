# ASP.net core 3x 微服务介绍(下)

> 原文：<https://itnext.io/an-introduction-of-a-microservices-with-asp-net-core-3x-part-2-628acc712a85?source=collection_archive---------3----------------------->

在您继续之前…

我们希望开始编写代码，并将我们的微服务部署到 Amazon Web Services。所以你需要准备一些东西。

第一步:准备好源代码控制系统。拥有一个源代码控制系统最简单的方法就是在 https://github.com[的 GitHub 注册一个免费账户。](https://github.com/)

**第二步:**安装。NET Core SDK。它可以是任何。网芯 SDK 只要不是。Net Core 3.0 或 3.1
安装 Visual Studio 2017 或更高版本

第三步:在 AWS([https://aws.amazon.com/free](https://aws.amazon.com/free))创建一个免费账户。您将需要此帐户来部署您的微服务。

如果您没有联系第 1 部分，请点击下面的链接。
(关于微服务架构和单片应用/架构及问题的介绍。)
采用 ASP.net 酷睿 3x 的微服务介绍(第一部分)
[https://qiita.com/alokrawat050/items/66a512dc840827107260](https://qiita.com/alokrawat050/items/66a512dc840827107260)

在这一部分，我们将进行注册，确认电子邮件，并将使用 AWS 设置。
让我们创建一个项目文件，并将进行 AWS Cognito 设置部分:

**第一步:打开 you vs 代码，创建一个. net 核心应用。**

![](img/8696d638b737029792cd53c4728965d9.png)

<: class="jp ir">1————>

![](img/76924d678140c88267f4df52ce00614f.png)

<: class="jp ir">2————>

![](img/149be082a78ed8a5df03373a2c19b9e4.png)

<: class="jp ir">3————>

![](img/84c577d431d9ac62c41b7de97137b849.png)

**第二步:在**项目中添加几个 AWS 包
进入“**项目**”->“**管理 NuGet 包……**”

![](img/c96c8df9ad9c8afe76a2acc7bd0ef7ce.png)

然后搜索“**亚马逊。AspNetCore . identity . cognito**

![](img/e8e24acb31a5fdd6134911552a0bf243.png)

然后搜索“**亚马逊。扩展. CognitoAuthentication**

![](img/e7ec52ea03ef97dcb9c543a302467d49.png)

**第三步:在 AWS**
中创建一个新用户这里我已经创建了一个名为“ **ad_word_user** 的新用户

![](img/1e70d5c3b0741ee134eff07cf87da830.png)

<: class="jp ir">1————>

![](img/a6dd0e6173eac2738e007aced49a83f3.png)

<: class="jp ir">2————>

![](img/5d8fb87fbdf7d51bf7d38521a18f3e80.png)

现在设置一个新的权限，**Amazon cognitodeveloperauthenticatedidentities**

![](img/168fe59c86bb5443a4f357e0290d476a.png)

**第四步:设置 AWS 认知到**
从您的仪表板中搜索**认知到**。

![](img/79ce75e073c9f444a9e1ac30bb57368e.png)

<: class="jp ir">3————>

![](img/1e2ba93a4ebe8bd755486f26a5ad2bbe.png)

<: class="jp ir">4————>

![](img/ee402e92404c864930d07cfe46068e64.png)

<: — — — — — — — — — — — — — — -5— — — — — — — — — — — — — -:>

![](img/8c402e2458d116ef6a86b3b55be44c2b.png)

<: class="jp ir">6————>

![](img/a91e872778c90194cd87a0494ee2f1e2.png)

<: class="jp ir">7————>

![](img/3b923e1d12e289175a3a4b61f8dd31b7.png)

<: class="jp ir">8————>

![](img/53bc000383e304a62c3e8e1dadfa84ca.png)

<: class="jp ir">9————>

![](img/4b0f29ea97d791535abadaeba0680841.png)

<: class="jp ir">10————>

![](img/bffaf1dfa16735d095e66acff90bc9ac.png)

<: class="jp ir">11————>

![](img/6445f0fd766e97492b4a9bd9728423a5.png)

<: class="jp ir">1**2**————>

![](img/800910ecc8ce55d4c42b2fd8a72ac149.png)

<: class="jp ir">13————>

![](img/067bcfa9cb09ddd6a2112e33ffba3914.png)

**第五步:新建一个 app 客户端**
新建一个 App 客户端，复制 **app_client_id** 和 **app_client_secret**

![](img/fbf1e56828d1b707182318e6767f2a89.png)

<: class="jp ir">1————>

![](img/534b63e4083085bd61473aa20b46bffa.png)

<: class="jp ir">2————>

![](img/ac0c50f203e299218ebf506a6ee1fa19.png)

**第六步:设置 app 客户端 ID 和客户端密码**
打开文件 **appsettings.json** 复制以下代码粘贴到你的文件中。

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "AWS": {
    "Region": "<YOUR_REGION>",
    "UserPoolClientId": "<YOUR_CLIENT_ID>",
    "UserPoolClientSecret": "<YOUR_CLIENT_SECRET>",
    "UserPoolId": "<YOUR_POOL_ID>"
  }
}
```

**用户池 Id** ，

![](img/3e6dde1ace919b21f527f082b5a79142.png)

**第七步:在以下目录下创建以下文件:**
■。在**控制器**目录下，创建一个 **Accounts.cs** 控制器。

```
right-click on Controllers directory
   add -> new file
   ASP.NET Core -> MVC Controller Class
```

■.在 **Models** 目录下，新建一个名为 **Accounts** 的目录，在新建的目录下，新建两个模型文件 **ConfirmModel.cs** 和 **SignupModel.cs**

```
right-click on Models directory
   add -> new folder -> [Accounts] then right-click on the newly created directory, called Accounts
   add -> new class
   General -> Empty Class
```

■.在**视图**目录中，创建一个名为 **Accounts** 的新目录，并在新创建的目录中创建两个模型文件 **Confirm.cshtml** 和 **Signup.cshtml**

```
right-click on Views directory
   add -> new folder -> [Accounts] then right-click on newly created directory, called Accounts
   add -> new file
   ASP.NET Core -> Razor Page
```

**第八步: **Accounts.cs** 文件中的编码:**T63，

```
using System.Threading.Tasks;
using Amazon.AspNetCore.Identity.Cognito;
using Amazon.Extensions.CognitoAuthentication;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using WebAdWord.Models.Accounts;public class Accounts : Controller
    {
        private readonly SignInManager<CognitoUser> _signInManager;
        private readonly UserManager<CognitoUser> _userManager;
        private readonly CognitoUserPool _pool; public Accounts(SignInManager<CognitoUser> signInManager, UserManager<CognitoUser> userManager, CognitoUserPool pool)
        {
            _signInManager = signInManager;
            _userManager = userManager;
            _pool = pool;
        }
    }public async Task<IActionResult> Signup()
        {
            var model = new SignupModel();
            return View(model);
        }[HttpPost]
        public async Task<IActionResult> Signup(SignupModel model)
        {
            if (ModelState.IsValid)
            {
                var user = _pool.GetUser(model.Email);
                if (user.Status != null)
                {
                    ModelState.AddModelError("UserExists", "User with this email alred exits");
                    return View(model);
                } user.Attributes.Add(CognitoAttribute.Name.AttributeName, model.Email);
                var createdUser = await _userManager.CreateAsync(user, model.Password).ConfigureAwait(false); if (createdUser.Succeeded)
                {
                    RedirectToAction("Confirm");
                }
            }
            return View();
        }[HttpGet]
        public async Task<IActionResult> Confirm(ConfirmModel model)
        {            
            return View(model);
        } [HttpPost]
        [ActionName("Confirm")]
        public async Task<IActionResult> Confirm_Post(ConfirmModel model)
        {
            if (ModelState.IsValid)
            {
                var user = await _userManager.FindByEmailAsync(model.Email).ConfigureAwait(false);
                if (user == null)
                {
                    ModelState.AddModelError("NotFound", "A user with the given email address was not found.");
                    return View(model);
                } var result = await (_userManager as CognitoUserManager<CognitoUser>).ConfirmSignUpAsync(user, model.Code, true).ConfigureAwait(false);
                if (result.Succeeded)
                {
                    return RedirectToAction("Index", "Home");
                }
                else
                {
                    foreach(var item in result.Errors)
                    {
                        ModelState.AddModelError(item.Code, item.Description);
                    } return View(model);
                }
            } return View(model);
        }
```

以上所有代码属于⬇︎
**公共类账户:控制人{
—此处—
}**

在**模型/账户/SignupModel.cs** 文件中，

```
using System.ComponentModel.DataAnnotations;namespace WebAdWord.Models.Accounts
{
    public class SignupModel
    {
        [Required]
        [EmailAddress]
        [Display(Name ="Email")]
        public string Email{ get; set; } [Required]
        [DataType(DataType.Password)]
        [StringLength(8, ErrorMessage ="Password must be at least six characters long!")]
        [Display(Name ="Password")]
        public string Password { get; set; } [Required]
        [DataType(DataType.Password)]
        [Compare("Password", ErrorMessage ="Password and its confirmation do not match")]
        public string ConfirmPassword { get; set; } public SignupModel()
        {
        }
    }
}
```

在**模型/账户/确认模型. cs** 文件中，

```
using System.ComponentModel.DataAnnotations;namespace WebAdWord.Models.Accounts
{
    public class ConfirmModel
    {
        [Required(ErrorMessage = "Email is required.")]
        [Display(Name ="Email")]
        [EmailAddress]
        public string Email { get; set; } [Required(ErrorMessage ="Code is required.")]
        public string Code { get; set; }        
    }
}
```

在**Views/Accounts/sign up . cs html**文件中，

```
@{
    ViewData["Title"] = "SignUp Page";
}@model WebAdWord.Models.Accounts.SignupModel;<div class="row">
    <div class="col-md-4">
        <form method="post" asp-controller="Accounts" asp-action="Signup">
            <h4>Create a new account</h4>
            <hr />
            <div asp-validation-summary="All" class="text-danger"> </div> <div class="form-group">
                <label asp-for="Email"></label>
                <input asp-for="Email" class="form-control" />
                <span asp-validation-for="Email" class="text-danger"></span>
            </div> <div class="form-group">
                <label asp-for="Password"></label>
                <input asp-for="Password" class="form-control" />
                <span asp-validation-for="Password" class="text-danger"></span>
            </div> <div class="form-group">
                <label asp-for="ConfirmPassword"></label>
                <input asp-for="ConfirmPassword" class="form-control" />
                <span asp-validation-for="ConfirmPassword" class="text-danger"></span>
            </div> <button type="submit" class="btn btn-default">Sign Up</button>
        </form>
    </div>
</div>
```

在**Views/Accounts/confirm . cs html**文件中，

```
@{
    ViewData["Title"] = "Confirm Page";
}@model WebAdWord.Models.Accounts.ConfirmModel;<div class="row">
    <div class="col-md-4">
        <form method="post" asp-controller="Accounts" asp-action="Confirm">
            <h4>Confirm the new account</h4>
            <hr />
            <div asp-validation-summary="All" class="text-danger"></div> <div class="form-group">
                <label asp-for="Email"></label>
                <input asp-for="Email" />
                <span asp-validation-for="Email" class="text-danger"></span>
            </div> <div class="form-group">
                <label asp-for="Code"></label>
                <input asp-for="Code" />
                <span asp-validation-for="Code" class="text-danger"></span>
            </div> <button type="submit">Confirm</button>
        </form>
    </div>
</div>
```

**第九步:调试代码:**

注册网址:[https://localhost:5001/Accounts/](https://localhost:5001/Accounts/Confirm)注册

![](img/166f6ca59fc3b235dce14c532e0d3a5b.png)

注册后，你会收到一封邮件。

![](img/037293e1d809428c0b85b1ec8d0499b0.png)

复制验证码然后进入下面的网址:
https://localhost:5001/Accounts/Confirm

![](img/712340c0d046e177d586fa0f60916f26.png)

成功确认后，您将重定向到主页。

![](img/65993ae47e4c551a4125027205fde9a3.png)

**步骤 10: AWS Cognito 仪表板**
在这里，您可以看到新创建的用户在仪表板中的显示。

![](img/68c8e4f28de948f5a240546162319a3f.png)

在下一部分中，我们将实现登录、忘记密码和重置密码功能。

如果你遇到任何错误，请与我分享。
如果本指南对您和您的团队有所帮助，请与他人分享！

**感谢&最诚挚的问候，
Alok Rawat**