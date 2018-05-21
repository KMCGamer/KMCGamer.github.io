---
layout: single
title: Downloading IBM Bigfix CSVs
categories: 
    - programming
tags: 
    - python
    - burp suite
---

I was given a task by my boss to find a way to download data from our BigFix server. The data was a web report in the form of a CSV, and was relatively easy to download if you went through their web interface. This needed to be automated however, so using the browser was not an option.

I spent countless hours going through the BigFix API and I was honestly surprised at what an unorganized mess it really was. There was absolutely no easy way of finding the information I needed. A lot of their documentation pages and forum posts were either 404'd, OR it didn't contain enough information. Needless to say, it was **frustrating**.

My boss recommended to me a solution I hadn't thought of, and that is <a href="https://portswigger.net/burp" target="_blank">Burp Suite</a>. Burp suite is a nifty little program that allows you to view requests and responses through a proxy you can set up through your browser. I could immediately see where he was going with this.

**The plan was as follows:**
1. Turn on the burp suite proxy.
2. Navigate to the servers login page.
3. Record the login POST and response.
4. Parse the session cookie from the response.
5. Navigate to the CSV download button and record the request.
6. Take that request and append our cookie to it.

After configuring the proxy in firefox, I recorded:
- The POST to the server with our credentials to log in.
- The response from the login containing our session cookie.
- The GET request when pressing the CSV download button.

Burp suite has a nice way of outputting to a curl command, but I like using Python. I used <a href="https://curl.trillworks.com/" target="_blank">curl.trillworks.com</a> to convert the commands to be used with Python's requests module.

My weapon of choice for practically all scripts is Python. I started out with something like this:
```python
import requests

def login(username, password):
    data = {
        "page": "LoggingIn",
        "fwdpage": "",
        "Username": username,
        "Password": password
    }

    try:
        response = requests.post('https://our.bigfix.server.edu/webreports',
                                 data=data, verify=False, allow_redirects=False)
    except Exception as err:
        exit()

    return response.headers
```

With this function, I could easily POST my credentials to the server and get a valid session cookie. Now all I had to do was parse it out of the headers.

```python
response = login("xxx","xxx") # Get the response headers
session_cookies = {cookie.split("=")[0]: cookie[:-23].split("=")[1] \
                   for cookie in response['Set-cookie'].split(", ")}
```

This is where is looks a little tricky. To nicely put the cookie into our upcoming GET request, I needed to parse it into a dictionary. The response headers had a lot of unnecessary junk in them, so I had to remove it by using `cookie[:-23]`. The above is called a <a href="https://www.smallsurething.com/list-dict-and-set-comprehensions-by-example/" target="_blank">dictionary comprehension</a>. I sometimes forget that these exist. They can really compress many lines of code into a simple one liner.

Great, now that we have our cookie nicely parsed, you can append it to the GET request.

```python
def grab_vulns_csv(session_cookies):
    params = (
        ('chartSection', '84z212e27eb46e178e98724a33b40f6ee48b5f01'),
        ('collapseState', '4e697d7d6ga8455v505cf25295561c96n2c5cfde'),
        ('wr_contentTable', '9dz6452073e1d34ba76627dbbb810778d858779d'),
        ('reportInfo', '4923112a84a076ab3159bb8b38b41cfb1e61d945'),
        ('newFilter', 'true'),
        ('filter', '{"matchType":"all","conditionList":[{"selectedContentTypeName":"Fixlet","selectorList":[{"selectedOperatorName":"is","selectedOperatorValue":"Fixlet"}],"selectedProperty":{"name":"Type","id":"Type"}},{"selectedContentTypeName":"Fixlet","selectorList":[{"selectedOperatorName":"is","selectedOperatorValue":"Visible"}],"selectedProperty":{"name":"Visibility","id":"Visibility"}},{"selectedContentTypeName":"Fixlet","selectorList":[{"selectedOperatorName":"greater than","selectedOperatorValue":"0"}],"selectedProperty":{"name":"Applicable Computer Count","id":"Applicable Computer Count"}},{"selectedContentTypeName":"Fixlet","selectorList":[{"selectedOperatorName":"contains","selectedOperatorValue":"Critical"}],"selectedProperty":{"name":"Source Severity","id":"Source Severity"}}]}'),
    )

    try:
        response = requests.get('https://our.bigfix.server.edu/csv/ExploreContent.csv',
                                params=params,
                                cookies=session_cookies,
                                verify=False,
                                allow_redirects=False)
        return response.text
    except Exception as err:
        exit()
```

This function returns `response.text` which will have all of our CSV information in it. Now all thats left is writing it to a file which can be done like this:

```python
import csv
from cStringIO import StringIO

vulns_csv = grab_vulns_csv(session_cookies).encode('utf8')
    with open('Vulnerabilities.csv', 'w') as file_obj:
        reader = csv.reader(StringIO(vulns_csv), delimiter=",")
        writer = csv.writer(file_obj)
        # Remove rows we dont need
        # We only need computer and name
        for row in reader:
            del row[7]
            del row[0:5]
            row.reverse() # reverse em so its computer first then name
            writer.writerow(row)
```

I only needed certain columns of information from the downloaded csv so I just deleted some rows as I wrote it to the file.

**And thats it! Mission accomplished!**

*Note: I changed around a bunch of strings in the code to protect sensitive information. Truthfully, I don't know if certain things are necessary to protect, but I'm doing it anyway just to be careful.*
