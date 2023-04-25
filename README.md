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
install_git("https://code.usgs.gov/asc/algaeClassify",ref="master") 
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

##Other taxonomic search functions. Version 2.0.0 includes functions for
searching the ITIS database and for using the Global Names Resolver
(GNR). These functions are based on the **ritis** and **taxize**
packages, respectively.

```{r}
#ITIS
genus_search_itis(genus="Mougeotia",higher=TRUE)

species_search_itis(genspp="Anabaena flos-aquae")

#GNR (Global Names Resolver)
#GNR/taxize use fuzzy/partial matching, and search multiple databases.
#If you do not find a hit in algaebase or ITIS, you can try searching for a 
#partial match via GNR

name<-"Anabaena flos-aquae"
gnr_simple(name=name,sourceid=3) #Search ITIS
gnr_simple(name=name,sourceid=NULL) #search for matches from any source

species_search_itis(genspp="Anabaena flosaquae") #check itis f0r gnr best match.
```
##Functional group classifications:

AlgaeClassify can also be used to assist users in classifying species
into one of several morphological/functional classifications: Morpho-
functional groups (MFG) [Salmaso et al. 2015] or the C-S-R framework as described for 
freshwater systems [Reynolds 1988; Reynolds 2006]

It is possible to derive MFG and CSR classifications by supplying trait values.
Relevant traits are surface area, volume and maximum linear dimension for CSR,
or presence/absence of flagella, colonies, filamentous growth forms, and 
other traits for MFG, as described in Salmaso et al. 2015.

```{r}
traits_to_csr(sav=0.2,msv=10,traitrange=traitranges)

traits_to_mfg(flagella = 1,size = "large",colonial = 1,filament = 0,
              centric = NA,gelatinous = 0,aerotopes = 0,
              class = "Euglenophyceae",order = "Euglenales")
```

Users may be able to retrieve relevant trait data from published sources. For 
convenience, algaeClassify has included tables of MFG and CSR trait values
for common European species, extracted from [Rimet et al. 2018].
<https://doi.org/10.1051/limn/2018009>.

```{r}
data("csrTraits")
data("mfgTraits")
head(csrTraits)
head(mfgTraits)
```

Alternatively, the user may be able to match species to a library of species:MFG
classifications derived via expert assessment as part of the
GEISHA John Wesley Powell Center fellowship.
<https://www.usgs.gov/centers/john-wesley-powell-center-for-analysis-and-synthesis/science/global-evaluation-impacts>

This library can be used to classify lists of species or single species names.
However, many morphofunctional traits are plastic and MFG classifications may
vary for a single species depending on location and environment. Use with
caution!
```{r}
data("species_mfg_library")
head(species_mfg_library)

species_to_mfg('Scenedesmus','bijuga')

data(lakegeneva)
lakegeneva<-genus_species_extract(lakegeneva,phyto.name='phyto_name')
lakegeneva.mfg<-species_to_mfg_df(lakegeneva)

head(lakegeneva.mfg)
```

If desired, species_to_mfg_df() can be run with a user-supplied library of 
species:MFG matches based on their own research, using the mfgDbase argument.
```{r}
help("specis_to_mfg_df")
```

Finally, the user can crosswalk MFG and CSR classifications, based on 
[Salmaso et al. 2015] and [Reynolds 1988].
```{r}
data(mfg_csr_library)
mfg_csr_convert(mfg="11a-NakeChlor")
```
##Other functions to visualize and manipulate long-term phytoplankton datasets:
This is a basic example which shows you how to use algaeClassify to 1) identify 
anomalies in atime-series of phytoplankton species 2) calculate aggregate 
abundance at a higher taxonomic level (genus) 3) re-plot species accumulation 
curves to see if the taxonomic standardization and aggregation to higher
taxonomy have resolved the anomalies.

phyto_ts_aggregate() can be used to aggregate phytoplankton timeseries based 
on abundance and grouping variables (genus, MFG, station id, etc.). If no 
abundance var is given, will aggregate to presence/absence.

date_mat() is used to transform long-term data into a matrix of abundance or
presence/absence, organized by taxa/group and date (day, month, or year). This
simplifies the process of formatting data for ordinations or other plots of 
abundance change over time.

accum() compares observed change in total species richness over time 
to simulated curves based on random draws with replacement from a list of all
unique species in the dataset. step discontinuities in the observed curve &
departures from the simulated range may single sudden ecological or 
methodological changes (including changes in taxonomic standards, personnel,
or equipment).

```{r}

data(lakegeneva) #load the demonstration dataset

accum(lakegeneva,phyto_name='phyto_name',column='biovol_um3_ml',n=100,
      datename='date_dd_mm_yy',dateformat='%d-%m-%y')

#clean up binomial names and extract genus and species to new columns
lakegeneva<-genus_species_extract(lakegeneva,phyto.name='phyto_name')

#aggregate abundance data to genus level
lakegeneva.genus<-phyto_ts_aggregate(lakegeneva,SummaryType='abundance',
                                      AbundanceVar='biovol_um3_ml',
                                      GroupingVar1='genus')
```

```{r}
#plot accumulation curve again, but at genus level
accum(lakegeneva.genus,phyto_name='genus',column='biovol_um3_ml',n=100,
      datename='date_dd_mm_yy',dateformat='%Y-%m-%d')

#classify taxa to CSR and visualize relative abundance of CSR groups by month
data(lakegeneva)

lakegeneva<-genus_species_extract(lakegeneva,phyto.name='phyto_name')
lakegeneva<-species_to_mfg_df(lakegeneva)
lakegeneva<-mfg_csr_convert_df(lakegeneva,mfg='MFG')
csrAbundance.by.month<-date_mat(lakegeneva,abundance.var='biovol_um3_ml',
                                taxa.name='CSR',time.agg='month')

#make a simple heatmap of mean daily csr group abundance by month
heatmap(csrAbundance.by.month,Rowv=NA,Colv=NA,col=viridis(10))
```

