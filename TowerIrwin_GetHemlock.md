# UW ACES 2016: Preparing simple pollen time series of Tsuga canadensis for time series analysis
Jack Williams & Simon Goring
`r format(Sys.time(), '%d %B, %Y')`  

# Goal

Retrieve simple time series of percent Tsuga canadensis from Tower Lake and Irwin Smith, for use in ACES seminar.  Being passed to Tony and Steve for time series statistics module.


# Analytical Steps

## Install and add packages
```r
# Uncomment this line if you haven't already installed any of these packages:
# install.packages(c("neotoma", "analogue"))

#Add the neotoma and analogue packages to your programming environment 
library(neotoma)
library("analogue")
```

## Get site data for Tower and Irwin


```r
#Sept 8, 2016:  this returns one site
tower_site <- get_site(sitename = 'Tower%')
#Sept 8, 2016:  this returns one sites ('Irwin%' returns 2).  
irwin_site <- get_site(sitename = 'Irwin Smith%')
```

## Download the datasets and sample metadata

`get_download` returns a list of download objects, one per dataset.  Each download object contains a suite of data for the samples in that dataset. 

```r
tower_data <- get_download(tower_site)
irwin_data <- get_download(irwin_site)

```
Both Irwin and Tower each have two datasets, one plant macrofossils, one pollen.  Checking to see which dataset is the pollen dataset
```r
#dataset.meta stores metadata for each pollen sample
#Irwin:  dataset 1 is plant macros, 2 is pollen
head(irwin_data)
#Tower:  dataset 1 is plant macros, 2 is pollen
head(tower_data)
```
sample.meta stores metadata for each pollen sample
taxon.list stores a list of taxa found  in the  dataset
counts stores the the counts, presence/absence data, or percentage data for each taxon for each sample

Bind the downloaded pollen datasets together into a single data frame.  'bind' is a function within the neotoma package.
```r
aces_demo_data=bind(irwin_data[[2]], tower_data[[2]])
```

## `compile_taxa`

Compile both sites to a standard list of taxa, using the 'WS64' list from Williams and Shuman 2008. The warning messages return  a number of taxa that cannot be converted using the existing data table.  Check to see which taxa have been converted by looking at the new taxon table.

```r
aces_poll64 <- compile_taxa(aces_demo_data, list.name = "WS64")
aces_poll64$taxon.list[,c("compressed", "taxon.name")]
```
##Convert the pollen data to percentages

Using tran function in analog to do percentages.  To make it clear which functions come from the `analogue` package the code uses `analogue::` before the function names.  This is just an explicit way to state the function source.  
Also converting output to data frame and renaming Tsuga undifferentiated to Tsuga_canadensis in counts table.  At these sites, this  is T. canadensis and we are removing the space.  

```r
i_junk <- analogue::tran(x = aces_poll64[[1]]$counts, method = 'percent')
t_junk <- analogue::tran(x = aces_poll64[[2]]$counts, method = 'percent')
irwin_pct<-data.frame(i_junk)
tower_pct<-data.frame(t_junk)
irwin_pct<-rename(irwin_pct, c("Tsuga.undifferentiated"="Tsuga.canadensis"))
tower_pct<-rename(tower_pct, c("Tsuga.undifferentiated"="Tsuga.canadensis"))
```

Create new data tables with just depths, ages, and hemlock percents. 
Rename column headers

```r
irwin_hemlock<-data.frame(aces_poll64[[1]]$sample.meta$depth,aces_poll64[[1]]$sample.meta$age,irwin_pct$Tsuga.canadensis)
tower_hemlock<-data.frame(aces_poll64[[2]]$sample.meta$depth,aces_poll64[[2]]$sample.meta$age,tower_pct$Tsuga.canadensis)
#Note that 'names' command assumes position of columns.
names(irwin_hemlock)[1]<-"Depth.cm"
names(irwin_hemlock)[2]<-"Age.yrB1950"
names(irwin_hemlock)[3]<-"Tsuga.canadensis"
names(tower_hemlock)[1]<-"Depth.cm"
names(tower_hemlock)[2]<-"Age.yrB1950"
names(tower_hemlock)[3]<-"Tsuga.canadensis"
```
Plot data to confirm
```r
plot(irwin_hemlock$Tsuga.canadensis,irwin_hemlock$Age.yrB1950)
plot(tower_hemlock$Tsuga.canadensis,tower_hemlock$Age.yrB1950)
```

Saving data to an .RData file for sharing and re-use

```r
save(irwin_hemlock,tower_hemlock,file="hemlock_irwin_tower.RData")
save.image()
```
