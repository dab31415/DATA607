<style>
.small-code pre code {
  font-size: 1em;
}
</style>

Reading an Unknown Set of Data Files
========================================================
author: Donald Butler
date: 09/29/2021
autosize: true

Introduction
========================================================

We run a polling script at each restaurant daily to identify the status of the point-of-sale terminals and kitchen controllers in the store. The results are then brought back to a file server. In the image below, you can see the list of files, one for each store, and a sample of their contents. For the daily report, we read each file to determine which devices are offline.

![Polling Data](DeviceStatus.png)

Generate a List of Files
========================================================
class: small-code

Use the list.files function to obtain a directory list


```r
localpath <- './DATA/'
# Get list of files in local DATA directory
filenames <- list.files(path = localpath, pattern = '.csv')

length(filenames)
```

```
[1] 31
```

```r
head(filenames, 10)
```

```
 [1] "00101_DeviceStatus.csv" "00102_DeviceStatus.csv" "00103_DeviceStatus.csv"
 [4] "00105_DeviceStatus.csv" "00106_DeviceStatus.csv" "00107_DeviceStatus.csv"
 [7] "00108_DeviceStatus.csv" "00110_DeviceStatus.csv" "00111_DeviceStatus.csv"
[10] "00112_DeviceStatus.csv"
```

Read Files
========================================================
class: small-code

For each file in the directory, use read_csv to load it and rbind to append it to a single data frame.


```r
library(tidyverse)

# Initialize data frame
df_local <- tibble()

for (i in 1:length(filenames)) {
  # read each file and append to df_local
  tmp <- read_csv(paste(localpath, filenames[i], sep = ''), col_types = cols(.default = 'c', 'Poll Time' = 't'),  show_col_types = FALSE)

  # if needed, process tmp data frame here before appending

  # append tmp data frame to df_local
  df_local <- rbind(df_local, tmp)
}
df_local
```

```
# A tibble: 31 x 21
   `Poll Date` `Poll Time` Site  POS1  POS2  POS3  POS4  POS5  POS6  POS7  POS8 
   <chr>       <time>      <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
 1 09/28/2021  05:58:11    00101 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 2 09/28/2021  05:58:45    00102 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 3 09/28/2021  05:58:10    00103 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 4 09/28/2021  05:58:02    00105 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 5 09/28/2021  05:58:00    00106 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 6 09/28/2021  05:58:00    00107 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 7 09/28/2021  05:58:31    00108 OK    OK    OK    OK    <NA>  OK    <NA>  OK   
 8 09/28/2021  05:58:05    00110 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 9 09/28/2021  05:58:07    00111 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
10 09/28/2021  05:58:03    00112 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
# ... with 21 more rows, and 10 more variables: POS9 <chr>, KC1 <chr>,
#   KC2 <chr>, KC3 <chr>, KC4 <chr>, KC5 <chr>, KC6 <chr>, KC7 <chr>,
#   KC8 <chr>, KC9 <chr>
```

Github Repository?
========================================================
class: small-code

For our class assignments, we've been asked to place our data files in a github repository, so I wondered if a similar technique could be used to load an unknown number of data files.

I found a solution in the github API which will return the contents of a path in the repository.
- https://docs.github.com/en/rest/reference/repos#get-repository-content

```
GET /repos/{owner}/{repo}/contents/{path}
```

The API will return a json string containing the contents of the path provided.

```
{
  "type": "file",
  "encoding": "base64",
  "size": 5362,
  "name": "README.md",
  "path": "README.md",
  "content": "encoded content ...",
  "sha": "3d21ec53a331a6f037a91c368710b99387d012c1",
  "url": "https://api.github.com/repos/octokit/octokit.rb/contents/README.md",
  "git_url": "https://api.github.com/repos/octokit/octokit.rb/git/blobs/3d21ec53a331a6f037a91c368710b99387d012c1",
  "html_url": "https://github.com/octokit/octokit.rb/blob/master/README.md",
  "download_url": "https://raw.githubusercontent.com/octokit/octokit.rb/master/README.md",
  "_links": {
    "git": "https://api.github.com/repos/octokit/octokit.rb/git/blobs/3d21ec53a331a6f037a91c368710b99387d012c1",
    "self": "https://api.github.com/repos/octokit/octokit.rb/contents/README.md",
    "html": "https://github.com/octokit/octokit.rb/blob/master/README.md"
  }
}
```

Get list of Github Repository Files
========================================================
class: small-code


```r
library(httr)

URL <- 'https://api.github.com/repos/dab31415/DATA607/contents/Projects/Context_Presentation/DATA'
APIResult <- GET(URL)
stop_for_status(APIResult)
APIResult
```

```
Response [https://api.github.com/repos/dab31415/DATA607/contents/Projects/Context_Presentation/DATA]
  Date: 2021-12-06 23:34
  Status: 200
  Content-Type: application/json; charset=utf-8
  Size: 34.6 kB
[
  {
    "name": "00101_DeviceStatus.csv",
    "path": "Projects/Context_Presentation/DATA/00101_DeviceStatus.csv",
    "sha": "a18c774d1de5f870a32a9721e8e5af4c2bdafa7b",
    "size": 168,
    "url": "https://api.github.com/repos/dab31415/DATA607/contents/Projects/C...
    "html_url": "https://github.com/dab31415/DATA607/blob/main/Projects/Conte...
    "git_url": "https://api.github.com/repos/dab31415/DATA607/git/blobs/a18c7...
    "download_url": "https://raw.githubusercontent.com/dab31415/DATA607/main/...
...
```

Convert Result JSON to data frame
========================================================
class: small-code


```r
library(jsonlite)

json <- fromJSON(content(APIResult,'text'))

glimpse(json)
```

```
Rows: 31
Columns: 10
$ name         <chr> "00101_DeviceStatus.csv", "00102_DeviceStatus.csv", "0010~
$ path         <chr> "Projects/Context_Presentation/DATA/00101_DeviceStatus.cs~
$ sha          <chr> "a18c774d1de5f870a32a9721e8e5af4c2bdafa7b", "a97dd0970b62~
$ size         <int> 168, 168, 168, 168, 168, 168, 176, 168, 168, 168, 166, 16~
$ url          <chr> "https://api.github.com/repos/dab31415/DATA607/contents/P~
$ html_url     <chr> "https://github.com/dab31415/DATA607/blob/main/Projects/C~
$ git_url      <chr> "https://api.github.com/repos/dab31415/DATA607/git/blobs/~
$ download_url <chr> "https://raw.githubusercontent.com/dab31415/DATA607/main/~
$ type         <chr> "file", "file", "file", "file", "file", "file", "file", "~
$ `_links`     <df[,3]> <data.frame[26 x 3]>
```

Read Github Repository Files
========================================================
class: small-code

Use the download_url attribute to read the csv files from the github repository.


```r
selected_files <- json %>%
	filter(grepl('.csv', name)) %>%
	select(download_url)

df_web <- tibble()
for (i in 1:nrow(selected_files)) {
  # read each file and append to df_local
  tmp <- read_csv(selected_files$download_url[i], col_types = cols(.default = 'c', 'Poll Time' = 't'), show_col_types = FALSE)
  
  # if needed, process tmp data frame here before appending

  # append tmp data frame to df_web
  df_web <- rbind(df_web, tmp)
}
df_web
```

```
# A tibble: 31 x 21
   `Poll Date` `Poll Time` Site  POS1  POS2  POS3  POS4  POS5  POS6  POS7  POS8 
   <chr>       <time>      <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
 1 09/28/2021  05:58:11    00101 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 2 09/28/2021  05:58:45    00102 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 3 09/28/2021  05:58:10    00103 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 4 09/28/2021  05:58:02    00105 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 5 09/28/2021  05:58:00    00106 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 6 09/28/2021  05:58:00    00107 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 7 09/28/2021  05:58:31    00108 OK    OK    OK    OK    <NA>  OK    <NA>  OK   
 8 09/28/2021  05:58:05    00110 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
 9 09/28/2021  05:58:07    00111 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
10 09/28/2021  05:58:03    00112 OK    OK    OK    OK    <NA>  <NA>  <NA>  <NA> 
# ... with 21 more rows, and 10 more variables: POS9 <chr>, KC1 <chr>,
#   KC2 <chr>, KC3 <chr>, KC4 <chr>, KC5 <chr>, KC6 <chr>, KC7 <chr>,
#   KC8 <chr>, KC9 <chr>
```

Conclusion
========================================================
class: small-code

A couple of techniques used to solve the initial problem.

- Using a web API to get information back into my project
- Converting the json text into a useful data format in R
- Implementing an R-Presentation document

