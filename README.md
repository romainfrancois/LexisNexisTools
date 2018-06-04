
LexisNexisTools
===============

[![Travis-CI Build Status](https://travis-ci.org/JBGruber/LexisNexisTools.svg?branch=master)](https://travis-ci.org/JBGruber/LexisNexisTools) [![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version-ago/LexisNexisTools)](http://cran.r-project.org/package=LexisNexisTools) [![CRAN\_Download\_Badge](http://cranlogs.r-pkg.org/badges/grand-total/LexisNexisTools)](http://cran.r-project.org/package=LexisNexisTools)

My PhD supervisor once told me that everyone doing newspaper analysis starts by writing code to read in files from the 'LexisNexis' newspaper archive. However, while I do recommend this exercise, not everyone has the time. This package takes TXT files downloaded from 'LexisNexis' in most languages. If you run into any problems or have an idea about a great new feature? The please file an issue report: <https://github.com/JBGruber/LexisNexisTools/issues>.

Installation
------------

Install via:

``` r
install.packages("LexisNexisTools")
```

Or get development version by installing `devtools` first (via `install.packages("devtools")`) then use:

``` r
devtools::install_github("JBGruber/LexisNexisTools")
```

Demo
----

### Load Package

``` r
library("LexisNexisTools")
```

### Set Working Directory to a location containing raw files from 'LexisNexis'.

``` r
# For example
setwd("C:/Test/LNTools_test")

# Or 
setwd("~/Test/LNTools_test")
```

If you do not have files from 'LexisNexis', you can use `lnt_sample()` to copy a sample file with mock data into your current working directory:

``` r
lnt_sample()
```

### Rename Files

'LexisNexis' does not give the .txt files proper names. The function `lnt_rename()` renames files to a standard format: "searchTerm\_startDate-endDate\_documenRange.txt" (e.g., "Obama\_20091201-20100511\_1-500.txt"). Note, that this will not work if your .txt files lack a cover page with this information. Currently, it seems, like 'LexisNexis' only delivers those cover pages when you first create a link to your search ("link to this search" on the results page), follow this link, and then download the txt files from there.

There are three ways in which you can rename the files:

``` r
# Either run lnt_rename() directly in your working directory without the x argument
report.df <- lnt_rename(report = TRUE)

# Or provide a folder path or paths
report.df <- lnt_rename(x = getwd(), report = TRUE)

# Or provide a character object with file names. Use list.files() to search for
# files in a certain path
my_files <- list.files(pattern = ".txt", path = getwd(),
                       full.names = TRUE, recursive = TRUE, ignore.case = TRUE)
report.df <- lnt_rename(x = my_files, report = TRUE)

report.df
```

    ## 
    ##  1 files renamed, in 0.0013 secs

| name\_orig | name\_new                               | status  |
|:-----------|:----------------------------------------|:--------|
| sample.TXT | SampleFile\_20091201-20100511\_1-10.txt | renamed |

Using `list.files()` instead of the built-in mechanism allows you to specify a file pattern. This might be a preferred option if you have a folder in which only some of the .txt files contain newspaper articles from 'LexisNexis' but other files have the ending .txt as well. If you are unsure what the txt files your chosen folder might contain, use the option `simulate = TRUE` (which is the default). The argument `report = TRUE` indicates that the output of the function in `R` will be a data.frame containing a report on which files have been changed on your drive and how.

### Read in 'LexisNexis' Files to Get Meta, Articles and Paragraphs

The main function of this package is `lnt_read()`. It converts the raw text files into three different `data.frames` nested in a special S4 object of class `LNToutput`. The three data.frames contain (1.) the metadata of the articles, (2.) the articles themselves, and (3.) the paragraphs (when `extract_paragraphs = TRUE`).

There are several important keywords that are used to split up the raw articles into article text and metadata. Those need to be provided in some form but can be left to 'auto' to use 'LexisNexis' defaults in several languages. All keywords can be regular expressions and need to be in most cases:

-   `start_keyword`: The English default is "\\d+ of \\d+ DOCUMENTS$" which stands for, for example, "1 of 112 DOCUMENTS". It is used to split up the text in the txt files into individual articles.
-   `end_keyword`: This keyword is used to remove unnecessary information at the end of an article. Usually, this is "^LANGUAGE:". Where the keyword isn't found, the additional information ends up in the article text.
-   `length_keyword`: This keyword, which is usually just "^LENGTH:" (or its equivalent in other languages) finds the information about the length of an article. However, since this is always the last line of the metadata, it is used to separate metadata and article text. There seems to be only one type of cases where this information is missing: if the article consists only of a graphic (which 'LexisNexis' does not retrieve). The final output from `lnt_read()` has a column named `Graphic`, which indicates if this keyword was missing. The article text then contains all metadata as well. In those cases, you should remove the whole article after inspecting it.

<a href="https://ibb.co/fj5YjG"><img src="https://preview.ibb.co/fOfNdb/LN.png" alt="LN" border="0"></a>

To use the function, you can again provide either file name(s), folder name(s) or nothing---to search the current working directory for txt files---as `x` argument:

``` r
LNToutput <- lnt_read(x = getwd())
```

    ## Creating LNToutput from a connection input...
    ##  ...files loaded [0.0012 secs]
    ##  ...articles split [0.0093 secs]
    ##  ...lengths extracted [0.0098 secs]
    ##  ...newspapers extracted [0.01 secs]
    ##  ...dates extracted [0.012 secs]
    ##  ...authors extracted [0.013 secs]
    ##  ...sections extracted [0.013 secs]
    ##  ...editions extracted [0.014 secs]
    ##  ...headlines extracted [0.014 secs]
    ##  ...dates converted [0.018 secs]
    ##  ...metadata extracted [0.019 secs]
    ##  ...article texts extracted [0.021 secs]
    ##  ...paragraphs extracted [0.029 secs]
    ## Elapsed time: 0.033 secs

The returned object of class `LNToutput` can easily be converted to regular data.frames using `@` to select the data.frame you want:

``` r
meta.df <- LNToutput@meta
articles.df <- LNToutput@articles
paragraphs.df <- LNToutput@paragraphs

# Print meta to get an idea of the data
head(meta.df, n = 3)
```

<table style="width:100%;">
<colgroup>
<col width="1%" />
<col width="14%" />
<col width="7%" />
<col width="4%" />
<col width="4%" />
<col width="6%" />
<col width="6%" />
<col width="21%" />
<col width="31%" />
<col width="3%" />
</colgroup>
<thead>
<tr class="header">
<th align="right">ID</th>
<th align="left">Source_File</th>
<th align="left">Newspaper</th>
<th align="left">Date</th>
<th align="left">Length</th>
<th align="left">Section</th>
<th align="left">Author</th>
<th align="left">Edition</th>
<th align="left">Headline</th>
<th align="left">Graphic</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="right">1</td>
<td align="left">SampleFile_20091201-20100511_1-10.txt</td>
<td align="left">Guardian.com</td>
<td align="left">2010-01-11</td>
<td align="left">355 words</td>
<td align="left">NA</td>
<td align="left">Andrew Sparrow</td>
<td align="left"></td>
<td align="left">Lorem ipsum dolor sit amet</td>
<td align="left">FALSE</td>
</tr>
<tr class="even">
<td align="right">2</td>
<td align="left">SampleFile_20091201-20100511_1-10.txt</td>
<td align="left">Guardian</td>
<td align="left">2010-01-11</td>
<td align="left">927 words</td>
<td align="left">NA</td>
<td align="left">Simon Tisdall</td>
<td align="left"></td>
<td align="left">Lorem ipsum dolor sit amet</td>
<td align="left">FALSE</td>
</tr>
<tr class="odd">
<td align="right">3</td>
<td align="left">SampleFile_20091201-20100511_1-10.txt</td>
<td align="left">The Sun (England)</td>
<td align="left">2010-01-11</td>
<td align="left">677 words</td>
<td align="left">FEATURES; Pg. 6</td>
<td align="left">TREVOR Kavanagh</td>
<td align="left">Edition 1; Scotland</td>
<td align="left">Edition 1; Scotland Lorem ipsum dolor sit amet</td>
<td align="left">FALSE</td>
</tr>
</tbody>
</table>

Alternatively, you can convert LNTOutput objects to formats common in other packages using the function `lnt_convert`:

``` r
quanteda_corpus <- lnt_convert(LNToutput, to = "quanteda")
```

See `?lnt_convert` to find out which formats are available or comment in [this issue](https://github.com/JBGruber/LexisNexisTools/issues/2) if you want a format added to the convert function.

### Identify Highly Similar Articles

In 'LexisNexis' itself, there is an option to group highly similar articles. However, experience shows that this feature does not always work perfectly. One common problem when working with 'LexisNexis' data is thus that many articles appear to be delivered twice or more times. While direct duplicates can be filtered out using, for example, `articles.df <- articles.df[!duplicated(articles.df$Article), ]` this does not work for articles with small differences. Hence when one comma or white space is different between two articles, they are normally treated as different.

The function `lnt_similarity()` combines the fast similarity measure from [quanteda](https://github.com/quanteda/quanteda) with the much slower but more accurate relative [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) to compare all articles published on a day. Calculating the Levenshtein distance might be very slow though if you have many articles published each day in your data set. If you think the less accurate similarity measure might be sufficient in your case, simply turn this feature off `rel_dist = FALSE`. The easiest way to use `lnt_similarity()` is to input a `LNToutput` object directly. However, it is also possible to provide texts, dates and IDs separately:

``` r
# Either provide a LNToutput
duplicates.df <- lnt_similarity(LNToutput = LNToutput,
                                rel_dist = FALSE)
```

``` r
# Or the important parts separatley
duplicates.df <- lnt_similarity(texts = LNToutput@articles$Article,
                                dates = LNToutput@meta$Date,
                                IDs = LNToutput@articles$ID,
                                rel_dist = FALSE)
```

    ## 
    Processing date 2010-01-08 ... 0 duplicates found       
    Processing date 2010-01-09 ... 0 duplicates found       
    Processing date 2010-01-10 ... 0 duplicates found       
    Processing date 2010-01-11 ... 8 duplicates found       
    Threshold = 0.99; 4 days processed; 4 duplicates found; in 0.35 secs

Now you can either remove those duplicates from the LNTOutput object:

``` r
#' generate new dataframes without highly similar duplicates
LNToutput@meta <-
  LNToutput@meta[!LNToutput@meta$ID %in% duplicates.df$ID_duplicate, ]
LNToutput@articles <-
  LNToutput@articles[!LNToutput@articles$ID %in% duplicates.df$ID_duplicate, ]
LNToutput@paragraphs <-
  LNToutput@paragraphs[!LNToutput@paragraphs$Art_ID %in% duplicates.df$ID_duplicate, ]
```

Or you directly output the data.frames without the duplicates:

``` r
#' generate new dataframes without highly similar duplicates
meta.df <-
  LNToutput@meta[!LNToutput@meta$ID %in% duplicates.df$ID.duplicate, ]
articles.df <-
  LNToutput@articles[!LNToutput@articles$ID %in% duplicates.df$ID_duplicate, ]
paragraphs.df <-
  LNToutput@paragraphs[!LNToutput@paragraphs$Art_ID %in% duplicates.df$ID_duplicate, ]

# Print e.g., meta to see how the data changed
head(meta.df, n = 3)
```

<table style="width:100%;">
<colgroup>
<col width="1%" />
<col width="1%" />
<col width="41%" />
<col width="6%" />
<col width="5%" />
<col width="5%" />
<col width="4%" />
<col width="8%" />
<col width="4%" />
<col width="16%" />
<col width="4%" />
</colgroup>
<thead>
<tr class="header">
<th align="left"></th>
<th align="right">ID</th>
<th align="left">Source_File</th>
<th align="left">Newspaper</th>
<th align="left">Date</th>
<th align="left">Length</th>
<th align="left">Section</th>
<th align="left">Author</th>
<th align="left">Edition</th>
<th align="left">Headline</th>
<th align="left">Graphic</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">1</td>
<td align="right">1</td>
<td align="left">/home/johannes/Documents/Github 2/LexisNexisTools/SampleFile_20091201-20100511_1-10.txt</td>
<td align="left">Guardian.com</td>
<td align="left">2010-01-11</td>
<td align="left">355 words</td>
<td align="left">NA</td>
<td align="left">Andrew Sparrow</td>
<td align="left"></td>
<td align="left">Lorem ipsum dolor sit amet</td>
<td align="left">FALSE</td>
</tr>
<tr class="even">
<td align="left">2</td>
<td align="right">2</td>
<td align="left">/home/johannes/Documents/Github 2/LexisNexisTools/SampleFile_20091201-20100511_1-10.txt</td>
<td align="left">Guardian</td>
<td align="left">2010-01-11</td>
<td align="left">927 words</td>
<td align="left">NA</td>
<td align="left">Simon Tisdall</td>
<td align="left"></td>
<td align="left">Lorem ipsum dolor sit amet</td>
<td align="left">FALSE</td>
</tr>
<tr class="odd">
<td align="left">7</td>
<td align="right">7</td>
<td align="left">/home/johannes/Documents/Github 2/LexisNexisTools/SampleFile_20091201-20100511_1-10.txt</td>
<td align="left">Guardian</td>
<td align="left">2010-01-08</td>
<td align="left">607 words</td>
<td align="left">NA</td>
<td align="left">Allegra Stratton</td>
<td align="left"></td>
<td align="left">ranch noble ash voice declaration</td>
<td align="left">FALSE</td>
</tr>
</tbody>
</table>

### Issues and questions?

These are the common functions in *LexisNexisTools*. Do you feel like anything is missing from the package or is one of the functions not doing its job? File and issue here to let me know: [issue tracker](https://github.com/JBGruber/LexisNexisTools/issues).
