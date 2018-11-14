# Bike_Share_Analysis
This is one of the Python projects I have conducted as a requirement for a Data Analyst nano degree @ Udacity.
I tried to demonstrate how data could be used to analyse consuer behavior and guide business decisions. 

## Introduction

Over the past decade, bicycle-sharing systems have been growing in number and popularity in cities across the world. Bicycle-sharing systems allow users to rent bicycles for short trips, typically 30 minutes or less. Thanks to the rise in information technologies, it is easy for a user of the system to access a dock within the system to unlock or return bicycles. These technologies also provide a wealth of data that can be used to explore how these bike-sharing systems are used.

In this project, I was expected to perform an exploratory analysis on data provided by [Motivate](https://www.motivateco.com/), a bike-share system provider for many major cities in the United States. The project essence is to compare the system usage between three large cities: New York City, Chicago, and Washington, DC. It is also expected to analyse if there are any differences within each system for those users that are registered, regular users and those users that are short-term, casual users.

## Language used: Python 3

## Code composition
extracting data and visualization

## General notes about the data:

You will generate new data files with five values of interest for each trip: trip duration, starting month, starting hour, day of the week, and user type. Each of these may require additional wrangling depending on the city:

- **Duration**: This has been given to us in seconds (New York, Chicago) or milliseconds (Washington). A more natural unit of analysis will be if all the trip durations are given in terms of minutes.
- **Month**, **Hour**, **Day of Week**: Ridership volume is likely to change based on the season, time of day, and whether it is a weekday or weekend. Use the start time of the trip to obtain these values. The New York City data includes the seconds in their timestamps, while Washington and Chicago do not. The [`datetime`](https://docs.python.org/3/library/datetime.html) package will be very useful here to make the needed conversions.
- **User Type**: It is possible that users who are subscribed to a bike-share system will have different patterns of use compared to users who only have temporary passes. Washington divides its users into two types: 'Registered' for users with annual, monthly, and other longer-term subscriptions, and 'Casual', for users with 24-hour, 3-day, and other short-term passes. The New York and Chicago data uses 'Subscriber' and 'Customer' for these groups, respectively. For consistency, you will convert the Washington labels to match the other two.

## Code and explanation

### A function for returning trip duration in minutes
This function takes as input a dictionary containing info about a single trip (datum) and its origin city (city) and returns the trip duration in units of minutes.


```
def duration_in_mins(datum, city):
    if city == 'NYC':
        duration = float(datum)/60
    elif city == 'Chicago':
        duration = float(datum)/60
    elif city == 'Washington':
        duration = (float(datum)/1000)/60
            
    return duration
  ```
 
### A function for returning month, hour and day of the week
Takes as input a dictionary containing info about a single trip (datum) and its origin city (city) and returns the month, hour, and day of the week in which the trip was made.
 
 
 ```
from datetime import datetime
def time_of_trip(datum, city):
   if city == 'NYC':
        month = int(datetime.strptime( datum, "%m/%d/%Y %H:%M:%S" ).strftime('%m'))
        hour = int(datetime.strptime( datum, "%m/%d/%Y %H:%M:%S" ).strftime('%H'))
        day_of_week = datetime.strptime( datum, "%m/%d/%Y %H:%M:%S" ).strftime('%A')
       
    elif city == 'Chicago' or city == 'Washington':
        month = int(datetime.strptime( datum, "%m/%d/%Y %H:%M" ).strftime('%m'))
        hour = int(datetime.strptime( datum, "%m/%d/%Y %H:%M" ).strftime('%H'))
        day_of_week = datetime.strptime( datum, "%m/%d/%Y %H:%M" ).strftime('%A')
        
           
    return (month, hour, day_of_week)
 ```
 
### A function that returns type of user
 Takes as input a dictionary containing info about a single trip (datum) and its origin city (city) and returns the type of system user that made the trip.
 
 ```
def type_of_user(datum, city):
    if datum == 'Registered':
        user_type = 'Subscriber'
    elif datum == 'Casual':
        user_type = 'Customer'
    else:
        user_type = datum
    return user_type
```

### Isolating the needed info in a new file
This function takes full data from the specified input file and writes the condensed data to a specified output file. The city argument determines how the input file will be parsed.

```
import csv 
from datetime import datetime
from pprint import pprint

def condense_data(in_file, out_file, city):
    with open(out_file, 'w') as f_out, open(in_file, 'r') as f_in:
        # set up csv DictWriter object - writer requires column names for the
        # first row as the "fieldnames" argument
        out_colnames = ['duration', 'month', 'hour', 'day_of_week', 'user_type']        
        trip_writer = csv.DictWriter(f_out, fieldnames = out_colnames)
        trip_writer.writeheader()
        
        trip_reader = csv.DictReader(f_in)
    
        # collect data from and process each row
        for row in trip_reader:
            # set up a dictionary to hold the values for the cleaned and trimmed
            # data point
                        
            new_point = {}
      
            if city=='NYC':
                new_point['duration'] = duration_in_mins(row['tripduration'], 'NYC')
                new_point['month'] = time_of_trip(row['starttime'], 'NYC')[0]
                new_point['hour'] = time_of_trip(row['starttime'], 'NYC')[1]
                new_point['day_of_week'] = time_of_trip(row['starttime'], 'NYC')[2]
                new_point['user_type'] = type_of_user(row['usertype'], 'NYC')
            
            elif city=='Chicago':
                new_point['duration'] = duration_in_mins(row['tripduration'], 'Chicago')
                new_point['month'] = time_of_trip(row['starttime'], 'Chicago')[0]
                new_point['hour'] = time_of_trip(row['starttime'], 'Chicago')[1]
                new_point['day_of_week'] = time_of_trip(row['starttime'], 'Chicago')[2]
                new_point['user_type'] = type_of_user(row['usertype'], 'Chicago')
            
            elif city=='Washington':
                new_point['duration'] = duration_in_mins(row['Duration (ms)'], 'Washington')
                new_point['month'] = time_of_trip(row['Start date'], 'Washington')[0] 
                new_point['hour'] = time_of_trip(row['Start date'], 'Washington')[1]
                new_point['day_of_week'] = time_of_trip(row['Start date'], 'Washington')[2]
                new_point['user_type'] = type_of_user(row['Member Type'], 'Washington')
            
            else: 
                return 'unidentified city'
            
```

### A function for returing number of trips made by each user
This function reads in a file with trip data and reports the number of trips made by subscribers, customers, and total overall.

```
def number_of_trips(filename):
    with open(filename, 'r') as f_in:
        # set up csv reader object
        reader = csv.DictReader(f_in)
        
        # initialize count variables
        n_subscribers = 0
        n_customers = 0
        n_unidentified = 0
        
        # tally up ride types
        for row in reader:
            if row['user_type'] == 'Subscriber':
                n_subscribers += 1
            elif row['user_type'] == 'Customer' :
                n_customers += 1
            else:
                n_unidentified += 1
        
        # compute total number of rides
        # n_total will indicate the highst number of trips in general
        n_total = n_subscribers + n_customers + n_unidentified
        
        # Percentage of subscribers
        Per_subscriber = round((n_subscribers/n_total) * 100,2)
       
        # Round all percentages to only two decimals to improve appearance 
        # Percentage of customers
        Per_customers = round((n_customers/n_total) * 100,2)
        # Percentage of unidentified clients (missing data)
        # The question did not ask to print this portion out
        Per_unidentified = round((n_unidentified/n_total) * 100, 2)

        
        
        # return tallies as a tuple
        return(n_total, Per_subscriber, Per_customers)

```

### Defining a function for collecting needed information for further data analysis

```
def trip_data(city_file):
    # opening the data source file
    with open(city_file, 'r') as f_in:
        reader = csv.DictReader(f_in)
        total_durations = 0
        number_of_trips = 0
        long_trips = 0 #this is for the trips longer than 30 min
        
        for row in reader:
            number_of_trips += 1
            trip_duration = float(row['duration'])
            total_durations = total_durations + trip_duration
            
            if trip_duration > 30:
                long_trips +=1
        
        AVG_trip_length = round((total_durations/number_of_trips),3) #Average trip length rounded to three decimals
        Percentage_Long_Trips = round((long_trips/number_of_trips) * 100 , 2)
        
        return (AVG_trip_length, Percentage_Long_Trips)

```


### Defining trip user type

```
def trip_usertype(city_file):
    # following steps described above for opening the data source file
    with open(city_file, 'r') as f_in:
        reader = csv.DictReader(f_in)
        
        Subscriber_trips = 0
        Subscriber_total_durations = 0
        Customer_trips = 0
        Customer_total_durations = 0
        
               
        for row in reader:
            user_type = row['user_type']
            if user_type == "Subscriber":
                Subscriber_trips += 1
                Subscriber_trip_duration = float(row['duration'])
                Subscriber_total_durations = Subscriber_trip_duration + Subscriber_total_durations
        
            elif user_type == "Customer":
                Customer_trips += 1
                Customer_trip_duration = float(row['duration'])
                Customer_total_durations = Customer_trip_duration + Customer_total_durations
        
        AVG_trip_length_Subscriber = round((Subscriber_total_durations/Subscriber_trips),1) #Average trip length rounded to one decimal
        AVG_trip_length_Customer = round((Customer_total_durations/Customer_trips),1)
        
        return (AVG_trip_length_Subscriber, AVG_trip_length_Customer)

```

## Visualization

### Defining functions for rapid ploting using histogrames

```
import matplotlib.pyplot as plt

%matplotlib inline 

def Histo_duration_Customers_adjusted(city_file):
    with open(city_file, 'r') as f_in:
        reader = csv.DictReader(f_in)
        data = []
        for row in reader:
            y = row['user_type']
            x = round(float(row['duration']), 1)
            
            if y=='Customer' and x < 75:
                data.append(x)


    plt.hist(data, normed=True, bins=15)
    #plt.axis([0, 75, 0, 50000])
    plt.title('Distribution of Trip Durations for Customers')
    plt.xlabel('Duration (m)')
    plt.show()

```

```
import matplotlib.pyplot as plt
%matplotlib inline 

def Histo_duration_Subscribers_adjusted(city_file):
    with open(city_file, 'r') as f_in:
        reader = csv.DictReader(f_in)
        data = []
        for row in reader:
            y = row['user_type']
            x = round(float(row['duration']), 1)
            
            if y=='Subscriber' and x < 75:
                data.append(x)


    plt.hist(data, normed=True, bins=15)
    #plt.axis([0, 75, 0, 50000])
    plt.title('Distribution of Trip Durations for Subscribers')
    plt.xlabel('Duration (m)')
    plt.show()

```

using these functions

```
cities = {'Washington':'./data/Washington-2016-Summary.csv', 
          'Chicago':'./data/Chicago-2016-Summary.csv',
          'NYC':'./data/NYC-2016-Summary.csv'}

for city in cities:
    city_file = cities[city] 
    print("In", city)
    print (Histo_duration_Customers_adjusted(city_file))
    print (Histo_duration_Subscribers_adjusted(city_file))
```

## Visualizing consumer behavior

### This function will produce pie charts that summarize when customers use the service as distribued over day hours during working days

```
#The disctribution in day use among Subscribers and Customers as a pie chart
import csv
import matplotlib.pyplot as plt


def Daily_use_workdays(filename):
    # this function will generate pie charts for the use of the service in week days
    
    city = filename.split('-')[0].split('/')[-1]
    
    with open(city_file, 'r') as f_in:
        reader = csv.DictReader(f_in)
        
        Hour_of_use_workdays_Subscriber = []
        Hour_of_use_workdays_Customer = []
        
        if city == 'NYC':
            for row in reader:
                datum = row['starttime']
                Client_type = row['usertype']
                hour = int(datetime.strptime( datum, "%m/%d/%Y %H:%M:%S" ).strftime('%H'))
                day_of_week = datetime.strptime( datum, "%m/%d/%Y %H:%M:%S" ).strftime('%A')
                if Client_type == 'Subscriber' and (day_of_week !='Saturday' or day_of_week != 'Sunday'):
                        Hour_of_use_workdays_Subscriber.append(hour)
                elif Client_type == 'Customer' and (day_of_week !='Saturday' or day_of_week != 'Sunday'):
                    Hour_of_use_workdays_Customer.append(hour)
            
        elif city == 'Chicago':
            for row in reader:
                datum = row['starttime']
                Client_type = row['usertype']
                hour = int(datetime.strptime( datum, "%m/%d/%Y %H:%M" ).strftime('%H'))
                day_of_week = datetime.strptime( datum, "%m/%d/%Y %H:%M" ).strftime('%A')
                if Client_type == 'Subscriber' and (day_of_week !='Saturday' or day_of_week != 'Sunday'):
                        Hour_of_use_workdays_Subscriber.append(hour)
                elif Client_type == 'Customer' and (day_of_week !='Saturday' or day_of_week != 'Sunday'):
                    Hour_of_use_workdays_Customer.append(hour)
        
        elif city == 'Washington':
            for row in reader:
                datum = row['Start date']
                Client_type = row['Member Type']
                hour = int(datetime.strptime( datum, "%m/%d/%Y %H:%M" ).strftime('%H'))
                day_of_week = datetime.strptime( datum, "%m/%d/%Y %H:%M" ).strftime('%A')
                if Client_type == 'Registered' and (day_of_week !='Saturday' or day_of_week != 'Sunday'):
                        Hour_of_use_workdays_Subscriber.append(hour)
                elif Client_type == 'Casual' and (day_of_week !='Saturday' or day_of_week != 'Sunday'):
                    Hour_of_use_workdays_Customer.append(hour)

        
        #Subscribers data
        Early_Morning_Subscriber = 0 #for use before 7Am
        Morning_Rushhour_Subscriber = 0 #for use between 7-9Am
        During_workinghours_Subscriber = 0 #for use between 9-16
        Late_Rushhour_Subscriber = 0 #for use between 16-18
        Night_Subscriber = 0 #for use after 18
        
        #Customers data
        Early_Morning_Customer = 0 #for use before 7Am
        Morning_Rushhour_Customer = 0 #for use between 7-9Am
        During_workinghours_Customer = 0 #for use between 9-16
        Late_Rushhour_Customer = 0 #for use between 16-18
        Night_Customer = 0 #for use after 18
        
        
        
        for point in Hour_of_use_workdays_Subscriber:
            if int(point) < 7 :
                Early_Morning_Subscriber += 1
            elif 7 <= int(point) <= 9:
                Morning_Rushhour_Subscriber +=1
            elif 9 < int(point) < 16:
                During_workinghours_Subscriber +=1
            elif 16 <= int(point) <= 18:
                Late_Rushhour_Subscriber += 1
            elif int(point) > 18 :
                Night_Subscriber += 1
        
        for point in Hour_of_use_workdays_Customer:
            if int(point) < 7 :
                Early_Morning_Customer += 1
            elif 7 <= int(point) <= 9:
                Morning_Rushhour_Customer +=1
            elif 9 < int(point) < 16:
                During_workinghours_Customer +=1
            elif 16 <= int(point) <= 18:
                Late_Rushhour_Customer += 1
            elif int(point) > 18 :
                Night_Customer += 1

        
        # Pie Chart for Subscribers
        #Reference: https://pythonspot.com/matplotlib-pie-chart/
        # Data to plot
        labels = 'Early_Morning', 'Morning_Rushhour', 'During_workinghours', 'Late_Rushhour', 'Night'
        sizes = [Early_Morning_Subscriber, Morning_Rushhour_Subscriber, During_workinghours_Subscriber, Late_Rushhour_Subscriber, Night_Subscriber]
        colors = ['gold', 'yellowgreen', 'lightcoral', 'lightskyblue', 'gray']
        

        # Plot
        plt.pie(sizes, labels=labels, colors=colors,
                autopct='%1.1f%%', shadow=True, startangle=140)

        plt.axis('equal')
        plt.show()
        print('Subscribers')
   
        # Pie Chart for Customers
        #Reference: https://pythonspot.com/matplotlib-pie-chart/
        # Data to plot
        labels = 'Early_Morning', 'Morning_Rushhour', 'During_workinghours', 'Late_Rushhour', 'Night'
        sizes = [Early_Morning_Customer, Morning_Rushhour_Customer, During_workinghours_Customer, Late_Rushhour_Customer, Night_Customer]
        colors = ['gold', 'yellowgreen', 'lightcoral', 'lightskyblue', 'gray']
        

        # Plot
        plt.pie(sizes, labels=labels, colors=colors,
                autopct='%1.1f%%', shadow=True, startangle=140)

        plt.axis('equal')
        plt.show()
        print('Customers')
        
 ```

Using this function

```
data_files = ['./data/Washington-CapitalBikeshare-2016.csv']

for city_file in data_files:
    print (Daily_use_workdays(city_file))
    print("Pie Charts representing Subscribers daily use in working days in Washington")
    
 ```
 
The generated pie charts will show when the service was used most and by whom. Such conclusion can drive business strategy such as targeted segments (customers) and advertising scale and the image to be conveyed to the customers.
 
 
 # How to contribute to this project
 Well, you can modify my code. I believe the code could be simplified. 
 You can also share other ways to analyze the data and the kind of insights it might lead to.




  
