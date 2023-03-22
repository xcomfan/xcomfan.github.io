---
layout: page
title: "Excel Notes"
permalink: /office/excel/
---

[comment]: <> (TODO: organize this once you are done with the course)

## Shortcut Keys

Look at the file you downloaded and fill in here.

* `ctrl` + `shift` + `↓` will select all values in a column.

## General

* By default text is left aligned and number are right aligned.  This is actually good because your decimals will look clean if they are right aligned.  If your numbers are being treated as text excel will give you the little triangle in the corner.

* Date value are also going to be right aligned (Excel treats them as numeric values)

* You can customize number formatting if you go custom option in the formatting pane.  For example you can do mmm-yyyy for Feb-2018 format.

* When using formulas in Excel there are two types of references to cells there are relative references (the default) which will got number of cells left right up down from the cell with the formula.  (Relative is from that location to another one) The second type of reference is the absolute reference is an absolute reference meaning that you want it to always grab that cell and not move around.  Do denote an absolute cell in your formula put a $ in front of each char of the cell designation. for example `=E9/$E$9`

* You can use the evaluate formula feature to step though how Excel evaluates a formula and debug complex formulas.

* You can use the fx button in the formula bar to bring up a dialog that lets you search for available functions.

* You can use the little green dot in the lower right hand corner of a cell to apply a formula to cells next to it.  Kind of like the dragging in Google sheets.

* You acn use the formula style = in a chart title to connect a title to a cell so that if cell changes the chart title changes.

* To create a template from a workbook do a save as and for type choose `Excel Template`.  This should save the file to the Office templates directory.  If you save it someplace else you will need to search for it each time you want to use it.

* When sorting a list you can add levels to sort by.  You can have up to 64 levels.

* You can do subtotals in a list.  Do do this you would first sort the column.  Then use the subtotal option and it will let you change by which group you want to create a subtotal.  You will also get a collapsible UI on the left hand side that lets you collapse groups within your lists.

* If you format a list as a table (you can pick any style you like) Excel will prompt you for where your data is and you will get a new tab in the ribbon with table tools.  There you can enable a Total Row a Header Row etc.  You can use the corner drag from lower right corner of table to stretch and grow you table.

* Conditional formatting button in ribbon has a find duplicates button. The remove duplicate option will be in Table tab if your formatted your data as a table, or under the data tab if its a list.

* If you are filling out an Excel function such as Sum clicking the fx button will give you a UI to fill in the arguments the function expects.  fx button is the function builder.

* Excel has a number of D functions (DCOUNT, DAVERAGE, etc) where the idea is you can sum pu values from a list based on the criteria.  you need to set up some sells with the column name right above the criteria value and then you can use the D formulas to make the counts for you.

* The subtotal function will let you create a subtotal that ignores hidden values allowing you to get a sum for filtered list.  So for example if you filter down to one category the subtotal will auto update for you the subtotal.  Its not just summing.  It can do averages and more.  See the Fx and help on this function for details.

## Data Validation

* From the Data Validation button you can set up a list of valid values and when you choose that cell that will be presented as a drop down list.

* In the data validation dialog you can use the Error Alert settings to control what the user gets if they input the wrong value.  There you can also control if they get a Warning or a stop.  

* The drop down list data validation thing is good for your DSUM category selection so that they pick from a list the category and your summation formula runs.

## Pivot Tables

* To create a pivot table you first need to format your data as a table. In the table design tab you can give the table a name.  That name you can use to create the Pivot table.  To actually create the table use insert -> pivot table.  And specify the name.

* Power pivot will not let you group so don't use that option if you care about grouping.

* Once you are editing your pivot table You can use the pivot table edit pane on the right and drag the available columns between the filters, columns, rows and values to build the report you want to build.

* In the Pivot Table analyze tab you can use the group selection to group and un-group rows.  For example you can Group January, February, March into Q1.  You can also drag into the Rows quadrant and Excel will auto group for you.  The order in the Rows quadrant matters as it will control wha the top and subgroup is  For example You can do Salesperson by region or region by salesperson.

* To format your pivot table cells you can use regular cell formatting, but if data is added the new cells won't get the format.  Instead you can left click on the value in the Sum of values quadrant and use Value field settings to make the format.

* If you drag a text value into the zum field Excel will default to counting the instances of that instance.

* You can use the value field settings menu to control what aggregation Excel uses (Sum, Count, Average, Min, Max, Product)

* You can drag a column into a quadrant twice to pick different calculations (sum in one average in another for example)

* The Value field settings show value as tab lets you do things such as difference from % of parent total % of parent column total.

* Pivot tables have a drill down feature where if you double click on a computed pivot table cell it will create another Pivot table for you (in new sheet) with the data that comprises that cell.  Its a good practice to rename that sheet when it gets created.

* Naturally you can create charts based on Pivot tables.  Just click on any cell of the pivot table and in the Pivot Analysis tab choose Pivot charts.

* Slicers are like filters but have dashboard like elements. From Pivot Table Analyze you can use insert slicer and add interactive elements. 

* Don't put spaces into table names.

## Power Pivot

* Power pivot will let you create a Pivot Table using two worksheets.  You need to create a data model in order to do this.

* To even use power Pivot you need to enable it.  Fore the version tht I am using you go to File -> Options Add Ins tab.  At the drop down below choose COM add-ins then Go button and check Power Pivot for Excel.

* Use teh diagram view in power pivot to relate fields to each other.  Can also right click on a cell and choose create relationship option.

* There is a KPI feature in power pivot that lets you add a cell/column of cells with a red yellow green status.  You need to first create an indicator such as an average or a sum.  Then you can create the KPI and play around with the setting to get the green yellow red to be the range you want.  Another non power pivot way to do this is to use conditional formatting -> icon sets.

## Working with large data sets.

* use freeze panes to lock the column headings.

* Under Data Table and outline section there are group options.  You can group rows or columns and Excel will give you a UI to expand and collapse the grouped sets.

* 3D Formulas is a way to reference cells from other sheets in your calculations.  To do this use ='SHEET_NAME'!B4 where B4 is the cell we are getting from SHEET_NAME

* You can consolidate data from multiple sheets with similar data.  Use the Data->Consolidate option and keep adding the sets of Data that you want to consolidate.  Excel will then apply the function you select and you can opt into it creating labels for you based on the top row or left column.

## Conditional Functions

* The name manager on Formulas tab is where you go to set up cell name ranges.

* IF function the value if true if false can be strings such as 'YES' or 'NO'

* Named references are absolute references so if you are using one you don't need to doe the $A$3 type syntax.

* Named references are a way to get data between worksheets.

* COUNTIF function will count a range based on condition.  For example if you need to count all the cells with a 'yes' value.

* SUMIF is like COUNTIF, but for summing.

* You can use IFERROR(MYOTHERFUNCTION,'what to display if error in MYOTHERFUNC') to clean up instances where you get an error from your function.  This is good to use in templates.

## Lookup functions

* VLOOKUP will look for a value in the leftmost column of a table, and then return a value in the same row from a column you specify.  By default table must be sorted in ascending order.  For the range lookup you can do a near match, but most time you would just choose false to have it find the exact match.

* HLOOKUP is like VLOOKUP but horizontal so it looks into value at top row.

* INDEX and MATCH function help you overcome the limit of VLOOKUP and HLOOKUP where they are essentially limited to looking up based on the first column.  INDEX will "Return a value or reference of the cell at the intersection of a particular row and column, in a given range.  The match function gives you the position if yo have a match.  So essentially you can combine these two to get a more customizable version of what VLOOKUP does.  INDEX and MATCH combo is also faster than using VLOOKUP.

* LEFT RIGHT and MID functions can be used to extract values from a single cell.  For example if you have a VIN number different parts of it have different values and you want to extract those so you can sort or filter by that data.

* SEARCH function lts you find the text you specify in a cell and return the position.  This is useful if you need to grab a first name from a cell when combined with LEFT RIGHT or MID.

## Auditing Excel Worksheets

* The Formulas tab Trace Precedents button will highlight for you all fo the cells that go into a formula. 

* The formula tab Trace Dependents button will show you what cells depend on the cell you have selected.

* To directly ref a cell in Another sheet use 'sheet_name'!C4

* Formula Tab watch window lets you select cells from any sheet that it will show you on the side so you can watch it change as you work with other sheets.

* Formula tab show formulas will expand all of the formulas you have on a sheet.  You can even print out the formulas.

## Protecting Excel worksheets

* Every cell has a locked attribute that is set true by default.  It is however not locked until you protect the sheet which you do via Review tab and Protect Sheet button.  To add/remove the cell specific loc Go to the cell Font menu and under protection tab add/remove locked property.  The lock lets you add an optional password.  There is also a check list of options of what you can and can't do once its locked and protected.

* The protect workbook button under review will let you protect the structure of the workbook.  It locks down the structure of the workbook itself so your formulas range names etc cannot me messed with.

* You can also add a workbook password to lock down the document.  Just got to File and Info section and use the options there.

## What if analysis

Data tab -> Data Tools section What if analysis -> Goals Seek lets you set a cell for which you want to change value to something and another cell that drives it and Excel will calculate what has to change for you to get to the goal.

### Solver tool

Solver tool is like Goal seek, but you can have Excel figure out multiple cells that drive your result not just one.

To activate File -> Options - Addins -> Manage Excel Addins check solver option.  It will add the solver tool to your ribbon bar.

Once you have the solver enabled, you can set the target cell and the cells that can be manipulated then add the constraints and it will fill in the cells which it can manipulate to solve your problem.

### Data Table

The idea of data table is if you have a formula that has a variable going in and you want to see what results are when that variable changes you can have data table fill that out for you.  You can have single or multi value data tables.

You find the data table in the Data tab under Dat Tools What if analysis.  

### Scenarios

The what if tools also have a scenario feature where you can store and save various scenarios (what certain cells will change to).  The you can pull up the scenario and click show it to have the data auto change and see what the results would be.

## Excel Macros

Exist under developer tab.  You can essentially record a set of steps into a Macro.  Once you recorded, you can make changes to the macro without re recording.  

All the functionality is under the developer tab.  You will need to go to the Visual Basic explorer from the ribbon and go to the module. which will show you the code being generated.

Note: you can sue .AutoFit to auto fit a column so you don't have hard numbers in your visual basic script.  (You can make the change in the VBA editor.)

From the developer tab you can also insert (using the insert button on the ribbon) a button that will run your macro.

* To get developer tab you may need to customize the ribbon.

## VBA section

* VBA stands for Visual Basic for Applications

* VBA is object oriented and everything is an object.  For example a sheet is an object.

* 