# Use Case Model


**Author**: Team 044: Jeff Clem, Samantha Ho, Hannah Morilak, Yuan Yang

## 1 Use Case Diagram

![UseCaseDiagram](../images/Design-Team/UseCaseModel.png)

## 2 Use Case Descriptions


#### Enter Current Job

- Requirements: The case allows the user to enter information for a current job, when he has not done so previously
- Pre-conditions: There is no job object saved into JobsList that is marked as Current Job (isCurrentJob==True in any job object)
- Post-conditions: There is one job object saved into JobsList that is marked as Current Job, and one only (isCurrentJob==True in one job object only) with input member fields 
- Scenarios: The use case will first prompt the user to enter fields for a current job. Then:
  - User can choose to return to main menu without saving
  - User can choose to input information into the fields:     
    - 1.) the input fields are of correct type and bounds, the app will show the Current Job details that have been saved and allow the user to return to main menu 
    - 2.) the input fields are not of correct type and/or bounds, the GUI expresses these errors, the user can choose to return to main menu or re-enter input fields. 
       - If the user re-enters correct input fields, then app will show Current Job details that have been saved and allow user to return to main menu
       - If the user re-enters incorrect input fields, the cycle from sub-scenario 2 restarts
       - If the user chooses to cancel, the app will return to the main menu
- Exceptional Scenario: (Enter Job Rejected Use Case)
  - if there is a job object marked as Current Job then the GUI will display corresponding message and allow user to return to main menu to edit Current Job

#### Edit Current Job

- Requirements: The case allows the user to edit information for a current job
- Pre-conditions: There is one job object saved into JobsList that is marked as Current Job (isCurrentJob==True in one job object)
- Post-conditions: There is one job object saved into JobsList that is marked as Current Job, and one only (isCurrentJob==True in one job object only) with the updated member fields 
- Scenarios: The use case will first prompt the user to edit fields for their current job. Then:
  - User can choose to return to main menu without saving
  - User can choose to input information into the fields:     
    - 1.) the input fields are of correct type and bounds, the app will show the Current Job details that have been saved and allow the user to return to main menu 
    - 2.) the input fields are not of correct type and/or bounds, the GUI expresses these errors, the user can choose to return to main menu or re-enter input fields. 
       - If the user re-enters correct input fields, then app will show Current Job details that have been saved and allow user to return to main menu
       - If the user re-enters incorrect input fields, the cycle from sub-scenario 2 restarts
       - If the user chooses to cancel, the app will return to the main menu
- Exceptional Scenario: (Edit Job Rejected Use Case)
  - if there is no job object marked as Current Job then the GUI will display corresponding message and allow user to return to main menu to enter Current Job

#### Enter Job Offer

- Requirements: The case allows the user to enter information for one job object
- Pre-conditions: None
- Post-conditions: There is one job object saved into JobsList that is not marked as Current Job (isCurrentJob==False) with the input member fields 
- Scenarios: The use case will first prompt the user to edit fields for this job offer. Then:
  - User can choose to return to main menu without saving
  - User can choose to input information into the fields:     
    - 1.) the input fields are of correct type and bounds, the app will show the Job Offer details that have been saved and allow the user to return to main menu 
    - 2.) the input fields are not of correct type and/or bounds, the GUI expresses these errors, the user can choose to return to main menu or re-enter input fields. 
       - If the user re-enters correct input fields, then app will show Job Offer details that have been saved and allow user to return to main menu
       - If the user re-enters incorrect input fields, the cycle from sub-scenario 2 restarts
       - If the user chooses to cancel, the app will return to the main menu

#### Adjust Comparison Weights

- Requirements: Allows user to change member fields of the weights class
- Pre-conditions: None 
- Post-conditions: Weights class member fields updated to input fields 
- Scenarios: The use case will first prompt the user to Adjust Comparison Weights. Then:
  - User can choose to return to main menu without saving
  - User can choose to input information into the fields:     
    - 1.) the input fields are of correct type and bounds, the app will show the weight details that have been saved and allow the user to return to main menu 
    - 2.) the input fields are not of correct type and/or bounds, the GUI expresses these errors, the user can choose to return to main menu or re-enter input fields. 
       - If the user re-enters correct input fields, then app will show comparison weight details that have been saved and allow user to return to main menu
       - If the user re-enters incorrect input fields, the cycle from sub-scenario 2 restarts
       - If the user chooses to cancel, the app will return to the main menu

#### Compare Two Jobs

- Requirements: Allows user to select 2 job objects from database of jobs and display them side by side 
- Pre-conditions: There are at least two job objects within JobList
- Post-conditions 2 chosen job objects are displayed side by side in GUI
- Scenarios: 
  - If pre-condition is met
    - App will display the entire list of ranked job objects
    - User can:
      - 1.) Return to main menu without comparing 
      - 2.) Choose two job objects to compare
          - the app will display the two jobs chosen side by side in the UI
          - then the user can choose to return to the main menu
    
- Exceptional Scenario: If pre-condition is not met, then the GUI will display an error (extended Comparison Rejected Use Case) and allow user to return to main menu


