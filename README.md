# Elements-of-AI-Building-AI
# Project Title

Final project for the Building AI course

## Summary

The idea is top create a search engine for EU case law which automates legal case law research. Based on the lawyers input, the program compares it with the case law files of the ECJ. It then gives a list of cases similar to the input. Baes on those cases the lawyer can assess his client's legal situation and chances.  

## Background

The program would make research for relevant case law faster and more efficient. EU law is cae law, so in order to build a case, a lawyer has to research cases by the ECJ which is relevant to his client's case. This idea was initially presented at the Rethinking Justice Hackathon in Herleen, NL, in 2018. 

## How is it used?

When a client, who has a legal challenge pertaining EU law, goes to a lawyer, the lawyer will write down the information given by the client. The lawyer - or his para legals - will then proceed to research EU cases similar to the client's case. With the tool, the lawyer would input his notes and the program could compare the notes to ECJ cases and output a list with similar cases. The lawyer then can go on to establish the (legal) relevancy for the client and build a case. The idea is built on the assumption that text (string) similarity is a proxy for text content relevancy.

This is the first draft written in R. This is how far I got:
```
## Text of Rulings (only competition rulings for simplicity)
# Load packages
pack <- c('repmis', 'xml2', 'rvest')

loaded <- base::lapply(pack
                       , base::library
                       , character.only = TRUE)

base::rm(pack, loaded)

# Set working directories
wrkdir <- c('add your working directory')

repmis::set_valid_wd(wrkdir)

base::rm(wrkdir)

## Read list
CompLaw <- utils::read.csv('2019-03-29_CompRulings.csv'
                           , header = TRUE
                           , sep = ','
                           , stringsAsFactors = FALSE
                           , na.strings = c('', 'NA')
                           , dec = '.'
                           , colClasses = c('character', 'character', 'character', 'character')
)

CompLaw$X <- base::as.numeric(x = CompLaw$X)

# Getting text of competition rulings from eur-lex.europa.eu
## Breaking down the pattern of the URL
URL_root <- 'https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?'
URL_lng <- 'from=EN'
URL_celex <- '&uri=CELEX:'

## Base of the URL pattern
URL_base <- base::paste0(URL_root, URL_lng, URL_celex
                         , collapse = NULL)

base::rm(URL_root, URL_lng, URL_celex)

## Get all the URLs to the rulings on eur-lex.europa.eu
CompLaw$URL_base <- URL_base
base::rm(URL_base)
CompLaw$URL <- base::paste0(CompLaw$URL_base, CompLaw$CELEX
                                , collapse = NULL)
CompLaw$URL_base <- NULL

#last_page <- base::as.numeric(base::max(x = CompLaw$X))

URL_list <- base::c(CompLaw$URL
                    , recursive = TRUE
                    , use.names = TRUE)

#URL_list_test <- URL_list[1:3]

## Parse the URLs and write the text into a data frame
df <- base::data.frame(SearchResult = as.character()
                       , stringsAsFactors = FALSE)

## Create an empty html file (UTF-8) in a directory of your choosing
a <- 'C:/Users/UX501/Documents/RethinkingJustice/HTML/HTML.html'

for (i in URL_list) {
  ## Simple but effective progress indicator
  base::cat('.')
  ## Write the url into the html file
  utils::download.file(url = i
                       , destfile = a)
  
  pg <- xml2::read_html(x = a)
  
  temp <- base::data.frame(b <- rvest::html_text(pg)
                           , stringsAsFactors = FALSE)
  
  df <- base::rbind(df, temp)
  
}

CompLawText <- df

base::rm(a, df, temp, i, pg, URL_list, b)

## Rename column of data frame
base::colnames(x = CompLawText) <- 'Text'

## Data frame with the body of the text of each case in a column
CompLaw_new <- base::cbind(CompLaw, CompLawText)

## Similarity of Vocabulary

# Load packages
pack <- c('repmis', 'stringdist')

loaded <- base::lapply(pack
                       , base::library
                       , character.only = TRUE)

base::rm(pack, loaded)

# Set working directories
wrkdir <- c('add your working directory')

repmis::set_valid_wd(wrkdir)

base::rm(wrkdir)

## Read data frame
CompLawText <- utils::read.csv('2019-04-21_CompRulings_Text.csv'
                               , header = TRUE
                               , sep = ','
                               , stringsAsFactors = FALSE
                               , na.strings = c('', 'NA')
                               , dec = '.'
                               , colClasses = c('character', 'character', 'character', 'character', 'character', 'character', 'character')
)

CompLawText$X.1 <- NULL
CompLawText$X <- base::as.numeric(x = CompLawText$X)

## Cosine similarity (Here I use a ECJ case as input text. You yould also use other text input.)
key <- '62017CJ0724'

selecr <- CompLawText[base::grep(pattern = key
                                 , x = CompLawText$CELEX)
                      , ]

selecc <- selecr$Text

CompLawText$keytxt <- selecc

base::rm(key, selecr, selecc)

dist <- stringdist::stringdist(a = CompLawText$keytxt, b = CompLawText$Text
                               , method = 'cosine')

CompLawText$dist <- dist

base::rm(dist)
```

The code is not perfect and far from finished.

## Data sources and AI methods
The cases can be accessed publicly via https://eur-lex.europa.eu/homepage.html?locale=en. 
The first idea was to get text similarity (between lawyers input and the EU case law) by calculating string distance. The text similarity would be given as cosine distance between the lawyer's input and the ECJ cases. The closer the distance the more relevant the ECJ case to the lawyer's initial input. 

## Challenges

The output is a similarity metric. A legal professional still needs to interpret the cases which are similar to the input. Text similarity is a proxy for relevancy. Text similarity does not, however, wholly account for relevancy. For example a case with only a few lines on a certain topic might not have a huge text similarity metric. But the relevancy of the lines for that legal case are huge, and thus, important to consider when comparing a client's case with ECJ case law. Such a tool can automate legal case research but not the (legal) interpretation of the findings. 

## What next?

The project could benefit from a legal profressional who works in/with EU law. This would help to understand the case law research process of lawyers better and how they identify relevant case law for their clients' issues. 
Today, the code should be (re-)written in Python. 

## Acknowledgments

Alexander Koch, with whom I was in the same team at the hackathon in Herleen back in 2018. We did pursue the idea a bit further afterwards.
