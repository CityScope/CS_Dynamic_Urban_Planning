# CS_Dynamic_Urban_Planning
Dynamic Urban Planning: An Agent-Based Model Coupling Mobility Mode and Housing Choice.

This ABM aims at characterizing citizens' behavioral patterns regarding their residential mobility and housing choice. The identification of these criteria will allow us to predict the consequences that potential urban disruptions and housing-related incentives might entail.

## Instructions

### A. Installation of GAMA

1. Clone this repository (clicking on the green Clone or download button on this page - to manage your repo you can also install Github Desktop on your computer)
2. Download GAMA (compatible with GAMA 1.8) (https://gama-platform.github.io/download)
3. Run GAMA
4. Choose a new Workspace (this is a temporary folder used for computation)
5. Right click on User Models --> Import --> GAMA Project
6. Select CS_Dynamic_Urban Planning


### B. Adaptation of the model

This model presents a generic framework that can be adapted to any city as long as the GIS data is changed and suitable census and transportation data is included.

### B.1 Calibration of the model

If you want to get the region-specific housing and mobility mode criteria, follow these steps and modify `calibrateDataPercentage.gaml` file.

#### Shapefiles

1. Set the `blockGroup_shapefile`containing the census block groups shapefile of the selected area. It should include the following attributes: `GEOID` unique ID, `INTPLAT` latitude, `INTPTLON` longitude, `TOWN`, `Neighbourh` neighbourhood.
2. Set the `available_apartments`containing the rental options (through a previous web scrapping process) in the area. It should include the following attributes: `Rent` rental cost, `NBedrooms` number of available bedrooms, `GEOID` unique ID. 
3. Set the `buildings_shapefile` containing the physical buildings in the area of interest. The attributes should be: `Usage` [Residential, Office...], `Scale`[S: small, M: medium, L: large], `Category`[commercial, restaurant...], `FAR` permitted floor area ratio, `Max_Height` maximum height, `TYPE`, `NAME` of the neighbourhood, `BUILDING_I` building ID, `Latitude`, `Longitude`, `GEOID` unique geographical ID.
4. Set the `T_lines_shapefile` containing the metro line topology. Attributes: `colorLine` color of the metro line.
5. Set the `T_stops_shapefile` containing the bus stops in the area. No attributes.
6. Set the `road_shapefile` containing the road topology of the area. No attributes.


#### .csv files

1. Set `criteria_home_file`. This file should include the types of citizen agents and the selected housing- and mobility-mode-related criteria that you want to calibrate. The values included in this file will be used for initializing the exploration process.
2. Set `activity_file`. It should describe the hourly schedule of citizen agents based on their profile.
3. Set `mode_file`. This file includes the price, speed, waiting time... characteristics of each of the available mobility modes.
4. Set `profile_file`. List of citizen agent profiles, the proportion within the total population, probability of owning a car, and probability of owning a bike.
5. Set `criteria_file`. This file should include the mobility-related parameters that each citizen profile takes into account when heading to a specific place. The values will be used for initializing the exploration process.
5. Set `population_file`. It is necessary to modify this file and adapt it to the available census data for each case:

| Block group Id1 | Block group Id2 | Total amount of citizens | Amount of citizens of profile 0 | ... | Amount of citizens of profile n | Diversity (Shannon Weaver) | Normalized diversity |
| --------------- | --------------- | ------------------------ | ------------------------------- | --- | ------------------------------- | -------------------------- | -------------------- |

6. Set `real_Kendall_data_file`. This file identifies the amount of workers from the finer granulated area (base case: Kendall) that live in each census block group.

|Block Group id2 | Number of workers living there | Profile |
| -------------- | ------------------------------ | ------- |

7. Set `real_mobility_data_file`. Distribution of usage for each transportation mode available in the area of interest.

Once these files are adapted for the desired district/city/area, running the `calibrateDataPercentage.gaml` model in batch mode leads to the identification of mobility mode and housing parameters that each profile takes into account when making these decisions. The methodology used for this purpose consists on the minimization of housing and mobility errors between the simulation and the imported real data.


### B.2 Main model

Model `mainModel.gaml` uses the calibrated criteria obtained from `calibrateDataPercentage.gaml` and stored in `"../includes/Criteria/incentivizedCriteria/CriteriaFileCalibrated.csv"` and `"./../includes/Criteria/incentivizedCriteria/CriteriaHomeCalibrated.csv"`and simulates citizens' reactions to various urban disruptions and financial incentives.
In order to adapt this model to the desired area, follow these instructions:

1. Import shapefiles and .csv files to `mainModel.gaml` the same way you did for the `calibrateDataPercentage.gaml`model. The only difference lies in `criteria_home_file` and `criteria_file`. This time the mentioned files should include the calibrated parameters.
2.  Run the `batch_save` experiment in order to get the **what if** scenarios for different amount of extra housing area built and financial incentives given.


### B.3. Training of the regressor model

This is a python script that trains the Response Surfaces linked to the following outputs: (1) the percentage of people working and living in the selected area according to their profile and depending on the amount of housing area built and the financial stimuli offered, (2) the construction area --grid-- occupancy rate, (3) the distribution of differnte mobility modes usage, (4) the mean commuting time, and (5) the mean commuting distance.
This allows to obtain the mentioned urban metrics real time. A k-neighbour regressor is deployed for this. 80% of the batch experiments will be used for training purposes, and 20% for testing. In order to be able to use this script for your specific use case, follow these instructions:

1. Open the `predictingValues.py` script located in the `results` folder.
2. Change `nameFileIn` and include the file where the results of the batch experiments are collected. This .csv file should follow this structure (it is prepared for 8 citizen agent profiles and 5 mobility modes, minor changes to reduce/increase this values would be needed):

| Total proportion of workers living in the area of interest | Proportion of workers with profile 0 | ... | Proportion of workers with profile 7 | Usage of mobility mode 1 | ... | Usage of mobility mode 4 | Mean commuting time | Mean commuting distance |  Construction site occupancy |
| ---------------------------------------------------------- | ------------------------------------ | --- | ------------------------------------ | ------------------------ | --- | ------------------------ | ------------------- | ----------------------- | ---------------------------- |

3. Select `nameFileOut`. This file will gather the results obtained from the regressor.
4. Select `nameStatFile` . This file includes the statistical results (mainly R^2 and RMSE values) of the regressor.


### B.4. CityScopable Model

The results obtained from the regressor model will be used to feed the real-time model or "CityScopable" model. The aforementioned urban-metrics will define the t=0 scenario that can then be used to visualize how the commuting process can be altered thanks to the suggested urban disruptions and incentives. To adapt this model to your specific use case:

1. Import the `buildings_shapefile`, `roads_shapefile`, `busStops_shapefile`, `TStops_shapefile` and `Tline_shapefile` just as you did in B.1. and B.2. This time these shapefiles will include only the area of interest (base case: Kendall) and not the surroundings (base case: Greater Boston Area).
2. Set the `entry_point_shapefile`. This file should include the entry points to the selected area, either if they are road entry points or metro/train entry points. Attributes: `mobility` indicates the type of entry point (road or metro).
3. Set the .csv files containing the results of the regressor. These could be the what-if scenarios that have been created for the calibrated case or for any hypothetical behavioral incentives that might have changed citizen agents' decision-making parameters.
4. Set `activity_file` with the daily schedule of each citizen profile.
5. Set `originalProfiles` (equivalent to `profile_file` in B.1.) and `mode_file` just as you did for B.1.

You are now ready to run the CityScopable gui and easily visualize citizen agents' reactions to potential urban disruptions and housing-related incentives.
 

