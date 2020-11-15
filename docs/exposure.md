## Exposure DB schema

The exposure schema was developed based on [GEM Taxonomy 2.0](https://wiki.openstreetmap.org/wiki/GED4ALL) to accommodate the most important spatial features commonly employed in risk analysis to identify and estimate exposed value. 
The [GED4ALL](https://www.hotosm.org/projects/global-exposure-database-for-all-ged4all) database schema contains the following tables: contribution, exposure\_model, asset, occupancy, cost, cost\_type, and tags.

Multiple types of structural/infrastructure assets and socio-economic exposure can be stored in the exposure database with attributes relevant to assessing risk from multiple hazards. 
The exposure schema provides 11 defined exposure types, for consistent classification of assets in different datasets: Residential, Commercial, Industrial, Healthcare, Education, Government, Infrastructure, Crop, Livestock, Forestry, and Mixed (as in _cf\_common.occupancy\_enum_).
The tables include general attributes that can be adapted to each asset type. More specific indicators can be stored using the _tags_ table, e.g. population gender and age, urban growth rate, GDP or others.

Exposure can be stored at multiple scales and using polygon (e.g., for a building footprint or number of buildings in a district), polyline (e.g. road network segment), and point (e.g. geolocated building at a lat,lon position). 
Exposure units can be assigned the number night-time or day-time occupants, to represent population exposure.

![Screenshot](img/exposure.png)
ERD (exposure schema): exposure table contents (green) and links to common tables (yellow). The schema includes a SQL view (cyan).

###Table: _ged4all.contribution_

This table contains metadata for the exposure dataset, including the exposure model source, data version and license, the project it was produced in, the purpose of the dataset and additional notes.

| **Req** | **Field name** | **Type** | **Reference table** | **Description** |
|:---:| --- | --- | --- | --- |
| **\*** | id | INT | | Unique number ID |
| **\*** | event\_set\_id | INT | | Unique number ID of event\_set |
| **\*** | model\_source | VARCHAR | | Name of source model |
| **\*** | model\_date | DATE | | Model release date |
| | notes | TEXT | | Details about the dataset |
| | version | VARCHAR | | Version of the dataset |
| | purpose | TEXT | | Purpose for what the data has been produced |
| | project | VARCHAR | | Project under which data has been produced |
| **\*** | contributed\_at | timestamp | | Date of contribution |
| **\*** | license\_code | VARCHAR | _cf\_common.license_ | Type of license |

<br/>
###Table: _ged4all.exposure\_model_

Each row in this table represents a collection of assets with a taxonomy\_source (GEM Taxonomy, SynerG, FAO, etc) and category (buildings, road network, etc). 
A model can optionally contain one or more cost types (see model\_cost\_type), e.g. structural, contents and can also optionally contain information about the area occupied by the assets. Finally, a model can optionally specify any number of &quot;tags&quot;, which can be applied to each asset.

| **Req** | **Field name** | **Type** | **Reference table** | **Description** |
|:---:| --- | --- | --- | --- |
| **\*** | id | INT | | Unique number ID |
| **\*** | name | VARCHAR | | Name of exposure model |
| | description | VARCHAR | | Description of exposure model |
| | taxonomy\_source | VARCHAR | | Name of taxonomy source |
| **\*** | category | ENUM | _ged4all.category\_enum_ | Type of asset |
| | area\_type | VARCHAR | | Aggregated or per asset |
| | area\_unit | VARCHAR | | Unit of measure of area |
| | tag\_names | VARCHAR | | Additional tag information |
| | use | ENUM | _cf\_common.occupancy\_enum_ | Usage type of the asset (e.g. residential, commercial) |

<br/>
###Table: _ged4all.asset_

Each row in the asset table represents a single asset or collection of assets, such as a building, aggregated collection of buildings, an item in an infrastructure network, or an area of farmland. 
The number\_of\_units field is used to specify the number of assets of this type at this location. 
The taxonomy value must be set to a valid taxonomy string using the taxonomy system specified in the exposure\_model table, and identifies the typology of this asset. 

There are two geometry fields providing geospatial information:
- The &quot;the\_geom&quot; field specifies a single point location and can be considered as a composite representation of latitude, longitude and a reference to WGS84. Only Point geometries are stored in this field.
- The &quot;full\_geom&quot; field is an optional representation of any geometry type (point, line, polygon, multipoint, multi-line, etc.) supported by the standard and can be used to describe the footprint of a building, regional boundaries, or lifelines.type (point, line, polygon, multipoint, multi-line, etc.) supported by the standard and can be used to describe the footprint of a building, regional boundaries, or lifelines.

| **Req** | **Field name** | **Type** | **Reference table** | **Description** |
|:---:| --- | --- | --- | --- |
| **\*** | id | INT | | Unique number ID |
| **\*** | exposure\_model\_id | INT | | Unique number ID of source exposure model |
| **\*** | asset\_ref | VARCHAR | | Alphanumeric code supplied by user |
| **\*** | taxonomy | VARCHAR | | Alphanumeric code for the taxonomy source |
| | number\_of\_units | FLOAT | | Number of assets represented |
| | area | FLOAT | _exposure\_model.area\_unit_ | Area of the asset in the units specified |
| **\*** | the\_geom | GEOM | | Associated geometry data (point location) |
| | full\_geom | GEOM | | Associated geometry data (polygon, multiline, multipolygon...) |

<br/>
###Table: _ged4all.cost\_type_

Each exposure model can optionally have one or more types of cost associated with loss or damage to assets. 
For example, the cost of the building structure by square meter or the cost of the contents of a single building. 
Each cost type entry describes a category of cost, an aggregation type and a unit.

| **Req** | **Field name** | **Type** | **Reference table** | **Description** |
|:---:| --- | --- | --- | --- |
| **\*** | id | INT | | Unique number ID |
| **\*** | exposure\_model\_id | INT | | Unique number ID of source exposure model |
| **\*** | cost\_type\_name | VARCHAR | | Type of asset cost (structural, non-structural, contents, business interruption) |
| **\*** | aggregation\_type | VARCHAR | | Aggregated or per asset |
| | unit | VARCHAR | | Cost unit of measure |

<br/>
###Table: _ged4all.cost_

Each row in the cost table represents a cost of a given cost type for a given asset. 
The asset\_id and cost\_type\_id fields are foreign keys used to identify an asset and model\_cost\_type. 
The value field provides the actual cost value in the unit specified in the corresponding model\_cost\_type entry.

| **Req** | **Field name** | **Type** | **Reference table** | **Description** |
|:---:| --- | --- | --- | --- |
| **_\*_** | id | INT | | Unique number ID |
| **\*** | asset\_id | INT | | Unique number ID of the asset |
| **\*** | cost\_type\_id | VARCHAR | | Unique number ID of the cost type |
| **\*** | value | FLOAT | | Cost value |
| | deductible | FLOAT | | |
| | insurance\_limit | FLOAT | | |

<br/>
###Table: _ged4all.occupancy_

The occupancy table is used to store information about the occupants of an asset in a given period, for example, the number of people in a given building during the day and during the night. 
For some cases the period might refer to a season, for example when considering livestock or agricultural assets. 
In some communities the term &quot;occupancy&quot; is used to refer to building usage, for example &quot;industrial&quot; or &quot;residential&quot;; this concept can be modelled in the GED4ALL schema using tags and/or by using a taxonomy system that supports building usage specification, such as the GEM Building Taxonomy.

| **Req** | **Field name** | **Type** | **Reference table** | **Description** |
|:---:| --- | --- | --- | --- |
| **_\*_** | id | INT | | Unique number ID |
| **\*** | asset\_id | INT | | Unique number ID of the asset |
| **\*** | period | VARCHAR | | Occupancy type (night/day) |
| **\*** | occupants | FLOAT | | Number of occupants |

<br/>
###Table: _ged4all.tags_

Each row in the tags table represents a name and value pair for a given asset. 
The asset\_id is a foreign key used to identify an asset. 
The name field represents the name of the tag while the value field contains the associated value. 
Tags may be used to store any named scalar information with an asset, but in particular are useful for storing socio-economic indicators and their associated values, and also for grouping assets into census tracts, for example.

| **Req** | **Field name** | **Type** | **Reference table** | **Description** |
|:---:| --- | --- | --- | --- |
| **_\*_** | id | INT | | Unique number ID |
| **\*** | asset\_id | INT | | Unique number ID of the asset |
| **\*** | name | VARCHAR | | Name of the tag |
| **\*** | value | VARCHAR | | Number associated with the tag |

<br/>
###Types

| **ENUM name** | Types | Description |
| --- | --- | --- |
| calc\_method\_enum | <ul><li>Buildings<li>Indicators<li>Infrastructure<li>Crops, livestock and forestry | Type of asset category |

<br/>

###Taxonomies
The exposure schema can accomodate different descriptions of assets using existing taxonomies which describe their characteristics (e.g. building occupancy, construction, age, height, etc. or road surface type). The taxonomy being applied to a dataset is defined in _ged4all.exposure\_model.taxonomy\_source_. The taxonomy string, which describes the characteristics of the asset (or group of assets of the same type), is defined at _ged4all.asset.taxonomy_. It uses a single field description because the schema is designed to use the GEM taxonomy, which is described in full here [here](https://www.globalquakemodel.org/gempublications/GEM-building-taxonomy-version-2.0) which defines the format of taxonomy strings. GEM provides a ['taxtweb'](https://platform.openquake.org/taxtweb/) tool for creating taxonomy strings and checking the translation of a string. Some example taxonomy strings are provided below:

| Type | Taxoomy examples |
| --- | --- |
| Buildings | <ul><li>M99/HEX:1/COM - Unreinforced Masonry/One storey above ground/Commercial use <li>M99/HEX:2/RES+RES1 - Unreinforced Masonry/Two storeys above ground/Residential Single Unit<li>CR/INF - Reinforced Concrete/Industrial use (height unknown) |
| Infrastructure | <ul><li>RD/PR - Primary road<li>RD/SE - Secondary road<li>... - ... |
| Crops, livestock and forestry | <ul><li>... - ...<li>... - ...<li>... - ... |


<br/>
