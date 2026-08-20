# Lab 7 Manual Data Pipeline

The instructions is based on MacOS Tahoe 26.5.2., Linux/WSL2 and MS Windows 11 Education with Python 3.14.6 installed.

# 1. Scenario

You are building a system to monitor cars entering and exiting five car parks. The data you are capturing includes the car plate number, the time of entry, the time of exit, and the car park. Over the past two months, there have been overcrowding issues at the car parks. You are required to prepare the data for data scientists and analysts to investigate the issue.

We begin this practical exercise with a common starting point: reading in data and exploring it using Microsoft Excel or other visualization or dashboarding tools.

# 2. Prepare the Python Environment


1. Open a terminal or Windows PowerShell (Run as Administrator) or Windows Subsystem for Linux (WSL) and run the following commands to keep all your Python code in a Python environment:

    ```bash
    mkdir -p ~/Documents/projects/ee3801/data

    cd ~/Documents/projects/ee3801

    # If you do not have python installed, download python3.14.6 from https://www.python.org/downloads/. Then execute the command below to create a Python environment for ee3801.
    python3 -m venv venv_ee3801

    # In MacOS or Linux/WSL2in , execute the command below to create a Python environment for ee3801.
    source venv_ee3801/bin/activate
    python -V
    python -m pip install ipykernel
    ```
    ```powershell
    # In Windows PowerShell (Run as Administrator), execute the command below to create a Python environment for ee3801.
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
    python3 -m venv venv_ee3801
    .\venv_ee3801\Scripts\Activate.ps1
    python -V
    python -m pip install ipykernel
    ```

2. Install Visual Studio Code in the local machine. Open Visual Studio Code and add the folder to your workspace: ~/Documents/projects/ee3801. In Visual Studio Code, `File` > `Add Folder to Workspace...`.

3. Right-click the added folder named ```ee3801``` and create a new Jupyter Notebook file named ```manual_data_pipeline.ipynb```.

4. Select ```Kernel``` > ```Python Environments...``` > ```venv_ee3801``` > `+ Code`\
\
For Windows users, 
- Select Install/Enable suggessted extensions 
- then select your ```Python interpreter``` first by choosing ```Kernel``` > ```Python: Select Interpreter``` > ```~/Documents/projects/ee3801/venv_ee3801/bin/python``` (For Windows, `~/Documents/projects/ee3801/venv_ee3801/Scripts/python.exe`). 
- then `Select Kernel` > `Python Environments...` > `venv_ee3801`.
- Add a new Code cell `+ Code`.

    ```python
    # Install Python libraries

    !python -m pip install faker
    !python -m pip install pandas
    !python -m pip install matplotlib
    !python -m pip install --upgrade pip
    ```
    
    When prompted ```Running cells with 'venv_ee3801 (Python 3.x.x)' requires the ipykernel package.```, click Install.
    
    
    ```python
    # Import libraries

    from faker import Faker
    import json
    from datetime import datetime, timedelta
    import random
    import pandas as pd
    import matplotlib.pyplot as plt

    # Set the date format
    date_format = "%d/%m/%Y %H:%M:%S"

    import os
    home_directory = os.path.expanduser("~")
    os.chdir(home_directory + '/Documents/projects/ee3801')
    ```

# 3. Generate Data

1. Copy and paste the following code into your notebook and run it.

    ```python
    # Faker is a Python library that generates fake data.
    # Here, we want to generate fake data to simulate cars entering the car park.
    fake = Faker()
    fake.license_plate()
    ```

2. Define the CarPark class and declare a function that generates cars entering the car park over the past 3 months, between 9 a.m. and 8 p.m., at one-minute and one-second intervals.


    ```python
    # Define the CarPark class
    class CarPark:
        def __init__(self, Plate, LocationID, Entry_DateTime, Exit_DateTime, Parking_Charges):
            self.Plate = Plate
            self.LocationID = LocationID
            self.Entry_DateTime = Entry_DateTime
            self.Exit_DateTime = Exit_DateTime
            self.Parking_Charges = Parking_Charges

    def generate_past_datetime(days):
        """
        Generates a random datetime object within the last 2-3 months,
        between 9 AM and 8 PM, including minute and second precision.

        Returns:
            datetime object.
        """
        now = datetime.now()

        # Define the time range for the past n months
        end_date = now
        start_date = now - timedelta(days=days)

        time_delta_total_seconds = int((end_date - start_date).total_seconds())

        # Generate a random number of seconds within the 2-3 month range
        random_seconds_past = random.randint(0, time_delta_total_seconds)
        random_date_base = start_date + timedelta(seconds=random_seconds_past)

        # Generate random time components between 9 AM and 8 PM
        random_hour = random.randint(9, 20)  # 9 to 20 (inclusive for 8 PM)
        random_minute = random.randint(0, 59)
        random_second = random.randint(0, 59)

        # Combine date and time components
        generated_datetime = random_date_base.replace(
            hour=random_hour,
            minute=random_minute,
            second=random_second,
            microsecond=0  # Set microseconds to 0 for minute/second precision
        )

        return generated_datetime

    # Generate cars entering the car park over the past 2-3 months, between 9 a.m. and 8 p.m., at one-minute and one-second intervals.
    def createPastCarEntry():
        car = CarPark(
            Plate=fake.license_plate(),
            LocationID="Park" + str(random.randint(0, 5)),
            Entry_DateTime=generate_past_datetime(90).strftime(date_format),  # Approximately 3 months ago
            Exit_DateTime="",
            Parking_Charges=float(0)
        )

        # Return data in JSON format
        return json.dumps(car.__dict__)
    ```

3. Generate 1000 cars.


    ```python
    # Generate 1000 cars
    carpark_system = []
    for i in range(1000):
        thiscar_dict = eval(createPastCarEntry())
        carpark_system.append(list(thiscar_dict.values()))
    
    carpark_system_df = pd.DataFrame(carpark_system, columns=list(eval(createPastCarEntry()).keys()))
    print(len(carpark_system_df))
    carpark_system_df.head()
        
    ```

4. Save the data to a CSV file for analysis.

    ```python
    # Export to CSV for further analysis
    carpark_system_df.to_csv("data/carpark_system.csv", encoding='utf-8-sig', index=False)
    ```

# 4. Python Visualisation

1. In your notebook, plot the data to show the total number of cars in each location.


    ```python
    carpark_system_df.groupby(['LocationID'])['LocationID'].count().plot(title="Total number of cars in each location")
    index_carpark_system_df = carpark_system_df.set_index('Entry_DateTime')
    index_carpark_system_df.index = pd.to_datetime(index_carpark_system_df.index, errors='coerce')
    index_carpark_system_df.head()
    ```

2. Plot a chart to show which time of day has more cars entering the car park.


    ```python
    df = pd.DataFrame({'hour':index_carpark_system_df.index.hour, 
        'plate': index_carpark_system_df['Plate']}).groupby('hour')['plate'].count()
    
    ax = df.plot(title="Total number of cars in each hour of the day")
    
    # show data labels
    for i, txt in enumerate(df):
        ax.annotate(txt, (df.index[i], df.values[i]), textcoords="offset points", xytext=(0,5), ha='center')
    plt.show()
    
    ```

3. Plot a chart to show how many cars are in each car park at every hour. Are the car parks overcrowded?


    ```python
    pd.DataFrame({'hour':index_carpark_system_df.index.hour, 
        'plate': index_carpark_system_df['Plate'], 
        'carpark': index_carpark_system_df['LocationID']})\
        .groupby(['hour','carpark'])['plate'].count()
    ```
    
    
    ```python
    df = pd.DataFrame({'hour':index_carpark_system_df.index.hour, 
        'plate': index_carpark_system_df['Plate'], 
        'carpark': index_carpark_system_df['LocationID']})\
        .groupby(['hour','carpark'])['plate'].count()
    df.unstack().plot(title="Total number of cars in each hour of the day for each carpark")
    ```

# 5. Microsoft Excel (Pivot Tables)

1. Open the carpark_system.csv file in Microsoft Excel. Select Entry_DateTime and choose the Number Format as Long Date.

    <img src="image/week7_image1.png" width="50%">

2. Click Insert > Pivot Chart > OK.

    <img src="image/week7_image2.png" width="50%">
    <img src="image/week7_image3.png" width="50%">

3. A new worksheet will open.

    <img src="image/week7_image4.png" width="50%">

4. Drag and drop the field name Entry_DateTime into Rows (Axis Categories), Plate into Values, LocationID into Columns (Legend Series). Note: If the date does not show Month or Day groupings, go to your system date settings and set to English (Singapore).

    <img src="image/week7_image5.png" width="50%">

5. Save your files as 'carpark_system.xlsx'.
<br>

# 6. Microsoft Power BI

The amount of data may increase over time, so we want to visualise the charts quickly whenever we refresh the data.

<b>For Windows users</b>

1. Go to powerbi.microsoft.com, download and install Microsoft Power BI.

2. Start MS Power BI.

<b>For Mac users</b>

1. Download the <a href="https://cloud.vmwarehorizon.com">VMware Horizon Client</a>.

2. Your computer will need to be connected to the NUS VPN.

3. Launch the VMware Horizon Client and create a connection to https://thinlab.nus.edu.sg.

4. Launch the virtual machine and click “Virtual Student Desktop”.

5. Log in using nusstu\<your student id> and your password.

6. In the virtual windows, start MS Power BI.

<b>Loading and visualising data</b>

1. Upload your generated carpark_system.csv file into OneDrive:

    "~/Library/CloudStorage/OneDrive-NationalUniversityofSingapore/ee3801/data/carpark_system.csv".
    

2. Right-click the three dots (...) and copy the link.

    <img src="image/week7_image6.png" width="50%">

3. Open MS Power BI Desktop.

    <img src="image/week7_image7.png" width="50%">

4. Select New > Report.

    <img src="image/week7_image8.png" width="50%">
    <img src="image/week7_image9.png" width="50%">

5. Select Get Data > Web. Paste the link you copied in step 2. Remove the question mark (?), the parameters, and the values at the end of the link. Click OK and then Load. An authentication prompt may appear; select Organisation > Sign in > Connect.

    <img src="image/week7_image10.png" width="50%">
    <img src="image/week7_image11.png" width="50%">
    <img src="image/week7_image34.png" width="50%">
    <img src="image/week7_image12.png" width="50%">

6. Select Transform Data.

    <img src="image/week7_image13.png" width="50%">

7. Change the Data Type of Entry_DateTime and Exit_DateTime to `Using Locale` > `Data Type`: `DateTime` > `Locale`: `English (Singapore)`.

    <img src="image/week7_image14.png" width="30%">
    <img src="image/week7_image15.png" width="50%"> <br>
    <img src="image/week7_image16.png" width="20%">

8. Save your MS Power BI file as `carpark_system.pbix` in ~/Documents/projects/ee3801/data/. Click on Apply. Click on Close and Apply.

    <img src="image/week7_image17.png" width="20%">

9. Drag and drop the Line chart into the workspace. Drag and drop Entry_DateTime into X-axis, Plate into Y-axis, LocationID into Legend.

    <img src="image/week7_image18.png" width="20%">

10. You can see the chart. Click the down arrow (V) beside X-axis > Entry_DateTime > Choose Date Hierarchy. Remove Year and Qtr.

    <img src="image/week7_image19.png" width="50%"> <br>
    <img src="image/week7_image20.png" width="20%">
    <img src="image/week7_image21.png" width="20%">
    <img src="image/week7_image22.png" width="20%">

11. New data is coming in. Use the code below to generate a new car entry for every hour from 9 a.m. to 8 p.m., at one-minute and one-second intervals.




    ```python
    def createNewCarEntry():
        car = CarPark(
            Plate= fake.license_plate(),
            LocationID="Park"+str(random.randint(0, 5)),
            Entry_DateTime=generate_past_datetime(1).strftime(date_format), # Approximately 1 day ago
            Exit_DateTime="",
            Parking_Charges=float(0)
        )
    
        return json.dumps(car.__dict__)
    ```

12. Generate 100 cars.


    ```python
    # Read the latest file
    carpark_system_df = pd.read_csv("data/carpark_system.csv", encoding='utf-8-sig', index_col=False)
    # carpark_system_df.drop(columns=['Unnamed: 0'], inplace=True)
    print(len(carpark_system_df))
    print(carpark_system_df.head())
    
    # Generate more cars, append them to the list, and save the CSV
    carpark_system = []
    for i in range(100):
        thiscar_dict = eval(createNewCarEntry())
        carpark_system.append(list(thiscar_dict.values()))
    
    new_carpark_system_df = pd.DataFrame(carpark_system, columns=list(eval(createNewCarEntry()).keys()))
    print(len(new_carpark_system_df))
    print(new_carpark_system_df.head())
    
    updated_carpark_system_df = pd.concat([carpark_system_df,new_carpark_system_df], axis=0)
    print(len(updated_carpark_system_df))
    print(updated_carpark_system_df.head())
    
    # Export to CSV for further analysis
    updated_carpark_system_df.to_csv("data/carpark_system.csv", encoding='utf-8-sig', index=False)
    # export to your OneDrive too for on-demand refresh (replace your file in OneDrive)
    updated_carpark_system_df.to_csv("~/Library/CloudStorage/OneDrive-NationalUniversityofSingapore/ee3801/data/carpark_system.csv", encoding='utf-8-sig', index=False)
        
    ```

13. The code above should replace the carpark_system.csv file in OneDrive. However, if it fails, you can still manually copy and replace your generated carpark_system.csv file in OneDrive.

    <img src="image/week7_image23.png" width="50%">

14. In MS Power BI, click the Refresh button.

    <img src="image/week7_image24.png" width="50%">

15. Drag and drop the Card visual into the workspace and count the number of plates.

    <img src="image/week7_image26.png" width="50%">

<!-- ## Microsoft Power Automate

As the data refreshes only when you click the Refresh button, if you have multiple users, how do you share this data with them and how do you ensure that they see up-to-date data?

1. Go to <a href="https://make.powerautomate.com">Microsoft Power Automate</a>
2. Sign in > My Flows > New flow.

<img src="week7_image27.png" width="50%">

3. There are a number of flows. We want to trigger a MS Power BI refresh whenever the MS Excel file is modified. Choose Automated Cloud Flow.

<img src="week7_image29.png" width="20%">
<img src="week7_image28.png" width="30%">

4. Enter Flow name "carpark_system" and Choose your flow's trigger "When an item is created or modified" Sharepoint and Create.

<img src="week7_image30.png" width="50%">

5. Click on the created trigger. Copy and paste the link from OneDrive into the Site Address and remove the question mark (?) and parameter with values from the link.

<img src="week7_image31.png" width="50%">
<img src="week7_image32.png" width="50%"> -->

# Conclusion

In this lab, you learned about:
- generating data manually,
- visualising data using Python,
- using Microsoft Excel, and
- creating a simple data pipeline in Microsoft Power BI.

This is the simplest form of a data pipeline and is often the first step in an organisation with a basic data maturity level. However, it is an essential step towards building a more automated and streamlined data pipeline.

<b>Questions to ponder</b>
- With Microsoft SharePoint through OneDrive and Microsoft Power BI, we are able to see the latest data with one click. How can we remove the need to click and still refresh the data?
- What is the time taken to refresh the data?
- As the data refreshes only when you click on the refresh button, if you have multiple users how do you share this data with them and how do you ensure that they will access up-to-date data?
- What if your company does not subscribe to Microsoft Power Platform?

# Submissions next Wed 9pm (8 Oct)
Submit your 
- .ipynb file as a PDF (Save your .ipynb file as an HTML file, open it in a browser, and print it as a PDF. )
- Excel file
- .pbix file. 

Include the following in your submission:

    Answer the Questions to ponder

~ The End ~
