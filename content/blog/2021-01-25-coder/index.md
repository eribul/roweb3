---
slug: "coder"
title: coder Makes Medical Coding less Messy
package_version: 0.13.6
author:
  - Erik Bülow
date: 2021-01-25
tags:
  - Software Peer Review
  - packages
  - R
  - community
  - coder
  - data-munging
# The summary below will be used by e.g. Twitter cards
description: "A very short summary of your post (~ 100 characters)"
twitterImg: blog/2021-01-25-coder/hex.png
twitterAlt: "Hex sticker for the coder package"
output: 
  html_document:
    keep_md: true
---



<!--html_preserve-->
{{< figure src = "hex.png" width = "150" alt = "Hex sticker for the coder package" class = "pull-right" link = "https://docs.ropensci.org/coder/">}}
<!--/html_preserve-->


A new R-package, `{coder}` has been [developed](https://github.com/ropensci/coder), [peer-reviewed](https://github.com/openjournals/joss-reviews/issues/2916), released by [rOpenSci](https://docs.ropensci.org/coder/), accepted by [CRAN](https://cran.r-project.org/package=coder), and advertised in a paper published by the Journal of Open Source Software [(JOSS)](https://joss.theoj.org/papers/10.21105/joss.02916). In this blog post, I will explain why this package might be useful for (epidemiological/medical/health care related) research.   


## Clinical mess

Once upon a time, in countries not far from ours, there were MDs and nurses making up funny names for any diseases they encountered. What Dr. A called X would be recognized only as Y by his dear colleague Dr. B. X, however, was also a name used by Dr. C, but then for a completely different condition. It was a mess!

Those times are long gone due to the fascinating story of standardized medical nomenclatures and coding. It all started in the first half of the 19th century and has been well documented in a [free on-line book](https://www.cdc.gov/nchs/data/misc/classification_diseases2011.pdf) by Iwao Moryama, Ruth Loy and Alastair Robb-Smith.

Today, when visiting the hospital, our (somatic) conditions are most likely recorded in a medical register by some adaptation of the 10th revision of the International Statistical Classification of Diseases and Related Health Problems (ICD-10). This is a global standard for medical coding administrated by the World Health Organization (WHO). Psychiatric conditions might as well be recorded by the Diagnostic and Statistical Manual of Mental Disorders (DSM). Prescribed medical substances are reported by the Anatomical Therapeutic Chemical (ATC) classification system. Surgical procedures performed in the Nordic countries are coded by the Nordic Medico-Statistical Committee's (NOMESCO's) Classification of Surgical Procedures (NCSP). All combined, there are several hundred thousands of different codes quantifying every possible interaction with health care. Although the mess has significantly decreased, the world of medical coding is still rather messy! Not only are there different versions of ICD (currently with ICD-11 in the pipe). There are also yearly revisions of the currently used version, as well as clinical versions, oncological versions, and several national implementations with different suffixes. All this might be well motivated from a clinical and/or administrative view-point, but it does make things difficult for researchers trying to pool data from different sources, countries and periods. 

Many people have realized this. [Charlson](https://pubmed.ncbi.nlm.nih.gov/3558716/), [Elixhauser](https://pubmed.ncbi.nlm.nih.gov/9431328/), [Sloan](https://pubmed.ncbi.nlm.nih.gov/12773842/) and others, have proposed to combine individual codes into broader categories to capture patient comorbidity more holistically. There are still several versions of each index, however. To use those indices in practice is also cumbersome since many data sources (large administrative registers) are big - bigger than what would fit into the random access memory (RAM) of a standard computer.



## R packages

There are some excellent R-packages to help with both the Charlson and Elixhauser comorbidity indices. [`{comorbidity}`](https://ellessenne.github.io/comorbidity/) by Alessandro Gasparini and [`{icd}`](https://jackwasey.github.io/icd/) by Jack O. Wasey and Michael Lang are very well documented with efficient implementations. To keep up with all different and newly proposed coding versions seems like a daunting, if not impossible task, however. Therefore, the `{coder}` package offers an alternative, more flexible, implementation. It is not hard-wired to Charlson or Elixhauser, although capabilities for those indices are included as well. `{data.table}` is used internally for computational efficiency, while the application programming interface (API) is more inspired by `{tidyverse}`. 



## Regular expressions

In `{coder}`, all relevant codes (from ICD, ATC, NCSP or whatever) are represented in a compact way by their corresponding regular expressions. Peptic ulcer disease for example is recognized as an important comorbidity by the Elixhauser classification. It might be coded by ICD-10 as “K25.x–K28.x” (see [table 2 here](https://journals.lww.com/lww-medicalcare/Abstract/2005/11000/Coding_Algorithms_for_Defining_Comorbidities_in.10.aspx)). This format is well understood by humans; "-" indicates an interval and "x" acts like a wild card for any additional alphanumerical characters. This might be translated into a code list (ignoring the dots): K257, K259, K267, K269, K277, K279, K287, K289. But why those codes only? Why not K250 or K289A? Well, not all codes are used in practice. This list is just an example corresponding to relevant codes from the clinical implementation of ICD-10 (ICD-10-CM) used in 2020 in the USA. The list might look different with data from the Swedish version of ICD-10 (ICD-10-SE) recorded in 2021. Hence, the use of regular expressions will relax the need of a complete and version-specific code list. This also increases the computational speed, which might be crucial if working with large populations (as from a national patient register spanning several decades et cetera). A regular expression for peptic ulcer disease would be `^K2[5-8][79]$` where "^" ("$") marks the beginning (end) of the character string, and where alternative digits are written within brackets (see `?regex` in R for details). 

By default, there are 413 of those regular expressions included in the `{coder}` package. They cover all conditions recognized by Charlson, Elixhauser, RxRisk V, the comorbidity-polypharmacy score (CPS), as well as for some diagnose specific adverse events classifications after hip or knee replacement. Charlson, Elixhauser and Rx Risk V have 6, 5 and 3 alternative regular expressions for each condition. Peptic ulcer disease for example can also be recognized from 1) `53([1-4][79]0)|V1271`; 2) `53([1-4]([4-69]1|7))`; or 3) `53[1-4][79]` based on three different versions of ICD-9-CM. Additional versions (think of the upcoming ICD-11) are relatively easy to implement by the user using a dedicated `classcodes` object (an enhanced data frame/tibble with alternative regular expressions in each column).

Although `classcodes` objects are relatively easy to implement, the interpretation of a regular expression might be hard to grasp intuitively. Or what about hypertension according to [Pratt's implementation](https://bmjopen.bmj.com/content/8/4/e021122) of the Rx Risk V classification based on ATC codes: `C0(2(A(B[0-9]{2}|C0[0-5])|DB(0[2-9]|[1-9][0-9]))|3((A[A-Z][0-9]{2}|BA(0[0-9]|1[01]))|DB(01|99)|C([AB][0-9]{2}|C0[01])|EA01)|9(BA0[2-9]|DA0[2-8]|C[A-X][0-9]{2}))`? There are several functions to aid such interpretation: `visualize()` will open the default web browser to show a structural diagram based on the [JavaScript Regular Expression Visualizer](https://jex.im/regulex). `summary()` and `codebook()` combines the features of `{coder}` with the [`{decoder}`](https://cran.r-project.org/package=decoder) package. By so, it is easy to make explainable code lists with both codes and their interpreted meaning.


## Use case

Assume we have a list of patients who received surgery at specified dates:

```r 
library(coder)
ex_people # Random names and dates included in the package
```

```
# A tibble: 100 x 2
   name              surgery   
   <chr>             <date>    
 1 Chen, Trevor      2021-01-04
 2 Graves, Acineth   2020-09-26
 3 Trujillo, Yanelly 2020-09-13
 4 Simpson, Kenneth  2020-12-16
 5 Chin, Nelson      2020-11-29
 6 Le, Christina     2020-07-03
 7 Kang, Xuan        2020-10-05
 8 Shuemaker, Lauren 2020-07-04
 9 Boucher, Teresa   2020-12-10
10 Le, Soraiya       2020-11-14
# ... with 90 more rows
```

Those patients (among others) were also recorded in a national patient register with date of hospital admissions and corresponding ICD-10 codes. Each patient can have multiple hospital visits over the years. Also, multiple diagnosis might have been recorded at each occasion.

```r 
ex_icd10
```

```
# A tibble: 2,376 x 4
   name                 admission  icd10 hdia 
   <chr>                <date>     <chr> <lgl>
 1 Tran, Kenneth        2020-07-18 S134A FALSE
 2 Tran, Kenneth        2021-01-01 W3319 FALSE
 3 Tran, Kenneth        2020-12-11 Y0262 TRUE 
 4 Tran, Kenneth        2020-11-03 X0488 FALSE
 5 Sommerville, Dominic 2020-12-23 V8104 FALSE
 6 Sommerville, Dominic 2020-08-03 B853  FALSE
 7 Sommerville, Dominic 2020-12-18 Q174  FALSE
 8 Sommerville, Dominic 2020-08-08 A227  FALSE
 9 Sommerville, Dominic 2020-12-13 H702  FALSE
10 Sommerville, Dominic 2020-04-06 X6051 TRUE 
# ... with 2,366 more rows
```

Using those two data sets, as well as a classification scheme (a `classcodes` object), we can easily identify all Charlson comorbidities for each patient:

```r 
ch <- 
  categorize(
    ex_people,                  # patients of interest 
    codedata = ex_icd10,        # Medical codes from national patient register
    cc = "charlson",            # Calculate Charlson comorbidity
    id = "name", code = "icd10" # Specify column names
  )
```

```
Classification based on: icd10
```

Now, how many of those patients were diagnosed with malignancy?

```r 
sum(ch$malignancy)
```

```
[1] 5
```

What is the distribution of the combined comorbidity index for each patient?

```r 
barplot(table(ch$charlson))
```
{{<figure src="unnamed-chunk-5-1.png" >}}

## Further usage

There are many more examples provided at the [`{coder}`  web page](https://docs.ropensci.org/coder/). It is easy to constrain the time frame to comorbidities recorded during only one year prior to surgery, or to 90 days for adverse events thereafter, or to use different versions of the regular expressions etc. Core functionality is provided by the `categorize()` function used above. Alternatively, more fine grained control is provided through a more explicit chain of commands: `codify() %>% classify() %>% index()`. So far, the package has been used in several research projects (cited by the [JOSS article](https://joss.theoj.org/papers/10.21105/joss.02916)). Contributations are most welcome and I would be very happy to [hear about](https://github.com/ropensci/coder/issues) possible additional use cases. And please note that there are no hard-coded limitations to patients and/or medical data. The package can be used with all types of codified data!


## Acknowledgement

An early version of the `{coder}` package was significantly improved after generous contributions made by [Emely C Zabore](https://github.com/zabore) and [David Robinson](/author/david-robinson/) during the peer-review process for rOpenSci. I therefore wish to express my deepest gratitudes to both of them, as well as to the rOpenSci community at large!