# Module 2 Final Project

<p align='center'>
An analysis of emergency department visits and admissions to hospitals in NYC, using Department of Health and Mental Hygiene (DOHMH) data from NYC Open Data. Specifically, the emergency department visits, with regard to influenza-like or pneumonia symptoms, during the years of 2020 - 2022.
</p>

## Directory

- [Stakeholder](#stakeholder)
   - [Project Scope](#project-scope)
      - [In-Depth Analysis](#analysis)
         - [Visualizations/Charts](#visualizations)
            - [Findings in the Data](#findings)
               - [Recommendations](#recommendations)
                  - [Reflections / What's Next](#reflections)

## Stakeholder

![](https://assets.codepen.io/5102/NYC_Health-logo.png)
<p align='center'>
The NYC Department of Health and Mental Hygiene, or DOHMH, is a government agency and one of the largest public health agencies in the world. They are entrusted with promoting awareness of health by providing public services, like investigating disease spread, providing health inspections, and setting up no-cost health clinics in neighborhoods where they are most needed. 
</p>

<p align='center'>
If you would like to learn more about the NYC DOHMH, you can keep up with their website, where they provide extensive information on the health services they provide.
</p>

- [Department of Health Homepage](https://www.nyc.gov/site/doh/index.page)

## Project Scope

### Group Members

- Thierno Barry
   - [Thierno's LinkedIn](https://www.linkedin.com/in/thierno-barry-333288284/)
- Rolando Mancilla-Rojas
   - [Rolando's LinkedIn](https://www.linkedin.com/in/rolandoma33/)

### Aims
<p align='center'> 
Our focus revolved around not only cleaning and analyzing the data, but an emphasized focus on creating powerful visualizations that impact the NYC stakeholders we are presenting to.
Within the scope of the project, our stakeholders, the Department of Health and Mental Hygiene (DOHMH), have asked us to provide insights on the following questions:
</p>

1. What boroughs have the highest emergency department visits to hospital admissions ratio?
   - Specifically, regarding symptoms of influenza-like or pneumonia illnesses.

2. Which of these areas needs the most financial support?
   - Where should the DOHMH give more funding? Where should they give less?
  
<p align='center'>
With these questions in mind, it was also crucial for us to present our data in an ethical way, since we were scoping out low-income neighborhoods to find where allocated funding should go. This is a sentiment we held throughout our project, considering our strong feelings towards helping New York neighborhoods.
</p>

### Sources

- [NYC Open Dataset](https://opendata.cityofnewyork.us/)
- [Cleaned CSV](https://drive.google.com/file/d/1ByDEStAVRacADbbdEW0v1k2JspoiGVZr/view?usp=drive_link)
- [Modified ZCTA Code Tabulation Areas](https://nychealth.github.io/covid-maps/modzcta-geo/about.html)
- [Figma Project Board](https://www.figma.com/board/5VQh66Q7veSFcQa61Ak83B/M2---Stand-In---Stand-Down?node-id=30-543&t=9sVOFV8FFDlbiQKw-0)
- [Tableau Dashboard](https://public.tableau.com/app/profile/thierno.barry7757/viz/myprojectmoddashboard/Dashboard1?publish=yes)


## Analysis

<p align='center'>
On top of the usual cleaning, we decided to create new columns that would aid in creating charts on Tableau. to do this, some transformations to the data types must be done
</p> 

A simple `df['date']=pd.to_datetime(df['date'])` and `df['mod_zcta']=df['mod_zcta'].astype(str)` was done to convert our dates to the datetime type. The mod_zcta, which represents the modified ZIP Code Tabulation Area (ZCTA), was transformed to a string to prevent any automatic transformations to the ZIP code numbers when passing the csv to other readers.

<p align='center'>
After, we created a new column, Admissions/Visits, which showed the number of people hospitalized within the number of people who visited an emergency department (ED) with symptoms of flu or pneumonia. This was done by simply dividing the admissions column by the visits column.
</p>

Next, we wanted to create a borough column that assigned each ZIP code to its respective borough. To do this, we created lists that represents the range of ZIP code values for each borough, using the [Modified ZCTA Code Tabulation Areas](https://nychealth.github.io/covid-maps/modzcta-geo/about.html) table.

```c++
#matching the zipcode to the borough name
zipcodes ={
'Manhattan': list(range(10001, 10282)),
'The Bronx': list(range(10451, 10475)),
'Brooklyn': list(range(11201, 11256)),
"Queens": list(range(11004, 11110)) + list(range(11351, 11698)),
'Staten Island': list(range(10301, 10314))}

# creating a function to apply to our new column
def my_borough(zipcode):
    try:
        zip_int= int(zipcode)
    except: 
        return None
    for borough, zip in zipcodes.items():
         if zip_int in zip:
            return borough
    return 'unknown'
        
df['borough']=df['mod_zcta'].apply(my_borough) # apply function to our original data set
df_zip_date['borough']=df_zip_date['mod_zcta'].apply(my_borough) # # apply function to our grouped data set
```
<p align='center'>
Next, an important step for us to quickly assess which areas were high, moderate, or low risk was to evaluate the rate of hospitalizations, or Admissions/Visits. To do this, we ran a method that used if statements to run the ratio through inequalities to determine what the rate would be identified as. Notably, we only accounted for rates that were higher than the first quartile of ED visits, since we figured that high rates of low ED visits were too insignificant and could skew our overall data.
</p>

```c++
# assigning Risks profile ( low moderate high risks )
def assign_risk(admission_rate):
    if admission_rate['ili_pne_visits'] < 120:# first quartile of number of visit for ili/pne
        return "Low Volume _ Not Rated"
    elif admission_rate['admissions/visits']>50:
        return 'High risk'
    elif admission_rate['admissions/visits']>30:
        return 'Moderate risk'
    else :
        return 'Low risk'
df_zip_date['Risk_level']=df_zip_date.apply(assign_risk, axis=1) # created a column Risk level, useful if you need to know how severe is the admission rate for a specific zip code
```

<p align='center'>
With these new columns all created, we were ready to begin charting and analyzing key insights!
</p>

## Visualizations

<p align='center'>
The culmination of our data manipulation came with creating the visualizations, where the creation of our new columns made it much easier. The only data type change that was required was the zip codes and boroughs, which could easily be assigned when viewing the data source on Tableau. After assigning the geographical location to both zip codes and boroughs, it was only a matter of creating charts that could best represent the analyses we wanted to convey: Queens being the highest-risk of hospitalizations.
</p>

![](README-files/dashboard-pic.png)

<p align='center'>
Notable moments here are the creation of the geomap using the ZIP codes, the inclusion of KPIs to add notable numbers, and the color alignment with the NYC Health schematics.
</p>


## Findings

<p align='center'>
As a result of our analysis, there were several key insights that were revealed in our dataset. In an overall sense, we found that Queens is the most at-risk when it comes to hospitalizations from ED symptom visits. However, there were other areas in different boroughs that had high-risk areas.
</p>

In order, the top boroughs with hospitalization risks per ED symptom visits are:
1. Queens
2. Brooklyn
3. Manhattan
4. Bronx
5. Staten Island

<p align='center'>
Upon inspecting the ZIP codes across all boroughs, we found that Manhattan took many of the top 10 spots, but Queens was a notable runner-up. This was an obscure find that required deeper analysis. Manhattan, for example, had the highest rates of hospitalizations across all boroughs, but did not account for as many visits to the emergency department as the other boroughs.
</p> 

## Recommendations

<p align='center'>
As a result of our findings, we looked back at our original aims and questions:
</p>

1. Which borough had the highest rate of hospitalizations?
2. Which of these boroughs needs the most financial support?

<p align='center'>
We found that Manhattan had the highest rates of hospitalization, but Queens had more severity since it accounted for a far greater number of ED visits. Therefore, our strongest recommendation is to provide more financial support to Queens, followed by Manhattan. Upon further investigation into the ZIP codes with high hospitalization rates in Queens, we found that those areas are representative of low-income, immigrant-filled areas. These areas seem to be struggling the most and could use the help of free health clinics provided by the DOHMH.
</p>

<p align='center'>
Furthermore, our analysis reveals that, when accounting for the number of visits, Staten Island had the lowest rates of hospitalizations and was deemed a low-risk borough. With outside sources, we saw that Staten Island is particularly regarded as one of the wealthiest boroughs in NYC. With speculation, the outcome of our analysis into Staten Island could be attributed to many people being able to afford private healthcare in this borough, barring the need for additional resources from the DOHMH.
</p>

## Reflections

![](https://elements-resized.envatousercontent.com/elements-video-cover-images/files/2722bf69-babe-4345-a7e1-ccde670c7410/inline_image_preview.jpg?w=500&cf_fit=cover&q=85&format=auto&s=6fe0552e335a6f9a103dd7a1e14158c2d23ac3d0ac40e2a99fb1c2412eef25ba)

<p align='center'>
Given the little time we have to present, we had a long conversation about what insights were worth mentioning. In doing so, we called back to our commitment to ethical data analysis and presentation. As ethical data analysts, we found that it was most important for us to showcase the problems that Queens faces as an underrepresented borough. Anecdotally, we know that the top ZIP codes we looked at were in Corona, an area typically associated with low-income, immigrant families. Given this is a background we both come from, we found it most important to represent these findings.
</p>

<p align='center'>
Moving forward, we hope to continue showcasing underrepresented areas across the field in our data analytical work!
</p>

***

> [!NOTE]
> This is a fictitious scenario created by the GitHub authors for academic purposes only.
