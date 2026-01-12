<h1 align='center'>Modelling the Abundance of UK Butterflies</h1>

## Project Background
### Context
As part of a new government-led green initiative to prevent biodiversity loss, regional councils are to receive a ‘nature grant’ that will be allocated for habitat restoration and species protection. This comes after a report in the State of Nature revealed the number endangered species in Great Britain had risen to 16% [1]. Many experts believe this is likely the result of environmental pressures such as, climate change, air pollution, agricultural pollution and invasive species [1].
### The Goal
It is well known that butterflies are excellent indicators of terrestrial biodiversity, due to the diverse range of habitats they occupy, and their sensitivity to environmental changes. To help regional councils decide how they should allocate their grant, the UK Data for Wildlife Conservation (UKDWC) has been commissioned to produce a report detailing the abundance trends of butterflies on a national scale. The report will highlight species of butterfly in decline and produce a list of ‘recommended actions’ that regional decision makers can use to focus their budgets accordingly. 
### About the UKDWC
The UKDWC is a non-profit environmental research organisation, that monitors biodiversity and ecosystems in the UK, reporting its findings to various charities and central government. Since its founding in 1963, the company has gained a reputation for its excellent ecological modelling capabilities and data-centric environmental reports. 

<p align='center'><image src='images/project_background/logo.png' width=300></p>

## Data Sourcing & Project Data
Data for the project was acquired by accessing the UKBMS (UK Butterfly Monitoring Scheme) ecological survey data, made available by the UKCEH (UK Centre for Ecology and Hydrology). Two datasets were used: ‘site indices 2023’, and ‘site location data 2023’. Before, the analysis, both datasets were cleaned and joined in Python to form the main project dataset ‘UKBMS_cleaned_v1’. 

<p align='center'><image src='images/project_background/data_structure.png' width=500></p>

The code used to clean the original datasets is available here:
ukbms_site_location_cleaning
ukbms_site_indices_cleaning
join_indices_location

## Executive Summary
In this project, UKBMS survey data is used to model the relative abundance of UK butterflies over the last 30 years. 
Of the 17 species analysed, nearly a third are found to be in long term decline, with 24% showing both short- and long-term decline. The Essex Skipper and Small Skipper have been identified as ‘high risk’, with population levels falling by 90% since 1993. Another struggler, the Small Tortoiseshell has declined at more than 5%/yr since 1993 and is forecast to become ‘high risk’ in 2037 if long term trends continue. Possible causes are found to be agricultural pollution, climate change and the influence of invasive species. The remainder of this report will highlight key metrics in the worst performing species, the implications and possible solutions. 
The figure below illustrates the performance of all butterfly species over the last 10-30 years. 

<p align='center'><image src='images/executive_summary/long_term_vs_short_term.png' width=956></p>


