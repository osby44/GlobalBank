# GlobalBank Dashboard Project

## Background
This is me stepping into the role of a freelance consultant tasked with creating a dashboard to help with decision-making around mortgage applications for a network of bank branches. 

## Problem statement

Employees at Global Bank have noticed that:
1.	Mortgage-application files are of varying quality across their network of branches.
2.	The length of time it takes to make a decision on whether to approve or reject a mortgage application is far too long

## Project goals
1.	To reduce the time it takes to decide whether to approve mortgage applications. 
2.	To improve the quality of approved application files

## Page Layout
We’ll set up several pages that respond to user needs:

•	A Home page with instructions for use.

•	A Mortgage Application page for analysis indicators 

•	A Branch Performance page for analysing branch performance. 

•	A Customer Indicators page to monitor the make-up of the borrower portfolio and to zoom in on individual borrowers’ (or potential borrowers’) profiles.

•	A “Show customer list” page to display a list of all the customers for easy view

## Here’s a summary of what was covered:

### Mortgage Application Page

***Visualization 1***

Global Bank wants to visualize past real-estate transactions. This is an indicator of the bank’s commercial activity, as the more customers you attract, the more mortgage applications you’ll get. A column chart is used here to visualize the loan transaction amounts over the years.

***Visualization 2***

Global Bank’s strategy is to focus their activity on customers with a better profile and on larger transactions, so fewer loans but higher amounts. The task here is to visualize how the average loan size has evolved over the years. Added a line chart visualization. Inserting the Application Date dimension to the X Axis field, and then Approved to the Legend field, and finally Transaction Amount to the Y Axis field. Filtered to only show approved mortgage applications. 

***Visualization 3***

Global Bank wants to visualize how the loan approval rate has evolved to answer the question: “Has the quality of application files improved over recent years?” A 100% stacked column chart was used to visualize mortgage applications per year adding the Approved dimension to the visualization in the Legend field.

***Visualization 4***

Branch managers want to be alerted to any real-estate transactions over $250,000. To do this, we created a table visualization on the mortgage application page for pending mortgage applications that are yet to be processed, adding the application dates and transaction amount to the Value field, and a visual alert added.

### Branch Performance page

***Visualization 5***

A simple map visualization was added to show the location of the bank branches spatially. Bubbles are used to represent the branches, and which changes according to the number of customers, using automatic aggregation.

***Visualization 6***

A multi-row card was used to visualize the total mortgage applications received by the branches.

***Visualization 7***

Global Bank wants to visualize the relative performance of its branches in terms of the quality of mortgage applications received. For this case, a stacked-column-chart was used to visualize the approval rate for mortgage applications by branches. Here, we add the branch number to the X axis and add Approved column to the Legend field. Also add the number of mortgage applications to the Y axis. 

***Visualization 8***

A ribbon-chart visualization was added to the page to visualize the relative performance of the branches as a ranking according to mortgage applications received from customers. Application date was added to the X Axis, the branch number to Legend, and the number of mortgages to the Y axis.

### Customer List Page

A “Show customer list” page with a table visualization to display a list of all the applications per branch and the transaction totals.

### Customer Indicators page

A Customer Indicators page to monitor the make-up of the borrower portfolio and to zoom in on individual borrowers’ (or potential borrowers’) profiles. A table visualization was added on this page to show this, adding the Customer Number, Number of applications, Customer Transaction total and Customer date of birth, to the Value field.

## Data Modeling 

There is a Down Payment table that only contains the size of the down payment and the mortgage-application numbers. This has several drawbacks:

•	This table has as many lines as the mortgage-application table, as each application has its own down payment. Separating these tables out has made maintenance much harder.

•	The column with application numbers is stored twice to create the join, thereby wasting storage space.

•	There is need to manage the interaction between the two tables when queries are run on the down payments (filters, etc.). The calculation will be much slower, which is affecting performance. 

It would be much simpler to have a single table for mortgage applications, with a separate column for down payments. The tables will be merged by adding a new column to the right on the event table (mortgage applications) and adding the down payments

## Create Different Roles to Manage Data Confidentiality

There is a data-confidentiality request from Global Bank, that branch managers and advisors should not able to access data from the whole of the bank network via the dashboard. They should only have access to data from their branch. This was done by making use of the ‘Manage Roles’ functionality in Power Bi Desktop.

## Calculating the Borrower’s score

This measure is done in a bid to achieve the first goal which is to reduce the time it takes the executives to make a decision on loan applications. The score is going to be derived from calculating the three eliminatory criteria defined by the bank.

The three eliminatory criteria are:

*Criterion 1*

If the age of the borrower plus the length of the loan is over 82 years (life expectancy), the mortgage application is automatically rejected as it’s highly likely that the borrower will never pay back the loan.

To calculate the borrower’s age I used DAX to create a new column “Customer Age” using the formula Customer Age = DATEDIFF('Family Status'[Date of Birth], NOW(), YEAR) in the “Family Status” table. We can get length of the mortgage from the “Mortgage Applications” table.

*Criterion 2*

If regularity of income is at 3, which is very irregular, the application is automatically rejected. We can get this information from the “Income Regularity” column on the “Pro Status” table.

*Criterion 3*

Criterion three is broken down into three stages:

1.	We need to calculate the exact sum that Global Bank is required to contribute to the transaction, e.g., the total real-estate transaction minus the customer down payment. 
2.	Next, the bank calculates the monthly mortgage repayment by dividing the loan amount by the loan duration. 
3.	We’ll compare the borrower’s average monthly income and the monthly repayment to decide whether they will be able to repay the loan every month. The precise calculation requires that the repayment amount be less than the monthly income divided by three plus the number of dependent children. 

Written as an algorithm, the formula is as follows:

1.	Criterion 1: IF Customer age + Mortgage length is greater than or equal to 82 then assign "REJECTED" to borrower score otherwise
2.	Criterion 2: IF Regularity of customer income is greater than or equal to 3 then assign "REJECTED" to borrower score otherwise
3.	Criterion 3: IF Average monthly income divided by 3 plus the number of dependent children is less than monthly mortgage repayments then assign "REJECTED" to borrower score otherwise assign "APPROVED" to borrower score.
   
Here is the resulting formula that was used to calculate the borrower score:

Borrower_score = IF(LOOKUPVALUE('Family Status'[Customer Age],'Family Status'[Customer_ID],'Mortgage Applications'[Customer #])>=82,"REJECTED",IF(LOOKUPVALUE('Pro  Status'[Income Regularity],'Pro  Status'[Customer #],'Mortgage Applications'[Customer #])== "3 : Very irregular","REJECTED",IF(([Loan Amount]/'Mortgage Applications'[Duration])>LOOKUPVALUE('Pro  Status'[Average Monthly Income],'Pro  Status'[Customer #],'Mortgage Applications'[Customer #])/(3+LOOKUPVALUE('Family Status'[Number of Dependent Children],'Family Status'[Customer_ID],'Mortgage Applications'[Customer #])),"REJECTED","APPROVED")))


