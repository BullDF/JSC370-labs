Lab 06 - Regular Expressions and Web Scraping
================

# Learning goals

- Use a real world API to make queries and process the data.
- Use regular expressions to parse the information.
- Practice your GitHub skills.

# Lab description

In this lab, we will be working with the [NCBI
API](https://www.ncbi.nlm.nih.gov/home/develop/api/) to make queries and
extract information using XML and regular expressions. For this lab, we
will be using the `httr`, `xml2`, and `stringr` R packages.

This markdown document should be rendered using `github_document`
document ONLY and pushed to your *JSC370-labs* repository in
`lab06/README.md`.

``` r
library(stringr)
library(xml2)
library(httr)
```

## Question 1: How many sars-cov-2 papers?

Build an automatic counter of sars-cov-2 papers using PubMed. You will
need to apply XPath as we did during the lecture to extract the number
of results returned by PubMed in the following web address:

    https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2

Complete the lines of code:

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/div[1]/h3/span")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9|,]+")
```

    ## [1] "218,712"

- How many sars-cov-2 papers are there?

*Answer here.* There are <span class="value">218,712</span> sars-cov-2
papers.

Don’t forget to commit your work!

## Question 2: Academic publications on COVID19 related to Toronto

Use the function `httr::GET()` to make the following query:

1.  Baseline URL:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

2.  Query parameters:

    - db: pubmed
    - term: covid19 toronto
    - retmax: 300

The parameters passed to the query are documented
[here](https://www.ncbi.nlm.nih.gov/books/NBK25499/).

``` r
library(httr)
query_ids <- GET(
  url = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(db = "pubmed", term = "covid19 toronto", retmax = 300)
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

The query will return an XML object, we can turn it into a character
list to analyze the text directly with `as.character()`. Another way of
processing the data could be using lists with the function
`xml2::as_list()`. We will skip the latter for now.

Take a look at the data, and continue with the next question (don’t
forget to commit and push your results to your GitHub repo!).

## Question 3: Get details about the articles

The Ids are wrapped around text in the following way:
`<Id>... id number ...</Id>`. we can use a regular expression that
extract that information. Fill out the following lines of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[0-9]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "[<Id>|</Id>]")
```

With the ids in hand, we can now try to get the abstracts of the papers.
As before, we will need to coerce the contents (results) to a list
using:

1.  Baseline url:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

2.  Query parameters:

    - db: pubmed
    - id: A character with all the ids separated by comma, e.g.,
      “1232131,546464,13131”
    - retmax: 300
    - rettype: abstract

**Pro-tip**: If you want `GET()` to take some element literal, wrap it
around `I()` (as you would do in a formula in R). For example, the text
`"123,456"` is replaced with `"123%2C456"`. If you don’t want that
behavior, you would need to do the following `I("123,456")`.

``` r
publications <- GET(
  url = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db = "pubmed",
    id = I(paste(ids, collapse = ",")),
    retmax = "300",
    rettype = "abstract"
  )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

With this in hand, we can now analyze the data. This is also a good time
for committing and pushing your work!

## Question 4: Distribution of universities, schools, and departments

Using the function `stringr::str_extract_all()` applied on
`publications_txt`, capture all the terms of the form:

1.  University of …
2.  … Institute of …

Write a regular expression that captures all such instances

``` r
institution <- str_extract_all(
  publications_txt,
  "(University of [:alpha:]+)|([:alpha:]+ Institute of [:alpha:]+)"
)
institution <- unlist(institution)
as.data.frame(table(institution))
```

    ##                               institution Freq
    ## 1                 and Institute of Health    1
    ## 2             Caledon Institute of Social    1
    ## 3      California Institute of Technology    2
    ## 4            Canadian Institute of Health    3
    ## 5           Catalan Institute of Oncology    1
    ## 6          Chinese Institute of Engineers    1
    ## 7              CIHR Institute of Genetics    1
    ## 8                CIHR Institute of Health    1
    ## 9       College Institute of Neuroscience    1
    ## 10           Gordon Institute of Business    1
    ## 11      Graduate Institute of Acupuncture    1
    ## 12         Heidelberg Institute of Global    2
    ## 13                In Institute of Network    1
    ## 14             India Institute of Medical    1
    ## 15                 IZA Institute of Labor    1
    ## 16              Knowledge Institute of St    1
    ## 17              Leeds Institute of Health    1
    ## 18  Massachusetts Institute of Technology    3
    ## 19             Meghe Institute of Medical    1
    ## 20        National Institute of Arthritis    1
    ## 21         National Institute of Diabetes    1
    ## 22           National Institute of Health    1
    ## 23           National Institute of Mental    1
    ## 24     National Institute of Neurological    1
    ## 25          National Institute of Science    1
    ## 26      Postgraduate Institute of Medical    1
    ## 27         Research Institute of Genetics    1
    ## 28         Research Institute of Manitoba    1
    ## 29               Research Institute of St    3
    ## 30              Research Institute of the    5
    ## 31          Saveetha Institute of Medical    1
    ## 32            Sinai Institute of Critical    1
    ## 33                the Institute of Health    1
    ## 34                 University of Aberdeen    1
    ## 35                 University of Adelaide    1
    ## 36                  University of alberta    1
    ## 37                  University of Alberta   64
    ## 38                University of Amsterdam    2
    ## 39                University of Antioquia    1
    ## 40                  University of Applied    1
    ## 41                  University of Arizona    4
    ## 42                 University of Auckland    7
    ## 43                University of Barcelona    2
    ## 44                     University of Bari    5
    ## 45                    University of Basel    2
    ## 46                   University of Beirut    1
    ## 47                 University of Belgrade    3
    ## 48                   University of Bergen    1
    ## 49                   University of Berlin    2
    ## 50                     University of Bern    2
    ## 51               University of Birmingham    3
    ## 52                     University of Bonn    1
    ## 53                  University of Brescia   18
    ## 54                  University of Bristol    1
    ## 55                  University of British  115
    ## 56                  University of Calgary   54
    ## 57               University of California   12
    ## 58                University of Cambridge    5
    ## 59                 University of Campinas    1
    ## 60                     University of Cape    1
    ## 61                University of Cartagena    1
    ## 62                  University of Chicago    3
    ## 63                  University of Cologne    4
    ## 64                 University of Colorado    1
    ## 65              University of Connecticut    7
    ## 66               University of Copenhagen    1
    ## 67                     University of Doha    3
    ## 68                  University of Eastern    2
    ## 69                   University of Exeter    1
    ## 70                  University of Florida    1
    ## 71                   University of Foggia    2
    ## 72                   University of Gdansk    1
    ## 73                   University of Geneva    2
    ## 74                  University of Granada    3
    ## 75                University of Groningen    1
    ## 76                   University of Guelph    9
    ## 77                    University of Halle    1
    ## 78                   University of Hawaii    1
    ## 79                   University of Health    1
    ## 80                 University of Helsinki    2
    ## 81            University of Hertfordshire    1
    ## 82                     University of Hong   16
    ## 83                 University of Illinois    6
    ## 84                   University of Kansas    4
    ## 85                     University of Kent    2
    ## 86                  University of Koblenz    2
    ## 87                     University of Kyiv    2
    ## 88                        University of L    1
    ## 89                    University of Leeds    2
    ## 90                University of Ljubljana    1
    ## 91                   University of London    1
    ## 92                 University of Manitoba   21
    ## 93                 University of Mannheim    1
    ## 94                 University of Maryland    1
    ## 95                  University of Mashhad    2
    ## 96                  University of Medical   22
    ## 97                 University of Medicine    2
    ## 98                University of Melbourne    4
    ## 99                  University of Messina    2
    ## 100                  University of Mexico    1
    ## 101                University of Michigan    7
    ## 102                   University of Milan    1
    ## 103                  University of Milano    5
    ## 104                 University of Mineiro    1
    ## 105                  University of Modena    1
    ## 106             University of Montpellier    1
    ## 107                University of Montreal    3
    ## 108                University of Montréal    4
    ## 109                 University of Navarra    1
    ## 110                   University of Negev    1
    ## 111                     University of New    3
    ## 112               University of Newcastle    1
    ## 113            University of Newfoundland    2
    ## 114                 University of Nigeria    1
    ## 115                   University of North    4
    ## 116                University of Northern    1
    ## 117                  University of Norway    1
    ## 118                   University of Notre    1
    ## 119              University of Nottingham    2
    ## 120                 University of Ontario    1
    ## 121                  University of Oregon    2
    ## 122                    University of Oslo    1
    ## 123                  University of Ottawa   61
    ## 124                  University of Oxford   11
    ## 125                   University of Padua    1
    ## 126                 University of Paraiba   13
    ## 127                 University of Pelotas    3
    ## 128            University of Pennsylvania    7
    ## 129              University of Pittsburgh   19
    ## 130                University of Plymouth    1
    ## 131                University of Pretoria    1
    ## 132                  University of Public    1
    ## 133                  University of Punjab    1
    ## 134              University of Queensland    6
    ## 135                  University of Regina    2
    ## 136                     University of Rio    2
    ## 137               University of Rochester    1
    ## 138                    University of Rome    7
    ## 139                     University of São    1
    ## 140            University of Saskatchewan    3
    ## 141                 University of Seville    4
    ## 142              University of Sherbrooke    1
    ## 143               University of Singapore    5
    ## 144                   University of South    1
    ## 145                University of Southern    7
    ## 146                  University of Sydney   46
    ## 147                  University of Tehran    1
    ## 148                   University of Texas    3
    ## 149                     University of the    2
    ## 150               University of Timisoara    3
    ## 151                 University of Toronto  848
    ## 152                  University of Trento    1
    ## 153                    University of Utah    4
    ## 154                 University of Vermont    2
    ## 155                University of Victoria   14
    ## 156                  University of Vienna    3
    ## 157                University of Virginia    1
    ## 158                 University of Waikato    1
    ## 159                  University of Warsaw    2
    ## 160              University of Washington    7
    ## 161                University of Waterloo   16
    ## 162                 University of Western   16
    ## 163                 University of Windsor    1
    ## 164               University of Wisconsin    1
    ## 165           University of Witwatersrand    1
    ## 166               University of Wuppertal    2
    ## 167                    University of York    2
    ## 168                  University of Zurich    4

Repeat the exercise and this time focus on schools and departments in
the form of

1.  School of …
2.  Department of …

And tabulate the results

``` r
schools_and_deps <- str_extract_all(
  publications_txt,
  "(School of [:alpha:]+)|(Department of [:alpha:])"
)
as.data.frame(table(schools_and_deps))
```

    ##            schools_and_deps Freq
    ## 1           Department of A   43
    ## 2           Department of B   38
    ## 3           Department of C   95
    ## 4           Department of D   31
    ## 5           Department of E   89
    ## 6           Department of F  115
    ## 7           Department of G   14
    ## 8           Department of H   82
    ## 9           Department of I   55
    ## 10          Department of J    1
    ## 11          Department of K    3
    ## 12          Department of L   10
    ## 13          Department of M  239
    ## 14          Department of N   62
    ## 15          Department of O   54
    ## 16          Department of P  498
    ## 17          Department of R   34
    ## 18          Department of S   50
    ## 19          Department of T    2
    ## 20          Department of V    2
    ## 21         School of Allied   12
    ## 22          School of Basic   10
    ## 23     School of Biomedical    1
    ## 24       School of Business    5
    ## 25         School of Cancer    2
    ## 26       School of Clinical   10
    ## 27      School of Community    1
    ## 28           School of Data    1
    ## 29      School of Dentistry    6
    ## 30      School of Economics    9
    ## 31    School of Educational    2
    ## 32    School of Engineering    5
    ## 33   School of Epidemiology   11
    ## 34      School of Geography    2
    ## 35         School of Global    6
    ## 36         School of Health   21
    ## 37        School of Hygiene    8
    ## 38  School of International    1
    ## 39     School of Journalism    1
    ## 40    School of Kinesiology    4
    ## 41           School of Life   11
    ## 42     School of Management    6
    ## 43        School of Medical    1
    ## 44       School of Medicine  161
    ## 45        School of Nursing   17
    ## 46   School of Occupational    8
    ## 47       School of Pharmacy    1
    ## 48       School of Physical    5
    ## 49  School of Physiotherapy    1
    ## 50     School of Population   24
    ## 51        School of Primary    2
    ## 52     School of Psychiatry    1
    ## 53     School of Psychology    5
    ## 54         School of Public  175
    ## 55 School of Rehabilitation   14
    ## 56         School of Social    8
    ## 57            School of the    2

## Question 5: Form a database

We want to build a dataset which includes the title and the abstract of
the paper. The title of all records is enclosed by the HTML tag
`ArticleTitle`, and the abstract by `AbstractText`.

Before applying the functions to extract text directly, it will help to
process the XML a bit. We will use the `xml2::xml_children()` function
to keep one element per id. This way, if a paper is missing the
abstract, or something else, we will be able to properly match PUBMED
IDS with their corresponding records.

``` r
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
```

Now, extract the abstract and article title for each one of the elements
of `pub_char_list`. You can either use `sapply()` as we just did, or
simply take advantage of vectorization of `stringr::str_extract`

``` r
abstracts <- str_extract(pub_char_list, "[YOUR REGULAR EXPRESSION]")
abstracts <- str_remove_all(abstracts, "[CLEAN ALL THE HTML TAGS]")
abstracts <- str_remove_all(abstracts, "[CLEAN ALL EXTRA WHITE SPACE AND NEW LINES]")
# alternatively, you can also use str_replace_all(abstracts, "[ORIGINAL]", "[REPLACEMENT]")
```

- How many of these don’t have an abstract?

*Answer here.*

Now, the title

``` r
titles <- str_extract(pub_char_list, "[YOUR REGULAR EXPRESSION]")
titles <- str_remove_all(titles, "[CLEAN ALL THE HTML TAGS]")
```

- How many of these don’t have a title ?

*Answer here.*

Finally, put everything together into a single `data.frame` and use
`knitr::kable` to print the results

``` r
database <- data.frame(
  "[DATA TO CONCATENATE]"
)
knitr::kable(database)

# The table will likely be huge. How can we make the output look better?
# (one idea: kableExtra::scroll_box())
```

Done! Knit the document, commit, and push.

## Final Pro Tip (optional)

You can still share the HTML document on github. You can include a link
in your `README.md` file as the following:

``` md
View [here](https://cdn.jsdelivr.net/gh/:user/:repo@:tag/:file) 
```

For example, if we wanted to add a direct link the HTML page of lecture
6, we could do something like the following:

``` md
View Week 6 Lecture [here]()
```
