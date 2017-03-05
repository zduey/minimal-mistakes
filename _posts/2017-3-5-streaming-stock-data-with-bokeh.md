---
title: Streaming Stock Price Data with Bokeh

categories:
    - snippets
tags:
    - python
    - data-viz
    - bokeh
---

## Overview 
As part of my 2017 goal to work on a small analytics-oriented web app, I started doing some research into what I would want to use for the visualization component. Being a huge fan of python, I wanted to try out [bokeh](http://bokeh.pydata.org/en/latest/), which touts interactive visualizations using pure python. Bokeh also allows for a number of different demployment options, including within a Flask app, so it seemed like a reasonable option to consider.

For a quick weekend hack, I opted to build a real-time price chart. The [Investors Exchange](https://www.iextrading.com/) (IEX) recently released an API that allows you to get the last trade price in real-time (and a bunch of other data for that matter) for equities trading on their exchange. 

The scope of the project was very small: build a single page bokeh app that would stream stock price quotes and allow the user to change which ticker to stream. The script boils down to three components:

1. A function to get the last traded price
2. A callback function to update the ticker being streamed
3. The code to set up the chart

## Data
The IEX api has a number of different endpoints. The base url is ```https://api.iextrading.com/1.0``` and the endpoint for accessing the last traded price is /tops. As an example, if you want the last traded price for Snap, Inc. you would send a GET request to: ```https://api.iextrading.com/1.0/tops?symbols=SNAP```. IEX provides very clear [documentation](https://www.iextrading.com/developer/), so I won't go into more detail about usage.

```python
import io
import requests
import pandas as pd


base = "https://api.iextrading.com/1.0/"

def get_last_price(symbol):
    payload = {
        "format": "csv",
        "symbols": symbol
    }
    endpoint = "tops/last"

    raw = requests.get(base + endpoint, params=payload)
    raw = io.BytesIO(raw.content)
    prices_df = pd.read_csv(raw, sep=",")
    prices_df["time"] = pd.to_datetime(prices_df["time"], unit="ms")
    prices_df["display_time"] = prices_df["time"].dt.strftime("%m-%d-%Y %H:%M:%S.%f")

    return prices_df
```

There are a few things to note about this section. First, I wanted to get the data into a pandas dataframe to do a little post-processing. Using the io library, you can store data in a memory buffer and then read out of that buffer with pandas. This saves you overhead of doing disk i/o. Second, the IEX API returns all times in milliseconds since the Unix epoch. Creating the ```display_time``` variable is done in order to have a nicely-formatted date in the tooltip for the chart. 

Bokeh has nice integration with pandas.The best option when your data is in a pandas dataframe is to use a ```ColumnDataSource``` object, which takes either a dictionary or a pandas dataframe as an argument. I had some trouble with constructing the ```ColumnDataSource``` directly from the pandas dataframe because it will include the dataframe's index as a column. Instead, I went with a slighly clunkier option and explicitly created a dictionary. 

```python
data = ColumnDataSource(dict(time=[], display_time=[], price=[]))
```
Once the data source is set up, there are a number of methods availabe. However, the only one I will be using is ```stream()``` which allows you to append new data to existing columns in your data source. 

```python
def update_price():
    new_price = get_last_price(symbol=TICKER)
    data.stream(dict(time=new_price["time"],
                     display_time=new_price["display_time"],
                     price=new_price["price"]), 
                10000)
    return
```

One thing to note is the second argument to ```stream()```, which controls how many datapoints will be kept in the datasource before rolling off. If you have a large datasource, keeping this parameter reasonably small will keep your browser from getting bogged down in rendering the chart.


## Updating the Ticker
Your ```ColumnDataSource``` object has a ```data``` attribute, which is the dictionary containing the underlying data. Directly modifying this attribute is what allows me to update what ticker is being streamed. When a user submits a new ticker to stream, this attribute is reset and begins receiving data after the next request to the IEX API.

```python
TICKER = ''

def update_ticker():
    global TICKER
    TICKER = ticker_textbox.value
    price_plot.title.text = "IEX Real-Time Price: " + ticker_textbox.value
    data.data = dict(time=[], display_time=[], price=[])

    return
```

One thing I am not a fan of is the apparent need to make TICKER a global variable. So far as I can tell, you cannot pass args to your callback functions. It is possible that I am mistaken about this, so I will post an update if I find a better way. The ```ticker_textbox``` is the bokeh text input widget. When ```update_ticker()``` is called, it uses the current value of that input widget as the new ticker to stream. The funciton also updates the title in the bokeh figure to reflect the currently-streaming ticker.

## Setting up the Dashboard
The dashboard is very simple, but there are still a number of components involved. First, to create the tooltip, you do so by creating a HoverTool object. 

```python
hover = HoverTool(tooltips=[
    ("Time", "@display_time"),
    ("IEX Real-Time Price", "@price")
    ])
```

The ```@``` syntax corresponds to variables in your data source. Any column in the data can be added to your tooltip. In this case, I am displaying the price and my formatted time variable.

To set up the plot, you create a ```figure``` object. Many of the figure's attributes can be configured during creation, or you can access and modify them after the fact. You can see both below:

```python
price_plot = figure(plot_width=800,
                    plot_height=400,
                    x_axis_type='datetime',
                    tools=[hover, ResizeTool(), SaveTool()])

price_plot.line(source=data, x='time', y='price')
price_plot.xaxis.axis_label = "Time"
price_plot.yaxis.axis_label = "IEX Real-Time Price"
price_plot.title.text = "IEX Real Time Price: " + TICKER
```

Next, I needed to create two widgets: a textbox for capturing tickers and a button to trigger the update. You can set a callback function for a widget by passing a callable to the ```on_click()```method of a widget. Finally, I bound the two user input widgets together in a widgetbox, which provides the benefit of ensuring all widgets have the same sizing mode. 

```python
ticker_textbox = TextInput(placeholder="Ticker")
update = Button(label="Update")
update.on_click(update_ticker)

inputs = widgetbox([ticker_textbox, update], width=200)
```

The last piece is to finish setting up the current document. I arrange the widgetbox and the figure into a single row and bind that to the view. Finally, to stream the data I specify ```update_price()``` as the callback function to use at a fixed interval of 1 second (1000ms).

```python
curdoc().add_root(row(inputs, price_plot, width=1600))
curdoc().add_periodic_callback(update_price, 1000)
```

## Running the App
The full code is available as a [gist](https://gist.github.com/zduey/66ed98cf3fc2b161df47c0c08954dc62). Download/clone/etc. the script and then run ```bokeh serve iex.py``` from the command line. The bokeh server will fire up and display the dashboard at port 5006. Type in your ticker, hit update, and the price data will begin streaming. Keep in mind that you will only get streaming data [when the market is open](https://www.iextrading.com/trading/). If you choose a lightly-traded product, it will be less interesting, so I recommend starting with a big-name firm. For a complete list, IEX provides a regularly-updated [list of tickers](https://www.iextrading.com/trading/eligible-symbols/).