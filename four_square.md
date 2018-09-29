
<a href="https://cognitiveclass.ai"><img src = "https://ibm.box.com/shared/static/9gegpsmnsoo25ikkbl4qzlvlyjbgxs5x.png" width = 400> </a>

<h1 align=center><font size = 5>Learning FourSquare API with Python</font></h1>

   

## Introduction

In this lab, you will learn in details how to make calls to the Foursquare API for different purposes. You will learn how to construct a URL to send a request to the API to search for a specific type of venues, to explore a particular venue, to explore a Foursquare user, to explore a geographical location, and to get trending venues around a location. Also, you will learn how to use the visualization library, Folium, to visualize the results.

## Table of Contents

<div class="alert alert-block alert-info" style="margin-top: 20px">

<font size = 3>
1. <a href="#item1">Foursquare API Search Function</a>    
2. <a href="#item2">Explore a Given Venue</a>   
3. <a href="#item3">Explore a User</a>  
4. <a href="#item4">Foursquare API Explore Function</a>  
5. <a href="#item5">Get Trending Venues</a>  
</font>
</div>

### Import necessary Libraries


```python
!conda install -c conda-forge geopy --yes 
from geopy.geocoders import Nominatim # module to convert an address into latitude and longitude values
import requests # library to handle requests
import pandas as pd # library for data analsysis
import numpy as np # library to handle data in a vectorized manner
import random # library for random number generation

# libraries for displaying images
from IPython.display import Image 
from IPython.core.display import HTML 
    
# tranforming json file into a pandas dataframe library
from pandas.io.json import json_normalize

!conda install -c conda-forge folium=0.5.0 --yes
import folium # plotting library

print('Folium installed')
print('Libraries imported.')
```

    Solving environment: done
    
    ## Package Plan ##
    
      environment location: /home/jupyterlab/conda
    
      added / updated specs: 
        - geopy
    
    
    The following packages will be downloaded:
    
        package                    |            build
        ---------------------------|-----------------
        geopy-1.17.0               |             py_0          49 KB  conda-forge
        geographiclib-1.49         |             py_0          32 KB  conda-forge
        certifi-2018.8.24          |        py36_1001         139 KB  conda-forge
        ------------------------------------------------------------
                                               Total:         220 KB
    
    The following NEW packages will be INSTALLED:
    
        geographiclib: 1.49-py_0        conda-forge
        geopy:         1.17.0-py_0      conda-forge
    
    The following packages will be UPDATED:
    
        certifi:       2018.8.24-py36_1 conda-forge --> 2018.8.24-py36_1001 conda-forge
    
    
    Downloading and Extracting Packages
    geopy-1.17.0         | 49 KB     | ##################################### | 100% 
    geographiclib-1.49   | 32 KB     | ##################################### | 100% 
    certifi-2018.8.24    | 139 KB    | ##################################### | 100% 
    Preparing transaction: done
    Verifying transaction: done
    Executing transaction: done
    Solving environment: done
    
    ## Package Plan ##
    
      environment location: /home/jupyterlab/conda
    
      added / updated specs: 
        - folium=0.5.0
    
    
    The following packages will be downloaded:
    
        package                    |            build
        ---------------------------|-----------------
        folium-0.5.0               |             py_0          45 KB  conda-forge
        altair-2.2.2               |           py36_1         461 KB  conda-forge
        branca-0.3.0               |             py_0          24 KB  conda-forge
        vincent-0.4.4              |             py_1          28 KB  conda-forge
        ------------------------------------------------------------
                                               Total:         558 KB
    
    The following NEW packages will be INSTALLED:
    
        altair:  2.2.2-py36_1 conda-forge
        branca:  0.3.0-py_0   conda-forge
        folium:  0.5.0-py_0   conda-forge
        vincent: 0.4.4-py_1   conda-forge
    
    
    Downloading and Extracting Packages
    folium-0.5.0         | 45 KB     | ##################################### | 100% 
    altair-2.2.2         | 461 KB    | ##################################### | 100% 
    branca-0.3.0         | 24 KB     | ##################################### | 100% 
    vincent-0.4.4        | 28 KB     | ##################################### | 100% 
    Preparing transaction: done
    Verifying transaction: done
    Executing transaction: done
    Folium installed
    Libraries imported.


### Define Foursquare Credentials and Version

##### Make sure that you have created a Foursquare developer account and have your credentials handy


```python
CLIENT_ID = '0EJ3W3T31AZSLAINH5VBFWLOZGRC2UWPH20XEVRTT2PRWAOK' # your Foursquare ID
CLIENT_SECRET = 'AL3MH0MTRRR0JWMJMSRZOBMSROZXKASPOR4ARNHNZJDLNXYD' # your Foursquare Secret
VERSION = '20180604'
LIMIT = 30
print('Your credentails:')
print('CLIENT_ID: ' + CLIENT_ID)
print('CLIENT_SECRET:' + CLIENT_SECRET)
```

    Your credentails:
    CLIENT_ID: 0EJ3W3T31AZSLAINH5VBFWLOZGRC2UWPH20XEVRTT2PRWAOK
    CLIENT_SECRET:AL3MH0MTRRR0JWMJMSRZOBMSROZXKASPOR4ARNHNZJDLNXYD


  

#### Let's again assume that you are staying at the Conrad hotel. So let's start by converting the Contrad Hotel's address to its latitude and longitude coordinates.


```python
import sys
import warnings

if not sys.warnoptions:
    warnings.simplefilter("ignore")

address = '102 North End Ave, New York, NY'

geolocator = Nominatim()
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print(latitude, longitude)
```

    40.7149555 -74.0153365


   

<a id="item1"></a>

## 1. Search for a specific venue category
> `https://api.foursquare.com/v2/venues/`**search**`?client_id=`**CLIENT_ID**`&client_secret=`**CLIENT_SECRET**`&ll=`**LATITUDE**`,`**LONGITUDE**`&v=`**VERSION**`&query=`**QUERY**`&radius=`**RADIUS**`&limit=`**LIMIT**

#### Now, let's assume that it is lunch time, and you are craving Italian food. So, let's define a query to search for Italian food that is within 500 metres from the Conrad Hotel. 


```python
search_query = 'Italian'
radius = 500
print(search_query + ' .... OK!')
```

    Italian .... OK!


#### Define the corresponding URL


```python
url = 'https://api.foursquare.com/v2/venues/search?client_id={}&client_secret={}&ll={},{}&v={}&query={}&radius={}&limit={}'.format(CLIENT_ID, CLIENT_SECRET, latitude, longitude, VERSION, search_query, radius, LIMIT)
url
```




    'https://api.foursquare.com/v2/venues/search?client_id=0EJ3W3T31AZSLAINH5VBFWLOZGRC2UWPH20XEVRTT2PRWAOK&client_secret=AL3MH0MTRRR0JWMJMSRZOBMSROZXKASPOR4ARNHNZJDLNXYD&ll=40.7149555,-74.0153365&v=20180604&query=Italian&radius=500&limit=30'



#### Send the GET Request and examine the results


```python
results = requests.get(url).json()
results
```




    {'meta': {'code': 200, 'requestId': '5bace6324c1f6774f8d640b7'},
     'response': {'venues': [{'id': '4fa862b3e4b0ebff2f749f06',
        'name': "Harry's Italian Pizza Bar",
        'location': {'address': '225 Murray St',
         'lat': 40.71521779064671,
         'lng': -74.01473940209351,
         'labeledLatLngs': [{'label': 'display',
           'lat': 40.71521779064671,
           'lng': -74.01473940209351}],
         'distance': 58,
         'postalCode': '10282',
         'cc': 'US',
         'city': 'New York',
         'state': 'NY',
         'country': 'United States',
         'formattedAddress': ['225 Murray St',
          'New York, NY 10282',
          'United States']},
        'categories': [{'id': '4bf58dd8d48988d1ca941735',
          'name': 'Pizza Place',
          'pluralName': 'Pizza Places',
          'shortName': 'Pizza',
          'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
           'suffix': '.png'},
          'primary': True}],
        'delivery': {'id': '294544',
         'url': 'https://www.seamless.com/menu/harrys-italian-pizza-bar-225-murray-st-new-york/294544?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=294544',
         'provider': {'name': 'seamless',
          'icon': {'prefix': 'https://igx.4sqi.net/img/general/cap/',
           'sizes': [40, 50],
           'name': '/delivery_provider_seamless_20180129.png'}}},
        'referralId': 'v-1538057778',
        'hasPerk': False},
       {'id': '4f3232e219836c91c7bfde94',
        'name': 'Conca Cucina Italian Restaurant',
        'location': {'address': '63 W Broadway',
         'lat': 40.71446,
         'lng': -74.010086,
         'labeledLatLngs': [{'label': 'display',
           'lat': 40.71446,
           'lng': -74.010086}],
         'distance': 446,
         'postalCode': '10007',
         'cc': 'US',
         'city': 'New York',
         'state': 'NY',
         'country': 'United States',
         'formattedAddress': ['63 W Broadway',
          'New York, NY 10007',
          'United States']},
        'categories': [{'id': '4d4b7105d754a06374d81259',
          'name': 'Food',
          'pluralName': 'Food',
          'shortName': 'Food',
          'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/default_',
           'suffix': '.png'},
          'primary': True}],
        'referralId': 'v-1538057778',
        'hasPerk': False},
       {'id': '3fd66200f964a520f4e41ee3',
        'name': 'Ecco',
        'location': {'address': '124 Chambers St',
         'crossStreet': 'btwn Church St & W Broadway',
         'lat': 40.71533713859952,
         'lng': -74.00884766217825,
         'labeledLatLngs': [{'label': 'display',
           'lat': 40.71533713859952,
           'lng': -74.00884766217825}],
         'distance': 549,
         'postalCode': '10007',
         'cc': 'US',
         'city': 'New York',
         'state': 'NY',
         'country': 'United States',
         'formattedAddress': ['124 Chambers St (btwn Church St & W Broadway)',
          'New York, NY 10007',
          'United States']},
        'categories': [{'id': '4bf58dd8d48988d110941735',
          'name': 'Italian Restaurant',
          'pluralName': 'Italian Restaurants',
          'shortName': 'Italian',
          'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/italian_',
           'suffix': '.png'},
          'primary': True}],
        'referralId': 'v-1538057778',
        'hasPerk': False}]}}



#### Get relevant part of JSON and transform it into a *pandas* dataframe


```python
# assign relevant part of JSON to venues
venues = results['response']['venues']

# tranform venues into a dataframe
dataframe = json_normalize(venues)
dataframe.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>categories</th>
      <th>delivery.id</th>
      <th>delivery.provider.icon.name</th>
      <th>delivery.provider.icon.prefix</th>
      <th>delivery.provider.icon.sizes</th>
      <th>delivery.provider.name</th>
      <th>delivery.url</th>
      <th>hasPerk</th>
      <th>id</th>
      <th>location.address</th>
      <th>...</th>
      <th>location.crossStreet</th>
      <th>location.distance</th>
      <th>location.formattedAddress</th>
      <th>location.labeledLatLngs</th>
      <th>location.lat</th>
      <th>location.lng</th>
      <th>location.postalCode</th>
      <th>location.state</th>
      <th>name</th>
      <th>referralId</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[{'id': '4bf58dd8d48988d1ca941735', 'name': 'P...</td>
      <td>294544</td>
      <td>/delivery_provider_seamless_20180129.png</td>
      <td>https://igx.4sqi.net/img/general/cap/</td>
      <td>[40, 50]</td>
      <td>seamless</td>
      <td>https://www.seamless.com/menu/harrys-italian-p...</td>
      <td>False</td>
      <td>4fa862b3e4b0ebff2f749f06</td>
      <td>225 Murray St</td>
      <td>...</td>
      <td>NaN</td>
      <td>58</td>
      <td>[225 Murray St, New York, NY 10282, United Sta...</td>
      <td>[{'label': 'display', 'lat': 40.71521779064671...</td>
      <td>40.715218</td>
      <td>-74.014739</td>
      <td>10282</td>
      <td>NY</td>
      <td>Harry's Italian Pizza Bar</td>
      <td>v-1538057778</td>
    </tr>
    <tr>
      <th>1</th>
      <td>[{'id': '4d4b7105d754a06374d81259', 'name': 'F...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>False</td>
      <td>4f3232e219836c91c7bfde94</td>
      <td>63 W Broadway</td>
      <td>...</td>
      <td>NaN</td>
      <td>446</td>
      <td>[63 W Broadway, New York, NY 10007, United Sta...</td>
      <td>[{'label': 'display', 'lat': 40.71446, 'lng': ...</td>
      <td>40.714460</td>
      <td>-74.010086</td>
      <td>10007</td>
      <td>NY</td>
      <td>Conca Cucina Italian Restaurant</td>
      <td>v-1538057778</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[{'id': '4bf58dd8d48988d110941735', 'name': 'I...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>False</td>
      <td>3fd66200f964a520f4e41ee3</td>
      <td>124 Chambers St</td>
      <td>...</td>
      <td>btwn Church St &amp; W Broadway</td>
      <td>549</td>
      <td>[124 Chambers St (btwn Church St &amp; W Broadway)...</td>
      <td>[{'label': 'display', 'lat': 40.71533713859952...</td>
      <td>40.715337</td>
      <td>-74.008848</td>
      <td>10007</td>
      <td>NY</td>
      <td>Ecco</td>
      <td>v-1538057778</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 23 columns</p>
</div>



#### Define information of interest and filter dataframe


```python
# keep only columns that include venue name, and anything that is associated with location
filtered_columns = ['name', 'categories'] + [col for col in dataframe.columns if col.startswith('location.')] + ['id']
dataframe_filtered = dataframe.loc[:, filtered_columns]
```


```python
dataframe_filtered.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>location.address</th>
      <th>location.cc</th>
      <th>location.city</th>
      <th>location.country</th>
      <th>location.crossStreet</th>
      <th>location.distance</th>
      <th>location.formattedAddress</th>
      <th>location.labeledLatLngs</th>
      <th>location.lat</th>
      <th>location.lng</th>
      <th>location.postalCode</th>
      <th>location.state</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Harry's Italian Pizza Bar</td>
      <td>[{'id': '4bf58dd8d48988d1ca941735', 'name': 'P...</td>
      <td>225 Murray St</td>
      <td>US</td>
      <td>New York</td>
      <td>United States</td>
      <td>NaN</td>
      <td>58</td>
      <td>[225 Murray St, New York, NY 10282, United Sta...</td>
      <td>[{'label': 'display', 'lat': 40.71521779064671...</td>
      <td>40.715218</td>
      <td>-74.014739</td>
      <td>10282</td>
      <td>NY</td>
      <td>4fa862b3e4b0ebff2f749f06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Conca Cucina Italian Restaurant</td>
      <td>[{'id': '4d4b7105d754a06374d81259', 'name': 'F...</td>
      <td>63 W Broadway</td>
      <td>US</td>
      <td>New York</td>
      <td>United States</td>
      <td>NaN</td>
      <td>446</td>
      <td>[63 W Broadway, New York, NY 10007, United Sta...</td>
      <td>[{'label': 'display', 'lat': 40.71446, 'lng': ...</td>
      <td>40.714460</td>
      <td>-74.010086</td>
      <td>10007</td>
      <td>NY</td>
      <td>4f3232e219836c91c7bfde94</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ecco</td>
      <td>[{'id': '4bf58dd8d48988d110941735', 'name': 'I...</td>
      <td>124 Chambers St</td>
      <td>US</td>
      <td>New York</td>
      <td>United States</td>
      <td>btwn Church St &amp; W Broadway</td>
      <td>549</td>
      <td>[124 Chambers St (btwn Church St &amp; W Broadway)...</td>
      <td>[{'label': 'display', 'lat': 40.71533713859952...</td>
      <td>40.715337</td>
      <td>-74.008848</td>
      <td>10007</td>
      <td>NY</td>
      <td>3fd66200f964a520f4e41ee3</td>
    </tr>
  </tbody>
</table>
</div>




```python
# function that extracts the category of the venue
def get_category_type(row):
    try:
        categories_list = row['categories']
    except:
        categories_list = row['venue.categories']
        
    if len(categories_list) == 0:
        return None
    else:
        return categories_list[0]['name']

# filter the category for each row
dataframe_filtered['categories'] = dataframe_filtered.apply(get_category_type, axis=1)

# clean column names by keeping only last term
dataframe_filtered.columns = [column.split('.')[-1] for column in dataframe_filtered.columns]

dataframe_filtered
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>address</th>
      <th>cc</th>
      <th>city</th>
      <th>country</th>
      <th>crossStreet</th>
      <th>distance</th>
      <th>formattedAddress</th>
      <th>labeledLatLngs</th>
      <th>lat</th>
      <th>lng</th>
      <th>postalCode</th>
      <th>state</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Harry's Italian Pizza Bar</td>
      <td>Pizza Place</td>
      <td>225 Murray St</td>
      <td>US</td>
      <td>New York</td>
      <td>United States</td>
      <td>NaN</td>
      <td>58</td>
      <td>[225 Murray St, New York, NY 10282, United Sta...</td>
      <td>[{'label': 'display', 'lat': 40.71521779064671...</td>
      <td>40.715218</td>
      <td>-74.014739</td>
      <td>10282</td>
      <td>NY</td>
      <td>4fa862b3e4b0ebff2f749f06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Conca Cucina Italian Restaurant</td>
      <td>Food</td>
      <td>63 W Broadway</td>
      <td>US</td>
      <td>New York</td>
      <td>United States</td>
      <td>NaN</td>
      <td>446</td>
      <td>[63 W Broadway, New York, NY 10007, United Sta...</td>
      <td>[{'label': 'display', 'lat': 40.71446, 'lng': ...</td>
      <td>40.714460</td>
      <td>-74.010086</td>
      <td>10007</td>
      <td>NY</td>
      <td>4f3232e219836c91c7bfde94</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ecco</td>
      <td>Italian Restaurant</td>
      <td>124 Chambers St</td>
      <td>US</td>
      <td>New York</td>
      <td>United States</td>
      <td>btwn Church St &amp; W Broadway</td>
      <td>549</td>
      <td>[124 Chambers St (btwn Church St &amp; W Broadway)...</td>
      <td>[{'label': 'display', 'lat': 40.71533713859952...</td>
      <td>40.715337</td>
      <td>-74.008848</td>
      <td>10007</td>
      <td>NY</td>
      <td>3fd66200f964a520f4e41ee3</td>
    </tr>
  </tbody>
</table>
</div>



#### Let's visualize the Italian restaurants that are nearby


```python
dataframe_filtered.name
```




    0          Harry's Italian Pizza Bar
    1    Conca Cucina Italian Restaurant
    2                               Ecco
    Name: name, dtype: object




```python
venues_map = folium.Map(location=[latitude, longitude], zoom_start=13) # generate map centred around the Conrad Hotel

# add a red circle marker to represent the Conrad Hotel
folium.features.CircleMarker(
    [latitude, longitude],
    radius=10,
    color='red',
    popup='Conrad Hotel',
    fill = True,
    fill_color = 'red',
    fill_opacity = 0.6
).add_to(venues_map)

# add the Italian restaurants as blue circle markers
for lat, lng, label in zip(dataframe_filtered.lat, dataframe_filtered.lng, dataframe_filtered.categories):
    folium.features.CircleMarker(
        [lat, lng],
        radius=5,
        color='blue',
        popup=label,
        fill = True,
        fill_color='blue',
        fill_opacity=0.6
    ).add_to(venues_map)

# display map
venues_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfZTlmOTRkNGRhOGFlNDAyOThjNTQyODdlOGJmYzU2YWYgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2U5Zjk0ZDRkYThhZTQwMjk4YzU0Mjg3ZThiZmM1NmFmIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF9lOWY5NGQ0ZGE4YWU0MDI5OGM1NDI4N2U4YmZjNTZhZiA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF9lOWY5NGQ0ZGE4YWU0MDI5OGM1NDI4N2U4YmZjNTZhZicsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNDAuNzE0OTU1NSwtNzQuMDE1MzM2NV0sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAxMywKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIG1heEJvdW5kczogYm91bmRzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiBbXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHdvcmxkQ29weUp1bXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NwogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfMWFhMTUwYWM1M2Q2NDE5ZmEyNmIzZWQ3Y2U4ZWFhZGIgPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICdodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZycsCiAgICAgICAgICAgICAgICB7CiAgImF0dHJpYnV0aW9uIjogbnVsbCwKICAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsCiAgIm1heFpvb20iOiAxOCwKICAibWluWm9vbSI6IDEsCiAgIm5vV3JhcCI6IGZhbHNlLAogICJzdWJkb21haW5zIjogImFiYyIKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfZTlmOTRkNGRhOGFlNDAyOThjNTQyODdlOGJmYzU2YWYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2I5OGZhZmFlNmRjMDQ0NmY4OWQ5MzJjZDllZmFlMDgyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDAuNzE0OTU1NSwtNzQuMDE1MzM2NV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJyZWQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICJyZWQiLAogICJmaWxsT3BhY2l0eSI6IDAuNiwKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2U5Zjk0ZDRkYThhZTQwMjk4YzU0Mjg3ZThiZmM1NmFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2Q5ZDMwM2YyM2IxODRlYmViY2EyNWQzNWQ1ZjZlODllID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzcyZDdlNDc3M2Q2NDQ3NzliNzkyOGQ4MjkxNjczMzZjID0gJCgnPGRpdiBpZD0iaHRtbF83MmQ3ZTQ3NzNkNjQ0Nzc5Yjc5MjhkODI5MTY3MzM2YyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q29ucmFkIEhvdGVsPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kOWQzMDNmMjNiMTg0ZWJlYmNhMjVkMzVkNWY2ZTg5ZS5zZXRDb250ZW50KGh0bWxfNzJkN2U0NzczZDY0NDc3OWI3OTI4ZDgyOTE2NzMzNmMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYjk4ZmFmYWU2ZGMwNDQ2Zjg5ZDkzMmNkOWVmYWUwODIuYmluZFBvcHVwKHBvcHVwX2Q5ZDMwM2YyM2IxODRlYmViY2EyNWQzNWQ1ZjZlODllKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNjNmE1YTJhMWQxYjQ5NjZiMTQxY2EzMDI0ZjM1MDRjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDAuNzE1MjE3NzkwNjQ2NzEsLTc0LjAxNDczOTQwMjA5MzUxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICJibHVlIiwKICAiZmlsbE9wYWNpdHkiOiAwLjYsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2U5Zjk0ZDRkYThhZTQwMjk4YzU0Mjg3ZThiZmM1NmFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzAyMGYyZThiMjJiOTQ0YWJiMjFjY2FmNWJiM2JhNTEzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzI2OTdlZjgzOWQ1ZjQwN2U5YTRhMTM5NTcyZmEzNzViID0gJCgnPGRpdiBpZD0iaHRtbF8yNjk3ZWY4MzlkNWY0MDdlOWE0YTEzOTU3MmZhMzc1YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGl6emEgUGxhY2U8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzAyMGYyZThiMjJiOTQ0YWJiMjFjY2FmNWJiM2JhNTEzLnNldENvbnRlbnQoaHRtbF8yNjk3ZWY4MzlkNWY0MDdlOWE0YTEzOTU3MmZhMzc1Yik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8zYzZhNWEyYTFkMWI0OTY2YjE0MWNhMzAyNGYzNTA0Yy5iaW5kUG9wdXAocG9wdXBfMDIwZjJlOGIyMmI5NDRhYmIyMWNjYWY1YmIzYmE1MTMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTQwMTdjNTljNjVlNGU2YWIyYzc2MmVhOGQxNTVlN2UgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0MC43MTQ0NiwtNzQuMDEwMDg2XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICJibHVlIiwKICAiZmlsbE9wYWNpdHkiOiAwLjYsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2U5Zjk0ZDRkYThhZTQwMjk4YzU0Mjg3ZThiZmM1NmFmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzQ5ZDdlZGQ2ZWIxZTQyYmNiYTk2NGVlYzM1ZGY3YjhhID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzk3NGIwMmMwNDU3MTQ2NDNiZTUxMTIxN2U3ZGU4NjVkID0gJCgnPGRpdiBpZD0iaHRtbF85NzRiMDJjMDQ1NzE0NjQzYmU1MTEyMTdlN2RlODY1ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Rm9vZDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNDlkN2VkZDZlYjFlNDJiY2JhOTY0ZWVjMzVkZjdiOGEuc2V0Q29udGVudChodG1sXzk3NGIwMmMwNDU3MTQ2NDNiZTUxMTIxN2U3ZGU4NjVkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzE0MDE3YzU5YzY1ZTRlNmFiMmM3NjJlYThkMTU1ZTdlLmJpbmRQb3B1cChwb3B1cF80OWQ3ZWRkNmViMWU0MmJjYmE5NjRlZWMzNWRmN2I4YSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yZmE5MDVlZWQzYTU0Mzg0YTQzZGM5YWU0YTJjZWM2YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQwLjcxNTMzNzEzODU5OTUyLC03NC4wMDg4NDc2NjIxNzgyNV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiYmx1ZSIsCiAgImZpbGxPcGFjaXR5IjogMC42LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9lOWY5NGQ0ZGE4YWU0MDI5OGM1NDI4N2U4YmZjNTZhZik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iZDdlZjVkNmM3MmQ0NTMxYjA1MzQ5NGQwMDc5NDIwNiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yNDY2ZDhhODIyOTk0OGYyODVjMmM1NjY0OTVmMmU5MSA9ICQoJzxkaXYgaWQ9Imh0bWxfMjQ2NmQ4YTgyMjk5NDhmMjg1YzJjNTY2NDk1ZjJlOTEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkl0YWxpYW4gUmVzdGF1cmFudDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYmQ3ZWY1ZDZjNzJkNDUzMWIwNTM0OTRkMDA3OTQyMDYuc2V0Q29udGVudChodG1sXzI0NjZkOGE4MjI5OTQ4ZjI4NWMyYzU2NjQ5NWYyZTkxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzJmYTkwNWVlZDNhNTQzODRhNDNkYzlhZTRhMmNlYzZjLmJpbmRQb3B1cChwb3B1cF9iZDdlZjVkNmM3MmQ0NTMxYjA1MzQ5NGQwMDc5NDIwNik7CgogICAgICAgICAgICAKICAgICAgICAKPC9zY3JpcHQ+" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



   

<a id="item2"></a>

## 2. Explore a Given Venue
> `https://api.foursquare.com/v2/venues/`**VENUE_ID**`?client_id=`**CLIENT_ID**`&client_secret=`**CLIENT_SECRET**`&v=`**VERSION**

### A. Let's explore the closest Italian restaurant -- _Harry's Italian Pizza Bar_


```python
venue_id = '4fa862b3e4b0ebff2f749f06' # ID of Harry's Italian Pizza Bar
url = 'https://api.foursquare.com/v2/venues/{}?client_id={}&client_secret={}&v={}'.format(venue_id, CLIENT_ID, CLIENT_SECRET, VERSION)
url
```




    'https://api.foursquare.com/v2/venues/4fa862b3e4b0ebff2f749f06?client_id=0EJ3W3T31AZSLAINH5VBFWLOZGRC2UWPH20XEVRTT2PRWAOK&client_secret=AL3MH0MTRRR0JWMJMSRZOBMSROZXKASPOR4ARNHNZJDLNXYD&v=20180604'



#### Send GET request for result


```python
result = requests.get(url).json()
print(result['response']['venue'].keys())
result['response']['venue']
```

    dict_keys(['id', 'name', 'contact', 'location', 'canonicalUrl', 'categories', 'verified', 'stats', 'url', 'price', 'hasMenu', 'likes', 'dislike', 'ok', 'rating', 'ratingColor', 'ratingSignals', 'delivery', 'menu', 'allowMenuUrlEdit', 'beenHere', 'specials', 'photos', 'reasons', 'hereNow', 'createdAt', 'tips', 'shortUrl', 'timeZone', 'listed', 'hours', 'popular', 'pageUpdates', 'inbox', 'attributes', 'bestPhoto', 'colors'])





    {'id': '4fa862b3e4b0ebff2f749f06',
     'name': "Harry's Italian Pizza Bar",
     'contact': {'phone': '2126081007', 'formattedPhone': '(212) 608-1007'},
     'location': {'address': '225 Murray St',
      'lat': 40.71521779064671,
      'lng': -74.01473940209351,
      'labeledLatLngs': [{'label': 'display',
        'lat': 40.71521779064671,
        'lng': -74.01473940209351}],
      'postalCode': '10282',
      'cc': 'US',
      'city': 'New York',
      'state': 'NY',
      'country': 'United States',
      'formattedAddress': ['225 Murray St',
       'New York, NY 10282',
       'United States']},
     'canonicalUrl': 'https://foursquare.com/v/harrys-italian-pizza-bar/4fa862b3e4b0ebff2f749f06',
     'categories': [{'id': '4bf58dd8d48988d1ca941735',
       'name': 'Pizza Place',
       'pluralName': 'Pizza Places',
       'shortName': 'Pizza',
       'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/pizza_',
        'suffix': '.png'},
       'primary': True},
      {'id': '4bf58dd8d48988d110941735',
       'name': 'Italian Restaurant',
       'pluralName': 'Italian Restaurants',
       'shortName': 'Italian',
       'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/italian_',
        'suffix': '.png'}}],
     'verified': False,
     'stats': {'tipCount': 55},
     'url': 'http://harrysitalian.com',
     'price': {'tier': 2, 'message': 'Moderate', 'currency': '$'},
     'hasMenu': True,
     'likes': {'count': 116,
      'groups': [{'type': 'others', 'count': 116, 'items': []}],
      'summary': '116 Likes'},
     'dislike': False,
     'ok': False,
     'rating': 7.3,
     'ratingColor': 'C5DE35',
     'ratingSignals': 205,
     'delivery': {'id': '294544',
      'url': 'https://www.seamless.com/menu/harrys-italian-pizza-bar-225-murray-st-new-york/294544?affiliate=1131&utm_source=foursquare-affiliate-network&utm_medium=affiliate&utm_campaign=1131&utm_content=294544',
      'provider': {'name': 'seamless',
       'icon': {'prefix': 'https://igx.4sqi.net/img/general/cap/',
        'sizes': [40, 50],
        'name': '/delivery_provider_seamless_20180129.png'}}},
     'menu': {'type': 'Menu',
      'label': 'Menu',
      'anchor': 'View Menu',
      'url': 'https://foursquare.com/v/harrys-italian-pizza-bar/4fa862b3e4b0ebff2f749f06/menu',
      'mobileUrl': 'https://foursquare.com/v/4fa862b3e4b0ebff2f749f06/device_menu'},
     'allowMenuUrlEdit': True,
     'beenHere': {'count': 0,
      'unconfirmedCount': 0,
      'marked': False,
      'lastCheckinExpiredAt': 0},
     'specials': {'count': 0, 'items': []},
     'photos': {'count': 145,
      'groups': [{'type': 'checkin',
        'name': "Friends' check-in photos",
        'count': 0,
        'items': []},
       {'type': 'venue',
        'name': 'Venue photos',
        'count': 145,
        'items': [{'id': '4fad980de4b091b4626c3633',
          'createdAt': 1336776717,
          'source': {'name': 'Foursquare for Android',
           'url': 'https://foursquare.com/download/#/android'},
          'prefix': 'https://igx.4sqi.net/img/general/',
          'suffix': '/ya1iQFI7pLjuIJp1PGDKlrZS3OJdHCF7tpILMmjv_2w.jpg',
          'width': 480,
          'height': 640,
          'user': {'id': '13676709',
           'firstName': 'Leony',
           'lastName': 'Naciri',
           'gender': 'none',
           'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
            'suffix': '/T0ANFNGNMCHUDEUE.jpg'}},
          'visibility': 'public'}]}],
      'summary': '0 photos'},
     'reasons': {'count': 1,
      'items': [{'summary': 'Lots of people like this place',
        'type': 'general',
        'reasonName': 'rawLikesReason'}]},
     'hereNow': {'count': 0, 'summary': 'Nobody here', 'groups': []},
     'createdAt': 1336435379,
     'tips': {'count': 55,
      'groups': [{'type': 'others',
        'name': 'All tips',
        'count': 55,
        'items': [{'id': '53d27909498e0523841340b6',
          'createdAt': 1406302473,
          'text': "Harry's Italian Pizza bar is known for it's amazing pizza, but did you know that the brunches here are amazing too? Try the Nutella French toast and we know you'll be sold.",
          'type': 'user',
          'canonicalUrl': 'https://foursquare.com/item/53d27909498e0523841340b6',
          'lang': 'en',
          'likes': {'count': 4,
           'groups': [{'type': 'others',
             'count': 4,
             'items': [{'id': '369426',
               'firstName': 'P.',
               'lastName': 'M.',
               'gender': 'male',
               'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
                'suffix': '/JPQYUWJKUT0H2OO4.jpg'}},
              {'id': '87587879',
               'firstName': 'Diane',
               'lastName': 'Danneels',
               'gender': 'female',
               'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
                'suffix': '/87587879-ESLRSZLQ2CBE2P4W.jpg'}},
              {'id': '87591341',
               'firstName': 'Tim',
               'lastName': 'Sheehan',
               'gender': 'male',
               'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
                'suffix': '/-Z4YK4VKE0JSVXIY1.jpg'}},
              {'id': '87473404',
               'firstName': 'TenantKing.com',
               'gender': 'none',
               'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
                'suffix': '/87473404-HI5DTBTK0HX401CA.png'},
               'type': 'page'}]}],
           'summary': '4 likes'},
          'logView': True,
          'agreeCount': 4,
          'disagreeCount': 0,
          'todo': {'count': 0},
          'user': {'id': '87473404',
           'firstName': 'TenantKing.com',
           'gender': 'none',
           'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
            'suffix': '/87473404-HI5DTBTK0HX401CA.png'},
           'type': 'page'}}]}]},
     'shortUrl': 'http://4sq.com/JNblHV',
     'timeZone': 'America/New_York',
     'listed': {'count': 50,
      'groups': [{'type': 'others',
        'name': 'Lists from other people',
        'count': 50,
        'items': [{'id': '4fa32fd0e4b04193744746b1',
          'name': 'Manhattan Haunts',
          'description': '',
          'type': 'others',
          'user': {'id': '24592223',
           'firstName': 'Becca',
           'lastName': 'McArthur',
           'gender': 'female',
           'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
            'suffix': '/24592223-RAW2UYM0GIB1U40K.jpg'}},
          'editable': False,
          'public': True,
          'collaborative': False,
          'url': '/becca_mcarthur/list/manhattan-haunts',
          'canonicalUrl': 'https://foursquare.com/becca_mcarthur/list/manhattan-haunts',
          'createdAt': 1336094672,
          'updatedAt': 1380845377,
          'photo': {'id': '4e8cc9461081e3b3544e12e5',
           'createdAt': 1317849414,
           'prefix': 'https://igx.4sqi.net/img/general/',
           'suffix': '/0NLVU2HC1JF4DXIMKWUFW3QBUT31DC11EFNYYHMJG3NDWAPS.jpg',
           'width': 492,
           'height': 330,
           'user': {'id': '742542',
            'firstName': 'Time Out New York',
            'gender': 'none',
            'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
             'suffix': '/XXHKCBSQHBORZNSR.jpg'},
            'type': 'page'},
           'visibility': 'public'},
          'followers': {'count': 22},
          'listItems': {'count': 187,
           'items': [{'id': 'v4fa862b3e4b0ebff2f749f06',
             'createdAt': 1342934485}]}},
         {'id': '4fae817be4b085f6b2a74d19',
          'name': 'USA NYC MAN FiDi',
          'description': 'Where to go for decent eats in the restaurant wasteland of Downtown NYC aka FiDi, along with Tribeca & Battery Park City.',
          'type': 'others',
          'user': {'id': '12113441',
           'firstName': 'Kino',
           'gender': 'male',
           'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
            'suffix': '/12113441-K5HTHFLU2MUCM0CM.jpg'}},
          'editable': False,
          'public': True,
          'collaborative': False,
          'url': '/kinosfault/list/usa-nyc-man-fidi',
          'canonicalUrl': 'https://foursquare.com/kinosfault/list/usa-nyc-man-fidi',
          'createdAt': 1336836475,
          'updatedAt': 1536019882,
          'photo': {'id': '55984992498e13ba75e353bb',
           'createdAt': 1436043666,
           'prefix': 'https://igx.4sqi.net/img/general/',
           'suffix': '/12113441_iOa6Uh-Xi8bhj2-gpzkkw8MKiAIs7RmOcz_RM7m8ink.jpg',
           'width': 540,
           'height': 960,
           'user': {'id': '12113441',
            'firstName': 'Kino',
            'gender': 'male',
            'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
             'suffix': '/12113441-K5HTHFLU2MUCM0CM.jpg'}},
           'visibility': 'public'},
          'followers': {'count': 20},
          'listItems': {'count': 272,
           'items': [{'id': 'v4fa862b3e4b0ebff2f749f06',
             'createdAt': 1373909433}]}},
         {'id': '5266c68a498e7c667807fe09',
          'name': 'Foodie Love in NY - 02',
          'description': '',
          'type': 'others',
          'user': {'id': '547977',
           'firstName': 'WiLL',
           'gender': 'male',
           'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
            'suffix': '/-Q5NYGDMFDMOITQRR.jpg'}},
          'editable': False,
          'public': True,
          'collaborative': False,
          'url': '/sweetiewill/list/foodie-love-in-ny--02',
          'canonicalUrl': 'https://foursquare.com/sweetiewill/list/foodie-love-in-ny--02',
          'createdAt': 1382467210,
          'updatedAt': 1391995585,
          'followers': {'count': 7},
          'listItems': {'count': 200,
           'items': [{'id': 'v4fa862b3e4b0ebff2f749f06',
             'createdAt': 1386809936}]}},
         {'id': '4fddeff0e4b0e078037ac0d3',
          'name': 'NYC Resturants',
          'description': '',
          'type': 'others',
          'user': {'id': '21563126',
           'firstName': 'Richard',
           'lastName': 'Revilla',
           'gender': 'male',
           'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
            'suffix': '/21563126_v05J1KPw_SVj6Ehq9g8B9jeAGjFUMsU5QGl-NZ8inUQ7pKQm5bKplW37EmR7jS2A7GYPBBAtl.jpg'}},
          'editable': False,
          'public': True,
          'collaborative': True,
          'url': '/rickr7/list/nyc-resturants',
          'canonicalUrl': 'https://foursquare.com/rickr7/list/nyc-resturants',
          'createdAt': 1339944944,
          'updatedAt': 1537546712,
          'photo': {'id': '5072dd13e4b09145cdf782d1',
           'createdAt': 1349704979,
           'prefix': 'https://igx.4sqi.net/img/general/',
           'suffix': '/208205_fGh2OuAZ9qJ4agbAA5wMVNOSIm9kNUlRtNwj1N-adqg.jpg',
           'width': 800,
           'height': 800,
           'user': {'id': '208205',
            'firstName': 'Thalia',
            'lastName': 'K',
            'gender': 'female',
            'photo': {'prefix': 'https://igx.4sqi.net/img/user/',
             'suffix': '/SNOOLCAW2AG04ZKD.jpg'}},
           'visibility': 'public'},
          'followers': {'count': 12},
          'listItems': {'count': 195,
           'items': [{'id': 't54ed3b13498e857fd7dbb6fc',
             'createdAt': 1514680908}]}}]}]},
     'hours': {'status': 'Closed until 11:30 AM',
      'richStatus': {'entities': [], 'text': 'Closed until 11:30 AM'},
      'isOpen': False,
      'isLocalHoliday': False,
      'dayData': [],
      'timeframes': [{'days': 'Mon–Wed, Sun',
        'open': [{'renderedTime': '11:30 AM–11:00 PM'}],
        'segments': []},
       {'days': 'Thu–Sat',
        'includesToday': True,
        'open': [{'renderedTime': '11:30 AM–Midnight'}],
        'segments': []}]},
     'popular': {'isOpen': False,
      'isLocalHoliday': False,
      'timeframes': [{'days': 'Today',
        'includesToday': True,
        'open': [{'renderedTime': 'Noon–2:00 PM'},
         {'renderedTime': '5:00 PM–10:00 PM'}],
        'segments': []},
       {'days': 'Fri',
        'open': [{'renderedTime': 'Noon–3:00 PM'},
         {'renderedTime': '5:00 PM–11:00 PM'}],
        'segments': []},
       {'days': 'Sat',
        'open': [{'renderedTime': 'Noon–11:00 PM'}],
        'segments': []},
       {'days': 'Sun',
        'open': [{'renderedTime': 'Noon–3:00 PM'},
         {'renderedTime': '5:00 PM–8:00 PM'}],
        'segments': []},
       {'days': 'Mon',
        'open': [{'renderedTime': 'Noon–2:00 PM'},
         {'renderedTime': '6:00 PM–8:00 PM'}],
        'segments': []},
       {'days': 'Tue–Wed',
        'open': [{'renderedTime': 'Noon–2:00 PM'},
         {'renderedTime': '5:00 PM–10:00 PM'}],
        'segments': []}]},
     'pageUpdates': {'count': 0, 'items': []},
     'inbox': {'count': 0, 'items': []},
     'attributes': {'groups': [{'type': 'price',
        'name': 'Price',
        'summary': '$$',
        'count': 1,
        'items': [{'displayName': 'Price', 'displayValue': '$$', 'priceTier': 2}]},
       {'type': 'payments',
        'name': 'Credit Cards',
        'summary': 'Credit Cards',
        'count': 7,
        'items': [{'displayName': 'Credit Cards',
          'displayValue': 'Yes (incl. American Express)'}]},
       {'type': 'outdoorSeating',
        'name': 'Outdoor Seating',
        'summary': 'Outdoor Seating',
        'count': 1,
        'items': [{'displayName': 'Outdoor Seating', 'displayValue': 'Yes'}]},
       {'type': 'serves',
        'name': 'Menus',
        'summary': 'Happy Hour, Brunch & more',
        'count': 8,
        'items': [{'displayName': 'Brunch', 'displayValue': 'Brunch'},
         {'displayName': 'Lunch', 'displayValue': 'Lunch'},
         {'displayName': 'Dinner', 'displayValue': 'Dinner'},
         {'displayName': 'Happy Hour', 'displayValue': 'Happy Hour'}]},
       {'type': 'drinks',
        'name': 'Drinks',
        'summary': 'Beer, Wine & Cocktails',
        'count': 5,
        'items': [{'displayName': 'Beer', 'displayValue': 'Beer'},
         {'displayName': 'Wine', 'displayValue': 'Wine'},
         {'displayName': 'Cocktails', 'displayValue': 'Cocktails'}]},
       {'type': 'diningOptions',
        'name': 'Dining Options',
        'summary': 'Delivery',
        'count': 5,
        'items': [{'displayName': 'Delivery', 'displayValue': 'Delivery'}]}]},
     'bestPhoto': {'id': '4fad980de4b091b4626c3633',
      'createdAt': 1336776717,
      'source': {'name': 'Foursquare for Android',
       'url': 'https://foursquare.com/download/#/android'},
      'prefix': 'https://igx.4sqi.net/img/general/',
      'suffix': '/ya1iQFI7pLjuIJp1PGDKlrZS3OJdHCF7tpILMmjv_2w.jpg',
      'width': 480,
      'height': 640,
      'visibility': 'public'},
     'colors': {'highlightColor': {'photoId': '4fad980de4b091b4626c3633',
       'value': -13619152},
      'highlightTextColor': {'photoId': '4fad980de4b091b4626c3633', 'value': -1},
      'algoVersion': 3}}



### B. Get the venue's overall rating


```python
try:
    print(result['response']['venue']['rating'])
except:
    print('This venue has not been rated yet.')
```

    7.3


That is not a very good rating. Let's check the rating of the second closest Italian restaurant.


```python
venue_id = '4f3232e219836c91c7bfde94' # ID of Conca Cucina Italian Restaurant
url = 'https://api.foursquare.com/v2/venues/{}?client_id={}&client_secret={}&v={}'.format(venue_id, CLIENT_ID, CLIENT_SECRET, VERSION)

result = requests.get(url).json()
try:
    print(result['response']['venue']['rating'])
except:
    print('This venue has not been rated yet.')
```

    This venue has not been rated yet.


Since this restaurant has no ratings, let's check the third restaurant.


```python
venue_id = '3fd66200f964a520f4e41ee3' # ID of Ecco
url = 'https://api.foursquare.com/v2/venues/{}?client_id={}&client_secret={}&v={}'.format(venue_id, CLIENT_ID, CLIENT_SECRET, VERSION)

result = requests.get(url).json()
try:
    print(result['response']['venue']['rating'])
except:
    print('This venue has not been rated yet.')
```

    7.7


Since this restaurant has a slightly better rating, let's explore it further.

### C. Get the number of tips


```python
result['response']['venue']['tips']['count']
```




    15



### D. Get the venue's tips
> `https://api.foursquare.com/v2/venues/`**VENUE_ID**`/tips?client_id=`**CLIENT_ID**`&client_secret=`**CLIENT_SECRET**`&v=`**VERSION**`&limit=`**LIMIT**

#### Create URL and send GET request. Make sure to set limit to get all tips


```python
## Ecco Tips
limit = 15 # set limit to be greater than or equal to the total number of tips
url = 'https://api.foursquare.com/v2/venues/{}/tips?client_id={}&client_secret={}&v={}&limit={}'.format(venue_id, CLIENT_ID, CLIENT_SECRET, VERSION, limit)

results = requests.get(url).json()
#results
```

#### Get tips and list of associated features


```python
tips = results['response']['tips']['items']

tip = results['response']['tips']['items'][0]
tip.keys()
```




    dict_keys(['id', 'createdAt', 'text', 'type', 'canonicalUrl', 'lang', 'likes', 'logView', 'agreeCount', 'disagreeCount', 'lastVoteText', 'lastUpvoteTimestamp', 'todo', 'user', 'authorInteractionType'])



#### Format column width and display all tips


```python
pd.set_option('display.max_colwidth', -1)

tips_df = json_normalize(tips) # json normalize tips

# columns to keep
filtered_columns = ['text', 'agreeCount', 'disagreeCount', 'id', 'user.firstName', 'user.lastName', 'user.gender', 'user.id']
tips_filtered = tips_df.loc[:, filtered_columns]

# display tips
tips_filtered
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>text</th>
      <th>agreeCount</th>
      <th>disagreeCount</th>
      <th>id</th>
      <th>user.firstName</th>
      <th>user.lastName</th>
      <th>user.gender</th>
      <th>user.id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A+ Italian food! Trust me on this: my mom’s side of the family is 100% Italian. I was born and bred to know good pasta when I see it, and Ecco is one of my all-time NYC favorites</td>
      <td>2</td>
      <td>0</td>
      <td>5ab1cb46c9a517174651d3fe</td>
      <td>Nick</td>
      <td>El-Tawil</td>
      <td>male</td>
      <td>484542633</td>
    </tr>
  </tbody>
</table>
</div>



Now remember that because we are using a personal developer account, then we can access only 2 of the restaurant's tips, instead of all 15 tips.

   

<a id="item3"></a>

## 3. Search a Foursquare User
> `https://api.foursquare.com/v2/users/`**USER_ID**`?client_id=`**CLIENT_ID**`&client_secret=`**CLIENT_SECRET**`&v=`**VERSION**

### Define URL, send GET request and display features associated with user


```python
user_id = '484542633' # user ID with most agree counts and complete profile

url = 'https://api.foursquare.com/v2/users/{}?client_id={}&client_secret={}&v={}'.format(user_id, CLIENT_ID, CLIENT_SECRET, VERSION) # define URL

# send GET request
results = requests.get(url).json()
user_data = results['response']['user']

# display features associated with user
user_data.keys()
```


```python
print('First Name: ' + user_data['firstName'])
print('Last Name: ' + user_data['lastName'])
print('Home City: ' + user_data['homeCity'])
```

#### How many tips has this user submitted?


```python
user_data['tips']
```

Wow! So it turns out that Nick is a very active Foursquare user, with more than 250 tips.

### Get User's tips


```python
# define tips URL
url = 'https://api.foursquare.com/v2/users/{}/tips?client_id={}&client_secret={}&v={}&limit={}'.format(user_id, CLIENT_ID, CLIENT_SECRET, VERSION, limit)

# send GET request and get user's tips
results = requests.get(url).json()
tips = results['response']['tips']['items']

# format column width
pd.set_option('display.max_colwidth', -1)

tips_df = json_normalize(tips)

# filter columns
filtered_columns = ['text', 'agreeCount', 'disagreeCount', 'id']
tips_filtered = tips_df.loc[:, filtered_columns]

# display user's tips
tips_filtered
```

#### Let's get the venue for the tip with the greatest number of agree counts


```python
tip_id = '5ab5575d73fe2516ad8f363b' # tip id

# define URL
url = 'http://api.foursquare.com/v2/tips/{}?client_id={}&client_secret={}&v={}'.format(tip_id, CLIENT_ID, CLIENT_SECRET, VERSION)

# send GET Request and examine results
result = requests.get(url).json()
print(result['response']['tip']['venue']['name'])
print(result['response']['tip']['venue']['location'])
```

### Get User's friends


```python
user_friends = json_normalize(user_data['friends']['groups'][0]['items'])
user_friends
```

Interesting. Despite being very active, it turns out that Nick does not have any friends on Foursquare. This might definitely change in the future.

### Retrieve the User's Profile Image


```python
user_data
```


```python
# 1. grab prefix of photo
# 2. grab suffix of photo
# 3. concatenate them using the image size  
Image(url='https://igx.4sqi.net/img/user/300x300/484542633_mK2Yum7T_7Tn9fWpndidJsmw2Hof_6T5vJBKCHPLMK5OL-U5ZiJGj51iwBstcpDLYa3Zvhvis.jpg')
```

  

<a id="item4"></a>

## 4. Explore a location
> `https://api.foursquare.com/v2/venues/`**explore**`?client_id=`**CLIENT_ID**`&client_secret=`**CLIENT_SECRET**`&ll=`**LATITUDE**`,`**LONGITUDE**`&v=`**VERSION**`&limit=`**LIMIT**

#### So, you just finished your gourmet dish at Ecco, and are just curious about the popular spots around the restaurant. In order to explore the area, let's start by getting the latitude and longitude values of Ecco Restaurant.


```python
latitude = 40.715337
longitude = -74.008848
```

#### Define URL


```python
url = 'https://api.foursquare.com/v2/venues/explore?client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}'.format(CLIENT_ID, CLIENT_SECRET, latitude, longitude, VERSION, radius, LIMIT)
url
```

#### Send GET request and examine results


```python
import requests
```


```python
results = requests.get(url).json()
'There are {} around Ecco restaurant.'.format(len(results['response']['groups'][0]['items']))
```

#### Get relevant part of JSON


```python
items = results['response']['groups'][0]['items']
items[0]
```

#### Process JSON and convert it to a clean dataframe


```python
dataframe = json_normalize(items) # flatten JSON

# filter columns
filtered_columns = ['venue.name', 'venue.categories'] + [col for col in dataframe.columns if col.startswith('venue.location.')] + ['venue.id']
dataframe_filtered = dataframe.loc[:, filtered_columns]

# filter the category for each row
dataframe_filtered['venue.categories'] = dataframe_filtered.apply(get_category_type, axis=1)

# clean columns
dataframe_filtered.columns = [col.split('.')[-1] for col in dataframe_filtered.columns]

dataframe_filtered.head(10)
```

#### Let's visualize these items on the map around our location


```python
venues_map = folium.Map(location=[latitude, longitude], zoom_start=15) # generate map centred around Ecco


# add Ecco as a red circle mark
folium.features.CircleMarker(
    [latitude, longitude],
    radius=10,
    popup='Ecco',
    fill=True,
    color='red',
    fill_color='red',
    fill_opacity=0.6
    ).add_to(venues_map)


# add popular spots to the map as blue circle markers
for lat, lng, label in zip(dataframe_filtered.lat, dataframe_filtered.lng, dataframe_filtered.categories):
    folium.features.CircleMarker(
        [lat, lng],
        radius=5,
        popup=label,
        fill=True,
        color='blue',
        fill_color='blue',
        fill_opacity=0.6
        ).add_to(venues_map)

# display map
venues_map
```

   

<a id="item5"></a>

## 5. Explore Trending Venues
> `https://api.foursquare.com/v2/venues/`**trending**`?client_id=`**CLIENT_ID**`&client_secret=`**CLIENT_SECRET**`&ll=`**LATITUDE**`,`**LONGITUDE**`&v=`**VERSION**

#### Now, instead of simply exploring the area around Ecco, you are interested in knowing the venues that are trending at the time you are done with your lunch, meaning the places with the highest foot traffic. So let's do that and get the trending venues around Ecco.


```python
# define URL
url = 'https://api.foursquare.com/v2/venues/trending?client_id={}&client_secret={}&ll={},{}&v={}'.format(CLIENT_ID, CLIENT_SECRET, latitude, longitude, VERSION)

# send GET request and get trending venues
results = requests.get(url).json()
results
```

### Check if any venues are trending at this time


```python
if len(results['response']['venues']) == 0:
    trending_venues_df = 'No trending venues are available at the moment!'
    
else:
    trending_venues = results['response']['venues']
    trending_venues_df = json_normalize(trending_venues)

    # filter columns
    columns_filtered = ['name', 'categories'] + ['location.distance', 'location.city', 'location.postalCode', 'location.state', 'location.country', 'location.lat', 'location.lng']
    trending_venues_df = trending_venues_df.loc[:, columns_filtered]

    # filter the category for each row
    trending_venues_df['categories'] = trending_venues_df.apply(get_category_type, axis=1)
```


```python
# display trending venues
trending_venues_df
```

Now, depending on when you run the above code, you might get different venues since the venues with the highest foot traffic are fetched live. 

### Visualize trending venues


```python
if len(results['response']['venues']) == 0:
    venues_map = 'Cannot generate visual as no trending venues are available at the moment!'

else:
    venues_map = folium.Map(location=[latitude, longitude], zoom_start=15) # generate map centred around Ecco


    # add Ecco as a red circle mark
    folium.features.CircleMarker(
        [latitude, longitude],
        radius=10,
        popup='Ecco',
        fill=True,
        color='red',
        fill_color='red',
        fill_opacity=0.6
    ).add_to(venues_map)


    # add the trending venues as blue circle markers
    for lat, lng, label in zip(trending_venues_df['location.lat'], trending_venues_df['location.lng'], trending_venues_df['name']):
        folium.features.CircleMarker(
            [lat, lng],
            radius=5,
            poup=label,
            fill=True,
            color='blue',
            fill_color='blue',
            fill_opacity=0.6
        ).add_to(venues_map)
```


```python
# display map
venues_map
```

<a id="item6"></a>

   

### Thank you for completing this lab!

This notebook was created by [Alex Aklson](https://www.linkedin.com/in/aklson/). I hope you found this lab interesting and educational. Feel free to contact me if you have any questions!

This notebook is part of a course on **Coursera** called *Applied Data Science Capstone*. If you accessed this notebook outside the course, you can take this course online by clicking [here](http://cocl.us/DP0701EN_Coursera_Week2_LAB1).

<hr>
Copyright &copy; 2018 [Cognitive Class](https://cognitiveclass.ai/?utm_source=bducopyrightlink&utm_medium=dswb&utm_campaign=bdu). This notebook and its source code are released under the terms of the [MIT License](https://bigdatauniversity.com/mit-license/).
