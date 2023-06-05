# Sherry Lehmann Catalog - Auto generated wine price information.

This project contain wine-price information extracted from Universirty of
Californa, Davis Library's digital collection of Sherry Lehmann Catalogs.  These
data are automatically extracted from the catalogs using some machine pattern
matching and processing steps that are more fully described in the [Wine Price
Extraction Github](https://github.com/ucd-library/wine-price-extraction)
repository.  That repo contains the `R` and `pgsql` scripts used to process the
files.  The original images used in the processing can all be found on the [UC
Davis Digital
Collections](https://digital.ucdavis.edu/collection/sherry-lehmann) website.  In
addition, a short description of the processing methodology can be found in this
[AAWE presentation](https://bit.ly/hart19aawe).

This is an **interium dataset only**, user beware.  We have not finalized our
first methodology for data extraction, and have not performed comprehensive
quality control on even the current methodology.  The next planned update of
this dataset is in September 2019.

# Data tables

There are a number of data tables in this repository, but they are basically
different flavors of the same dataset, with some format and content
modifications.

## wine_search.csv

The `wine_search.csv` represent the dataset that was used as input to our
original wine-search application.  The data columns are identified below. 

The `ark:` directory contains the same data as json files, one json file for
each catalog.

Note that an [ark](https://ezid.cdlib.org/) corresponds to a globally unique
identifier for each Sherry Lehmann catalog.  The catalogs can be identified
with these arks.  They can also be used to link to the original data.  For
example, the catalog for identifier `d7fk5w` can be located with
https://digital.ucdavis.edu/ark:/87287/d7fk5w, and the 18th page with the
https://digital.ucdavis.edu/ark:/87287/d7fk5w/media/images/d7fk5w-018.jpg url.
Every ARK has the same `ark:/87287/` shoulder.

| Name | Description |
| ---  | --- |
| page_ark | The page identifier for the page containing the entry |
| ark | The ARK identifier for the catalog |
| name | The extracted name of the wine |
| year_published | The year the corresponding catalog was published |
| vintage | The extracted vintage |
| type | *The type of wine, Still/Sparkling/Fortified.  This is not currently represented in the extracted data.* |
| color | Red/White/Rosé|
| country | Extracted or matched country of production |
| section | *Not used in the extracted product* |
| bottle_type | *Currently not used in extracted product* |
| perprice | Price per bottle |
| caseprice | Price per case  |
| bbox | GeoJSON formatted box desscribing the location of the price on the page.  The origin (0,0) is in the upper left of the full image. |



## Postgresql Tables

The following tables were used for some quality checking of the extracted wine
prices. These data were used in a postgresql database, and because of some
formatting, they are probably easiest to use in the same environment.

### entries.csv

The *entries.csv* file expands on the wine-search.csv file with some additional
columns that were used for evaluating the entries, and comparing to the manually
entered prices.  The columns shared with the wine-search.csv are not included in
the description below.

| Name | Description |
| ---  | --- |
|  | See above for; ark, page_ark, year_published,perprice,caseprice |
|name_id | Unique identifier for the entry |
producer| Guessed producer for the entry|
variety| Guessed variery for the entry |
region| Guessed region for the entry |
province| Guessed province for the entry |
price_cnt|Count of the number of columns associated with this entry|
price_header|List of the columns (by header) used for perprice and caseprice|
price_per_column|List of all prices for this entry|
header_per_column|List of all headers for prices for this entry|
ul| POSTGIS geometry for upper left of entry|
left_side| POSTGIES geometry for left side of entry|

The table can be created with:

```sql
create table entries (
page_ark text,
ark text,
page integer,
year_published integer,
name_id text primary key,
name text,
country text,
vintage integer,
color text,
producer text,
variety text,
region text,
province text,
perprice numeric(7,2),
caseprice numeric(7,2),
price_cnt integer,
price_header text[],
price_per_column text[],
header_per_column text[],
ul geometry,
left_side geometry,
bbox box2d);
```

### entry_flags.csv

The *entry Flags.csv* file contains some of the quality flags from the
processing steps.  Unfortunately, many of these flag definitions will not be
immediately apparent without some understanding of the [Wine Price
Extraction](https://github.com/ucd-library/wine-price-extraction) processing,
especially those concerning the name matching algorythm.

| Name | Description |
| ---  | --- |
| name_id | The entry identifier |
| flag | The identified problem with the entry |

## Manual entries and comparisons

Two additional tables, `manual_entries.csv` and `vs_manual_entries.csv`, contain
information regarding comparisons to manually entered records.  The manual data
contains most of the same information as the `wine_search.csv`.  However in the
original manual entry, the entries were only identified with a point not a
bounding box.

|   Name    |  Description |
| --- | --- |
|  | See above for; page_ark, year_published,perprice, caseprice, country,color,type |
| section | The major section of the catalog (section title) holding the entry |
| xy | approximate entry location |

The `vs_manual_entries.csv` file is a join of these manual entries with the best
match for the entries table where such a match can be found.

