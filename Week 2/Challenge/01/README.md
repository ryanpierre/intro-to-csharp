# Making a simple web app with C# and ASP.net

## Introduction 

Your first task is going to be creating a simple web app using ASP.NET. .NET is a variety of libraries packaged together that microsoft develops and maintains in order to make development quick in the microsoft family of languages (C#, F#, etc)

ASP.NET is an extension of that library package, which contains common tools and libraries for building web apps. 

We won't be too concerned with building the app for now, we'll cover that in more detail in week 4 and week 5. For now, we're just going to cover the basics of getting an app set up in Visual Studio.

## Learning objectives
- Create an MVC application in ASP.NET
- Write unit tests for controllers in ASP.NET
- Explain ActionResults
- Explain Type Casting

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

Another way to write this is 

```csharp
  ViewResult result = controller.Index() as ViewResult;
```

We can also make the test a bit more specific. If we have a method called `Index` in our controller, then it will automatically render a view called Index if it can be found. However, we can also change the controller code to read:

```csharp
  return View("Index");
```

and then change our test to the following:

```csharp
 [TestMethod]
  public void Can_Render_Index()
  {
      var mock = new Mock<ILogger<HomeController>>();

      HomeController controller = new(_mockLogger.Object);

      // We add the casting before controller.Index();
      ViewResult result = (ViewResult)controller.Index();

      Assert.AreEqual("Index", result.ViewName);
  }
```

In order to have a more specific test (i.e. that the privacy page was rendered)

Go ahead and try running the application. You'll likely receive an error that says you need a valid certificate to run https. Hit 'run' and follow the dialogues to install the microsoft local https certificate on your keychain. You should now be able to see the basic boilerplate web app in your browser.

What we're doing here is creating a [Self Signed Certificate](https://en.wikipedia.org/wiki/Self-signed_certificate). These are useful in development for working https, but not suitable for production. We'll talk about Certificate Authorities at a later excercise, but for now suffice to say our self signed certifcate is not signed by a certifcate authority, and users visiting our website would receive a "This website's connection is insecure" warning at least in chrome.

Once you've finished writing all of your tests for the HomeController methods, move on to the next excercise.

Useful reading:
- [Creating unit tests in C# ASP.NET](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/unit-testing/creating-unit-tests-for-asp-net-mvc-applications-cs)
- [ASP.NET Action Results](https://rachelappel.com/2013/04/02/asp-net-mvc-actionresults-explained/)
- [The as keyword in C#](https://stackoverflow.com/questions/7566212/when-should-you-use-the-as-keyword-in-c-sharp)
- 








