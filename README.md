# algaeClassify

<!-- badges: start -->
<!-- badges: end -->

The goal of algaeClassify is to facilitate the analysis of taxonomic and functional trait
data for phytoplankton.

## Installation

You can install the released version of algaeClassify from [CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("algaeClassify")
```

To use the new algaebase search functions, you MUST install v2.0.0 from github with:
``` r
require(devtools)
install_github("vppatil/algaeClassifyV2",ref="main",dependencies=TRUE)
```

Next, load the package and ensure you have the correct version installed.
``` r
library(algaeClassify)
citation("algaeClassify")
```

##April 5, 2023: New functions for querying algaeBase (www.algaebase.org)!!!!
Algaebase search function examples
``` r
#check out the package
help(package="algaeClassify")

#View the new algaebase search functions:
help("algaebase_species_search")
help("algaebase_genus_search")
help("algaebase_search_df")
```

##Using your API key

The Algaebase functions require an API key. You can obtain one from [Algaebase](www.algaebase.org/api).

There are several options for using your api key.
1) assigning it to an R object, and using it in function calls
```{r eval=FALSE}
apikey<- "asasdfasdfasdfasfd" (not a real key)
algaebase_genus_search(genus="Anabaena",apikey=apikey)
```

2) Saving it in a text file (e.g. "keyfile.txt"). You will need to give the filename and path using the api_file function argument.
```{r eval=FALSE}
algaebase_genus_search(genus="Anabaena",api_file="keyfile.txt")
```

3) Finally, you can set your key as an environment variable.
To do so, open or create a .Renviron text file in your home directory.
One way to do this is by running the following line:
```{r eval=FALSE}
file.edit("~/.Renviron")
```
Add a line to the .Renviron file like the following, but use your actual key after the = symbol:

ALGAEBASE_APIKEY=yourKeyHere

Finally, save and close the file, then restart R for changes to take effect.
Once the ALGAEBASE_APIKEY variable is defined, you do not need to specify it in the algaebase search functions. 

## Algaebase search function examples:
#You can search for a single genus
``` r
?algaebase_genus_search 
algaebase_genus_search("Anabaena")
```

#Or a genus and species name
``` r
algaebase_species_search("Anabaena","flos-aquae")

#There are several arguments for these functions.
#you can control whether to include higher taxonomy in the output
algaebase_genus_search(genus="Navicula",higher=TRUE)

#You can also choose to include the full species name with author and date in output
algaebase_genus_search(genus="Navicula",higher=TRUE,long=TRUE)

#The default only returns exact matches and the most recent entry in algaebase, but you can override that behavior
algaebase_species_search(genus="Nitzschia",species="acicularis",newest.only=FALSE,exact.matches.only=FALSE,long=TRUE)
```

In all cases, the output will return the currently accepted name, as well as the name that was supplied by the user. There are columns indicating whether the input name is currently accepted and whether an exact match was found.

If desired, you can view the raw output in JSON format
``` r
algaebase_genus_search(genus="Cyclotella",higher=TRUE,print.full.json=TRUE)
```

##Search a list of names:
Finally, you can submit a data.frame of phytoplankton names to algaebase
the data frame should have columns named genus and species

This will only return 1 result per name.
If there are no exact matches it will return NA
If there is no match for genus+species, it will search for a genus-only match
or you can specify genus.only searches for the entire dataset.
``` r
data(lakegeneva) #load small example dataset
head(lakegeneva) #view example dataset

lakegeneva<-genus_species_extract(lakegeneva,phyto.name="phyto_name")
lakegeneva<-lakegeneva[!duplicated(lakegeneva$phyto_name),]
lakegeneva.algaebase<-algaebase_search_df(lakegeneva,higher=TRUE,genus.name="genus",species.name="species")
head(lakegeneva.algaebase)
``` 

##Other taxonomic search functions.
Version 2.0.0 includes functions for searching the ITIS database and for using the Global Names Resolver (GNR). These functions are based on the **ritis** and **taxize** packages, respectively.
``` r
help("genus_search_itis")
help("species_search_itis")
help("itis_search_df")

#ITIS
genus_search_itis(genus="Mougeotia",higher=TRUE)

species_search_itis(genspp="Anabaena flos-aquae")

#GNR (Global Names Resolver)
#GNR/taxize use fuzzy/partial matching, and search multiple databases.
#If you do not find a hit in algaebase or ITIS, you can try searching for a 
#partial match via GNR

help("gnr_simple")
help("gnr_simple_df")

name<-"Anabaena flos-aquae"
gnr_simple(name=name,sourceid=3) #Search ITIS
gnr_simple(name=name,sourceid=NULL) #search for matches from any source
```

##Examples of other algaeClassify functions:
This is a basic example which shows you how to use algaeClassify to 
1) identify anomalies in a time-series of phytoplankton species
2) calculate aggregate abundance at a higher taxonomic level (genus)
3) re-plot species accumulation curves to see if the taxonomic standardization and 
aggregation to higher taxonomy have resolved the anomalies.

``` r
library(algaeClassify)

data(lakegeneva) #load a demonstration dataset

#view species accumulation curve over duration of dataset to check for anomalies
accum(lakegeneva,phyto_name='genus',column='biovol_um3_ml',n=100,datename='date_dd_mm_yy',dateformat='%d-%m-%y')

#clean up binomial names and extract genus and species to new columns
lakegeneva<-genus_species_extract(lakegeneva,phyto.name='phyto_name')

#aggregate abundance data to genus level
lakegeneva.genus<-phyto_ts_aggregate(lakegeneva,SummaryType='abundance',AbundanceVar='biovol_um3_ml',
                    GroupingVar1='genus')

#plot accumulation curve again, but at genus level
accum(lakegeneva.genus,phyto_name='genus',column='biovol_um3_ml',n=100,datename='date_dd_mm_yy',dateformat='%Y-%m-%d')

#assign species to morphofunctional groups using species names
lakegeneva.mfg<-species_to_mfg_df(lakegeneva)

#plot accumulation curve for morphofunctionalgroups
lakegeneva.mfg<-phyto_ts_aggregate(lakegeneva.mfg,SummaryType='abundance',AbundanceVar='biovol_um3_ml',
                    GroupingVar1='MFG')
					
accum(lakegeneva.mfg,phyto_name='MFG',column='biovol_um3_ml',n=100,datename='date_dd_mm_yy',dateformat='%Y-%m-%d')




```

