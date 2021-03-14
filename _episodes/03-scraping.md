---
title: "Web Scraping"
teaching: 0
exercises: 0
questions:
- "How can I obtain data in a programmatic way from web pages?"
objectives:
- "Understand "
keypoints:
- TODO 
---

Sometimes, 
the data we are looking for is not available from an API,
but it is available on web pages that we can view with our browser.
As an example task,
in this episode we are going 
to use
the BeautifulSoup Python package
for web scraping
to find all the relevant information
about future Software Carpentry Workshop events.

## Exploring HTML code in the browser

Navigate to [carpentries.org](www.carpentries.org).
The page we see has been rendered by the browser
from the HTML, CSS (Cascading Style Sheet) and JavaScript code
that is available or linked in the page in some way.

In most browsers 
(Chrome, Chromium and Firefox), 
we can look at the HTML source code
of the page we are viewing
with the `CTRL+u` shortcut
(alternatively, you can 
right click on the page 
and choose "view source"
from the context menu).

Things to notice:
- HTML elements can be nested, 
  and form (approximately) a tree. 
- Most elements have an opening tag `<tagname>`, 
  and a corresponding closing one `</tagname>`.
  For examples, see the [reference on w3schools.com](https://www.w3schools.com/tags/default.asp)
- Each element can have __attributes__, 
  defined in the opening tag.

Another way to explore the HTML code
is to use the Developer Tools.
In most browser,
(Chrome, Chromium and Firefox), 
you can use the `CTRL+Shift+I` key combination 
to open the Developer Tools
(alternatively, find the right option
in your browser menu).

By using these, 
by pressing the combination `CTRL+Shift+C`
(or clicking on the mouse pointer icon 
in the top left of the window)
you can hover with the mouse 
on the elements in the rendered page
and view their properties.
If you click on one of these,
the relevant part of the HTML code 
will be shown to you.

By using these techniques, 
we can understand how to locate the elements 
that we want 
when using BeautifulSoup 
later on.

### Relevant HTML tags for this lessons 
There is a number of tags 
that may be interesting in general,
but specifically for what follows, 
we need to notice:
- the `<table>` tag, which starts a table,
  which is composed of rows;
- the `<tr>` tag, which starts a Table Row
  inside a `<table> ... </table>` element;
- the `<td>` tag, which starts a Table Data cell
  inside a `<tr> ... </tr>` element;
- the `<a>` flag, meaning an "anchor" object,
  used for __hyperlinks__, 
  usually in the form 
  `<a href="http://somewhere.com/an/u/r/i">`,
  i.e., with an "href" attribute.

## Scraping the page with Beautiful Soup

From the [BeautifulSoup documentaion](https://www.crummy.com/software/BeautifulSoup/bs4/doc/):

> Beautiful Soup is a Python library for pulling data out of HTML and XML files. It works with your favorite parser to provide idiomatic ways of navigating, searching, and modifying the parse tree. It commonly saves programmers hours or days of work.
{: .quote}

First of all, 
we import the necessary libraries 
and use `requests` to GET 
the Carpentries website content:
~~~
import requests
from bs4 import BeautifulSoup

response = requests.get("http://www.carpentries.org")
response
~~~
{: .language-python}
~~~
<Response [200]>
~~~
{: .output}
Request was successful.
The HTML of the web page 
is in the `text` member of the response.
We can pass that directly 
the the BeautifulSoup constructor,
obtainint a soup object 
that we still need to navigate:
~~~
soup = BeautifulSoup(r.text,"html.parser")
~~~
Looking at the HTML code,
we see that just above our table 
there is the text "Upcoming Carpentries Workshops"
inside a `<h2>` tag (code reindented for clarity)
~~~
...
<div class="row">
  <div class="medium-12 columns">
    <h2>Upcoming Carpentries Workshops</h2>
    
    Click on an individual event to learn more about that event, including contact information and registration instructions.

    <table class="table table-striped" style="width: 100%;">
      <tr> <td>
          <img src="https://carpentries.org/assets/img/logos/lc.svg" title="lc workshop" alt="lc logo" width="24" height="24" class="flags"/>
        </td>

        <td>
          <img src="https://carpentries.org/assets/img/flags/24/us.png" title="US" alt="us"  class="flags"/>
          
          <img src="https://carpentries.org/assets/img/flags/24/w3.png" title="Online" alt="globe image" class="flags"/>
          
          <a href="https://annajiat.github.io/2021-01-22-uab-NNLM-online">University of Alabama at Birmingham (online)</a>
          
          <br/>
          <b>Instructors:</b> Annajiat Alim Rasel, Cody Hennesy, Camilla Bressan, Mary Ann Warner
          
        </td>
        <td>
          Jan 22 - Apr 23, 2021
        </td>
      </tr>
...
~~~
{: .language-html}
We can then look for the table 
by finding the HTML element 
that contains that text:
~~~
(soup.find(string = "Upcoming Carpentries Workshops"))
~~~
{: .language-python}
~~~
'Upcoming Carpentries Workshops'
~~~
{: .output}
But how do we get the parent element?
We can use the `find_parents()` method,
which returns a list of parents 
of a given element,
starting from immediate parent 
of the element itself 
and ending with the root element
(`soup` in this case).
The second parent in the list
is the one that also contains 
the table we are interested in:
~~~
(soup
 .find(string = "Upcoming Carpentries Workshops")
 .find_parents()[1])
~~~
{: .language-python}
~~~
<div class="medium-12 columns">
<h2>Upcoming Carpentries Workshops</h2>
          
	  Click on an individual event to learn more about that event, including contact information and registration instructions.

<table class="table table-striped" style="width: 100%;">
<tr>
<td>
<img alt="lc logo" class="flags" height="24" src="https://carpentries.org/assets/img/logos/lc.svg" title="lc workshop" width="24">
</img></td>
<td>
<img alt="us" class="flags" src="https://carpentries.org/assets/img/flags/24
~~~
{: .language-html}
It seems we are on the right track.
Now let's focus on the "table" element:
~~~
(soup
 .find(string = "upcoming carpentries workshops")
 .find_parents()[1]
 .find("table"))
~~~
{: .language-python}
~~~
<table class="table table-striped" style="width: 100%;">
<tr>
<td>
<img alt="lc logo" class="flags" height="24" src="https://carpentries.org/assets/img/logos/lc.svg" title="lc workshop" width="24">
</img></td>
<td>
<img alt="us" class="flags" src="https://carpentries.org/assets/img/flags/24/us.png" title="US">
<img alt="globe image" class="flags" src="https://carpentries.org/assets/img/flags/24/w3.png" title="Online">
<a href="https://annajiat.github.io/2021-01-22-uab-NNLM-online">University of
~~~
{: .language-html}
Now we can get a list of row elements with
~~~
rows = (soup
 .find(string = "upcoming carpentries workshops")
 .find_parents()[1]
 .find("table")
 .find_all("tr"))
~~~
{: .language-python}
Let's focus now on the first element:
~~~
rows[0]
~~~
{: .language-python}
~~~
<tr>
<td>
<img alt="lc logo" class="flags" height="24" src="https://carpentries.org/assets/img/logos/lc.svg" title="lc workshop" width="24">
</img></td>
<td>
<img alt="us" class="flags" src="https://carpentries.org/assets/img/flags/24/us.png" title="US">
<img alt="globe image" class="flags" src="https://carpentries.org/assets/img/flags/24/w3.png" title="Online">
<a href="https://annajiat.github.io/2021-01-22-uab-NNLM-online">University of Alabama at Birmingham (online)</a>
<br>
<b>Instructors:</b> Annajiat Alim Rasel, Cody Hennesy, Camilla Bressan, Mary Ann Warner
      
      
	</br></img></img></td>
<td>
		Jan 22 - Apr 23, 2021
	</td>
</tr>
~~~
{: .language-html}
We can now split the row 
into three table data elements:
~~~
td0, td1, td2 = rows[0].find_all("td")
~~~
{: .language-python}
If we want the link to the workshop page,
we can look at the `<a>` tag in `td1`,
and specifically at its `href` attribute:
~~~
link = td1.find("a")["href"]
link
~~~
{: .language-python}
~~~
'https://annajiat.github.io/2021-01-22-uab-NNLM-online'
~~~
{: .output}
We can get a list of instructor names 
from the text content of `td1`:
~~~
td1_text_split = td1.text.split("Instructors:")
instructors = [ name.strip() for name in td1_text_split[1].split(",")]
instructors
~~~
{: .language-python}
~~~
# names redacted 
['instructor 1', 'instructor 2', 'instructor 3', 'instructor 4'] 
~~~
{: .output}

> ## A more direct way
> Can we look directly for table elements in the soup?
> How would you do that?
> Would that work?
> > ## Solution
> > We can check how many 
> > `table` elements are in the soup 
> > with
> > ~~~
> > len(soup.find_all("table"))
> > ~~~
> > {: .language-python}
> > We gather that there is only one table in the soup,
> > so that should be the right one! 
> > We can thus use `soup.find("table")` 
> > to reach the right element
> > right away.
> {: .solution}
{: .challenge}

> ## List the workshops 
> Create a list of all the workshops,
> reportin for each one:
> - link
> - location
> - date
> - names of instructors
> 
> > ## Solution
> > ~~~
> > rows = soup.find("table").find_all("tr")
> > def process_row(row):
> >     _,td1,td2 = row.find_all("td")
> >     link = td1.find("a")["href"]
> >     
> >     td1_parts = td1.text.split("Instructors:")
> >     location = td1_parts[0].strip()
> >     instructors_string = td1_parts[1].strip()
> >     instructors = [ n.strip() for n in instructors_string.split(",")]
> >     date = td2.text.strip()
> >     
> >     return dict(
> >        link = link,
> >        location = location,
> >        instructors = instructors,
> >        date = date
> >     ) 
> >     
> > workshops = [ process_row(row) for row in rows ]
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}


> ## Scraping Energy market data
> Look at this page:
> [Energy Market Data](https://www.epexspot.com/en/market-data?market_area=GB&trading_date=2021-03-14&delivery_date=2021-03-15&underlying_year=&modality=Auction&sub_modality=DayAhead&product=60&data_mode=table&period=)
> How would you extract the price of the energy as a function of time?
> > ## Solution
> > TODO
> {: .solution}
{: .challenge}

## Javascript code, the DOM and Selenium

The JavaScript code running on the page 
can actively change the structure of the HTML document.
For some web pages, 
this is an crucial part
of the rendering process:
in some of those cases 
the JavaScript code must be run
to download 
the data you are looking for 
from another URL,
and populate the web page with that data
and any additional element of the page design.

In those cases, BeautifulSoup might not be enough
(as it does not run the JavaScript code on the page),
but you can use the Selenium Web Driver
to load the page in a fully fledged browser 
and automate the interaction with it.
