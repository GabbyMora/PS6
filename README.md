/***********************//////////Gabby Mora////////////***********************/

                         /////////*Problem Set 6*//////
                         ////////*Data Management*/////
                         //////////*Fall 2017*/////////

/*------------------------------------------------------------------------------
Research Questions
1- Does socio-economic status affect the number of animal abuse incidents in NYC?
2- Is there a connection between socio-economic status and other demographic items
and the number of animal abuse incidents in NYC?
3- Is there a difference between male and female when it comes to the severity
of animal abuse incidents in NYC?
4- What are the main factors affecting the severity of animal abuse incidents
in NYC?
------------------------------------------------------------------------------*/


/******************************************************************************
--------------------------------HYPOTHESIS-------------------------------------
********************************************************************************
As research in animal abuse points to an increase in incidents in areas of low
income and low education, this analysis of data from New York City should show
that socio-economic status is the main predictor for animal abuse.
With the exception of the Demographics and Animal Abuse datasets, the datasets
used in this research were selected as proxys for low socio-economic status as
areas with access to gardens and farmers markets tend to be areas of higher
socio-economic status. Furthermore, the variables connected to available financial
options found in the HIV Testing Centers dataset serve as a proxy income as it 
is expected that HIV Testing Centers found in areas of lower socio-economic status
will offer more financial options.
For this PS, I only used the Demographics dataset because it was the merge that
worked the best and it provided me with a socio-economic related variable as it 
included the nummber of people who received government assistance, which is an
indicator of socio-economic status.
*******************************************************************************/


/*******************************************************************************
----------------------------------DATASETS--------------------------------------
Please note that all my datasets for this research came from the NYC Open Data
website https://opendata.cityofnewyork.us/
*******************************************************************************/


/*------------------------------------------------------------------------------
********************************************************************************
--------------------------------------------------------------------------------
                       CLEAN UP AND RESHAPING MY DATASETS
--------------------------------------------------------------------------------
********************************************************************************
------------------------------------------------------------------------------*/

import excel "https://docs.google.com/uc?id=0BywXSn44t-HlS19LS0Z1cnVoQkE&export=download", firstrow clear
 
/*______________________________________________________________________________
                            ANIMAL CRUELTY DATASET
______________________________________________________________________________*/

drop UniqueKey CreatedDate ClosedDate Agency IncidentAddress StreetName CrossStreet1 CrossStreet2 IntersectionStreet1 IntersectionStreet2 AddressType Landmark Status DueDate ResolutionActionUpdatedDate CommunityBoard Borough XCoordinateStatePlane YCoordinateStatePlane ParkFacilityName ParkBorough SchoolName SchoolNumber SchoolRegion SchoolCode SchoolPhoneNumber SchoolAddress SchoolCity SchoolState SchoolZip SchoolNotFound SchoolorCitywideComplaint VehicleType TaxiCompanyBorough TaxiPickUpLocation BridgeHighwayName BridgeHighwayDirection RoadRamp BridgeHighwaySegment GarageLotName FerryDirection FerryTerminalName Latitude Longitude Location

edit 
 
tab Descriptor

/*This helped me see the number of animal abuse incidents per severity, which I
will use later to collapse into 3 categories according to the level of severity

               Descriptor |      Freq.     Percent        Cum.
--------------------------+-----------------------------------
                  Chained |         40        6.31        6.31
                   In Car |         38        5.99       12.30
                Neglected |        292       46.06       58.36
               No Shelter |         23        3.63       61.99
Other (complaint details) |        149       23.50       85.49
                 Tortured |         92       14.51      100.00
--------------------------+-----------------------------------
                    Total |        634      100.00
*/


encode Descriptor, generate(Descriptor_n)

move Descriptor_n Descriptor

drop Descriptor

rename Descriptor_n Descriptor

tab LocationType

/*This helped me see how many different types of locations were included in the 
dataset ad lead me to combine all residential areas

             Location Type |      Freq.     Percent        Cum.
---------------------------+-----------------------------------
                Commercial |          2        0.32        0.32
           House and Store |          3        0.47        0.79
           Park/Playground |         14        2.21        3.00
      Residential Building |         18        2.84        5.84
Residential Building/House |        379       59.78       65.62
          Store/Commercial |         52        8.20       73.82
           Street/Sidewalk |        166       26.18      100.00
---------------------------+-----------------------------------
                     Total |        634      100.00

*/

describe LocationType

replace LocationType = subinstr(LocationType, "/", "",.)

generate Location=0
//ResPrivate=1, Public=2, NonResidential=0//

replace Location=0 if LocationType=="StoreCommercial" | LocationType=="House and Store" | LocationType=="Commercial" 

replace Location=1 if LocationType=="Residential Building" | LocationType=="Residential BuildingHouse"

replace Location=2 if LocationType=="StreetSidewalk" | LocationType=="ParkPlayground"

encode LocationType, generate(LocationType_n)

move LocationType_n LocationType

drop LocationType

rename LocationType_n LocationType

rename IncidentZip ZipCode

save Animal_Abuse_COMPLETE, replace

/*______________________________________________________________________________
                          HIV TESTING CENTERS DATASET
______________________________________________________________________________*/
clear

import excel "https://docs.google.com/uc?id=0BywXSn44t-HlSmdVS2swdU4ycVE&export=download", firstrow clear

drop SiteID AgencyID Address City Borough State BriefDescription AgesServed GendersServed RequiredDocuments Website PhoneNumber BuildingFloorSuite AdditionalInformation

drop HoursMonday HoursTuesday HoursWednesday HoursThursday HoursFriday HoursSaturday HoursSunday Intake OtherInsurances SiteLanguages

encode ZipCode, generate(ZipCode_n)

drop ZipCode

rename ZipCode_n ZipCode

replace ZipCode=0 if ZipCode==.

save HIV_Testing_Centers_Clean, replace

/*______________________________________________________________________________
                          DEMOGRAPHICS DATASET
______________________________________________________________________________*/

clear

import excel "https://docs.google.com/uc?id=0BywXSn44t-HlMURzMmVjUExDRjg&export=download", firstrow clear

rename JURISDICTIONNAME ZipCode

drop COUNTGENDERUNKNOWN PERCENTGENDERUNKNOWN PERCENTFEMALE PERCENTMALE COUNTGENDERTOTAL PERCENTGENDERTOTAL PERCENTPACIFICISLANDER PERCENTHISPANICLATINO PERCENTAMERICANINDIAN PERCENTASIANNONHISPANIC PERCENTWHITENONHISPANIC PERCENTBLACKNONHISPANIC PERCENTOTHERETHNICITY PERCENTETHNICITYUNKNOWN COUNTETHNICITYTOTAL PERCENTETHNICITYTOTAL COUNTPERMANENTRESIDENT PERCENTPERMANENTRESIDENT COUNTUSCITIZEN PERCENTUSCITIZEN COUNTOTHERCITIZENSTATUS PERCENTOTHERCITIZENSTATUS COUNTCITIZENSTATUSUNKNOWN PERCENTCITIZENSTATUSUNKNOWN COUNTCITIZENSTATUSTOTAL PERCENTCITIZENSTATUSTOTAL PERCENTRECEIVESPUBLICASSISTAN PERCENTNRECEIVESPUBLICASSISTA PERCENTPUBLICASSISTANCEUNKNOW PERCENTPUBLICASSISTANCETOTAL

rename(COUNTPARTICIPANTS COUNTFEMALE COUNTMALE COUNTPACIFICISLANDER COUNTHISPANICLATINO COUNTAMERICANINDIAN COUNTASIANNONHISPANIC COUNTWHITENONHISPANIC COUNTBLACKNONHISPANIC COUNTOTHERETHNICITY COUNTETHNICITYUNKNOWN COUNTRECEIVESPUBLICASSISTANCE COUNTNRECEIVESPUBLICASSISTANC COUNTPUBLICASSISTANCEUNKNOWN) (TotalParticipants Female Male PacificIslander HispanicLatino AmericanIndian Asian White AfricanAmerican OtherEthnicity EthnicityUnknown PublicAssistance NoPublicAssistance PublicAssistanceUnknown)

save Demographics_Clean, replace

/*______________________________________________________________________________
                           FILMING PERMITS DATASET
______________________________________________________________________________*/

clear

import excel "https://docs.google.com/uc?id=0BywXSn44t-HlWnhZVkFzN1pNZ28&export=download", firstrow clear

drop EventID StartDateTime EndDateTime EnteredOn CommunityBoards Country ParkingHeld EventAgency SubCategoryName 

rename ZipCodes ZipCode

gen newzip=substr(ZipCode, 1, 5)

gen newzip2=substr(ZipCode, -5, 5) 

drop ZipCode

rename newzip ZipCode

drop newzip2

encode ZipCode, generate(ZipCode_n)

drop ZipCode

rename ZipCode_n ZipCode

save Filming_Permits_Clean, replace

/*______________________________________________________________________________
                              GARDENS DATASET
______________________________________________________________________________*/

clear

import excel "https://docs.google.com/uc?id=0BywXSn44t-HlRGQ5ZC1aYW9nYzA&export=download", firstrow clear

drop PropID Boro CommunityBoard CouncilDistrict CrossStreets Latitude Longitude CensusTract BIN BBL NTA

rename Postcode ZipCode

encode ZipCode, generate(ZipCode_n)

drop ZipCode

rename ZipCode_n ZipCode

replace ZipCode=0 if ZipCode==.

save Gardens_Clean, replace

/*______________________________________________________________________________
                          FARMERS MARKET DATASET
________________________________________________________________________________*/

clear

import excel "https://docs.google.com/uc?id=0BywXSn44t-HlTUhfR3ZQQ3hyZlE&export=download", firstrow clear

drop AdditionalInformation Latitude Longitude

save Farmers_Markets_Clean, replace

/*------------------------------------------------------------------------------
********************************************************************************
--------------------------------------------------------------------------------
                GETTING MY DATA READY FOR CROSS DATASET ANALYSIS
--------------------------------------------------------------------------------
********************************************************************************
------------------------------------------------------------------------------*/

clear

use "Demographics_Clean"

merge 1:m ZipCode using "Animal_Abuse_Complete" 

drop if Descriptor==.

drop _merge


recode Descriptor (6=6) (1=5) (2=4) (3=2) (4=3) (5=1), gen (Severity) 
/*I recoded so I could analyze the level of abuse (Descriptor) according to the
severity of the issue with 1 being the least severe and 6 the most: Other, 
Neglected, No Shelter, In Car, Chained, Torture).*/


save "Animal_Abuse_COMPLETE", replace


/*------------------------------------------------------------------------------
********************************************************************************
--------------------------------------------------------------------------------
                        ANALIZING MY DATA WITH GRAPHS
--------------------------------------------------------------------------------
********************************************************************************
------------------------------------------------------------------------------*/

hist Location, freq

/*This graph helped me quickly see that the majority of the animal abuse incidents
in the dataset ocurred in private/residential locations. I had previously recoded
all residential locations to be represented by 1 because I believe that abuse in
residential locations would mean that people are doing it in private while abuse
in the street or sidewalks would be done in public*/

hist Severity, freq

/*This graph showed me that Other is the most common incident of animal
abuse in NYC, with Neglected being the second most common. Unfortunately, no data
was found in reference to what "Other" applied to. The third most common incident
was Torture, which was surprising as I did not expect that many people to still
engage in torture of dogs. This lead me to want to look more into the level of
education and socio-economic status associated with these cases but I did not 
have the data on educational level so I will only focus on socio-economic status*/

/*------------------------------------------------------------------------------
********************************************************************************
--------------------------------------------------------------------------------
                          DESCRIPTIVE STATISTICS
--------------------------------------------------------------------------------
********************************************************************************
------------------------------------------------------------------------------*/

clear

/*I became interested in looking at what are of NYC had the largest amount of
animal abuse incidents, so I did a tabulation to quickly see the number.*/

use "Animal_Abuse_COMPLETE"

/*Based on earlier findings of the commonality of each animal abuse incident by
severity, I did a regression between socio-economic status (where I used Public
Assistance as a proxy)*/

tab Severity

tab PublicAssistance

regress Severity PublicAssistance

est sto SPA

/*Since the t value is below p=0.05 and positive, Public Assistance does not have a 
significant impact on the increase of severity of the animal abuse incident*/


/*I went back to my research questions and did a regression between the descriptor
and Public Assistance and Race. I will then crosstabulate race and
socio-economic status to see if there is a correlation*/

tab PacificIslander

tab HispanicLatino

tab AmericanIndian

tab Asian

tab White

tab AfricanAmerican

regress Severity PublicAssistance PacificIslander HispanicLatino AmericanIndian Asian White AfricanAmerican

est sto SPAR

/*Though the coefficient shows that Public Assistance has a negative impact on the 
increaseof the severity of the animal abuse cases, the P value tells me that this 
is not significant as it cannot only be traced to Public Assistance*/

/*After running these regressions I looked back at the dataset and realized that
PublicAssistance included the number of people who received public assistance
in each observation, so I recoded and generated a new Variable that would only
show me if people received it or not*/

recode PublicAssistance (1 2 3 = 1), gen (ReceivesPA)

tab ReceivesPA

regress Severity ReceivesPA

est sto SR

/*No significant results were found as the t value was above 0.05*/

save "Animal_Abuse_COMPLETE", replace

table Descriptor PublicAssistance

table White Asian HispanicLatino AmericanIndian AfricanAmerican PublicAssistance

table ZipCode ReceivesPA

table ReceivesPA Severity
/*This table actually helped me see that the amount of people receiving public
assistance had less cases of animal abuse than those receiving public
assitance, which completely went against my hypothesis*/

ssc install outreg

regress ZipCode PublicAssistance
outreg using SR, replace 

regress White ZipCode PublicAssistance
outreg using SR, replace

regress Asian ZipCode PublicAssistance 
outreg using SR, replace

regress HispanicLatino ZipCode PublicAssistance
outreg using SR, replace

regress AmericanIndian ZipCode PublicAssistance
outreg using SR, replace

regress AfricanAmerican ZipCode PublicAssistance
outreg using SR, replace

regress White Asian HispanicLatino AmericanIndian AfricanAmerican ZipCode PublicAssistance
outreg using SR, replace

regress Male ZipCode PublicAssistance
outreg using SR, replace

regress Female ZipCode PublicAssistance
outreg using SR, replace

regress Male Female ZipCode PublicAssistance
outreg using SR, replace

regress Male Female White Asian HispanicLatino AmericanIndian AfricanAmerican ZipCode PublicAssistance
outreg using SR, replace 

regress ZipCode ReceivesPA
outreg using SR, replace 

regress White ZipCode ReceivesPA
outreg using SR, replace

regress Asian ZipCode ReceivesPA
outreg using SR, replace

regress HispanicLatino ZipCode ReceivesPA
outreg using SR, replace

regress AmericanIndian ZipCode ReceivesPA
outreg using SR, replace

regress AfricanAmerican ZipCode ReceivesPA
outreg using SR, replace

regress White Asian HispanicLatino AmericanIndian AfricanAmerican ZipCode ReceivesPA
outreg using SR, replace

regress Male ZipCode ReceivesPA
outreg using SR, replace

regress Female ZipCode ReceivesPA
outreg using SR, replace

regress Male Female ZipCode ReceivesPA
outreg using SR, replace

regress Male Female White Asian HispanicLatino AmericanIndian AfricanAmerican ZipCode ReceivesPA
outreg using SR, replace 

/*I was hoping that these regressions and outreg would help me better understand
the data so I could see the correlations. However, the same results as earlier 
were found and it is not possible to say that any of these variables have an
impact on animal abuse, though some of them do show that they increase the severity
of animal abuse incidents*/

/*I decided to try a couple more statistical tests to make sure there was no 
relationship between the variables*/

corr ReceivesPA Descriptor

corr ReceivesPA Severity

corr ZipCode ReceivesPA 

corr White Severity

corr HispanicLatino Severity

/*Since I got the same results with the correlations (no significance), I decided
to not run any other correlations because I expected the same results as the 
regressions*/


/*------------------------------------------------------------------------------
********************************************************************************
--------------------------------------------------------------------------------
                                  CONCLUSIONS
--------------------------------------------------------------------------------
********************************************************************************
--------------------------------------------------------------------------------
The dataset Animal Abuse showed a significant number of animal abuse cases in NYC.
The variables offered in that database did not offer explanations or reasons why
the number of cases was distributed in such way. In attempting to find the possible
reasons for the number of animal abuse cases as well as the severity of the cases,
I tried to look at the correlation of different demographic data points and the
animal abuse cases.
In PS6, I tried to get a better understanding of each demographic group (that is
the reason I ran a tab before every regression) so I could see the individual 
numbers before looking at the statistical analysis. This did not provide any
further clues regarding the issue of animal abuse incidents and severity according
to demographic group.
I think that looking at this type of data is a helpful tool for beginning analysis
but after thinking about it, in combination with the discussion we had about the 
East Village and law enforcement, I believe that in order to get an accurate
picture of the factors that affect the number of animal abuse incidents as well
as the severity of the cases, a review of the characteristics of each zip code,
including education levels, law enforcement resources, and number of dogs per
zip code is paramount to create a complete picture that may help in future
research to understand the reasons behind animal abuse IF animal abuse is connected
to items that can be quantified.
I would like to go back and dig deeper into the East Village and see why there
are so many incidents of animal abuse in that zip code. It is possible that
learning more about this neighborhood will help me develop other hypotheses
or guide me into what data to look at in order to try to find an answer to the
issue of animal abuse*/

