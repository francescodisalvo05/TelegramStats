# TelegramStats

Today I'm going to analyze my **telegram conversation** with my girlfriend. For collecting your own data you have to go to the chat that you want to analyze, then click on the option panel and finally click on *export chat history*! For obvious reasons I won't insert the JSON file, but you can use yours with the previous explanation. <br /> <br />

This time I won't ask any particular questions, but I'll go trough the analysis as I feel. So, let's begin! Firstly, we need to look something about the **JSON** file: 


```python
{
 "name": #chat_name,
 "type": "personal_chat",
 "id": #id,
 "messages": [
  {
   "id": #
   "type": #
   "date": #
   "from": #
   "from_id": #,
   "text": #
  },
     ...
 ]
}
```

My **telegram id** is "1#######0", and my girlfriend is "9#######8"

Sencondly, we have to add the **libraries** that we'll use soon


```python
import numpy as np
import pandas as pd
import json

import matplotlib.pyplot as plt
from matplotlib import cm
import seaborn as sns

from datetime import datetime
```

Thirdly, we have to create the **dataframe** by the **JSON** file


```python
df = pd.read_json("ChatExport_2020-09-03/result.json")
df.head(1)
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
      <th>type</th>
      <th>id</th>
      <th>messages</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>#</td>
      <td>personal_chat</td>
      <td>#</td>
      <td>{'id': #, 'type': '#', 'date': '#...</td>
    </tr>
  </tbody>
</table>
</div>



We need just the "messages" column, so we can do as following


```python
df = df["messages"]
df.head(3)
```




    0    {'id': #, 'type': 'service', 'date': '#...
    1    {'id': #, 'type': '#', 'date': '#...
    2    {'id': #, 'type': '#', 'date': '#...
    Name: messages, dtype: object



### Number of messages
The first thing that I'd like to know is who sent more messages between us. Since there are some records without "from_id" but with "action_id" (e.g. voice call) we have to handle the KeyErr with a try catch. When we find this kind of error we can just "continue" and skip this cycle. 


```python
# I'm the first one and she's the second one ("id" : value)
count_message = {"1#######0" : 0, "9#######8" : 0}

for i in range(0,df.size):
    try: 
        count_message[str(df[i]["from_id"])]+= 1
    except KeyError:
        continue
```


```python
index = ["ME", "SHE"]     
values = [count_message["1#######0"],count_message["9#######8"]]
plt.bar(index, values,color=["#0000E5","#CC0000"])
```




    <BarContainer object of 2 artists>




![Error](https://github.com/francescodisalvo05/TelegramStats/blob/master/Screen/output_11_1.png?raw=true)


It seems that I'm a **stalker**! 

### Most common daytime
Now I'd like to discover when we mostly talk during the the day. For semplicity I'll divide the day in 4 blocks of 6 hours each: <br />
- **morning** : from 6.00am to 11.59am
- **afternoon** : from 12.00pm to 5.59pm
- **eveneng** : from 6.00pm to 11.59am
- **night** : from 12.00am to 5.59am <br />

We can follow the previus method. We need to change just the for construct. Before we start, let's check how to get the date.


```python
df[0]["date"]
```




    '2019-08-13T15:32:33'



So we need to get from the 11th character to the latter


```python
count_message_daytime = {"morning" : 0, "afternoon" : 0, "eveneng" : 0, "night" : 0}

# boundary
morning_l = datetime.strptime('06:00:00', '%H:%M:%S')
morning_m = datetime.strptime('11:59:59', '%H:%M:%S')
afternoon_l = datetime.strptime('12:00:00', '%H:%M:%S')
afternoon_m = datetime.strptime('17:59:59', '%H:%M:%S')
eveneng_l = datetime.strptime('18:00:00', '%H:%M:%S')
eveneng_m = datetime.strptime('23:59:59', '%H:%M:%S')
night_l = datetime.strptime('00:00:00', '%H:%M:%S')
night_m = datetime.strptime('5:59:59', '%H:%M:%S')

for i in range(0,df.size):
    time = df[i]["date"][11:]
    time = datetime.strptime(time, '%H:%M:%S')
    if ( morning_l <= time <= morning_m):
        count_message_daytime["morning"] += 1
    elif ( afternoon_l <= time <= afternoon_m):
        count_message_daytime["afternoon"] += 1
    elif ( eveneng_l <= time <= eveneng_m):
        count_message_daytime["eveneng"] += 1
    elif ( night_l <= time <= night_m):
        count_message_daytime["night"] += 1
    
count_message_daytime
```




    {'morning': 15401, 'afternoon': 23942, 'eveneng': 25626, 'night': 2639}




```python
index = ["Morning", "Afternoon", "Evening", "Night"]     
values = [ count_message_daytime["morning"], 
           count_message_daytime["afternoon"], 
           count_message_daytime["eveneng"], 
           count_message_daytime["night"]
         ]
colors=["#236e96", "#15b2d3", "#ffd700", "#f3872f"]
plt.bar(index, values, color=colors)
```




    <BarContainer object of 4 artists>




![Error](https://github.com/francescodisalvo05/TelegramStats/blob/master/Screen/output_17_1.png?raw=true)


### Text length and audio duration
Let's discover who is the talkative of the couple! The procedure is similar to the previous ones


```python
me = {"length_message" : 0, "number_message" : 0, "length_audio" : 0, "number_audio" : 0}
she = {"length_message" : 0, "number_message" : 0, "length_audio" : 0, "number_audio" : 0}

#me = "1#######0" | she =  "9#######8" 

for i in range(0,df.size):
    try: 
        if (df[i]["from_id"] == 1#######0):
            me["length_audio"] += df[i]["duration_seconds"]
            me["number_audio"] += 1
        else: 
            she["length_audio"] += df[i]["duration_seconds"]
            she["number_audio"] += 1
    except KeyError:
        # duration_seconds is not present, so it's a message
        try: 
            if (df[i]["from_id"] == 1#######0 and len(df[i]["text"]) > 0):
                me["length_message"] += len(df[i]["text"])
                me["number_message"] += 1
            elif (len(df[i]["text"]) > 0): 
                she["length_message"] += len(df[i]["text"])
                she["number_message"] += 1
        except KeyError:
            continue
    

print(me)
print(she)
```

    {'length_message': 628109, 'number_message': 29175, 'length_audio': 45490, 'number_audio': 3079}
    {'length_message': 524535, 'number_message': 24985, 'length_audio': 94651, 'number_audio': 6491}



```python
index = ["ME", "SHE"]
value_m = [ me["length_message"]/me["number_message"], she["length_message"]/she["number_message"] ]
value_a = [ me["length_audio"]/me["number_audio"], she["length_audio"]/she["number_audio"] ]
print(value_m)
print(value_a)
```

    [21.529014567266497, 20.993996397838703]
    [14.774277362780124, 14.58188260668618]



```python
fig, (ax1, ax2) = plt.subplots(1, 2,figsize=(15,4))

ax1.barh(index, value_m,color=["#0000E5","#CC0000"])
ax1.set_title("average messages length")
ax2.barh(index, value_a,color=["#0000E5","#CC0000"])
ax2.set_title("average audio length")
```




    Text(0.5, 1.0, 'average audio length')




![Error](https://github.com/francescodisalvo05/TelegramStats/blob/master/Screen/output_21_1.png?raw=true)


I'll finish, I promise!


```python

```
