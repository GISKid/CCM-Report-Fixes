# CCM-Report-Fixes
This repo contains custom workarounds and solutions for common issues with reporting within CCM and SalesForce. For Ontario's Contact Tracing and Case Management software.

## Purpose

Calculate the difference between the End of Isolation date and the Self Isolation Intervention end date to ensure that End of Isolation is correctly documented in an investigation for data quality.

## Problem

If time is 12 am the calculation is off by a day, therefore if the time is 12 am add one day to the calculation

Currently, when performing calculations with date/time and date fields, there are unexpected results due to the difference in the timezone/daylight savings time within Salesforce and the user profile.

Forcing the date/time to a date field, also does NOT work as time is still retained in the background within the calculated field.

For example, when End of Isolation dates == 12 AM any calculations with a date only field (end of intervention) may not work as expected.
e.g.

End of isolation = "2021-06-15, 12:00 a.m"
End date of Intervention = "2021-06-15"

Within a calculated field in salesforce: End of Isolation - End date of Intervention = -1

Expected result: 0

## Solution:

![image](https://user-images.githubusercontent.com/12127746/122117271-a8a48280-cdf4-11eb-8b8e-2beafda213f7.png)

### Assumptions

End of isolation and intervention end date occur in the same month (adding in month may add complexity if near end of month/beginning)

`IF (
  VALUE( MID( TEXT( Case.End_of_Isolation__c - 4 ), 12, 2 ) )== 4 , ((DAY( DATEVALUE(Case.End_of_Isolation__c))) - (DAY(Hospitalization__c.End_Date__c))) + 1,
  (DAY( DATEVALUE(Case.End_of_Isolation__c))) - (DAY(Hospitalization__c.End_Date__c))
  )
`

What this does is take any value that ends at 12 AM and adds a day.

    #########STEP BY STEP what each line does####################



    # get the time value from end of isolation, MID gets characters from the middle of a text string
    #starts at char 12  and selects the next 2 digits
    # minus 4 for time zone
    VALUE( MID( TEXT( Case.End_of_Isolation__c - 4 ), 12, 2 ) )

    VALUE( MID( TEXT( Case.End_of_Isolation__c - 4 ), 12, 2 ) )== 4 
    # If time is equal to 4 (the conversion that happens within CCM calculations 12am == 4 )
    #then do the following calculation

    #Get the day from the DDTM value - must convert to DATEVALUE first
    ((DAY( DATEVALUE(Case.End_of_Isolation__c))) 

    #Get the day for the end date for the intervention

      DAY(Hospitalization__c.End_Date__c)

    # all together

      #Calculate the difference between the end date and end of isolation date for those that have a time of 12 am, add one day to these values to account for time issue  
    IF (VALUE( MID( TEXT( Case.End_of_Isolation__c - 4 ), 12, 2 ) )== 4 , ((DAY( DATEVALUE(Case.End_of_Isolation__c))) - (DAY(Hospitalization__c.End_Date__c)))+1

    # If the time is not 12 am then do regular subtraction
    DATEVALUE(Case.End_of_Isolation__c))) - (DAY(Hospitalization__c.End_Date__c)
