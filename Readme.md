# Project Goals
1.  Clean and prep company lease records for utilizing Tableau's Mapping Visualizatons

2. Build Tableau Dashboard displaying key lease information for company CEO and public use 

3.  Host on Company Website Built on Bootstrap
See here: http://magnumproducing.com/assets.html

<br>

# Libraries 
* Code: Python 3.7
* Packages:  Geopandas, Pandas, FuzzyWuzzy Requests, Datetime

# Pertinent Jupyter Notebooks to this ReadMe
* 2. BLM Master Spreadsheet - Clean for Tableau.ipynb
* 3. Mapbox Prep - PLSS Merge.ipynb
<br>

# 1.  Cleaning & Prepping Data: Data Feching, Cleaning Company Lease Records, & Merging PLSS Data

**Primary Goal:**  Partial or "Fuzzy" match county names from company lease records to official county names and FIPS code

## Data Sources:
1. US Counties Shapefile
    * Contains US County FIPS code and county boundaries
2. US State, FIPS, & Postal Code
    * *Webscraped*
3.  Internal Company Lease Record (Excel File)
    * Master spreadsheet holding all information on each company lease
4.  Internal Company Lease Shapefile 
    * Contains polygons of lease boudnaries for each company lease
5.  PLSS Shapefiles
    * Contains geographic subdivisions for US Counties (Township/Range)

<hr>

## Tasks
1.  Fetch data sources and join US Counties Shapefile with Webscraped State Table
2.  Create unique, compounded string with county and state postal code for "Fuzzy Matching" betweeen Internal Lease Records and Offical US County Names with FIPS codes
3.  Run Fuzzy Match Algorithms to join official county names and FIPS codes to Internal Lease Records
4.  Join Internal Company Lease Shapefile to Internal Lease Records holding Fuzzy Matched county names
5. Clean final joined datatable for exporting into ESRI Shapefile to be read into Tableau
6.  Combine all internal PLSS files and cleanup identical columns to host on Mapbox as one layer 

## Code Snippets

Webscraping State Table and Joining to Counties Shapefile
```
stlink = 'https://www.nrcs.usda.gov/wps/portal/nrcs/detail/?cid=nrcs143_013696'
stTable = pd.read_html(req.get(stlink).content)[0]

#last row had null fips due to extra read row from html source
stTable = stTable.iloc[:-1, :]
stTable["FIPS"] = stTable["FIPS"].astype(int)
stTable.rename(columns = {"FIPS": "State Fips"}, inplace = True)

usCounties = pd.merge(usCounties, stTable.iloc[:,1:], how="left", left_on = "STATE_FIPS", right_on = "State Fips")
```

Function for Fuzzy Matching
```
def fuzzyMatchCustom(query, choices, User_score_cutoff = 0):
    '''
    This function will fuzzy matching to find matches for lease serial numbers in our master spreadsheet to BLM lR2000 
    database serial numbers or counties with official county names
    
    '''
    
    #assigingn an 88 score cutoff to avoid false matches
    match = process.extractOne(query, choices, score_cutoff=User_score_cutoff)
    if type(match) == type(None):
        return "No Match"
    else:
        return match[0]
```
Applying Function to Internal Lease Record Spreadsheet
```
spreadsheet["Fuzzy Matched County"] = spreadsheet["County for Matching"].apply(lambda x: fuzzyMatchCustom(str(x), usCounties["County Fuzzy Choices"], User_score_cutoff = 88))
```

# 2. Tableau Dashboard Images
1. Dashboard Not Filtered
![Tableau Not Filtered](Images/DashboardNoFilter.png)

<hr>
2.  Dashboard Filtered to Select Counties

![Tableau Filtered](Images/DashboardFiltered.png)