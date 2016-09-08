# UW ACES 2016: Getting Tsuga canadensis pollen percentage data
Jack Williams   
`r format(Sys.time(), '%d %B, %Y')`  

# Goal

Retrieve simple time series of percent Tsuga canadensis from Tower Lake and Irwin Smith, for use in time series analysis

# The `neotoma` Package


```r
# Uncomment this line if you haven't already installed any of these packages:
# install.packages(c("neotoma", "analogue", "rworldmap"))

#Add the neotoma package to your programming environment 
library(neotoma)
```


## Getting site data for Tower and Irwin


```r
#Sept 8, 2016:  this returns one site
tower_site <- get_site(sitename = 'Tower%')
#Sept 8, 2016:  this returns one sites ('Irwin%' returns 2).  
irwin_site <- get_site(sitename = 'Irwin Smith%')
```

## Download the datasets and sample metadata (get_downloads)

`get_download` returns a list of download objects, one per dataset.  Each download object contains a suite of data for the samples in that dataset. 


```r
tower_data <- get_download(tower_site)
irwin_data <- get_download(irwin_site)

```
2 datasets in Irwin and 8 in Tower.  Checking to see which dataset is the pollen dataset
```r
#dataset.meta stores metadata for each pollen sample
#Irwin:  dataset 1 is plant macros, 2 is pollen
head(irwin_data)
#Tower:  dataset 1 is plant macros, 2 is pollen
head(tower_data)
```

```r
#sample.meta stores metadata for each pollen sample
#Accessing the second dataset to look at pollen dataset instead of macrofossil dataset
head(irwin_data[[2]]$sample.meta)
```

```r
#taxon.list stores a list of taxa found  in the  dataset
head(irwin_data[[2]]$taxon.list)
```

```r
#counts stores the the counts, presence/absence data, or percentage data for each taxon for each sample
head(irwin_data[[2]]$counts)
head(tower_data[[2]]$counts)
```

## Helper functions

### `compile_taxa`

The level of taxonomic resolution can vary among analysts.  Often for multi-site analyses it is helpful to aggregate to a common taxonomic resolution. The `compile_taxa` function in `neotoma` will do this.  To help support rapid prototyping, `neotoma` includes a few pre-built taxonomic lists, so far mostly for North American pollen types. **However**, the function also supports the use of a custom-built `data.frame` for aligning taxonomies.  Because new taxa are added to Neotoma regularly (based on analyst identification), it is worthwhile to check the assignments performed by the `compile_taxa` function, and to build your own  compilation table.


```r
#The 'WS64' list derives from Williams and Shuman 2008
#Accessing second dataset to look at pollen dataset instead of macrofossil dataset
irwin_data_pollen <-irwin_data[[2]]
irwin_data_poll64 <- compile_taxa(irwin_data_pollen, list.name = "WS64")
head(irwin_data_poll64)
```

You'll notice that warning messages return  a number of taxa that cannot be converted using the existing data table.  Are these taxa important?  They may be important for you.  Check to see which taxa have been converted by looking at the new taxon table:


```r
irwin_data_poll64$taxon.list[,c("compressed", "taxon.name")]
```

### Plotting

There are several options for plotting stratigraphic data in R.  The `rioja` package [@rioja_package] and `analogue` [@analogue_package] each have methods, and other possibilities exist.  Here we will show simple plotting using the `analogue` package. To make it clear which functions come from the `analogue` package I will use `analogue::` before the function names.  This is just an explicit way to state the function source.  If you choose not to do this you will not encounter any problems unless multiple packages have similarly named functions.

Convert the pollen data to percentages
```r
library("analogue")
irwin_data_pct <- analogue::tran(x = irwin_data_poll64$counts, method = 'percent')
tower_data_pct <- analogue::tran(x = tower_data_poll64$counts, method = 'percent')
head(irwin_data_pct)
head(tower_data_pct)
```

Create new data tables with just depths, ages, and hemlock percents.  SIMON - THIS IS WHERE I'm STUCK

I'm pretty sure that I've grabbed the right depth/age information but also pretty sure that I haven't gotten the right column for hemlock percentages.  According to taxa.list, hemlock is variable 14, but a quick print shows that this isn't hemlock.  

There has to be a better way of doing this...

```r
irwin_hemlock<-data.frame(irwin_data_poll64$sample.meta$depth,irwin_data_poll64$sample.meta$age,irwin_data_pct[,c(14)])
tower_hemlock<-data.frame(tower_data_poll64$sample.meta$depth,tower_data_poll64$sample.meta$age,tower_data_pct[,c(14)])

```

Saving data to an .RData file for sharing and re-use

```r
save(kettle_site,spicer_site,tower_site,kettle_data,tower_data,file="neotoma_3sites.RData")
save.image()
```
# Conclusions

The `neotoma` package, and the Neotoma Paleoecological Database provide a powerful tool for data analysis and data manipulation.  These data tools can support exploratory data analysis, teaching opportunities, and research projects across disciplines.  More information about using Neotoma is available from the database's [Manual](https://neotoma-manual.readthedocs.io/en/latest/), and from the `neotoma` package [paper](http://www.openquaternary.com/articles/10.5334/oq.ab/).  The 'neotoma'  source code is available at [GitHub](https://github.com/ropensci/neotoma)

# References
