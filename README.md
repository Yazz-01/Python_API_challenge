What's the Weather Like APIs,JSON & Python traversals

1. Background

Python requests, Google Maps and OpenWeather APIs, and JSON traversals to answer a fundamental question: "What's the weather like as we approach the equator?" Proving it gets hotter closer to the Equator.


2. Dependencies and set up for the beginning

import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests
import time
from scipy.stats import linregress
import datetime as dt # for the datestamp on the output
import json
from pprint import pprint
import seaborn as sb

Import API key
from api_keys import weather_api_key

Incorporated citipy to determine city based on latitude and longitude
from citipy import citipy

Output File (CSV)
output_data_file = "output_data/cities.csv"

Range of latitudes and longitudes
lat_range = (-90, 90)
lng_range = (-180, 180)


Part I - WeatherPy

1. It was created a Python script to visualize the weather of 500+ cities across the world of varying distance from the equator. Uutilizing a Python library citipy and the OpenWeatherMap API (https://openweathermap.org/api) to create a representative model of weather across world cities.

2. Generate Cities List

a) List for holding coordinates(lat and longs) and cities
coordinates = []
cities = []
country = []
latitud = []
longitude = []

b) Create a set of random lat and lng combinations
lats = np.random.uniform(lat_range[0], lat_range[1], size=1500)
longs = np.random.uniform(lng_range[0], lng_range[1], size=1500)
coordinates = zip(lats, longs)

c) Identify nearest city for each lat, lng combination
for coord in coordinates:
    #lats,longs= coord
    city = citipy.nearest_city(coord[0], coord[1])
    
    # If the city is unique, then add it to a our cities list
   if city not in cities:
        cities.append(city.city_name)
        country.append(city.country_code)
        latitud.append(coord[0])
        longitude.append(coord[1])

d) Print the city count to confirm sufficient count
print("Cities", len(cities))
print("Country", len(country))
print("Lats", len(latitud))
print("Longs", len(longitude))

e) Creting the DataFrame with the nearest cities from the random coordinates 
city_dict={
    "Latitud":latitud,"Longitude":longitude,"City":cities,"Country":country}

cities_df= pd.DataFrame.from_dict(city_dict,orient='index').transpose()

f) Droping duplicates
cities_clean = cities_df.drop_duplicates("City",keep="first")

3. Perform API calls

#Weather information, saving config information

url_current = "http://api.openweathermap.org/data/2.5/weather?"
units = "imperial"

#Setting lists to replace latitud and longitude for nearest city to actual coordinates of city
actual_lat = []
actual_long = []

#Setting new weather parameters to retrieve

maxTemp = []
humidity = []
cloudiness = []
windSpeed = []
infoDate = []
city_success = []

#Variables used in the for loop for printings
#Printing the number of record 
num_record = 0

#Printing the number of set starting from 1 changing the number of sets
num_set = 1

#Printing the name of the current city
city_curr= []
country_success =[]

#Printing first message of retrieval of data
print('''Beginning Data Retrieval     
-----------------------------''')

#Looping through all the cities for weather information retrieving
for index,row in cities_clean.iterrows():
    curr_city= row["City"]
    curr_country= row["Country"]
      
#Build query URL
    query_url =f"{url_current}q={curr_city},{curr_country}&units={units}&appid={weather_api_key}"
    
   response= requests.get(query_url).json()
   
#Creating Exceptions to continue runnig the code
    num_record = num_record + 1
    try:
        actual_lat.append(response["coord"]["lat"])
        actual_long.append(response["coord"]["lon"])
        maxTemp.append(response["main"]["temp_max"])
        humidity.append(response["main"]["humidity"])
        windSpeed.append(response["wind"]["speed"])
        cloudiness.append(response["clouds"]["all"])
        country.append(response["sys"]["country"])
        infoDate.append(response["dt"])
        city_success.append(curr_city)
        country_success.append(curr_country)
        
  #Integrating the information retrieved
       
        
        print(f"Processing Record {num_record} of Set {num_set}| {curr_city}")
        
   except:
        print(f"City not found. Skipping...")
        
   if num_record == 35:
            num_set = num_set+1
            num_record = 0
            # Time between them
            time.sleep(6)
            
    #if index == 5:
     #   break
            
print('''-----------------------------
Data Retrieval Complete      
-----------------------------''')

#Displaying the DataFrame
cities_weather_df= pd.DataFrame({"City":city_success,"Lat":actual_lat,"Lng":actual_long,"Max Temp":maxTemp,
                                 "Humidity":humidity,"Cloudiness":cloudiness,"Wind Speed":windSpeed,
                                 "Country":country_success,"Date":infoDate})
                                 
 #Save dataframe in output_data file. Export the city data into a .csv.

cities_weather_df.to_csv("output_data/weather_city_data.csv", index=False, header=True)
       
#Inspecting the data and remove the cities where the humidity > 100%.
#The first requirement is to create a series of scatter plots to showcase the #following relationships:
stats = cities_weather_df.describe()


 #Get the indices of cities that have humidity over 100%.
hum = cities_weather_df.loc[(cities_weather_df["Humidity"] > 100),:]

#Making a new DataFrame equal to the city data to drop all humidity outliers by index
#Passing "inplace=False" will make a copy of the city_data DataFrame, which we call "clean_city_data".
clean_city_data = cities_weather_df.drop_duplicates("City",keep="first")

#Plotting the Data


* Temperature (F) vs. Latitude
#Create a scatter plot which compares MPG to horsepower
clean_city_data.plot(kind="scatter", x="Lat", y="Max Temp", color="skyblue", alpha=0.75, edgecolors = "black")
plt.title("City Latitude vs. Max Temperature")
plt.grid (b=True, which="major",axis="both",linestyle="-") 
plt.xlabel("Latitude")
plt.ylabel("Max Temp")
plt.savefig("Figures/Fig-1_Latitude_vs_Max Temp.png")
plt.show()

* Humidity (%) vs. Latitude
clean_city_data.plot(kind="scatter", x="Lat", y="Humidity", color="skyblue", alpha=0.75, edgecolors = "black",
              title="City Latitude vs. Humidity ")
plt.grid (b=True, which="major",axis="both",linestyle="-")  
plt.xlabel("Latitude")
plt.ylabel("Humidity")
plt.savefig("Figures/Fig-2_Latitude_vs_Humidity.png")
plt.show()

* Cloudiness (%) vs. Latitude
clean_city_data.plot(kind="scatter", x="Lat", y="Cloudiness", color="skyblue", alpha=0.75, edgecolors = "black",
              title="City Latitude vs. Coudiness (%) ")
plt.grid (b=True, which="major",axis="both",linestyle="-")  
plt.xlabel("Latitude")
plt.ylabel("Cloudiness (%)")
plt.savefig("Figures/Fig-3_Latitude_vs_Cloudiness.png")
plt.show()

* Wind Speed (mph) vs. Latitude
clean_city_data.plot(kind="scatter", x="Lat", y="Wind Speed", color="skyblue", alpha=0.75, edgecolors = "black",
              title="City Latitude vs. Wind Speed ")
plt.grid (b=True, which="major",axis="both",linestyle="-")  
plt.title(f"City Latitude vs. Wind Speed")
plt.xlabel("Latitude")
plt.ylabel("Wind Speed (mph)")
plt.grid(b=True, which="major",axis="both",linestyle="-") 
plt.savefig("Figures/Fig-4_Latitude_vs_Wind Speed.jpg")
plt.show()

After each plotting, added explations of what the code is analyzing.

3. Run linear regression on each relationship separating the plots into Northern Hemisphere (greater than or equal to 0 degrees latitude) and Southern Hemisphere (less than 0 degrees latitude):

#Using the fucntion .loc to filter the values < or > than 0. Important, use the original DF, otherwise not possible to do it.

Northern_Hemisphere = cities_weather_df.loc[cities_weather_df["Lat"] > 0]

Southern_Hemisphere = cities_weather_df.loc[cities_weather_df["Lat"] < 0]

* Northern Hemisphere - Temperature (F) vs. Latitude
x_values = Northern_Hemisphere['Lat']
y_values = Northern_Hemisphere['Max Temp']
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(-50,85),fontsize=15,color="red")
plt.xlabel('Latitude')
plt.ylabel('Max Temp')
print(f"The r-squared is: {rvalue**2}")
plt.savefig("Figures/Fig-5_NortHemphere_Latitude_vs_Wind Speed_ LinearRegression.png")
plt.show()


* Southern Hemisphere - Temperature (F) vs. Latitude

x_values = Southern_Hemisphere['Lat']
y_values = Southern_Hemisphere['Max Temp']
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(-50,85),fontsize=15,color="red")
plt.xlabel('Latitude')
plt.ylabel('Max Temp')
print(f"The r-squared is: {rvalue**2}")
plt.savefig("Figures/Fig-6_SouthHemphere_Latitude_vs_MaxTemp_ LinearRegression.png")
plt.show()

* Northern Hemisphere - Humidity (%) vs. Latitude
x_values = Northern_Hemisphere['Lat']
y_values = Northern_Hemisphere['Humidity']
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(-50,85),fontsize=15,color="red")
plt.xlabel('Norhtern Hemisphere Latitude')
plt.ylabel('Humidity')
print(f"The r-squared is: {rvalue**2}")
plt.savefig("Figures/Fig-7_NortHemphere_Latitude_vs_Humidity_ LinearRegression.png")
plt.show()

* Southern Hemisphere - Humidity (%) vs. Latitude
x_values = Southern_Hemisphere['Lat']
y_values = Southern_Hemisphere['Humidity']
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(-50,85),fontsize=15,color="red")
plt.xlabel('Southern Hemisphere Latitude')
plt.ylabel('Humidity')
print(f"The r-squared is: {rvalue**2}")
plt.savefig("Figures/Fig-8_SouthHemphere_Latitude_vs_Humidity_ LinearRegression.png")
plt.show()

* Northern Hemisphere - Cloudiness (%) vs. Latitude
x_values = Northern_Hemisphere['Lat']
y_values = Northern_Hemisphere['Cloudiness']
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(-50,85),fontsize=15,color="red")
plt.xlabel('Norhtern Hemisphere Latitude')
plt.ylabel('Cloudiness')
print(f"The r-squared is: {rvalue**2}")
plt.savefig("Figures/Fig-9_NortHemphere_Latitude_vs_Cloudiness_ LinearRegression.png")
plt.show()

* Southern Hemisphere - Cloudiness (%) vs. Latitude
x_values = Southern_Hemisphere['Lat']
y_values = Southern_Hemisphere['Cloudiness']
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(-50,85),fontsize=15,color="red")
plt.xlabel('Southern Hemisphere Latitude')
plt.ylabel('Cloudiness')
print(f"The r-squared is: {rvalue**2}")
plt.savefig("Figures/Fig-10_SouthHemphere_Latitude_vs_Cloudiness_ LinearRegression.png")
plt.show()

* Northern Hemisphere - Wind Speed (mph) vs. Latitude
x_values = Northern_Hemisphere['Lat']
y_values = Northern_Hemisphere['Wind Speed']
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(-50,85),fontsize=15,color="red")
plt.xlabel('Norhtern Hemisphere Latitude')
plt.ylabel('Wind Speed')
print(f"The r-squared is: {rvalue**2}")
plt.savefig("Figures/Fig-11_NortHemphere_Latitude_vs_Wind Speed_ LinearRegression.png")
plt.show()


* Southern Hemisphere - Wind Speed (mph) vs. Latitude
x_values = Southern_Hemisphere['Lat']
y_values = Southern_Hemisphere['Wind Speed']
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(-50,85),fontsize=15,color="red")
plt.xlabel('Southern Hemisphere Latitude')
plt.ylabel('Wind Speed')
print(f"The r-squared is: {rvalue**2}")
plt.savefig("Figures/Fig12_SouthHemphere_Latitude_vs_Wind Speed_ LinearRegression.png")
plt.show()


### Part II - VacationPy
1. Creating a heat map that displays the humidity for every city from Part I.

#Dependencies and Setup
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests
import gmaps
import os

#Import API key
from api_keys import g_key

#Reaing the csv file
cities_csv = "../Weatherpy/output_data/weather_city_data.csv"
weather_cities = pd.read_csv(cities_csv)

#Configuring the gmaps
gmaps.configure(api_key=g_key)

2. Store 'Lat' and 'Lng' into  locations 
locations = weather_cities[["Lat","Lng"]].astype(float)

#Convert Poverty Rate to float and store
humidity = weather_cities["Humidity"].astype(float)

#DataFrame to find ideal weather condition
#A max temperature lower than 80 degrees but higher than 70.

ideal_weather = weather_cities.loc[
    (weather_cities["Max Temp"] >=70) & 
    (weather_cities["Max Temp"] <=80)&
    (weather_cities["Wind Speed"] <=10)&
    (weather_cities["Cloudiness"] == 0)]

len(ideal_weather)


#Drop any rows that don't contain all three conditions. You want to be sure the weather is ideal.
ideal_weather= ideal_weather.dropna()

#Store into variable named hotel_df.
hotel_df = ideal_weather.reset_index()

#Add a "Hotel Name" column to the DataFrame.
hotel_df["Hotel Name"] = ""

# Hit the Google Places API for each city's coordinates.

# set up a parameters dictionary
params = {
    "radius": 5000,
    "types": "lodging",
    "key": g_key
}

curr_hotel
#Looping through all the information retrieving
for index,row in hotel_df.iterrows():
    lat= row["Lat"]
    lng= row["Lng"]
    
   params["location"] = f"{lat},{lng}"
    
  #Use the search term: "Hotel" and our lat/lng
    base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"

  #make request and print url
    name_address = requests.get(base_url, params=params)
    
   #convert to json
    name_address = name_address.json()
    
   #Grab the first hotel from the results and store the name
    try:
        hotel_df.loc[index, "Hotel Name"] = name_address["results"][0]["name"]
    except (KeyError, IndexError):
        print("Missing field/result... skipping.")

#Using the template add the hotel marks to the heatmap
info_box_template = """
<dl>
<dt>Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
</dl>
"""
#Store the DataFrame Row

hotel_info = [info_box_template.format(**row) for index, row in hotel_df.iterrows()]
locations = hotel_df[["Lat", "Lng"]]

3. Add marker layer ontop of heat map
#Set parameters to search for hotels with 5000 meters.
locations.head()
#dataframe with columns ('latitude', 'longitude', 'magnitude')
fig = gmaps.figure(map_type='HYBRID')
heatmap_layer = gmaps.heatmap_layer(locations)
markers = gmaps.marker_layer(locations, info_box_content = hotel_info)
fig.add_layer(heatmap_layer)
fig.add_layer(markers)

#Display figure
fig



