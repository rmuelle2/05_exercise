---
title: 'Weekly Exercises #5'
author: "Rylan Mueller"
output: 
  html_document:
    keep_md: TRUE
    toc: TRUE
    toc_float: TRUE
    df_print: paged
    code_download: true
---


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, error=TRUE, message=FALSE, warning=FALSE)
```

```{r libraries}
library(tidyverse)     # for data cleaning and plotting
library(gardenR)       # for Lisa's garden data
library(lubridate)     # for date manipulation
library(openintro)     # for the abbr2state() function
library(palmerpenguins)# for Palmer penguin data
library(maps)          # for map data
library(ggmap)         # for mapping points on maps
library(gplots)        # for col2hex() function
library(RColorBrewer)  # for color palettes
library(sf)            # for working with spatial data
library(leaflet)       # for highly customizable mapping
library(ggthemes)      # for more themes (including theme_map())
library(plotly)        # for the ggplotly() - basic interactivity
library(gganimate)     # for adding animation layers to ggplots
library(transformr)    # for "tweening" (gganimate)
library(gifski)        # need the library for creating gifs but don't need to load each time
library(shiny)         # for creating interactive apps
theme_set(theme_minimal())
```

```{r data}
# SNCF Train data
small_trains <- read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-02-26/small_trains.csv") 

# Lisa's garden data
data("garden_harvest")

# Lisa's Mallorca cycling data
mallorca_bike_day7 <- read_csv("https://www.dropbox.com/s/zc6jan4ltmjtvy0/mallorca_bike_day7.csv?dl=1") %>% 
  select(1:4, speed)

# Heather Lendway's Ironman 70.3 Pan Am championships Panama data
panama_swim <- read_csv("https://raw.githubusercontent.com/llendway/gps-data/master/data/panama_swim_20160131.csv")

panama_bike <- read_csv("https://raw.githubusercontent.com/llendway/gps-data/master/data/panama_bike_20160131.csv")

panama_run <- read_csv("https://raw.githubusercontent.com/llendway/gps-data/master/data/panama_run_20160131.csv")

#COVID-19 data from the New York Times
covid19 <- read_csv("https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-states.csv")

```

## Put your homework on GitHub!

Go [here](https://github.com/llendway/github_for_collaboration/blob/master/github_for_collaboration.md) or to previous homework to remind yourself how to get set up. 

Once your repository is created, you should always open your **project** rather than just opening an .Rmd file. You can do that by either clicking on the .Rproj file in your repository folder on your computer. Or, by going to the upper right hand corner in R Studio and clicking the arrow next to where it says Project: (None). You should see your project come up in that list if you've used it recently. You could also go to File --> Open Project and navigate to your .Rproj file. 

## Instructions

* Put your name at the top of the document. 

* **For ALL graphs, you should include appropriate labels and alt text.** 

* Feel free to change the default theme, which I currently have set to `theme_minimal()`. 

* Use good coding practice. Read the short sections on good code with [pipes](https://style.tidyverse.org/pipes.html) and [ggplot2](https://style.tidyverse.org/ggplot2.html). **This is part of your grade!**

* **NEW!!** With animated graphs, add `eval=FALSE` to the code chunk that creates the animation and saves it using `anim_save()`. Add another code chunk to reread the gif back into the file. See the [tutorial](https://animation-and-interactivity-in-r.netlify.app/) for help. 

* When you are finished with ALL the exercises, uncomment the options at the top so your document looks nicer. Don't do it before then, or else you might miss some important warnings and messages.

## Warm-up exercises from tutorial

  1. Choose 2 graphs you have created for ANY assignment in this class and add interactivity using the `ggplotly()` function.
```{r, fig.alt= "A scatterplot of the bill length and bill depth of certain species of penguins with different colors for each species."}
penguin_graph <- penguins %>%
  ggplot(aes(x = bill_length_mm,
             y = bill_depth_mm, 
             color = species)) + 
  geom_point(na.rm = TRUE) +
  labs(x = "", y = "", title = "Bill Length (mm) vs. Bill Depth (mm) of Penguins", color = "Species")

ggplotly(penguin_graph)
```
  
```{r, fig.alt="A line graph showing the cumulative number of COVID-19 cases in Iowa, Minnesota, North and South Dakota, and Wisconsin with separate lines for each state."}
midwest_covid <- covid19 %>% 
  filter(state %in% c("Minnesota", "Wisconsin", "Iowa", "North Dakota", "South Dakota")) %>%
  ggplot(aes(x = date, y = cases, color = state))+
  geom_line() +
  labs(x = "", y = "", title = "Cumulative COVID cases in some Midwest states", color = "State")

ggplotly(midwest_covid)
```


  
  2. Use animation to tell an interesting story with the `small_trains` dataset that contains data from the SNCF (National Society of French Railways). These are Tidy Tuesday data! Read more about it [here](https://github.com/rfordatascience/tidytuesday/tree/master/data/2019/2019-02-26).
  
```{r, fig.alt="An animated bar graph showing the top 15 French train stations with the most departures from 2015 to 2018.", eval=FALSE}
trains <- small_trains %>% 
  group_by(year, departure_station) %>% 
  summarize(total_departs = sum(total_num_trips)) %>% 
  filter(n() >= 15) %>% 
  top_n(n = 15, wt = total_departs) %>% 
  arrange(year, total_departs) %>% 
  mutate(rank = 1:n()) %>% 
  
  ggplot(aes(x = total_departs,
             y = factor(rank),
             fill = departure_station,
             group = departure_station)) +
  geom_col() +
    geom_text(aes(label = departure_station),
            x = 10,
            hjust = "left") +
  scale_x_continuous(limits = c(-80,NA),
                     breaks = c(seq(0,400,20))) +
  labs(title = "15 French Stations with the Most Departures",
       subtitle = "Year: {round(frame_time,0)}",
       x = "", 
       y = "") +
  theme(axis.line = element_blank(), 
        panel.grid = element_blank(),
        axis.text.y = element_blank(),
        legend.position = "none") +
  scale_fill_viridis_d() +
  transition_time(year)

animate(trains, nframes = 200, duration = 30)
anim_save("trains.gif")
knitr::include_graphics("trains.gif")
```
  

## Garden data

  3. In this exercise, you will create a stacked area plot that reveals itself over time (see the `geom_area()` examples [here](https://ggplot2.tidyverse.org/reference/position_stack.html)). You will look at cumulative harvest of tomato varieties over time. I have filtered the data to the tomatoes and find the *daily* harvest in pounds for each variety. The `complete()` function creates a row for all unique `date`/`variety` combinations. If a variety is not harvested on one of the harvest dates in the dataset, it is filled with a value of 0. 
  You should do the following:
  * For each variety, find the cumulative harvest in pounds.  
  * Use the data you just made to create a static cumulative harvest area plot, with the areas filled with different colors for each variety and arranged (HINT: `fct_reorder()`) from most to least harvested weights (most on the bottom).  
  * Add animation to reveal the plot over date. Instead of having a legend, place the variety names directly on the graph (refer back to the tutorial for how to do this).

```{r, eval=FALSE, fig.alt="An animated line graph of the cumulative harvest (in lbs) of different tomatoes overlaid on each other."}
racing_tomato <- garden_harvest %>% 
  filter(vegetable == "tomatoes") %>% 
  group_by(date, variety) %>% 
  summarize(daily_harvest_lb = sum(weight)*0.00220462) %>% 
  ungroup() %>% 
  complete(variety, 
           date, 
           fill = list(daily_harvest_lb = 0)) %>% 
  group_by(variety) %>% 
  mutate(cum_tomato = cumsum(daily_harvest_lb)) %>% 
  
  ggplot(aes(x = date, y = cum_tomato)) +
  geom_area(aes(fill = variety)) + 
  geom_text(aes(label = variety)) +
  labs(title = "Cumulative Harvest (in lbs) of Tomato Varieties", y = "", subtitle = "Date: {frame_along}")+
  theme(legend.position = "none") +
  transition_reveal(date)

animate(racing_tomato, nframes = 200, duration = 30)
anim_save("racing_tomato.gif")
knitr::include_graphics("racing_tomato.gif")
```


## Maps, animation, and movement!

  4. Map Lisa's `mallorca_bike_day7` bike ride using animation! 
  Requirements:
  * Plot on a map using `ggmap`.  
  * Show "current" location with a red point. 
  * Show path up until the current point.  
  * Color the path according to elevation.  
  * Show the time in the subtitle.  
  * CHALLENGE: use the `ggimage` package and `geom_image` to add a bike image instead of a red point. You can use [this](https://raw.githubusercontent.com/llendway/animation_and_interactivity/master/bike.png) image. See [here](https://goodekat.github.io/presentations/2019-isugg-gganimate-spooky/slides.html#35) for an example. 
  * Add something of your own! And comment on if you prefer this to the static map and why or why not.
  
```{r, eval=FALSE, fig.alt= "An animated line map of a bike ride in Mallorca, Spain with the line and leading point colored by the elevation."}
mallorca_map <- get_stamenmap(
    bbox = c(left = 2.3412, bottom = 39.5385, right = 2.7422, top = 39.7183), 
    maptype = "terrain",
    zoom = 11
)
mallorca_anim <- ggmap(mallorca_map) +
  geom_path(data = mallorca_bike_day7, 
             aes(x = lon, y = lat, color = ele),
             size = 1) +
  geom_point(data = mallorca_bike_day7,
             aes(x = lon, y = lat, color = ele),
             size = 5) +
  scale_color_viridis_c(option = "magma") +
  theme_map() +
  transition_reveal(time) +
  labs(title = "Mallorca Bike Ride", subtitle = "Time: {frame_along}") +
  theme(legend.background = element_blank())

animate(mallorca_anim, nframes = 200, duration = 30)
anim_save("mallorca_anim.gif")
knitr::include_graphics("mallorca_anim.gif")
```
  **I like the animation, however I do not think it is necessary in this case. With the static map, one can quickly look at the map and see the path. But with the animated map it takes awhile.**
  
  5. In this exercise, you get to meet Lisa's sister, Heather! She is a proud Mac grad, currently works as a Data Scientist where she uses R everyday, and for a few years (while still holding a full-time job) she was a pro triathlete. You are going to map one of her races. The data from each discipline of the Ironman 70.3 Pan Am championships, Panama is in a separate file - `panama_swim`, `panama_bike`, and `panama_run`. Create a similar map to the one you created with my cycling data. You will need to make some small changes: 1. combine the files putting them in swim, bike, run order (HINT: `bind_rows()`), 2. make the leading dot a different color depending on the event (for an extra challenge, make it a different image using `geom_image()!), 3. CHALLENGE (optional): color by speed, which you will need to compute on your own from the data. You can read Heather's race report [here](https://heatherlendway.com/2016/02/10/ironman-70-3-pan-american-championships-panama-race-report/). She is also in the Macalester Athletics [Hall of Fame](https://athletics.macalester.edu/honors/hall-of-fame/heather-lendway/184) and still has records at the pool. 
  
```{r, eval=FALSE, fig.alt="An animated line map of Heather's triathlon in Panama with the leading point colored by the event."}
triathlon <- bind_rows(panama_swim, panama_bike, panama_run)
  
panama_map <- get_stamenmap(
    bbox = c(left = -79.5821, bottom = 8.9048, right = -79.4819, top = 8.9891), 
    maptype = "terrain",
    zoom = 15
)

triathlon_anim <- ggmap(panama_map) +
  geom_path(data = triathlon,
            aes(x = lon, y = lat),
            size = 1)+
  geom_point(data = triathlon,
             aes(x = lon, y = lat, color = event),
             size = 5) + 
  theme_map() +
  transition_reveal(time) +
  labs(title = "Heather's Triathlon in Panama",subtitle = "Time: {frame_along}") +
  theme(legend.background = element_blank())

animate(triathlon_anim, nframes = 200, duration = 30)
anim_save("triathlon_anim.gif")
knitr::include_graphics("triathlon_anim.gif")
```
  
## COVID-19 data

  6. In this exercise you will animate a map of the US, showing how cumulative COVID-19 cases per 10,000 residents has changed over time. This is similar to exercises 11 & 12 from the previous exercises, with the added animation! So, in the end, you should have something like the static map you made there, but animated over all the days. The code below gives the population estimates for each state and loads the `states_map` data. Here is a list of details you should include in the plot:
  
  * Put date in the subtitle.   
  * Because there are so many dates, you are going to only do the animation for the the 15th of each month. So, filter only to those dates - there are some lubridate functions that can help you do this.   
  * Use the `animate()` function to make the animation 200 frames instead of the default 100 and to pause for 10 frames on the end frame.   
  * Use `group = date` in `aes()`.   
  * Comment on what you see.  

```{r, eval=FALSE, fig.alt= "An animated map of the cumulative COVID cases per 10,000 people in the United States on the 15th of every month."}
march9_covid <- covid19 %>% 
  filter(mday(date) == 15) %>% 
  mutate(state = str_to_lower(state))

census_pop_est_2018 <- read_csv("https://www.dropbox.com/s/6txwv3b4ng7pepe/us_census_2018_state_pop_est.csv?dl=1") %>% 
  separate(state, into = c("dot","state"), extra = "merge") %>% 
  select(-dot) %>% 
  mutate(state = str_to_lower(state))

covid_with_population <-
  march9_covid %>% 
  left_join(census_pop_est_2018,
            by = "state") %>% 
  mutate(covid_per_10000 = (cases/est_pop_2018)*10000)


states_map <- map_data("state")
  
  animated_map <- ggplot() +
  geom_map(data = covid_with_population,
           map = states_map,
           aes(map_id = state,
               fill = covid_per_10000,
               group = date)) +
  expand_limits(x = states_map$long, y = states_map$lat) + 
  scale_fill_viridis_c(option = "magma", direction = -1) +
  theme_map()+
  transition_time(date) +
  labs(title = "COVID cases per 10,000 people", fill = "COVID per 10000", subtitle = "Date: {frame_time}")
  
  animate(animated_map, nframes = 200, duration = 10)
  anim_save("animated_map.gif")
  knitr::include_graphics("animated_map.gif")
```
**At first, the whole US, regardless of the state's population, has a low number of cases per 10,000. This makes sense at the beginning of the pandemic. As the date increases, each state gets darker and darker, because the map shows the number of cumulative cases.**

## Your first `shiny` app (for next week!)

  7. This app will also use the COVID data. Make sure you load that data and all the libraries you need in the `app.R` file you create. You should create a new project for the app, separate from the homework project. Below, you will post a link to the app that you publish on shinyapps.io. You will create an app to compare states' daily number of COVID cases per 100,000 over time. The x-axis will be date. You will have an input box where the user can choose which states to compare (`selectInput()`), a slider where the user can choose the date range, and a submit button to click once the user has chosen all states they're interested in comparing. The graph should display a different line for each state, with labels either on the graph or in a legend. Color can be used if needed.
  
  
Put the link to your app here: https://rmuelle2.shinyapps.io/Exercise_5/
  
## GitHub link

  8. Below, provide a link to your GitHub repo with this set of Weekly Exercises. 


**DID YOU REMEMBER TO UNCOMMENT THE OPTIONS AT THE TOP?**
