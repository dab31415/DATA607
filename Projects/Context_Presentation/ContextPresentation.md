Loading Multiple Data Files
========================================================
author: Donald Butler
date: 09/28/2021
autosize: true

Introduction
========================================================

We run a polling script at each restaurant daily to identify the status of the point-of-sale terminals and kitchen controllers in the store. The results are then brought back to a file server. In the image below, you can see the list of files, one for each store, and a sample of their contents. For the daily report, we read each file to determine which devices are offline.

![Polling Data](DeviceStatus.png)

Generate a List of Files
========================================================

Use the list.files function to obtain a directory list


```r
localpath <- './DATA/'
# Get list of files in local DATA directory
filenames <- list.files(path = localpath, pattern = '.csv')
head(filenames, 10)
```

```
 [1] "00101_DeviceStatus.csv" "00102_DeviceStatus.csv" "00103_DeviceStatus.csv"
 [4] "00105_DeviceStatus.csv" "00106_DeviceStatus.csv" "00107_DeviceStatus.csv"
 [7] "00108_DeviceStatus.csv" "00110_DeviceStatus.csv" "00111_DeviceStatus.csv"
[10] "00112_DeviceStatus.csv"
```

Loop through the list of files
========================================================

For each file in the directory, use read_csv to load it and rbind to append it to a single data frame.


```r
library(tidyverse)

# Create full path names
filepaths <- paste(localpath, filenames, sep = '')

# Initialize data frame
df_local <- tibble()

for (i in 1:length(filepaths)) {
  # read each file and append to df_local
  df_local <- rbind(df_local, read_csv(filepaths[i], col_types = cols(.default = 'c', 'Poll Time' = 't'),  show_col_types = FALSE))
}
```

Final Result
========================================================

From here, we can tidy the dataset and generate the report that we need.


```r
glimpse(df_local)
```

```
Rows: 31
Columns: 21
$ `Poll Date` <chr> "09/28/2021", "09/28/2021", "09/28/2021", "09/28/2021", "0~
$ `Poll Time` <time> 05:58:11, 05:58:45, 05:58:10, 05:58:02, 05:58:00, 05:58:0~
$ Site        <chr> "00101", "00102", "00103", "00105", "00106", "00107", "001~
$ POS1        <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ POS2        <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ POS3        <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ POS4        <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ POS5        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA~
$ POS6        <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, "OK", "O~
$ POS7        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA~
$ POS8        <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, NA, NA, ~
$ POS9        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA~
$ KC1         <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, "OK", NA~
$ KC2         <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, "OK", NA~
$ KC3         <chr> "OK", "OK", "OK", "OK", "OK", "OK", NA, "OK", "OK", "OK", ~
$ KC4         <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ KC5         <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ KC6         <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ KC7         <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ KC8         <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, "OK", NA~
$ KC9         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA~
```


Github Repository?
========================================================

For our class assignments, we've been asked to place our data files in a github repository, so I wondered if a similar technique could be used to load an unknown number of data files.

I found a solution in the github API, that would allow a call to return the contents of a directory within a repository.
- https://docs.github.com/en/rest/reference/repos


Get Directory Files
========================================================


```r
library(httr)

URL <- 'https://api.github.com/repos/dab31415/DATA607/contents/Projects/Context_Presentation/DATA'
APIResult <- GET(URL)
stop_for_status(APIResult)
APIResult
```

```
Response [https://api.github.com/repos/dab31415/DATA607/contents/Projects/Context_Presentation/DATA]
  Date: 2021-09-29 06:52
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

Loop through download_url attribute
========================================================

Use the download_url attribute to read the csv files from the github repository.


```r
df_web <- tibble()
for (i in 1:nrow(json)) {
  df_web <- rbind(df_web, read_csv(json$download_url[i], col_types = cols(.default = 'c', 'Poll Time' = 't'), show_col_types = FALSE))
}
```

Final Result from Github
========================================================


```r
glimpse(df_web)
```

```
Rows: 31
Columns: 21
$ `Poll Date` <chr> "09/28/2021", "09/28/2021", "09/28/2021", "09/28/2021", "0~
$ `Poll Time` <time> 05:58:11, 05:58:45, 05:58:10, 05:58:02, 05:58:00, 05:58:0~
$ Site        <chr> "00101", "00102", "00103", "00105", "00106", "00107", "001~
$ POS1        <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ POS2        <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ POS3        <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ POS4        <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ POS5        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA~
$ POS6        <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, "OK", "O~
$ POS7        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA~
$ POS8        <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, NA, NA, ~
$ POS9        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA~
$ KC1         <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, "OK", NA~
$ KC2         <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, "OK", NA~
$ KC3         <chr> "OK", "OK", "OK", "OK", "OK", "OK", NA, "OK", "OK", "OK", ~
$ KC4         <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ KC5         <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ KC6         <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ KC7         <chr> "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK", "OK"~
$ KC8         <chr> NA, NA, NA, NA, NA, NA, "OK", NA, NA, NA, NA, NA, "OK", NA~
$ KC9         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA~
```

