# CS_Dynamic_Urban_Planning
Dynamic Urban Planning: An Agent-Based Model Coupling Mobility Mode and Housing Choice.

This ABM aims at characterizing citizens' behavioral patterns regarding their residential mobility and housing choice. The identification of these criteria will allow us to predict the consequences that potential urban disruptions and housing-related incentives might entail.

## Instructions

### A. Installation of GAMA

1. Clone this repository (clicking on the green Clone or download button on this page - to manage your repo you can also install Github Desktop on your computer
2. Download GAMA (compatible with GAMA 1.8) [here] (https://gama-platform.github.io/download)
3. Run GAMA
4. Choose a new Workspace (this is a temporary folder used for computation)
5. Right click on User Models --> Import --> GAMA Project
6. Select CS_Dynamic_Urban Planning


### B. Adaptation of the model

This model presents a generic framework where, as long as the GIS data is adapted to a city and suitable census and transportation data is included.

### B.1 Calibration of the model

If you want to get the region-specific housing and mobility mode criteria, use the following steps with the `calibrateDataPercentage.gaml` file.

#### Shapefiles

1. Set the `blockGroup_shapefile`containing the census block group shapefile of the selected city. It should include the following attributes: `GEOID` unique ID, `INTPLAT` latitude, `INTPTLON` longitude, `TOWN`, `Neighbourh`neighbourhood.
2. Set the `available_apartments`containing the rental options (through a previous web scrapping process) in th earea. It should include the following attributes: `Rent` rental cost, `NBedrooms`number of available bedrooms`, `GEOID` unique ID. 
3. Set the `buildings_shapefile` containing the physical buildings in the area of interest. The attributes should be: `Usage` [Residential, Office...], `Scale`[S: small, M: medium, L: large], `Category`[commercial, restaurant...], `FAR` permitted floor area ratio, `Max_Height` maximum height, `TYPE`, `NAME`of the neighbourhood, `BUILDING_I` building ID, `Latitude`, `Longitude`, `GEOID` unique geographical ID.
4. Set the `T_lines_shapefile` containing the metro line topology. Attributes: `colorLine` color of the metro line.
5. Set the `T_stops_shapefile`containing the bus stops in the area. No attributes.
6. Set the `road_shapefile` containing the road topology of the area. No attributes.


#### .csv files

1. Set `criteria_home_file`. This file should include the types of citizen agents and the selected housing- and mobility-mode-related criteria that you want to calibrate. The values included in this file will be used for the initialization of the exploration process.
2. Set `activity_file`. It should describe the hourly schedule of citizen agents based on their profile
3. Set `mode_file`. This file the price, speed, waiting time... characteristics of each of the available mobility modes
4. Set `profile_file`. List of citizen agent profiles, the proportion within the total considered population, probability of owning a car, and probability of owning a bike.
5. Set `criteria_file`. This file should include the mobility-related parameters that each citizen agent profile takes into account when heading to each building type.
5. Set `population_file`. It is necessary to modify this file and adapt it to the available census data for each case:

| Block group Id1 | Block group Id2 | Total amount of citizens | Amount of citizens of profile 0 | ... | Amount of citizens of profile i | Diversity (Shannon Weaver) | Normalized diversity |
| --------------- | --------------- | ------------------------ | ------------------------------- | --- | ------------------------------- | -------------------------- | -------------------- |

6. Set `real_Kendall_data_file`. This file identifies the amount of workers from the finer granulated area (base case: Kendall) that live in each census block group.

|Block Group id2 | Number of workers living there | Profile |

7. Set `real_mobility_data_file`. Distribution of usage for each transportation mode available in the area of interest.

Once these files are adapted for the desired district/city/area, running the `calibrateDataPercentage.gaml` model in batch mode leads to the identification of mobility mode and housing parameters that each profile takes into account when making these decisions. The methodology used for this purpose consists on the minimization of housing and mobility errors between the simulation and the imported real data.


### B.2 Main model

Model `mainModel.gaml` uses the calibrated criteria obtained from `calibrateDataPercentage.gaml` and stored in `"../includes/Criteria/incentivizedCriteria/CriteriaFileCalibrated.csv"` and `"./../includes/Criteria/incentivizedCriteria/CriteriaHomeCalibrated.csv"`and simulates citizens' reactions to various urban disruptions and financial incentives.
In order to adapt this model to the desired area, follow these instructions:

1. Import shapefiles and .csv files to `mainModel.gaml` the same way you did for the `calibrateDataPercentage.gaml`model. The only difference lies in `criteria_home_file` and `criteria_file` where the calibrated parameters are included.
2.  Run the `batch_save` experiment in order to get the **what if** scenarios for different amount of extra housing area built and financial incentives given.


### B.3. Training of the regressor model

 

