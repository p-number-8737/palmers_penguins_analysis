# Palmers_Penguins_Analysis
This is a repository storing all of the files for my RProject using the Palmers Penguins dataset.

---
title: "Palmer_Penguins_Analysis"
output:
  html_document: default
date: "2023-12-07"
---
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

**candidate number: 1064454**

## QUESTION 1: Data Visualisation for Science Communication

### a) My Palmer Penguins figure:

```{r bad figure code, echo=FALSE}
# There is no need to provide the code for your bad figure, just use echo=FALSE so the code is hidden. Make sure your figure is visible after you knit it. 

# Load necessary libraries
library(ggplot2)
library(readr)

# Load the Palmer Penguins dataset
penguins <- read_csv('~/Downloads/penguins_clean.csv')

# Create a poorly designed plot
ggplot(penguins, aes(x = flipper_length_mm, y = body_mass_g, color = species)) +
  geom_point(size = 12, alpha = 0.5) + 
  scale_color_manual(values = c("yellow", "green", "blue")) + 
  theme_minimal() +
  labs(title = "Flipper Length vs Body Mass in Penguins", 
       x = "Bill Length (mm)", 
       y = "Body Mass (g)") +
  theme(plot.title = element_text(size = 10, face = "italic"), 
        axis.text.x = element_text(angle = 90, vjust = 0.5, color = "white"), 
        axis.text.y = element_text(face = "bold", color = "white"), 
        legend.position = "top", 
        plot.background = element_rect(fill = "orange")) 

```

### b) How do my data visualisation choices mislead the reader about the underlying data?

Data visualisation is the first important step in understanding the relationship between data categories i.e. body mass and bill length. There are 4 key ways to ensure correct data visualisation: clear graphical elements, showing data, honest magnitudes and showing patterns.

**In this scatter graph I mislead the reader in the data visualisation methods including:**

1. Large, semi-translucent points which overlap which loses detail and obscures individual data points. The semi-translucence distorts the colour of overlapping points which obscures data.

2. Colour choices for data points are not colour-blind accessible and are distracting. This means data points are difficult to distinguish and may be misinterpreted.

3. Title is small and axis labels/ text is rotated making it hard to read and fails to effectively communicate the plot to readers.

4. The legend is at the top and is not clear which interferes with visualising the plot

5. Using an orange plot background detracts from the plotted data as it is difficult to focus on individual points further obscuring the relationship in the underlying data.

This poor visualisation may contribute to misinterpretation/ obscuring data which reduces the reproducibility of this work for other readers as it is not a reliable information source. This hinders the ability to build upon this data by other researchers which is crucial for open-resources and replicating the data presented. Poor data visualisation may reduce the accessibility of the work which is also a major drawback for open-source science and further limits collaboration with other researchers. Poor visualisation may also reduce the credibility of the underlying data to the reader promoting skepticism about my data analysis. 


**references:**
- Baker, M. (2016). 1,500 scientists lift the lid on reproducibility. Nature, 533(7604), 452–454. https://doi.org/10.1038/533452a 
- McKiernan, E. C., Bourne, P. E., Brown, C. T., Buck, S., Kenall, A., Lin, J., McDougall, D., Nosek, B. A., Ram, K., Soderberg, C. K., Spies, J. R., Thaney, K., Updegrove, A., Woo, K. H., &amp; Yarkoni, T. (2016). How open science helps researchers succeed. eLife, 5. https://doi.org/10.7554/elife.16800 
- Raj, A. (2023, May 28). 12 bad data visualization examples explained. Code Conquest. https://www.codeconquest.com/blog/12-bad-data-visualization-examples-explained/ 
------------------------------------------------------------
-------------

## QUESTION 2: Data Pipeline

------------------------------------------------------------------------

### Introduction
This data analysis will use the Palmer Penguins dataset - collected by Gorman et al 2014 - to explore the relationships between different species and their bill length. The Palmer Penguins dataset contains the following columns: species, island, culmen length (mm), culmen depth (mm), body mass (g), flipper length (mm) and sex.

The first step is to load the necessary packages:
```{r loading the dataset and packages}
#install and load packages required for analysis
#install.packages("palmerpenguins") <- the dataset
#install.packages("janitor") <- for cleaning the data
#install.packages("dplyr") <- for creating functions to filter data
#install.packages("ggplot2") <- for plotting figures
#install.packages("car") <- for checking statistical test assumptions later

library(palmerpenguins)
library(janitor)
library(ggplot2)
library(dplyr)
library(car)

```

The second step is to accurately clean the Palmer Penguins dataset. This is important to ensure accuracy, remove NAs (which skew analysis), standardise formats, and improve data readability to ensure correct data analysis.
```{r Cleaning the data}
#this is a file of functions which have been created to clean this dataset
source("functions/cleaning.r")

#assess the data - this is not tidy
str(penguins_raw)

#pipe the cleaning functions to clean columns names, shorten species names, and remove NAs at the same time
penguins_clean <- penguins_raw %>%
    clean_column_names() %>%
    shorten_species() %>%
    remove_NA()

#check the cleaned data has removed NAs and cleaned the columns names
summary(is.na(penguins_clean))
names(penguins_clean)

#subset the data to only include the species and culmen length columns for further analysis
penguins_analysis <- subset_columns(penguins_clean, c("culmen_length_mm", "species"))

```

The next step should be to produce an initial exploratory figure of the cleaned and subset penguin data.
```{r exploration of the data}

#histogram plot of culmen length distribution categorised by species

ggplot(penguins_analysis, aes(x = culmen_length_mm, fill = species)) +
  geom_histogram(bins = 30, position = "dodge") +
  scale_fill_manual(values = c("Adelie" = "deeppink2", "Chinstrap" = "seagreen2", "Gentoo" = "turquoise3")) +
  labs(title = "Distribution of Bill Lengths by Species",
       x = "Bill Length (mm)",
       y = "Count") +
  theme_bw() # Adding a white background theme for a cleaner look

#saving the plot 
ggsave("exploratory_figure.png", dpi = 300)


```
The **histogram plot** has been used to visualise the distribution of a continuous variable - culmen length - and how this differs across species. This can be used to identify skew, outliers and comparative species distributions as it is easily interpretable for initial data exploration. 

Based on this initial histogram it suggests there is a difference in culmen length between species - specifically Adelie when compared to Chinstrap and Gentoo.

### Hypothesis
**H0:** There is no significant difference in the culmen length across different penguin species.

**HA:** There is a significant difference in bill length across different penguin species.

### Statistical Methods

I will conduct an **Analysis of Variance** to test if there are significant differences in the mean culmen length across species. This is a suitable test as ANOVA is designed to compare means across multiple categories - i.e. 3 species: Adelie, Chinstrap and Gentoo. ANOVA can also assess within and between group variance to identify whether differences observed are greater than expected due to random variation within each species. ANOVA also reduces the type 1 error rate of running multiple comparative analyses for each species. 

The key assumptions of this test include:normal distribution of data within each category, equal variance and random sampling. The previous exploratory histogram suggests a normal distribution of culmen lengths for each species, but I will check the assumption of this using a Q-Q plot, and check the assumptions for equal variance using Levene's test.

Assuming these are met if the ANOVA outputs a significant result further post-hoc tests can be used to identify which species differ in culmen length. In this case I would use a Tukey-Kramer test to compare every species against eachother in pairwise comparisons and identify those which have the most significant difference in mean culmen lengths.
```{r Checking the ANOVA assumptions are met}

#Q-Q plot for normal distribution
ggplot(penguins_analysis, aes(sample = culmen_length_mm))+
  facet_wrap(~species) +
  stat_qq() +
  stat_qq_line()

#Levene's Test for equal variances
library(car)
leveneTest(culmen_length_mm ~ species, data = penguins_analysis)


```
The Q-Q plot results show each of the three species approximately following the reference line, this suggests they each have a normal distribution of culmen length. In addition, Levene's test has produced a relatively small F-statistic suggesting lower discrepancy in variance between species. The p-value of this test is not significant at 0.1033 meaning the difference in variance is not significant between species. Therefore, both the assumptions of equal variance and normal distribution have been met and ANOVA can be used for further statistical analysis.

```{r Statistical analysis of culmen length and species}

#fit a linear model to the penguin_analysis data
culmen_mod1 <- lm(culmen_length_mm ~ species, penguins_analysis)

#print the output of the linear model to find the mean culmen length estimates for each species
culmen_mod1

#run anova on this linear model
anova(culmen_mod1)

#a significant result was shown so conduct a Tukey-Kramer test
TukeyHSD(aov(culmen_mod1))
```

### Results & Discussion

The results of the ANOVA test show there is statistically significant difference in the mean culmen length between species with a **p-value of 2.2e-16**. The post-hoc Tukey Kramer analysis shows that all species significantly differ in culmen length, however Adelie has a larger difference to both Gentoo and Chinstrap than these do to eachother. 

The results of this ANOVA test can be visualised in a **violin plot**. This plot is useful for showing the entirety of the data distribution to show the highest density of points for each species. This plot can also show measures of central tendency i.e. the median/mean and data spread i.e. IQR. The violin plot is a relatively intuitive plot to understand and shows differences in distribution between species very effectively. These plots can be particularly useful for larger sample size - as in this dataset - where individual data points may cluster and obscure the patterns shown.
```{r Plotting Results}

ggplot(penguins_analysis, aes(x = species, y = culmen_length_mm, fill = species)) +
  geom_violin() +
  geom_boxplot(width = 0.1, fill = NA, color = "black", alpha = 0.6) +  geom_point(aes(colour = species), alpha = 0.3) +
  scale_fill_manual(values = c("Adelie" = "deeppink2", "Chinstrap" = "seagreen2", "Gentoo" = "turquoise3")) +
  labs(title = "Culmen Lengths by Penguin Species",
       x = "Species",
       y = "Culmen Length (mm)") +
  theme_bw() 

ggsave("results_figure_violin.png")


```

### Conclusion

My previous linear model analysis shows the mean culmen length estimate for Adelie is 38.824mm, Chinstrap is 10.010mm, and Gentoo is 8.744mm. The results of the ANOVA test suggest that there are significant differences in mean culmen length across the different penguin species. Further analysis under a Tukey-Kramer test suggest that this significant difference in mean culmen length is seen between all three species - however there is the largest significant difference in Adelie-Gentoo and Adelie-Chinstrap comparisons. This result aligns with the mean culmen length approximated from the linear model fit as Adelie clearly has a much larger mean culmen length.

Therefore, these results mean I can reject the null hypothesis of no significant difference in mean culmen length across different species. These results suggest each species may adapt to different ecological or evolutionary mechanisms which promote this divergent culmen length. For example, differences in culmen length may be related to variation in diet, habitat condition, or feeding modes between the species. These could have broader implications for how the species interact with their environment and other species. 


------------------------------------------------------------------------

## QUESTION 3: Open Science

### a) GitHub

*My GitHub link:* <https://github.com/p-number-8737/palmers_penguins_analysis.git>

### b) Share your repo with a partner, download, and try to run their data pipeline.

*Partner's GitHub link:* <https://github.com/1064807/Palmer_Penguins.git>


### c) Reflect on your experience running their code. (300-500 words)

-   *What elements of your partner's code helped you to understand their data pipeline?*

The analysis was well-structured in clear sections. There were many comments separating the code which was very helpful as it made running each code chunk easy to follow, and also provided explanations for what each line of code was doing making it easily interpretable. 

This created a smooth flow across the data pipeline as each step of the analysis was clearly defined in chunks, with comments explaining how and why the particular code was being used. This was particularly useful when constructing visual plots i.e. in box and violin plots, where there were large chunks of code which may be difficult to dissect. 


-   *Did it run? Did you need to fix anything?*

Yes the code ran relatively smoothly when I downloaded. However, I needed to fix the source("functions/Functions.R") when cleaning the data. This is because the functions file was not uploaded to the GitHub, only the Function.R was. This was the only issue I encountered and once this was fixed the code ran smoothly. 

-   *What suggestions would you make for improving their code to make it more understandable or reproducible, and why?*

I would suggest including extra comments when cleaning the data to explain the functions being used. It may also be helpful to introduce jitter plots and what they visualise, as well as providing some analysis of this exploratory plot before the hypothesis section. 

When constructing the violin plot I would remove/ change the colour of the added data points as in my opinion these detract from the overall graph interpretation and make it difficult to view. I would also explain in more detail what the purpose of the interaction plots is as whilst there were comments explaining the code for the interaction plot it was slightly confusing how this information helped in understanding the data analysis.

In addition, I would include a code chunk which checked the assumptions of the ANOVA model were being met for the dataset - i.e. normal distribution and Q-Q plot - before beginning the statistical analysis rather than assuming that they are assumed to have been met. I would also explain the purpose of the interaction plots, as whilst these are visually clear and have comments explaining what they are, it is not clearly explained how their inclusion improves our understanding of the data analysis. 

-   *If you needed to alter your partner's figure using their code, do you think that would be easy or difficult, and why?*

I think that due to the clearly defined coding chunks they have used it would be relatively easy to alter this code, particularly as there are many comments explaining the function of each line fo code. This would mean I would be able to easily identify the line of code which produces the output I am attempting to alter. 

### d) Reflect on your own code based on your experience with your partner's code and their review of yours. (300-500 words)

-   *What improvements did they suggest, and do you agree?*

I did not have a partner review my code as I was unable to find somebody who had not already shared and reviewed their code with another partner. However, when emailing Lydia France I was told this was okay and I could use a partners code to analyse myself and not have had them analyse mine!

Despite this, based on my review of my partners code there are some clear improvements I would suggest for my code: 

1. Include more comments explaining the output of my code - particularly at the beginning when cleaning the data

2. Include more comments explaining the purpose of each line of code for my ggplots - particularly for the violin plot where there are no comments explaining the codes function

3. Improve the violin plot I produced - i.e. by increasing transparency of the plot and making the output visualised more sophisticated

4. Include code which controls resolution and size dimensions of my ggplots when saved as .pngs to improve their quality when uploaded on GitHub

5. Include comments during the statistical analysis which explains the output of the models i.e. lm and ANOVA

6. Potential troubleshooting guides for any code which has potential errors/ difficulty running

7. Break the statistical analysis chunk into smaller, manageable chunks of code for each analysis step

-   *What did you learn about writing code for other people?*

I have learnt that people write code very differently and with their own personal style. This means that whilst I may think my code is clear it may actually be difficult to interpret as I am writing it with inside knowledge of why the data pipeline is structured the way it is. Therefore, it is important to explain every step of the methodology in a data pipeline to ensure that my code is reproducible and easily interpreted. This is particularly important for commenting why code is being used, using descriptive variable names with a purpose, error handling, data cleaning, setup information, and producing readable plots.
