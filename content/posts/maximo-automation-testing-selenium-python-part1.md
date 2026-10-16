---
title: "Part 1 — Maximo Automation Testing through Selenium with Python — Dev Environment Setup and SSO"
date: 2024-09-08
tags: [maximo, testing, selenium, python]
draft: false
---


### **1. Background**

Comparing to manual testing, automation testing definitely has a lot of advantages: More precise testing, decrease testing time dramatically, integrate with DevOps process, etc. But the major challenge is to implement automation testing, especially for the complicate scenario. For example, some test cases required combine the functional and integration testing together. In my current Maximo project, there are a lot of test cases complicate like this. Each release requires a lot of effort to retest some of the test cases manually, also need to do the full regression testing manually to make sure the updates in the new release not breaking existing functions / interfaces. I started to look into automation testing solution for Maximo.

If your organization already upgraded / implemented to Maximo Application Suite (MAS), and your automation testing requirement only focus on functions, maybe you can try IBM Maximo Test Automation Framework, which you can find detail information from here:

On these serial blogs, I will only focus on the Maximo automation testing through Selenium with Python, not leveraging the IBM Maximo Test Automation Framework. There are couple reasons for it, but the major one is my project is not on the MAS boat, and my testing requirement is more complicate than functional testing in Maximo. Selenium is a powerful tool for controlling web browsers through programs and performing browser automation. It is functional for all browsers, works on all major OS and its scripts are written in various languages i.e. Python, Java, C#, etc. Selenium with Python opens up many possibilities for efficient and effective web application testing. Through Selenium with Python, we can implement the automation testing to meet the complicate testing case scenario requirement.

### **2. Development Environment Setup**

The following is the technical stack we will leverage in this experiment.

(1) Coding Editor — VS Code

(2) Coding language — Python

(3) Unit Testing Package — Python unittest

(4) Automation Testing Framework / Tool — Selenium

(5) Browser — Chrome

(6) OS — Windows 10 /11

If you are not familiar with above technologies, strongly recommend you go through the following topics to have basic concept of them before we move to next step.

**Python unittest — **Python unit testing framework was originally inspired by JUnit and has a similar flavor as major unit testing frameworks in other languages. It supports test automation, sharing of setup and shutdown code for tests, aggregation of tests into collections, and independence of the tests from the reporting framework.

[https://docs.python.org/3/library/unittest.html#](https://docs.python.org/3/library/unittest.html#)

**Selenium WebDriver **— Selenium supports automation of all the major browsers in the market through the use of *WebDriver*. WebDriver is an API and protocol that defines a language-neutral interface for controlling the behaviour of web browsers. Each browser is backed by a specific WebDriver implementation, called a *driver*. The driver is the component responsible for delegating down to the browser, and handles communication to and from Selenium and the browser.

[https://www.selenium.dev/documentation/webdriver/getting_started/](https://www.selenium.dev/documentation/webdriver/getting_started/)

Now before we can start our automation testing coding, we need to setup our development environment. The following are the high level setup steps, you can refer to the links on each step from official website for the tool’s setup and configuration.

**Step 1**: Install VS Code

VS Code is the most popular light weight coding editor, which has tons of the extension tools can be used. You can download it from official website and install it.

[https://code.visualstudio.com/download](https://code.visualstudio.com/download)

Please select the right version which match to your OS.

**Step 2:** Install Python and Pythons extension in VS Code.

Please follow the steps on [this page](https://code.visualstudio.com/docs/python/python-tutorial) to install Python and VS Code Python Extension

**Step 3:** Install Selenium

Open VS Code, in the terminal window (if you don’t have one, you can open a new one through menu Terminal -> New Terminal to open one. Type the following command in the terminal to install Selenium package.

```bash
pip install -U selenium
```

After install selenium successfully through pip, you can type the below command in the terminal to check the installed selenium version.

```bash
pip show selenium
```

You should be able to see similar information as following:

![](https://cdn-images-1.medium.com/max/800/1*WZajijuawxpLkmdqTxvgqw.png)

**Step 4: **Install ChromeDriver and configure the path

WebDriver is an open source tool for automated testing of webapps across many browsers. It provides capabilities for navigating to web pages, user input, JavaScript execution, and more. ChromeDriver is a standalone server that implements the [W3C WebDriver standard](https://w3c.github.io/webdriver/webdriver-spec.html). ChromeDriver is available for Chrome on Android and Chrome on Desktop (Mac, Linux, Windows and ChromeOS). In our experiment, we will use ChromeDriver as the webdriver for our testing. You can download the latest ChromeDriver stable version from the below link:

[https://googlechromelabs.github.io/chrome-for-testing/#stable](https://googlechromelabs.github.io/chrome-for-testing/#stable)

After you download the ChromeDriver package, unzip to a folder, use the below command add ChromeDriver to the path, then you don’t need to pass ChromeDriver folder information in the code every time.

```
setx /M path "%path%; your chromedrive folder path"
```

*Note: If you don’t have administrator permission, you can remove /M parameter from the above command to add the path to current user only.*

### **3. Requirement**

After complete development environment setup, let’s take a look what test case we gotten implement automation test through above technical stack. Assume Maximo was implemented authentication through Single Sign On (SSO) solution, we would like to use Selenium with Python to implement a user login. This scenario is a pretty common start point for most of the automation testing, simulate a particular role user (for example Asset Manager) login the system to perform some functions test case later.

### **4. Implementation**

To be able to implement above requirement, we need to use chrome browser incognito mode which can separate simulated login user session will not be impacted by current windows login user session.

In the VS Code, open a blank folder for your project, create a new python file, type the following code in the new python file.

```python
import unittest
from selenium import webdriver
from selenium.webdriver import Keys,ChromeOptions
from selenium.webdriver.common.by import By
import time

class LoginTestCase(unittest.TestCase):
    # setUp function will be run before any test case to be executed
    def setUp(self):
        #Open incognito mode for simulated user login
        chrome_options = ChromeOptions()
        chrome_options.add_argument("--incognito")
        self.browser = webdriver.Chrome(options=chrome_options)
        self.addCleanup(self.browser.quit)

    # Test case for single sign on login
    def test_SSO_Login(self):
        #Give 5 second to allow browser to redirect to single sign on login page
        self.browser.implicitly_wait(5)
        #Need to update to your maximo instance url
        self.browser.get('https//yourmaximo.com')
        #Find login email textbox on the SSO login page and auto fill in the login email
        userelement=self.browser.find_element(By.ID,"i0116")
        #Need to update to your user email account to login
        userelement.send_keys("your@email.com")
        userelement.send_keys(Keys.ENTER)
        #Allow 5 second to load password page        
        time.sleep(5)
        #Find password textbox on the SSO login page and auto fill in the login email
        pwdelement=self.browser.find_element(By.ID,"i0118")
        #Need to update to your user password
        pwdelement.send_keys("yourpassword")
        pwdelement.send_keys(Keys.ENTER)
        time.sleep(5)
        #Sometime after login there is popup window ask user whether Stay signed in. Find yes button and simulate click it.
        yesbutton=self.browser.find_element(By.ID,"idSIButton9")
        if yesbutton is not None:
            yesbutton.send_keys(Keys.ENTER)        
        #Allow 5 second to redirect to Maximo Start Center after SSO login successfully
        time.sleep(5)
        #Check if successfuly login Maximo or not through assert current page title is start center or not.
        self.assertIn('Start Center', self.browser.title)
       
if __name__ == '__main__':
    unittest.main(verbosity=2)
```

The comments in above code already have pretty good explanation. The only tip I would like to provide here is how to get html element ID which can be provided to the function *find_element(). *When you open the page you are trying to find element ID in chrome, click 3 dot icon at the right top, select the menu More tools ->Developer Tools.

![](https://cdn-images-1.medium.com/max/800/1*eBrgndnb2q4P7glNg8zoeA.png)

On the developers tool screen, click left top arrow button.

![](https://cdn-images-1.medium.com/max/800/1*8ht8Eht1yG6JhJhwLcNr1Q.png)

It will allow you when you click the web page element, it will automatically locate the html element on the development tools source code tab, which you can identify the element ID if it has.

![](https://cdn-images-1.medium.com/max/800/1*k9QcEQdDLPhYkRZ-U8BPiw.png)

After you run the python code in VS Code, if it is successful pass the test case, you should see the OK result in the output as following:

![](https://cdn-images-1.medium.com/max/800/1*7yv9-tWivWAZhaSPjSTF_g.png)

### **5. Summary**

Let’s recap what I introduce in this blog: We setup development environment for automation testing development for Selenium with Python in VS Code. We develop python script to implement automation test case for Maximo login through Single Sign On (SSO). In next blog, we will explore the implementation of automation testing for more complicate test case to combine business function with integration.

---
*Originally published on [Medium](https://medium.com/@jackyqiubao/part-1-maximo-automation-testing-through-selenium-with-python-dev-environment-setup-and-sso-6c048fbd25cb).*
