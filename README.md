# Learning goals

-   Use the `merge()` function to join two datasets.
-   Deal with missings and impute data.
-   Identify relevant observations using `quantile()`.
-   Practice your GitHub skills.

# Lab description

For this lab we will be dealing with the meteorological dataset `met`.
In this case, we will use `data.table` to answer some questions
regarding the `met` dataset, while at the same time practice your
Git+GitHub skills for this project.

This markdown document should be rendered using `github_document`
document.

# Part 1: Setup a Git project and the GitHub repository

1.  Go to wherever you are planning to store the data on your computer,
    and create a folder for this project

2.  In that folder, save [this
    template](https://github.com/JSC370/JSC370-2025/blob/main/labs/lab05/lab05-wrangling-gam.Rmd)
    as “README.Rmd”. This will be the markdown file where all the magic
    will happen.

3.  Go to your GitHub account and create a new repository of the same
    name that your local folder has, e.g., “JSC370-labs”.

4.  Initialize the Git project, add the “README.Rmd” file, and make your
    first commit.

5.  Add the repo you just created on GitHub.com to the list of remotes,
    and push your commit to origin while setting the upstream.

Most of the steps can be done using command line:

    # Step 1
    cd ~/Documents
    mkdir JSC370-labs
    cd JSC370-labs

    # Step 2
    wget https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd
    mv lab05-wrangling-gam.Rmd README.Rmd
    # if wget is not available,
    curl https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd --output README.Rmd

    # Step 3
    # Happens on github

    # Step 4
    git init
    git add README.Rmd
    git commit -m "First commit"

    # Step 5
    git remote add origin git@github.com:[username]/JSC370-labs
    git push -u origin master

You can also complete the steps in R (replace with your paths/username
when needed)

    # Step 1
    setwd("~/Documents")
    dir.create("JSC370-labs")
    setwd("JSC370-labs")

    # Step 2
    download.file(
      "https://raw.githubusercontent.com/JSC370/jsc370-2023/main/labs/lab05/lab05-wrangling-gam.Rmd",
      destfile = "README.Rmd"
      )

    # Step 3: Happens on Github

    # Step 4
    system("git init && git add README.Rmd")
    system('git commit -m "First commit"')

    # Step 5
    system("git remote add origin git@github.com:[username]/JSC370-labs")
    system("git push -u origin master")

Once you are done setting up the project, you can now start working with
the MET data.

## Setup in R

1.  Load the `data.table` (and the `dtplyr` and `dplyr` packages).

<!-- -->

    library(data.table)

    ## Warning: package 'data.table' was built under R version 4.3.3

    library(dplyr)

    ## Warning: package 'dplyr' was built under R version 4.3.3

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     between, first, last

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(dtplyr)

    ## Warning: package 'dtplyr' was built under R version 4.3.3

1.  Load the met data from
    <https://raw.githubusercontent.com/JSC370/JSC370-2024/main/data/met_all_2023.gz>,
    and also the station data. For the latter, you can use the code we
    used during lecture to pre-process the stations data:

<!-- -->

    # Download the data
    stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
    stations[, USAF := as.integer(USAF)]

    ## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion

    # Dealing with NAs and 999999
    stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
    stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
    stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

    # Selecting the three relevant columns, and keeping unique records
    stations <- unique(stations[, list(USAF, CTRY, STATE)])

    # Dropping NAs
    stations <- stations[!is.na(USAF)]

    # Removing duplicates
    stations[, n := 1:.N, by = .(USAF)]
    stations <- stations[n == 1,][, n := NULL]

    # Read in the met data
    met <- data.table::fread("met_all.gz")

    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection

    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection
    ## Warning in writeBin(bfr, con = out, size = 1L): problem writing to connection

    ## Warning in data.table::fread("met_all.gz"): Stopped early on line 634006.
    ## Expected 30 fields but found 44. Consider fill=TRUE and comment.char=. First
    ## discarded non-empty line:
    ## <<720967,457,2019,8,17,13,58,30.718,-91.479,12,,9,C,0,5,22000,5,9,N,16093,5,N,5,28,C,24,C,,9,78.922513432000,5,9,N,16093,5,N,5,18.9,5,10,5,1008.2,5,56.4477107746572>>

1.  Merge the data as we did during the lecture. Use the `merge()` code
    and you can also try the tidy way with `left_join()`

<!-- -->

    met_stations <-
      left_join(
        met, stations, by = c("USAFID" = "USAF")
      )

    # met <- merge(met, stations, all.x = TRUE, all.y = FALSE, by.x="USAFID", by.y="USAF")

## Question 1: Representative station for the US

Across all weather stations, what stations have the median values of
temperature, wind speed, and atmospheric pressure? Using the
`quantile()` function, identify these three stations. Do they coincide?

    medians <- met_stations[, .(
      temp_50 = quantile(temp, probs = .5, na.rm = TRUE),
      wind.sp_50 = quantile(wind.sp, probs = .5, na.rm = TRUE),
      atm.press_50 = quantile(atm.press, probs = .5, na.rm = TRUE)
    )]

    medians

    ##    temp_50 wind.sp_50 atm.press_50
    ##      <num>      <num>        <num>
    ## 1:    24.3        2.1       1014.8

    # medians by stations
    station_med <- met_stations[, .(
      temp = quantile(temp, probs = .5, na.rm = TRUE),
      wind.sp = quantile(wind.sp, probs = .5, na.rm = TRUE),
      atm.press = quantile(atm.press, probs = .5, na.rm = TRUE)
    ), by = .(USAFID, STATE, lat, lon)]

    station_med[ , temp_dist := abs(temp - medians$temp_50)]
    median_temp_station <- station_med[temp_dist == 0]
    median_temp_station

    ## Empty data.table (0 rows and 8 cols): USAFID,STATE,lat,lon,temp,wind.sp...

    station_med[ , wind.sp_dist := abs(wind.sp - medians$wind.sp_50)]
    median_wind.sp_station <- station_med[wind.sp_dist == 0]
    median_wind.sp_station

    ##     USAFID  STATE    lat      lon  temp wind.sp atm.press temp_dist
    ##      <int> <char>  <num>    <num> <num>   <num>     <num>     <num>
    ##  1: 720110     TX 30.784  -98.662 31.00     2.1        NA      6.70
    ##  2: 720120     SC 32.224  -80.697 28.00     2.1        NA      3.70
    ##  3: 720198     MI 46.410  -86.650 15.00     2.1    1012.0      9.30
    ##  4: 720258     MN 46.619  -93.310 17.00     2.1        NA      7.30
    ##  5: 720266     IN 41.275  -85.840 21.00     2.1        NA      3.30
    ##  6: 720272     WA 48.467 -122.416 18.00     2.1        NA      6.30
    ##  7: 720273     TX 28.973  -95.863 28.60     2.1        NA      4.30
    ##  8: 720279     NC 34.602  -78.578 25.00     2.1        NA      0.70
    ##  9: 720283     MN 43.677  -92.180 19.10     2.1        NA      5.20
    ## 10: 720284     MI 42.574  -84.811 20.50     2.1        NA      3.80
    ## 11: 720291     TX 32.747  -96.531 27.00     2.1        NA      2.70
    ## 12: 720293     IA 42.453  -91.948 20.80     2.1        NA      3.50
    ## 13: 720298     TX 31.869  -95.218 28.10     2.1        NA      3.80
    ## 14: 720299     TX 32.456  -96.913 29.10     2.1        NA      4.80
    ## 15: 720303     TX 30.872  -96.622 29.50     2.1        NA      5.20
    ## 16: 720306     MO 38.956  -94.371 29.40     2.1    1008.8      5.10
    ## 17: 720308     NE 41.196  -96.112 24.20     2.1        NA      0.10
    ## 18: 720309     IA 40.947  -91.511 22.00     2.1        NA      2.30
    ## 19: 720318     TX 33.110  -98.555 29.30     2.1        NA      5.00
    ## 20: 720322     ID 48.300 -116.560 16.00     2.1        NA      8.30
    ## 21: 720323     TX 30.243  -98.910 28.10     2.1        NA      3.80
    ## 22: 720326     IA 42.219  -92.026 21.60     2.1        NA      2.70
    ## 23: 720327     WI 44.892  -91.868 17.70     2.1        NA      6.60
    ## 24: 720330     IL 38.607  -87.727 22.80     2.1        NA      1.50
    ## 25: 720333     CA 33.898 -117.602 22.20     2.1    1012.4      2.10
    ## 26: 720339     AZ 32.142 -111.175 32.00     2.1        NA      7.70
    ## 27: 720340     MI 44.626  -86.201 21.20     2.1        NA      3.10
    ## 28: 720344     IA 42.732  -95.556 20.50     2.1        NA      3.80
    ## 29: 720351     IA 41.226  -92.491 20.90     2.1        NA      3.40
    ## 30: 720355     MD 39.615  -78.761 22.10     2.1        NA      2.20
    ## 31: 720357     OK 35.950  -96.773 26.10     2.1        NA      1.80
    ## 32: 720367     MN 45.372  -94.746 19.00     2.1        NA      5.30
    ## 33: 720367     MN 45.372  -94.747 19.15     2.1        NA      5.15
    ## 34: 720374     FL 27.916  -82.449 30.00     2.1        NA      5.70
    ## 35: 720377     AR 35.638  -91.176 26.00     2.1    1013.5      1.70
    ## 36: 720383     FL 30.704  -87.023 25.60     2.1    1013.2      1.30
    ## 37: 720406     CA 38.150 -122.550 19.00     2.1        NA      5.30
    ## 38: 720407     NJ 39.928  -74.292 24.00     2.1    1016.8      0.30
    ## 39: 720407     NJ 39.933  -74.300 23.00     2.1    1017.0      1.30
    ## 40: 720412     IA 41.828  -94.160 21.20     2.1        NA      3.10
    ## 41: 720414     OH 40.333  -82.517 21.00     2.1        NA      3.30
    ## 42: 720415     MI 43.433  -86.000 19.90     2.1        NA      4.40
    ## 43: 720436     KS 37.450  -94.733 23.40     2.1        NA      0.90
    ## 44: 720447     KY 36.665  -88.373 24.00     2.1        NA      0.30
    ## 45: 720455     KY 37.633  -84.333 24.00     2.1        NA      0.30
    ## 46: 720456     KY 38.067  -83.983 23.00     2.1        NA      1.30
    ## 47: 720493     VT 44.883  -72.233 18.00     2.1        NA      6.30
    ## 48: 720493     VT 44.889  -72.229 17.70     2.1        NA      6.60
    ## 49: 720498     VA 37.400  -77.517 23.90     2.1    1016.5      0.40
    ## 50: 720528     CO 38.698 -106.070 17.30     2.1        NA      7.00
    ## 51: 720528     CO 38.817 -106.117 16.60     2.1        NA      7.70
    ## 52: 720531     CO 38.783 -108.067 24.60     2.1        NA      0.30
    ## 53: 720545     CT 41.384  -72.506 22.00     2.1        NA      2.30
    ## 54: 720561     OK 36.175  -96.152 27.30     2.1        NA      3.00
    ## 55: 720575     IN 40.031  -86.251 21.00     2.1        NA      3.30
    ## 56: 720576     CA 38.533 -121.783 23.00     2.1        NA      1.30
    ## 57: 720589     WI 44.783  -88.550 19.50     2.1        NA      4.80
    ## 58: 720593     IN 41.333  -86.667 21.00     2.1        NA      3.30
    ## 59: 720601     SC 33.650  -81.683 25.00     2.1        NA      0.70
    ## 60: 720601     SC 33.649  -81.685 27.00     2.1        NA      2.70
    ## 61: 720602     SC 33.250  -81.383 26.00     2.1        NA      1.70
    ## 62: 720608     SC 34.181  -79.335 25.00     2.1        NA      0.70
    ## 63: 720609     SC 32.917  -80.633 25.00     2.1        NA      0.70
    ## 64: 720609     SC 32.921  -80.641 26.00     2.1        NA      1.70
    ## 65: 720612     SC 32.400  -80.633 27.00     2.1        NA      2.70
    ## 66: 720617     TX 29.800  -95.900 29.00     2.1        NA      4.70
    ## 67: 720634     SC 34.315  -81.109 25.00     2.1        NA      0.70
    ## 68: 720636     MO 38.350  -93.683 23.00     2.1        NA      1.30
    ## 69: 720644     AZ 33.420 -112.686 33.00     2.1        NA      8.70
    ## 70: 720647     TX 31.106  -98.196 29.00     2.1        NA      4.70
    ## 71: 720651     OH 40.225  -83.352 21.25     2.1        NA      3.05
    ## 72: 720701     IA 41.052  -93.690 22.50     2.1        NA      1.80
    ## 73: 720713     OH 40.204  -84.532 21.00     2.1        NA      3.30
    ## 74: 720734     ID 43.581 -116.523 24.00     2.1        NA      0.30
    ## 75: 720735     FL 30.349  -85.788 26.70     2.1    1015.4      2.40
    ## 76: 720736     IN 41.066  -86.182 23.00     2.1        NA      1.30
    ## 77: 720741     NV 35.947 -114.861 33.00     2.1        NA      8.70
    ## 78: 720887     WI 46.117  -89.883 16.20     2.1        NA      8.10
    ## 79: 720903     IN 38.700  -87.130 22.60     2.1        NA      1.70
    ## 80: 720916     LA 27.633  -90.450 31.00     2.1        NA      6.70
    ## 81: 720928     OH 40.280  -83.115 22.00     2.1        NA      2.30
    ## 82: 720929     WI 45.506  -91.981 17.00     2.1        NA      7.30
    ## 83: 720942     NE 41.241  -96.594 23.10     2.1        NA      1.20
    ## 84: 720949     MO 37.974  -92.691 23.00     2.1        NA      1.30
    ## 85: 720961     IN 40.711  -86.375 21.00     2.1        NA      3.30
    ##     USAFID  STATE    lat      lon  temp wind.sp atm.press temp_dist
    ##     wind.sp_dist
    ##            <num>
    ##  1:            0
    ##  2:            0
    ##  3:            0
    ##  4:            0
    ##  5:            0
    ##  6:            0
    ##  7:            0
    ##  8:            0
    ##  9:            0
    ## 10:            0
    ## 11:            0
    ## 12:            0
    ## 13:            0
    ## 14:            0
    ## 15:            0
    ## 16:            0
    ## 17:            0
    ## 18:            0
    ## 19:            0
    ## 20:            0
    ## 21:            0
    ## 22:            0
    ## 23:            0
    ## 24:            0
    ## 25:            0
    ## 26:            0
    ## 27:            0
    ## 28:            0
    ## 29:            0
    ## 30:            0
    ## 31:            0
    ## 32:            0
    ## 33:            0
    ## 34:            0
    ## 35:            0
    ## 36:            0
    ## 37:            0
    ## 38:            0
    ## 39:            0
    ## 40:            0
    ## 41:            0
    ## 42:            0
    ## 43:            0
    ## 44:            0
    ## 45:            0
    ## 46:            0
    ## 47:            0
    ## 48:            0
    ## 49:            0
    ## 50:            0
    ## 51:            0
    ## 52:            0
    ## 53:            0
    ## 54:            0
    ## 55:            0
    ## 56:            0
    ## 57:            0
    ## 58:            0
    ## 59:            0
    ## 60:            0
    ## 61:            0
    ## 62:            0
    ## 63:            0
    ## 64:            0
    ## 65:            0
    ## 66:            0
    ## 67:            0
    ## 68:            0
    ## 69:            0
    ## 70:            0
    ## 71:            0
    ## 72:            0
    ## 73:            0
    ## 74:            0
    ## 75:            0
    ## 76:            0
    ## 77:            0
    ## 78:            0
    ## 79:            0
    ## 80:            0
    ## 81:            0
    ## 82:            0
    ## 83:            0
    ## 84:            0
    ## 85:            0
    ##     wind.sp_dist

    station_med[ , atm.press_dist := abs(atm.press - medians$atm.press_50)]
    median_atm.press_station <- station_med[atm.press_dist == 0]
    median_atm.press_station

    ## Empty data.table (0 rows and 10 cols): USAFID,STATE,lat,lon,temp,wind.sp...

Knit the document, commit your changes, and save it on GitHub. Don’t
forget to add `README.md` to the tree, the first time you render it.

## Question 2: Representative station per state

Just like the previous question, you are asked to identify what is the
most representative, the median, station per state. This time, instead
of looking at one variable at a time, look at the euclidean distance. If
multiple stations show in the median, select the one located at the
lowest latitude.

    station_med[, temp_50 := quantile(temp, probs = .5, na.rm = TRUE), by = STATE]
    station_med[, wind.sp_50 := quantile(wind.sp, probs = .5, na.rm = TRUE), by = STATE]

    # euclidean distance
    station_med[, eudist := sqrt(
      (temp - temp_50)^2 + (wind.sp - wind.sp_50)^2 
      # we omitted atm.press in lab, but we could add another term (atm.press - atm.press_50)^2
    )]

    # choose most representative, median, station per state with the lowest euclidean dist
    id_station <- station_med[, .SD[which.min(eudist)], by = STATE]

    id_station <- merge(
      x = id_station, y = stations,
      by.x = "USAFID", by.y = "USAF",
      all.x = TRUE, all.y = FALSE
    )

Knit the doc and save it on GitHub.

## Question 3: In the middle?

For each state, identify what is the station that is closest to the
mid-point of the state. Combining these with the stations you identified
in the previous question, use `leaflet()` to visualize all ~100 points
in the same figure, applying different colors for those identified in
this question.

    library(leaflet)

    ## Warning: package 'leaflet' was built under R version 4.3.3

    # 1) get the midpoint of the state
    mid_point <- met_stations[, .(
      lon_50 = quantile(lon, probs = .5, na.rm = TRUE),
      lat_50 = quantile(lat, probs = .5, na.rm = TRUE)
    ), by = STATE]

    mid <- merge(x = met_stations, y = mid_point, by = "STATE")

    # 2) calculate eudist for lon and lat
    mid[, mid_eudist := sqrt(
      (lon - lon_50)^2 + (lat - lat_50)^2
    )]

    # 3) find closest station to the midpoint of the state
    mid_station <- mid[, .SD[which.min(mid_eudist)], by = STATE]

    leaflet() %>% 
      addProviderTiles('CartoDB.Positron') %>%
      addCircles(
        data = mid_station, 
        lat = ~lat, lng = ~lon, popup = "geographic mid station",
        opacity = 1, fillOpacity = 1, radius = 400, color = "blue") %>% 
      addCircles(
        data = id_station, 
        lat = ~lat, lng = ~lon, popup = "eudist mid station",
        opacity = 1, fillOpacity = 1, radius = 400, color = "magenta") 

<div class="leaflet html-widget html-fill-item" id="htmlwidget-8234fd28920de077de3b" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-8234fd28920de077de3b">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addProviderTiles","args":["CartoDB.Positron",null,null,{"errorTileUrl":"","noWrap":false,"detectRetina":false}]},{"method":"addCircles","args":[[34.229,34.1,33.417,38.909,40.033,41.384,28.867,32.214,41.4,48.299,38.607,41.066,37.158,37.633,29.976,39.608,42.575,45.372,37.974,33.433,36.361,48.39,41.196,40.617,36.422,39.183,40.225,35.483,45.417,40.435,34.283,44.016,31.106,41.552,37.239,44.883,47.104,44.251,39,42.796],[-86.256,-93.066,-112.683,-121.351,-105.217,-72.506,-82.56699999999999,-83.128,-92.946,-116.56,-87.727,-86.182,-95.77800000000001,-84.333,-92.084,-77.008,-84.81100000000001,-94.746,-92.691,-88.849,-78.529,-100.024,-96.11199999999999,-74.25,-105.29,-119.733,-83.352,-97.81699999999999,-123.817,-75.38200000000001,-80.56699999999999,-97.086,-98.196,-112.062,-76.71599999999999,-72.233,-122.287,-90.855,-80.274,-109.806],400,null,null,{"interactive":true,"className":"","stroke":true,"color":"blue","weight":5,"opacity":1,"fill":true,"fillColor":"blue","fillOpacity":1},"geographic mid station",null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]},{"method":"addCircles","args":[[41.425,45.417,46.677,41.275,43.677,40.138,33.254,39,39.167,32.142,35.864,42.796,31.043,43.743,34.1,41.417,39.928,36.422,41.828,40.333,39.767,37.578,44.567,37.85,40.617,41.384,39.192,41.552,38.533,44.783,44.016,38.35,31.554,30.349,47.451,34.81,32.304,35.937,43.067,27.63],[-88.419,-123.817,-122.983,-85.84,-92.18000000000001,-75.265,-97.581,-80.274,-77.167,-111.175,-98.42100000000001,-109.806,-86.312,-111.097,-93.066,-96.117,-74.292,-105.29,-94.16,-82.517,-101.8,-84.77,-72.017,-76.883,-103.267,-72.506,-119.734,-112.062,-121.783,-88.55,-97.086,-93.68300000000001,-81.883,-85.788,-99.151,-82.703,-90.411,-77.547,-83.267,-90.45],400,null,null,{"interactive":true,"className":"","stroke":true,"color":"magenta","weight":5,"opacity":1,"fill":true,"fillColor":"magenta","fillOpacity":1},"eudist mid station",null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]}],"limits":{"lat":[27.63,48.39],"lng":[-123.817,-72.017]}},"evals":[],"jsHooks":[]}</script>

Knit the doc and save it on GitHub.

## Question 4: Means of means

Using the `quantile()` function, generate a summary table that shows the
number of states included, average temperature, wind-speed, and
atmospheric pressure by the variable “average temperature level,” which
you’ll need to create.

Start by computing the states’ average temperature. Use that measurement
to classify them according to the following criteria:

-   low: temp &lt; 20
-   Mid: temp &gt;= 20 and temp &lt; 25
-   High: temp &gt;= 25

<!-- -->

    met_stations[ , elev_cat := fifelse(
      elev < 90, "low-elev", "high-elev"
    )]

    ## Warning in `[.data.table`(met_stations, , `:=`(elev_cat, fifelse(elev < :
    ## Invalid .internal.selfref detected and fixed by taking a (shallow) copy of the
    ## data.table so that := can add this new column by reference. At an earlier
    ## point, this data.table has been copied by R (or was created manually using
    ## structure() or similar). Avoid names<- and attr<- which in R currently (and
    ## oddly) may copy the whole data.table. Use set* syntax instead to avoid copying:
    ## ?set, ?setnames and ?setattr. If this message doesn't help, please report your
    ## use case to the data.table issue tracker so the root cause can be fixed or this
    ## message improved.

Once you are done with that, you can compute the following:

-   Number of entries (records),
-   Number of NA entries,
-   Number of stations,
-   Number of states included, and
-   Mean temperature, wind-speed, and atmospheric pressure.

All by the levels described before.

    library(tidyr)

    ## Warning: package 'tidyr' was built under R version 4.3.3

    library(kableExtra)

    ## Warning: package 'kableExtra' was built under R version 4.3.3

    ## 
    ## Attaching package: 'kableExtra'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     group_rows

    summary_table <- met_stations %>% 
      group_by(STATE, elev_cat) %>% 
      summarize(temp_mean = mean(temp, na.rm = T)) %>% 
      pivot_wider(names_from = elev_cat, values_from = temp_mean)

    ## `summarise()` has grouped output by 'STATE'. You can override using the
    ## `.groups` argument.

    kable(summary_table, booktabs = TRUE) %>% 
      kable_styling(font_size = 10) %>% 
      kable_paper("hover", full_width = F)

<table class="table lightable-paper lightable-hover" style="font-size: 10px; margin-left: auto; margin-right: auto; font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
STATE
</th>
<th style="text-align:right;">
high-elev
</th>
<th style="text-align:right;">
low-elev
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
AL
</td>
<td style="text-align:right;">
25.84353
</td>
<td style="text-align:right;">
26.78506
</td>
</tr>
<tr>
<td style="text-align:left;">
AR
</td>
<td style="text-align:right;">
26.59029
</td>
<td style="text-align:right;">
26.69127
</td>
</tr>
<tr>
<td style="text-align:left;">
AZ
</td>
<td style="text-align:right;">
28.42515
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
CA
</td>
<td style="text-align:right;">
25.09679
</td>
<td style="text-align:right;">
20.32356
</td>
</tr>
<tr>
<td style="text-align:left;">
CO
</td>
<td style="text-align:right;">
20.84742
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
CT
</td>
<td style="text-align:right;">
22.44858
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
FL
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
27.33816
</td>
</tr>
<tr>
<td style="text-align:left;">
GA
</td>
<td style="text-align:right;">
26.32966
</td>
<td style="text-align:right;">
26.22137
</td>
</tr>
<tr>
<td style="text-align:left;">
IA
</td>
<td style="text-align:right;">
21.39755
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
ID
</td>
<td style="text-align:right;">
20.44709
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
IL
</td>
<td style="text-align:right;">
23.02695
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
IN
</td>
<td style="text-align:right;">
21.33242
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
KS
</td>
<td style="text-align:right;">
23.62669
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
KY
</td>
<td style="text-align:right;">
23.67964
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
LA
</td>
<td style="text-align:right;">
29.23461
</td>
<td style="text-align:right;">
28.17126
</td>
</tr>
<tr>
<td style="text-align:left;">
MD
</td>
<td style="text-align:right;">
23.63678
</td>
<td style="text-align:right;">
25.67102
</td>
</tr>
<tr>
<td style="text-align:left;">
MI
</td>
<td style="text-align:right;">
20.68652
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
MN
</td>
<td style="text-align:right;">
19.07052
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
MO
</td>
<td style="text-align:right;">
23.98282
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
MS
</td>
<td style="text-align:right;">
25.73138
</td>
<td style="text-align:right;">
27.29035
</td>
</tr>
<tr>
<td style="text-align:left;">
NC
</td>
<td style="text-align:right;">
23.65205
</td>
<td style="text-align:right;">
25.11932
</td>
</tr>
<tr>
<td style="text-align:left;">
ND
</td>
<td style="text-align:right;">
18.32006
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
NE
</td>
<td style="text-align:right;">
23.14443
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
NJ
</td>
<td style="text-align:right;">
NA
</td>
<td style="text-align:right;">
23.98582
</td>
</tr>
<tr>
<td style="text-align:left;">
NM
</td>
<td style="text-align:right;">
14.28740
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
NV
</td>
<td style="text-align:right;">
26.07013
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
OH
</td>
<td style="text-align:right;">
21.51218
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
OK
</td>
<td style="text-align:right;">
27.94309
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
OR
</td>
<td style="text-align:right;">
19.47799
</td>
<td style="text-align:right;">
17.16329
</td>
</tr>
<tr>
<td style="text-align:left;">
PA
</td>
<td style="text-align:right;">
22.40980
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
SC
</td>
<td style="text-align:right;">
25.39819
</td>
<td style="text-align:right;">
26.24438
</td>
</tr>
<tr>
<td style="text-align:left;">
SD
</td>
<td style="text-align:right;">
19.14961
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
TX
</td>
<td style="text-align:right;">
29.34137
</td>
<td style="text-align:right;">
29.85299
</td>
</tr>
<tr>
<td style="text-align:left;">
UT
</td>
<td style="text-align:right;">
24.09218
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
VA
</td>
<td style="text-align:right;">
23.18590
</td>
<td style="text-align:right;">
24.68687
</td>
</tr>
<tr>
<td style="text-align:left;">
VT
</td>
<td style="text-align:right;">
17.77719
</td>
<td style="text-align:right;">
21.10825
</td>
</tr>
<tr>
<td style="text-align:left;">
WA
</td>
<td style="text-align:right;">
19.35326
</td>
<td style="text-align:right;">
18.98941
</td>
</tr>
<tr>
<td style="text-align:left;">
WI
</td>
<td style="text-align:right;">
18.58308
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
WV
</td>
<td style="text-align:right;">
21.94820
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
WY
</td>
<td style="text-align:right;">
16.98953
</td>
<td style="text-align:right;">
NA
</td>
</tr>
</tbody>
</table>

    # summary table by temperature classes without elevation stuff
    state_avg_temp <- met_stations %>%
      group_by(STATE) %>%
      summarize(
        num_records = n(),
        num_na = sum(is.na(temp) | is.na(wind.sp) | is.na(atm.press)), 
        num_stations = n_distinct(USAFID),
        temp_mean = mean(temp, na.rm = TRUE),
        wind.sp_mean = mean(wind.sp, na.rm = TRUE),
        atm.press_mean = mean(atm.press, na.rm = TRUE)
      )

    state_avg_temp <- state_avg_temp %>%
      mutate(avg_temp_level = case_when(
        temp_mean < 20 ~ "low",
        temp_mean >= 20 & temp_mean < 25 ~ "mid",
        temp_mean >= 25 ~ "high"
      ))

    summary_table <- state_avg_temp %>%
      group_by(avg_temp_level) %>%
      summarize(
        num_states = n(),
        num_records = sum(num_records),
        num_na = sum(num_na), # there is a lot of na values!! mostly in atm.press
        num_stations = sum(num_stations),
        temp_mean = mean(temp_mean, na.rm = TRUE),
        wind_speed_mean = mean(wind.sp_mean, na.rm = TRUE),
        pressure_mean = mean(atm.press_mean, na.rm = TRUE)
      )

    kable(summary_table, booktabs = TRUE) %>% 
      kable_styling(font_size = 10) %>% 
      kable_paper("hover", full_width = F)

<table class="table lightable-paper lightable-hover" style="font-size: 10px; margin-left: auto; margin-right: auto; font-family: &quot;Arial Narrow&quot;, arial, helvetica, sans-serif; width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
avg\_temp\_level
</th>
<th style="text-align:right;">
num\_states
</th>
<th style="text-align:right;">
num\_records
</th>
<th style="text-align:right;">
num\_na
</th>
<th style="text-align:right;">
num\_stations
</th>
<th style="text-align:right;">
temp\_mean
</th>
<th style="text-align:right;">
wind\_speed\_mean
</th>
<th style="text-align:right;">
pressure\_mean
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
high
</td>
<td style="text-align:right;">
11
</td>
<td style="text-align:right;">
295074
</td>
<td style="text-align:right;">
287194
</td>
<td style="text-align:right;">
148
</td>
<td style="text-align:right;">
27.15919
</td>
<td style="text-align:right;">
2.118478
</td>
<td style="text-align:right;">
1016.196
</td>
</tr>
<tr>
<td style="text-align:left;">
low
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
97831
</td>
<td style="text-align:right;">
97092
</td>
<td style="text-align:right;">
43
</td>
<td style="text-align:right;">
18.09266
</td>
<td style="text-align:right;">
2.584204
</td>
<td style="text-align:right;">
1015.957
</td>
</tr>
<tr>
<td style="text-align:left;">
mid
</td>
<td style="text-align:right;">
20
</td>
<td style="text-align:right;">
241099
</td>
<td style="text-align:right;">
233088
</td>
<td style="text-align:right;">
112
</td>
<td style="text-align:right;">
22.73435
</td>
<td style="text-align:right;">
1.937401
</td>
<td style="text-align:right;">
1015.643
</td>
</tr>
</tbody>
</table>

Knit the document, commit your changes, and push them to GitHub.

## Question 5: Advanced Regression

Let’s practice running regression models with smooth functions on X. We
need the `mgcv` package and `gam()` function to do this.

-   using your data with the median values per station, examine the
    association between median temperature (y) and median wind speed
    (x). Create a scatterplot of the two variables using ggplot2. Add
    both a linear regression line and a smooth line.

-   fit both a linear model and a spline model (use `gam()` with a cubic
    regression spline on wind speed). Summarize and plot the results
    from the models and interpret which model is the best fit and why.

<!-- -->

    library(mgcv)

    ## Loading required package: nlme

    ## 
    ## Attaching package: 'nlme'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     collapse

    ## This is mgcv 1.9-0. For overview type 'help("mgcv-package")'.

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 4.3.3

    # example from lab with atm.press and temp
    station_med_lt <- lazy_dt(station_med)
    station_med_lt <- station_med_lt %>% 
      filter(between(atm.press, 1000, 1020)) %>% 
      collect()

    ggplot(station_med_lt, aes(x=atm.press, y=temp)) +
      geom_point() +
      geom_smooth(method = "lm", col = "cyan") +
      geom_smooth(method = "gam", col = "blue")

    ## `geom_smooth()` using formula = 'y ~ x'

    ## `geom_smooth()` using formula = 'y ~ s(x, bs = "cs")'

![](README_files/figure-markdown_strict/unnamed-chunk-15-1.png)

    lm_mod <- lm(temp~atm.press, data=station_med_lt)
    summary(lm_mod)

    ## 
    ## Call:
    ## lm(formula = temp ~ atm.press, data = station_med_lt)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -11.1478  -0.6383   0.7434   2.3125   5.7682 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 959.8827   242.4687   3.959 0.000352 ***
    ## atm.press    -0.9227     0.2390  -3.860 0.000467 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 3.931 on 35 degrees of freedom
    ## Multiple R-squared:  0.2986, Adjusted R-squared:  0.2786 
    ## F-statistic:  14.9 on 1 and 35 DF,  p-value: 0.000467

    # bs = cr means cubic regression line, k = 20 is 20 degrees of freedom
    gam_mod <- gam(temp~s(atm.press, bs = "cr", k = 20), data = station_med_lt)
    summary(gam_mod)

    ## 
    ## Family: gaussian 
    ## Link function: identity 
    ## 
    ## Formula:
    ## temp ~ s(atm.press, bs = "cr", k = 20)
    ## 
    ## Parametric coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  23.9284     0.6463   37.02   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Approximate significance of smooth terms:
    ##              edf Ref.df    F  p-value    
    ## s(atm.press)   1      1 14.9 0.000467 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## R-sq.(adj) =  0.279   Deviance explained = 29.9%
    ## GCV = 16.338  Scale est. = 15.455    n = 37

    plot(gam_mod)

![](README_files/figure-markdown_strict/unnamed-chunk-16-1.png)

    # looking at wind.sp
    station_med_lt <- lazy_dt(station_med)
    station_med_lt <- station_med_lt %>% 
      collect()

    ggplot(station_med_lt, aes(x=wind.sp, y=temp)) +
      geom_point() +
      geom_smooth(method = "lm", col = "cyan") +
      geom_smooth(method = "gam", col = "blue")

    ## `geom_smooth()` using formula = 'y ~ x'

    ## Warning: Removed 5 rows containing non-finite outside the scale range
    ## (`stat_smooth()`).

    ## `geom_smooth()` using formula = 'y ~ s(x, bs = "cs")'

    ## Warning: Removed 5 rows containing non-finite outside the scale range
    ## (`stat_smooth()`).

    ## Warning: Removed 5 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

![](README_files/figure-markdown_strict/unnamed-chunk-17-1.png)

    lm_mod <- lm(temp~wind.sp, data=station_med_lt)
    summary(lm_mod)

    ## 
    ## Call:
    ## lm(formula = temp ~ wind.sp, data = station_med_lt)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -19.7620  -2.9424   0.1903   2.9841   9.9341 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  23.7739     0.3763  63.185   <2e-16 ***
    ## wind.sp       0.1123     0.1489   0.754    0.451    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 4.316 on 417 degrees of freedom
    ##   (5 observations deleted due to missingness)
    ## Multiple R-squared:  0.001362,   Adjusted R-squared:  -0.001033 
    ## F-statistic: 0.5687 on 1 and 417 DF,  p-value: 0.4512

    # bs = cr means cubic regression line, k = 20 is 20 degrees of freedom
    gam_mod <- gam(temp~s(wind.sp, bs = "cr", k = 20), data = station_med_lt)
    summary(gam_mod)

    ## 
    ## Family: gaussian 
    ## Link function: identity 
    ## 
    ## Formula:
    ## temp ~ s(wind.sp, bs = "cr", k = 20)
    ## 
    ## Parametric coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   24.009      0.197   121.9   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Approximate significance of smooth terms:
    ##             edf Ref.df     F p-value    
    ## s(wind.sp) 15.1  16.55 4.289  <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## R-sq.(adj) =  0.126   Deviance explained = 15.8%
    ## GCV = 16.915  Scale est. = 16.265    n = 419

    plot(gam_mod)

![](README_files/figure-markdown_strict/unnamed-chunk-18-1.png)
