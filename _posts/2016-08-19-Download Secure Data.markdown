---
layout: post
title:  "Download Secure Data"
date:   2016-08-19 11:08:00
categories: Python
---

In the previous post, I have introduce how to auto login to the HSBC website using python. But my final purpose is to download the eStatements of my credit card and saving accounts monthly. So in the post I'll introduce the steps to auto download the eStatements.

The first step is to find out the route to the page where the eStatements could be viewed.

It can be easily found that eStatements view page could be accessed under "Card" tab --> eStatements and service link. Therefore, after login to the online banking account. To implement the mentioned steps, it could be done by the following codes.

```python
    # route to Card tab page
   driver.get("https://www1.personal.ebanking.hsbc.com.hk/1/3/cards?__cmd-All_MenuRefresh=")

   # click e-Statement hyperlink
   link = driver.find_element_by_xpath("//a[@href='https://www1.personal.ebanking.hsbc.com.hk/1/3/my-hsbc/estatement?cmd-ECorrespDummyPortlet_in=&designateType=Card']")
   link.click()
```

Next, we have to find out the source download address of these pdf files so that we can save it to the local file. After several trials, I found that there is no easy way to get the source address. The address is embedded in new open window which is triggered by some javascript codes and will be changed each time. So my solution is to open the pdf file and then grep the source address in the page source by the following codes:

```python
# switch to new open window
   new_win = driver.window_handles[1]
   driver.switch_to.window(new_win)

   # print(driver.page_source)
   pdf_file = driver.find_element_by_tag_name("iframe")
   path = pdf_file.get_attribute("src")
```

However, even though we can get the source address it does not mean that we can call some selenium api so that it will download the file for us. Because Selenium ***doest not*** support the download feature. Therefore, I switch to another open lib for help i.e., request.

### what is Request?

> Requests is an elegant and simple HTTP library for Python, built for human beings. You are currently looking at the documentation of the development release.

You can find more tutorial and introductions in [Request](http://docs.python-requests.org/en/master/user/quickstart/).

We can get the e-Statements source content by using request get method and then write it to the local file by triggering file write method. But there is still one problem, that is we have to obtain the secure session so that we can proceed the download action. This would be done by injecting the cookies data retrieved via Selenium's get_cookie() to the request session. The following codes show you how to do it.

```python
    headers = {
        "User-Agent":
            "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36"
    }
    s= requests.session()
    s.headers.update(headers)

    cookies = driver.get_cookies()
    for cookie in driver.get_cookies():
        # c = {cookie['name']: cookie['value']}
        s.cookies.set(cookie['name'], cookie['value'])

    local_filename = "/Users/kelvinleung/Downloads/Test/Test_Args/" + filename + ".pdf"
    r = s.get(path)
    open(local_filename, 'wb').write(r.content)
```

By the way, it's better to use a fake header to cheat the HSBC server as it may discard the request  without header information to prevent attack.

So far, we can be able to auto login to the website and download the statement files.
