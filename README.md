

```python
import os 
import pandas as pd
import csv
import numpy as np
import json
import requests as req
import matplotlib.pyplot as plt
import openweathermapy.core as ow
import time 
import random
from citipy import citipy
```


```python

# lat and lon variables * Lat and lon can be generated with 2 min/max values but the decimal values make it more precise/// also in the case of this exercise it seems that you need to start with lat/lon data before 
# random city data. My initial approach for getting random.sample city data using citipy was not working

lat = list(np.arange(-90, 90.01, 0.01))
lon = list(np.arange(-180, 180.01, 0.01))

# get random lat and lon 

lat_ran_samp = random.sample(lat, 1000)
lon_ran_samp = random.sample(lon, 1000)

```


```python
# create database for lat, lon *** Tried making DF with empty columns for city data, but only got errors. 
# Conclusion: intital DF needs data, so don't try to cut corners *** 

cities_df = pd.DataFrame({"Latitude": lat_ran_samp, "Longitude": lon_ran_samp})
cities_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.88</td>
      <td>-12.32</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-74.94</td>
      <td>-87.39</td>
    </tr>
    <tr>
      <th>2</th>
      <td>37.64</td>
      <td>-100.84</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-1.64</td>
      <td>96.33</td>
    </tr>
    <tr>
      <th>4</th>
      <td>65.10</td>
      <td>-99.13</td>
    </tr>
  </tbody>
</table>
</div>




```python
# create columns for cities ** Note: be sure to also include "country", because "city" alone only gives vague data 

cities_df["City"] = ""
cities_df["Country"] = ""

```


```python
# find city and country for lat and lon data /// 

for index, row in cities_df.iterrows(): 
    city = citipy.nearest_city(row["Latitude"], row["Longitude"])
    cities_df.set_value(index, "City", city.city_name)
    cities_df.set_value(index, "Country", city.country_code)  

cities_df=cities_df.drop_duplicates(["City"], keep='first')        
cities_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>City</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.88</td>
      <td>-12.32</td>
      <td>buchanan</td>
      <td>lr</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-74.94</td>
      <td>-87.39</td>
      <td>punta arenas</td>
      <td>cl</td>
    </tr>
    <tr>
      <th>2</th>
      <td>37.64</td>
      <td>-100.84</td>
      <td>garden city</td>
      <td>us</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-1.64</td>
      <td>96.33</td>
      <td>padang</td>
      <td>id</td>
    </tr>
    <tr>
      <th>4</th>
      <td>65.10</td>
      <td>-99.13</td>
      <td>thompson</td>
      <td>ca</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Openweather API call /// 

# Create API settings and store keys 
api_key = "9ab33afcfca2ec3d291dbc05ddc722ca"
settings = {"units":"imperial", "appid": api_key}
url = "http://api.openweathermap.org/data/2.5/weather?"
units = "imperial"
# 'http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q='
complete_query_url = "http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q="
```


```python
cities_df["Temperature"] = ""
cities_df["Humidity"] = ""
cities_df["Clouds"] = ""
cities_df["Wind"] = ""

SleepCounter = 0
PullCounter = 0
BatchCounter = 1

### Loop through the APIs to construct new columns in DF --- refer to "scrap files for 
# alternatibve methods of performing this request. This appears to be the best method.

for index, row in cities_df.iterrows():
    try:
        query = complete_query_url + row["City"].replace(" ","+") + "," + row["Country"]
        get = req.get(query)
        owjson = get.json()
        cities_df.set_value(index, "Temperature", owjson["main"]["temp"])
        cities_df.set_value(index, "Humidity", owjson["main"]["humidity"])
        cities_df.set_value(index, "Clouds", owjson["clouds"]["all"])
        cities_df.set_value(index, "Wind", owjson["wind"]["speed"])
    except:
        cities_df.set_value(index, "Temperature", "FAIL")
    
    PullCounter += 1
    
    SleepCounter += 1
    
    # If loop to ensure not overloading the weather API
    if SleepCounter == 50:
        print("~~~ sleep ~~~")
        time.sleep(10)
        print("")
        SleepCounter = 0
        BatchCounter += 1
    
    # Printing API link
    print("----- begin request -----")
    print("Processing Record " + str(PullCounter) + " of Set " + str(BatchCounter) +" | " + row["City"])
    print(query)
    print("--- request complete ---")
```

    ----- begin request -----
    Processing Record 1 of Set 1 | buchanan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=buchanan,lr
    --- request complete ---
    ----- begin request -----
    Processing Record 2 of Set 1 | punta arenas
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=punta+arenas,cl
    --- request complete ---
    ----- begin request -----
    Processing Record 3 of Set 1 | garden city
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=garden+city,us
    --- request complete ---
    ----- begin request -----
    Processing Record 4 of Set 1 | padang
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=padang,id
    --- request complete ---
    ----- begin request -----
    Processing Record 5 of Set 1 | thompson
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=thompson,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 6 of Set 1 | butaritari
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=butaritari,ki
    --- request complete ---
    ----- begin request -----
    Processing Record 7 of Set 1 | luderitz
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=luderitz,na
    --- request complete ---
    ----- begin request -----
    Processing Record 8 of Set 1 | qaanaaq
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=qaanaaq,gl
    --- request complete ---
    ----- begin request -----
    Processing Record 9 of Set 1 | zboriste
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=zboriste,ba
    --- request complete ---
    ----- begin request -----
    Processing Record 10 of Set 1 | east london
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=east+london,za
    --- request complete ---
    ----- begin request -----
    Processing Record 11 of Set 1 | matamba
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=matamba,tz
    --- request complete ---
    ----- begin request -----
    Processing Record 12 of Set 1 | grand river south east
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=grand+river+south+east,mu
    --- request complete ---
    ----- begin request -----
    Processing Record 13 of Set 1 | kapaa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kapaa,us
    --- request complete ---
    ----- begin request -----
    Processing Record 14 of Set 1 | hilo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=hilo,us
    --- request complete ---
    ----- begin request -----
    Processing Record 15 of Set 1 | cherskiy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cherskiy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 16 of Set 1 | san juan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=san+juan,us
    --- request complete ---
    ----- begin request -----
    Processing Record 17 of Set 1 | san carlos de bariloche
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=san+carlos+de+bariloche,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 18 of Set 1 | bosaso
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bosaso,so
    --- request complete ---
    ----- begin request -----
    Processing Record 19 of Set 1 | rikitea
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=rikitea,pf
    --- request complete ---
    ----- begin request -----
    Processing Record 20 of Set 1 | hermanus
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=hermanus,za
    --- request complete ---
    ----- begin request -----
    Processing Record 21 of Set 1 | hobart
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=hobart,au
    --- request complete ---
    ----- begin request -----
    Processing Record 22 of Set 1 | belmonte
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=belmonte,br
    --- request complete ---
    ----- begin request -----
    Processing Record 23 of Set 1 | tiksi
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tiksi,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 24 of Set 1 | castro
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=castro,cl
    --- request complete ---
    ----- begin request -----
    Processing Record 25 of Set 1 | tuatapere
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tuatapere,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 26 of Set 1 | mandalgovi
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mandalgovi,mn
    --- request complete ---
    ----- begin request -----
    Processing Record 27 of Set 1 | jamestown
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=jamestown,sh
    --- request complete ---
    ----- begin request -----
    Processing Record 28 of Set 1 | sentyabrskiy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sentyabrskiy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 29 of Set 1 | saint anthony
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saint+anthony,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 30 of Set 1 | ushuaia
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ushuaia,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 31 of Set 1 | adamovka
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=adamovka,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 32 of Set 1 | bokspits
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bokspits,bw
    --- request complete ---
    ----- begin request -----
    Processing Record 33 of Set 1 | barrow
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=barrow,us
    --- request complete ---
    ----- begin request -----
    Processing Record 34 of Set 1 | provideniya
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=provideniya,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 35 of Set 1 | aquiraz
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=aquiraz,br
    --- request complete ---
    ----- begin request -----
    Processing Record 36 of Set 1 | new norfolk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=new+norfolk,au
    --- request complete ---
    ----- begin request -----
    Processing Record 37 of Set 1 | general pico
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=general+pico,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 38 of Set 1 | wanning
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=wanning,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 39 of Set 1 | taolanaro
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=taolanaro,mg
    --- request complete ---
    ----- begin request -----
    Processing Record 40 of Set 1 | upernavik
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=upernavik,gl
    --- request complete ---
    ----- begin request -----
    Processing Record 41 of Set 1 | la ronge
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=la+ronge,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 42 of Set 1 | vaitupu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vaitupu,wf
    --- request complete ---
    ----- begin request -----
    Processing Record 43 of Set 1 | mataura
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mataura,pf
    --- request complete ---
    ----- begin request -----
    Processing Record 44 of Set 1 | sabla
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sabla,bg
    --- request complete ---
    ----- begin request -----
    Processing Record 45 of Set 1 | porto walter
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=porto+walter,br
    --- request complete ---
    ----- begin request -----
    Processing Record 46 of Set 1 | palmas de monte alto
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=palmas+de+monte+alto,br
    --- request complete ---
    ----- begin request -----
    Processing Record 47 of Set 1 | belushya guba
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=belushya+guba,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 48 of Set 1 | tasiilaq
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tasiilaq,gl
    --- request complete ---
    ----- begin request -----
    Processing Record 49 of Set 1 | siuna
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=siuna,ni
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 50 of Set 2 | krasnouralsk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=krasnouralsk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 51 of Set 2 | jiddah
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=jiddah,sa
    --- request complete ---
    ----- begin request -----
    Processing Record 52 of Set 2 | lukovetskiy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lukovetskiy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 53 of Set 2 | paamiut
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=paamiut,gl
    --- request complete ---
    ----- begin request -----
    Processing Record 54 of Set 2 | otradnoye
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=otradnoye,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 55 of Set 2 | faya
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=faya,td
    --- request complete ---
    ----- begin request -----
    Processing Record 56 of Set 2 | zhicheng
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=zhicheng,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 57 of Set 2 | budogoshch
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=budogoshch,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 58 of Set 2 | kamenskoye
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kamenskoye,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 59 of Set 2 | nizhneyansk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nizhneyansk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 60 of Set 2 | alofi
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=alofi,nu
    --- request complete ---
    ----- begin request -----
    Processing Record 61 of Set 2 | sumbe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sumbe,ao
    --- request complete ---
    ----- begin request -----
    Processing Record 62 of Set 2 | wexford
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=wexford,ie
    --- request complete ---
    ----- begin request -----
    Processing Record 63 of Set 2 | natal
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=natal,br
    --- request complete ---
    ----- begin request -----
    Processing Record 64 of Set 2 | cape town
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cape+town,za
    --- request complete ---
    ----- begin request -----
    Processing Record 65 of Set 2 | tilichiki
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tilichiki,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 66 of Set 2 | illoqqortoormiut
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=illoqqortoormiut,gl
    --- request complete ---
    ----- begin request -----
    Processing Record 67 of Set 2 | mogadishu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mogadishu,so
    --- request complete ---
    ----- begin request -----
    Processing Record 68 of Set 2 | korla
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=korla,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 69 of Set 2 | pisco
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=pisco,pe
    --- request complete ---
    ----- begin request -----
    Processing Record 70 of Set 2 | lima
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lima,pe
    --- request complete ---
    ----- begin request -----
    Processing Record 71 of Set 2 | cidreira
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cidreira,br
    --- request complete ---
    ----- begin request -----
    Processing Record 72 of Set 2 | bonavista
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bonavista,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 73 of Set 2 | oskarshamn
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=oskarshamn,se
    --- request complete ---
    ----- begin request -----
    Processing Record 74 of Set 2 | kailua
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kailua,us
    --- request complete ---
    ----- begin request -----
    Processing Record 75 of Set 2 | tuktoyaktuk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tuktoyaktuk,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 76 of Set 2 | bubaque
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bubaque,gw
    --- request complete ---
    ----- begin request -----
    Processing Record 77 of Set 2 | rognan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=rognan,no
    --- request complete ---
    ----- begin request -----
    Processing Record 78 of Set 2 | ribeira grande
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ribeira+grande,pt
    --- request complete ---
    ----- begin request -----
    Processing Record 79 of Set 2 | bac lieu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bac+lieu,vn
    --- request complete ---
    ----- begin request -----
    Processing Record 80 of Set 2 | mrirt
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mrirt,ma
    --- request complete ---
    ----- begin request -----
    Processing Record 81 of Set 2 | chokurdakh
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=chokurdakh,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 82 of Set 2 | wahran
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=wahran,dz
    --- request complete ---
    ----- begin request -----
    Processing Record 83 of Set 2 | ovsyanka
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ovsyanka,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 84 of Set 2 | arraial do cabo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=arraial+do+cabo,br
    --- request complete ---
    ----- begin request -----
    Processing Record 85 of Set 2 | waingapu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=waingapu,id
    --- request complete ---
    ----- begin request -----
    Processing Record 86 of Set 2 | storforshei
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=storforshei,no
    --- request complete ---
    ----- begin request -----
    Processing Record 87 of Set 2 | vaini
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vaini,to
    --- request complete ---
    ----- begin request -----
    Processing Record 88 of Set 2 | atuona
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=atuona,pf
    --- request complete ---
    ----- begin request -----
    Processing Record 89 of Set 2 | oktyabrskoye
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=oktyabrskoye,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 90 of Set 2 | puerto ayora
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=puerto+ayora,ec
    --- request complete ---
    ----- begin request -----
    Processing Record 91 of Set 2 | albany
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=albany,au
    --- request complete ---
    ----- begin request -----
    Processing Record 92 of Set 2 | bambous virieux
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bambous+virieux,mu
    --- request complete ---
    ----- begin request -----
    Processing Record 93 of Set 2 | college
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=college,us
    --- request complete ---
    ----- begin request -----
    Processing Record 94 of Set 2 | bluff
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bluff,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 95 of Set 2 | thanh hoa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=thanh+hoa,vn
    --- request complete ---
    ----- begin request -----
    Processing Record 96 of Set 2 | chuy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=chuy,uy
    --- request complete ---
    ----- begin request -----
    Processing Record 97 of Set 2 | port elizabeth
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=port+elizabeth,za
    --- request complete ---
    ----- begin request -----
    Processing Record 98 of Set 2 | norman wells
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=norman+wells,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 99 of Set 2 | georgetown
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=georgetown,sh
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 100 of Set 3 | klaksvik
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=klaksvik,fo
    --- request complete ---
    ----- begin request -----
    Processing Record 101 of Set 3 | bauchi
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bauchi,ng
    --- request complete ---
    ----- begin request -----
    Processing Record 102 of Set 3 | yellowknife
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=yellowknife,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 103 of Set 3 | nawa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nawa,sy
    --- request complete ---
    ----- begin request -----
    Processing Record 104 of Set 3 | san cristobal
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=san+cristobal,ec
    --- request complete ---
    ----- begin request -----
    Processing Record 105 of Set 3 | molchanovo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=molchanovo,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 106 of Set 3 | tecoanapa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tecoanapa,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 107 of Set 3 | salinopolis
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=salinopolis,br
    --- request complete ---
    ----- begin request -----
    Processing Record 108 of Set 3 | kieta
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kieta,pg
    --- request complete ---
    ----- begin request -----
    Processing Record 109 of Set 3 | saint-philippe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saint-philippe,re
    --- request complete ---
    ----- begin request -----
    Processing Record 110 of Set 3 | katsuura
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=katsuura,jp
    --- request complete ---
    ----- begin request -----
    Processing Record 111 of Set 3 | dingle
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=dingle,ie
    --- request complete ---
    ----- begin request -----
    Processing Record 112 of Set 3 | bugembe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bugembe,ug
    --- request complete ---
    ----- begin request -----
    Processing Record 113 of Set 3 | buariki
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=buariki,ki
    --- request complete ---
    ----- begin request -----
    Processing Record 114 of Set 3 | gzhatsk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=gzhatsk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 115 of Set 3 | te anau
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=te+anau,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 116 of Set 3 | dindori
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=dindori,in
    --- request complete ---
    ----- begin request -----
    Processing Record 117 of Set 3 | cienfuegos
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cienfuegos,cu
    --- request complete ---
    ----- begin request -----
    Processing Record 118 of Set 3 | saldanha
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saldanha,za
    --- request complete ---
    ----- begin request -----
    Processing Record 119 of Set 3 | port hardy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=port+hardy,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 120 of Set 3 | vasilyevo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vasilyevo,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 121 of Set 3 | atambua
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=atambua,id
    --- request complete ---
    ----- begin request -----
    Processing Record 122 of Set 3 | lolua
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lolua,tv
    --- request complete ---
    ----- begin request -----
    Processing Record 123 of Set 3 | saint george
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saint+george,bm
    --- request complete ---
    ----- begin request -----
    Processing Record 124 of Set 3 | caldwell
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=caldwell,us
    --- request complete ---
    ----- begin request -----
    Processing Record 125 of Set 3 | channel-port aux basques
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=channel-port+aux+basques,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 126 of Set 3 | cacoal
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cacoal,br
    --- request complete ---
    ----- begin request -----
    Processing Record 127 of Set 3 | naraina
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=naraina,in
    --- request complete ---
    ----- begin request -----
    Processing Record 128 of Set 3 | barentsburg
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=barentsburg,sj
    --- request complete ---
    ----- begin request -----
    Processing Record 129 of Set 3 | teguldet
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=teguldet,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 130 of Set 3 | kodiak
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kodiak,us
    --- request complete ---
    ----- begin request -----
    Processing Record 131 of Set 3 | talnakh
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=talnakh,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 132 of Set 3 | maraba
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=maraba,br
    --- request complete ---
    ----- begin request -----
    Processing Record 133 of Set 3 | kampot
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kampot,kh
    --- request complete ---
    ----- begin request -----
    Processing Record 134 of Set 3 | uruzgan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=uruzgan,af
    --- request complete ---
    ----- begin request -----
    Processing Record 135 of Set 3 | bafq
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bafq,ir
    --- request complete ---
    ----- begin request -----
    Processing Record 136 of Set 3 | esperance
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=esperance,au
    --- request complete ---
    ----- begin request -----
    Processing Record 137 of Set 3 | abu kamal
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=abu+kamal,sy
    --- request complete ---
    ----- begin request -----
    Processing Record 138 of Set 3 | lavrentiya
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lavrentiya,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 139 of Set 3 | yaan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=yaan,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 140 of Set 3 | save
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=save,bj
    --- request complete ---
    ----- begin request -----
    Processing Record 141 of Set 3 | taoudenni
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=taoudenni,ml
    --- request complete ---
    ----- begin request -----
    Processing Record 142 of Set 3 | vardo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vardo,no
    --- request complete ---
    ----- begin request -----
    Processing Record 143 of Set 3 | necochea
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=necochea,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 144 of Set 3 | maple creek
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=maple+creek,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 145 of Set 3 | langsa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=langsa,id
    --- request complete ---
    ----- begin request -----
    Processing Record 146 of Set 3 | kapit
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kapit,my
    --- request complete ---
    ----- begin request -----
    Processing Record 147 of Set 3 | helena
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=helena,us
    --- request complete ---
    ----- begin request -----
    Processing Record 148 of Set 3 | bolungarvik
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bolungarvik,is
    --- request complete ---
    ----- begin request -----
    Processing Record 149 of Set 3 | willowmore
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=willowmore,za
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 150 of Set 4 | port alfred
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=port+alfred,za
    --- request complete ---
    ----- begin request -----
    Processing Record 151 of Set 4 | daokou
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=daokou,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 152 of Set 4 | longlac
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=longlac,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 153 of Set 4 | emerald
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=emerald,au
    --- request complete ---
    ----- begin request -----
    Processing Record 154 of Set 4 | khatanga
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=khatanga,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 155 of Set 4 | shitanjing
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=shitanjing,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 156 of Set 4 | bay city
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bay+city,us
    --- request complete ---
    ----- begin request -----
    Processing Record 157 of Set 4 | mar del plata
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mar+del+plata,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 158 of Set 4 | buala
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=buala,sb
    --- request complete ---
    ----- begin request -----
    Processing Record 159 of Set 4 | snyder
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=snyder,us
    --- request complete ---
    ----- begin request -----
    Processing Record 160 of Set 4 | ancud
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ancud,cl
    --- request complete ---
    ----- begin request -----
    Processing Record 161 of Set 4 | aksarka
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=aksarka,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 162 of Set 4 | sitka
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sitka,us
    --- request complete ---
    ----- begin request -----
    Processing Record 163 of Set 4 | nanortalik
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nanortalik,gl
    --- request complete ---
    ----- begin request -----
    Processing Record 164 of Set 4 | half moon bay
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=half+moon+bay,us
    --- request complete ---
    ----- begin request -----
    Processing Record 165 of Set 4 | cheney
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cheney,us
    --- request complete ---
    ----- begin request -----
    Processing Record 166 of Set 4 | busselton
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=busselton,au
    --- request complete ---
    ----- begin request -----
    Processing Record 167 of Set 4 | bacolod
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bacolod,ph
    --- request complete ---
    ----- begin request -----
    Processing Record 168 of Set 4 | airai
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=airai,pw
    --- request complete ---
    ----- begin request -----
    Processing Record 169 of Set 4 | yulara
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=yulara,au
    --- request complete ---
    ----- begin request -----
    Processing Record 170 of Set 4 | bredasdorp
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bredasdorp,za
    --- request complete ---
    ----- begin request -----
    Processing Record 171 of Set 4 | sao felix do xingu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sao+felix+do+xingu,br
    --- request complete ---
    ----- begin request -----
    Processing Record 172 of Set 4 | grand gaube
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=grand+gaube,mu
    --- request complete ---
    ----- begin request -----
    Processing Record 173 of Set 4 | swan hill
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=swan+hill,au
    --- request complete ---
    ----- begin request -----
    Processing Record 174 of Set 4 | vaitape
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vaitape,pf
    --- request complete ---
    ----- begin request -----
    Processing Record 175 of Set 4 | leningradskiy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=leningradskiy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 176 of Set 4 | lata
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lata,sb
    --- request complete ---
    ----- begin request -----
    Processing Record 177 of Set 4 | rzhaksa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=rzhaksa,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 178 of Set 4 | olkhovatka
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=olkhovatka,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 179 of Set 4 | belyy yar
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=belyy+yar,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 180 of Set 4 | turayf
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=turayf,sa
    --- request complete ---
    ----- begin request -----
    Processing Record 181 of Set 4 | labuhan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=labuhan,id
    --- request complete ---
    ----- begin request -----
    Processing Record 182 of Set 4 | teterow
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=teterow,de
    --- request complete ---
    ----- begin request -----
    Processing Record 183 of Set 4 | san jose
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=san+jose,bo
    --- request complete ---
    ----- begin request -----
    Processing Record 184 of Set 4 | lompoc
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lompoc,us
    --- request complete ---
    ----- begin request -----
    Processing Record 185 of Set 4 | dali
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=dali,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 186 of Set 4 | geraldton
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=geraldton,au
    --- request complete ---
    ----- begin request -----
    Processing Record 187 of Set 4 | saryg-sep
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saryg-sep,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 188 of Set 4 | rungata
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=rungata,ki
    --- request complete ---
    ----- begin request -----
    Processing Record 189 of Set 4 | chumikan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=chumikan,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 190 of Set 4 | mizdah
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mizdah,ly
    --- request complete ---
    ----- begin request -----
    Processing Record 191 of Set 4 | fare
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=fare,pf
    --- request complete ---
    ----- begin request -----
    Processing Record 192 of Set 4 | portland
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=portland,au
    --- request complete ---
    ----- begin request -----
    Processing Record 193 of Set 4 | cabo san lucas
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cabo+san+lucas,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 194 of Set 4 | dunda
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=dunda,tz
    --- request complete ---
    ----- begin request -----
    Processing Record 195 of Set 4 | cambridge
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cambridge,gb
    --- request complete ---
    ----- begin request -----
    Processing Record 196 of Set 4 | gwadar
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=gwadar,pk
    --- request complete ---
    ----- begin request -----
    Processing Record 197 of Set 4 | mayna
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mayna,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 198 of Set 4 | saleaula
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saleaula,ws
    --- request complete ---
    ----- begin request -----
    Processing Record 199 of Set 4 | victoria
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=victoria,sc
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 200 of Set 5 | turan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=turan,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 201 of Set 5 | sundumbili
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sundumbili,za
    --- request complete ---
    ----- begin request -----
    Processing Record 202 of Set 5 | harper
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=harper,lr
    --- request complete ---
    ----- begin request -----
    Processing Record 203 of Set 5 | comodoro rivadavia
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=comodoro+rivadavia,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 204 of Set 5 | auki
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=auki,sb
    --- request complete ---
    ----- begin request -----
    Processing Record 205 of Set 5 | visnes
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=visnes,no
    --- request complete ---
    ----- begin request -----
    Processing Record 206 of Set 5 | vero beach
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vero+beach,us
    --- request complete ---
    ----- begin request -----
    Processing Record 207 of Set 5 | mutsamudu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mutsamudu,km
    --- request complete ---
    ----- begin request -----
    Processing Record 208 of Set 5 | westport
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=westport,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 209 of Set 5 | vikhorevka
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vikhorevka,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 210 of Set 5 | oistins
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=oistins,bb
    --- request complete ---
    ----- begin request -----
    Processing Record 211 of Set 5 | cap malheureux
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cap+malheureux,mu
    --- request complete ---
    ----- begin request -----
    Processing Record 212 of Set 5 | mount gambier
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mount+gambier,au
    --- request complete ---
    ----- begin request -----
    Processing Record 213 of Set 5 | yerbogachen
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=yerbogachen,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 214 of Set 5 | praia da vitoria
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=praia+da+vitoria,pt
    --- request complete ---
    ----- begin request -----
    Processing Record 215 of Set 5 | doka
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=doka,sd
    --- request complete ---
    ----- begin request -----
    Processing Record 216 of Set 5 | pinawa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=pinawa,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 217 of Set 5 | rondonopolis
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=rondonopolis,br
    --- request complete ---
    ----- begin request -----
    Processing Record 218 of Set 5 | lasa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lasa,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 219 of Set 5 | san francisco
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=san+francisco,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 220 of Set 5 | aberdeen
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=aberdeen,us
    --- request complete ---
    ----- begin request -----
    Processing Record 221 of Set 5 | ugoofaaru
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ugoofaaru,mv
    --- request complete ---
    ----- begin request -----
    Processing Record 222 of Set 5 | nikolskoye
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nikolskoye,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 223 of Set 5 | zlobin
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=zlobin,by
    --- request complete ---
    ----- begin request -----
    Processing Record 224 of Set 5 | sijunjung
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sijunjung,id
    --- request complete ---
    ----- begin request -----
    Processing Record 225 of Set 5 | mao
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mao,td
    --- request complete ---
    ----- begin request -----
    Processing Record 226 of Set 5 | los llanos de aridane
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=los+llanos+de+aridane,es
    --- request complete ---
    ----- begin request -----
    Processing Record 227 of Set 5 | kavaratti
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kavaratti,in
    --- request complete ---
    ----- begin request -----
    Processing Record 228 of Set 5 | rojhan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=rojhan,pk
    --- request complete ---
    ----- begin request -----
    Processing Record 229 of Set 5 | senno
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=senno,by
    --- request complete ---
    ----- begin request -----
    Processing Record 230 of Set 5 | galitsy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=galitsy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 231 of Set 5 | karkaralinsk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=karkaralinsk,kz
    --- request complete ---
    ----- begin request -----
    Processing Record 232 of Set 5 | torbay
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=torbay,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 233 of Set 5 | ahuimanu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ahuimanu,us
    --- request complete ---
    ----- begin request -----
    Processing Record 234 of Set 5 | narsaq
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=narsaq,gl
    --- request complete ---
    ----- begin request -----
    Processing Record 235 of Set 5 | ketchikan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ketchikan,us
    --- request complete ---
    ----- begin request -----
    Processing Record 236 of Set 5 | degollado
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=degollado,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 237 of Set 5 | listvyanka
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=listvyanka,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 238 of Set 5 | barberino di mugello
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=barberino+di+mugello,it
    --- request complete ---
    ----- begin request -----
    Processing Record 239 of Set 5 | xining
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=xining,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 240 of Set 5 | umzimvubu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=umzimvubu,za
    --- request complete ---
    ----- begin request -----
    Processing Record 241 of Set 5 | polyarnyy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=polyarnyy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 242 of Set 5 | monterey
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=monterey,us
    --- request complete ---
    ----- begin request -----
    Processing Record 243 of Set 5 | umm lajj
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=umm+lajj,sa
    --- request complete ---
    ----- begin request -----
    Processing Record 244 of Set 5 | mount isa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mount+isa,au
    --- request complete ---
    ----- begin request -----
    Processing Record 245 of Set 5 | aguimes
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=aguimes,es
    --- request complete ---
    ----- begin request -----
    Processing Record 246 of Set 5 | atar
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=atar,mr
    --- request complete ---
    ----- begin request -----
    Processing Record 247 of Set 5 | nam tha
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nam+tha,la
    --- request complete ---
    ----- begin request -----
    Processing Record 248 of Set 5 | namibe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=namibe,ao
    --- request complete ---
    ----- begin request -----
    Processing Record 249 of Set 5 | ilhabela
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ilhabela,br
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 250 of Set 6 | dikson
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=dikson,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 251 of Set 6 | bilma
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bilma,ne
    --- request complete ---
    ----- begin request -----
    Processing Record 252 of Set 6 | huescar
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=huescar,es
    --- request complete ---
    ----- begin request -----
    Processing Record 253 of Set 6 | marsh harbour
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=marsh+harbour,bs
    --- request complete ---
    ----- begin request -----
    Processing Record 254 of Set 6 | george
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=george,za
    --- request complete ---
    ----- begin request -----
    Processing Record 255 of Set 6 | hithadhoo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=hithadhoo,mv
    --- request complete ---
    ----- begin request -----
    Processing Record 256 of Set 6 | moose factory
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=moose+factory,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 257 of Set 6 | zhigansk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=zhigansk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 258 of Set 6 | lagoa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lagoa,pt
    --- request complete ---
    ----- begin request -----
    Processing Record 259 of Set 6 | hamilton
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=hamilton,bm
    --- request complete ---
    ----- begin request -----
    Processing Record 260 of Set 6 | felidhoo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=felidhoo,mv
    --- request complete ---
    ----- begin request -----
    Processing Record 261 of Set 6 | ijaki
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ijaki,ki
    --- request complete ---
    ----- begin request -----
    Processing Record 262 of Set 6 | sanok
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sanok,pl
    --- request complete ---
    ----- begin request -----
    Processing Record 263 of Set 6 | petatlan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=petatlan,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 264 of Set 6 | requena
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=requena,pe
    --- request complete ---
    ----- begin request -----
    Processing Record 265 of Set 6 | eirunepe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=eirunepe,br
    --- request complete ---
    ----- begin request -----
    Processing Record 266 of Set 6 | kaeo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kaeo,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 267 of Set 6 | asau
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=asau,tv
    --- request complete ---
    ----- begin request -----
    Processing Record 268 of Set 6 | kavieng
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kavieng,pg
    --- request complete ---
    ----- begin request -----
    Processing Record 269 of Set 6 | nyrob
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nyrob,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 270 of Set 6 | mergui
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mergui,mm
    --- request complete ---
    ----- begin request -----
    Processing Record 271 of Set 6 | don benito
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=don+benito,es
    --- request complete ---
    ----- begin request -----
    Processing Record 272 of Set 6 | longyearbyen
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=longyearbyen,sj
    --- request complete ---
    ----- begin request -----
    Processing Record 273 of Set 6 | hambantota
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=hambantota,lk
    --- request complete ---
    ----- begin request -----
    Processing Record 274 of Set 6 | babanusah
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=babanusah,sd
    --- request complete ---
    ----- begin request -----
    Processing Record 275 of Set 6 | tsihombe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tsihombe,mg
    --- request complete ---
    ----- begin request -----
    Processing Record 276 of Set 6 | saint-francois
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saint-francois,gp
    --- request complete ---
    ----- begin request -----
    Processing Record 277 of Set 6 | puerto escondido
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=puerto+escondido,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 278 of Set 6 | zemetchino
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=zemetchino,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 279 of Set 6 | sur
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sur,om
    --- request complete ---
    ----- begin request -----
    Processing Record 280 of Set 6 | gurskoye
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=gurskoye,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 281 of Set 6 | bandarbeyla
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bandarbeyla,so
    --- request complete ---
    ----- begin request -----
    Processing Record 282 of Set 6 | srednekolymsk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=srednekolymsk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 283 of Set 6 | washougal
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=washougal,us
    --- request complete ---
    ----- begin request -----
    Processing Record 284 of Set 6 | bud
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bud,no
    --- request complete ---
    ----- begin request -----
    Processing Record 285 of Set 6 | paradwip
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=paradwip,in
    --- request complete ---
    ----- begin request -----
    Processing Record 286 of Set 6 | terra nova
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=terra+nova,br
    --- request complete ---
    ----- begin request -----
    Processing Record 287 of Set 6 | suleja
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=suleja,ng
    --- request complete ---
    ----- begin request -----
    Processing Record 288 of Set 6 | gangakher
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=gangakher,in
    --- request complete ---
    ----- begin request -----
    Processing Record 289 of Set 6 | beloha
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=beloha,mg
    --- request complete ---
    ----- begin request -----
    Processing Record 290 of Set 6 | pauini
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=pauini,br
    --- request complete ---
    ----- begin request -----
    Processing Record 291 of Set 6 | antofagasta
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=antofagasta,cl
    --- request complete ---
    ----- begin request -----
    Processing Record 292 of Set 6 | morondava
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=morondava,mg
    --- request complete ---
    ----- begin request -----
    Processing Record 293 of Set 6 | la asuncion
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=la+asuncion,ve
    --- request complete ---
    ----- begin request -----
    Processing Record 294 of Set 6 | zheleznodorozhnyy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=zheleznodorozhnyy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 295 of Set 6 | port macquarie
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=port+macquarie,au
    --- request complete ---
    ----- begin request -----
    Processing Record 296 of Set 6 | caravelas
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=caravelas,br
    --- request complete ---
    ----- begin request -----
    Processing Record 297 of Set 6 | horsham
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=horsham,au
    --- request complete ---
    ----- begin request -----
    Processing Record 298 of Set 6 | mahebourg
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mahebourg,mu
    --- request complete ---
    ----- begin request -----
    Processing Record 299 of Set 6 | chapais
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=chapais,ca
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 300 of Set 7 | alice springs
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=alice+springs,au
    --- request complete ---
    ----- begin request -----
    Processing Record 301 of Set 7 | aswan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=aswan,eg
    --- request complete ---
    ----- begin request -----
    Processing Record 302 of Set 7 | fortuna
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=fortuna,us
    --- request complete ---
    ----- begin request -----
    Processing Record 303 of Set 7 | zlatoustovsk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=zlatoustovsk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 304 of Set 7 | sao filipe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sao+filipe,cv
    --- request complete ---
    ----- begin request -----
    Processing Record 305 of Set 7 | clyde
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=clyde,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 306 of Set 7 | saskylakh
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saskylakh,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 307 of Set 7 | attawapiskat
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=attawapiskat,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 308 of Set 7 | vila franca do campo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vila+franca+do+campo,pt
    --- request complete ---
    ----- begin request -----
    Processing Record 309 of Set 7 | tahe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tahe,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 310 of Set 7 | nemuro
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nemuro,jp
    --- request complete ---
    ----- begin request -----
    Processing Record 311 of Set 7 | floreffe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=floreffe,be
    --- request complete ---
    ----- begin request -----
    Processing Record 312 of Set 7 | avarua
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=avarua,ck
    --- request complete ---
    ----- begin request -----
    Processing Record 313 of Set 7 | inhambane
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=inhambane,mz
    --- request complete ---
    ----- begin request -----
    Processing Record 314 of Set 7 | saryshagan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=saryshagan,kz
    --- request complete ---
    ----- begin request -----
    Processing Record 315 of Set 7 | cam pha
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cam+pha,vn
    --- request complete ---
    ----- begin request -----
    Processing Record 316 of Set 7 | roald
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=roald,no
    --- request complete ---
    ----- begin request -----
    Processing Record 317 of Set 7 | kimbe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kimbe,pg
    --- request complete ---
    ----- begin request -----
    Processing Record 318 of Set 7 | diego de almagro
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=diego+de+almagro,cl
    --- request complete ---
    ----- begin request -----
    Processing Record 319 of Set 7 | sinnamary
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sinnamary,gf
    --- request complete ---
    ----- begin request -----
    Processing Record 320 of Set 7 | coquimbo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=coquimbo,cl
    --- request complete ---
    ----- begin request -----
    Processing Record 321 of Set 7 | rio gallegos
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=rio+gallegos,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 322 of Set 7 | hofn
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=hofn,is
    --- request complete ---
    ----- begin request -----
    Processing Record 323 of Set 7 | shirpur
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=shirpur,in
    --- request complete ---
    ----- begin request -----
    Processing Record 324 of Set 7 | takapau
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=takapau,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 325 of Set 7 | nema
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nema,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 326 of Set 7 | amderma
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=amderma,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 327 of Set 7 | storslett
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=storslett,no
    --- request complete ---
    ----- begin request -----
    Processing Record 328 of Set 7 | khandyga
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=khandyga,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 329 of Set 7 | severo-kurilsk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=severo-kurilsk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 330 of Set 7 | sorland
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sorland,no
    --- request complete ---
    ----- begin request -----
    Processing Record 331 of Set 7 | cayenne
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=cayenne,gf
    --- request complete ---
    ----- begin request -----
    Processing Record 332 of Set 7 | faanui
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=faanui,pf
    --- request complete ---
    ----- begin request -----
    Processing Record 333 of Set 7 | hasaki
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=hasaki,jp
    --- request complete ---
    ----- begin request -----
    Processing Record 334 of Set 7 | kruisfontein
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kruisfontein,za
    --- request complete ---
    ----- begin request -----
    Processing Record 335 of Set 7 | pevek
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=pevek,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 336 of Set 7 | vostok
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vostok,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 337 of Set 7 | makakilo city
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=makakilo+city,us
    --- request complete ---
    ----- begin request -----
    Processing Record 338 of Set 7 | nalut
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nalut,ly
    --- request complete ---
    ----- begin request -----
    Processing Record 339 of Set 7 | whithorn
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=whithorn,gb
    --- request complete ---
    ----- begin request -----
    Processing Record 340 of Set 7 | okhotsk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=okhotsk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 341 of Set 7 | quipama
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=quipama,co
    --- request complete ---
    ----- begin request -----
    Processing Record 342 of Set 7 | clyde river
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=clyde+river,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 343 of Set 7 | bitung
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bitung,id
    --- request complete ---
    ----- begin request -----
    Processing Record 344 of Set 7 | kamaishi
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kamaishi,jp
    --- request complete ---
    ----- begin request -----
    Processing Record 345 of Set 7 | lebu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=lebu,cl
    --- request complete ---
    ----- begin request -----
    Processing Record 346 of Set 7 | laguna
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=laguna,br
    --- request complete ---
    ----- begin request -----
    Processing Record 347 of Set 7 | bardiyah
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bardiyah,ly
    --- request complete ---
    ----- begin request -----
    Processing Record 348 of Set 7 | san patricio
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=san+patricio,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 349 of Set 7 | umiray
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=umiray,ph
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 350 of Set 8 | bayir
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bayir,jo
    --- request complete ---
    ----- begin request -----
    Processing Record 351 of Set 8 | srandakan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=srandakan,id
    --- request complete ---
    ----- begin request -----
    Processing Record 352 of Set 8 | ossora
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ossora,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 353 of Set 8 | kutum
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kutum,sd
    --- request complete ---
    ----- begin request -----
    Processing Record 354 of Set 8 | souillac
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=souillac,mu
    --- request complete ---
    ----- begin request -----
    Processing Record 355 of Set 8 | carnarvon
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=carnarvon,au
    --- request complete ---
    ----- begin request -----
    Processing Record 356 of Set 8 | vilyuysk
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=vilyuysk,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 357 of Set 8 | kedgwick
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kedgwick,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 358 of Set 8 | gambela
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=gambela,et
    --- request complete ---
    ----- begin request -----
    Processing Record 359 of Set 8 | montlucon
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=montlucon,fr
    --- request complete ---
    ----- begin request -----
    Processing Record 360 of Set 8 | bathsheba
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bathsheba,bb
    --- request complete ---
    ----- begin request -----
    Processing Record 361 of Set 8 | izvoarele
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=izvoarele,ro
    --- request complete ---
    ----- begin request -----
    Processing Record 362 of Set 8 | pimentel
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=pimentel,pe
    --- request complete ---
    ----- begin request -----
    Processing Record 363 of Set 8 | rudnyy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=rudnyy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 364 of Set 8 | pingzhuang
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=pingzhuang,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 365 of Set 8 | warqla
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=warqla,dz
    --- request complete ---
    ----- begin request -----
    Processing Record 366 of Set 8 | kununurra
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kununurra,au
    --- request complete ---
    ----- begin request -----
    Processing Record 367 of Set 8 | kuala terengganu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kuala+terengganu,my
    --- request complete ---
    ----- begin request -----
    Processing Record 368 of Set 8 | jalu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=jalu,ly
    --- request complete ---
    ----- begin request -----
    Processing Record 369 of Set 8 | merauke
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=merauke,id
    --- request complete ---
    ----- begin request -----
    Processing Record 370 of Set 8 | manvi
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=manvi,in
    --- request complete ---
    ----- begin request -----
    Processing Record 371 of Set 8 | tevaitoa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tevaitoa,pf
    --- request complete ---
    ----- begin request -----
    Processing Record 372 of Set 8 | bengkulu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bengkulu,id
    --- request complete ---
    ----- begin request -----
    Processing Record 373 of Set 8 | ahipara
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ahipara,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 374 of Set 8 | san pancrazio salentino
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=san+pancrazio+salentino,it
    --- request complete ---
    ----- begin request -----
    Processing Record 375 of Set 8 | paracuru
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=paracuru,br
    --- request complete ---
    ----- begin request -----
    Processing Record 376 of Set 8 | isla mujeres
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=isla+mujeres,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 377 of Set 8 | warwick
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=warwick,au
    --- request complete ---
    ----- begin request -----
    Processing Record 378 of Set 8 | grindavik
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=grindavik,is
    --- request complete ---
    ----- begin request -----
    Processing Record 379 of Set 8 | tapes
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tapes,br
    --- request complete ---
    ----- begin request -----
    Processing Record 380 of Set 8 | ostrovnoy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ostrovnoy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 381 of Set 8 | tezu
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tezu,in
    --- request complete ---
    ----- begin request -----
    Processing Record 382 of Set 8 | beringovskiy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=beringovskiy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 383 of Set 8 | bethel
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bethel,us
    --- request complete ---
    ----- begin request -----
    Processing Record 384 of Set 8 | akersberga
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=akersberga,se
    --- request complete ---
    ----- begin request -----
    Processing Record 385 of Set 8 | nishihara
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nishihara,jp
    --- request complete ---
    ----- begin request -----
    Processing Record 386 of Set 8 | bell ville
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bell+ville,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 387 of Set 8 | guerrero negro
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=guerrero+negro,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 388 of Set 8 | keflavik
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=keflavik,is
    --- request complete ---
    ----- begin request -----
    Processing Record 389 of Set 8 | amapa
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=amapa,br
    --- request complete ---
    ----- begin request -----
    Processing Record 390 of Set 8 | praya
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=praya,id
    --- request complete ---
    ----- begin request -----
    Processing Record 391 of Set 8 | bowen
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bowen,au
    --- request complete ---
    ----- begin request -----
    Processing Record 392 of Set 8 | bunia
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bunia,cd
    --- request complete ---
    ----- begin request -----
    Processing Record 393 of Set 8 | khasan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=khasan,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 394 of Set 8 | northam
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=northam,au
    --- request complete ---
    ----- begin request -----
    Processing Record 395 of Set 8 | muros
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=muros,es
    --- request complete ---
    ----- begin request -----
    Processing Record 396 of Set 8 | khowai
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=khowai,in
    --- request complete ---
    ----- begin request -----
    Processing Record 397 of Set 8 | khani
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=khani,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 398 of Set 8 | ulaangom
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ulaangom,mn
    --- request complete ---
    ----- begin request -----
    Processing Record 399 of Set 8 | mys shmidta
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mys+shmidta,ru
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 400 of Set 9 | brae
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=brae,gb
    --- request complete ---
    ----- begin request -----
    Processing Record 401 of Set 9 | bitanhuan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bitanhuan,ph
    --- request complete ---
    ----- begin request -----
    Processing Record 402 of Set 9 | blagoyevo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=blagoyevo,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 403 of Set 9 | jauharabad
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=jauharabad,pk
    --- request complete ---
    ----- begin request -----
    Processing Record 404 of Set 9 | ilulissat
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ilulissat,gl
    --- request complete ---
    ----- begin request -----
    Processing Record 405 of Set 9 | halalo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=halalo,wf
    --- request complete ---
    ----- begin request -----
    Processing Record 406 of Set 9 | bellevue
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bellevue,us
    --- request complete ---
    ----- begin request -----
    Processing Record 407 of Set 9 | bargal
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bargal,so
    --- request complete ---
    ----- begin request -----
    Processing Record 408 of Set 9 | thunder bay
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=thunder+bay,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 409 of Set 9 | ola
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ola,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 410 of Set 9 | canico
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=canico,pt
    --- request complete ---
    ----- begin request -----
    Processing Record 411 of Set 9 | panacan
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=panacan,ph
    --- request complete ---
    ----- begin request -----
    Processing Record 412 of Set 9 | stornoway
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=stornoway,gb
    --- request complete ---
    ----- begin request -----
    Processing Record 413 of Set 9 | ambilobe
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ambilobe,mg
    --- request complete ---
    ----- begin request -----
    Processing Record 414 of Set 9 | greeley
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=greeley,us
    --- request complete ---
    ----- begin request -----
    Processing Record 415 of Set 9 | phan rang
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=phan+rang,vn
    --- request complete ---
    ----- begin request -----
    Processing Record 416 of Set 9 | jambi
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=jambi,id
    --- request complete ---
    ----- begin request -----
    Processing Record 417 of Set 9 | iqaluit
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=iqaluit,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 418 of Set 9 | mzimba
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mzimba,mw
    --- request complete ---
    ----- begin request -----
    Processing Record 419 of Set 9 | kismayo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kismayo,so
    --- request complete ---
    ----- begin request -----
    Processing Record 420 of Set 9 | porto novo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=porto+novo,cv
    --- request complete ---
    ----- begin request -----
    Processing Record 421 of Set 9 | shaunavon
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=shaunavon,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 422 of Set 9 | blyth
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=blyth,gb
    --- request complete ---
    ----- begin request -----
    Processing Record 423 of Set 9 | carlisle
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=carlisle,us
    --- request complete ---
    ----- begin request -----
    Processing Record 424 of Set 9 | bourg-en-bresse
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bourg-en-bresse,fr
    --- request complete ---
    ----- begin request -----
    Processing Record 425 of Set 9 | melilla
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=melilla,es
    --- request complete ---
    ----- begin request -----
    Processing Record 426 of Set 9 | artyom
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=artyom,az
    --- request complete ---
    ----- begin request -----
    Processing Record 427 of Set 9 | nizhniy kuranakh
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=nizhniy+kuranakh,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 428 of Set 9 | general roca
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=general+roca,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 429 of Set 9 | solnechnyy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=solnechnyy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 430 of Set 9 | payo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=payo,ph
    --- request complete ---
    ----- begin request -----
    Processing Record 431 of Set 9 | baykalovo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=baykalovo,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 432 of Set 9 | libourne
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=libourne,fr
    --- request complete ---
    ----- begin request -----
    Processing Record 433 of Set 9 | andra
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=andra,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 434 of Set 9 | kaitangata
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kaitangata,nz
    --- request complete ---
    ----- begin request -----
    Processing Record 435 of Set 9 | weligama
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=weligama,lk
    --- request complete ---
    ----- begin request -----
    Processing Record 436 of Set 9 | clearlake
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=clearlake,us
    --- request complete ---
    ----- begin request -----
    Processing Record 437 of Set 9 | bereda
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=bereda,so
    --- request complete ---
    ----- begin request -----
    Processing Record 438 of Set 9 | aklavik
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=aklavik,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 439 of Set 9 | yar-sale
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=yar-sale,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 440 of Set 9 | algiers
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=algiers,dz
    --- request complete ---
    ----- begin request -----
    Processing Record 441 of Set 9 | winsum
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=winsum,nl
    --- request complete ---
    ----- begin request -----
    Processing Record 442 of Set 9 | kerki
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=kerki,tm
    --- request complete ---
    ----- begin request -----
    Processing Record 443 of Set 9 | strezhevoy
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=strezhevoy,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 444 of Set 9 | adrar
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=adrar,dz
    --- request complete ---
    ----- begin request -----
    Processing Record 445 of Set 9 | newport
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=newport,us
    --- request complete ---
    ----- begin request -----
    Processing Record 446 of Set 9 | mocuba
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=mocuba,mz
    --- request complete ---
    ----- begin request -----
    Processing Record 447 of Set 9 | shelburne
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=shelburne,ca
    --- request complete ---
    ----- begin request -----
    Processing Record 448 of Set 9 | safford
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=safford,us
    --- request complete ---
    ----- begin request -----
    Processing Record 449 of Set 9 | ust-omchug
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ust-omchug,ru
    --- request complete ---
    ~~~ sleep ~~~
    
    ----- begin request -----
    Processing Record 450 of Set 10 | port blair
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=port+blair,in
    --- request complete ---
    ----- begin request -----
    Processing Record 451 of Set 10 | joshimath
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=joshimath,in
    --- request complete ---
    ----- begin request -----
    Processing Record 452 of Set 10 | ocean city
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ocean+city,us
    --- request complete ---
    ----- begin request -----
    Processing Record 453 of Set 10 | emba
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=emba,kz
    --- request complete ---
    ----- begin request -----
    Processing Record 454 of Set 10 | guozhen
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=guozhen,cn
    --- request complete ---
    ----- begin request -----
    Processing Record 455 of Set 10 | sataua
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=sataua,ws
    --- request complete ---
    ----- begin request -----
    Processing Record 456 of Set 10 | felipe carrillo puerto
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=felipe+carrillo+puerto,mx
    --- request complete ---
    ----- begin request -----
    Processing Record 457 of Set 10 | varadero
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=varadero,cu
    --- request complete ---
    ----- begin request -----
    Processing Record 458 of Set 10 | namatanai
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=namatanai,pg
    --- request complete ---
    ----- begin request -----
    Processing Record 459 of Set 10 | coihaique
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=coihaique,cl
    --- request complete ---
    ----- begin request -----
    Processing Record 460 of Set 10 | zhigalovo
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=zhigalovo,ru
    --- request complete ---
    ----- begin request -----
    Processing Record 461 of Set 10 | tucuman
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tucuman,ar
    --- request complete ---
    ----- begin request -----
    Processing Record 462 of Set 10 | charleston
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=charleston,us
    --- request complete ---
    ----- begin request -----
    Processing Record 463 of Set 10 | ingham
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ingham,au
    --- request complete ---
    ----- begin request -----
    Processing Record 464 of Set 10 | tonstad
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=tonstad,no
    --- request complete ---
    ----- begin request -----
    Processing Record 465 of Set 10 | ndioum
    http://api.openweathermap.org/data/2.5/weather?appid=9ab33afcfca2ec3d291dbc05ddc722ca&units=imperial&q=ndioum,sn
    --- request complete ---



```python
# remove rows without data 
summary_df = cities_df.loc[cities_df["Temperature"] != "FAIL", :]
print(summary_df.count())
summary_df.head()
```

    Latitude       396
    Longitude      396
    City           396
    Country        396
    Temperature    396
    Humidity       396
    Clouds         396
    Wind           396
    dtype: int64





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>City</th>
      <th>Country</th>
      <th>Temperature</th>
      <th>Humidity</th>
      <th>Clouds</th>
      <th>Wind</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.88</td>
      <td>-12.32</td>
      <td>buchanan</td>
      <td>lr</td>
      <td>77</td>
      <td>88</td>
      <td>75</td>
      <td>6.31</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-74.94</td>
      <td>-87.39</td>
      <td>punta arenas</td>
      <td>cl</td>
      <td>53.6</td>
      <td>62</td>
      <td>40</td>
      <td>25.28</td>
    </tr>
    <tr>
      <th>2</th>
      <td>37.64</td>
      <td>-100.84</td>
      <td>garden city</td>
      <td>us</td>
      <td>48.65</td>
      <td>81</td>
      <td>90</td>
      <td>2.39</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-1.64</td>
      <td>96.33</td>
      <td>padang</td>
      <td>id</td>
      <td>79.57</td>
      <td>100</td>
      <td>44</td>
      <td>1.95</td>
    </tr>
    <tr>
      <th>4</th>
      <td>65.10</td>
      <td>-99.13</td>
      <td>thompson</td>
      <td>ca</td>
      <td>-5.81</td>
      <td>58</td>
      <td>75</td>
      <td>12.75</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Save to csv file
summary_df.to_csv("ow_data.csv")
```


```python
# create DFs for charts 

# * Temperature (F) vs. Latitude
temp_df = summary_df[["Latitude", "Temperature"]]

# * Humidity (%) vs. Latitude
humidity_df = summary_df[["Latitude", "Humidity"]]

# * Cloudiness (%) vs. Latitude
clouds_df = summary_df[["Latitude", "Clouds"]]

# * Wind Speed (mph) vs. Latitude
wind_df = summary_df[["Latitude", "Wind"]]
```


```python
# Chart 1: Temperature (F) vs. Latitude

plt.scatter(temp_df["Latitude"],temp_df["Temperature"], marker="o", facecolors="red", edgecolors="black")
plt.title("City Latitude v. Temperature (%)")
plt.xlabel("Latitude") 
plt.ylabel("Temperature (F)") 
plt.ylim((-120, 120))
plt.axhline(y=32, linestyle='--', linewidth = 1, color = "black", alpha=.50)
plt.axvline(x=0, linestyle = "--", linewidth = 1, color = "black", alpha = .50)
plt.savefig("Lat_vs_Temp.png")
# plt.grid(alpha=.50)
plt.show()
```


![png](output_10_0.png)



```python
# Chart 2: Humidity (%) vs. Latitude

plt.scatter(humidity_df["Latitude"], humidity_df["Humidity"], marker="o", facecolors="red", edgecolors="black")
plt.title("City Latitude v. Humidity (%)")
plt.xlabel("Latitude")  
plt.ylabel("Humidity %")
plt.ylim((-100, 120))
plt.xlim((-80, 100))
plt.axhline(y=0, linestyle='--', linewidth = 1, color = "black", alpha=.50)
plt.axvline(x=0, linestyle = "--", linewidth = 1, color = "black", alpha = .50)
plt.savefig("Lat_vs_Hum.png")
plt.grid=True
plt.show()
```


![png](output_11_0.png)



```python
# Chart 3: Cloudiness (%) vs. Latitude

plt.scatter(clouds_df["Latitude"],clouds_df["Clouds"], marker="o", facecolors="red", edgecolors="black")
plt.title("City Latitude v. Cloudiness %")
plt.xlabel("Latitude") 
plt.ylabel("Cloudiness (%)") 
plt.ylim((-10, 120))
plt.axhline(y=0, linestyle='--', linewidth = 1, color = "black", alpha=.50)
plt.axvline(x=0, linestyle = "--", linewidth = 1, color = "black", alpha = .50)
plt.savefig("Lat_vs_Cloud.png")
# plt.grid(alpha=.50)
plt.show()

```


![png](output_12_0.png)



```python
# Chart 4: Wind Speed (mph) vs. Latitude

plt.scatter(wind_df["Latitude"],wind_df["Wind"], marker="o", facecolors="red", edgecolors="black")
plt.title("City Latitude v. Wind Speed (mph)")
plt.xlabel("Latitude") 
plt.ylabel("Wind Speed (mph)") 
plt.ylim((-10, 60))
plt.axhline(y=0, linestyle='--', linewidth = 1, color = "black", alpha=.50)
plt.axvline(x=0, linestyle = "--", linewidth = 1, color = "black", alpha = .50)
plt.savefig("Lat_vs_WindSpeed.png")
# plt.grid(alpha=.50)
plt.show()
```


![png](output_13_0.png)



```python

```
