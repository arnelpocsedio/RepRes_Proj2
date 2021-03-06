---
title: "Reproducible Research: Peer Assessment 2"
author: "Arnel"
date: "12/16/2020"
output: 
    html_document:
        keep_md: true
---


```r
knitr:: opts_chunk$set(echo = TRUE, results = 'asis', message = FALSE, warning = FALSE)
```
## __FLOODS and TORNADOES are the ultimate threat to the US population's health and economy.__
&nbsp;  
&nbsp;  

### __Synopsis__  

We explore the 1950 to 2011 data of the US NOAA Storm Database to study the risk associated with natural calamities. More information about the database can be found be in this [documentation][id]. We start by processing data to be amenable for analysis. This includes formatting the damages more precisely and correctly clasifying the 48 event types, among others. Based on the amount of damages (USD) and number of injuries and fatalities, we report here that floods and tornadoes are the most pressing threat to the US population's health and economy. Overall, more than 150 billion USD of damages were due to floods. Tornados on the other hand caused more than 91 thousand injuries and more than 5 thousand fatalities. Therefore, efforts should be made to try to minimize this risks.   

[id]:https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf   
&nbsp;  
&nbsp;  

### __Data Processing__  
&nbsp;  

<font size="4.5"> _Getting the data and quick overview_</font>   

We first download the data. We then read it into R and name it "storm".

```r
#function to download the data
getdata = function(fileURL) {
        if(!file.exists("./data")){dir.create("./data")}
        
        fileURL = fileURL
        if(!file.exists("./data/dataset.csv.bz2")){
                download.file(fileURL, destfile = "./data/dataset.csv.bz2")
        }
        
}

fileURL = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"

getdata(fileURL) #download the data

storm = read.csv("./data/dataset.csv.bz2", stringsAsFactors = FALSE) #read the data

dim(storm) #check dimensions
```

[1] 902297     37
  
We check first the 37 variables in the data.  
   

```r
library(xtable)
print(xtable(matrix(c(names(storm),"",""), 13, 3)), type = "html")
```

<!-- html table generated in R 3.6.2 by xtable 1.8-4 package -->
<!-- Tue Dec 29 15:26:43 2020 -->
<table border=1>
<tr> <th>  </th> <th> 1 </th> <th> 2 </th> <th> 3 </th>  </tr>
  <tr> <td align="right"> 1 </td> <td> STATE__ </td> <td> COUNTY_END </td> <td> CROPDMG </td> </tr>
  <tr> <td align="right"> 2 </td> <td> BGN_DATE </td> <td> COUNTYENDN </td> <td> CROPDMGEXP </td> </tr>
  <tr> <td align="right"> 3 </td> <td> BGN_TIME </td> <td> END_RANGE </td> <td> WFO </td> </tr>
  <tr> <td align="right"> 4 </td> <td> TIME_ZONE </td> <td> END_AZI </td> <td> STATEOFFIC </td> </tr>
  <tr> <td align="right"> 5 </td> <td> COUNTY </td> <td> END_LOCATI </td> <td> ZONENAMES </td> </tr>
  <tr> <td align="right"> 6 </td> <td> COUNTYNAME </td> <td> LENGTH </td> <td> LATITUDE </td> </tr>
  <tr> <td align="right"> 7 </td> <td> STATE </td> <td> WIDTH </td> <td> LONGITUDE </td> </tr>
  <tr> <td align="right"> 8 </td> <td> EVTYPE </td> <td> F </td> <td> LATITUDE_E </td> </tr>
  <tr> <td align="right"> 9 </td> <td> BGN_RANGE </td> <td> MAG </td> <td> LONGITUDE_ </td> </tr>
  <tr> <td align="right"> 10 </td> <td> BGN_AZI </td> <td> FATALITIES </td> <td> REMARKS </td> </tr>
  <tr> <td align="right"> 11 </td> <td> BGN_LOCATI </td> <td> INJURIES </td> <td> REFNUM </td> </tr>
  <tr> <td align="right"> 12 </td> <td> END_DATE </td> <td> PROPDMG </td> <td>  </td> </tr>
  <tr> <td align="right"> 13 </td> <td> END_TIME </td> <td> PROPDMGEXP </td> <td>  </td> </tr>
   </table>
&nbsp;  

  
We also look at some events in some States with fatalities/injuries as well as those that incurred economic damages, both property and crops.   

```r
#select sample columns and rows to show
cols = c("BGN_DATE" ,"STATE", "EVTYPE", "PROPDMG", "PROPDMGEXP", "CROPDMG", "CROPDMGEXP",  "INJURIES", "FATALITIES")
samplerows <- storm$PROPDMG>1 & storm$PROPDMGEXP == "M" & storm$CROPDMG>1 & storm$CROPDMGEXP == "M" &
        storm$INJURIES>1 & storm$FATALITIES>1

library(xtable)
print(xtable(storm[samplerows, cols]), type = "html")
```

<!-- html table generated in R 3.6.2 by xtable 1.8-4 package -->
<!-- Tue Dec 29 15:26:43 2020 -->
<table border=1>
<tr> <th>  </th> <th> BGN_DATE </th> <th> STATE </th> <th> EVTYPE </th> <th> PROPDMG </th> <th> PROPDMGEXP </th> <th> CROPDMG </th> <th> CROPDMGEXP </th> <th> INJURIES </th> <th> FATALITIES </th>  </tr>
  <tr> <td align="right"> 191345 </td> <td> 1/19/1993 0:00:00 </td> <td> CA </td> <td> WINTER STORM </td> <td align="right"> 5.00 </td> <td> M </td> <td align="right"> 5.00 </td> <td> M </td> <td align="right"> 5.00 </td> <td align="right"> 3.00 </td> </tr>
  <tr> <td align="right"> 196833 </td> <td> 10/5/1995 0:00:00 </td> <td> GA </td> <td> THUNDERSTORM WINDS </td> <td align="right"> 75.00 </td> <td> M </td> <td align="right"> 50.00 </td> <td> M </td> <td align="right"> 7.00 </td> <td align="right"> 8.00 </td> </tr>
  <tr> <td align="right"> 196835 </td> <td> 3/13/1993 0:00:00 </td> <td> GA </td> <td> BLIZZARD </td> <td align="right"> 50.00 </td> <td> M </td> <td align="right"> 50.00 </td> <td> M </td> <td align="right"> 385.00 </td> <td align="right"> 5.00 </td> </tr>
  <tr> <td align="right"> 310082 </td> <td> 4/8/1998 0:00:00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 200.00 </td> <td> M </td> <td align="right"> 2.20 </td> <td> M </td> <td align="right"> 258.00 </td> <td align="right"> 32.00 </td> </tr>
  <tr> <td align="right"> 315331 </td> <td> 11/4/1998 0:00:00 </td> <td> FL </td> <td> TROPICAL STORM </td> <td align="right"> 30.00 </td> <td> M </td> <td align="right"> 20.00 </td> <td> M </td> <td align="right"> 65.00 </td> <td align="right"> 2.00 </td> </tr>
  <tr> <td align="right"> 347479 </td> <td> 11/10/1998 0:00:00 </td> <td> WI </td> <td> HIGH WIND </td> <td align="right"> 10.36 </td> <td> M </td> <td align="right"> 1.58 </td> <td> M </td> <td align="right"> 14.00 </td> <td align="right"> 4.00 </td> </tr>
  <tr> <td align="right"> 384271 </td> <td> 2/13/2000 0:00:00 </td> <td> GA </td> <td> TORNADO </td> <td align="right"> 20.00 </td> <td> M </td> <td align="right"> 2.00 </td> <td> M </td> <td align="right"> 175.00 </td> <td align="right"> 11.00 </td> </tr>
  <tr> <td align="right"> 384278 </td> <td> 2/14/2000 0:00:00 </td> <td> GA </td> <td> TORNADO </td> <td align="right"> 3.50 </td> <td> M </td> <td align="right"> 3.00 </td> <td> M </td> <td align="right"> 15.00 </td> <td align="right"> 6.00 </td> </tr>
  <tr> <td align="right"> 487299 </td> <td> 1/5/2003 0:00:00 </td> <td> CA </td> <td> HIGH WIND </td> <td align="right"> 3.28 </td> <td> M </td> <td align="right"> 28.00 </td> <td> M </td> <td align="right"> 11.00 </td> <td align="right"> 2.00 </td> </tr>
  <tr> <td align="right"> 488046 </td> <td> 10/26/2003 0:00:00 </td> <td> CA </td> <td> WILDFIRE </td> <td align="right"> 55.22 </td> <td> M </td> <td align="right"> 22.00 </td> <td> M </td> <td align="right"> 18.00 </td> <td align="right"> 2.00 </td> </tr>
  <tr> <td align="right"> 503070 </td> <td> 5/4/2003 0:00:00 </td> <td> MO </td> <td> TORNADO </td> <td align="right"> 40.00 </td> <td> M </td> <td align="right"> 3.00 </td> <td> M </td> <td align="right"> 37.00 </td> <td align="right"> 3.00 </td> </tr>
  <tr> <td align="right"> 639436 </td> <td> 3/12/2006 0:00:00 </td> <td> TX </td> <td> WILDFIRE </td> <td align="right"> 49.90 </td> <td> M </td> <td align="right"> 45.40 </td> <td> M </td> <td align="right"> 8.00 </td> <td align="right"> 12.00 </td> </tr>
  <tr> <td align="right"> 800635 </td> <td> 4/24/2010 0:00:00 </td> <td> MS </td> <td> TORNADO </td> <td align="right"> 90.00 </td> <td> M </td> <td align="right"> 6.00 </td> <td> M </td> <td align="right"> 35.00 </td> <td align="right"> 5.00 </td> </tr>
  <tr> <td align="right"> 800792 </td> <td> 4/24/2010 0:00:00 </td> <td> MS </td> <td> TORNADO </td> <td align="right"> 140.00 </td> <td> M </td> <td align="right"> 4.00 </td> <td> M </td> <td align="right"> 53.00 </td> <td align="right"> 4.00 </td> </tr>
   </table>
&nbsp;  


We saw that the damages data in the example has letter "M"  in PROPDMGEXP or CROPDMGEXP which stands for millions. So the first row tells us that a winter storm in California devasted 5 million USD each in property and crop damages. So that we can compare more properly, we will convert the damage data into more exact values. In the [documentation][id], the letters: "K", "M" and "B" represent thousand, millions and billions respectively. For our purpose we will ignore unknown and unclear damage values. 
&nbsp;  
&nbsp;  

<font size="4.5"> _Correcting data on "damages"_</font>   
  

```r
unique(storm$PROPDMGEXP) #check letters and other values in PRODMGEXP and CROPDMGEXP
```

 [1] "K" "M" ""  "B" "m" "+" "0" "5" "6" "?" "4" "2" "3" "h" "7" "H" "-" "1" "8"

```r
unique(storm$CROPDMGEXP) 
```

[1] ""  "M" "K" "m" "B" "?" "0" "k" "2"

```r
#we will convert B, M, K and also H (for hundreds) into multipliers
storm$PROPDMGEXP2 = ifelse(storm$PROPDMGEXP == "B" | storm$PROPDMGEXP == "b", 1000000000,
                           ifelse(storm$PROPDMGEXP == "M" | storm$PROPDMGEXP == "m", 1000000,
                                  ifelse(storm$PROPDMGEXP == "K" | storm$PROPDMGEXP == "k", 1000,
                                         ifelse(storm$PROPDMGEXP == "H" | storm$PROPDMGEXP == "h", 100,
                                                NA 
                                                )
                                         )
                                )
                           )


storm$CROPDMGEXP2 = ifelse(storm$CROPDMGEXP == "B" | storm$CROPDMGEXP == "b", 1000000000,
                           ifelse(storm$CROPDMGEXP == "M" | storm$CROPDMGEXP == "m", 1000000,
                                  ifelse(storm$CROPDMGEXP == "K" | storm$CROPDMGEXP == "k", 1000,
                                         ifelse(storm$CROPDMGEXP == "H" | storm$CROPDMGEXP == "h", 100,
                                                NA 
                                                )
                                         )
                                )
                           )

#We create new column for damage data
storm$PROPDMG2 = storm$PROPDMG*storm$PROPDMGEXP2
storm$CROPDMG2 = storm$CROPDMG*storm$CROPDMGEXP2
sum(is.na(storm$PROPDMG2))#check ambigous values (i.e not with B, M, K or H)
```

[1] 466248

```r
sum(is.na(storm$CROPDMG2))
```

[1] 618440

```r
#we reassign NA into zero for the total damage computation
storm$PROPDMG2[is.na(storm$PROPDMG2)] <- 0
storm$CROPDMG2[is.na(storm$CROPDMG2)] <- 0
storm$TOTALDMG = storm$PROPDMG2 + storm$CROPDMG2 

#we also compute total of injuries and fatalities
storm$TOTAL_INJUR_FATAL = storm$INJURIES + storm$FATALITIES 
```
&nbsp;  


<font size="4.5"> _Data subsetting_</font>   
  
For our purpose, we are only interested with the data on the effects of the different natural calamities on economic and popultion health damages. Thus, we create smaller subset of the data with relevant rows and columns, we call this storm2.   

```r
#we select relevant columns only
cols2 = c("BGN_DATE" ,"STATE", "EVTYPE", "PROPDMG2",  "CROPDMG2", "TOTALDMG", "INJURIES", "FATALITIES", "TOTAL_INJUR_FATAL")
storm2 = storm[, cols2]

#give better names for the variables
names(storm2)[c(4:6,9)] = c("PROPERTY_DAMAGE", "CROP_DAMAGE", "TOTAL_DAMAGE", "TOTAL_INJURIES_FATALITIES")

#we remove rows with no relevant data
storm2 = storm2[storm2$TOTAL_DAMAGE>0 | storm2$TOTAL_INJURIES_FATALITIES>0,]


#we also format the date column
library(lubridate)
storm2$BGN_DATE = mdy_hms(storm2$BGN_DATE)

print(xtable(storm2[1:10,]), type = "html")
```

<!-- html table generated in R 3.6.2 by xtable 1.8-4 package -->
<!-- Tue Dec 29 15:26:45 2020 -->
<table border=1>
<tr> <th>  </th> <th> BGN_DATE </th> <th> STATE </th> <th> EVTYPE </th> <th> PROPERTY_DAMAGE </th> <th> CROP_DAMAGE </th> <th> TOTAL_DAMAGE </th> <th> INJURIES </th> <th> FATALITIES </th> <th> TOTAL_INJURIES_FATALITIES </th>  </tr>
  <tr> <td align="right"> 1 </td> <td align="right"> -621907200.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 25000.00 </td> <td align="right"> 0.00 </td> <td align="right"> 25000.00 </td> <td align="right"> 15.00 </td> <td align="right"> 0.00 </td> <td align="right"> 15.00 </td> </tr>
  <tr> <td align="right"> 2 </td> <td align="right"> -621907200.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 2500.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2500.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> </tr>
  <tr> <td align="right"> 3 </td> <td align="right"> -595296000.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 25000.00 </td> <td align="right"> 0.00 </td> <td align="right"> 25000.00 </td> <td align="right"> 2.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2.00 </td> </tr>
  <tr> <td align="right"> 4 </td> <td align="right"> -585964800.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 2500.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2500.00 </td> <td align="right"> 2.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2.00 </td> </tr>
  <tr> <td align="right"> 5 </td> <td align="right"> -572140800.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 2500.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2500.00 </td> <td align="right"> 2.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2.00 </td> </tr>
  <tr> <td align="right"> 6 </td> <td align="right"> -572140800.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 2500.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2500.00 </td> <td align="right"> 6.00 </td> <td align="right"> 0.00 </td> <td align="right"> 6.00 </td> </tr>
  <tr> <td align="right"> 7 </td> <td align="right"> -572054400.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 2500.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2500.00 </td> <td align="right"> 1.00 </td> <td align="right"> 0.00 </td> <td align="right"> 1.00 </td> </tr>
  <tr> <td align="right"> 8 </td> <td align="right"> -566265600.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 2500.00 </td> <td align="right"> 0.00 </td> <td align="right"> 2500.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> </tr>
  <tr> <td align="right"> 9 </td> <td align="right"> -564364800.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 25000.00 </td> <td align="right"> 0.00 </td> <td align="right"> 25000.00 </td> <td align="right"> 14.00 </td> <td align="right"> 1.00 </td> <td align="right"> 15.00 </td> </tr>
  <tr> <td align="right"> 10 </td> <td align="right"> -564364800.00 </td> <td> AL </td> <td> TORNADO </td> <td align="right"> 25000.00 </td> <td align="right"> 0.00 </td> <td align="right"> 25000.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> </tr>
   </table>
&nbsp;  


<font size="4.5"> _Processing the "event types"_</font>   
   
From the [documentation][id], we know that there are 48 event types. In our data there are more types so we group them properly into the 48. To do this we standarize the strings of the event types and we utilize regular expression in R.   

```r
length(unique(storm2$EVTYPE)) #more than 48 event types
```

[1] 484

```r
head(unique(storm2$EVTYPE)) #check some event type values
```

[1] "TORNADO"                   "TSTM WIND"                
[3] "HAIL"                      "ICE STORM/FLASH FLOOD"    
[5] "WINTER STORM"              "HURRICANE OPAL/HIGH WINDS"

```r
tail(unique(storm2$EVTYPE))
```

[1] "MARINE THUNDERSTORM WIND" "MARINE STRONG WIND"      
[3] "ASTRONOMICAL LOW TIDE"    "DENSE SMOKE"             
[5] "MARINE HAIL"              "FREEZING FOG"            

```r
#convert strings to all lower caps, no space in at the start
storm2$EVTYPE2 = tolower(storm2$EVTYPE)

#remove spaces and "/"
storm2$EVTYPE2 = gsub("[[:space:]]", "", storm2$EVTYPE2)
storm2$EVTYPE2 = gsub("/", "", storm2$EVTYPE2)

#rename tstm as thunderstorm
storm2$EVTYPE2 = gsub("tstm", "thunderstorm", storm2$EVTYPE2)

#check new strings of the event types
head(unique(storm2$EVTYPE2))
```

[1] "tornado"                "thunderstormwind"       "hail"                  
[4] "icestormflashflood"     "winterstorm"            "hurricaneopalhighwinds"

```r
tail(unique(storm2$EVTYPE2))
```

[1] "lakeshoreflood"      "marinestrongwind"    "astronomicallowtide"
[4] "densesmoke"          "marinehail"          "freezingfog"        

   
After a bit of standardization, we can now search for key words to classify the different event types.   

```r
#hail and marine hail, Events 1 and 2
unique(grep(".*hail.*", storm2$EVTYPE2, value = TRUE)) #searching for "hail" in the EVTYPE2 variable
hails = unique(grep(".*hail.*", storm2$EVTYPE2, value = TRUE))
hails = hails[c(1,4:12,14,16:21)] #remove those with more than 1 event or ambigous
 
#sleet and ice storm, E3 and E4
unique(grep(".*sleet.*", storm2$EVTYPE2, value = TRUE))
sleets = unique(grep(".*sleet.*", storm2$EVTYPE2, value = TRUE))
sleets = sleets[c(1,4)]

unique(grep(".*ice.*storm.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*freez.*rain.*", storm2$EVTYPE2, value = TRUE))
ice1 = unique(grep(".*ice.*storm.*", storm2$EVTYPE2, value = TRUE))
fzrain = unique(grep(".*freez.*rain.*", storm2$EVTYPE2, value = TRUE))
icestorms = c(ice1[c(2,4:6)], fzrain[c(1,2,5,7)]) #"freezing rain" is also called "ice storm"

#seiche E5
unique(grep(".*seiche.*", storm2$EVTYPE2, value = TRUE)) #one unique value, need  not to be corrected

#blizzard E6
unique(grep(".*blizzard.*", storm2$EVTYPE2, value = TRUE))
blizzards = unique(grep(".*blizzard.*", storm2$EVTYPE2, value = TRUE))
blizzards = blizzards[c(1,2)]

#avalanche E7
unique(grep(".*avalanche.*", storm2$EVTYPE2, value = TRUE))

#heavy snow and lake-effect snow, E8 and E9
unique(grep(".*snow.*", storm2$EVTYPE2, value = TRUE))
snows = unique(grep(".*snow.*", storm2$EVTYPE2, value = TRUE))
heavysnows = snows[c(1:3,7:9,18,19,21,24,26,30,39,41,47)]
lakesnows = grep(".*lake.*", snows, value = TRUE)

#frost freeze E10
unique(grep(".*frost.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*freeze.*", storm2$EVTYPE2, value = TRUE))
frosts = unique(grep(".*frost.*", storm2$EVTYPE2, value = TRUE))
freezes = unique(grep(".*freeze.*", storm2$EVTYPE2, value = TRUE))
frostfreezes = c(frosts, freezes)
frostfreezes = unique(frostfreezes)

#winter weather E11
unique(grep(".*winter.*", storm2$EVTYPE2, value = TRUE))
winterweathers = unique(grep(".*winter.*", storm2$EVTYPE2, value = TRUE))
winterweathers = winterweathers[c(1,3,7)]

#heat and excessive heat, E12 and E13
unique(grep(".*heat.*", storm2$EVTYPE2, value = TRUE))
heat1 = unique(grep(".*heat.*", storm2$EVTYPE2, value = TRUE))
heats = heat1[c(1,3,8)] 
exheats = heat1[c(2,4,6,9)] 

#drought E14
unique(grep(".*drought.*", storm2$EVTYPE2, value = TRUE))

#wildfire E15
unique(grep(".*wild.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*fire.*", storm2$EVTYPE2, value = TRUE))
wildfires = unique(grep(".*fire.*", storm2$EVTYPE2, value = TRUE))[c(1:3,5:8)]


#floods E16, E17, E18 and E19
unique(grep(".*flood.*", storm2$EVTYPE2, value = TRUE))
floods1 = unique(grep(".*flood.*", storm2$EVTYPE2, value = TRUE))

flashfloods = grep(".*flash.*",floods1, value = TRUE)
flashfloods
flashfloods = flashfloods[c(2,3,5:7,9:14)]

coastalfloods = grep(".*coastal.*",floods1, value = TRUE)
coastalfloods
coastalfloods = c(coastalfloods[c(1,3,5)],"erosioncstl flood")

lakefloods = grep(".*lake.*",floods1, value = TRUE)
lakefloods

floods = floods1[!(floods1 %in% c(flashfloods, coastalfloods, lakefloods))]
floods
floods = floods[c(1,3,6:16,18:20,25,28,29)]

#tornado E20
unique(grep(".*tornado.*", storm2$EVTYPE2, value = TRUE))
tornadoes = unique(grep(".*tornado.*", storm2$EVTYPE2, value = TRUE))
tornadoes = tornadoes[c(1,2,6:10)]

#tsunami E21
unique(grep(".*tsunami.*", storm2$EVTYPE2, value = TRUE))

#waterspout E22
unique(grep(".*water.*spout.*", storm2$EVTYPE2, value = TRUE))
waterspouts = unique(grep(".*water.*spout.*", storm2$EVTYPE2, value = TRUE))
waterspouts = waterspouts[c(1,3)]

#surf E23
unique(grep(".*surf.*", storm2$EVTYPE2, value = TRUE))
highsurfs = unique(grep(".*surf.*", storm2$EVTYPE2, value = TRUE))
highsurfs = highsurfs[c(2,3,5,7:10)]

#rip current E24
unique(grep(".*rip.*current.*", storm2$EVTYPE2, value = TRUE))
ripcurrents = unique(grep(".*rip.*current.*", storm2$EVTYPE2, value = TRUE))
ripcurrents = ripcurrents[c(1,3)]

#dense fog and freezing fog E25 and E26
unique(grep(".*fog.*", storm2$EVTYPE2, value = TRUE))

#wind E27 to E34
unique(grep(".*wind.*", storm2$EVTYPE2, value = TRUE))
winds = unique(grep(".*wind.*", storm2$EVTYPE2, value = TRUE))

thunderstormwinds = grep(".*thunder.*", winds, value = TRUE)
thunderstormwinds 
thunderstormwinds = thunderstormwinds[c(1,2,6:8,11:15,17:34,37,39:44,46,47)]

highwinds = grep(".*high.*", winds, value = TRUE)
highwinds
highwinds = highwinds[c(2,3,5,8,15:20)]

strongwinds = grep(".*strong.*", winds, value = TRUE)
strongwinds
strongwinds = strongwinds[c(1:3)]

grep(".*cold.*", winds, value = TRUE)
grep(".*chill.*", winds, value = TRUE)
coldwindchills = grep(".*cold.*", winds, value = TRUE)[c(2,4)]

grep(".*extreme.*", winds, value = TRUE)
extremecoldwindchills = grep(".*extreme.*", winds, value = TRUE)

#marine high/strong/thunderstorm winds
grep(".*marine.*", winds, value = TRUE)


#lightning E35
unique(grep(".*lightning.*", storm2$EVTYPE2, value = TRUE))
lightnings = unique(grep(".*lightning.*", storm2$EVTYPE2, value = TRUE))
lightnings = lightnings[c(1,7,10:12)]

#dust E36 and E37
unique(grep(".*dust.*", storm2$EVTYPE2, value = TRUE))

#rain E38
unique(grep(".*rain.*", storm2$EVTYPE2, value = TRUE))
rains = unique(grep(".*rain.*", storm2$EVTYPE2, value = TRUE))
heavyrains = rains[c(1,2,8,9,12,15,18,25,27,29)]

#debris flow E39
unique(grep(".*debris.*flow.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*debris.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".flow.*", storm2$EVTYPE2, value = TRUE))

#dense smoke E40
unique(grep(".*smoke.*", storm2$EVTYPE2, value = TRUE))

#funnel cloud E41
unique(grep(".*funnel.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*cloud.*", storm2$EVTYPE2, value = TRUE))

#volcanic ash E42
unique(grep(".*volcan.*ash.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*volcan.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*ash.*", storm2$EVTYPE2, value = TRUE))

#other "storms"
storms = unique(grep(".*storm.*", storm2$EVTYPE2, value = TRUE))

#winter storm E43
winterstorms = grep(".*winter.*", storms, value = TRUE)
winterstorms
winterstorms = winterstorms[c(1,3)]

#tropical storm E44
tropicalstorms = grep(".*tropical.*", storms, value = TRUE)
tropicalstorms

#storm surge tide E45
grep(".*tid.*", storms, value = TRUE)
grep(".*surge.*", storms, value = TRUE)
stormsurgetides = grep(".*surge.*", storms, value = TRUE)

#hurricane typhoon E46
unique(grep(".*hurricane.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*typhoon.*", storm2$EVTYPE2, value = TRUE))
hurricanes = c(unique(grep(".*hurricane.*", storm2$EVTYPE2, value = TRUE)), "typhoon")

#tropical depression E47
unique(grep(".*tropical.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*depression.*", storm2$EVTYPE2, value = TRUE))
tropicaldepressions = unique(grep(".*tropical.*", storm2$EVTYPE2, value = TRUE))

#astronomical low tide E48
unique(grep(".*astronomical.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*low.*", storm2$EVTYPE2, value = TRUE))
unique(grep(".*tid.*", storm2$EVTYPE2, value = TRUE))
```

   
After we look for keywords of event types, we now group them by assign a common event type label.   
   

```r
#reassign event types into common labels
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% hails, "hail", storm2$EVTYPE2) #1
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% sleets, "sleet", storm2$EVTYPE2) #2
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% icestorms, "icestorm", storm2$EVTYPE2) #3
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% blizzards, "blizzard", storm2$EVTYPE2) #4
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% heavysnows, "heavysnow", storm2$EVTYPE2) #5
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% lakesnows, "lakeeffectsnow", storm2$EVTYPE2) #6
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% frostfreezes, "frostfreezes", storm2$EVTYPE2) #7
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% winterweathers, "winterweather", storm2$EVTYPE2) #8

storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% heats, "heat", storm2$EVTYPE2) #9
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% exheats, "excessiveheat", storm2$EVTYPE2) #10
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% wildfires, "wildfire", storm2$EVTYPE2) #11

storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% flashfloods, "flashflood", storm2$EVTYPE2) #12
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% coastalfloods, "coastalflood", storm2$EVTYPE2) #13
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% lakefloods, "lakeshoreflood", storm2$EVTYPE2) #14
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% floods, "flood", storm2$EVTYPE2) #15

storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% tornadoes, "tornado", storm2$EVTYPE2) #16
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% waterspouts, "waterspout", storm2$EVTYPE2) #17
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% highsurfs, "highsurf", storm2$EVTYPE2) #18
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% ripcurrents, "ripcurrent", storm2$EVTYPE2) #19

storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% thunderstormwinds, "thunderstormwind", storm2$EVTYPE2) #20
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% highwinds, "highwinds", storm2$EVTYPE2) #21
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% strongwinds, "strongwind", storm2$EVTYPE2) #22
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% coldwindchills, "coldwindchill", storm2$EVTYPE2) #23
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% extremecoldwindchills, "extremecoldwindchill",
                        storm2$EVTYPE2) #24

storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% lightnings, "lightning", storm2$EVTYPE2) #25
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% heavyrains, "heavyrain", storm2$EVTYPE2) #26

storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% winterstorms, "winterstorm", storm2$EVTYPE2) #27
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% tropicalstorms, "tropicalstorm", storm2$EVTYPE2) #28
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% stormsurgetides, "stormsurgetide", storm2$EVTYPE2) #29
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% hurricanes, "hurricanetyphoon", storm2$EVTYPE2) #30
storm2$EVTYPE2 = ifelse(storm2$EVTYPE2 %in% tropicaldepressions, "tropicaldepression", storm2$EVTYPE2) #31

#Some event types need not to be changed (i.e. the event types are clearly stated). 
#These were:
#32 marine hail; 33 seiche; 34 avalanche; 35 drought; 36 tsunami; 37 dense fog; 38 freezing fog
#39 marine high wind; 40 marine strong wind; 41 marine thunderstorm winds
#42 dust devil; 43 dust storm; 44 debris flow; 45 dense smoke; 46 funnel cloud
#47 volcanic ash; 48 astronomical low tide; 
```
&nbsp;  


<font size="4.5"> _Final dataset_</font>   
    
We create the final dataset and call it "storm3". We use partial matching to consider minor typhographical errors. We first load the names of the 48 event types.   
   

```r
#load the names of the 48 events
EVTYPE48 = c("Astronomical Low Tide", "Avalanche", "Blizzard", "Coastal Flood", "Cold Wind Chill", 
             "Debris Flow", "Dense Fog", "Dense Smoke", "Drought", "Dust Devil", "Dust Storm",
             "Excessive Heat", "Extreme Cold Wind Chill", "Flash Flood", "Flood", "Frost Freeze",
             "Funnel Cloud", "Freezing Fog", "Hail", "Heat", "Heavy Rain", "Heavy Snow", "High Surf",
             "High Wind", "Hurricane Typhoon", "Ice Storm", "Lake Effect Snow", "Lakeshore Flood",
             "Lightning", "Marine Hail", "Marine High Wind", "Marine Strong Wind", 
             "Marine Thunderstorm Wind", "Rip Current", "Seiche", "Sleet", "Storm Surge Tide", 
             "Strong Wind", "Thunderstorm Wind", "Tornado", "Tropical Depression", "Tropical Storm",
             "Tsunami", "Volcanic Ash", "Waterspout", "Wildfire", "Winter Storm", "Winter Weather")

EVTYPE48B = tolower(EVTYPE48) #convert lowercase
EVTYPE48B = gsub("[[:space:]]", "", EVTYPE48) #remove spaces

#We search for other events with approximate match (minor typhograpical errors), string difference of at most 3
library(stringdist)
storm2$EVTYPE3 = amatch(storm2$EVTYPE2, EVTYPE48B, maxDist = 7) 

#use correct names of the events
EVTYPE48 = cbind(NCODE = 1:48, EVTYPE48) 
storm3 = merge(storm2, EVTYPE48, by.x = "EVTYPE3", by.y = "NCODE", all.x = TRUE, sort = FALSE)

#percent of the event types unclassified
sum(is.na(storm3$EVTYPE48)/254331)*100
```

[1] 0.4596372

```r
#We were not able to classify less than 0.5% of the events distinctly into 48 types. For our purpose, this is good enough to draw some summaries. Majority of these are likely multiple events observed all together within the same time period. 

storm3$EVTYPE2 = NULL #remove unnecessary variables
storm3$EVTYPE3 = NULL

names(storm3)[10] = "EVENT_TYPE" #proper name
```
&nbsp;  
&nbsp;  
&nbsp;  
   
### __Results__   
   
The objective of this study is to find out the most devastating natural calamities. We can check first the calamity that caused the most harm through the years.

```r
#maximum values of damages, injuries and fatalities
max(storm3$PROPERTY_DAMAGE)
```

[1] 1.15e+11

```r
max(storm3$CROP_DAMAGE)
```

[1] 5e+09

```r
max(storm3$INJURIES)
```

[1] 1700

```r
max(storm3$FATALITIES)
```

[1] 583

```r
storm3B = storm3[c(which.max(storm3$PROPERTY_DAMAGE), which.max(storm3$CROP_DAMAGE), which.max(storm3$INJURIES), which.max(storm3$FATALITIES)),]

storm3B$"PROPERTY_DAMAGE (millions)" = storm3B$PROPERTY_DAMAGE/1000000
storm3B$"CROP_DAMAGE (millions)" = storm3B$CROP_DAMAGE/1000000
storm3B$"TOTAL_DAMAGE (millions)" = storm3B$TOTAL_DAMAGE/1000000

print(xtable(storm3B[,c(1,2,10:13,7:9)]), type = "html")
```

<!-- html table generated in R 3.6.2 by xtable 1.8-4 package -->
<!-- Tue Dec 29 15:26:45 2020 -->
<table border=1>
<tr> <th>  </th> <th> BGN_DATE </th> <th> STATE </th> <th> EVENT_TYPE </th> <th> PROPERTY_DAMAGE (millions) </th> <th> CROP_DAMAGE (millions) </th> <th> TOTAL_DAMAGE (millions) </th> <th> INJURIES </th> <th> FATALITIES </th> <th> TOTAL_INJURIES_FATALITIES </th>  </tr>
  <tr> <td align="right"> 191257 </td> <td align="right"> 1136073600.00 </td> <td> CA </td> <td> Flood </td> <td align="right"> 115000.00 </td> <td align="right"> 32.50 </td> <td align="right"> 115032.50 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> </tr>
  <tr> <td align="right"> 220865 </td> <td align="right"> 746755200.00 </td> <td> IL </td> <td> Flash Flood </td> <td align="right"> 5000.00 </td> <td align="right"> 5000.00 </td> <td align="right"> 10000.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> </tr>
  <tr> <td align="right"> 23416 </td> <td align="right"> 292550400.00 </td> <td> TX </td> <td> Tornado </td> <td align="right"> 250.00 </td> <td align="right"> 0.00 </td> <td align="right"> 250.00 </td> <td align="right"> 1700.00 </td> <td align="right"> 42.00 </td> <td align="right"> 1742.00 </td> </tr>
  <tr> <td align="right"> 235646 </td> <td align="right"> 805507200.00 </td> <td> IL </td> <td> Heat </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> <td align="right"> 0.00 </td> <td align="right"> 583.00 </td> <td align="right"> 583.00 </td> </tr>
   </table>
&nbsp;  


In terms of damages, the flood in California on 2006 caused the greatest damage at about 115 billion USD while the ice storm in Mississippi on 1994 caused the most crop damage at about 5 billion USD. The tornado in Texas on 1979 lead to most injuries, that is, 1700 people. More devasting is the heat wave in Illinois on 1995 which claimed 583 lives.
&nbsp;  
   
We now check the cumulative effects of these natural calamities. We look at first economic damages.   

```r
#summarize by total
cumul_propdmg = aggregate(PROPERTY_DAMAGE ~ EVENT_TYPE, data = storm3, function(x) sum(x, na.rm = TRUE))
cumul_cropdmg = aggregate(CROP_DAMAGE ~ EVENT_TYPE, data = storm3, function(x) sum(x, na.rm = TRUE))
cumul_totaldmg = aggregate(TOTAL_DAMAGE ~ EVENT_TYPE, data = storm3, function(x) sum(x, na.rm = TRUE))

cumul_dmg = merge(cumul_propdmg, cumul_cropdmg, by = "EVENT_TYPE")
cumul_dmg = merge(cumul_dmg, cumul_totaldmg, by = "EVENT_TYPE")

cumul_dmg2 = cumul_dmg[order(cumul_dmg$TOTAL_DAMAGE, decreasing = TRUE)[1:10],1:3] #top ten only

library(tidyr)
cumul_dmg2 = gather(cumul_dmg2, DAMAGE_TYPE, DAMAGE, PROPERTY_DAMAGE:CROP_DAMAGE, factor_key=TRUE)

#plot for comparison
library(ggplot2)
p1 = ggplot(data = cumul_dmg2, aes(x = EVENT_TYPE, y = DAMAGE/1000000, fill = DAMAGE_TYPE)) + 
    geom_bar(stat = 'identity') + coord_flip()
p1 = p1 + labs(title = "Ten most damaging natural calamities in the US", 
    y = "Damage in millions USD", x = "Event type")
p1 = p1 + geom_text(aes(label=round(DAMAGE/1000000)), size=2,
                    position = position_dodge(width = 0.5), hjust = -0.1)
p1 = p1 + theme(legend.position = "bottom", legend.title = element_text(size = 8),
                legend.text = element_text(size = 6))

p1
```

<img src="RepRes_files/figure-html/unnamed-chunk-11-1.png" width="100%" />

   
We see that __floods negatively caused the greatest economic consequence overall at about 145 billion USD__. This is followed by Hurricane or typhoon (~ 85 billion USD), tornado (almost 57 billion USD), storm suge tides (almost 48 billion USD) and flash floods (~22 billion USD). In terms of crop damage, drought contributed the most at almost 14 billion USD followed by flash floods (more than 6 billion USD) and floods (~ 5.8 billion USD).
&nbsp;  
   
Next, we look at injuries and fatalities.   

```r
#summarize by total
cumul_injury = aggregate(INJURIES ~ EVENT_TYPE, data = storm3, function(x) sum(x, na.rm = TRUE))
cumul_fatality = aggregate(FATALITIES ~ EVENT_TYPE, data = storm3, function(x) sum(x, na.rm = TRUE))

cumul_injury2 = cumul_injury[order(cumul_injury$INJURIES, decreasing = TRUE)[1:10],] #top ten only
cumul_fatality2 = cumul_fatality[order(cumul_fatality$FATALITIES, decreasing = TRUE)[1:10],] #top ten only


#plots for comparison
p2 = ggplot(data = cumul_injury2, aes(x = EVENT_TYPE, y = INJURIES)) + 
    geom_bar(stat = 'identity') + coord_flip()
p2 = p2 + labs(title = "Top 10 natural calamities that caused the most injuries in the US", 
    y = "Injury count", x = "Event type")
p2 = p2 + geom_text(aes(label=INJURIES), size=2,
                    position = position_dodge(width = 0.5), hjust = -0.1)
p2


p3 = ggplot(data = cumul_fatality2, aes(x = EVENT_TYPE, y = FATALITIES)) + 
    geom_bar(stat = 'identity') + coord_flip()
p3 = p3 + labs(title = "\n\n\n\n Top 10 natural calamities that caused the most deaths in the US", 
    y = "Fatality count", x = "Event type")
p3 = p3 + geom_text(aes(label=FATALITIES), size=2,
                    position = position_dodge(width = 0.5), hjust = -0.1)
p3
```

<img src="RepRes_files/figure-html/figures-side-1.png" width="90%" /><img src="RepRes_files/figure-html/figures-side-2.png" width="90%" />

   
__In general, tornado is the most harmful storm event to the US population's health__. It recorded more than 91 thousand injuries since 1950 up to 2011. This is followed by thunderstorm winds (9509), floods (7910), excessive heat (6730) and lightning (5231). Similarly tornadoes killed a lot of people, that is, 5638. This is followed by heat waves (3132 total for heat and excessive heat), flash floods (1037) and lightning (818).
&nbsp;  
&nbsp;  
&nbsp;  
    
In summary, floods, hurricanes or typhoons and tornadoes are the most damaging storm events in terms of econmomic consequence. On the other hand, tornadoes, heat waves and floods (both floods and flashfloods) are the most harmul storm events for the US population's health. Floods and tornadoes therefore, pose the most threat to the US and risk management of these events should be prioritize, among others.   
