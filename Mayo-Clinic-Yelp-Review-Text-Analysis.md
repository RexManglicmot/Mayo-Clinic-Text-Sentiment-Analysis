Mayo Clinic Yelp Review Text Analysis
================
RexManglicmot

-   <a href="#status-working-project"
    id="toc-status-working-project">Status: Working project</a>
-   <a href="#introduction" id="toc-introduction">Introduction</a>
-   <a href="#webscraping-yelp-data"
    id="toc-webscraping-yelp-data">Webscraping Yelp Data</a>
-   <a href="#eda-the-data" id="toc-eda-the-data">EDA the Data</a>

## Status: Working project

1.  Need to fix why the head function is showing up on Rstudio but not
    on the html document
2.  Need to create word clouds
3.  Import and adjust dictionaries
4.  Fix grammar

## Introduction

Every year U.S.News publishes the best hospitals ranked within the U.S.
Although the list does not contain all the hospitals within the U.S., it
contains about ZYX amount. The top ranked hospital for 2022-23 is the
Mayo Clinic based on U.S. News methodology.

To gain a better understanding why the Mayo Clinic is \#1, I decided to
use a Text Analysis on comments made by Yelp reviewers. Thus, this
projects aims to understand user reviews through the Yelp platform,
which is aimed to provide reviews to many businesses, including
healthcare institutions (both private and public).

The metrics used to understand Yelp are as follow

This project is comprised of the following chapters:

1.  Webscraping Yelp Data
2.  Cleaning data
3.  EDA the Data
4.  Visualizations
5.  Plotting Word Cloud
6.  Plotting

## Webscraping Yelp Data

Yelp data was scrapped on XYZ via the [Yelp](https://www.yelp.com/)
website. Within the search engine bar, I typed in “\### *Mayo Clinic*”
and used the first result to scrape the data.

There were 228 reviews in total and the goal was to scrape all 228
reviews containing 4 metrics:

1.  reviewer name
2.  reviewer location
3.  review rating
4.  review text

After various hours debugging the code and looking at html tags to
discern appropiate tags to scrape, I was able to obtain all reviews
within Yelp. Troubleshooting code included unintentionally scraping a
response from a Mayo Clinic official that accrued in more than the 228
reviews.

Nonetheless, below is the code used to scrape data. Mayo_Clinic file is
available within the repository.

``` r
#load libraries
library(tidyverse)
library(rvest)

#create an object to store the webpage address
url <- 'https://www.yelp.com/biz/mayo-clinic-rochester-12?osq=Mayo+clinic'

#convert the url to an html object for R processing
webpage <- read_html(url)

#create object to know page number on the webpage
webpageNum <- webpage %>%
  html_elements(xpath = "//div[@class= ' border-color--default__09f24__NPAKY text-align--center__09f24__fYBGO']") %>%
  html_text() %>%
  str_extract('of.*') %>%
  str_remove('of ') %>%
  as.numeric()

#create a sequence to iterate for page number
webpageSeq <- seq(from = 0, to = (webpageNum * 10)-10, by = 10)

#store items into empty objects
reviewer_name_all = c()
reviewer_location_all = c()
review_rating_all = c()
review_text_all = c()

#create a for loop to get values throughout the 23 pages
for (i in webpageSeq) {
  #need to create an if statement because the 1st page web address
  if (i == 0) {
    webpage <- read_html(url)
    #need to create else because webpage has more content than 1st page web 
    #address
  } else {
    webpage <- read_html(paste0(url, '&start=', i))
  }
  
  #reviewer name
  reviewer_name <- webpage %>%
    #return elements that I specify via xpath method
    #xpath is useful for locating elements
    #// means to search within entire document
    #* means to return any of the elements
    # div means to search within document with the div tags
    html_elements(xpath = "//div[starts-with(@class,' user-passport')]") %>%
    #look within a tag within the previous element
    html_elements(xpath = ".//a[starts-with(@href, '/user_details')]") %>%
    html_text()
  
  #reviewer location
  reviewer_location <- webpage %>%
    #location is within the same div tag, so use same code
    html_elements(xpath = "//div[starts-with(@class,' user-passport')]") %>%
    #location is also located within the span tag
    html_elements(xpath = ".//span[@class= ' css-qgunke']") %>%
    html_text() %>%
    #remove "Location" 
    #pipe remaining values that are not "Location"
    .[. !='Location']
  
  #review rating
  review_rating <- webpage %>%
    html_elements(xpath = "//div[starts-with(@class, ' review')]") %>%
    #within div tag there is an aria-label
    #contains function to look for aria-label that has rating  
    html_elements(xpath = "(.//div[contains(@aria-label, 'star rating')])[1]")%>%
    #ratings are not text, so must use different method and specify which 
    #attribute to obtain
    html_attr('aria-label') %>%
    #remove star rating
    str_remove_all(' star rating') %>%
    #convert into a numeric
    as.numeric()
  
  #review text
  review_text <- webpage %>%
    html_elements(xpath = "//div[starts-with(@class, ' review')]") %>%
    #look throughout webpage with the p tag
    #to get the first comment and not worr about business comment,
    #need to put in brackets
    html_elements(xpath = "(.//p[starts-with(@class, 'comment')])[1]") %>%
    #html_elements(xpath = ".//span[starts-with(@class, ' raw')]") %>%
    html_text()
  
  #appending to appropriate objects
  reviewer_name_all = append(reviewer_name_all, reviewer_name)
  reviewer_location_all = append(reviewer_location_all, reviewer_location)
  review_rating_all = append(review_rating_all, review_rating)
  review_text_all = append(review_text_all, review_text)
  
}

#create a dataframe containing appended values
Mayo_Clinic <- data.frame('name' = reviewer_name_all,
                          'location' = reviewer_location_all,
                          'rating' = review_rating_all,
                          'text'= review_text_all)

#view csv file
#head(Mayo_Clinic)
```

## EDA the Data

Next, I needed to load packages, load data, and run an intital analysis
on the raw data to better.

``` r
#load libraries
#install.packages('tidyverse') #had to re-install for some reason on 11/12/22
#install.packages('ggraph') #had to re-install for some reason on 11/12/22

library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.4.0      ✔ purrr   0.3.5 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.1      ✔ stringr 1.4.1 
    ## ✔ readr   2.1.3      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(tidytext)
library(widyr)
library(RColorBrewer)
library(wordcloud)
library(igraph)
```

    ## 
    ## Attaching package: 'igraph'
    ## 
    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     as_data_frame, groups, union
    ## 
    ## The following objects are masked from 'package:purrr':
    ## 
    ##     compose, simplify
    ## 
    ## The following object is masked from 'package:tidyr':
    ## 
    ##     crossing
    ## 
    ## The following object is masked from 'package:tibble':
    ## 
    ##     as_data_frame
    ## 
    ## The following objects are masked from 'package:stats':
    ## 
    ##     decompose, spectrum
    ## 
    ## The following object is masked from 'package:base':
    ## 
    ##     union

``` r
library(ggraph)
```

``` r
#load Mayo Clinic data
data <- read.csv('Mayo_Clinic.csv')

#View data
#12/8/22 problem
head(data) #why is this not showing up on the html_document?? BUG here
```

    ##   X       name          location rating
    ## 1 1  Stacey C.  Indianapolis, IN      5
    ## 2 2    Arin W.       Chicago, IL      1
    ## 3 3 Jessica S.   Kansas City, MO      5
    ## 4 4    Fran H. San Francisco, CA      1
    ## 5 5   Annie R.        Denver, CO      1
    ## 6 6    Bess L.  Laguna Beach, CA      1
    ##                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             text
    ## 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 I only have good things to say about Mayo.  A family member was diagnosed by their  local doctor with a condition that required surgery to correct.  Several local doctors preform this surgery occasionally buy not routinely so we contacted Mayo where they have a specialist who focuses on this procedure and does over 300 a year.  We transmitted the records and tests electronically, had phone consultations, scheduled a week long visit.  All of the staff was professional, engaging, and seemed to honestly care about their job and the patients.  The cross coordination, communication, and care are so different than typically health care.
    ## 2                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   The clinic might be great. I'll never know because they rejected me via form email that told me to "keep perusing local options" after making me jump through many hoops and having me sign upfor the patient portal which I now have no use for.  Also the person on the phone kept interrupting me and flatly saying "I need something that I can put in as a code".Both the phone calls I had were unsympathetic and frankly a waste. I'd bet money no actual person looked at my paperwork. Just a machine that scanned for keywords. It gets worse. My parents got in easy breezy for Executive Physicals which are extensive but only available to executives and their spouses. Which makes zero sense. (Hint: the answer is elitism)
    ## 3                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                It's hard to put into words the size of this place. Amazing medical center with top notch doctors, nurses, and all the support staff. A collaborative effort from all involved.
    ## 4                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             Received a text saying I missed my appt with my provider. Problem is I haven't even been set up to see a provider. So I called and was not very happy with Missy in General Internal Medicine. A little kindness and an apology for the error, rather than insist it was a phone call from central that I failed to pick up, would have been sufficient. I asked why they didn't leave a VM? "Is there anything else you need?" was Missy's reply. Absolutely not.
    ## 5 I moved here for a surgery. I had a tumor on my adrenal gland, and they couldn't figure out what it was. These tumors are notoriously hard to diagnose. My endocrinologist, Dr. Naan, who is the head of the department, was disrespectful and didn't take my case seriously. The first time I saw her, we did nothing. As we were standing up to leave she tells me that 1/3 of tumors like mine are benign, and she says that like it's a good thing. I stopped and said that this means 2/3 are malignant, and she just says "yup" and continues walking out the door. I was in shock. I'm only 36 y.o., and I had been told previously that it was probably benign. I never thought a doctor, let alone a department head, would be that flippant about telling me I had a 2/3 chance of having cancer. Next session, I told her I was really stressed about that, and I had spent an hour w a friend talking about it. Dr Naan replied "uh oh, I hope you weren't drinking". So, apparently Mayo Clinic department heads think patients have to be drunk to be worried about having a 2/3 chance of cancer. She was beyond dehumanizing. My surgeon, Dr McKenzie, didn't do any education during our surgical consult pertaining to the surgical procedure, what I can expect to happen during surgery, or the risks of surgery. I wrote a review, that was supposed to be confidential, detailing the things he left out. The resident came to see me the day off the surgery and educated me on exactly the things I mentioned in my anonymous review, nothing more, nothing less. He didn't tell me I'd have a catheter placed inside me during the operation, and no one told me after either. I only found out bc I was urinating blood, and because of the pain. Neither my endocrinologist or surgeon made any plans to follow up with me after surgery to discuss what the mass is, and the surgeon wrote explicitly in his notes that a follow up visit was not needed....so no one had plans to follow up and tell me what the mass was. And they didn't. I had some questions that I messaged them, a resident called back to answer my questions, but he never mentioned what the mass was until I asked. Then he said he was going to bring it up. The tumor wasn't in the same place they thought it was, it was actually outside the adrenal gland, and the resident said the adrenal gland would have died anyway, if they had just resected the tumor and not the whole gland. However, the tumor was not even inside the adrenal gland. No one told me that either. I read it in a report, and I brought it up to the resident. The resident said the surgeon didn't bring it up because the surgeon could not differentiate between tumor and regular adrenal gland. This surgeon does many, many adrenal resections each year, he's regarded as highly skilled, and he's does do partial adrenalectomies...so how is it possible that this surgeon could not differentiate between normal adrenal gland and a tumor that's sitting outside but adjacent to the adrenal? The surgeon just didn't want to educate me on anything that a normal patient would want to know -surgery risks, surgery outcomes, etc. I really wanted just the tumor removed, and Dr Naan mentioned that she wasn't sure the tumor was inside the adrenal, but the surgeon never said anything about that. The resident said that no one could have known that the tumor was outside the gland. Between Dr Naan and Dr McKenzie, it's just more dehumanizing bs than I can take. This is hard enough, but add on doctors like them who obviously don't care about my case at all, and the process becomes exponentially worse. I cannot believe I moved out here, to try to get better care, and ended up with doctors like these two who have been so offensive and absent on my case. Choosing Mayo was a huge mistake, and I don't recommend them. Go somewhere else, anywhere else, where they treat people like humans.
    ## 6                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         Seen by resident doctor  in emergency  room. As a retired  medical  professional the treatment  lacked  what I would expect from  a hospital which  claims  to be number 1 in the country.  I believe  that a person  treatment  wouldn't  be any different  at any teaching  hospital in the country.

``` r
#quick check to see if reviewer_name appears more than once
data %>%
  count(name, sort = TRUE)
```

    ##                name n
    ## 1       Courtney C. 2
    ## 2        Jessica D. 2
    ## 3           John M. 2
    ## 4       Michelle F. 2
    ## 5        Phillip H. 2
    ## 6              A S. 1
    ## 7          Aaron J. 1
    ## 8           Alan R. 1
    ## 9       Aleyysha A. 1
    ## 10       Alfonso E. 1
    ## 11        Amanda W. 1
    ## 12           Ame H. 1
    ## 13           Amy R. 1
    ## 14        Angela F. 1
    ## 15          Anne S. 1
    ## 16         Annie R. 1
    ## 17     Anonymous P. 1
    ## 18          Arin W. 1
    ## 19           Art R. 1
    ## 20        Artguy C. 1
    ## 21      Aurielle B. 1
    ## 22        Bailey M. 1
    ## 23      BeachGuy T. 1
    ## 24      Benjamin K. 1
    ## 25          Bess L. 1
    ## 26          Bill S. 1
    ## 27            BK J. 1
    ## 28          Blai L. 1
    ## 29           Bob A. 1
    ## 30           Bob B. 1
    ## 31           Bob C. 1
    ## 32           Bon H. 1
    ## 33          Brad B. 1
    ## 34         Brady L. 1
    ## 35            Ca F. 1
    ## 36       Camille B. 1
    ## 37         Candy P. 1
    ## 38        Cassie H. 1
    ## 39       Charles F. 1
    ## 40        Chelsy M. 1
    ## 41        Cheryl M. 1
    ## 42        Cheryl P. 1
    ## 43         Chris W. 1
    ## 44      Christal S. 1
    ## 45     Christina M. 1
    ## 46         Chuck S. 1
    ## 47         Cindy D. 1
    ## 48         Cindy F. 1
    ## 49       Claudia T. 1
    ## 50       Cynthia E. 1
    ## 51             D R. 1
    ## 52             D W. 1
    ## 53           Dan B. 1
    ## 54        Daniel D. 1
    ## 55          Dave G. 1
    ## 56         David B. 1
    ## 57         David F. 1
    ## 58         David H. 1
    ## 59         David J. 1
    ## 60         David K. 1
    ## 61         David P. 1
    ## 62         David W. 1
    ## 63          Dawn H. 1
    ## 64          Dawn R. 1
    ## 65           Deb J. 1
    ## 66        Debbie F. 1
    ## 67        Debbie S. 1
    ## 68       Deborah W. 1
    ## 69         Debra S. 1
    ## 70           Dee W. 1
    ## 71         Donna H. 1
    ## 72          Doug P. 1
    ## 73          Emma .. 1
    ## 74          Emma H. 1
    ## 75          Eric S. 1
    ## 76         Ettya F. 1
    ## 77           Eva N. 1
    ## 78          Fran H. 1
    ## 79         Frank C. 1
    ## 80       Garrett R. 1
    ## 81          Gena M. 1
    ## 82         Genie H. 1
    ## 83         Geoff T. 1
    ## 84        George K. 1
    ## 85         Glenn W. 1
    ## 86          Greg S. 1
    ## 87         Hasan E. 1
    ## 88         Heidi D. 1
    ## 89         Helen B. 1
    ## 90         Holly T. 1
    ## 91             J G. 1
    ## 92          Jack F. 1
    ## 93          Jack K. 1
    ## 94    Jacqueline C. 1
    ## 95         James E. 1
    ## 96       Jamison T. 1
    ## 97        Janice J. 1
    ## 98       Jasmine B. 1
    ## 99           Jen R. 1
    ## 100        Jenni K. 1
    ## 101       Jennie D. 1
    ## 102     Jennifer K. 1
    ## 103     Jennifer L. 1
    ## 104     Jennifer M. 1
    ## 105      Jessica L. 1
    ## 106      Jessica S. 1
    ## 107      Jessica T. 1
    ## 108       Jessie G. 1
    ## 109          Jim L. 1
    ## 110           Jo M. 1
    ## 111         John D. 1
    ## 112          Jon D. 1
    ## 113          Jon E. 1
    ## 114       Joseph S. 1
    ## 115        Joyce S. 1
    ## 116         Judy M. 1
    ## 117        Julia F. 1
    ## 118        Julie W. 1
    ## 119       Justin A. 1
    ## 120       Justin F. 1
    ## 121          Kal M. 1
    ## 122        Karen A. 1
    ## 123        Karen H. 1
    ## 124          Kat K. 1
    ## 125    Katherine F. 1
    ## 126        Katie N. 1
    ## 127          Ken P. 1
    ## 128          Kim H. 1
    ## 129          Kim W. 1
    ## 130        Kirby S. 1
    ## 131      Koizumi Y. 1
    ## 132         Koko C. 1
    ## 133       Lauren L. 1
    ## 134       Laurie A. 1
    ## 135      Leticia W. 1
    ## 136        Linda P. 1
    ## 137        Linda R. 1
    ## 138      Lindsay D. 1
    ## 139         Lisa E. 1
    ## 140         Lisa V. 1
    ## 141       Lonnie D. 1
    ## 142        Lucas P. 1
    ## 143         Lucy J. 1
    ## 144            M F. 1
    ## 145            M M. 1
    ## 146       Maddie S. 1
    ## 147       Marcia B. 1
    ## 148       Marcie S. 1
    ## 149     Margaret H. 1
    ## 150        Marge S. 1
    ## 151        Marie W. 1
    ## 152         Mary B. 1
    ## 153         Mary S. 1
    ## 154       Meagan J. 1
    ## 155        Megan T. 1
    ## 156        Megan W. 1
    ## 157          Mel M. 1
    ## 158          Mia F. 1
    ## 159      Michael K. 1
    ## 160      Michael R. 1
    ## 161     Michelle A. 1
    ## 162     Michelle W. 1
    ## 163        Minda P. 1
    ## 164      Miranda C. 1
    ## 165           MJ S. 1
    ## 166          mrs r. 1
    ## 167            N S. 1
    ## 168         Neal M. 1
    ## 169       Palash A. 1
    ## 170          pam m. 1
    ## 171     Patricia B. 1
    ## 172     Patricia D. 1
    ## 173     Patricia W. 1
    ## 174      Patrick W. 1
    ## 175         Paul D. 1
    ## 176         Paul M. 1
    ## 177       Philip M. 1
    ## 178       Rachel T. 1
    ## 179       Rachel W. 1
    ## 180      Rebecca G. 1
    ## 181      Rebecca W. 1
    ## 182       Robert P. 1
    ## 183       Roland K. 1
    ## 184         Ryan H. 1
    ## 185           S. F. 1
    ## 186        Sahil K. 1
    ## 187          Sal A. 1
    ## 188        Sandy N. 1
    ## 189        Sandy S. 1
    ## 190        Sarah M. 1
    ## 191        Scott A. 1
    ## 192        Scott H. 1
    ## 193         Shan Z. 1
    ## 194        Shari P. 1
    ## 195       Sharon O. 1
    ## 196       Shelly J. 1
    ## 197       Sheryl M. 1
    ## 198        Sonja G. 1
    ## 199       Stacey C. 1
    ## 200         Stan E. 1
    ## 201         Stan L. 1
    ## 202     Stefanie S. 1
    ## 203    Stephanie M. 1
    ## 204    Stephanie P. 1
    ## 205       Steven J. 1
    ## 206       Stuart W. 1
    ## 207          Sue B. 1
    ## 208          Sun E. 1
    ## 209        Susan E. 1
    ## 210     Susannah H. 1
    ## 211         Tara H. 1
    ## 212         Tash K. 1
    ## 213         Teri D. 1
    ## 214       Terrie M. 1
    ## 215 Thiyagarajan C. 1
    ## 216      Timothy T. 1
    ## 217         Tina B. 1
    ## 218         Toby G. 1
    ## 219        Tyler R. 1
    ## 220          Van T. 1
    ## 221       Vivian Q. 1
    ## 222      William W. 1
    ## 223           Zo H. 1

5 people reviewed twice.

``` r
#quick check to see and sum if any NA are in rating variable
sum(is.na(data$rating))
```

    ## [1] 0

``` r
#calculate summary statistics
summary(data$rating)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    1.00    1.00    5.00    3.39    5.00    5.00

Looking at the summary statistics we see that the median and the max of
the list is 5. What this means is that there are more of the 5 “ratings”
in the list than there are others. The best way to view this, is via the
barchart.

``` r
#create a barchart to count the values
ggplot(data, aes(x=rating)) +
  geom_bar(color='black', fill='steelblue') +
  scale_fill_brewer(palette="Dark2")
```

![](Mayo-Clinic-Yelp-Review-Text-Analysis_files/figure-gfm/EDA%205-1.png)<!-- -->

``` r
#create dataframe to build stacked barchart
rating <- c(1,2,3,4,5)
total <- c((sum(data$rating == 1)),
           (sum(data$rating == 2)),
          (sum(data$rating == 3)),
          (sum(data$rating == 4)),
          (sum(data$rating == 5)))

#create a table of all the rating sums
table3 <-data.frame(rating, total)
print(table3)
```

    ##   rating total
    ## 1      1    72
    ## 2      2    17
    ## 3      3    10
    ## 4      4     8
    ## 5      5   121

The barchart is an tried and true plot used to plot discrete variales.
Here, we see that the ratings from 2 to 4, in terms of count, are
minimal comapred to 1 and 5;with the 5 rating being the most popular
amongnst the reviewers. Next is to see the distribution of such.