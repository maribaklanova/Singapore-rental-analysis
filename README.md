# Singapore Rental Analysis Project

## Summary

The project provides valuable insights into the Singapore rental market. 
Data was collected from official sources s.a. URA API, OneMap API, and a geojson file on planning areas.
Data cleaning and pre-processing steps were taken to address missing values, outliers, and data transformations.
The analysis focused on rental prices by district, project, neighborhood (planning area), number of bedrooms, and region.
Data visualization included various plots/charts and a rent-aggregating choropleth map of Singapore.

By combining the insights from this project with available options and our specific needs like the property amenities, we can make a more data-driven decision about Singapore rental search!

For the code please refer to [the notebook](https://github.com/maribaklanova/Singapore-rental-analysis/blob/60731e2fe6279d243723301fcd6ada563f524748/Singapore_rent_project_MB.ipynb).

## Project Overview

There is a lack of projects analyzing the private property rental market in Singapore. This project aims to contribute to filling this gap.
This project aims to analyze reliable rental data to understand the Singapore rental market and support further decisions on our residential rent with evidence-based insights. I’ll analyze data to investigate rental rates depending on the date, location, and property characteristics using Python with relevant libraries s.a. pandas, NumPy, GeoPandas, matplotlib, seaborn, folium, SciPy, etc. What I learn will then guide us in the choice of neighborhood and type of housing. This project will provide me with empirical evidence on suitable Singapore areas and patterns/trends of the rental market which allow us to narrow the search range, reduce non-productive efforts and stress levels, and simplify the adjustment process when relocating to Singapore.

## Data Sources

In Singapore, all the private rental contracts are registered with the Inland Revenue Authority of Singapore, and the Urban Redevelopment Authority collects the data on them. 
For this project, I will be using the data on private residential properties with rental contracts from Jan-2022 through Dec-2023 obtained with the [API service of Urban Redevelopment Authority](https://www.ura.gov.sg/maps/api/#private-residential-properties-rental-contract), complementing it with the data related to locations obtained with [OneMap API of Singapore Land Authority](https://www.onemap.gov.sg/apidocs/) and from [geojson file on Planning Area Boundary](https://beta.data.gov.sg/datasets/d_4765db0e87b9c86336792efe8a1f7a66/view). 


## Data acquisition

I downloaded the data from URA API service with the related [Python wrapper](https://pypi.org/project/ura-api/) in json format, normalizing and combining the data into an initial dataframe.
During further dataframe complementing, I downloaded missing coordinates and added columns with World Global System coordinates to it from OneMAp API using [PyOneMap Python package](https://pypi.org/project/pyonemap/).

## Data Cleaning & Transformation

I removed from initial dataframe columns that were not needed for my analysis, converted data types into suitable formats, handled null values, and added new calculated columns using functions I defined. The description of the prepared dataframe and the results of its pre-processing are given below:

| Column      |         Description                      | Data type / transformation | % NaN | Data cleaning            |  
|-------------|------------------------------------------|----------------------------|-------|--------------------------|
| leaseDate   | The lease commencement date of the rental contract| converted to *datetime64*  |  0    | N/A                      |
| project     | The name of the project                  | *object*                   |  0    | N/A                      |
| street      | The street name that the project is on   | *object*                   |  0    | N/A                      |
| district    | The postal district that the transacted property falls in (01...28) | *object*                   |  0    | N/A                      | 
| x           | The x coordinates of the property address in SVY21 format| *object*                   |  0.1  | The blanks were filled with coordinates matching the street address using a function that accesses the OneMap API Client  |
| y           | The x coordinates of the property address in SVY21 format| *object*                   |  0.1  | -//- |
| noOfBedRoom | The number of bedrooms                   | converted to *int64*       |  9.8  | The blanks were replaced by average values for properties with similar floor area |
| areaSqft    | The floor area range of the rented property in square feet| *object*                   |  0    | N/A                      |
| rent        | The monthly rent in Singapore dollars    | *int64*                    |  0    | N/A                      |
|                                                                                                                        |
|             |                                 **additional columns**                                                       |
|                                                                                                                        |
| AvgAreaSqft | Average floor area of the rented property calculated based on the range| *float64*                  |  0    | N/A                      |
| SGDperSqft  | Unit rental cost per sq. ft.             | *float64*                  |  0    | N/A                      |


My initial evaluation of the dataframe main statistics resulted in the necessity to trim outliers of the most variable data columns.

![Boxplots of average area and rent before trimming outliers](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/ade2caa5-611e-4d68-8304-91a97e12731a)

As the lower boundaries calculated with IQR were negative and the distributions were not normal, I estimated and then trimmed the outliers using [DistFit package](https://pypi.org/project/distfit/). 
The results of the distribution model fitting for rent and floor area with the boundaries of confidence intervals are shown below.
Floor area distribution |  Rent distribution
:-------------------------:|:-------------------------:
![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/1430f500-0678-43aa-a8ea-eedf6807773b)  |  ![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/0e5cc3a9-805b-455b-b3b9-031f7e630672)

This is what the data looked like after handling outliers.

![Boxplots of average area and rent after trimming outliers](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/e84816e4-3e4f-41a0-901c-afd25042f50d)

The next challenge I faced was to add WGS coordinates to the dataframe as I needed them to locate the names of the neighborhood (planning area) and region of each property. To achieve this, I defined the function converting SVY21 coordinates into WGS84 format accessing OneMap API, and applied it to a reference dataframe with *x* and *y* coordinates, then merged it with the main dataframe. 
After that I modified the dataframe with GeoPandas, adding a geometry column based on which I added the columns with neighborhood (planning area) and region using the [geojson file](https://beta.data.gov.sg/datasets/d_4765db0e87b9c86336792efe8a1f7a66/view) 

Thus, I additionally included the following columns in the final dataframe:

| Column      |         Description                         | Data type  |
|-------------|---------------------------------------------|------------|
| latitude    | The WGS84 latitude of the property address  | *object*   | 
| longitude   | The WGS84 longitude of the property address | *object*   |  
| geometry    | The geospatial point for the location coordinates       | *geometry* |
| PlanAreaName| The name of the neighborhood (planning area) property located in| *object*   |
| REGION_N    | The name of the Singapore region property located in   | *object*   |


## Data Analysis

### 1. Reviewing main statistics and correlation
First of all, I checked the main statistics of the numerical data, s.a. mean, minimum, maximum, median, 1st and 3rd quartiles, standard deviation

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number of bedrooms</th>
      <th>avg fl. area, sq.ft.</th>
      <th>rent, SGD</th>
      <th>SGD per sq.ft.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>140988</td>
      <td>140988</td>
      <td>140988</td>
      <td>140988</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.089674</td>
      <td>950.252980</td>
      <td>4125.458606</td>
      <td>4.724969</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.832923</td>
      <td>362.389937</td>
      <td>1278.095568</td>
      <td>1.591491</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1</td>
      <td>450</td>
      <td>566</td>
      <td>0.485714</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1</td>
      <td>650</td>
      <td>3200</td>
      <td>3.529412</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2</td>
      <td>950</td>
      <td>4000</td>
      <td>4.521739</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>3</td>
      <td>1250</td>
      <td>4900</td>
      <td>5.777778</td>
    </tr>
    <tr>
      <th>max</th>
      <td>6</td>
      <td>2124.976910</td>
      <td>7542</td>
      <td>15.926667</td>
    </tr>
  </tbody>
</table>

Then I explored average, median and mode values by regions:

<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="3" halign="left">number of bedrooms</th>
      <th colspan="3" halign="left">avg fl. area, sq.ft.</th>
      <th colspan="3" halign="left">rent, SGD</th>
      <th colspan="3" halign="left">SGD per sq.ft.</th>
    </tr>
    <tr>
      <th></th>
      <th>mean</th>
      <th>median</th>
      <th>mode</th>
      <th>mean</th>
      <th>median</th>
      <th>mode</th>
      <th>mean</th>
      <th>median</th>
      <th>mode</th>
      <th>mean</th>
      <th>median</th>
      <th>mode</th>
    </tr>
    <tr>
      <th>REGION_N</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CENTRAL REGION</th>
      <td>2.0</td>
      <td>2.0</td>
      <td>2</td>
      <td>948.0</td>
      <td>950.0</td>
      <td>450.0</td>
      <td>4440.0</td>
      <td>4300.0</td>
      <td>4000</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>EAST REGION</th>
      <td>2.0</td>
      <td>2.0</td>
      <td>2</td>
      <td>966.0</td>
      <td>950.0</td>
      <td>950.0</td>
      <td>3608.0</td>
      <td>3500.0</td>
      <td>3000</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>NORTH REGION</th>
      <td>2.0</td>
      <td>2.0</td>
      <td>2</td>
      <td>910.0</td>
      <td>950.0</td>
      <td>1150.0</td>
      <td>3304.0</td>
      <td>3200.0</td>
      <td>3000</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>NORTH-EAST REGION</th>
      <td>2.0</td>
      <td>2.0</td>
      <td>2</td>
      <td>887.0</td>
      <td>850.0</td>
      <td>450.0</td>
      <td>3528.0</td>
      <td>3400.0</td>
      <td>3000</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>WEST REGION</th>
      <td>2.0</td>
      <td>2.0</td>
      <td>3</td>
      <td>1007.0</td>
      <td>1050.0</td>
      <td>1250.0</td>
      <td>3921.0</td>
      <td>3800.0</td>
      <td>4000</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>4.0</td>
    </tr>
  </tbody>
</table>

and standard correlation coefficients:

![Correlation matrix](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/106ec802-8fc1-4b3e-a4a4-b95b08b90a3d)

* Statistical indicators provide a guide to the ranges, distribution features, and averages of rents and unit rental cost, floor area, and number of bedrooms, with the mean not significantly different from the median after handling outliers.
* We can distinguish the difference in the mean values across the regions with the Central region having the highest rental rates and the West offering more spacious properties.
* The correlation map presents a strong positive correlation between the average area and the number of bedrooms as well as rent, while the correlation between the unit price and average area, as well as the number of bedrooms, is a pretty strong negative.

### 2. Which districts and projects are top by the number of lease contracts?

Diving into the data, I grouped and sorted dataframes showing districts rated by the number of lease contracts

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>district</th>
      <th>lease_count</th>
      <th>percentage</th>
      <th>neigborhood</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>12149</td>
      <td>8.6</td>
      <td>[BEDOK, KALLANG, MARINE PARADE, GEYLANG]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>09</td>
      <td>11790</td>
      <td>8.4</td>
      <td>[ROCHOR, SINGAPORE RIVER, MUSEUM, ORCHARD, NOV...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>19</td>
      <td>11187</td>
      <td>7.9</td>
      <td>[HOUGANG, SENGKANG, PUNGGOL, TOA PAYOH, SERANG...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>14</td>
      <td>10188</td>
      <td>7.2</td>
      <td>[GEYLANG, BEDOK, KALLANG, TAMPINES]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10</td>
      <td>9681</td>
      <td>6.9</td>
      <td>[TANGLIN, BUKIT TIMAH, NEWTON, ORCHARD, RIVER ...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>05</td>
      <td>7877</td>
      <td>5.6</td>
      <td>[CLEMENTI, QUEENSTOWN]</td>
    </tr>
    <tr>
      <th>6</th>
      <td>16</td>
      <td>7232</td>
      <td>5.1</td>
      <td>[BEDOK, TAMPINES]</td>
    </tr>
    <tr>
      <th>7</th>
      <td>03</td>
      <td>7037</td>
      <td>5.0</td>
      <td>[BUKIT MERAH, SINGAPORE RIVER, QUEENSTOWN, TAN...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>23</td>
      <td>6146</td>
      <td>4.4</td>
      <td>[CHOA CHU KANG, BUKIT BATOK, BUKIT PANJANG, BU...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>18</td>
      <td>5961</td>
      <td>4.2</td>
      <td>[TAMPINES, PASIR RIS, PAYA LEBAR]</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>5735</td>
      <td>4.1</td>
      <td>[BUKIT TIMAH, NOVENA, TOA PAYOH]</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>5636</td>
      <td>4.0</td>
      <td>[KALLANG, TOA PAYOH, NOVENA, HOUGANG]</td>
    </tr>
    <tr>
      <th>12</th>
      <td>21</td>
      <td>4819</td>
      <td>3.4</td>
      <td>[BUKIT TIMAH, BUKIT BATOK, CLEMENTI, BUKIT PAN...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>22</td>
      <td>4377</td>
      <td>3.1</td>
      <td>[JURONG EAST, JURONG WEST]</td>
    </tr>
    <tr>
      <th>14</th>
      <td>01</td>
      <td>3915</td>
      <td>2.8</td>
      <td>[OUTRAM, DOWNTOWN CORE, SINGAPORE RIVER]</td>
    </tr>
    <tr>
      <th>15</th>
      <td>20</td>
      <td>3811</td>
      <td>2.7</td>
      <td>[ANG MO KIO, TOA PAYOH, BISHAN, SERANGOON]</td>
    </tr>
    <tr>
      <th>16</th>
      <td>13</td>
      <td>3783</td>
      <td>2.7</td>
      <td>[TOA PAYOH, GEYLANG, SERANGOON]</td>
    </tr>
    <tr>
      <th>17</th>
      <td>08</td>
      <td>3629</td>
      <td>2.6</td>
      <td>[KALLANG, ROCHOR, HOUGANG]</td>
    </tr>
    <tr>
      <th>18</th>
      <td>02</td>
      <td>3115</td>
      <td>2.2</td>
      <td>[OUTRAM, DOWNTOWN CORE, BUKIT MERAH]</td>
    </tr>
    <tr>
      <th>19</th>
      <td>17</td>
      <td>2592</td>
      <td>1.8</td>
      <td>[PASIR RIS]</td>
    </tr>
    <tr>
      <th>20</th>
      <td>27</td>
      <td>2506</td>
      <td>1.8</td>
      <td>[SEMBAWANG, MANDAI, YISHUN]</td>
    </tr>
    <tr>
      <th>21</th>
      <td>04</td>
      <td>2213</td>
      <td>1.6</td>
      <td>[SOUTHERN ISLANDS, BUKIT MERAH]</td>
    </tr>
    <tr>
      <th>22</th>
      <td>07</td>
      <td>2149</td>
      <td>1.5</td>
      <td>[DOWNTOWN CORE, KALLANG, ROCHOR]</td>
    </tr>
    <tr>
      <th>23</th>
      <td>28</td>
      <td>1688</td>
      <td>1.2</td>
      <td>[HOUGANG, SENGKANG, SERANGOON, ANG MO KIO]</td>
    </tr>
    <tr>
      <th>24</th>
      <td>25</td>
      <td>894</td>
      <td>0.6</td>
      <td>[WOODLANDS]</td>
    </tr>
    <tr>
      <th>25</th>
      <td>26</td>
      <td>732</td>
      <td>0.5</td>
      <td>[ANG MO KIO, YISHUN]</td>
    </tr>
    <tr>
      <th>26</th>
      <td>06</td>
      <td>146</td>
      <td>0.1</td>
      <td>[DOWNTOWN CORE]</td>
    </tr>
  </tbody>
</table>

As we can see, the number of leases in the top 5 districts is almost 40 % of the total count.

Top 15 projects by the number of lease contracts with the mean indicators of rent and rental cost per sq.ft.:

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>project</th>
      <th>lease_count</th>
      <th>percentage</th>
      <th>average_rent</th>
      <th>avg_rent_per_sqft</th>
      <th>districts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NON-LANDED HOUSING DEVELOPMENT</td>
      <td>3207</td>
      <td>2.27</td>
      <td>3533.52</td>
      <td>3.49</td>
      <td>[01, 10, 08, 15, 09, 14, 11, 23, 05, 13, 12, 0...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>THE SAIL @ MARINA BAY</td>
      <td>1016</td>
      <td>0.72</td>
      <td>5118.67</td>
      <td>6.65</td>
      <td>[01]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>D'LEEDON</td>
      <td>741</td>
      <td>0.53</td>
      <td>5072.09</td>
      <td>5.26</td>
      <td>[10]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CITY SQUARE RESIDENCES</td>
      <td>705</td>
      <td>0.50</td>
      <td>4518.13</td>
      <td>4.85</td>
      <td>[08]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>J GATEWAY</td>
      <td>701</td>
      <td>0.50</td>
      <td>4250.21</td>
      <td>6.51</td>
      <td>[22]</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ICON</td>
      <td>685</td>
      <td>0.49</td>
      <td>4640.21</td>
      <td>6.55</td>
      <td>[02]</td>
    </tr>
    <tr>
      <th>6</th>
      <td>STIRLING RESIDENCES</td>
      <td>680</td>
      <td>0.48</td>
      <td>4749.60</td>
      <td>7.46</td>
      <td>[03]</td>
    </tr>
    <tr>
      <th>7</th>
      <td>MARINA ONE RESIDENCES</td>
      <td>674</td>
      <td>0.48</td>
      <td>5404.92</td>
      <td>6.77</td>
      <td>[01]</td>
    </tr>
    <tr>
      <th>8</th>
      <td>PARC ESTA</td>
      <td>631</td>
      <td>0.45</td>
      <td>4784.14</td>
      <td>7.44</td>
      <td>[14]</td>
    </tr>
    <tr>
      <th>9</th>
      <td>WATERTOWN</td>
      <td>630</td>
      <td>0.45</td>
      <td>3326.77</td>
      <td>4.85</td>
      <td>[19]</td>
    </tr>
    <tr>
      <th>10</th>
      <td>COMMONWEALTH TOWERS</td>
      <td>618</td>
      <td>0.44</td>
      <td>4096.27</td>
      <td>7.16</td>
      <td>[03]</td>
    </tr>
    <tr>
      <th>11</th>
      <td>MELVILLE PARK</td>
      <td>610</td>
      <td>0.43</td>
      <td>3443.37</td>
      <td>3.23</td>
      <td>[18]</td>
    </tr>
    <tr>
      <th>12</th>
      <td>SIMS URBAN OASIS</td>
      <td>610</td>
      <td>0.43</td>
      <td>3529.90</td>
      <td>5.81</td>
      <td>[14]</td>
    </tr>
    <tr>
      <th>13</th>
      <td>BAYSHORE PARK</td>
      <td>590</td>
      <td>0.42</td>
      <td>3463.58</td>
      <td>3.46</td>
      <td>[16]</td>
    </tr>
    <tr>
      <th>14</th>
      <td>PARC RIVIERA</td>
      <td>590</td>
      <td>0.42</td>
      <td>3670.59</td>
      <td>5.37</td>
      <td>[05]</td>
    </tr>
  </tbody>
</table>

2.3% of the rental data obviously provided with no specified project name, so we see the 'NON-LANDED HOUSING DEVELOPMENT' filler at the top. From the rest most popular project for rent is The Sail at Marina Bay. As we can notice, number of leases doesn't correlate with rent rates in the selected projects.

### 3. Exploring the rent area-wise patterns on the map

For the geomapping I used the grouped dataframe, spatial data from the geojson file, and Folium library tools. I also collected data on private educational institutions (most of which are international schools) from OneMap API to locate them on the map.
Here is the map preview:

![Map preview](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/abb77036-2bc6-4e22-889b-335fd9b553cd)

[**LINK to the map in html**](https://raw.githack.com/maribaklanova/Singapore-rental-analysis/main/SG_rent_map.html)

The map allows us to look closer at the varying rent in Singapore neighborhoods from the central region with the most expensive rents to the north with more affordable options while assessing the location and number of schools nearby. We can also check the count of leases for the past 2 years and average rent per sq. ft.

### 4. Data visualization

I performed some data grouping and used matplotlib, Seaborn, and joyplot to visually represent the data variations, ranges, trends, and relations.

#### 4.1 Density charts of rent values for regions

![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/a9c8a1a5-6176-4b0c-8f14-4e482d85660e)

* As we can observe, The Central Region has the highest rents among regions with the distribution that is widest and most skewed upwards. So, we can expect more expensive rental properties here with a higher probability. 
* The plots for the rest regions show a more compact distribution and lower values with the narrowest range of rents in the North Region. 
* The East and North-East Regions exhibit similar distributions with close peak rent values. I took a closer look at the last two below with statistical hypothesis testing.

#### 4.2 Rent dynamics across regions

As the raw data had the limited date format of *mmyy*, I couldn't perform the proper time-series analysis. However, we can evaluate the general month-to-month dynamic.

![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/ba02bd16-8572-43f3-a096-457ec35457bd)

* In the line plot we see a notable rise in rental values in 2022 with a visible calming of the trend in 2023. 
* It also shows the distinct gap between rent in the Central region and the others as it has the highest rents, while the North Region has the lowest. 
* Again, The East and North-East Regions are closest in level and fluctuations of the rent.

#### 4.3 Number of properties leased in different Singapore areas

![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/4c0e5d5c-e153-4ae9-8773-054dd9657557)

* Here, The Central Region leads by a margin with the highest number of leased properties, followed by the East Region. 
* The bar chart also specifies planning areas within the regions: Bedok in the East and Geylang in the Central Region have significantly higher numbers of leased properties compared to other areas. 
* The North exhibits the most modest count of rental contracts.

#### 4.4 Quantity of leased properties by number of bedrooms

![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/e7b4ab18-1ab3-414b-8104-b86261a86a05)

* As for the quantity of rental contracts, this chart supports my findings from the previous one.
* The majority of leases for all regions is for 1 to 3-bedroom properties with 2-bedrooms being the most common.

#### 4.5 Average quarter rent rate by number of bedrooms

![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/614f1086-d1a4-4fba-a8b8-57b9fc0055d5)

* The average rent obviously positively correlates with the number of bedrooms with the exception of 6-bedroom properties. 
* While we can see slight fluctuations in rent prices, this chart can't show significant upward or downward trends from Q1 2022 to Q4 2023.

#### 4.6 Scattering of the data depending on property size

![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/527574dd-8b93-4710-bc14-eaee5fca7509)

* Key observations for the left scatter plot:
	- As the average area increases, there is a general trend of decreasing unit rental price per sqft: larger properties tend to have a lower unit rental price per square foot.
	- The dispersion of the values for the Central Region is slightly skewed upwards, especially for 1-3 bedroom properties.

* Key observations for the scatter plot of size-wise pricing values:
	- We can notice a general trend of the rent growth with the average floor area increasing.
	- There is an accumulation of data points at lower average areas and rent prices, which supports my previous observations of most commonly leased (not big) properties and rent distribution.
	- We can also see the wider range of the dispersion for the Central Region with an offset towards the higher rent and property sizes.

#### 4.7 Unit rent rate distributions categorized by number of rooms and regions

![](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/ede5eaf3-f2ec-456c-8cc3-1a159ca006f2)

* Rent trends by region:
	- The Central region tends to have higher rent prices per square foot compared to other regions.
	- Besides the Central region, the distributions of the unit rent rates for 1-3 bedroom properties across the regions don't significantly vary.

* Bedroom count impact is coherent with the previous findings:
	- The number of bedrooms is negatively correlated with a general trend for rent price per square foot: larger apartments (more bedrooms) mostly tend to have lower prices per square foot.
	- However, there are some interesting deviations like the non-evidence of the above-mentioned trend for properties with more than 3 bedrooms in the North-East and East regions and the North region having the highest unit rent for 5-bedroom properties.


### 5. Is there a difference between rents for 2-bedroom properties in the East and North-East regions? (statistic hypothesis test)

Based on findings from the EDA, I decided to verify if the rents of the most commonly leased — 2-bedroom — properties in the East and North-East regions are not so different.
Therefore, I sampled the data accordingly for a hypothesis test. The significance level for the tests is set at 5%.

#### Variance comparison for the samples: 

- 2-bedroom rent variance for East:  994441.4
- Size of East rent sample:  5500
- 2-bedroom rent variance for North-East:  883672.64
- Size of North-East sample:  5500
- The variance ratio: 1.1253504271652661

![The probability density distribution of the considered samples:](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/0e0f7c00-3b5a-44ff-bc4b-281ded6a71f3)

The samples do not differ significantly in variance, therefore, we use the Student's test to test the hypothesis.

#### Hypothesis:

**H0** - there's no statistically significant difference between rental costs for 2-bedroom properties in East and North-East regions
**H1** - there's a statistically significant difference between rental costs for 2-bedroom properties in East and North-East regions

#### The test results:

| | |
|---------------|------------------|
| t-statistic = 3.929  | Above the critical value for a given level of significance and number of degrees of freedom. It's positive, which means that most of the values of the 1st sample (East) are higher than those in the 2nd sample (North-East) |
| p = 0.0000858 | Under a significance level, so we reject the null hypothesis.|

Rental rates for 2-bedroom properties in the East are statistically different (mostly higher) from those in the North-East.

![The boxplots of the samples:](https://github.com/maribaklanova/Singapore-rental-analysis/assets/162949373/6a714aaa-51f4-4a9d-95d6-b62ed62e07cb)


## Results/Findings

Key findings include:
* The average rental rate is around 4000 SGD per month varying from more like 4300-4400 SGD for the Central region to 3200-3300 SGD for the North.
* The most commonly rented properties are 2-bedrooms with an average rent of ~4080 SGD as of Q4 2023.
* The Central Region is the most popular for residential rent and has the highest rents.
* North Region has the most affordable options, however the number of leases for the past 2 years in it is the lowest across the regions.
* The map provides a location-wise understanding of rental rates with less and more affordable areas along with locations of private schools within them. 
* Districts like Bedok and Geylang have stand-out high lease counts.
* Rent generally increases with the number of bedrooms (except 6-bedrooms) and the size of the floor area.
* Larger properties tend to have a lower unit rental cost per sqft.

## Recommendations

This project's results could be helpful in a few ways:

* Narrowing the search:

	- Affordability: This project highlights variations of rent rates across the regions and neighborhoods (planning areas). This can be a starting point to focus our search considering our budget.
	- Popularity: Knowing districts with high lease counts might indicate areas with more availability or a vibrant rental community.

* Understanding rental trends:

	- Rental rate change: Understanding that the 2022 rent growth trend flattened out in 2023 helps to correctly access and negotiate the rental offers. 
	- Bedroom quantity: The project suggests varying average rental rates for the properties with a different number of bedrooms. This can help to decide on the right size based on specific needs and budget expectations.
	- Cost-per-square-foot: The project hints that larger properties might have a lower cost per square foot. This could be a good option if space efficiency is prioritized.
