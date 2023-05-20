
Flight Tracking System by Using Python




## Author

Muhammad Bilal Rafique (192089)


## Introduction

A flight tracking system is a system that is used to track the movement of planes in airspace. The system provides live data of the airplanes' location, altitude, speed, and direction. This system is beneficial for air traffic controllers and airlines to monitor air traffic and ensure safety. Moreover, flight tracking systems are also used for flight planning, flight management, and providing real-time flight information to passengers.

 Python is a powerful programming language that is widely used for data analysis, machine learning, and artificial intelligence. In this project, I will be using Python to develop a flight tracking system that will retrieve live air traffic data from OpenSky Network and plot the location of airplanes on a map using the Bokeh library. The system will provide real-time data on the airplanes' location, altitude, and speed.

 In this project, I will discuss the process of getting air traffic data, processing the data in Python, the type of API file, the libraries used in this project, how the code works for monitoring air traffic control in real-time, maps and airline data, and further improvements in the project.
## Getting Air Traffic Data

In this project, I am using open air traffic data from OpenSky Network. OpenSky Network is a non-profit consortium that provides open air traffic data to the public, particularly for research and non-commercial purposes. The data can be accessed through the REST API, Python API, and Java API. In this project, I'm using the REST API to retrieve live air traffic data.
 

To retrieve the data using the REST API, I can use the request operation. There are two types of requests that can be used. The first one is a request for a specific airplane based on time in UNIX timestamp or ICAO24 address. The second one can get all airplane data within an area extent using the WGS84 coordinate system. Moreover, access to the data can be done anonymously or as a registered user. For anonymously requesting data, it has a 10-second resolution and 5 seconds for registered users.

 In this project, I'll be using the second type of request. I'll define an area extent with minimum and maximum coordinates and then send the query to get all airplane data within the area. For example, we want to fetch data over the Republic of Poland with a minimum coordinate of 14.12, 49.00, and a maximum coordinate of 24.15, 54.83. The query for a registered user will be as follows:

#REST API QUERY

user_name = 'Bilal_Rafique'

password = '***********'

url_data = 'https://' + user_name + ':' + password + '@opensky-network.org/api/states/all?' +
'lamin=' + str (lat_min) + '&lomin=' + str (lon_min) + '&lamax=' + str (lat_max) + '&lomax='+ str (lon_max)

response = requests . get(url_data).json()

Now, I'll copy the registered user request and paste it into a browser, and after getting a response, it seems to work. 

The return response is in JSON format with two keys. The first one is time, and the second one is states, which contains data for each airplane in a list array. The list array stores many data types, such as ICAO24 address, airplane call sign, origin country, time position, last contact, longitude, latitude, barometer altitude, and so on. For a complete explanation of the data response and further information about the OpenSky Network API, please refer to the OpenSky Network API Documentation.

## Processing Air Traffic Data in Python

In this project, I'm using Jupyter Notebook with Python 3.8.2 and some other libraries such as Bokeh 2.1.1, Pandas 0.25.3, requests, json, and numpy. I'll first define the extent coordinate in WGS84 with the respective variables lon_min, lat_min, lon_max, and lat_max. Based on the extent coordinate, I'll make the request query using the requests library. If you are a registered user, give the username and password in the user_name and password variables. Based on the given extent coordinates and user data, I'll create a request query and get the response in JSON format.

The response data will be dumped into Panda's data frame, and the blank or empty data will be replaced with a "NaN' value with 'No Data'. The data frame's head method will be used to view the first five top rows of the data.
## Type of API File

The OpenSky Network API provides two types of data files for air traffic data: states and flights. The states file contains the current state vector data for each aircraft, while the flights file contains historical and predicted flight data for each aircraft. In this project, I'm using the states file because I'm interested in real-time data.
## Libraries Used in this Project

In this project, I'm using several libraries in Python to process and visualize the air traffic data. The libraries used are:
 
1. requests: to make HTTP requests and retrieve data from the OpenSky Network API.
2. JSON: to convert the retrieved data from JSON format to a Python dictionary.
3. pandas: to store and manipulate the data in a tabular format.
4. bokeh: to create interactive visualizations and plots.
5. NumPy: to perform numerical operations on arrays and matrices. 
## Plotting Airplane on the Map

After retrieving the data, I plotted the location of all airplanes on a map using the Bokeh library. Then, I'll import the library and NumPy library that were used in coordinate conversion. I implemented a coordinate conversion function called wgs84_web_mercator_ that transformed WGS84 coordinates into the Web Mercator coordinate system. This transformation was required because we used a STAMEN_TERRAIN base map in a web browser that has a web Mercator projection (EPSG: 3857).

After creating the coordinate conversion function and performing the transformation for data frame and extent coordinates, I set up the plotting figure settings. We used the Mercator projection for the x-axis and y-axis and plotted the latitude and longitude of the airplanes on the map. I also added a circle glyph to represent the location of each airplane, with the size of the circle representing the altitude of the airplane.

The following is the code until this step.

#IMPORT PLOTTING LIBRARIES

from bokeh.plotting import figure, show

from bokeh.tile_providers import get_provider,STAMEN_TERRAIN

from bokeh.models import HoverTool,LabelSet,ColumnDataSource

import numpy as np

#FUNCTION TO CONVERT GCS WGS84 TO WEB MERCATOR

#POINT

def wgs84_web_mercator_point(lon,lat):

k = 6378137

x= lon * (k * np.pi/180.0)

y= np.log(np.tan((90 + lat) * np.pi/360.0)) * k

return x,y

#DATA FRAME

def wgs84_to_web_mercator(df, lon="long", lat="lat"):

k = 6378137

df["x"] = df[lon] * (k * np.pi/180.0)

df["y"] = np.log(np.tan((90 + df[lat]) * np.pi/360.0)) * k

return df

#COORDINATE CONVERSION

xy_min=wgs84_web_mercator_point(lon_min,lat_min)

xy_max=wgs84_web_mercator_point(lon_max,lat_max)

wgs84_to_web_mercator(flight_df)

flight_df['rot_angle']=flight_df['true_track']*-1 #Rotation angle

icon_url='https://.....' #Icon url

flight_df['url']=icon_url

#FIGURE SETTING

x_range,y_range=([xy_min[0],xy_max[0]], [xy_min[1],xy_max[1]])

p=figure(x_range=x_range,y_range=y_range,x_axis_type='mercator',
y_axis_type='mercator', sizing_mode='scale_width',plot_height=300)

#PLOT BASEMAP AND AIRPLANE POINTS

flight_source=ColumnDataSource(flight_df)

tile_prov=get_provider(STAMEN_TERRAIN)

p.add_tile(tile_prov,level='image')

p.image_url(url='url', x='x', y='y',source=flight_source,anchor='center',angle_units='deg',angle='rot_angle',h_units='scree
n',w_units='screen',w=40,h=40)

p.circle('x','y',source=flight_source,fill_color='red' hover_color='yellow',size=10,fill_alpha=0.8,line_width=0)

#HOVER INFORMATION AND LABEL

my_hover=HoverTool()

my_hover.tooltips=[('Call sign','@callsign'),('Origin
Country','@origin_country'),('velocity(m/s)','@velocity'),('Altitude(m)','@baro_altitude')]

labels = LabelSet(x='x', y='y', text='callsign', level='glyph',
x_offset=5, y_offset=5, source=flight_source,render_mode='canvas',background_fill_color='white',text_font_size="8pt")

p.add_tools(my_hover)

p.add_layout(labels)

show(p)


##  How Does Code Work For Monitoring Air Traffic Control In Real Time?

The code for monitoring air traffic control in real-time works in the following steps:

1. Define the area of interest by specifying the minimum and maximum latitude and longitude coordinates.
2. Make a request to the OpenSky Network API to retrieve the current state vector data for all aircraft within the specified area.
3. Convert the retrieved data from JSON format to a Python dictionary using the JSON
library.

4. Store the data in a panda Data Frame for easier manipulation and analysis.

5. Clean the data by replacing any missing or empty values with NaN values.

6. Convert the latitude and longitude coordinates from WGS84 to Web Mercator
projection using the wgs84_web_mercator_ function.

7. Create a bokeh plot of the aircraft positions on a map using the Mercator projection.

8. Add interactive features to the plot, such as hovering over the aircraft icons to display their call signs and other information.
9. Update the plot every few seconds to display the latest positions of the aircraft.
## Maps and Airline Data

To visualize the air traffic data, I used the Bokeh library to create interactive maps and plots. Bokeh is a Python library for creating interactive visualizations and plots in modern web browsers. Bokeh provides a high-level interface for creating beautiful and interactive visualizations that can be easily customized and embedded into web applications.

For the map visualization, I used the STAMEN_TERRAIN tiles from Bokeh's built-in tile providers. These tiles provide a high-resolution map with terrain and labels for cities and other landmarks.

In addition to the air traffic data, I also used airline data to add more information to the plot. I obtained airline data from the OpenFlights database, which contains information on airlines, airports, and routes. I used the airline data to display the airline logos next to the aircraft icons on the map.
## Building Flight Tracking Application

Now that I have successfully obtained and processed the air traffic data and plotted the airplanes on a map, I can move forward with building a flight tracking application. In this section, I will discuss the steps needed to build a simple flight tracking application using Python.

1. GUI (Graphical User Interface)

The first step in building the flight tracking application is to design a graphical user interface
(GUI) that will be used by the user to interact with the application. There are several GUI
libraries available in Python such as tkinter, PyQt, wxPython, and PyGTK, among others. For
this tutorial, we will be using tkinter which is a built-in Python library for creating GUIs.
The GUI will have a simple layout consisting of a map area to display the airplane locations,
a drop-down menu to select the country, and a button to update the map. The user will be ableto select a country from the drop-down menu, and the application will fetch the latest air
traffic data for that country and display the airplanes on the map.

2. Fetching Data
To fetch the latest air traffic data for the selected country, we will modify the code we wrote
earlier to include a function that takes the country as an input and returns the air traffic data
for that country. The function will use the same REST API call we used earlier, with the only
difference being that we will pass the selected country as a parameter.

3. Updating the Map
Once we have fetched the latest air traffic data for the selected country, we need to update the
map to display the airplane locations. To do this, we will modify the plotting code we wrote
earlier to include a function that takes the air traffic data as input and updates the map with
the new airplane locations.

4. Putting it All Together
Now that we have the functions to fetch the latest air traffic data for the selected country and
update the map with the new airplane locations, we can put it all together to build the flight
tracking application.

The code I use for the flight tracking application is below:

'''FLIGHT TRACKING WITH PYTHON AND OPEN-AIR TRAFFIC DATA'''

#IMPORT LIBRARY

import requests

import json

import pandas as pd

from bokeh.plotting import figure

from bokeh.models import HoverTool,LabelSet,ColumnDataSource

from bokeh.tile_providers import get_provider, STAMEN_TERRAIN

import numpy as np

from bokeh.server.server import Server

from bokeh.application import Application

from bokeh.application.handlers.function

import FunctionHandler

#FUNCTION TO CONVERT GCS WGS84 TO WEB MERCATOR

#DATAFRAME

def wgs84_to_web_mercator(df, lon="long", lat="lat"):

k = 6378137

df["x"] = df[lon] * (k * np.pi/180.0)

df["y"] = np.log(np.tan((90 + df[lat]) * np.pi/360.0)) * k

return df

#POINT

def wgs84_web_mercator_point(lon,lat):

k = 6378137

x= lon * (k * np.pi/180.0)

y= np.log(np.tan((90 + lat) * np.pi/360.0)) * k

return x,y

#AREA EXTENT COORDINATE WGS84

lon_min,lat_min = 14.12, 49.00

lon_max,lat_max = 24.15, 54.83

#COORDINATE CONVERSION

xy_min=wgs84_web_mercator_point(lon_min,lat_min)

xy_max=wgs84_web_mercator_point(lon_max,lat_max)

#COORDINATE RANGE IN WEB MERCATOR

x_range,y_range=([xy_min[0],xy_max[0]], [xy_min[1],xy_max[1]])

#REST API QUERY

user_name='Bilal_Rafique'

password='******'

url_data='https://'+user_name+':'+password+'@openskynetwork.org/api/states/all?'+'lamin='+str(lat_min)+'&lomin='+str(lon_min)+'&lamax='+str(lat
_max)+'&lomax='+str(lon_max)

#FLIGHT TRACKING FUNCTION

def flight_tracking(doc):

#init bokeh column data source

flight_source = ColumnDataSource({'icao24':[],'callsign':[],'origin_country':[],'time_position':[],'last_contact':[],'long':[],'lat':[],'baro_altitude':[],'on_ground':[],'velocity':[],'true_track':[],'vertical_rate':[],'sensors':[],'geo_altitude':[],'squawk':[],'spi':[],'position_source':[],'x':[],'y':[],'rot_angle':[],'url':[]})

#UPDATING FLIGHT DATA

def update():

response=requests.get(url_data).json()

#CONVERT TO PANDAS DATA FRAME

col_name=['icao24','callsign','origin_country','time_position' 'last_contact','long','lat','baro_altitude','on_ground','velocity',
'true_track','vertical_rate','sensors','geo_altitude','squawk','spi','position_source']

flight_df=pd.DataFrame(response['states'])

flight_df=flight_df.loc[:,0:16]

flight_df.columns=col_name

wgs84_to_web_mercator(flight_df)

flight_df=flight_df.fillna('No Data')

flight_df['rot_angle']=flight_df['true_track']*-1

icon_url='https:...' #icon url

flight_df['url']=icon_url

#CONVERT TO BOKEH DATASOURCE AND STREAMING

n_roll=len(flight_df.index)

flight_source.stream(flight_df.to_dict(orient='list'),n_roll)

#CALLBACK UPATE IN AN INTERVAL

doc.add_periodic_callback(update, 5000) #5000 ms/10000 ms for registered user.

#PLOT AIRCRAFT POSITION

p=figure(x_range=x_range,y_range=y_range,x_axis_type='mercator',y_axis_type='mercator',sizing_mode='scale_width',plot_height=300)

tile_prov=get_provider(STAMEN_TERRAIN)

p.add_tile(tile_prov,level='image')

p.image_url(url='url', x='x',y='y',source=flight_source,anchor='center',angle_units='deg',angle='rot_angle',h_units='scree
n',w_units='screen',w=40,h=40)

p.circle('x','y',source=flight_source,fill_color='red',hover_color='yellow',size=10,fill_alpha=0
.8,line_width=0)

#ADD HOVER TOOL AND LABEL

my_hover=HoverTool()

my_hover.tooltips=[('Call sign','@callsign'),('Origin
Country','@origin_country'),('velocity(m/s)','@velocity'),('Altitude(m)','@baro_altitude')]

labels = LabelSet(x='x', y='y', text='callsign', level='glyph',
x_offset=5, y_offset=5, source=flight_source,
render_mode='canvas',background_fill_color='white',text_font_size="8pt")

p.add_tools(my_hover)

p.add_layout(labels)

doc.title='REAL TIME FLIGHT TRACKING'

doc.add_root(p)

#SERVER CODE

apps = {'/': Application(FunctionHandler(flight_tracking))}

server = Server(apps, port=8084) #define an unused port

server.start()
## Conclusion

In this project, processing air traffic data in Python can be a challenging but rewarding task. With the right tools and techniques, we can extract valuable insights from large volumes of data, which can inform decisions related to air traffic control, airport management, and airline operations.

By using popular Python libraries like pandas, NumPy, and matplotlib, we can efficiently manipulate, analyze, and visualize air traffic data. Additionally, incorporating machine learning and deep learning techniques can help us predict future trends and detect anomalies in air traffic patterns.

Finally, it is important to note that air traffic data can be sensitive and requires careful handling to protect privacy and security. Always follow best practices for data management and ensure that you are following relevant regulations and laws.

Overall, processing air traffic data in Python can be a challenging yet rewarding endeavor and can help us uncover valuable insights to improve air travel and safety.