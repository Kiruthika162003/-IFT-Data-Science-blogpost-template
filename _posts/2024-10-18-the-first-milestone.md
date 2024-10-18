---
layout: post

title: A brief tutorial on downloading the NHL PlaybyPlay dataset
---

### The Question

Write a brief tutorial on how your team downloaded the dataset. Imagine :) that you were searching for a guide on how to download the play-by-play data; your guide should make you go “Perfect - this is exactly what I was looking for!”. Include your function/class, and provide an example of using it. Please ensure you’re not just demonstrating that your functionality works - this is also an exercise in documenting and communicating your implementation. It does not need to be extremely complicated, but we expect something slightly more coherent and digestible than just screenshots of your functions/code.

### Preliminaries - identifying the API and GameID

### API

The NHL meticulously collects comprehensive data for each of its games, covering everything from season information and team standings to specific in-game events. This includes detailed play-by-play data that provides extensive information about every event during a game. You can access this data by making a GET request to the following API endpoint: https://statsapi.web.nhl.com/api/v1/game/[GAME_ID]/feed/live/. The play-by-play data contains detailed records of events such as faceoffs, shots, goals, saves, and hits.

### Game ID
Game IDs are composed of three parts:

* Season Start Year: This is the year when the season begins. For example, '2017' corresponds to the 2017-2018 season.
* Game Type Code: A two-digit number indicating the type of game:
'01' for pre-season games
'02' for regular season games
'03' for playoff games
'04' for all-star games
* Game Number: This varies based on the type of game:
    * Regular Season: Game numbers range from 1 to 1271.
    * Playoffs: The numbering is more detailed:
        * The second digit denotes the playoff round.
        * The third digit specifies the matchup within that round.
        * The fourth digit indicates the specific game number in a best-of-seven series.

### The Steps
This tutorial focuses on how to fetch NHL play-by-play data from the NHL's API, handle potential errors, and store the data locally. The programming language used is Python.

* Step 1: Importing Libraries
```python
import os
import requests
import json
```
**`os`**: Used for interacting with the operating system (e.g., creating directories).  
**`requests`**: This library is essential for making HTTP requests to the NHL API.  
**`json`**: This library is used to work with the JSON data format, which is how the API provides the data.

* Step 2: Creating the **`fetch_nhl_play_by_play_data`** Function
```python
def fetch_nhl_play_by_play_data(season, game_type, game_number):
    """
    Fetch NHL play-by-play data for a given game in a season.

    Args:
        season (int): The starting year of the NHL season (e.g., 2016 for the 2016-17 season).
        game_type (str): The type of game (e.g., "01" for preseason, "02" for regular, "03" for playoffs).
        game_number (str): The specific game number, padded appropriately.

    Returns:
        dict: JSON data of the play-by-play for the specified game, or None if not found.
    """
    game_id = f"{season}{game_type}{game_number}"
    url = f"https://api-web.nhle.com/v1/gamecenter/{game_id}/play-by-play"

    try:
        response = requests.get(url)
        response.raise_for_status()  # Raise an error for bad responses (e.g., 404)
        data = response.json()
        return data
    except requests.exceptions.HTTPError as errh:
        if response.status_code == 404:
            print(f"Game ID {game_id} not found (404). Skipping.")
        else:
            print("HTTP Error:", errh)
    except requests.exceptions.RequestException as err:
        print("Request Error:", err)
        return None
```
__Constructing the API URL:__
It takes the `season`, `game_type`, and `game_number` as input.
It constructs the `game_id` and the API URL using f-strings for easy formatting:
```python
game_id = f"{season}{game_type}{game_number}"
url = f"https://api-web.nhle.com/v1/gamecenter/{game_id}/play-by-play"
```
Here, we have to bear in mind the composition of the Game ID, which is composed of three sections:
1. The starting year of the season (e.g., "2020" for the 2020-2021 season).
2. The game type ("01" for pre-season, "02" for regular season, "03" for playoffs, and "04" for all-star games).
3. The game number, which differs between regular season and playoff games.
__Making the Request:__
It uses `requests.get(url)` to send a GET request to the API endpoint:
```python
response = requests.get(url)
```
__Error Handling:__
A **`try...except`** block handles potential errors during the request: 
```python
try:
    response = requests.get(url)
    response.raise_for_status()  # Raise an error for bad responses (e.g., 404)
    data = response.json()
    return data
except requests.exceptions.HTTPError as errh:
    if response.status_code == 404:
        print(f"Game ID {game_id} not found (404). Skipping.")
    else:
        print("HTTP Error:", errh)
except requests.exceptions.RequestException as err:
    print("Request Error:", err)
    return None
```
__Returning Data:__
If the request is successful, it parses the JSON response using `response.json()` and returns the data.

* Step 3: Creating the **`save_data_to_file`** Function
```python
def save_data_to_file(data, file_path):
    """
    Save fetched data to a JSON file.

    Args:
        data (dict): The JSON data to save.
        file_path (str): The file path to save the data to.
    """
    if data:
        with open(file_path, 'w') as file:
            json.dump(data, file, indent=4)
        print(f"Data saved to {file_path}")
    else:
        print("No data to save.")
```
This function takes the **`data`** to be saved and the **`file_path`** as arguments.
It opens the file in write mode (**`'w'`**) and uses **`json.dump()`** to save the data in JSON format with an indent for readability.

* Step 4: The Main Execution (if __name__ == "__main__":)
```python
if __name__ == "__main__":
  # Iterate over each season from 2016 to 2024
  for year in range(2016, 2016):
      for game_type in ["02", "03"]:
          if game_type == "02":
              game_numbers = list(range(1, 1272))
          else:
              game_numbers = list(range(1, 132))

          for game_num in game_numbers:
              if game_type == "03":  # Special formatting for playoff games
                  game_id_str = f"0{str(game_num).zfill(3)}"  # Pad with a leading zero, followed by 3 digits (e.g., "0001" -> "00001")
              else:
                  game_id_str = str(game_num).zfill(4)  # Zero-pad game number to 4 digits

              data_folder = os.getenv("NHL_DATA_FOLDER", "/content/nhl_data")  # Default to local folder if env variable not set
              os.makedirs(data_folder, exist_ok=True)

              file_path = os.path.join(data_folder, f"game_{year + 1}_{game_type}_{game_id_str}.json")

              # Check if data already exists locally
              if os.path.exists(file_path):
                  print(f"Data already exists for game {year + 1}-{game_type}-{game_id_str}. Skipping download.")
                  continue

              # Fetch and save data
              data = fetch_nhl_play_by_play_data(year, game_type, game_id_str)
              if data:
                  save_data_to_file(data, file_path)
```
__Setting up the Data Directory:__
It gets the data folder path from the `NHL_DATA_FOLDER` environment variable or defaults to `"/content/nhl_data"`.  
It creates the data directory if it doesn't exist using `os.makedirs(data_folder, exist_ok=True)`.
```
```
__Iterating Through Games:__
It iterates through years (only 2016 in this case), game types (`"02"` for regular season, `"03"` for playoffs), and game numbers.
It formats the game_id_str differently for playoff games.
```
```
__Checking for Existing Data:__
It constructs the `file_path` where the data will be saved.
It checks if the file already exists at that path. If it does, it skips downloading the data.
```
```
__Fetching and Saving:__
It calls `fetch_nhl_play_by_play_data` to get the data.
It calls `save_data_to_file` to save the data if it was successfully fetched.


## Interactive Debugging Tool

### The Question

 Take a screenshot of the tool and add it to the blog post, accompanied with the code for the tool and a brief (1-2 sentences) description of what your tool does. You do not need to worry about embedding the tool into the blogpost.

### Tool Code

```markdown

```python
import matplotlib.pyplot as plt
import ipywidgets as widgets
from IPython.display import display
from PIL import Image


import os

nhl_files = os.listdir('/content/nhl_data')



def get_metadata(game_id, event_id, files):
  f = open('/content/nhl_data/' + nhl_files[game_id - 1])
  selected_game = json.load(f)
  selected_event = {}



  for e in selected_game['plays']:
    if e['eventId'] == event_id:
      selected_event = e
      break


  return len(selected_game['plays']) - 1, selected_game['id'], selected_game['startTimeUTC'], selected_game['homeTeam']['name']['default'], selected_game['awayTeam']['name']['default'], e









rink_image_path = '/content/nhl_rink.png'  # Replace with your ice rink image file path
rink_image = Image.open(rink_image_path)
center_x, center_y = rink_image.width / 2, rink_image.height / 2

# Function to plot a single event on the rink image
def plot_single_event_on_rink(event, game_id, event_id):
    # Create a plot with the rink image as the background
    fig, ax = plt.subplots(figsize=(10, 6))
    ax.imshow(rink_image, extent=[-center_x, center_x, center_y, -center_y])  # Adjust extent to center origin


    # Plot the single event
    ax.scatter(event['details']['xCoord'], event['details']['yCoord'], c='red', s=200, marker='o')
    ax.text(event['details']['xCoord'] + 5, event['details']['yCoord'], f"Game {game_id} - {event['eventId']}", fontsize=12, color='white')

    # Set plot title and hide axis
    plt.title(f"Game ID: {game_id}, Event ID: {event_id}")
    ax.axis('off')  # Hide axis
    plt.show()

# Create interactive sliders
game_id_slider = widgets.IntSlider(min=1, max=len(nhl_files), step=1, value=1, description='Game ID')
event_id_slider = widgets.IntSlider(min=1, max=2, step=1, value=1, description='Event ID')

# Function to update the plot based on slider values
def update_plot(game_id, event_id):
    # Get the selected event based on the slider value
    events_length, id, time, home, away, selected_event = get_metadata(game_id, event_id, nhl_files)

    # Update event_id slider range based on the number of events in the selected game

    event_id_slider.max = events_length

    print('id:'+str(id))
    print(time)
    print(home+'(home) vs '+away+'(away)')

    print(selected_event)


    if 'details' in selected_event.keys():
      if 'xCoord' in selected_event['details'].keys():
        # Plot the selected event on the rink
        plot_single_event_on_rink(selected_event, game_id, event_id)

    print(selected_event)


# Link sliders to the update function
widgets.interact(update_plot, game_id=game_id_slider, event_id=event_id_slider)

# Display the sliders
display(game_id_slider, event_id_slider)

```

### Input Images

![Image 1](images\screenshot_interactive_debug_1.png)
![Image 2](images\screenshot_interactive_debug_2.png)

```
```

## Tidy data

---
layout: post

title: Tidy data
---

### The Questions
1. In your blog post, include a small snippet of your final dataframe (e.g. using `head(10)`). You can just include a screenshot rather than fighting to get the tables neatly formatted in HTML/markdown.

2. You’ll notice that the “strength” field (i.e. even, power play, short handed) only exists for goals, not shots. Furthermore, it doesn’t include the actual strength of players on the ice (i.e. 5 on 4, or 5 on 3, etc). Discuss how you could add the actual strength information (i.e. 5 on 4, etc.) to both shots and goals, given the other event types (beyond just shots and goals) and features available. You don’t need to implement this for this milestone. 

3. In a few sentences, discuss at least 3 additional features you could consider creating from the data available in this dataset. We’re not looking for any particular answers, but if you need some inspiration, could a shot or goal be classified as a rebound/shot off the rush (explain how you’d determine these!) ?

### A small snippet of our final dataframe

The following table provides a detailed log of events from a specific hockey game, including shots and goals.

| Game ID     | Period | Time in Period | Event Type | Team ID | X Coordinate | Y Coordinate | Shot Type | Shooter ID | Goalie ID | Empty Net | Strength |
|-------------|--------|----------------|------------|---------|--------------|--------------|-----------|------------|-----------|-----------|----------|
| 2016020025  | 1      | 01:03          | Shot       | 3       | -56.0        | -40.0        | Wrist     | 8474151    | 8475622.0 | False     | None     |
| 2016020025  | 1      | 01:13          | Goal       | 19      | 81.0         | 3.0          | Wrist     | 8475765    | 8468685.0 | False     | Even     |
| 2016020025  | 1      | 02:26          | Shot       | 3       | -55.0        | -27.0        | Wrist     | 8471686    | 8475622.0 | False     | None     |
| 2016020025  | 1      | 02:35          | Shot       | 3       | -39.0        | -33.0        | Wrist     | 8475692    | 8475622.0 | False     | None     |
| 2016020025  | 1      | 04:42          | Shot       | 3       | -40.0        | 28.0         | Snap      | 8476431    | 8475622.0 | False     | None     |
| 2016020025  | 1      | 05:25          | Goal       | 3       | -66.0        | -7.0         | Wrist     | 8475184    | 8475622.0 | False     | Even     |
| 2016020025  | 1      | 06:59          | Shot       | 3       | -61.0        | -17.0        | Wrist     | 8475692    | 8475622.0 | False     | None     |
| 2016020025  | 1      | 07:06          | Shot       | 3       | -54.0        | -12.0        | Wrist     | 8471686    | 8475622.0 | False     | None     |
| 2016020025  | 1      | 08:20          | Shot       | 3       | -79.0        | 5.0          | Wrist     | 8473546    | 8475622.0 | False     | None     |
| 2016020025  | 1      | 08:45          | Shot       | 19      | 55.0         | -27.0        | Slap      | 8474102    | 8468685.0 | False     | None     |

### Potential to add actual strength information to both shots and goals
For every shot or goal, start with two counters, one for each team, and set them both to 5. Look at the time of the shot or goal and find the earliest event in the past that happened more than 5 minutes before it. Then, check if there were any penalties in those 5 minutes. If there was a minor penalty and it happened within 2 minutes of the shot or goal, find out which team was penalized and subtract 1 from their counter. If there was a major penalty, find out which team was penalised and subtract 1 from their counter. Finally, make a new feature called "strength" for each shot or goal that shows the values of both counters, with the counter for the team that is shooting first and the counter for the team that is defending second.

### 3 Potential additional features - Discussion

**Shot Distance from Goal (shot_distance)**

Calculates the Euclidean distance of each shot from the goal.

**Time Between Shots (time_between_shots)**

Calculates the time elapsed since the previous shot, providing context on the game's pace.

**Odd-Man Rush Indicator (odd_man_rush)**

Marks whether a shot was part of an odd-man rush, indicating an advantageous attacking situation.


## Simple Visualisations

### The Questions

1. Question 1
    * (a) Produce a figure comparing the shot types over all teams (i.e. just aggregate all of the shots), in a season of your choosing. 
    * (b) Overlay the number of goals overtop the number of shots. 

        (i) What appears to be the most dangerous type of shot? 

        (ii) The most common type of shot? 

        (iii) Why did you choose this figure? Add this figure and discussion to your blog post.
        
2. Question 2
    * (a) What is the relationship between the distance a shot was taken and the chance it was a goal? 
    * (b)

        (i) Produce a figure for each season between 2018-19 to 2020-21 to answer this, and add it to your blog post along with a couple of sentences describing your figure. 

        (ii) Has there been much change over the past three seasons? 

        (iii) Why did you choose this figure?

3. Combine the information from the previous sections to produce a figure that shows the goal percentage (# goals / # shots) as a function of both distance from the net, and the category of shot types (you can pick a single season of your choice). Briefly discuss your findings; e.g. what might be the most dangerous types of shots?

### Shot types comparison over all teams in Season 2016-2017


![Shot Types Comparison](images\shot_compare.png)


For the 2016-2017 season, despite the wrist shot, with over 40 000 shots taken, being the most **common** (frequent), the most **dangerous** shot type would depend on the conversion efficiency (goals/shots). Deflected shots have the highest conversion rate, at 0.20. This means about 20% of deflected shots result in goals, making it the most effective shot type in terms of goal-scoring efficiency.

### Choice of figure

The bar chart shows shots (in blue) and the overlayed red line plot (in red) together for each shot type, providing a clear comparison between total attempts and successful conversions (goals). By overlaying, the chart illustrates how a shot type contributes to total shots, while also highlighting how much of that total results in goals.

The distance between the each red point and the centre of each bar immediately helps visualise the efficiency of each shot type without needing complex calculations. The conversion rate labels (above each bar) provide a direct numeric measure of efficiency, showing the percentage of shots that result in goals. This saves the viewer from having to calculate it manually.

The figure employs a dual y-axis system, with one axis representing total shots and the other showing total goals, facilitating a clear comparison between the volume of shots taken and their effectiveness in scoring goals. Additionally, the graph includes goal-to-shot ratios, displayed as red points, which provide valuable insights into the efficiency of each shot type. By presenting multiple metrics—total shot volume in blue bars and goal outcomes in red—the figure enables easy identification of not only the most frequently used shot types but also those that are most successful at converting shots into goals, which can be more meaningful in sports analytics.

The conversion rates placed above each bar give immediate insight into efficiency without overcrowding the figure. This avoids the need for a separate efficiency plot and integrates all necessary data in a single view. The layout of shot types on the x-axis makes it easy to see and compare the different types side by side.

Moreover, the color difference (blue for shots and red for goals) makes it easy to distinguish between attempts and successes, providing a clear visual contrast that aids interpretation.

### Relationship between the distance a shot was taken and the chance it was a goal

![Shot distance vs Chance of Goal  2018-2019](images\shot_distance_vs_chance_of goal_2018-2019.png)
![Shot distance vs Chance of Goal  2019-2020](images\shot_distance_vs_chance_of goal_2019-2020.png)
![Shot distance vs Chance of Goal  2020-2021](images\shot_distance_vs_chance_of goal_2020-2021.png)

### The Relationship

As the distance from which a shot is taken increases, the chance of it becoming a goal decreases. Close-range shots have the highest probability of scoring, while shots from longer distances are less likely to result in goals. This reflects the reality of games in NHL. This trend is consistent across different seasons, emphasizing the advantage of proximity in improving scoring chances. From the figure, the best shots—those with the highest probability of resulting in goals—are taken from within the closest bins, typically up to 25 units. These short-distance shots consistently show the highest goal probabilities across all three seasons.

### Trends or consistency ?

The core trend has remained stable: closer shots have higher goal probabilities. However, there have been slight variations, particularly in the fluctuations beyond 75 units. For example, the 2020-2021 season shows more pronounced peaks and troughs at longer distances, suggesting some strategic or gameplay adjustments. But in general, the relationship between shot distance and goal probability has stayed consistent.

### Choice of figure

The line graphs clearly show how goal probability changes with shot distance. They make it easy to see the overall trend and identify specific patterns within each season. The use of upper bound bins on the horizontal axis provides clear demarcations of distance ranges. The graphs not only show the overall trend but also capture fluctuations in goal probability at longer distances. They allow viewers to grasp both the high-level trend and the finer details, such as the minor peaks and troughs that indicate successful long-range shots.

### Goal Percentage by Shot Type and Distance Range (2018-2019 Season)

![Shot distance vs Chance of Goal  2018-2019](images\heatmap_goal_percentage_distance_range_2018-2019.png)

This heatmap provides insights into the effectiveness of various shot types at different distances. The shot types are listed on the y-axis, including backhand, deflected, slap, snap, tip-in, wrap-around, and wrist shots. The x-axis shows the distance ranges, categorized as Close (0-15), Mid (15-30), Long (30-60), and Very Long (60+). The color intensity in the heatmap represents the goal percentage, with darker blue shades indicating higher percentages and lighter shades indicating lower percentages. This visual representation allows viewers to understand which shot types are most effective at various distances.

The findings are as follows:

* Dangerous, high-percentagem shots include snap shots from the Close (0-15) range which have the highest goal percentage at 22.32%, indicating they're most effective at short distances.

* Zero-Percentage Shots such as wrap-around shots have a 0.00% goal percentage from Long (30-60) and Very Long (60+) distances, suggesting these are ineffective at longer ranges.

* Deflected shots from the Long (30-60) range have a relatively high goal percentage at 15.91%, showing some effectiveness even from further away.

#### A little extra

##### Figure 1

![Goal Percentage by Shot Type and Distance Range](images\bars_goal_percentage_distance_range_2018-2019.png)

* Close range (0-15 ft): All shot types have relatively high goal percentages, with tip-ins and snap shots showing the highest success rates, followed closely by deflections and slap shots.
* Mid range (15-30 ft): The goal percentages drop slightly, but snap shots, wrist shots, and deflections still show solid performance in this range.
* Long range (30-60 ft): Goal percentages drop further for all shot types. Deflected shots lead this range, followed by snap shots and slap shots.
* Very long range (60+ ft): Only a few shot types have a chance of scoring from this distance, with wrist shots and snap shots having the highest goal percentages, although the overall chances are low.

##### Figure 2

![Goal Outcome by Distance Range](images\kd_goal_percentage_distance_range_2018-2019.png)

This diagram represents the shot distance distribution by goal outcome for the 2018-2019 season. The x-axis displays the shot distance from the net in feet, ranging from 0 to 200, while the y-axis shows the density of shot occurrences for two different outcomes:

* Blue area (isGoal = 0): Represents shots that did not result in goals.
* Orange area (isGoal = 1): Represents shots that resulted in goals.

Takeaways:

* Close range (0-15 feet): The majority of goals (orange) occur in this range, with a high density of successful shots close to the net. The density of missed shots (blue) is also high but less than the successful ones.

* Mid range (15-50 feet): As the distance increases, the proportion of goals decreases compared to missed shots. The density of unsuccessful shots (blue) remains substantial, but goals (orange) become less frequent.

* Long range (50+ feet): Very few goals are scored from distances beyond 50 feet. The density of unsuccessful shots continues to drop as the shot distance increases, and goals from this range are extremely rare.



##### Figure 3

![Goal Outcome by Distance Range 2](\images\plot_shot_distance_vs_chance_of goal_2019-2020.png)

This plot displays the relationship between shot distance and goal outcome for the 2019-2020 season, using a hexbin plot. The x-axis represents shot distance from 0 to 200 feet, while the y-axis represents the goal outcome, where:

* 0 (lower part) = No goal
* 1 (upper part) = Goal

The color intensity, shown in the color bar on the right, indicates the shot count—darker shades represent a higher number of shots at a given distance.

The Insights:
* The majority of the shots occur at short distances, as seen by the cluster of darker blue hexagons between 0 and 50 feet. This is typical because shots close to the net are more frequent in hockey games.

* The hexagons at the top of the plot (y=1, indicating goals) are clustered primarily within short distances (around 0 to 40 feet). This shows that most goals are scored from close range, with very few goals coming from beyond 50 feet.

* Most of the missed shots (y=0) occur at shorter distances too, but there is a spread of missed shots extending to longer distances (up to 100 feet and beyond). The density of shots decreases as the distance increases, as shown by the lighter colors further from the net.

## Advanced Visualisations

### The Questions
1. Export the 4 plot offensive zone plots to HTML, and embed it into your blog post. Your plot must allow users to select any team during the selected season. 

2. Discuss (in a few sentences) what you can interpret from these plots.

3. Consider the Colorado Avalanche; take a look at their shot map during the 2016-17 season. Discuss what you could say about the team during this season. Now look at the shot map for the Colorado Avalanche for the 2020-21 season, and discuss what you could conclude from these differences. Does this make sense?

4. Consider the Buffalo Sabres, which have been a team that has struggled over recent years, and compare them to the Tampa Bay Lightning, a team which has won the Stanley for two years in a row. Look at the shot maps for these two teams from the 2018-19, 2019-20, and 2020-21 seasons. Discuss what observations you can make. Is there anything that could explain the Lightning’s success, or the Sabres’ struggles? How complete of a picture do you think this paints?

### The Heatmaps

{% include excess_shot_rate_heatmap2016.html %}
{% include excess_shot_rate_heatmap2017.html %}
{% include excess_shot_rate_heatmap2018.html %}
{% include excess_shot_rate_heatmap2019.html %}
{% include excess_shot_rate_heatmap2020.html %}

* Nearly every team tends to take shots close to the net. Some teams, i.e. the Sharks, have consistently maintained their offensive style over multiple seasons, with most shots occurring near the net or about 50 feet away on the left side of the rink whilst other teams, i.e., the Capitals, have changed their offensive approach across multiple seasons; they shifted from primarily shooting from central x-axis positions to adopting a more varied origin for their shooting (shooting from different positions), i.e. in the 2020-2021 season.

* The Colorado Avalanche underwent a significant shift in their offensive tactics. In the 2016-2017 season, they had a lower-than-average shot frequency than other teams in the league average from the area directly in front of the net. However, in the 2020-2021 season, there offensive tactics were seemingly deliberately changed as indicated by the increase in their shot rate near the net to well above the league average.

* Both the Buffalo Sabres and the Tampa Bay Lightning primarily take shots from near to mid-range distances. The Sabres tend to direct their shots more toward the left and right sides of the rink, with fewer attempts coming from the middle. Conversely, the Lightning focus most of their shooting efforts down the center, with less emphasis on the sides. he Lightning concentrate most of their shots down the center of the rink, which is typically the most effective area for scoring goals. Shots from the center have better angles and are more challenging for goaltenders to defend, increasing the likelihood of scoring. In contrast, the Sabres tend to direct their shots toward the left and right sides of the rink, with fewer attempts from the middle. Shots from the sides often have poorer angles and are easier for goaltenders to save, which could contribute to fewer goals scored and, consequently, more losses. These differences in shooting strategies might partially explain why the Lightning have been more successful, as they capitalize on high-percentage scoring areas, while the Sabres may not be optimizing their shot locations as effectively. However, this data provides only a partial picture of each team's overall performance. In hockey many variables, including defensive play, goaltending quality, special teams performance (power plays and penalty kills), player skill levels, coaching strategies, injuries, and team chemistry, come into play. To fully understand the Lightning's success and the Sabres' struggles, one would need to consider all these elements in addition to shooting patterns.