### dotnet_selenium_lessonseven

## Refactoring for Easy Maintenance

### The problem with duplicate browser automation code
Removing duplication by moving driver operations from the test script to a Page Object Model of a page.

### An Overview of the Page Object Model Pattern for Browser Tests

* Page Object Model provides test code with a logical view of the user interface while abstracting away the low level details of the user interface implementation. 
* Reduce test code duplication
* insulate tests from UI changes
---  
** Improve Readability of Tests
** Return and accept fundamental types

### Which HTML Elements Should You Add to a Page Object Model?
### Creating an Initial POM
* Add a PageObjectModels folder
* Add ```HomePage.cs``` file 
```C#
using OpenQA.Selenium;
using System.Collections.ObjectModel;

namespace CreditCards.UITests.PageObjectModels
{
    class HomePage
    {
        private readonly IWebDriver Driver;

        public HomePage(IWebDriver driver)
        {
            Driver = driver;
        }

        public ReadOnlyCollection<IWebElement> ProductCells
        {
            get
            {
                return Driver.FindElements(By.TagName("td"));
            }
        }
    }
}

```
* update ```CreditCardWebAppShould.cs``` file
* using Home object in the script 
```C#
 var homePage = new HomePage(driver);
```


```C#
  [Fact]
        public void DisplayProductsAndRates()
        {
            using (IWebDriver driver = new ChromeDriver())
            {
                driver.Navigate().GoToUrl(HomeUrl);
                var homePage = new HomePage(driver);

                DemoHelper.Pause();

                Assert.Equal("Easy Credit Card", homePage.Products[0].name);
                Assert.Equal("20% APR", homePage.Products[0].interestRate);

                Assert.Equal("Silver Credit Card", homePage.Products[1].name);
                Assert.Equal("18% APR", homePage.Products[1].interestRate);

                Assert.Equal("Gold Credit Card", homePage.Products[2].name);
                Assert.Equal("17% APR", homePage.Products[2].interestRate);
            }
        }

```

### Returning Fundamental Types from POMs
```C#
public ReadOnlyCollection<(string name, string interestRate)> Products
        {
            get
            {
                var products = new List<(string name, string interestRate)>();

                var productCells = Driver.FindElements(By.TagName("td"));

                for (int i = 0; i < productCells.Count - 1; i += 2)
                {
                    string name = productCells[i].Text;
                    string interestRate = productCells[i + 1].Text;
                    products.Add((name, interestRate));
                }

                return products.AsReadOnly();
            }
        }```

### Navigating to a POM page
* Update page object model Home class with:
```C#
 public void NavigateTo()
        {
            Driver.Navigate().GoToUrl(PageUrl);
        }
```
* Use NavigateTo in the test script
```C#
[Fact]
        public void DisplayProductsAndRates()
        {
            using (IWebDriver driver = new ChromeDriver())
            {                
                var homePage = new HomePage(driver);
                homePage.NavigateTo();

                DemoHelper.Pause();

                Assert.Equal("Easy Credit Card", homePage.Products[0].name);
                Assert.Equal("20% APR", homePage.Products[0].interestRate);

                Assert.Equal("Silver Credit Card", homePage.Products[1].name);
                Assert.Equal("18% APR", homePage.Products[1].interestRate);

                Assert.Equal("Gold Credit Card", homePage.Products[2].name);
                Assert.Equal("17% APR", homePage.Products[2].interestRate);
            }
        }
```
### Checking a Page is Loaded in a POM
* Add following to the ```homepage.cs``` file in the PageObjectModels folder
```C#
 private void EnsurePageLoaded()
        {
            bool pageHasLoaded = (Driver.Url == PageUrl) && (Driver.Title == PageTitle);

            if (!pageHasLoaded)
            {
                throw new Exception($"Failed to load page. Page URL = '{Driver.Url}' Page Source: \r\n {Driver.PageSource}");
            }
        }
```


### Refactoring the LoadHomePage and ReloadHOmePageOnBack Tests
* Inside ```CreditCardWebAppShould.cs``` file, modify the LoadHomePage test method to use EnsurePageLoaded()
* Change ReloadHomePageOnBack test method 
```C#
   [Fact]
        [Trait("Category", "Smoke")]
        public void LoadHomePage()
        {
            using (IWebDriver driver = new ChromeDriver())
            {
                var homePage = new HomePage(driver);
                homePage.NavigateTo();
            }
        }

     [Fact]
        [Trait("Category", "Smoke")]
        public void ReloadHomePageOnBack()
        {
            using (IWebDriver driver = new ChromeDriver())
            {
                var homePage = new HomePage(driver);
                homePage.NavigateTo();

                string initialToken = homePage.GenerationToken;
               
                driver.Navigate().GoToUrl(AboutUrl);               
                driver.Navigate().Back();

                homePage.EnsurePageLoaded();

                string reloadedToken = homePage.GenerationToken;

                Assert.NotEqual(initialToken, reloadedToken);
            }
        }
```

### Refactoring Continued

### Refactoring the Remaining CredtitCardWebAppShould Tests

### Navigating to other POMs

### Encapsulating Explicit Waits in POMs

### Form Filling and Submission with POMs

### Adding and ApplicationCompletePage POMs

### Testing Form Validation with Page Object Models

### Reusing WebDriver Instances to Speed up Test Execution

### Tackling Brittle Tests with JavaScript

### Completing the Refactoring to Page Object Models

### Page Object Models Using Selenium Support

#### Page Object Model Considerations