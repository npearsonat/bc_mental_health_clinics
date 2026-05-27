# BC Mental Health Clinic Access Analysis

This project examines publicly available mental health clinic data alongside 2021 Census 
demographics to identify gaps in mental health service accessibility across British Columbia. 
My key findings involve the language availability gaps present in certain dissemination areas. 

![BC Vulnerability](graphics/vuln_score_bc.png)

# Data

The data used in this project is all publicly available and open source. British Columbia does not publicise their patient health data beyond broad statistics, so I worked with census data geographic based clinic data in order to draw connections between the people these clinics are meant to serve and the accessibility of their services.

**mental-health.xlsx:** Source clinic data from HealthLink BC's Mental Health and Substance Use (MHSU) directory, downloaded from the BC Data Catalogue. Contains ~4,200 service listings across BC including service name, organization, taxonomy classification, audience, languages offered, wheelchair accessibility, address, and coordinates.

**census_da_bc_2021.xlsx:** Intermediate dataset of 2021 Census profile variables extracted at the dissemination area level for BC. Generated from the raw Statistics Canada census profile file below.

**2021_92-151_X.xlsx Statistics Canada Geographic Attribute File (2021):** Contains DA-level centroid coordinates and urban/rural population centre classification.

**98-401-X2021006_English_CSV_data_Britis...xlsx:** Raw 2021 Census Profile data for BC at the dissemination area level, downloaded from Statistics Canada. ~3.5GB — not included in repository due to size.

**lda_000b21a_e/ 2021:** Cartographic Boundary File for dissemination areas, downloaded from Statistics Canada. Shapefile format, clipped to coastline.

# Available Geographic Information

Dissemination areas are the smallest standard geographic units for which Statistics Canada publishes census data. They are geographic areas containing 400-700 people, and Statistics Canada publishes aggregate statistics about individuals living in these areas. These include things such as income, immigration status, languages spoken and age. The basic assumption of this project is that we can use this information as a substitute for more specific information about the patients who visit these clinics. But there is also a benefit to using this data: we can find out information about people who live near mental health clinics even if they do not visit them. 

The mental health clinic geographic data is used to pinpoint the location of mental health clinics in communities and compare their location to dissemination areas or DAs. I create a radius around DAs depending on their size, urban/suburban categorization, and shape, in order to find DAs that have less access than others. 

# Methodology

## Vulnerability Index

The vulnerability index is a composite score designed to identify dissemination areas 
whose populations are at elevated risk of requiring mental health services. It is constructed 
from 16 census variables spanning socioeconomic status, demographics, housing, and social 
isolation, each converted to a percentage of the DA population:

- **Age:** Youth (15-24) and seniors (65+)
- **Income:** Median after-tax income, low income prevalence (LIM-AT), shelter cost burden (>30% of income)
- **Housing:** Unsuitable housing, major repairs needed, renting
- **Employment:** Unemployment rate
- **Education:** No high school diploma
- **Social isolation:** Living alone, single parent families
- **Identity:** Indigenous identity, visible minority, immigrants, non-official language speakers, recent movers

Each variable is normalized to a 0-1 scale using min-max normalization and averaged equally 
across all 16 variables, producing a score between 0 and 1. A higher score indicates greater 
estimated need for mental health services. Equal weighting was chosen for transparency and 
reproducibility, though future work could apply evidence-based weights.

## Clinic Access Methodology
Rather than using a fixed radius to measure clinic accessibility, this project assigns each 
DA a variable search radius based on its urban/rural classification and physical size:

| Classification | Base Radius |
|---------------|-------------|
| Large Urban (100,000+) | 10km |
| Medium City (30,000-99,999) | 15km |
| Small Town (1,000-29,999) | 20km + DA radius |
| Rural | 40km + DA radius |

The DA radius is calculated as the radius of a circle with equivalent area to the DA 
(`√(area/π)`). This accounts for the fact that rural DAs can cover hundreds of square 
kilometres, making centroid-based distance measurements unreliable. A minimum of 40km 
is applied to all rural DAs regardless of size, reflecting realistic travel distances 
to services in rural BC.

For each DA, the number of clinics within its search radius is counted using a spatial 
buffer join in GeoPandas projected to BC Albers (EPSG:3153) for accurate distance measurement.

# Graphics

## Vulnerability Score

| BC Province | Metro Vancouver |
|-------------|-----------------|
| ![BC Vulnerability](graphics/vuln_score_bc.png) | ![Vancouver Vulnerability](graphics/vuln_score_vancouver.png) |

You'll find that there is quite an even spread of areas with high vulernability score all accross BC. They are not limited to areas of low income, nor do they subscribe to a particularlty strong geographic pattern. There are many high risk areas near downtown Vancouver, Richmond and Surrey. However, these areas are often very well covered compared to other areas of the Province. 

## Highly Vulnerable DAs With Few Nearby Clinics

![BC Vulnerability](graphics/bc_no_clinics.png)

These are the DAs with high vulnerability score but do not have any nearby clinics. The clinics are identified by a blue dot. The calculation is based on how rural they are, as well as the size of the DA. You'll see in this map that many cities are very well covered, even with a reduced search radius of around 10km. The areas where mental health clinics are not in close proximity are up north in very sparsely populated regions of the province.

## DAs With Gaps in Language Coverage

![BC Vulnerability](graphics/van_lang_gap.png)

### Language Access Gap Methodology
To identify communities underserved by language-matched mental health services, this analysis 
cross-references 2021 Census mother tongue data with the languages offered by nearby clinics.

For each of the 10 most common non-English languages in the clinic dataset (Punjabi, Mandarin, 
Cantonese, Tagalog, Spanish, Korean, Vietnamese, Hindi, Arabic, and Farsi) DAs where more 
than 15% of residents speak that language as their mother tongue are identified. The number 
of nearby clinics offering that language is then counted within each DA's variable search radius.

DAs are flagged as underserved when their clinic count for a given language falls below the 
25th percentile of all DAs with a significant speaker population for that language. This 
relative threshold accounts for the fact that urban areas have far more clinics overall.
A DA in Surrey with 13 Punjabi clinics nearby may still be underserved relative to its 
large Punjabi-speaking population.

Mother tongue was used rather than total language knowledge, as it is a stronger indicator 
of preference for language-matched mental health services. Research consistently shows that 
therapy delivered in a patient's first language produces significantly better outcomes.

| White Rock | Langley |
|-------------|-----------------|
| ![BC Vulnerability](graphics/white_rock_mandarin.png) | ![Vancouver Vulnerability](graphics/langley_korean.png) |

White Rock has a high population of Mandarin speakers with 0 clinics within 10km. Langley similarly has up to 18% Korean DAs with 0 clinics within 10km.


| Prince George| Kelowna |
|-------------|-----------------|
| ![BC Vulnerability](graphics/prince_george_punjabi.png) | ![Vancouver Vulnerability](graphics/punjabi_kelowna.png) |

Prince George and Kelowna have high populations of Punjabi speakers, yet they both seem to have 0 clinics within either 15km or 40km of a few DAs. These ranges were estimated based on urban/rural classification. 

# Recommendation

My recommendation is to investigate areas of oversaturation and possibly transfer practitioners to mental health clinics where they are needed. For example, Vancouver is a very well covered area of BC. There are many areas of the city where there is lots of access to clinics in a multitude of languages. I think some clinitions or translators should be moved to White Rock, or these areas should recieve an expansion in their multilanguage offerings.
