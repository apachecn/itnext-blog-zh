# Azure 函数& wkhtmltopdf:将 HTML 转换为 pdf

> 原文：<https://itnext.io/to-azure-functions-wkhtmltopdf-convert-html-to-pdf-9dc69bcd843b?source=collection_archive---------2----------------------->

![](img/8326a467d2c45981934edc2cc76878fe.png)

我们将使用 Azure Functions &[wkhtmltopdf](https://wkhtmltopdf.org/)工具从 HTML 文件生成 pdf 文件。出于多种原因，您可能需要创建 PDF 文件，例如为销售生成发票、为患者生成医疗报告、为客户生成保险表格等。有几种方法可以做到这一点。

首先，你可以使用 [Adobe](https://www.adobe.com/in/acrobat/online/sign-pdf.html) 的填充和签名工具来填写表格，但这主要需要人工交互，因此不可扩展，也不方便。

第二种选择是你直接创建一个 pdf 文件。基于您正在使用的平台，您将拥有直接创建 pdf 文件的工具。如果这是一个非常简单的 pdf，你可以采取这种方法。

这让我们想到了最后一个也是最方便的选择。 [wkhtmltopdf](https://wkhtmltopdf.org/) 是一个将 HTML 转换成 pdf 的非常棒的工具。因为它是免费的、开源的，并且可以在几乎所有平台上编译，所以它是我们的最佳选择。

# 先决条件

*   微软 VS 代码
*   Azure 门户网站[上的账户](https://portal.azure.com/)
*   Linux Basic (B1)应用服务计划。如果您已经有 Windows Basic (B1)应用服务计划，您可以使用该计划。
*   Azure 存储帐户。

# Azure 函数

由于将 HTML 转换成 PDF 是一项耗时的任务，我们不应该在我们的主 web 服务器上运行它。否则，它可能会开始阻塞其他重要的请求。Azure 函数是委派此类任务的最佳方式。

为了创建一个函数，你首先需要在你的机器上安装 Azure 函数。基于你的操作系统安装 [Azure 功能核心工具](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=macos%2Ccsharp%2Cbash#install-the-azure-functions-core-tools)。安装完成后，打开命令行工具，启动下面的命令。html2pdf 是您的项目名称。你可以用任何名字代替它。

```
func init html2pdf
```

在执行命令时，它会要求一个工作运行时。这里选择 *1。dotnet* 作为微软的产品，它为 dotnet 提供了强大的支持。这将在当前目录中生成一个名为 *html2pdf* 的文件夹。由于 Visual Studio 代码允许直接发布到 Azure 函数，我们将使用它来编码和部署。

在 VS 代码中打开项目后，创建一个名为 *Html2Pdf.cs* 的文件。Azure 函数提供了各种各样的[触发器](https://www.serverless360.com/blog/azure-functions-triggers-and-bindings)来执行函数。现在，我们将从 http 触发器开始，即可以通过 HTTP 协议直接调用该函数。在我们新创建的文件中粘贴以下内容。

```
using System;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;namespace Html2Pdf
{
    public class Html2Pdf
    {
        // The name of the function
        [FunctionName("Html2Pdf")]

        // The first arugment tells that the functions can be triggerd by a POST HTTP request. 
        // The second argument is mainly used for logging information, warnings or errors
        public void Run([HttpTrigger(AuthorizationLevel.Function, "POST")] Html2PdfRequest Request, ILogger Log)
        {
        }
    }
}
```

我们已经创建了一个框架，现在我们将在其中填充细节。您可能已经注意到，请求变量的类型是 *Html2PdfRequest* 。所以让我们创建一个模型 *Html2PdfRequest.cs* 类如下。

```
namespace Html2Pdf
{
    public class Html2PdfRequest
    {
        // The HTML content that needs to be converted.
        public string HtmlContent { get; set; }

        // The name of the PDF file to be generated
        public string PDFFileName { get; set; }
    }
}
```

# 丁克托普夫

为了从我们的托管代码中调用 wkhtmltopdf，使用了一种称为 P/Invoke 的技术。简而言之，P/Invoke 允许我们访问非托管库中的结构、回调和函数。有一个很好的 P/Invoke 包装器，名为 [DinkToPdf](https://github.com/rdvojmoc/DinkToPdf) ，允许我们抽象出技术细节。

您可以通过 [nuget](https://www.nuget.org/packages/DinkToPdf/) 将 DinkToPdf 添加到您的项目中。只需从根文件夹中运行命令。

```
dotnet add package DinkToPdf --version 1.0.8
```

是时候在我们的类顶部添加一些代码了。

```
// Read more about converter on: [https://github.com/rdvojmoc/DinkToPdf](https://github.com/rdvojmoc/DinkToPdf)
// For our purposes we are going to use SynchronizedConverter
IPdfConverter pdfConverter = new SynchronizedConverter(new PdfTools());// A function to convert html content to pdf based on the configuration pased as arguments
// Arguments:
// HtmlContent: the html content to be converted
// Width: the width of the pdf to be created. e.g. "8.5in", "21.59cm" etc.
// Height: the height of the pdf to be created. e.g. "11in", "27.94cm" etc.
// Margins: the margis around the content
// DPI: The dpi is very important when you want to print the pdf.
// Returns a byte array of the pdf which can be stored as a file
private byte[] BuildPdf(string HtmlContent, string Width, string Height, MarginSettings Margins, int? DPI = 180)
{
  // Call the Convert method of SynchronizedConverter "pdfConverter"
  return pdfConverter.Convert(new HtmlToPdfDocument()
            {
                // Set the html content
                Objects =
                {
                    new ObjectSettings
                    {
                        HtmlContent = HtmlContent
                    }
                },
                // Set the configurations
                GlobalSettings = new GlobalSettings
                {
                    // PaperKind.A4 can also be used instead PechkinPaperSize
                    PaperSize = new PechkinPaperSize(Width, Height),
                    DPI = DPI,
                    Margins = Margins
                }
            });
}
```

我已经添加了行内注释，以便自我解释。如果你有任何问题，可以在下面的评论区问我。让我们从 Run 方法中调用上面创建的函数。

```
// PDFByteArray is a byte array of pdf generated from the HtmlContent 
var PDFByteArray = BuildPdf(Request.HtmlContent, "8.5in", "11in", new MarginSettings(0, 0, 0,0));
```

一旦生成了字节数组，让我们将它作为一个 blob 存储在 Azure 存储中。在上传 blob 之前，请创建一个容器。完成后，在 PDFByteArray 后添加以下代码。

```
// The connection string of the Storage Account to which our PDF file will be uploaded
// Make sure to replace with your connection string.
var StorageConnectionString = "DefaultEndpointsProtocol=https;AccountName=<YOUR ACCOUNT NAME>;AccountKey=<YOUR ACCOUNT KEY>;EndpointSuffix=core.windows.net";// Generate an instance of CloudStorageAccount by parsing the connection string
var StorageAccount = CloudStorageAccount.Parse(StorageConnectionString);// Create an instance of CloudBlobClient to connect to our storage account
CloudBlobClient BlobClient = StorageAccount.CreateCloudBlobClient();// Get the instance of CloudBlobContainer which points to a container name "pdf"
// Replace your own container name
CloudBlobContainer BlobContainer = BlobClient.GetContainerReference("pdf");// Get the instance of the CloudBlockBlob to which the PDFByteArray will be uploaded
CloudBlockBlob Blob = BlobContainer.GetBlockBlobReference(Request.PDFFileName);// Upload the pdf blob
await Blob.UploadFromByteArrayAsync(PDFByteArray, 0, PDFByteArray.Length);
```

添加此代码后，您会看到一些错误和警告。首先，添加缺失的导入语句。其次，将*运行*函数的返回类型从 *void* 改为 *async Task* 。下面是最终的 *Html2Pdf.cs* 文件的样子。

```
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using DinkToPdf;
using IPdfConverter = DinkToPdf.Contracts.IConverter;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using System.Threading.Tasks;namespace Html2Pdf
{
    public class Html2Pdf
    {
        // Read more about converter on: [https://github.com/rdvojmoc/DinkToPdf](https://github.com/rdvojmoc/DinkToPdf)
        // For our purposes we are going to use SynchronizedConverter
        IPdfConverter pdfConverter = new SynchronizedConverter(new PdfTools());// A function to convert html content to pdf based on the configuration pased as arguments
        // Arguments:
        // HtmlContent: the html content to be converted
        // Width: the width of the pdf to be created. e.g. "8.5in", "21.59cm" etc.
        // Height: the height of the pdf to be created. e.g. "11in", "27.94cm" etc.
        // Margins: the margis around the content
        // DPI: The dpi is very important when you want to print the pdf.
        // Returns a byte array of the pdf which can be stored as a file
        private byte[] BuildPdf(string HtmlContent, string Width, string Height, MarginSettings Margins, int? DPI = 180)
        {
            // Call the Convert method of SynchronizedConverter "pdfConverter"
            return pdfConverter.Convert(new HtmlToPdfDocument()
            {
                // Set the html content
                Objects =
                {
                    new ObjectSettings
                    {
                        HtmlContent = HtmlContent
                    }
                },
                // Set the configurations
                GlobalSettings = new GlobalSettings
                {
                    // PaperKind.A4 can also be used instead of width & height
                    PaperSize = new PechkinPaperSize(Width, Height),
                    DPI = DPI,
                    Margins = Margins
                }
            });
        }// The name of the function
        [FunctionName("Html2Pdf")]// The first arugment tells that the functions can be triggerd by a POST HTTP request. 
        // The second argument is mainly used for logging information, warnings or errors
        public async Task Run([HttpTrigger(AuthorizationLevel.Function, "POST")] Html2PdfRequest Request, ILogger Log)
        {
            // PDFByteArray is a byte array of pdf generated from the HtmlContent 
            var PDFByteArray = BuildPdf(Request.HtmlContent, "8.5in", "11in", new MarginSettings(0, 0, 0, 0));// The connection string of the Storage Account to which our PDF file will be uploaded
            var StorageConnectionString = "DefaultEndpointsProtocol=https;AccountName=<YOUR ACCOUNT NAME>;AccountKey=<YOUR ACCOUNT KEY>;EndpointSuffix=core.windows.net";

            // Generate an instance of CloudStorageAccount by parsing the connection string
            var StorageAccount = CloudStorageAccount.Parse(StorageConnectionString);// Create an instance of CloudBlobClient to connect to our storage account
            CloudBlobClient BlobClient = StorageAccount.CreateCloudBlobClient();// Get the instance of CloudBlobContainer which points to a container name "pdf"
            // Replace your own container name
            CloudBlobContainer BlobContainer = BlobClient.GetContainerReference("pdf");

            // Get the instance of the CloudBlockBlob to which the PDFByteArray will be uploaded
            CloudBlockBlob Blob = BlobContainer.GetBlockBlobReference(Request.PDFFileName);

            // Upload the pdf blob
            await Blob.UploadFromByteArrayAsync(PDFByteArray, 0, PDFByteArray.Length);
        }
    }
}
```

编码部分到此结束。

# wkhtmltopdf

我们仍然需要在我们的项目中添加 wkhtmltopdf 库。在选择特定的 Azure 应用计划时，有一些注意事项。根据计划，我们将必须获得 wkhtmltopdf 库。出于我们的目的，我们选择了 Linux 基本(B1)应用服务计划，因为 Windows 基本(B1)应用服务计划的成本是它的 5 倍。

在写这篇博客的时候，Azure App 服务计划正在使用带有 amd64 架构的 Debian 10。对我们来说很好，DinkToPdf 为 Linux、Windows 和 MacOS 提供了预编译库。下载。所以把它放在你的项目的根文件夹中。我在 MacOS 上工作，所以我也下载了 libwkhtmltox.dylib。如果您正在使用 Windows，或者您已经在 Windows 应用服务计划上托管了 Azure 功能，则必须下载 libwkhtmltox.dll。这是我们的项目结构的样子。

![](img/e531f0fc5728c25b7e63afbf574ba040.png)

当我们创建一个构建时，我们需要包含。所以图书馆。为此，请打开 csproj 文件，并将以下内容添加到 ItemGroup 中。

```
<None Update="./libwkhtmltox.so">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <CopyToPublishDirectory>Always</CopyToPublishDirectory>
</None>
```

以下是完整的 csproj 文件:

```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <AzureFunctionsVersion>v3</AzureFunctionsVersion>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="DinkToPdf" Version="1.0.8" />
    <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="3.0.11" />
  </ItemGroup>
  <ItemGroup>
    <None Update="host.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
    <None Update="local.settings.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
      <CopyToPublishDirectory>Never</CopyToPublishDirectory>
    </None>
    <None Update="./libwkhtmltox.so">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
      <CopyToPublishDirectory>Always</CopyToPublishDirectory>
    </None>
  </ItemGroup>
</Project>
```

# 创建 Azure Functions 应用

在我们部署 Azure 功能之前，我们必须在 Azure 门户中创建 Azure 功能。你可以去 Azure 门户网站开始创建 *Azure Functions* 资源。为了清晰起见，你可以看下面的截图。

![](img/42998e0be6bd6253724ea4f906476c2c.png)

在下面的截图中，请确保在此处至少选择或创建*基本*计划。其次，在操作系统中选择 *Linux* 。

![](img/e596ca90926aac7224c5e8cffa0b3620.png)

拥有*应用洞察*很好，因为您将能够查看日志和监控功能。此外，它几乎不花什么钱。如以下截图所示，如果您想启用它，请选择*是*。

![](img/92d07db9d5e8c7636369241056229d68.png)

选择 Next: Tags 并再次单击 Next，然后单击 *Create* 创建您的资源。创建 *Azure Functions* 资源可能需要几分钟时间。

# 部署到 Azure 功能

一旦创建完成，我们将通过 VS 代码将代码直接部署到 Azure 功能中。为此，你必须进入扩展并安装 Azure Functions 扩展。在它的帮助下，我们将能够登录和管理 Azure 功能。

![](img/5e743f4fc4d835e2474ee3031738bcb3.png)

安装完成后，你会在边栏上看到 Azure 图标。当点击时，它将打开一个面板，其中有一个选项*登录 Azure* 。

![](img/b08effb2263ee1f079b5b199cf981e08.png)

选择*登录 Azure* ，这将打开一个浏览器，您可以在其中使用您的帐户登录。一旦登录，你可以回到 VS 代码，在你的侧面板中看到 Azure 函数列表。

![](img/aa609b451770ea665ad8cef7bfb26775.png)

对我来说，有 4 个功能应用程序。因为您可能只创建了一个，所以它将显示一个。是时候部署应用程序了。

按下 *F1* 打开带有操作列表的菜单。选择*Azure Functions:Deploy to Function App…*这将打开一个 Azure Functions 列表，您可以部署到其中。选择我们新创建的 Azure Funtions 应用程序。这将要求弹出一个确认窗口，所以请继续部署它。部署你的应用需要几分钟时间。

# 配置 wkhtmltopdf

一旦你部署了 Azure 功能，还有最后一件事要做。我们需要将 *libwkhtmltox.so* 添加到 Azure Functions 应用程序的适当位置。登录 Azure 门户并导航到我们的 Azure Functions 应用程序。在侧面板上搜索 SSH 并点击 *Go* 按钮。

![](img/aa01b7631ae032deb17d9ba6d27f088c.png)

这将在新选项卡中打开一个 SSH 控制台。我们的站点位于/home/site/wwwroot。因此，通过键入以下命令导航到该文件夹。

```
cd /home/site/wwwroot/bin
```

当您执行 *ls* 命令查看文件内容时，您不会看到 *libwkhtmltox.so* 文件。它实际上位于/home/site/wwwroot。

这不是正确的立场。我们需要将它复制到 bin 文件夹中。为此，请执行以下命令。

```
cp ../libwkhtmltox.so libwkhtmltox.so
```

如果你知道如何将文件放入 bin 文件夹的更好方法，请在下面的评论中提出建议。

就是这样！！！你已经有了一个全功能的 Azure Functions 应用。是时候从我们的演示网站项目中调用它了。

# 调用 Azure 函数

尽管如此，我们仍然需要测试和调用我们的函数。在此之前，我们需要获得调用函数所需的*代码*。代码*是一个秘密，需要包含它来安全地调用函数。要获取*代码*，请导航至 Azure Portal 并打开您的功能应用。在侧面板中搜索*功能。**

![](img/25165071f7f46a4ce8b966586f8c5030.png)

您将在列表中看到 *Html2Pdf* 。单击该功能将打开详细视图。在侧面板中有一个*功能键*的选项。选择该选项查看已经为您添加的隐藏默认*代码*。

![](img/bf202e41fb5240a562906507dbb9d421.png)

复制代码并保存在手边，因为代码中会用到它。为了测试该功能，我为您创建了一个示例控制台应用程序。替换基本 url，代码如下:

```
using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Newtonsoft.Json;namespace Demo.ConsoleApp
{
    public class Program
    {
        public static async Task Main(string[] args)
        {
            string AzureFunctionsUrl = "https://<Your Base Url>/api/Html2Pdf?code=<Replace with your Code>";using (HttpClient client = new HttpClient())
            {
                var Request = new Html2PdfRequest
                {
                    HtmlContent = "<h1>Hello World</h1>",
                    PDFFileName = "hello-world.pdf"
                };
                string json = JsonConvert.SerializeObject(Request);
                var buffer = System.Text.Encoding.UTF8.GetBytes(json);
                var byteContent = new ByteArrayContent(buffer);byteContent.Headers.ContentType = new MediaTypeHeaderValue("application/json");using (HttpResponseMessage res = await client.PostAsync(AzureFunctionsUrl, byteContent))
                {
                    if (res.StatusCode != HttpStatusCode.NoContent)
                    {
                        throw new Exception("There was an error uploading the pdf");
                    }
                }
            }
        }
    }public class Html2PdfRequest
    {
        // The HTML content that needs to be converted.
        public string HtmlContent { get; set; }// The name of the PDF file to be generated
        public string PDFFileName { get; set; }
    }}
```

同样，代码应该是不言自明的。如果您有任何反馈或问题，请在下面的评论区提问。一旦你运行上面的控制台应用程序，它将在 Azure Storage 的 *pdf* 容器中创建一个*hello-world.pdf*文件。

# 结论

关于如何使用 Azure 函数将 HTML 转换成 PDF 的教程到此结束。对于任何反馈、问题或博客主题，您可以在下面留下评论。订阅时事通讯，了解即将推出的激动人心的新教程。

也可以关注我的[中](https://arjav-dave.medium.com/)、[dev . to](https://dev.to/arjavdave)&[hash node](https://blog.royalecheese.com/)。

访问[我的博客](https://arjavdave.com/)获取更多这样的教程。