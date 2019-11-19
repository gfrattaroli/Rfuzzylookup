# Rfuzzylookup
A way to automate fuzzy lookup between two table files

The current script takes data from the official edubase database (http://ea-edubase-api-prod.azurewebsites.net/edubase/home.xhtml)
and a local csv file (not provided in the script).

This script was made with the intention of automating fuzzy lookup between two files without using any Office package software.

The following is a guide for first users :

README IF THIS IS YOUR FIRST USE

To use the script you will need to download the following files:

	- https://cran.r-project.org/mirrors.html (This will download R, please choose a mirror that is close to you. If you want to know more about R click here: https://www.r-project.org/about.html)
	- https://www.rstudio.com/products/rstudio/download/#download (This will download R Studio Desktop Open Source, which is an IDE for R, if you want to know more click here : https://www.rstudio.com/products/rstudio/)

Once you’ve installed R Studio, open the file ‘Script.R’

Select ALL the code (cmd + A) and RUN it (cmd + Enter)

If any problems with getting the data, try again in few minutes, it could happen that the Edubase website is busy.

The final matched database will be named ‘FinalDB.xlsx’.


The script will perform the following actions:

	- Run a fuzzy lookup between CRT and all open schools from Edubase, filtering out schools from Scotland and Northern Ireland, matching on School Name,Address and Postcode

	- For all the Non-matched schools, all schools from England are being merged with their exact URN match from Edubase

	- All URNs are then merged with the EduLink file 


Enjoy!


