---
layout: project
title: "Utah Air Particulate Analysis"
author: "Greg Lee"
categories: documentation
tags: [documentation]
image: pm_slc.jpg
---
Particulate matter 2.5 (PM2.5) is a carcinogenic material associated with adverse cardiovascular events and cancer incidence. In the winter, Utah experiences acute inversions which trap particulates close to the urban population. The study of these particulates are critical in helping the Utahn population properly mitigate exposure. Herein, a spatial analysis is performed on scraped, hourly PurpleAir and AirNow PM2.5 exposures from April 7th-14th. Primary findings demonstrate PurpleAir sensors to have better rural coverage and decreased accuracy when compared to the Environmental Protection Agency's (EPA) AirNow measurements.
<br>
<hr>
## Background
The Salt Lake City basin is an established outdoor kingpin with a rapidly expanding technology sector. The geographic shape of the region leads to “sinkholes” of pollution with notably high concentrations of particulate matter and ozone. With reduced wind and precipitation, warm air traps pollutants in the basin resulting in high pollutant concentrations, breathable by city dwellers. These trapped toxins are dangerous to human health. Particulate matter 2.5 (PM2.5) is robustly associated with many negative disease outcomes including non-fatal heart attacks, asthma and increased cancer risk.

![Inversion from Mt. Baldy, Alta]({{ site.github.url }}/assets/img/purp_air/inversion.JPG){:class="project-featured-image"}
<br>
<br>
##  Data Acquisition
Two entities which monitor and predict particulate matter concentrations are the Environmental Protection Agency (EPA) and Purple Air. The EPA tracks air quality in real-time via a network of sensors known as [AirNow](https://www.airnow.gov/). These data are available at minute intervals, starting in 1990, via an API on their website. The spatial coverage in Utah is limited since there are only a few sensors (N=27 in 2020) nearby areas of high urban density. [PurpleAir](https://www2.purpleair.com/) (1030 Legacy Sensors and 524 Experimental Sensors in 2020) publicly release real-time data via a web portal with no back-log. One advantage to PurpleAir data is its widespread spatial coverage, especially in the Salt Lake City basin. 

![Sensor Locations]({{ site.github.url }}/assets/img/purp_air/sensor_locations.png){:class='project-featured-image'}

AirNow and PurpleAir sensors detect several different pollutants including particulate matter and ozone. PurpleAir tracks a greater number of particulates, notably PM2.5, PM10 and volatile organic compounds while AirNow sensors only record PM2.5. Additionally, PurpleAir sensors measure temperature, humidity, and pressure. Within Utah, the tracking, analysis and forecasting of PM2.5 is critical in protecting public health. Few studies analyze the PurpleAir data as public data access is limited. In order to explore how the COVID-19 pandemic stay-at-home guidelines are affecting air quality in Utah, I collected and analyzed hourly PM2.5 data from PurpleAir and EPA AirNow for the week of April 7th to 14th. Please note, both data repositories are publicly accessible. Often, PurpleAir sensors are placed on residential properties and users have consented for their information to be shared publicly. AirNow sensor data are curated and released by the EPA. Data does not contain any information about the property or residents, simply the location (Lat/Lon). For additional questions concerning data privacy, please visit the [PurpleAir](https://www2.purpleair.com/policies/privacy-policy) [AirNow](https://docs.airnowapi.org/faq) websites.

Hourly AirNow measurements were queried from their [API](https://docs.airnowapi.org/)from April 7th to April 14th for 2019 and 2020. The PurpleAir data collection was a bit more complex as data had to be collected in real-time and downloaded to a local database. In order to automate this task, a directed acyclical graph (DAG) was created in Apache Airflow. This program collected hourly measurements starting April 7th at 5:00 AM MST till April 14th at 5:00AM MST. Code for this application can be found within the repository under [purp_air.py](needgithublink). It was necessary to run this program with an Airflow pipeline as the collection [link](https://docs.google.com/document/d/15ijz94dXJ-YAZLi9iZ_RaBwrZ4KtYeCy08goGBwnbCU/edit) was inconsistent, occasionally rejecting requests. Airflow is capable hierarchical collection, forcing the program to retry until collection was completed.

<br>
<br>
## Cleaning and Quality Control 
Data cleaning represented the crux of this project due to data sparsity and standardization. The PurpleAir data contained roughly 170,000 entries recorded by 1,030 sensors. Any sensors which showcased values over 150 [CONC] where removed as no areas in Utah has remarkably high PM2.5 during the chosen analysis week. Visualization of the null values showcased the PurpleAir data to be missing approximately 6% of PM2.5 values and 45% of weather variables such as pressure, temperature and humidity. The AirNow data contained no missing values. In order to fill the PurpleAir missing values, an algorithm was created to fill values based upon the closest sensor, both in space and time. This filling process was not perfect and several values had to be annotated by hand. For this subset, values were filled with the global average of the data. After running, several points had to be dealt with by hand as no other sensors of reasonable proximity had data for the desired time point. 

<img src="{{ site.github.url }}/assets/img/purp_air/preplot_nearest_sensor.png" width="380" height="380">

Once the missing data was filled, the accuracy of the PurpleAir sensor data was examined. In order to gauge error, average PM2.5 levels were calculated for zip-code level aggregations which contained both PurpleAir and AirNow sensors (Average per zip: 13 PurpleAir and 1 AirNow). On average, the PurpleAir sensors differed by 2.06 ug/m^3, a value greater than one standard deviation of the AirNow data (std = 1.27 ug/m^3). While the zip-code aggregation introduces some error to this analysis, the difference is great enough to conclude the PurpleAir data is significantly inaccurate.


<img src="{{ site.github.url }}/assets/img/purp_air/sensor_diffs.png" width="400" height="400">


Modeling was tested as method to alleviate the error of the PurpleAir measurements. Multiple-linear regression was performed to predict the zip-code level AirNow concentrations from the zip-code aggregated PurpleAir PM2.5 and associated meteorological variables alongside an approximate time variable. As expected, the explained variance was low, ~55%, implying there was no reliable method to adjust the PurpleAir concentrations to reflect the AirNow sensor accuracy. In order to avoid erroneous conclusions, only the trends of the PurpleAir data were analyzed. With more data, better models might be developed to properly adjust the PurpleAir concentrations so accuracy could be improved. Unfortunately, the AirNow coverage is limited to urban-dense areas so model generalizability to rural areas would be unlikely. 

<br>
<br>
## Results
Daily and hourly PM2.5 concentrations was analyzed with zip code level aggregation for both PurpleAir and EPA AirNow measurements. Overall, PM2.5 concentrations in the state of Utah were below 10 for the week of April 7th – 14th for both 2019 and 2020. This is unsurprising as the air quality was recorded as "good" with no stay in advisories for both years. The weather was sunny and without heavy wind allowing for reliable sensor measurements, although two methods of quantification still exhibit disagreement.  

<img src="{{ site.github.url }}/assets/img/purp_air/state_aggregates.png" width="1000" height="250">

Zip code analysis of PurpleAir PM2.5 concentrations elucidated up-regulated PM2.5 in Zip Codes XXX which translate into the counties of Coalville, Goshen and North Logan.

<img src="{{ site.github.url }}/assets/img/purp_air/coalville_lineplot.png" width="1000" height="250">

 These observations were missed by the AirNow monitors due to their limited spatial coverage. This emphasizes one strength of the PurpleAir data: spatial coverage for rural populations! While these measures may not be highly precise, they allow for general trend capture which may help alert sensitive populations to stay inside to minimize risk of an adverse health event. 

<img src="{{ site.github.url }}/assets/img/purp_air/Rural_Sensor_cover_more.png" width="1000" height="300">

Examination of daily trends showcased that AirNow monitors may also smooth their data, as temporal cycles expected in PM2.5 concentration due to wind and anthropocentric factors were indistinguishable. In contrast, PurpleAir data is more sensitive to these cycles with some evidence of bimodal daily peaks around 9AM and 5PM. This suspicion is confirmed by AirNow and PurpleAir [documentation](https://www2.purpleair.com/community/faq#!hc-how-do-purpleair-sensors-compare-to-regulatory-particulate-matter-sensors) which showcases the Airnow data to use 24 hour smoothing filter. When smoothed with a 24 hour rolling filter, the PurpleAir data appears to mirror the trends of the AirNow data, although the magnitude of the concentrations remain inaccurate inaccurate.

![PurpleAir vs EPA AirNow]({{ site.github.url }}/assets/img/purp_air/hourlypm25_84105.png)
![Smoothed PurpleAir]({{ site.github.url }}/assets/img/purp_air/smoothed_purpleair.png)

<br>
<br>
## Limitations and Future Plans
The purpose of this project was to explore how the COVID-19 stay-at-home orders affected the air quality within the state of Utah. Unfortunately, the size of data, sensor accuracy and season of analysis severely dampen the significance of any conclusions reached. Overall, a weeks worth of hourly data was collected for >1000 sensors equating to roughly 200,000 data points. These data were spatially concentrate in the Salt Lake Basin and failed to have good representation in rural cities. As a whole, PM2.5 is affected by many highly variable factors such as meteorology and traffic patterns. Much more data would be necessary to concretely isolate signal from noise. The method of PM2.5 concentration calculation differs between PurpleAir and AirNow sensors further confounding the accuracy of measurements. PurpleAir sensors use a [laser particle counter](https://www2.purpleair.com/community/faq#!hc-how-do-purpleair-sensors-compare-to-regulatory-particulate-matter-sensors) while AirNow sensors use a [filter weight measurement](https://www2.purpleair.com/community/faq#!hc-how-do-purpleair-sensors-compare-to-regulatory-particulate-matter-sensors). While more accurate, the AirNow sensors are expensive and hard to maintain. In order to make the technology widely available to the public, PurpleAir sensors have reduced accuracy, subject to miscalculation during times of precipitation as the moisture occludes the laser quantification. The sensor quantification methods and resulting error limit the accuracy of any conclusions. To combat this spatial aggregation and averaging was utilized in any comparison in an attempt better model the air quality. PM2.5 concentrations in Utah are highly dependent on the season with winter months (December-March) being notorious for poor air quality. As the examined data does not include any seasonality, it fails to examine days where the air pollution is particularly hazardous. As such, distinguishing any noticeable difference correlated with the COVID-19 stay-at-home orders as the signal of interest is muddled by sensor noise. In total, this analysis best serves as practice with collecting, manipulating and visualizing geospatial data in python as any conclusions are occluded by small data size, sensor accuracy and seasonality. 

With an increased amount of temporal data, trends of PM2.5 concentrations may be better understood and predicted. Seasonality could be removed from the data yielding more accurate conclusions. Greater temporal coverage would introduce a higher percentage of hazardous PM2.5 concentrations into the data. Having a greater percentage of "bad air days" would be beneficial as this is the main outcome of interest with air quality. If trained with a large amount of PurpleAir data and proxies for atmospheric changes, deep learning algorithms may be trained to predict PM2.5 levels. If more data was captured, filling of missing data may become less necessary, decreasing any bias introduced by spatial filling. 
<br>
<br>

## Conclusion
Utah showcased non-hazardous levels of PM2.5 for the week of April 7th, in both 2019 and 2020. Through this biased lens, it appears the COVID-19 stay-at-home orders did not reduce PM2.5 levels despite reduced driving and business. These results are arbitrary as the data sample size limits the impact of the analysis and the conclusions are confounded by factors such as season and spike. 

This analysis best serves to demonstrate the “wrangling” attitude necessary to tease meaning from unrefined spatial and temporal data. There is noticeable, unpredictable error between the PurpleAir and AirNow PM2.5 measurements making the magnitude of exposure difficult to quantify. PurpleAir sensors have higher spatial representation in Utah than AirNow sensors which may be beneficial in alerting rural populations of toxic air. This greater spacial variability is partially due to an increased number of PurpleAir Sensors. In total, the PurpleAir data represents a data-resource for fine spatial monitoring of rural and urban PM2.5 concentrations. These data are best used for teasing apart general air quality trends and accentuate the need for improved real-time monitors which are inexpensive and accessible.  

Visually though (my biased view of course!), the haze seems lessened with the sunsets shimmering onward wondrously. 