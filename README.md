# analyze-property
This project takes an export of transaction data associated with an income property and provides visualizations of the expenses, income and P&L in a year over year fashion.

It was built using transaction data exported from Mint, although any CSV of transaction data could potentially be used.  

This projects leverages jupyter  `.ipython/juptyer` form files.

## Checklist to build and start the application

Prerequisites:

1) The utilities in this package require conda.  If this is not yet installed please, download and install it from here:  https://conda.io/miniconda.html

2) Once properly installed run the following command in this directory:

    conda env create -f environment.yml

3) This will create the environment for the scripts to run in and download all necessary dependencies.   After this process completes run this command:

    source activate analyze-property
    
This will "activate" the anaconda environment that will allow python to run.  (For those using an autoenv type utitlity, there is also a .env file for automatically activating the environment whenever the directory is entered.)

## Preparing the data
This project was built using transaction data exported from Mint, although any CSV of transaction data could potentially be used.  My mechanism for managing the data in Mint is to assign all transactions related to a particular property to a single mint "Category" named after the property, and exporting all transacetions associated with that category to a CSV.  After the Mint Export, the csv is edited and a new column called "Label" is added which provides categorization.   

The column name which provides the categorization information, is configurable in the project.  In analyze_property_transactions.ipynb edit the line:

CATEGORY = "Label"

In addition to a column which provides categorization information, this project assumes the following standard Mint fields are also available:
* Date -- A string in datetime formate, ie: 01/05/2018
* Description -- A description of the transaction
* Amount - The amount of the transaction.  This will be interpreted as a float
* Transaction Type -- Either "credit" or "debit" to indicate an expense or income.  The project currently expects at least one "credit" and one "debit" in any given year's worth of transaction.
* Label - The column that indicates the transactions category.  This can be a different value as described above.

While users are free to assign transactions to any category they like it this project assumes that the following categories are used:

* Closing -- costs associated with the original closing of the property.  These transactions are excluded from the year-over-year P&L analysis and used as the basis cost to determine the annual ROI.
* Rent -- rental income
* Security -- this category is used for security fees collected and returned and is also removed from the P&L analysis
* Security_Kept -- this category is used for circumstances where a prior security deposit is kept and converted to income (usually to cover repair costs, etc)
* Utiltities - if transactions are classified as Utilties, some additional analysis is done to compare costs (if the property owner pays utiltiy bills) and income (if tenants reimburse property owner)

Finally, any category that has a "Transaction Type" of both "debit" and "credit" is assumed to be an expense, where the credits are ultimately subtracted from the debits to determine overall expense.   Categories that have only "credit" (ie: "Rent") are categorized as income.   The net of this is data that include "debits" in the "Rent" category may not work correctly without some data or code massaging.

## Preparing the unit specific data
Optionally, some analysis can be performed at a per unit level.   In order for this to work the following additional steps need to be taken

1) When preparing the data as described above, add an additional column called "Unit" to the transation data.   When appropriate assign a transation to a unit name (ie: "1W" or "Upstairs").    Examples of unit specific transactions might be rent collected, security related transactions, or repairs specific to a unit.
2) Create a csv File that enumerates all the units and the average monthly rent per year.   An example of what this file might look like is:
   
   | Year | First Floor | Second Floor |
   |------|-------------|--------------|
   |2018  | 1000        | 1500         |
   |2017  | 900         | 1450         |

3) In the first cell set the value of PATH_TO_RENTS to your CSV file

If this step is excluded everything in the notebook should work as expected however the last cell, which provides the per unit analysis, will fail.

## Analyzing the data

Start Jupyter from the working directory (the project directory with these scripts):

    jupyter notebook 

This will open up a page in your default browser.  It should look something like:

![Initial Jupyter Page](/tutorial_images/InitialJupyterPage.png)

From there click the link for the file `analyze_property_transactions.ipynb`.  It should open in a new tab.

Once this file is open you can begin running the spending analysis, cell by cell.
In the tab that Jupyter opened for you when you selected the notebook, click and highlight the very first cell, it should look something like:

![First Jupyter Cell](/tutorial_images/JupyterFirstCell.png)

Note that it sets the input file to "transactions.csv", but you can change it to whatever you named your transaction file.  Similarly, the variable CATEGORY is set to 'Label'.  If, for example you have a dedicated mint account for your property and categorize directly in mint, you can change this to 'Category'.

With the cell highlighted, run the cell either by clicking on the Run button or by using the keyboard shortcut Shift+Enter.   

Jupyter will automatically shift the focus to the next cell, so you may need to scroll up to see the output.  If all goes well you should now see the first five rows of the spending data loaded as a pandas dataframe:

![After Running First Jupyter Cell](/tutorial_images/AfterFirstCellRun.png)

You should, see the first few lines of a data frame that looks like each row has a single trasaction in it it.  If you have this you are ready for the next cell. 

Highlight and run the second cell which will attempt to adjust any "credits" in the transaction data for expense categories.  As an example if you returned some tools to the store, the "income" associated with this return will be used to offest your expense.

Each time an adjustment like this is made the details are printed out.  

![After Credit Adjustment](/tutorial_images/AdjustCredits.png)

Examine the output to make sure it all makes sense and adjust the transaction data as needed.

Finally, once the income and expense data is ready, you can move through the remaining cells to perform following:
* set the colors for categories so they are consistent across visualizations
* show overall expenses and income year over year
* pie chart breakdown of expensese for each year
* P&L year over year broken down by unit

## Additional functions not fully documented
The following notebooks may also be of use or may be too specific to the author's particular situation.   Feel free to try them out, no warranty is made and its likely I forgot to document stuff.

* analyze_utility_transactions -- this notebook reads the same transaction input data but focuses on the utility information.   It is useful for a property owner whose units are individually metered, but who pays the bills for those meters.  It also assumes that tenants are responsible for utility costs above an amount specified in a utils-limits.csv (configurable in Cell 1 via the PATH_TO_UTILITY_LIMITS).   An example of this CSV file might be
  
    | Unit        | Amount |
    |-------------|--------|
    |First Floor  | 100    |
    |Second Floor | 150    |
  
* analyze_security -- show the security collected and returned for each unit.  Uses the same PATH_TO_RENTS file as described above
* generate_supplemental_incom_and_loss_tax_info -- this notebook will read the transactions for a particular year (specified in Cell 1 via the TAX_YEAR variable), and attempts to output a CSV that can be used to fill out Schedule E Supplemental Income and Loss for your federal taxes.    Also specified in CELL 1 are:
  * MORTGAGE_INTEREST -- obtained from your lender
  * EST_DEPRECIATION -- obtained from prior year taxes or CPA
  * EST_AMMORTIZATION -- obtained from prior year taxes or CPA

The Schedule E information is output to the OUTPUT_CSV file specified in Cell 1

## Still on the TO DO list

[ ] Clean out all the old files from the original project and all my half baked stuff

[ ] Be smarter about years that might have no expense or credit associated with them.  Ideally we have one of each so that the legends in the expense and income visualizations have the same color for the years.

[ ]  Make all the column names configurable via  a configuration file that non programmers can edit
