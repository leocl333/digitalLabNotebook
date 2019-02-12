download.file(url="https://ndownloader.figshare.com/files/2292169", destfile = "data/portal_data_joined.csv")

surveys <- b("data/portal_data_joined.csv")

surveys_complete <- surveys %>%  filter(!is.na(weight), !is.na(hindfoot_length),!is.na(sex)) 

species_counts <- surveys_complete %>% count(species_id) %>% filter(n >= 50)

surveys_complete <- surveys_complete %>% filter(species_id %in% species_counts$species_id)

write_csv(surveys_complete, path = "data_output/surveys_complete.csv")

Welcome to etherpad.net Pad!

This pad text is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents!
Images can be integrated either via drag and drop or copy and paste.

Beware that pad contents will NOT be deleted. Anyone who has the address to this pad can access it, meaning you should carefully handle the link.

In case of problems, you may contact: me@factor.cc

This service is free to use. However, if you appreciate this, please consider donating at https://factor.cc/donate

Welcome to the R sessions for Dunedin ResBaz 2019

Users are expected to follow our code of conduct: https://docs.carpentries.org/topic_folders/policies/code-of-conduct.html

Feel free to take collaborative notes on this doc and to leave feedback (anonymous if you like! Feedback makes our teaching better!), and Alana will post the  code at the end of the lesson.

Instructor and helpers: Alana, Ludo, and Cecila

# Notes from Data visualisation in R
Lesson link if you want to go over stuff later:
https://datacarpentry.org/R-ecology-lesson/04-visualization-ggplot2.html    

Learning objectives for today:
Learn how to produce a bunch of different kinds of plots and modify the aesthetics like labels and colour
We are also going to learn about universal plot settings and faceting
And combining this together we are going to be able to build complex and customized plots

download.file("https://github.com/laninsky/R-ecology-lesson/blob/master/data_output/surveys_complete.csv","data_output/surveys_complete.csv")


# Notes from Data Manipulation
Lesson links if you want to go over stuff later!:
https://datacarpentry.org/R-ecology-lesson/03-dplyr.html

Feel free to take collaborative notes on this doc and to leave feedback (anonymous if you like! Feedback makes our teaching better!), and Alana willpost the rest of the code at the end of the lesson.

We are specifically going to cover using dplyr to select columns and rows we are interested in
How to use pipes to whiz our data from one fuction to another
how to add new columns based on doing stuff to existing columns
how to split our data up, do stuff, and then summarize the output so we can get combined summary statistics and few other technical bits and pieces

# If you haven't installed tidyverse already
install.packages("tidyverse")

#otherwise
library(tidyverse)

# We only have to run install.packages() the first time
# after that we can just check it out using library()

# To download data
download.file("https://ndownloader.figshare.com/files/2292169","portal_data_joined.csv")

# Go back up one out of data folder, and set as wd

# read_csv to read data into R
surveys <- read_csv("data/portal_data_joined.csv")

# Have a look at our data
str(surveys)

#Preview the data
View(surveys)

surveys

# grab a subset of columns: select() seleCt
select()
# grab a subset of rows: filter() filteR
filter()
# create summary statistics on grouped data
group_by() %>% summarise()
# sort results
arrange()
# counthings
count()

# select - first argument is our data - our tibble
# subsequent arguments are columns we want
select(surveys,plot_id,species_id,weight)

# To select everything BUT, use a - in front of column name
select(surveys,-record_id,-species_id)

# To pull our rows
filter(surveys,year==1995)

# To both filter and select, we could use intermediate steps
surveys2 <- filter(surveys, weight < 5)
surveys2
surveys_sml <- select(surveys2,species_id,sex,weight)
surveys_sml

# You could use nested functions
surveys_sml <- select(filter(surveys, weight < 5),species_id,sex,weight)
surveys_sml

# Third option is using pipes %>% 
# The shortcut for making a pipe is Shift+Ctrl+m
# %>% 
surveys %>% filter(weight < 5) %>% select(species_id,sex,weight)

surveys_sml <- surveys %>% filter(weight < 5) %>% select(species_id,sex,weight)
surveys_sml

# How to create new columns
surveys %>% mutate(weight_kg=weight/1000)

# You can get fancy
surveys %>% mutate(weight_kg=weight/1000,weight_kg2=weight_kg^2)

# Use head to preview data
surveys %>% mutate(weight_kg=weight/1000,weight_kg2=weight_kg^2) %>% head()

# To filter out NAs
# If things are NA you can find this out using is.na()
# If you want to find things that are NOT NA !is.na()
surveys %>% filter(!is.na(weight)) %>% 
  mutate(weight_kg = weight/1000) %>% head()

# group_by tells R what columns we want to group things by
# summarise tells it what variable you want to summarise
surveys %>% group_by(sex) %>% 
  summarise(mean_weight=mean(weight))

# To get it to work, tell it to ignore NAs
surveys %>% group_by(sex) %>% 
  summarise(mean_weight=mean(weight, na.rm=TRUE))

# To see more lines is use print function
surveys %>% group_by(sex,species_id) %>% 
  summarise(mean_weight=mean(weight,na.rm=TRUE)) %>% 
  print(n=72)

# How about filtering some of those missing combinations earlier?
surveys %>% filter(!is.na(weight)) %>% 
  group_by(sex,species_id) %>% 
  summarise(mean_weight=mean(weight)) %>% print(n=72)

# Can create more than one new column in same function call
surveys %>% filter(!is.na(weight)) %>% 
  group_by(sex,species_id) %>% 
  summarise(mean_weight=mean(weight), min_weight=min(weight))

# What about sorting the dataset?
surveys %>% filter(!is.na(weight)) %>% 
  group_by(sex,species_id) %>% 
  summarise(mean_weight=mean(weight), min_weight=min(weight)) %>% arrange(min_weight)

# What about sorting the dataset from big to small?
surveys %>% filter(!is.na(weight)) %>% 
  group_by(sex,species_id) %>% 
  summarise(mean_weight=mean(weight), min_weight=min(weight)) %>% arrange(desc(min_weight))

# Getting counts of stuff we are interested in from our data
surveys %>% count(sex)

# Sorting by count
surveys %>% count(species_id, sort=TRUE)

# If you want count but not to sort by it
surveys %>% count(sex, species) %>% arrange(species,desc(n))
# compared to arranging by total count
surveys %>% count(sex, species,sort=TRUE)

# To create new tables out of existing ones, summarsing stuff
# spread()
spread_gw <- surveys %>% filter(!is.na(weight)) %>% 
  group_by(genus,plot_id) %>% 
  summarize(mean_weight=mean(weight))

spread_gw

# Here goes spread
surveys_spread <- spread_gw %>% spread(key=genus,value=mean_weight)

surveys_spread

# Getting rid of observations for animals where they don't
# have weight, hindfoot length or sex
surveys_complete <- surveys %>% 
  filter(!is.na(weight),!is.na(hindfoot_length),!is.na(sex)) 
surveys_complete

# Extract the species IDs of the most common species
species_counts <- surveys_complete %>% 
  count(species_id) %>% filter(n >= 50)

species_counts

# In surveys complete we only want to keep those
# species that are also in species_counts
surveys_complete <- surveys_complete %>% 
  filter(species_id %in% species_counts$species_id)

surveys_complete
# if your filtering has gone OK, 30463 rows, 13 columns
dim(surveys_complete)


# To write out data
write_csv(surveys_complete,"data_output/surveys_complete.csv")





# Notes from Intro to R

Lesson links if you want to go over stuff later!:
    https://datacarpentry.org/R-ecology-lesson/00-before-we-start.html
    https://datacarpentry.org/R-ecology-lesson/01-intro-to-r.html

Instructor and helpers: Alana, Paul, Ludo, and Harry

Feel free to take collaborative notes on this doc and to leave feedback (anonymous if you like! Feedback makes our teaching better!), and Alana will post the rest of the code at the end of the lesson.

Learning objectives, part 1
Describe the purpose of the RStudio Script, Console, Environment, and Plots panes.
Organize files and directories for a set of analyses as an R Project, and understand the purpose of the working directory.
Use the built-in RStudio help interface to search for more information on R functions.
Demonstrate how to provide sufficient information for troubleshooting with the R user community.

Learning Objectives, part 2
Define the following terms as they relate to R: object, assign, call, function, arguments, options.
Assign values to objects in R.
Learn how to _name_ objects
Use comments to inform script.
Solve simple arithmetic operations in R.
Call functions and use arguments to change their default options.
Inspect the content of vectors and manipulate their content.
Subset and extract values from vectors.
Analyze vectors with missing data.

# If you need to check your working directory (wd
getwd()
# To send stuff down from script to console Ctrl+Enter

# Set your working directory 
setwd("/where/you/want/to/be")

getwd(
)  

# If you know a function or command's name but don't know how
# to use it
?barplot

# If you want to know what arguments a function needs
args(barplot)

# If you are really flying blind
??kruskal

# R can do maths for you
3 + 5
12/7

# objects and variables are the same thing
# to "save" or assign a value to a variable
# we use the assignment operator <-
# object/variable <- value
weight_kg <- 55
# Ctrl+Enter to shot it down to console

# Allowed names cannot have number first
2pac <- "I like rap"
#letters first
pac2 <- "I like rap"

# R is case sensitive
Weight_KG <- 60
weight_kg
Weight_KG

# a few other things you cannot use: if, for, else
else <- 200

# you COULD use other function names as variable names
mean <- 200
mean

# Is a word already a function?
mean()
thisdefinitielyisnotafunctionprobably()

# Don't put dots in name
it.won.t.stop.you.though <- 600

# To make R tell you what it is doing when assignign
(weight_kg <- 55)

# Convert kg to lbs
weight_kg *2.2

# To capture new value we need to assign to a new variable
weight_lb <- weight_kg *2.2
weight_lb
# You can use tab to autocomplete things

# If we no change weight_kg
weight_kg <- 100
weight_kg * 2.2
weight_lb

# Hashes stop anything on that line to the right being run
20*2
#30*2

# Shift+Ctrl+c to comment or uncomment bunch of stuff
# blah
# blah
# blah

# A function called sqrt
sqrt()
# argument = 4
sqrt(4)
sqrt_four <- sqrt(4)
sqrt_four

# the function round has multiple arguments, some are defaults
round(3.14159)

# args command we learned about earlier
args(round)
?round

# for 2 whatevers
round(3.14159,2)

# can change up order of arguments
round(digits=2,x=3.14159)

# Use the c() function to chuck elements into a vector
weight_g <- c(50, 60, 65, 82)
weight_g

# Vectors can contain characters
animals <- c("cat", "dog","mouse")
animals
animals <- c(cat, dog,mouse)
cat()

# Want to see how much stuff is in your vector
length(weight_g)
length(animals)

# Type of data in your vector
class(animals)
class(weight_g)

# Crzy frankenvector
frankenvector <- c(3,4,"dog")
frankenvector
class(frankenvector)

# Add stuff to your vectors using c()
giantvector <- c(animals,weight_g,frankenvector)
giantvector

# Use weird square bracekts to pull out things from vectors
frankenvector[2]

# Make somethign bigger than frankenvector with frankenveco
frankenvector[c(1,1,2,2,3,3)]

# condtional subsetting

weight_g[weight_g < 62]
weight_g


# Matching text
c("rat","dog") %in% animals
