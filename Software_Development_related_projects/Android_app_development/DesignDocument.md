# Design Document
**Author**: Team 044: Jeff Clem, Samantha Ho, Hannah Morilak, Yuan Yang

---
**This is the Version 2 of the design document: modified Version1 during D3 construction phase based on better understanding of the app.**

What are modified:
* Modified Class Diagram to reflect the actual design of app during implementation.
  1. Entry class name is changed to "appDatabase" because we implemented most functions in this class.
  2. Added extra functions in "appDatabase" and "ComparisonSetting" class to fulfill the functions needed for realizing app functions.
  3. Utility classes are deleted because they are not necessary in this specific app.
  4. Changed some data types. 

Please refer to commit ID we submitted for D2 for Version 1.

---


## 1 Design Considerations

### 1.1 Assumptions
* The app will be deployed and run in an android environment
* The app will be created using API 29 & Android 10.0
* The app will be limited to one user, per deployment, and all its functionality will be limited to only one user. There will be no individual details of each user
    * This doesn't create any issues, but it lowers scale and removes any elements of authentication.
* The app can enter and edit job instances but not remove them.
    * This seems like a large missing functionality or potential issue later on.

### 1.2 Constraints

* The app state should persist between runs. This can be accomplished by leveraging Android SQLite.
* The app only developed to run in an Android environment.

### 1.3 System Environment

* The minimum API level for the app should be “API 29: Android 10.0 (Q)”. 
* Emulator to run this product will be running the software with the above minimum API level in Android Studio. 
* The app will only be tested on a Pixel XL4 virtual machine.

## 2 Architectural Design

### 2.1 Component Diagram
As shown in the designed component diagram below, three user interface components uses services of Job and Comparison Setting components.
* Job Entry/Modification UI access the data in Job component: input/edit data in Job.
* Setting Modification UI access the in Comparison Setting component: edit data Comparison Setting.
* Job Comparison UI uses Job and Comparison Setting: compare two job objects based on settings.

Moreover, the data in Job and Comparison Setting components are persisted using services provided by Persistence infrastructure. The persistence infrastructures rely on AppDB component to provide services.

![ComponentDiagram](../images/Diagrams/ComponentDiagram.png)

### 2.2 Deployment Diagram
As shown in the deployment diagram below, all the components are deployed on an Android device.

![DeploymentDiagram](../images/Diagrams/DeploymentDiagram.png)

## 3 Low-Level Design

### 3.1 Class Diagram

The class diagram shown below is the diagram that team designed during D1 with no modification. Please refer to [ClassDiagramDescription](../Design-Team/design-description.md) for the description of the class diagram. Moreover,the Job component in Component Diagram consist of Job class in Class diagram, while the Comparison Setting component consist of ComparisonSetting class.

This is the version 2, please refer to description in the figure for explanation of revision: (Please refer to commit ID we submitted for D2 for Version 1.)

![ClassDiagram](../images/Diagrams/design_ver2.png)

### 3.2 Other Diagrams (Sequence Diagram)
As shown in the figure below, the Sequence Diagram is designed to describe behaviors of the system. 

![SequenceDiagram](../images/Diagrams/SequenceDiagram.png)

## 4 User Interface Design
When creating the User Interface, we wanted to keep the design simple and intuitive. 
Our Design includes 5 main pages which can be found below.

Main Menu:

![MainMenu](../images/Design-Team/main-menu.png)

Enter/Edit Current Job Page:

![CurrentJobPage](../images/Design-Team/current-job.png)

Enter New Job Page:

![NewJobPage](../images/Design-Team/new-job.png)

Settings Page:

![SettingsPage](../images/Design-Team/settings.png)

Comparison Page:

![ComparePage](../images/Design-Team/compare-page.png)
