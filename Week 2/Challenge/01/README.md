# Making a simple web app with C# and ASP.net

Your first task for this week will be to create a simple web app with C# and ASP.NET

- Razor Views ? Razor View Runtime Compilation ?
 - Razor Pages is a newer, simplified web application programming model. It removes much of the ceremony of ASP.NET MVC by adopting a file-based routing approach. Each Razor Pages file found under the Pages directory equates to an endpoint. Razor Pages have an associated C# objected called the page model, which holds each page's behavior. Additionally, each page works on the limited semantics of HTML, only supporting GET and POST methods.
 - https://www.jetbrains.com/dotnet/guide/tutorials/basics/razor-pages/
- Startup ? Program ? what are these default files in the ASP project ?
  - https://www.c-sharpcorner.com/article/what-is-startup-class-and-program-cs-in-asp-net-core/

The web app should just serve a simple hello world page ! that's it.

- Create unit tests project
- Create controller tests - render the ViewResults from the controller methods and test not null. check the ViewBag message matches what we expect.
- Talk about type casting - we can solve the actionresult/viewresult problem either by casting or by changing the controller class to use ViewResult - OO tip - always use the most specific type possible. viewresult implements actionresult, but since we know the specific type it will be returning, we can tighten the scope and make our tests not have to use casting.

## Introduction 

Your first task is going to be creating a simple web app using ASP.NET. .NET is a variety of libraries packaged together that microsoft develops and maintains in order to make development quick in the microsoft family of languages (C#, F#, etc)

ASP.NET is an extension of that library package, which contains common tools and libraries for building web apps. 

We won't be too concerned with building the app for now, we'll cover that in more detail in week 4 and week 5. For now, we're just going to cover the basics of getting an app set up in Visual Studio.

## Task 

We'll start out by making a new ASP.net app in Visual Studio. Open up the app and create a new project. Then choose "Web Application (Model View Controller)". Leave the default settings other than changing the .NET core version to 5.0.

You can call the solution and project name whatever you'd like. I chose `HelloInternet` and `App`. Add another project to the solution using the MSTests template for your unit tests. 

It can sometimes be helpful once your project grows large to have the directory structure of your unit tests match the structure of the app. As you can see, the MVC template already has quite a bit of boilerplate code added. Add a folder to the unit tests project called `controllers` and make a test file inside called `HomeController.Tests.cs`. Don't forget to add your app as a dependency to the test project.

Add a couple of tests to ensure your controllers are working okay. We won't worry about adding other tests. For now, we're just concerned about adding a few tests we can automate later. Its sufficient just to write tests to ensure that the views have rendered correctly, since that's all our controller methods do. Something like:

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using App.Controllers;
using Moq;
using Microsoft.Extensions.Logging;
using Microsoft.AspNetCore.Mvc;

namespace AppTests
{
    [TestClass]
    public class HomeControllerTests
    {
        [TestMethod]
        public void Can_Render_Index()
        {
            var mock = new Mock<ILogger<HomeController>>();

            HomeController controller = new(mock.Object);

            ViewResult result = controller.Index();

            Assert.IsNotNull(result);
        }
    }
}
```

Note that we need to mock a logger class because it's a requirement of the controller class. Since we'll likely need to re use it to test the Privacy and Error pages as well, we can utilise some new annotations: `TestInitialize` and `ClassInitialize`. The former ensures some operation is run before each test, and the latter runs a function before all tests are run. Since we don't need our logged to be refreshed between each test, we can use `ClassInitialize` here:

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using App.Controllers;
using Moq;
using Microsoft.Extensions.Logging;
using Microsoft.AspNetCore.Mvc;

namespace AppTests
{
    [TestClass]
    public class HomeControllerTests
    {
        private Mock<ILogger<HomeController>> _mockLogger;

        [ClassInitialize]
        public void On_Class_Initialize()
        {
            _mockLogger = new Mock<ILogger<HomeController>>();
        }

        [TestMethod]
        public void Can_Render_Index()
        {
            var mock = new Mock<ILogger<HomeController>>();

            HomeController controller = new(_mockLogger.Object);

            ViewResult result = controller.Index();

            Assert.IsNotNull(result);
        }
    }
}
```

You might also find that the ViewResult line causes a compiler error because the Index method returns an `ActionResult`, not a `ViewResult`

An ActionResult is just the base class that other types of results a controller can return inherit from. It could be a ViewResult, RedirectResult, etc depending on the type of response we're hoping to respond with. If there was a chance, based on some condition that a controller action might return *multiple* types of responses, then it would be wise to leave the return type on our controller actions to `ActionResult`. Otherwise, it's best practice with OO programming to choose the most specific type possible as the return type of any method. In this case, our index method always returns a `new View()`, so we can change the return type of our Index action to `ViewResult` (and the other two, Privacy and Error pages).

Another technique we can use in other scenarios where it's not possible to restrict scope of the function is to **cast** the function caller. For example, in the example above ViewResult inherits from ActionResult, and if we know that controller.Index() is going to return a ViewResult despite the return type being ActionResult, we can cast the result instead and leave the return type as ActionResult:

```csharp
 [TestMethod]
  public void Can_Render_Index()
  {
      var mock = new Mock<ILogger<HomeController>>();

      HomeController controller = new(_mockLogger.Object);

      // We add the casting before controller.Index();
      ViewResult result = (ViewResult)controller.Index();

      Assert.IsNotNull(result);
  }
```

Go ahead and try running the application. You'll likely receive an error that says you need a valid certificate to run https. Hit 'run' and follow the dialogues to install the microsoft local https certificate on your keychain. You should now be able to see the basic boilerplate web app in your browser.

What we're doing here is creating a [Self Signed Certificate](https://en.wikipedia.org/wiki/Self-signed_certificate). These are useful in development for working https, but not suitable for production. We'll talk about Certificate Authorities at a later excercise, but for now suffice to say our self signed certifcate is not signed by a certifcate authority, and users visiting our website would receive a "This website's connection is insecure" warning at least in chrome.

Once you've finished writing all of your tests for the HomeController, let's move on to the next excercise.








