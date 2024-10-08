# Lab 7 Manual Data Pipeline¶

# 1. Scenario¶

You are building a system to monitor Cars entering and exiting carparks. The data that you are capturing is car plate number, time of entry, time of exit and the carpark. In the past two months there seemed to be a overcrowding issues at the carparks. You are to prepare the data for the Data Scientists and Analysts to investigate the issue.

We start this practical exercises with a common starting point, reading in data and explore the data using Microsoft Excel or other visualisations or dashboarding tools.

# 2. Prepare Python Environment¶

1. Open a terminal to run these commands to contain all your python codes in the python environment

    `cd /Users/<username>/projects/ee3801`

    `python3 -m venv venv_ee3801`

    `source venv_ee3801/bin/activate`

    `pip3 install faker`
2. Create a new jupyter notebook file "manual\_data\_pipeline.ipynb".
3. Import the python libraries

In [ ]:

```
# import libraries

from faker import Faker
import json
from datetime import datetime, timedelta
import random
import pandas as pd
import matplotlib.pyplot as plt

import os
os.chdir('/Users/<username>/projects/ee3801')

# set the date format
date_format = "%d/%m/%Y %H:%M:%S"
```

# 3. Generate data¶

1. Copy and paste these codes into your notebook and run.

In [ ]:

```
# faker is a python library that generates fake data
# here we want to generate fake data to simulate cars entering into the carpark
fake=Faker()
fake.license_plate()
```

1. Define the CarPark class and declare function that generate cars entering the carpark for the past 2-3 months between 9 am to 8 pm for every minute and second.

In [ ]:

```
# define the CarPark class
class CarPark:
    def __init__(self, Plate, LocationID, Entry_DateTime, Exit_DateTime, Parking_Charges):
        self.Plate = Plate
        self.LocationID = LocationID
        self.Entry_DateTime = Entry_DateTime
        self.Exit_DateTime = Exit_DateTime
        self.Parking_Charges = Parking_Charges

# generate cars entering the carpark for the past 2-3 months between 9 am to 8 pm for every minute and second
def createPastCarEntry():
    days  = random.randint(1, 60) # 2-3 months
    hours = random.randint(9, 20)
    minutes = random.randint(1, 60)
    seconds = random.randint(1, 60)

    ts = datetime.now() - timedelta(days=days, hours=hours, minutes=minutes, seconds=seconds)

    car = CarPark(
        Plate= fake.license_plate(),
        LocationID="Park"+str(random.randint(0, 5)),
        Entry_DateTime=ts.strftime(date_format), 
        Exit_DateTime="",
        Parking_Charges=float(0)
    )

    # return a json format
    return json.dumps(car.__dict__) 
```

1. Generate 1000 cars.

In [ ]:

```
# Generate 1000 cars
carpark_system = []
for i in range(1000):
    thiscar_dict = eval(createPastCarEntry())
    carpark_system.append(list(thiscar_dict.values()))

carpark_system_df = pd.DataFrame(carpark_system, columns=list(eval(createPastCarEntry()).keys()))
print(len(carpark_system_df))
carpark_system_df.head()
    
```

1. Save to csv file for analysis.

In [ ]:

```
# export to csv for further analysis
carpark_system_df.to_csv("data/carpark_system.csv", encoding='utf-8-sig')
```

# 4. Python Visualisation¶

1. In your notebook, plot the data to show the total number of cars in each location.

In [ ]:

```
carpark_system_df.groupby(['LocationID'])['LocationID'].count().plot()
index_carpark_system_df = carpark_system_df.set_index('Entry_DateTime')
index_carpark_system_df.index = pd.to_datetime(index_carpark_system_df.index, errors='coerce')
```

1. Plot a chart to show which time of the day has more cars entering the carpark.

1. Plot a chart to show how many cars is in the carpark every time. Is the carparks overcrowded?

# 5. Microsoft Excel (Pivot Tables)¶

1. Open the carpark\_system.csv file in Microsoft Excel.

![No description has been provided for this image](image/week7_image1.png)
1. Click on Insert &gt; Pivot Chart &gt; Ok

![No description has been provided for this image](image/week7_image2.png)
![No description has been provided for this image](image/week7_image3.png)
1. A new worksheet will be opened.

![No description has been provided for this image](image/week7_image4.png)
1. Drag and drop the field name Entry\_DateTime into Rows, Plate into Values, LocationID into Columns.

![No description has been provided for this image](image/week7_image5.png)

# 6. Microsoft Power BI¶

The amount of data might increase as the days goes by, we want to quickly visualise the charts everytime we refresh the data.

**For Windows users,**

1. Go to powerbi.microsoft.com. Download and install Microsoft Power BI.
2. Start MS Power BI.

**For Mac users, **

1. [Download VMWare Horizon Client](https://customerconnect.vmware.com/en/downloads/info/slug/desktop_end_user_computing/vmware_horizon_clients/horizon_8)
2. [Launch the VMWAre Horizon Client and create connection to https://thinlab.nus.edu.sg](https://thinlab.nus.edu.sg)
3. Launch the virtual machine, click on “Click Here” or “Or Click Here”.
4. Login using nusstu&lt;your student id&gt; and password
5. In the virtual windows, start MS Power BI.
6. Your computer will need to be in NUS VPN.

**Loading and visualising data**

1. Upload your generated carpark\_system.csv file into OneDrive.
2. Right click on the triple dots (...) and copy the link.

![No description has been provided for this image](image/week7_image6.png)
1. Open MS Power BI Desktop.

![No description has been provided for this image](image/week7_image7.png)
1. Select New &gt; Report.

![No description has been provided for this image](image/week7_image8.png)
![No description has been provided for this image](image/week7_image9.png)
1. Select Get Data &gt; Web. Paste the link that you have copied in step 2. Remove the question mark (?), parameters and values at the end of the link. Click Ok and Load.

![No description has been provided for this image](image/week7_image10.png)
![No description has been provided for this image](image/week7_image11.png)
![No description has been provided for this image](image/week7_image12.png)
1. Select Transform Data.

![No description has been provided for this image](image/week7_image13.png)
1. Change the Data Type of Entry\_DateTime and Exit\_DateTime to Locale &gt; DateTime &gt; Locale &gt; English (Singapore).

![No description has been provided for this image](image/week7_image14.png)
![No description has been provided for this image](image/week7_image15.png) 

![No description has been provided for this image](image/week7_image16.png)
1. Save your MS Power BI file as carpark\_system.pbix. Click on Apply. Click on Close and Apply.

![No description has been provided for this image](image/week7_image17.png)
1. Drag and drop the Line chart into the workspace. Drag and drop Entry\_DateTime into X-axis, Plate into Y-axis, LocationID into Legend.

![No description has been provided for this image](image/week7_image18.png)
1. You can see the chart. Click on the arrow down (V) beside X-axis Entry\_DateTime &gt; Choose Date Hierarchy. Remove Year and Qtr.

![No description has been provided for this image](image/week7_image19.png) 
 ![No description has been provided for this image](image/week7_image20.png) ![No description has been provided for this image](image/week7_image21.png) ![No description has been provided for this image](image/week7_image22.png)

1. New data is coming in... Use the codes below to generate new car entry for every hour from 9 am to 8 pm for every minute and second.

In [ ]:

```
def createNewCarEntry():
    # days  = random.randint(1, 60) # 2-3 months
    hours = random.randint(9, 20)
    minutes = random.randint(1, 60)
    seconds = random.randint(1, 60)

    ts = datetime.now() - timedelta(minutes=minutes, seconds=seconds)

    car = CarPark(
        Plate= fake.license_plate(),
        LocationID="Park"+str(random.randint(0, 5)),
        Entry_DateTime=ts.strftime(date_format), #.isoformat(),
        Exit_DateTime="",
        Parking_Charges=float(0)
    )

    return json.dumps(car.__dict__)
```

1. Generate 100 cars.

In [ ]:

```
# read latest file
carpark_system_df = pd.read_csv("lab/carpark_system.csv", encoding='utf-8-sig')
carpark_system_df.drop(columns=['Unnamed: 0'], inplace=True)
print(len(carpark_system_df))
carpark_system_df.head()

# Generate more cars, append to list and save csv
carpark_system = []
for i in range(100):
    thiscar_dict = eval(createNewCarEntry())
    carpark_system.append(list(thiscar_dict.values()))

new_carpark_system_df = pd.DataFrame(carpark_system, columns=list(eval(createNewCarEntry()).keys()))
print(len(new_carpark_system_df))
print(new_carpark_system_df.head())

updated_carpark_system_df = pd.concat([carpark_system_df,new_carpark_system_df], axis=0)
print(len(updated_carpark_system_df))
updated_carpark_system_df.head()

# export to csv for further analysis
updated_carpark_system_df.to_csv("data/carpark_system.csv", encoding='utf-8-sig')
# export to your OneDrive too for on-demand refresh
updated_carpark_system_df.to_csv("/Users/peihuacher/Library/CloudStorage/OneDrive-NationalUniversityofSingapore/Courses/CDE_EDIC/ee3801/lab/carpark_system.csv", encoding='utf-8-sig')
    
```

1. Upload and replace your generated carpark\_system.csv file into OneDrive.

![No description has been provided for this image](image/week7_image23.png)
1. In MS Power BI, click on the refresh button.

![No description has been provided for this image](image/week7_image24.png)
1. Drag and drop the Card into the workspace and count the number of Plates.

![No description has been provided for this image](image/week7_image26.png)

# Conclusion¶

In this lab, you learned the

- manual task of generating data,
- visualising using python,
- Microsoft Excel and
- created a simple data pipeline in Microsoft Power BI.

This is the most simple form of data pipeline that is often the first steps in an organisation with a basic data maturity level. However this is an essential step towards building a more automated streamlined data pipeline.

**Questions to ponder**

- With MS Sharepoint through OneDrive and MS Power BI we are able to see the latest data by one click. How do we remove the step to click and yet refresh the data?
- What is the time taken to refresh the data?
- As the data refreshes only when you click on the refresh button, if you have multiple users how do you share this data with them and how do you ensure that they see up to date data?
- What if your company does not subscribe to Microsoft Power Platform?

# Submissions next Wed 9pm (9 Oct 2024)¶

Submit your ipynb as a pdf, excel file and pbi file. Save your ipynb as a html file, open in browser and print as a pdf. Include in your submission:

```
Answer the Questions to ponder
```

~ The End ~