# Section 12: Web Scraping

*Web scraping* is the term for using a program to download and process content from the web.

Modules will cover in this chapter:

- **webbrowser** Comes with Python and opens a browser to a specific page.

- **requests** Downloads files and web pages from the internet.

- **bs4** Parses HTML, the format that web pages are written in.

- **selenium** Launches and controls a web browser. The selenium module is able to fill in forms and simulate mouse clicks in this browser.

  

## 1. Webbrowser Module

The `webbrowser` module’s `open()` function can launch a new browser to a specified URL. 

```python
import webbrowser
webbrowser.open('https://inventwithpython.com')
```

This is about the only thing the webbrowser module can do.



### Project 1. MAPIT.PY

**Exercise 1** Write a script to automatically launch the map in your browser using the contents of your clipboard

This is what your program does:

1. Gets a street address from the command line arguments or clipboard
2. Opens the web browser to the Google Maps page for the address

This means your code will need to do the following:

1. Read the command line arguments from `sys.argv`.
2. Read the clipboard contents.
3. Call the webbrowser.open() function to open the web browser.

**Step 1**: Figure Out the URL

`https://www.google.com/maps/search/your_address_string`

**Step 2**: Handle the Command Line Arguments

```python
#! python 3
# mapIt.py - Launches a map in the browser using an address from the
# command line or clipboard.


import webbrowser, sys
if len(sys.argv) > 1:
    # Get address from command line.
    address = ''.join(sys.argv[1:])


# TODO: Get address from clipboard.
```

Command line arguments are usually separated by spaces, but in this case, you want to interpret all of the arguments as a single string. Since `sys.argv` is a list of strings, you can pass it to the `join()` method, which returns a single string value. You don’t want the program name in this string, so instead of `sys.argv`, you should pass `sys.argv[1:]` to chop off the first element of the array. 

If you run the program by entering this into the command line . . .

> mapit 870 Valencia St, San Francisco, CA 94110

 . . the `sys.argv` variable will contain this list value:

> ['mapIt.py', '870', 'Valencia', 'St, ', 'San', 'Francisco, ', 'CA', '94110']

**Step 3**: Handle the Clipboard Content and Launch the Browser

```python
#! python 3
# mapIt.py - Launches a map in the browser using an address from the
# command line or clipboard.


import webbrowser, sys, pyperclip
if len(sys.argv) > 1:
    # Get address from command line.
    address = ''.join(sys.argv[1:])
else:
    # Get address from clipboard.
    address = pyperclip.paste()


webbrowser.open('http://www.google.com/maps/search/' + address)
```



## 2. Request Module

### 2.1 Downloading Files from the Web with the requests Module

The `requests` module lets you easily download files from the web without having to worry about complicated issues such as network errors, connection problems, and data compression. The requests module doesn’t come with Python, so you’ll have to install it first. From the command line, run `pip install --user requests`. 

**Downloading a Web Page with the requests.get()` Function**

The `requests.get()` function takes a string of a URL to download. By calling `type()` on `requests.get()`’s return value, you can see that it returns a `Response` object, which contains the response that the web server gave for your request. 

```python
>>> import requests
>>> res = requests.get('https://automatetheboringstuff.com/files/rj.txt')
>>> type(res)
<class 'requests.models.Response'>
>>> res.status_code == requests.codes.ok
True
>>> len(res.text)
178978
>>> print(res.text[:250])
The Project Gutenberg EBook of Romeo and Juliet, by William Shakespeare



This eBook is for the use of anyone anywhere at no cost and with

almost no restrictions whatsoever.  You may copy it, give it away or

re-use it under the terms of the Projec
```

If the request succeeded, the downloaded web page is stored as a string in the `Response` object’s `text` variable. This variable holds a large string of the entire play; the call to `len(res.text)` shows you that it is more than 178,000 characters long. Finally, calling `print(res.text[:250]) `displays only the first 250 characters.

If the request failed and displayed an error message, like “Failed to establish a new connection” or “Max retries exceeded,” then check your internet connection.

**Checking for Errors**

the Response object has a status_code attribute that can be checked against `requests.codes.ok` (a variable that has the integer value 200) to see whether the download succeeded. A simpler way to check for success is to call the `raise_for_status()` method on the `Response` object. This will raise an exception if there was an error downloading the file and will do nothing if the download succeeded.

```python
>>> res = requests.get('https://inventwithpython.com/page_that_does_not_exist')
>>> res.raise_for_status()
Traceback (most recent call last):
  File "<pyshell#11>", line 1, in <module>
    res.raise_for_status()
  File "Your_path\models.py", line 941, in raise_for_status
    raise HTTPError(http_error_msg, response=self)
requests.exceptions.HTTPError: 404 Client Error: Not Found for url: https://inventwithpython.com/page_that_does_not_exist
```

The `raise_for_status()` method is a good way to ensure that a program halts if a bad download occurs. This is a good thing: You want your program to stop as soon as some unexpected error happens. If a failed download *isn’t* a deal breaker for your program, you can wrap the `raise_for_status() line` with `try` and `except` statements to handle this error case without crashing.

```python
>>> import requests
>>> res = requests.get('https://inventwithpython.com/page_that_does_not_exist')
>>> try:
	res.raise_for_status()
except Exception as exc:
	print('There was a problem: %s'%(exc))

	
There was a problem: 404 Client Error: Not Found for url: https://inventwithpython.com/page_that_does_not_exist
```

Always call `raise_for_status()` after calling `requests.get()`. You want to be sure that the download has actually worked before your program continues.



### 2.2 Saving Downloaded Files to the Hard Drive

You can save the web page to a file on your hard drive with the standard `open()` function and `write()` method. There are some slight differences, though. First, you must open the file in **write binary** mode by passing the string `wb` as the second argument to `open()`. Even if the page is in plaintext (such as the *Romeo and Juliet* text you downloaded earlier), you need to write binary data instead of text data in order to maintain the *Unicode encoding* of the text.

To write the web page to a file, you can use a `for` loop with the `Response` object’s `iter_content()` method.

```python
>>> import requests
>>> res = requests.get('https://automatetheboringstuff.com/files/rj.txt')
>>> res.raise_for_status()
>>> playFile = open('RomeoAndJuliet.txt', 'wb')
>>> for chunk in res.iter_content(100000):
	playFile.write(chunk)

	
100000
78978
>>> playFile.close()
```

The `iter_content()` method returns “chunks” of the content on each iteration through the loop. Each chunk is of the `bytes` data type, and you get to specify how many bytes each chunk will contain. One hundred thousand bytes is generally a good size, so pass 100000 as the argument to `iter_content()`. The file *RomeoAndJuliet.txt* will now exist in the current working directory. 

> **UNICODE ENCODINGS**
>
> Unicode encodings are beyond the scope of this book, but you can learn more about them from these web pages:
>
> - Joel on Software: The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!): *https://www.joelonsoftware.com/articles/Unicode.html*
> - Pragmatic Unicode: *https://nedbatchelder.com/text/unipain.html*
>
> ![FactsofLife](https://nedbatchelder.com/text/unipain_pix/041.png)
>
> To review, these are the five unavoidable Facts of Life:
>
> 1. All input and output of your program is bytes.
> 2. The world needs more than 256 symbols to communicate text.
> 3. Your program has to deal with both bytes and Unicode.
> 4. A stream of bytes can’t tell you its encoding.
> 5. Encoding specifications can be wrong.
>
> ![ProTips](https://nedbatchelder.com/text/unipain_pix/042.png)
>
> These are the three Pro Tips to keep in mind as you build your software to keep your code Unicode-clean:
>
> 1. Unicode sandwich: keep all text in your program as Unicode, and convert as close to the edges as possible.
> 2. Know what your strings are: you should be able to explain which of your strings are Unicode, which are bytes, and for your byte strings, what encoding they use.
> 3. Test your Unicode support. Use exotic strings throughout your test suites to be sure you’re covering all the cases.
>
> ![UnicodeSandwich](https://nedbatchelder.com/text/unipain_pix/034.png)

The write() method returns the number of bytes written to the file. In the previous example, there were 100,000 bytes in the first chunk, and the remaining part of the file needed only 78,981 bytes.

To review, here’s the complete process for downloading and saving a file:

1. Call `requests.get()` to download the file.
2. Call `open()` with `'wb'` to create a new file in write binary mode.
3. Loop over the `Response` object’s `iter_content()` method.
4. Call `write()` on each iteration to write the content to the file.
5. Call `close()` to close the file.

That’s all there is to the `requests` module! The for loop and `iter_content()` stuff may seem complicated compared to the `open()/write()/close()` workflow you’ve been using to write text files, but it’s to ensure that the requests module doesn’t eat up too much memory even if you download massive files. You can learn about the requests module’s other features from *[https://requests.readthedocs.org](https://requests.readthedocs.org/)*



## 3. bs4 Module

### 3.1 HTML Basics

> #### beginner tutorials for Learning HTML
>
> - *https://developer.mozilla.org/en-US/learn/html/*
> - *https://htmldog.com/guides/html/beginner/*
> - *https://www.codecademy.com/learn/learn-html*

There are many different tags in HTML. Some of these tags have extra properties in the form of *attributes* within the angle brackets. For example, the `<a>` tag encloses text that should be a link. The URL that the text links to is determined by the `href` attribute. Here’s an example:

```html
Al's free <a href="https://inventwithpython.com">Python books</a>.
```

Some elements have an id attribute that is used to uniquely identify the element in the page. You will often instruct your programs to seek out an element by its `id` attribute, so figuring out an element’s id attribute using the browser’s developer tools is a common task in writing web scraping programs.

**Viewing the Source HTML of a Web Page**

To look at the HTML source of the web pages, right-click and select **View Source** or **View page source** to see the HTML text of the page.

I highly recommend viewing the source HTML of some of your favorite sites. It’s fine if you don’t fully understand what you are seeing when you look at the source. You just need enough knowledge to pick out data from an existing site.

**Opening Your Browser’s Developer Tools**

You can also look through a page’s HTML using your browser’s developer tools. In Chrome and Internet Explorer for Windows, the developer tools are already installed, and you can press `F12` to make them appear. Pressing F12 again will make the developer tools disappear. In Chrome, you can also bring up the developer tools by selecting **View** ▸ **Developer** ▸ **Developer Tools**.

In Firefox, you can bring up the Web Developer Tools Inspector by pressing `CTRL-SHIFT-C` on Windows and Linux.

After enabling or installing the developer tools in your browser, you can right-click any part of the web page and select **Inspect Element** from the context menu to bring up the HTML responsible for that part of the page. This will be helpful when you begin to parse HTML for your web scraping programs. 

> **DON’T USE REGULAR EXPRESSIONS TO PARSE HTML**
>
> Locating a specific piece of HTML in a string seems like a perfect case for regular expressions. However, I advise you against it. There are many different ways that HTML can be formatted and still be considered valid HTML, but trying to capture all these possible variations in a regular expression can be tedious and error prone. A module developed specifically for parsing HTML, such as `bs4`, will be less likely to result in bugs.
>
> You can find an extended argument for why you shouldn’t parse HTML with regular expressions at *https://stackoverflow.com/a/1732454/1893164/*.

**Using the Developer Tools to Find HTML Elements**

Once your program has downloaded a web page using the `requests` module, you will have the page’s HTML content as a single string value. Now you need to figure out which part of the HTML corresponds to the information on the web page you’re interested in.

Say you want to scraping the weather information for particular ZIP codes from *https://weather.gov/*. Right-click where it is on the page and select **Inspect Element** from the context menu that appears. This will bring up the Developer Tools window, which shows you the HTML that produces this particular part of the web page. Note that if the *https://weather.gov/* site changes the design of its web pages, you’ll need to repeat this process to inspect the new elements.

![InspectionElements](https://automatetheboringstuff.com/2e/images/000094.jpg)

From the developer tools, you can see that the HTML responsible for the forecast part of the web page is `<div class="col-sm-10 forecast-text">Sunny, with a high near 64. West wind 11 to 16 mph, with gusts as high as 21 mph.</div>`. It seems that the forecast information is contained inside a `<div>` element with the `forecast-text` CSS class. Right-click on this element in the browser’s developer console, and from the context menu that appears, select **Copy** ▸ **CSS Selector**. This will copy a string such as `'div.row-odd:nth-child(1) > div:nth-child(2)'` to the clipboard. You can use this string for `Beautiful Soup’s select()` or `Selenium’s find_element_by_css_selector()` methods.



### 3.2 Parsing HTML with the Beautiful Soup Module

Beautiful Soup is a module for extracting information from an HTML page (and is much better for this purpose than regular expressions). The Beautiful Soup module’s name is `bs4` (for Beautiful Soup, version 4). To install it, you will need to run `pip install --user beautifulsoup4` from the command line. While `beautifulsoup4` is the name used for installation, to import Beautiful Soup you run import bs4.

For this chapter, the Beautiful Soup examples will *parse* (that is, analyze and identify the parts of) an HTML file on the hard drive. Alternatively, download it from *https://nostarch.com/automatestuff2/*.

```html
<!-- This is the example.html example file. -->

<html><head><title>The Website Title</title></head>
<body>
<p>Download my <strong>Python</strong> book from <a href="https://
inventwithpython.com">my website</a>.</p>
<p class="slogan">Learn Python the easy way!</p>
<p>By <span id="author">Al Sweigart</span></p>
</body></html>
```



**Creating a BeautifulSoup Object from HTML**

The `bs4.BeautifulSoup()` function needs to be called with a string containing the HTML it will parse. The `bs4.BeautifulSoup()` function returns a `BeautifulSoup` object.

```python
>>> import requests, bs4
>>> res = requests.get('https://nostarch.com')
>>> res.raise_for_status()
>>> noStarchSoup = bs4.BeautifulSoup(res.text, 'html.parser')
>>> type(noStarchSoup)
<class 'bs4.BeautifulSoup'>
```

This code uses `requests.get()` to download the main page from the No Starch Press website and then passes the text attribute of the response to `bs4.BeautifulSoup()`. The `BeautifulSoup` object that it returns is stored in a variable named `noStarchSoup`.

You can also load an HTML file from your hard drive by passing a `File` object to `bs4.BeautifulSoup() `along with a second argument that tells Beautiful Soup which parser to use to analyze the HTML.

```python
>>> exampleFile = open('example.html')
>>> exampleSoup = bs4.BeautifulSoup(exampleFile, 'html.parser')
>>> type(exampleSoup)
<class 'bs4.BeautifulSoup'>
```

The `html.parser` parser used here comes with Python. However, you can use the faster `lxml` parser if you install the third-party `lxml` module. Forgetting to include this second argument will result in a `UserWarning: No parser was explicitly specified warning`.

**Finding an Element with the select() Method**

You can retrieve a web page element from a `BeautifulSoup` object by calling the `select()` method and passing a string of a CSS *selector* for the element you are looking for. Selectors are like regular expressions: they specify a pattern to look for—in this case, in HTML pages instead of general text strings.

> A good selector tutorial in the resources at *https://nostarch.com/automatestuff2/*)
>
> https://automatetheboringstuff.com/list-of-css-selector-tutorials.html

| **Selector passed to the \**select()\** method** | **Will match . . .**                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| soup.select('div')                               | All elements named <div>                                     |
| soup.select('#author')                           | The element with an id attribute of author                   |
| soup.select('.notice')                           | All elements that use a CSS class attribute named notice     |
| soup.select('div span')                          | All elements named <span> that are within an element named <div> |
| soup.select('div > span')                        | All elements named <span> that are *directly* within an element named <div>, with no other element in between |
| soup.select('input[name]')                       | All elements named <input> that have a name attribute with any value |
| soup.select('input[type="button"]')              | All elements named <input> that have an attribute named type with value button |

The various selector patterns can be combined to make sophisticated matches. For example, `soup.select('p #author')` will match any element that has an `id` attribute of `author`, as long as it is also inside a `<p>` element. Instead of writing the selector yourself, you can also right-click on the element in your browser and select **Inspect Element**. When the browser’s developer console opens, right-click on the element’s HTML and select **Copy** ▸ **CSS Selector** to copy the selector string to the clipboard and paste it into your source code.

The `select()` method will return a **list** of `Tag` objects, which is how Beautiful Soup represents an HTML element. The list will contain one `Tag` object for every match in the `BeautifulSoup` object’s HTML. Tag values can be passed to the `str()` function to show the HTML tags they represent. Tag values also have an `attrs` attribute that shows all the HTML attributes of the tag as a **dictionary**. Using the *example.html* file from earlier, enter the following into the interactive shell:

```python
>>> import bs4
>>> exampleFile = open('example.html')
>>> exampleSoup = bs4.BeautifulSoup(exampleFile.read(), 'html.parser')
>>> elems = exampleSoup.select('#author')
>>> type(elems) # elems is a list of Tag objects.
<class 'bs4.element.ResultSet'>
>>> len(elems)		       
1
>>> type(elems[0])		       
<class 'bs4.element.Tag'>
>>> str(elems[0]) # The Tag object as a string.	       
'<span id="author">Al Sweigart</span>'
>>> elems[0].getText()		       
'Al Sweigart'
>>> elems[0].attrs	       
{'id': 'author'}
```

This code will pull the element with `id="author"` out of our example HTML. We use `select('#author')` to return a list of all the elements with `id="author"`. We store this list of **Tag** objects in the variable `elems`, and l`en(elems)` tells us there is one **Tag** object in the list; there was one match. Calling `getText()` on the element returns the element’s text, or inner HTML. The text of an element is the content between the opening and closing tags: in this case, `Al Sweigart`.

Passing the element to `str()` returns a string with the starting and closing tags and the element’s text. Finally, `attrs` gives us a dictionary with the element’s attribute, `id`, and the value of the id attribute, `author`.

You can also pull all the `<p>` elements from the `BeautifulSoup object`. 

```python
>>> pElems = exampleSoup.select('p')
>>> str(pElems[0])
'<p>Download my <strong>Python</strong> book from <a href="https://
inventwithpython.com">my website</a>.</p>'
>>> pElems[0].getText()
'Download my Python book from my website.'
>>> str(pElems[1])
'<p class="slogan">Learn Python the easy way!</p>'
>>> pElems[1].getText()
'Learn Python the easy way!'
>>> str(pElems[2])
'<p>By <span id="author">Al Sweigart</span></p>'
>>> pElems[2].getText()
'By Al Sweigart'
```

This time, `select()` gives us a list of three matches, which we store in `pElems`. Using `str()` on `pElems[0]`, `pElems[1]`, and `pElems[2]` shows you each element as a string, and using `getText()` on each element shows you its text.

**Getting Data from an Element’s Attributes**

The `get()` method for `Tag` objects makes it simple to access attribute values from an element. The method is passed a string of an attribute name and returns that attribute’s value. 

```python
>>> import bs4
>>> soup = bs4.BeautifulSoup(open('example.html'), 'html.parser')
>>> spanElem = soup.select('span')[0]
>>> str(spanElem)
'<span id="author">Al Sweigart</span>'
>>> spanElem.get('id')
'author'
>>> spanElem.get('some_nonexistent_addr') == None
True
>>> spanElem.attrs
{'id': 'author'}
```

Here we use `select()` to find any `<span>` elements and then store the first matched element in `spanElem`. Passing the attribute name `'id'` to `get()` returns the attribute’s value, `author`.



### Project 1. OPENING ALL SEARCH RESULT

Write a script to do this with the search results page for the Python Package Index at *https://pypi.org/*

This is what your program does:

1. Gets search keywords from the command line arguments
2. Retrieves the search results page
3. Opens a browser tab for each result

This means your code will need to do the following:

1. Read the command line arguments from sys.argv.
2. Fetch the search result page with the requests module.
3. Find the links to each search result.
4. Call the webbrowser.open() function to open the web browser.

**Step 1: Get the Command Line Arguments and Request the Search Page**

Before coding anything, you first need to know the URL of the search result page. By looking at the browser’s address bar after doing a search, you can see that the result page has a URL like *https://pypi.org/search/?q=*. The `requests` module can download this page and then you can use `Beautiful Soup` to find the search result links in the HTML. Finally, you’ll use the `webbrowser` module to open those links in browser tabs.

```python
#! python3
# searchpypi.py - Opens several search results.

import requests, sys, webbrowser, bs4
print('Searching...') # display text while downloading the search result page
res = requests.get('https://google.com/search?q=' 'https://pypi.org/search/?q=' 
	+ ' '.join(sys.argv[1:]))
res.raise_for_status()


# TODO: Retrieve top search result links.


# TODO: Open a browser tab for each result.
```

The user will specify the search terms using command line arguments when they launch the program. These arguments will be stored as strings in a list in `sys.argv`.

**Step 2: Find All the Results**

```python
#! python3
# searchpypi.py - Opens several google results.
import requests, sys, webbrowser, bs4
--snip--
# Retrieve top search result links.
soup = bs4.BeautifulSoup(res.text, 'html.parser')
# Open a browser tab for each result.
linkElems = soup.select('.package-snippet')
```

If you look at the `<a>` elements, though, the search result links all have `class="package-snippet"`. Looking through the rest of the HTML source, it looks like the `package-snippet` class is used only for search result links. You don’t have to know what the CSS class `package-snippet` is or what it does. You’re just going to use it as a marker for the `<a>` element you are looking for. You can create a `BeautifulSoup` object from the downloaded page’s HTML text and then use the selector `.package-snippet` to find all `<a>` elements that are within an element that has the `package-snippet` CSS class. Note that if the PyPI website changes its layout, you may need to update this program with a new CSS selector string to pass to `soup.select()`. The rest of the program will still be up to date.

**Step 3: Open Web Browsers for Each Result**

```python
#! python3
# searchpypi.py - Opens several search results.
import requests, sys, webbrowser, bs4
--snip--
# Open a browser tab for each result.
linkElems = soup.select('.package-snippet')
numOpen = min(5, len(linkElems))
for i in range(numOpen):
    urlToOpen = 'https://pypi.org' + linkElems[i].get('href')
    print('Opening', urlToOpen)
    webbrowser.open(urlToOpen)
```

By default, you open the first five search results in new tabs using the `webbrowser` module. However, the user may have searched for something that turned up fewer than five results. The `soup.select()` call returns a list of all the elements that matched your `.package-snippet` selector, so the number of tabs you want to open is either 5 or the length of this list (whichever is smaller).

The built-in Python function `min()` returns the smallest of the integer or float arguments it is passed. (There is also a built-in `max()` function that returns the largest argument it is passed.) You can use `min()` to find out whether there are fewer than five links in the list and store the number of links to open in a variable named `numOpen`. Then you can run through a for loop by calling `range(numOpen)`.

On each iteration of the loop, you use `webbrowser.open()` to open a new tab in the web browser. Note that the `href` attribute’s value in the returned `<a>` elements do not have the initial https://pypi.org part, so you have to concatenate that to the `href` attribute’s string value.

Now you can instantly open the first five PyPI search results for, say, *boring stuff* by running `searchpypi boring stuff` on the command line! (See [Appendix B](https://automatetheboringstuff.com/2e/chapter12/#calibre_link-35) for how to easily run programs on your operating system.)



### Project 2. DOWNLOADING ALL XKCD COMICS

Here’s what your program does:

1. Loads the XKCD home page
2. Saves the comic image on that page
3. Follows the Previous Comic link
4. Repeats until it reaches the first comic

This means your code will need to do the following:

1. Download pages with the `requests` module
2. Find the URL of the comic image for a page using Beautiful Soup
3. Download and save the comic image to the hard drive with `iter_content()`
4. Find the URL of the Previous Comic link, and repeat

**Step 1: Design the Program**

- The URL of the comic’s image file is given by the `href` attribute of an `<img>` element.
- The `<img>` element is inside a `<div id="comic">` element.
- The Prev button has a `rel` HTML attribute with the value `prev`.
- The first comic’s Prev button links to the *https://xkcd.com/#* URL, indicating that there are no more previous pages.

```python
#! python3
# downloadXkcd.py - Downloads every single XKCD comic.

import requests, os, bs4

url = 'https://xkcd.com' # starting url
os.makedirs('xkcd', exist_ok=True) # store comics in ./xkcd
while not url.endswith('#'):
	# TODO: Download the page

	# TODO: Find the URL of the comic image

	# TODO: Download the image

	# TODO: Save the image to ./xkcd

	# TODO: Get the Prev button's url

print('Done.')
```

You’ll have a `url` variable that starts with the value 'https://xkcd.com' and repeatedly update it (in a for loop) with the URL of the current page’s Prev link. At every step in the loop, you’ll download the comic at `url`. You’ll know to end the loop when url ends with `#`'.

You will download the image files to a folder in the current working directory named `xkcd`. The call `os.makedirs()` ensures that this folder exists, and the `exist_ok=True` keyword argument prevents the function from throwing an exception if this folder already exists. 

**Step 2: Download the Web Page**

```python
while not url.endswith('#'):
    # Download the page.
    print('Downloading page %s...' % url)
    res = requests.get(url)
    res.raise_for_status()

    soup = bs4.BeautifulSoup(res.text, 'html.parser')

    # TODO: Find the URL of the comic image.
    # TODO: Download the image.
    # TODO: Save the image to ./xkcd.
    # TODO: Get the Prev button's url.
print('Done.')
```

First, print `url` so that the user knows which URL the program is about to download; then use the `requests` module’s `request.get()` function to download it. As always, you immediately call the `Response` object’s `raise_for_status()` method to throw an exception and end the program if something went wrong with the download. Otherwise, you create a `BeautifulSoup` object from the text of the downloaded page.

**Step 3: Find and Download the Comic Image**

```python
while not url.endswith('#'):
	# Download the page.

	# Find the URL of the comic image
	comicElem = soup.select('#comic img')
	if comicElem == []:
		print('Could not find comic image.')
	else:
		comicURL = 'https' + comicElem[0].get('src')
		# Download the image.
		print('Downloading image %s...' % (comicURL))
		res = requests.get(comicURL)
		res.raise_for_status()

	# TODO: Save the image to ./xkcd
	# TODO: Get the Prev button's url
print('Done.')
```

From inspecting the XKCD home page with your developer tools, you know that the `<img>` element for the comic image is inside a `<div>` element with the `id` attribute set to `comic`, so the selector `#comic img` will get you the correct `<img>` element from the `BeautifulSoup` object.

A few XKCD pages have special content that isn’t a simple image file. That’s fine; you’ll just skip those. If your selector doesn’t find any elements, then `soup.select('#comic img')` will return a blank list. When that happens, the program can just print an error message and move on without downloading the image.

Otherwise, the selector will return a list containing one `<img>` element. You can get the `src` attribute from this `<img>` element and pass it to `requests.get()` to download the comic’s image file.

**Step 4: Save the Image and Find the Previous Comic**

```python
while not url.endswith('#'):
	# Download the page.
	# Find the URL of the comic image
    
    # Save the image to ./xkcd.
    imageFile = open(os.path.join('xkcd', os.path.basename(comicUrl)),
'wb')
    for chunk in res.iter_content(100000):
        imageFile.write(chunk)
        imageFile.close()

    # Get the Prev button's url.
    prevLink = soup.select('a[rel="prev"]')[0]
    url = 'https://xkcd.com' + prevLink.get('href')

print('Done.')
```

At this point, the image file of the comic is stored in the res variable. You need to write this image data to a file on the hard drive.

You’ll need a filename for the local image file to pass to `open()`. The comicUrl will have a value like 'https://imgs.xkcd.com/comics/heartbleed_explanation.png'—which you might have noticed looks a lot like a file path. And in fact, you can call `os.path.basename()` with` comicUrl`, and it will return just the last part of the URL, `heartbleed_explanation.png`. You can use this as the filename when saving the image to your hard drive. You join this name with the name of your `xkcd` folder using `os.path.join()` so that your program uses backslashes `\` on Windows and forward slashes `/` on macOS and Linux. Now that you finally have the filename, you can call `open()` to open a new file in `wb` “write binary” mode.

To save files you’ve downloaded using requests, you need to loop over the return value of the `iter_content()` method. The code in the `for` loop writes out chunks of the image data (at most 100,000 bytes each) to the file and then you close the file. The image is now saved to your hard drive.

The selector `'a[rel="prev"]'` identifies the `<a>` element with the `rel` attribute set to `prev`, and you can use this `<a>` element’s `href` attribute to get the previous comic’s URL, which gets stored in `url`. Then the `while` loop begins the entire download process again for this comic.

This project is a good example of a program that can automatically follow links in order to scrape large amounts of data from the web. You can learn about Beautiful Soup’s other features from its documentation at *https://www.crummy.com/software/BeautifulSoup/bs4/doc/.*

**Ideas for Similar Programs**

Downloading pages and following links are the basis of many web crawling programs. Similar programs could also do the following:

- Back up an entire site by following all of its links.
- Copy all the messages off a web forum.
- Duplicate the catalog of items for sale on an online store.

The requests and `bs4` modules are great as long as you can figure out the URL you need to pass to `requests.get()`. However, sometimes this isn’t so easy to find. Or perhaps the website you want your program to navigate requires you to log in first. The `selenium` module will give your programs the power to perform such sophisticated tasks.



## 4. Controlling the Browser with the selenium Module

### 4.1 Starting a selenium-Controlled Browser

```python
>>> from selenium import webdriver
>>> browser = webdriver.Firefox()
>>> type(browser)
<class 'selenium.webdriver.firefox.webdriver.WebDriver'>
>>> browser.get('https://inventwithpython.com')
```

If you encounter the error message “'geckodriver' executable needs to be in PATH.”, then you need to manually download the webdriver for Firefox before you can use selenium to control it. For Firefox, go to *https://github.com/mozilla/geckodriver/releases* and download the geckodriver for your operating system. (“Gecko” is the name of the browser engine used in Firefox.) For Chrome, go to *https://sites.google.com/a/chromium.org/chromedriver/downloads* and download the ZIP file for your operating system. For installing a specific version of selenium, you might run `pip install --user -U selenium==<version number>`



### 4.2 Finding Elements on the Page

The `find_element_*` methods return a single `WebElement` object, representing the first element on the page that matches your query. The `find_elements_*` methods return a list of `WebElement_*` objects for *every* matching element on the page.

| **Method name**                                              | **WebElement** object/list returned                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| browser.find_element_by_class_name(*name*)<br/>browser.find_elements_by_class_name(*name*) | Elements that use the CSS class *name*                       |
| browser.find_element_by_css_selector(*selector*)<br/>browser.find_elements_by_css_selector(*selector*) | Elements that match the CSS *selector*                       |
| browser.find_element_by_id(*id*)<br/>browser.find_elements_by_id(*id*) | Elements with a matching *id* attribute value                |
| browser.find_element_by_link_text(*text*)<br/>browser.find_elements_by_link_text(*text*) | `<a>` elements that completely match the *text* provided     |
| browser.find_element_by_partial_link_text(*text*)<br/>browser.find_elements_by_partial_link_text(*text*) | `<a>` elements that contain the *text* provided              |
| browser.find_element_by_name(*name*)<br/>browser.find_elements_by_name(*name*) | Elements with a matching *name* attribute value              |
| browser.find_element_by_tag_name(*name*)<br/>browser.find_elements_by_tag_name(*name*) | Elements with a matching tag *name* (case-insensitive; an `<a>` element is matched by 'a' and 'A') |

If no elements exist on the page that match what the method is looking for, the selenium module raises a `NoSuchElement` exception. If you do not want this exception to crash your program, add `try` and `except` statements to your code. Once you have the `WebElement` object, you can find out more about it by reading the attributes or calling the methods.

| **Attribute or method** | **Description**                                              |
| ----------------------- | ------------------------------------------------------------ |
| tag_name                | The tag name, such as 'a' for an `<a>` element               |
| get_attribute(*name*)   | The value for the element’s name attribute                   |
| text                    | The text within the element, such as 'hello' in `<span>hello </span>` |
| clear()                 | For text field or text area elements, clears the text typed into it |
| is_displayed()          | Returns `True` if the element is visible; otherwise returns `False` |
| is_enabled()            | For input elements, returns `True` if the element is enabled; otherwise returns `False` |
| is_selected()           | For checkbox or radio button elements, returns `True` if the element is selected; otherwise returns `False` |
| location                | A dictionary with keys `x` and `y` for the position of the element in the page |

**Example**

```python
>>> from selenium import webdriver
>>> browser = webdriver.Firefox()
>>> browser.get('https://inventwithpython.com')
>>> try:
	elem = browser.find_element_by_class_name('cover-thumb')
	print('Found <%s> element with that class name!' % (elem.tag_name))
except:
	print('Was not able to find an element with that name.')

	
Found <img> element with that class name!
```



### 4.3 Clicking the Page

`WebElement` objects returned from the `find_element_*` and `find_elements_*` methods have a `click()` method that simulates a mouse click on that element. This method can be used to 

- follow a link
- make a selection on a radio button
- click a Submit button
- or trigger whatever else might happen when the element is clicked by the mouse 

**Example**

```python
>>> from selenium import webdriver
>>> browser = webdriver.Firefox()
>>> browser.get('https://inventwithpython.com')
>>> linkElem = browser.find_element_by_link_text('Read Online for Free')
>>> type(linkElem)
<class 'selenium.webdriver.firefox.webelement.FirefoxWebElement'>
>>> linkElem.click() # follows the "Read Online for Free" link
```

This gets the `WebElement` object for the `<a>` element with the text *Read It Online*, and then simulates clicking that `<a>` element.



### 4.4 Filling Out and Submitting Forms

Sending keystrokes to text fields on a web page is a matter of finding the `<input>` or `<textarea>` element for that text field and then calling the `send_keys()`method. 

**Example**

```python
>>> from selenium import webdriver
>>> browser = webdriver.Firefox()
>>> browser.get('https://login.metafilter.com')
>>> userElem = browser.find_element_by_id('user_name')
>>> userElem.send_keys('your_real_username_here')
>>> 
>>> passwordElem = browser.find_element_by_id('user_pass')
>>> passwordElem.send_keys('your_real_password_here')
>>> passwordElem.submit()
```

Calling the `submit()` method on any element will have the same result as clicking the Submit button for the form that element is in. (You could have just as easily called emailElem.submit(), and the code would have done the same thing.) You can always use the browser’s inspector to verify the `id` if MetaFilter changed the `id` of the Username and Password text fields.

> Avoid putting your passwords in source code whenever possible. It’s easy to accidentally leak your passwords to others when they are left unencrypted on your hard drive. If possible, have your program prompt users to enter their passwords from the keyboard using the `pyinputplus.inputPassword()` function			



### 4.5 Sending Special Keys

The `selenium` module has a module for keyboard keys that are impossible to type into a string value, which function much like escape characters. These values are stored in attributes in the `selenium.webdriver.common.keys` module. To make life easier, just run`from selenium.webdriver.common.keys import Keys` at the top of your program and you can simply write `Keys` anywhere.

| **Attributes**                                    | **Meanings**                                     |
| ------------------------------------------------- | ------------------------------------------------ |
| Keys.DOWN, Keys.UP, Keys.LEFT, Keys.RIGHT         | The keyboard arrow keys                          |
| Keys.ENTER, Keys.RETURN                           | The `ENTER` and `RETURN` keys                    |
| Keys.HOME, Keys.END, Keys.PAGE_DOWN, Keys.PAGE_UP | The `HOME`, `END`, `PAGEDOWN`, and `PAGEUP` keys |
| Keys.ESCAPE, Keys.BACK_SPACE, Keys.DELETE         | The `ESC`, `BACKSPACE`, and `DELETE` keys        |
| Keys.F1, Keys.F2, . . . , Keys.F12                | The F1 to F12 keys at the top of the keyboard    |
| Keys.TAB                                          | The `TAB` key                                    |

**Example**

```python
>>> from selenium import webdriver
>>> from selenium.webdriver.common.keys import Keys
>>> browser = webdriver.Firefox()
>>> browser.get('https://nostarch.com')
>>> htmlElem = browser.find_element_by_tag_name('html')
>>> htmlElem.send_keys(Keys.END)     # scrolls to bottom
>>> htmlElem.send_keys(Keys.HOME)    # scrolls to top
```

The `<html>` tag is the base tag in HTML files: the full content of the HTML file is enclosed within the `<html>` and `</html>` tags. Calling `browser.find_element_by_tag_name('html')` is a good place to send keys to the general web page. This would be useful if, for example, new content is loaded once you’ve scrolled to the bottom of the page.



### 4.6 Clicking Browser Buttons

`browser.back()` Clicks the Back button.

`browser.forward()` Clicks the Forward button.

`browser.refresh()` Clicks the Refresh/Reload button.

`browser.quit()` Clicks the Close Window button.



## 4.7 More Info on Selenium

It also can modify your browser’s cookies, take screenshots of web pages, and run custom JavaScript. To learn more about these features, you can visit the selenium documentation at *https://selenium-python.readthedocs.org/*.



## Summary

The `requests` module makes downloading straightforward, and with some basic knowledge of HTML concepts and selectors, you can utilize the `BeautifulSoup` module to parse the pages you download.

But to fully automate any web-based tasks, you need direct control of your web browser through the `selenium` module. The `selenium` module will allow you to log in to websites and fill out forms automatically. 
