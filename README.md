2/13/2022

# Downloading MyAnimeList data for Google Data Analytics Capstone

The scripts and files contained within encompass my capstone project for
the Google Data Analytics Professional Certificate. It combines my
interest in television viewership data and allows such data to be
explored with a visualization. Data will be:

-   Downloaded via a website API
-   Cleaned, transformed, and manipulated into tables that can be used
    for reporting [here](https://github.com/patmendoza330/animelistclean)
-   Loaded onto the kaggle website [here](https://www.kaggle.com/patmendoza/myanimelist-api)
-   Dashboard created via Tableau to explore [here](https://public.tableau.com/app/profile/patrick.mendoza5877/viz/WhatAnimetoWatchNextMyAnimeList/Dashboard)

The MyAnimeList website has a vast repository of ratings and rankings of
viewership data and will be my data source for the project. However,
data could just as easily come from Netflix viewer data, car purchase
preferences, or amazon ratings.

## Necessary steps to access the API

To access the API I needed to setup an account on MyAnimeList and create
an API application in the API panel of your profile. For more
information, see <https://myanimelist.net/forum/?topicid=1973077>.

## First things first - setting some variables

Set your working directory to wherever you’d like in the
`WORKINDIRECTORY` section.

Additionally, I’ll also be loading in my token for accessing the API
(which will be deleted on posting this code). If you need to use this
script to download data from the API, simply enter your clientID in the
`CLIENTIDHERE` section.

``` r
wd1 = "WORKINGDIRECTORY"
setwd(wd1)
clientID <- 'CLIENTIDHERE'
```

## Install any necessary packages

These three packages may take a few minutes to load, but will allow us
to access the API for the MyAnimeList website and carry out some
manipulations of the JSON format so that we can put the data into
dataframes.

Additionally, we’ll use the tidyverse to manipulate/transform data.

``` r
install.packages(c("httr", "jsonlite", "tidyverse"))
```

Next, we want to load the three libraries:

``` r
library(httr)
library(jsonlite)
library(tidyverse)
```

## The MyAnimeList API

While I will briefly touch on some of the specifics of the API, more
information can be found at
<https://myanimelist.net/apiconfig/references/api/v2> - the official API
website and this github page
<https://github.com/SuperMarcus/myanimelist-api-specification>

### The anime list query

The url `https://api.myanimelist.net/v2/anime` allows you to query the
API with a specific string. The parameters are:

-   q - which is the query string.
-   limit - which limits the results.

A completed string would look like:
`https://api.myanimelist.net/v2/anime?q=hi_score&limit=4`.

Using that string, I’m searching on ‘hi\_score’ and limiting results to
4. Lets look at the output of the request and the format:

``` r
res <- GET('https://api.myanimelist.net/v2/anime?q=hi_score&limit=4', add_headers(`X-MAL-CLIENT-ID`= clientID))
stop_for_status(res)
data = fromJSON(rawToChar(res$content))
names(data)
```

    ## [1] "data"   "paging"

``` r
glimpse(data)
```

    ## List of 2
    ##  $ data  :'data.frame':  4 obs. of  1 variable:
    ##   ..$ node:'data.frame': 4 obs. of  3 variables:
    ##   .. ..$ id          : int [1:4] 39570 38422 21877 10958
    ##   .. ..$ title       : chr [1:4] "High Score Girl II" "High Score Girl: Extra Stage" "High Score Girl" "High Score"
    ##   .. ..$ main_picture:'data.frame':  4 obs. of  2 variables:
    ##  $ paging:List of 1
    ##   ..$ next: chr "https://api.myanimelist.net/v2/anime?offset=4&q=hi_score&limit=4"

You may notice that I needed to add my credentials in the `add headers`
section. This needs to be completed for each request.

The output is made up of two elements. The data and paging elements.

Within data there is:

-   node.id - the MyAnimeList (mal) id
-   node.title - title of the anime
-   Pictures in both medium and large format
    -   node.main\_picture.medium
    -   node.main\_picture.large

Paging is made up of the next API request. In this case, since we
limited the number of results to 4, it uses an `offset` in order to
return the next four.

I’ll be utilizing the `flatten` function within the `httr` library to
construct dataframes of subsections of the API call:

``` r
df <- as.data.frame(flatten(data$data))
knitr::kable(df, "pipe") 
```

|    id | title                        | main\_picture.medium                                           | main\_picture.large                                             |
|------:|:-----------------------------|:---------------------------------------------------------------|:----------------------------------------------------------------|
| 39570 | High Score Girl II           | <https://api-cdn.myanimelist.net/images/anime/1560/99904.jpg>  | <https://api-cdn.myanimelist.net/images/anime/1560/99904l.jpg>  |
| 38422 | High Score Girl: Extra Stage | <https://api-cdn.myanimelist.net/images/anime/1057/111384.jpg> | <https://api-cdn.myanimelist.net/images/anime/1057/111384l.jpg> |
| 21877 | High Score Girl              | <https://api-cdn.myanimelist.net/images/anime/1668/91345.jpg>  | <https://api-cdn.myanimelist.net/images/anime/1668/91345l.jpg>  |
| 10958 | High Score                   | <https://api-cdn.myanimelist.net/images/anime/6/36265.jpg>     | <https://api-cdn.myanimelist.net/images/anime/6/36265l.jpg>     |

### The anime details

The same API call (as above) also allows you to query on a specific
anime as long as you have the identifier to incorporate into the request
(e.g. `https://api.myanimelist.net/v2/anime/21877`).

To obtain additional information, you need to use the `fields`
parameter.

``` r
res <- GET('https://api.myanimelist.net/v2/anime/21877?fields=id,title,main_picture,alternative_titles,start_date,end_date,synopsis,mean,rank,popularity,num_list_users,num_scoring_users,nsfw,created_at,updated_at,media_type,status,genres,num_episodes,start_season,broadcast,source,average_episode_duration,rating,pictures,background,related_anime,related_manga,recommendations,studios,statistics', add_headers(`X-MAL-CLIENT-ID`= clientID))
stop_for_status(res)
data = fromJSON(rawToChar(res$content))
names(data)
```

    ##  [1] "id"                       "title"                   
    ##  [3] "main_picture"             "alternative_titles"      
    ##  [5] "start_date"               "end_date"                
    ##  [7] "synopsis"                 "mean"                    
    ##  [9] "rank"                     "popularity"              
    ## [11] "num_list_users"           "num_scoring_users"       
    ## [13] "nsfw"                     "created_at"              
    ## [15] "updated_at"               "media_type"              
    ## [17] "status"                   "genres"                  
    ## [19] "num_episodes"             "start_season"            
    ## [21] "broadcast"                "source"                  
    ## [23] "average_episode_duration" "rating"                  
    ## [25] "pictures"                 "background"              
    ## [27] "related_anime"            "related_manga"           
    ## [29] "recommendations"          "studios"                 
    ## [31] "statistics"

``` r
glimpse(data)
```

    ## List of 31
    ##  $ id                      : int 21877
    ##  $ title                   : chr "High Score Girl"
    ##  $ main_picture            :List of 2
    ##   ..$ medium: chr "https://api-cdn.myanimelist.net/images/anime/1668/91345.jpg"
    ##   ..$ large : chr "https://api-cdn.myanimelist.net/images/anime/1668/91345l.jpg"
    ##  $ alternative_titles      :List of 3
    ##   ..$ synonyms: list()
    ##   ..$ en      : chr "Hi Score Girl"
    ##   ..$ ja      : chr "<U+30CF><U+30A4><U+30B9><U+30B3><U+30A2><U+30AC><U+30FC><U+30EB>"
    ##  $ start_date              : chr "2018-07-14"
    ##  $ end_date                : chr "2018-09-29"
    ##  $ synopsis                : chr "The year is 1991, and arcade video games are the latest craze. Becoming a professional gamer is a far-fetched d"| __truncated__
    ##  $ mean                    : num 7.8
    ##  $ rank                    : int 877
    ##  $ popularity              : int 869
    ##  $ num_list_users          : int 215304
    ##  $ num_scoring_users       : int 108656
    ##  $ nsfw                    : chr "white"
    ##  $ created_at              : chr "2013-12-31T08:49:46+00:00"
    ##  $ updated_at              : chr "2022-02-14T19:41:44+00:00"
    ##  $ media_type              : chr "tv"
    ##  $ status                  : chr "finished_airing"
    ##  $ genres                  :'data.frame':    5 obs. of  2 variables:
    ##   ..$ id  : int [1:5] 4 11 22 23 42
    ##   ..$ name: chr [1:5] "Comedy" "Game" "Romance" "School" ...
    ##  $ num_episodes            : int 12
    ##  $ start_season            :List of 2
    ##   ..$ year  : int 2018
    ##   ..$ season: chr "summer"
    ##  $ broadcast               :List of 2
    ##   ..$ day_of_the_week: chr "saturday"
    ##   ..$ start_time     : chr "00:30"
    ##  $ source                  : chr "manga"
    ##  $ average_episode_duration: int 1470
    ##  $ rating                  : chr "pg_13"
    ##  $ pictures                :'data.frame':    3 obs. of  2 variables:
    ##   ..$ medium: chr [1:3] "https://api-cdn.myanimelist.net/images/anime/8/57555.jpg" "https://api-cdn.myanimelist.net/images/anime/1668/91345.jpg" "https://api-cdn.myanimelist.net/images/anime/1252/93554.jpg"
    ##   ..$ large : chr [1:3] "https://api-cdn.myanimelist.net/images/anime/8/57555l.jpg" "https://api-cdn.myanimelist.net/images/anime/1668/91345l.jpg" "https://api-cdn.myanimelist.net/images/anime/1252/93554l.jpg"
    ##  $ background              : chr ""
    ##  $ related_anime           :'data.frame':    1 obs. of  3 variables:
    ##   ..$ node                   :'data.frame':  1 obs. of  3 variables:
    ##   .. ..$ id          : int 38422
    ##   .. ..$ title       : chr "High Score Girl: Extra Stage"
    ##   .. ..$ main_picture:'data.frame':  1 obs. of  2 variables:
    ##   ..$ relation_type          : chr "sequel"
    ##   ..$ relation_type_formatted: chr "Sequel"
    ##  $ related_manga           : list()
    ##  $ recommendations         :'data.frame':    10 obs. of  2 variables:
    ##   ..$ node               :'data.frame':  10 obs. of  3 variables:
    ##   .. ..$ id          : int [1:10] 34280 14741 35860 12467 35968 47257 14813 1596 48926 4224
    ##   .. ..$ title       : chr [1:10] "Gamers!" "Chuunibyou demo Koi ga Shitai!" "Karakai Jouzu no Takagi-san" "Nazo no Kanojo X" ...
    ##   .. ..$ main_picture:'data.frame':  10 obs. of  2 variables:
    ##   ..$ num_recommendations: int [1:10] 7 6 4 2 2 2 2 2 2 1
    ##  $ studios                 :'data.frame':    1 obs. of  2 variables:
    ##   ..$ id  : int 7
    ##   ..$ name: chr "J.C.Staff"
    ##  $ statistics              :List of 2
    ##   ..$ status        :List of 5
    ##   .. ..$ watching     : chr "11196"
    ##   .. ..$ completed    : chr "133937"
    ##   .. ..$ on_hold      : chr "5321"
    ##   .. ..$ dropped      : chr "8614"
    ##   .. ..$ plan_to_watch: chr "57215"
    ##   ..$ num_list_users: int 216283

The data obtained incorporates all of the fields that I’ve listed in the
`fields` section.

You’ll notice that this response is made up of lists of lists. I can’t
create a flattened data frame at this point because the lists are made
up of differing sizes.

``` r
as.data.frame(flatten(data))
```

    ## Error in (function (..., row.names = NULL, check.rows = FALSE, check.names = TRUE, : arguments imply differing number of rows: 1, 0, 5, 3, 10

Because of this, I’ll create tables off of some sections of the call,
then set those fields to `NULL,` so that I can create a dataframe on the
remaining fields (this will make more sense below).

I’ll use this API request to obtain the bulk of the information that
will be used to build tables.

***Please note:*** the ID for the anime must be known to use this
request. This leads me to:

### The anime ranking

This API request will allow you to obtain a list of anime identifiers
and other details (including rank) for the following categories:

-   all
-   airing
-   upcoming
-   tv
-   ova
-   movie
-   special
-   bypopularity
-   favorite

Here is a sample of a request and the output:

``` r
res <- GET('https://api.myanimelist.net/v2/anime/ranking?ranking_type=all&limit=4', add_headers(`X-MAL-CLIENT-ID`= clientID))
stop_for_status(res)
data = fromJSON(rawToChar(res$content))
names(data)
```

    ## [1] "data"   "paging"

``` r
str(data$data)
```

    ## 'data.frame':    4 obs. of  2 variables:
    ##  $ node   :'data.frame': 4 obs. of  3 variables:
    ##   ..$ id          : int  5114 48583 38524 9253
    ##   ..$ title       : chr  "Fullmetal Alchemist: Brotherhood" "Shingeki no Kyojin: The Final Season Part 2" "Shingeki no Kyojin Season 3 Part 2" "Steins;Gate"
    ##   ..$ main_picture:'data.frame': 4 obs. of  2 variables:
    ##   .. ..$ medium: chr  "https://api-cdn.myanimelist.net/images/anime/1223/96541.jpg" "https://api-cdn.myanimelist.net/images/anime/1948/120625.jpg" "https://api-cdn.myanimelist.net/images/anime/1517/100633.jpg" "https://api-cdn.myanimelist.net/images/anime/5/73199.jpg"
    ##   .. ..$ large : chr  "https://api-cdn.myanimelist.net/images/anime/1223/96541l.jpg" "https://api-cdn.myanimelist.net/images/anime/1948/120625l.jpg" "https://api-cdn.myanimelist.net/images/anime/1517/100633l.jpg" "https://api-cdn.myanimelist.net/images/anime/5/73199l.jpg"
    ##  $ ranking:'data.frame': 4 obs. of  1 variable:
    ##   ..$ rank: int  1 2 3 4

``` r
knitr::kable(as.data.frame(flatten(data$data)), "pipe")
```

|    id | title                                       | main\_picture.medium                                           | main\_picture.large                                             | rank |
|------:|:--------------------------------------------|:---------------------------------------------------------------|:----------------------------------------------------------------|-----:|
|  5114 | Fullmetal Alchemist: Brotherhood            | <https://api-cdn.myanimelist.net/images/anime/1223/96541.jpg>  | <https://api-cdn.myanimelist.net/images/anime/1223/96541l.jpg>  |    1 |
| 48583 | Shingeki no Kyojin: The Final Season Part 2 | <https://api-cdn.myanimelist.net/images/anime/1948/120625.jpg> | <https://api-cdn.myanimelist.net/images/anime/1948/120625l.jpg> |    2 |
| 38524 | Shingeki no Kyojin Season 3 Part 2          | <https://api-cdn.myanimelist.net/images/anime/1517/100633.jpg> | <https://api-cdn.myanimelist.net/images/anime/1517/100633l.jpg> |    3 |
|  9253 | Steins;Gate                                 | <https://api-cdn.myanimelist.net/images/anime/5/73199.jpg>     | <https://api-cdn.myanimelist.net/images/anime/5/73199l.jpg>     |    4 |

As you can see, the format for this response allows creation of a
dataframe. This is because the lists are all of the same size. It also
will be used as the basis for obtaining the list of all of the IDs to
pull detailed data.

## Code for downloading the dataset

### Obtaining the anime ranking

The API retrieves values from a point in time. Since I plan on
maintaining this somewhat frequently (possibly as often as every two
weeks), I will incorporate a time field setup as below:

``` r
tm_ky_de <- '2/11/22' # MANUAL ADJUSTMENT EACH LOAD
tm_ky <- 2 # MANUAL ADJUSTMENT EACH LOAD
tm_entry <- data.frame(tm_ky = tm_ky, tm_ky_de = tm_ky_de, stringsAsFactors = FALSE)
```

Right now this incorporates hard-coded dates, I will update this in the
future to generate these automatically.

Next I will pull the ranking information into a table. Because this
table will download relatively fast, I will download all categories of
the ranking system by looping through each (even though I am only
interestd in “tv”).

``` r
ranking_types <- c('all', 'airing', 'upcoming', 'tv', 'ova', 'movie', 'special', 'bypopularity', 'favorite')
```

This list comprises all of the ranking types that I will cycle through.

This API call has a limit of 500 listings per request, as a result, I
will not only loop through the categories, but also the paging of each
category.

``` r
for (items in ranking_types){ # I've broken this loop into the first call for the category, then it will cycle through each of the subsequent page requests before reaching the end, then it will cycle onto the next category.
  get_call <- paste0('https://api.myanimelist.net/v2/anime/ranking?ranking_type=',items, '&limit=500')
  res <- GET(get_call, add_headers(`X-MAL-CLIENT-ID`= clientID))
  stop_for_status(res)
  data = fromJSON(rawToChar(res$content))
  temp_load <- flatten(data$data)
  temp_load$main_picture <- NULL # I won't obtain the pictures for this round, I'll grab those in the API call where I grab all of the other attributes for the anime.
  temp_load <- as.data.frame(temp_load)
  temp_load$rank_category <- items   # to be able to differentiate between the categories an additional column will be added that lists the category
  if (exists('rank_table')){ # if the rank table already exists, I'll bind a new row, otherwise the variable will be initialized.
    rank_table <- rbind(rank_table, temp_load)
  } else {
    rank_table <- temp_load
  }
  temp_load <- NULL
  i = 0
  # This will cycle through all of the subsequent pages of the API call until there are no more for the category
  while (!is.null(data$paging[["next"]])){
    i = i + 500
    print(i)
    res <- GET(data$paging[["next"]], add_headers(`X-MAL-CLIENT-ID`= clientID))
    data = fromJSON(rawToChar(res$content))
    temp_load <- flatten(data$data)
    temp_load$main_picture <- NULL
    temp_load <- as.data.frame(temp_load)
    temp_load$rank_category <- items
    rank_table <- rbind(rank_table, temp_load)
    temp_load <- NULL
  }
  print(paste('The calls for the', items, 'have completed'))
}

# add the time key to the data frame
rank_table <- cbind(tm_ky = tm_ky, rank_table)
```

Here are the first few lines from the table constructed:

``` r
knitr::kable(head(rank_table), "pipe")
```

| tm\_ky |    id | title                                       | rank | rank\_category |
|-------:|------:|:--------------------------------------------|-----:|:---------------|
|      2 |  5114 | Fullmetal Alchemist: Brotherhood            |    1 | all            |
|      2 | 48583 | Shingeki no Kyojin: The Final Season Part 2 |    2 | all            |
|      2 | 38524 | Shingeki no Kyojin Season 3 Part 2          |    3 | all            |
|      2 |  9253 | Steins;Gate                                 |    4 | all            |
|      2 | 28977 | Gintama°                                    |    5 | all            |
|      2 | 42938 | Fruits Basket: The Final                    |    6 | all            |

### Obtaining detailed information

First, I need the list of IDs that correspond with my area of interest,
“tv.”

``` r
mal_id_to_pull <- rank_table %>%
  filter(rank_category == 'tv') %>%
  distinct(id)
```

Next, I’ll construct a list of the fields that I want to pull from the
API. These fields will be the basis for the database.

``` r
fields_to_pull <- c('id', 'title','alternative_titles', 'main_picture', 'start_date', 'end_date', 'synopsis', 'mean', 'media_type','media_type', 'status', 'num_episodes', 'start_season', 'rating', 'studios', 'nsfw','genres', 'rank', 'popularity', 'num_scoring_users', 'statistics')
```

There are five initial tables that I will be building. They are:

1.  `anime_table` - information about the anime (e.g. title)
2.  `anime_syn_table` - synonyms for titles
3.  `anime_genres_table` - genres that are associated with the title
4.  `anime_studios_table` - studios that produced the title
5.  `anime_ranking_table` - ranking information

Below are three caveats to the data that gets loaded. I’ll build these
criteria into `if` statements so that only data that meets these
conditions gets loaded:

-   The title has a mean - this is the score on a scale of 0-10. The
    score for the anime is one of the building blocks of my
    visualization, so I don’t want to include anything that doesn’t have
    one.
-   The title has a rank - this is the method that MyAnimeList uses to
    ensure that non-adult titles do not make their way into the
    rankings. I’d like my application to have a wide audience so I will
    use this condition as well as the next criteria to ensure that I
    exclude adult-oriented material.
-   The rating is not `r+` - to exclude adult-oriented material.

To create the five tables above, I need to cycle through each of their
IDs, request the information through the API, load each record as a row,
then move onto the next ID.

*Please note*: I’ve built a break into the code below so that only the
first 400 IDs are pulled in. If running for the entire list you will
need to remove that if statement at the end of the loop.

``` r
i <- 0
for (j in 1:nrow(mal_id_to_pull)) { # to cycle through all of my IDs
  get_call <- paste0('https://api.myanimelist.net/v2/anime/', mal_id_to_pull[j,],'?fields=', paste(fields_to_pull,collapse = ","))
  res <- GET(get_call, add_headers(`X-MAL-CLIENT-ID`= clientID))
  stop_for_status(res)
  data = fromJSON(rawToChar(res$content))
  if (!is.null(data$mean) & !is.null(data$rank)) { # the two conditions that I mentioned above
    if (is.null(data$rating)) { # this is the last condition mentioned above
      rating ="N/A"
    } else {
      rating = data$rating
    }
    if (rating != "r+") {
      # Load the SYNONYMS table
      if (length(data$alternative_titles$synonyms)!=0){
        syn_table_load <- as.data.frame(cbind(tm_ky = tm_ky, id = data$id, synonyms = data$alternative_titles$synonyms), stringsAsFactors = FALSE)
        if (exists('anime_syn_table')){
          anime_syn_table <- dplyr::bind_rows(anime_syn_table, syn_table_load)
          } else {
          anime_syn_table <- syn_table_load
          }
        syn_table_load <- NULL
      }
      data$alternative_titles$synonyms <- NULL # this is the first case where I need to set a field to NULL so that I can create a dataframe based of remaining fields in the request
      # Load the GENRES table
      if (length(data$genres$id)!=0){
        genres_table_load <- as.data.frame(cbind(tm_ky = tm_ky, id = data$id, genres_id = data$genres$id, genres_de = data$genres$name), stringsAsFactors = FALSE)
        if (exists('anime_genres_table')){
          anime_genres_table <- dplyr::bind_rows(anime_genres_table, genres_table_load)
          } else {
          anime_genres_table <- genres_table_load
          }
        genres_table_load <- NULL
      }
      data$genres <- NULL
      # Load the STUDIOS table
      if (length(data$studios$id)!=0){
        studios_table_load <- as.data.frame(cbind(tm_ky = tm_ky, id = data$id, studio_id = data$studios$id, studio_de = data$studios$name), stringsAsFactors = FALSE)
        if (exists('anime_studios_table')){
          anime_studios_table <- dplyr::bind_rows(anime_studios_table, studios_table_load)
          } else {
          anime_studios_table <- studios_table_load
          }
        studios_table_load <- NULL
      }
      data$studios <- NULL
      # Load the RANKING table, mean, rank, popularity, num_scoring_users, statistics
      if (length(data$statistics$num_list_users)!=0){
        ranking_table_load <- as.data.frame(cbind(tm_ky = tm_ky, id = data$id, mean = data$mean, rank = data$rank, popularity = data$popularity, num_scoring_users = data$num_scoring_users, statistics.watching = data$statistics$status$watching, statistics.completed = data$statistics$status$completed, statistics.on_hold = data$statistics$status$on_hold, statistics.dropped = data$statistics$status$dropped, statistics.plan_to_watch = data$statistics$status$plan_to_watch, statistics.num_scoring_users = data$statistics$num_list_users), stringsAsFactors = FALSE)
        if (exists('anime_ranking_table')){
          anime_ranking_table <- dplyr::bind_rows(anime_ranking_table, ranking_table_load)
          } else {
          anime_ranking_table <- ranking_table_load
          }
        ranking_table_load <- NULL
      }
      data$statistics <- NULL
      data$mean <- NULL
      data$rank <- NULL
      data$popularity <- NULL
      data$num_scoring_users <- NULL
      # Load the anime table
      for (k in 1:length(data)) { #cycles through each item in data and checks to see if its empty, if so, it will set it to NULL
        if (k <= length(data)) { # need to add this because if I delete columns my original loop will extend beyond boundaries
          if (length(data[[k]])==0) {
            data[k] <- NULL
          }
        }
      }
      temp_load <- as.data.frame(data, stringsAsFactors = FALSE)
      if (exists('anime_table')) {
        anime_table <- dplyr::bind_rows(anime_table, temp_load)
      } else {
        anime_table <- temp_load
      }
      temp_load <- NULL
    }
  }
  i = i + 1
  if (i %% 500 == 0) { # to print to the console the status of every 500 requests
    print(paste('Finished loading', i, 'items'))
    print("Waiting for 5 minutes before proceeding") # I've built in a pause into the requests, otherwise, the server will deny requests
    Sys.sleep(60)
    print("4 minutes remaining")
    Sys.sleep(60)
    print("3 minutes remaining")
    Sys.sleep(60)
    print("2 minutes remaining")
    Sys.sleep(60)
    print("1 minute remaining")
    Sys.sleep(60)
    print("Starting up again")
  }
  if (j == 400) { # REMOVE THIS IF RUNNING FOR ENTIRE LIST
    break
  }
}
print("done collecting detailed information")
anime_table <- cbind(tm_ky = tm_ky, anime_table) %>%
  mutate(across(c(tm_ky, id), as.character)) # alll other tables come across as characters in these two fields. Since I'm building additional tables off of these tables, I'll need to temporarily convert them into character fields.
print("Job done!")
```

### Additional tables created and modifications to existing tables

I’d like to make the tables smaller if possible, so I’m going to create
some lookup tables for the `anime_genres_table` and the
`anime_studios_table`

``` r
genres_l <- anime_genres_table %>%
  distinct(tm_ky, genres_id, genres_de) %>%
  arrange(genres_id)
knitr::kable(head(genres_l), "pipe")
```

| tm\_ky | genres\_id | genres\_de |
|:-------|:-----------|:-----------|
| 2      | 1          | Action     |
| 2      | 10         | Fantasy    |
| 2      | 11         | Game       |
| 2      | 13         | Historical |
| 2      | 14         | Horror     |
| 2      | 15         | Kids       |

This looks good, we can remove the description from the
anime\_genres\_table now:

``` r
anime_genres_table <- anime_genres_table %>%
  select(tm_ky, id, genres_id) %>%
  arrange(tm_ky, id, genres_id)
knitr::kable(head(anime_genres_table))
```

| tm\_ky | id  | genres\_id |
|:-------|:----|:-----------|
| 2      | 1   | 1          |
| 2      | 1   | 2          |
| 2      | 1   | 24         |
| 2      | 1   | 29         |
| 2      | 1   | 4          |
| 2      | 1   | 8          |

I want to make a studios lookup, similar to the genres:

``` r
studios_l <- anime_studios_table %>%
  distinct(tm_ky, studio_id, studio_de) %>%
  arrange(tm_ky, studio_id)
knitr::kable(head(studios_l))
```

| tm\_ky | studio\_id | studio\_de           |
|:-------|:-----------|:---------------------|
| 2      | 1          | Studio Pierrot       |
| 2      | 10         | Production I.G       |
| 2      | 101        | Studio Hibari        |
| 2      | 103        | Tatsunoko Production |
| 2      | 1075       | C-Station            |
| 2      | 11         | Madhouse             |

Now I need to remove the description from the anime\_studios\_table

``` r
anime_studios_table <- anime_studios_table %>%
  select(tm_ky, id,studio_id) %>%
  arrange(tm_ky, id, studio_id)
knitr::kable(head(anime_studios_table))
```

| tm\_ky | id    | studio\_id |
|:-------|:------|:-----------|
| 2      | 1     | 14         |
| 2      | 10030 | 7          |
| 2      | 10049 | 37         |
| 2      | 10087 | 43         |
| 2      | 10162 | 10         |
| 2      | 10165 | 2          |

At this point, I have created the five main tables as well as their
lookup tables. An enhancement that I’d like to build in is having a
demographic table.

Demographics (Kids, Shoujo, Shounen, Seinen, Josei) have been built into
the genres field.

See below for examples of genres\_de that contains a demographic
element:

``` r
anime_table %>% 
  left_join(anime_genres_table) %>% 
  inner_join(genres_l) %>%
  filter(genres_de == "Shounen") %>%
  select(!synopsis) %>%
  head() %>%
  knitr::kable("pipe")
```

    ## Joining, by = c("tm_ky", "id")

    ## Joining, by = c("tm_ky", "genres_id")

| tm\_ky | id    | title                                       | main\_picture.medium                                           | main\_picture.large                                             | alternative\_titles.en                   | alternative\_titles.ja                                                                                                           | start\_date | end\_date  | media\_type | status            | num\_episodes | start\_season.year | start\_season.season | rating | nsfw  | genres\_id | genres\_de |
|:-------|:------|:--------------------------------------------|:---------------------------------------------------------------|:----------------------------------------------------------------|:-----------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------|:------------|:-----------|:------------|:------------------|--------------:|-------------------:|:---------------------|:-------|:------|:-----------|:-----------|
| 2      | 5114  | Fullmetal Alchemist: Brotherhood            | <https://api-cdn.myanimelist.net/images/anime/1223/96541.jpg>  | <https://api-cdn.myanimelist.net/images/anime/1223/96541l.jpg>  | Fullmetal Alchemist: Brotherhood         | &lt;U+92FC&gt;&lt;U+306E&gt;&lt;U+932C&gt;&lt;U+91D1&gt;&lt;U+8853&gt;&lt;U+5E2B&gt; FULLMETAL ALCHEMIST                         | 2009-04-05  | 2010-07-04 | tv          | finished\_airing  |            64 |               2009 | spring               | r      | white | 27         | Shounen    |
| 2      | 48583 | Shingeki no Kyojin: The Final Season Part 2 | <https://api-cdn.myanimelist.net/images/anime/1948/120625.jpg> | <https://api-cdn.myanimelist.net/images/anime/1948/120625l.jpg> | Attack on Titan: The Final Season Part 2 | &lt;U+9032&gt;&lt;U+6483&gt;&lt;U+306E&gt;&lt;U+5DE8&gt;&lt;U+4EBA&gt; The Final Season Part 2                                   | 2022-01-10  | NA         | tv          | currently\_airing |             0 |               2022 | winter               | r      | white | 27         | Shounen    |
| 2      | 38524 | Shingeki no Kyojin Season 3 Part 2          | <https://api-cdn.myanimelist.net/images/anime/1517/100633.jpg> | <https://api-cdn.myanimelist.net/images/anime/1517/100633l.jpg> | Attack on Titan Season 3 Part 2          | &lt;U+9032&gt;&lt;U+6483&gt;&lt;U+306E&gt;&lt;U+5DE8&gt;&lt;U+4EBA&gt; Season3 Part.2                                            | 2019-04-29  | 2019-07-01 | tv          | finished\_airing  |            10 |               2019 | spring               | r      | white | 27         | Shounen    |
| 2      | 28977 | Gintama°                                    | <https://api-cdn.myanimelist.net/images/anime/3/72078.jpg>     | <https://api-cdn.myanimelist.net/images/anime/3/72078l.jpg>     | Gintama Season 4                         | &lt;U+9280&gt;&lt;U+9B42&gt;°                                                                                                    | 2015-04-08  | 2016-03-30 | tv          | finished\_airing  |            51 |               2015 | spring               | pg\_13 | white | 27         | Shounen    |
| 2      | 9969  | Gintama’                                    | <https://api-cdn.myanimelist.net/images/anime/4/50361.jpg>     | <https://api-cdn.myanimelist.net/images/anime/4/50361l.jpg>     | Gintama Season 2                         | &lt;U+9280&gt;&lt;U+9B42&gt;’                                                                                                    | 2011-04-04  | 2012-03-26 | tv          | finished\_airing  |            51 |               2011 | spring               | pg\_13 | white | 27         | Shounen    |
| 2      | 11061 | Hunter x Hunter (2011)                      | <https://api-cdn.myanimelist.net/images/anime/1337/99013.jpg>  | <https://api-cdn.myanimelist.net/images/anime/1337/99013l.jpg>  | Hunter x Hunter                          | HUNTER×HUNTER(&lt;U+30CF&gt;&lt;U+30F3&gt;&lt;U+30BF&gt;&lt;U+30FC&gt;×&lt;U+30CF&gt;&lt;U+30F3&gt;&lt;U+30BF&gt;&lt;U+30FC&gt;) | 2011-10-02  | 2014-09-24 | tv          | finished\_airing  |           148 |               2011 | fall                 | pg\_13 | white | 27         | Shounen    |

The different demographics that I want to capture and their descriptions
are: 15 - kids, 25 - Shoujo, 27 - Shounen, 42 - Seinen, 43 - Josei

Lets construct a new demographic table and an associated lookup table;

``` r
anime_demo_table <- anime_genres_table %>%
  filter(genres_id %in% c(15,25,27,42,43)) %>%
  rename(demo_id = genres_id)
knitr::kable(head(anime_demo_table))
```

| tm\_ky | id    | demo\_id |
|:-------|:------|:---------|
| 2      | 10030 | 27       |
| 2      | 10049 | 27       |
| 2      | 10162 | 43       |
| 2      | 10165 | 27       |
| 2      | 10271 | 42       |
| 2      | 10379 | 25       |

``` r
demo_l <- genres_l %>%
  filter(genres_id %in% c(15,25,27,42,43)) %>%
  rename(demo_id = genres_id, demo_de = genres_de)
knitr::kable(head(demo_l))
```

| tm\_ky | demo\_id | demo\_de |
|:-------|:---------|:---------|
| 2      | 15       | Kids     |
| 2      | 25       | Shoujo   |
| 2      | 27       | Shounen  |
| 2      | 42       | Seinen   |
| 2      | 43       | Josei    |

Now, we need to remove those ids from the genres and genres lookup
tables.

``` r
anime_genres_table <- anime_genres_table %>%
  filter(!genres_id %in% c(15,25,27,42,43))

genres_l <- genres_l %>%
  filter(!genres_id %in% c(15,25,27,42,43))
```

### Final updates to tables

After reviewing the data and attempting to create some initial views and
filters in Tableau, its become apparent that I need to include columns
that collapse the values in the demographic, genres, synonyms, and
studios tables into single columns within the `anime_table` table.

To do that, I’ll create temporary dataframes that contain the ID and
comma delimited values, then join those to the main table so that the
additional columns can be added.

``` r
temp_demo_table <- anime_demo_table %>%
  inner_join(demo_l, by='demo_id') %>%
  group_by(id) %>%
  summarise(demo_de=paste(demo_de, collapse = ","))

temp_genres_table <- anime_genres_table %>%
  inner_join(genres_l, by='genres_id') %>%
  group_by(id) %>%
  summarise(genres_de=paste(genres_de, collapse = ","))

temp_studios_table <- anime_studios_table %>%
  inner_join(studios_l, by='studio_id') %>%
  group_by(id) %>%
  summarise(studios_de=paste(studio_de, collapse = ","))

temp_syn_table <- anime_syn_table %>%
  group_by(id) %>%
  summarise(synonyms=paste(synonyms, collapse = ","))
```

Here is what one of those tables looks like:

``` r
knitr::kable(head(temp_genres_table), "pipe")
```

| id    | genres\_de                                 |
|:------|:-------------------------------------------|
| 1     | Action,Adventure,Sci-Fi,Space,Comedy,Drama |
| 10030 | Romance,Comedy,Drama                       |
| 10049 | Action,Supernatural,Demons                 |
| 10087 | Action,Fantasy,Supernatural                |
| 10162 | Slice of Life                              |
| 10165 | School,Slice of Life,Comedy                |

Now, I need to join those temp tables and add those columns to the main
table.

While I’m at it, I’m also going to create a new column
`alternative title.` This columns will contain the english name by
default, and the anime title if that value is null. Similar to a
`coalesce` SQL call.

``` r
anime_table <- anime_table %>%
  left_join(temp_demo_table, by='id') %>%
  left_join(temp_genres_table, by='id') %>%
  left_join(temp_studios_table, by='id') %>%
  left_join(temp_syn_table, by='id') %>%
  mutate(alternative_title = case_when(
    is.null(alternative_titles.en) ~ title,
    trimws(alternative_titles.en) == '' ~ title,
    !is.null(trimws(alternative_titles.en)) ~ alternative_titles.en
  ))

anime_table %>%
  select(!synopsis) %>%
  head() %>%
  knitr::kable("pipe")
```

| tm\_ky | id    | title                                       | main\_picture.medium                                           | main\_picture.large                                             | alternative\_titles.en                   | alternative\_titles.ja                                                                                                                   | start\_date | end\_date  | media\_type | status            | num\_episodes | start\_season.year | start\_season.season | rating | nsfw  | demo\_de | genres\_de                                        | studios\_de           | synonyms                                                                          | alternative\_title                       |
|:-------|:------|:--------------------------------------------|:---------------------------------------------------------------|:----------------------------------------------------------------|:-----------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------|:------------|:-----------|:------------|:------------------|--------------:|-------------------:|:---------------------|:-------|:------|:---------|:--------------------------------------------------|:----------------------|:----------------------------------------------------------------------------------|:-----------------------------------------|
| 2      | 5114  | Fullmetal Alchemist: Brotherhood            | <https://api-cdn.myanimelist.net/images/anime/1223/96541.jpg>  | <https://api-cdn.myanimelist.net/images/anime/1223/96541l.jpg>  | Fullmetal Alchemist: Brotherhood         | &lt;U+92FC&gt;&lt;U+306E&gt;&lt;U+932C&gt;&lt;U+91D1&gt;&lt;U+8853&gt;&lt;U+5E2B&gt; FULLMETAL ALCHEMIST                                 | 2009-04-05  | 2010-07-04 | tv          | finished\_airing  |            64 |               2009 | spring               | r      | white | Shounen  | Action,Fantasy,Adventure,Military,Comedy,Drama    | Bones                 | Hagane no Renkinjutsushi: Fullmetal Alchemist,Fullmetal Alchemist (2009),FMA,FMAB | Fullmetal Alchemist: Brotherhood         |
| 2      | 48583 | Shingeki no Kyojin: The Final Season Part 2 | <https://api-cdn.myanimelist.net/images/anime/1948/120625.jpg> | <https://api-cdn.myanimelist.net/images/anime/1948/120625l.jpg> | Attack on Titan: The Final Season Part 2 | &lt;U+9032&gt;&lt;U+6483&gt;&lt;U+306E&gt;&lt;U+5DE8&gt;&lt;U+4EBA&gt; The Final Season Part 2                                           | 2022-01-10  | NA         | tv          | currently\_airing |             0 |               2022 | winter               | r      | white | Shounen  | Action,Fantasy,Super Power,Military,Mystery,Drama | MAPPA                 | Shingeki no Kyojin Season 4,Attack on Titan Season 4                              | Attack on Titan: The Final Season Part 2 |
| 2      | 38524 | Shingeki no Kyojin Season 3 Part 2          | <https://api-cdn.myanimelist.net/images/anime/1517/100633.jpg> | <https://api-cdn.myanimelist.net/images/anime/1517/100633l.jpg> | Attack on Titan Season 3 Part 2          | &lt;U+9032&gt;&lt;U+6483&gt;&lt;U+306E&gt;&lt;U+5DE8&gt;&lt;U+4EBA&gt; Season3 Part.2                                                    | 2019-04-29  | 2019-07-01 | tv          | finished\_airing  |            10 |               2019 | spring               | r      | white | Shounen  | Action,Fantasy,Super Power,Military,Mystery,Drama | Wit Studio            | NA                                                                                | Attack on Titan Season 3 Part 2          |
| 2      | 9253  | Steins;Gate                                 | <https://api-cdn.myanimelist.net/images/anime/5/73199.jpg>     | <https://api-cdn.myanimelist.net/images/anime/5/73199l.jpg>     | Steins;Gate                              | STEINS;GATE                                                                                                                              | 2011-04-06  | 2011-09-14 | tv          | finished\_airing  |            24 |               2011 | spring               | pg\_13 | white | NA       | Sci-Fi,Psychological,Suspense,Drama               | White Fox             | NA                                                                                | Steins;Gate                              |
| 2      | 28977 | Gintama°                                    | <https://api-cdn.myanimelist.net/images/anime/3/72078.jpg>     | <https://api-cdn.myanimelist.net/images/anime/3/72078l.jpg>     | Gintama Season 4                         | &lt;U+9280&gt;&lt;U+9B42&gt;°                                                                                                            | 2015-04-08  | 2016-03-30 | tv          | finished\_airing  |            51 |               2015 | spring               | pg\_13 | white | Shounen  | Action,Historical,Parody,Samurai,Sci-Fi,Comedy    | Bandai Namco Pictures | Gintama’ (2015)                                                                   | Gintama Season 4                         |
| 2      | 42938 | Fruits Basket: The Final                    | <https://api-cdn.myanimelist.net/images/anime/1085/114792.jpg> | <https://api-cdn.myanimelist.net/images/anime/1085/114792l.jpg> |                                          | &lt;U+30D5&gt;&lt;U+30EB&gt;&lt;U+30FC&gt;&lt;U+30C4&gt;&lt;U+30D0&gt;&lt;U+30B9&gt;&lt;U+30B1&gt;&lt;U+30C3&gt;&lt;U+30C8&gt; The Final | 2021-04-06  | 2021-06-29 | tv          | finished\_airing  |            13 |               2021 | spring               | pg\_13 | white | Shoujo   | Romance,Slice of Life,Supernatural,Drama          | TMS Entertainment     | Fruits Basket 3rd Season,Fruits Basket (2019) 3rd Season,Furuba                   | Fruits Basket: The Final                 |

``` r
temp_demo_table <- NULL
temp_genres_table <- NULL
temp_studios_table <- NULL
temp_syn_table <- NULL
```

Now we need to export the tables

``` r
write.csv(anime_demo_table, "temp_anime_demo_table.csv", row.names = FALSE)
write.csv(anime_genres_table, "temp_anime_genres_table.csv", row.names = FALSE)
write.csv(anime_ranking_table, "temp_anime_ranking_table.csv", row.names = FALSE)
write.csv(anime_studios_table, "temp_anime_studios_table.csv", row.names = FALSE)
write.csv(anime_syn_table, "temp_anime_syn_table.csv", row.names = FALSE)
write.csv(anime_table, "temp_anime_table.csv", row.names = FALSE)
write.csv(demo_l, "temp_demo_l.csv", row.names = FALSE)
write.csv(genres_l, "temp_genres_l.csv", row.names = FALSE)
write.csv(rank_table, "temp_rank_table.csv", row.names = FALSE)
write.csv(studios_l, "temp_studios_l.csv", row.names = FALSE)
write.csv(tm_entry, "temp_tm_ky.csv", row.names = FALSE)

anime_demo_table <- NULL
anime_genres_table <- NULL
anime_ranking_table <- NULL
anime_studios_table <- NULL
anime_syn_table <- NULL
anime_table <- NULL
demo_l <- NULL
genres_l <- NULL
rank_table <- NULL
studios_l <- NULL
tm_entry <- NULL
```

That is everything that is needed to extract the base tables.
[Next](https://github.com/patmendoza330/animelistclean), I will go through the process for cleaning the data
and loading them into their final table format.
