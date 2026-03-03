Selenium-Maven-Template
=======================

[![Join the chat at https://gitter.im/Ardesco/Selenium-Maven-Template](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/Ardesco/Selenium-Maven-Template?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

### Purpose & Summary

This repository is a **ready-to-use Maven project template for Java-based Selenium WebDriver test automation**. Its goal is to eliminate boilerplate setup so that teams can clone it and start writing browser tests immediately.

**What it provides out of the box:**

- **Selenium 4 + TestNG + AssertJ** – pre-wired dependencies for writing and asserting browser-level tests in Java.
- **Automatic WebDriver binary management** – the `driver-binary-downloader-maven-plugin` downloads the correct browser driver (ChromeDriver, GeckoDriver, EdgeDriver, etc.) for your OS automatically; no manual setup needed.
- **Multi-browser support** – switch between Firefox, Chrome, Edge, IE, Opera, and Brave via a single command-line flag (`-Dbrowser=chrome`).
- **Headless mode** – run tests without a visible browser window (`-Dheadless=true`, enabled by default).
- **Parallel test execution** – configure the number of concurrent test threads with `-Dthreads=N`.
- **Selenium Grid / cloud support** – point tests at a remote Grid or SauceLabs with `-Dremote=true -DseleniumGridURL=...`.
- **Page Object Model pattern** – example page objects (`GoogleHomePage`, `GoogleSearchPage`) demonstrate how to structure maintainable test code.
- **Screenshot on failure** – a TestNG listener automatically captures a screenshot whenever a test fails and saves it to `target/screenshots`.
- **Proxy support** – configure an HTTP proxy for tests via Maven properties.

**Is it helpful for a Java test automation project? Yes.**  
If your project needs Selenium-based UI tests written in Java, this template gives you a production-ready starting structure — driver management, logging (Log4j 2), parallel execution, and reporting scaffolding — so you can focus on writing test logic rather than wiring infrastructure together.

---

### Example: Writing Your First Test

The steps below show exactly how to add a brand-new automated test to the project. The full working version already lives in [`src/test/java/com/lazerycode/selenium/tests/GoogleExampleIT.java`](src/test/java/com/lazerycode/selenium/tests/GoogleExampleIT.java).

#### Step 1 – Create a Page Object

A Page Object models one page (or component) of your web application. It hides raw Selenium calls behind readable methods, keeping your tests clean.

Create `src/test/java/com/lazerycode/selenium/page_objects/GoogleHomePage.java`:

```java
package com.lazerycode.selenium.page_objects;

import com.lazerycode.selenium.DriverBase;
import com.lazerycode.selenium.util.Query;
import org.openqa.selenium.By;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

import static com.lazerycode.selenium.util.AssignDriver.initQueryObjects;

public class GoogleHomePage {

    // Locate the search bar by the HTML name attribute
    private final Query searchBar = new Query().defaultLocator(By.name("q"));
    // Locate the Search button by its name attribute
    private final Query googleSearch = new Query().defaultLocator(By.name("btnK"));

    private final WebDriverWait wait;

    public GoogleHomePage() throws Exception {
        // Wire all Query fields to the current driver thread
        initQueryObjects(this, DriverBase.getDriver());
        wait = new WebDriverWait(DriverBase.getDriver(), Duration.ofSeconds(15));
    }

    /** Types the search term into the Google search bar and returns this page. */
    public GoogleHomePage enterSearchTerm(String searchTerm) {
        wait.until(ExpectedConditions.presenceOfElementLocated(searchBar.by()));
        searchBar.findWebElement().clear();
        searchBar.findWebElement().sendKeys(searchTerm);
        return this;
    }

    /** Dismisses the cookie consent banner if present. */
    public GoogleHomePage acceptCookies() {
        Query acceptCookiesPopup = new Query().defaultLocator(By.xpath("//*[.='I agree']"));
        new WebDriverWait(DriverBase.getDriver(), Duration.ofSeconds(5))
                .until(ExpectedConditions.presenceOfElementLocated(acceptCookiesPopup.by()));
        acceptCookiesPopup.findWebElement().click();
        return this;
    }

    /** Submits the search and returns the results page object. */
    public GoogleSearchPage submitSearch() throws Exception {
        googleSearch.findWebElement().submit();
        return new GoogleSearchPage();
    }
}
```

#### Step 2 – Create the Test Class

Test classes must end in `IT` (Integration Test) so that the Maven Failsafe plugin picks them up. Extend `DriverBase` to get a managed, thread-safe `WebDriver` instance.

Create `src/test/java/com/lazerycode/selenium/tests/GoogleExampleIT.java`:

```java
package com.lazerycode.selenium.tests;

import com.lazerycode.selenium.DriverBase;
import com.lazerycode.selenium.page_objects.GoogleHomePage;
import com.lazerycode.selenium.page_objects.GoogleSearchPage;
import org.openqa.selenium.WebDriver;
import org.testng.annotations.Test;

import static org.assertj.core.api.AssertionsForClassTypes.assertThat;

public class GoogleExampleIT extends DriverBase {

    @Test
    public void googleCheeseExample() throws Exception {
        // 1. Get the thread-safe WebDriver managed by DriverBase
        WebDriver driver = getDriver();

        // 2. Navigate to the page under test
        driver.get("http://www.google.com");

        // 3. Interact with the page via the Page Object – no raw Selenium in the test
        GoogleHomePage googleHomePage = new GoogleHomePage();
        googleHomePage.acceptCookies();                                    // dismiss cookie banner
        GoogleSearchPage results = googleHomePage
                                        .enterSearchTerm("Cheese")        // type into search bar
                                        .submitSearch();                  // click Search

        // 4. Wait for the result page to load, then assert the page title
        results.waitForPageTitleToStartWith("Cheese");
        assertThat(results.getPageTitle()).isEqualTo("Cheese - Google Search");
    }
}
```

#### Step 3 – Run the Test

```shell
# Run all integration tests in headless Firefox (default)
mvn clean verify

# Run in headless Chrome instead
mvn clean verify -Dbrowser=chrome

# Run with a visible browser window (useful while writing tests)
mvn clean verify -Dbrowser=chrome -Dheadless=false

# Run with 4 parallel threads to speed up a large suite
mvn clean verify -Dthreads=4
```

When the test fails, a screenshot is automatically saved to `target/screenshots/` — no extra setup needed.

---

A maven template for Selenium 4 that has the latest dependencies so that you can just check out and start writing tests in four easy steps. If you like what you see have a look at
my Selenium book [Mastering Selenium Webdriver](https://www.amazon.co.uk/Mastering-Selenium-WebDriver-Mark-Collin/dp/1784394351).

1. Open a terminal window/command prompt
2. Clone this project.
3. `cd Selenium-Maven-Template` (Or whatever folder you cloned it into)
4. `mvn clean verify`

All dependencies should now be downloaded and the example google cheese test will have run successfully in headless mode (Assuming you have Firefox installed in the default
location)

### What should I know?

- To run any unit tests that test your Selenium framework you just need to ensure that all unit test file names end, or start with "test" and they will be run as part of the build.
- The maven failsafe plugin has been used to create a profile with the id "selenium-tests". This is active by default, but if you want to perform a build without running your
  selenium tests you can disable it using:
```shell
mvn clean verify -P-selenium-tests
```

- The maven-failsafe-plugin will pick up any files that end in IT by default. You can customise this is you would prefer to use a custom identifier for your Selenium tests.

### Known problems...

- It looks like SafariDriver is no longer playing nicely and we are waiting on Apple to fix it... Running safari driver locally in server mode and connecting to it like a grid
  seems to be the workaround.

### Anything else?

Yes you can specify which browser to use by using one of the following on the command line:

- `-Dbrowser=firefox`
- `-Dbrowser=chrome`
- `-Dbrowser=ie`
- `-Dbrowser=edge`
- `-Dbrowser=opera`
- `-Dbrowser=brave`

If you want to toggle the use of chrome or firefox in headless mode set the headless flag (by default the headless flag is set to true)

- `-Dheadless=true`
- `-Dheadless=false`

You don't need to worry about downloading the IEDriverServer, EdgeDriver, ChromeDriver , OperaChromiumDriver, or GeckoDriver binaries, this project will do that for you
automatically.

You can specify a grid to connect to where you can choose your browser, browser version and platform:

- `-Dremote=true`
- `-DseleniumGridURL=http://{username}:{accessKey}@ondemand.saucelabs.com:80/wd/hub`
- `-Dplatform=xp`
- `-Dbrowser=firefox`
- `-DbrowserVersion=44`

You can even specify multiple threads (you can do it on a grid as well!):

- `-Dthreads=2`

You can also specify a proxy to use

- `-DproxyEnabled=true`
- `-DproxyHost=localhost`
- `-DproxyPort=8080`
- `-DproxyUsername=fred`
- `-DproxyPassword=Password123`

If the tests fail screenshots will be saved in `${project.basedir}/target/screenshots`

If you need to force a binary overwrite you can do:

- `-Doverwrite.binaries=true`

### It's not working!!!

You have probably got outdated driver binaries, by default they are not overwritten if they already exist to speed things up. You have two options:

- `mvn clean verify -Doverwrite.binaries=true`
- Delete the `selenium_standalone_binaries` folder in your resources directory

### Brave fails when in headless mode

Currently, Brave seems to be a bit flaky when running in headless mode, I would suggest running with `-Dheadless=false`

### It's looking for the Brave binary in the wrong location

You probably don't have the brave binary installed in one of the default locations that this codebase is expecting.  That's OK though, you can specify it by seeting the following system property:

- `-DbraveBinaryLocation=/path/to/brave-browser`
