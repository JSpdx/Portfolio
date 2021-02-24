# Multi-Table Search


## Overview  

During my time as an intern at Prosper IT Consulting, I created a search feature that queried multiple tables in the database and returned results with the search key highlighted.

### Technologies used

C#, ASP.NET MVC, REGEX, LINQ and Razor

### Requirements/User Story

Back-end:  Currently, the search categories only search one field each in their corresponding tables.  We want a more flexible search.  When searching the following tables, search the corresponding fields simultaneously, i.e. the user's search term can appear in any of the 3 related fields

Cast Member: Search in Name, YearJoined, Bio 
Production:  Search in Title, Playwright, Description
Part: Seach in Character, Production Title, CastMember Name

Front-end:

* In the returned results, highlight the search term wherever it appears.  Use a blue highlight with white text
* Provide a way for the user to click on each result and be taken to the detail page for the corresponding cast member, production, or part.  Come up with the best implementation for that
* The results section has black text over white background.  This looks incompatible with the black background of the Archive page.  Please restylize to a compatible color scheme
* Please label the start of the results section
* Finally, please reformat the results section so that everything doesn't look so cramped like they do now:


### Bulk Events Creation.

![zxc](https://drive.google.com/file/d/1HpjmMIMc9MLLErdFUWUmNlPidYCI0K6F/view?usp=sharing)

   This feature allows an admin to create and edit multiple calendar events based off of a start date, end date, show start time, day(s) of the week that shows will occur, and interval of weeks between shows. When the user is satisfied with their list, they can then submit it to the database. It uses moment.js to handle dates and times, and uses jQuery's AJAX method to pass data to and from the controller.


   Here is the action method in the controller that passes information about the dates and show times to the browser:
   It receives the AJAX request, queries the database using LINQ, then converts it to JSON and passes it back to the browser as a response to the AJAX call.