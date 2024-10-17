---
layout: post

title: Interactive Debugging Tool
---

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
 

