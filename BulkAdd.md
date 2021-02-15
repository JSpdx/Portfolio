
## Theatre Vertigo CMS

### Technologies used 

- C# / ASP.NET MVC
- Javascript / jQuery / JSON
- CSHTML / CSS / Bootstrap / Razor
- AJAX / Moment.js / Entity Framework / MSSQL / Visual Studio / Azure DevOps

### Overview  

During my time as an intern at Prosper IT Consulting, I did full stack work on a content management system for a theater production company.

### Features added

- Built widget that allows an admin to add multiple calendar events, based off of a number of parameters.
- Created an infinite scrolling feature for photo index page (page loads more photos when user reaches the bottom)
- Created an animated modal image for photo page  
- Enhanced flexibility of search feature  
- Restricted access of rental request form to Admins
- Made pictures of cast members dynamically resize / made text layover clickable  
- Modified season managers to track season  
- Updated Season Manager with role restrictions, altered functionality based on user role
- Updated footer to be responsive, and fixed a bug where the Footer was not always pinned to the bottom

## Highlights

### Bulk Events Creation.

![](https://media.giphy.com/media/fV7aTJFEFXEszn3Iu7/giphy.gif)

   This feature allows an admin to create and edit multiple calendar events based off of a start date, end date, show start time, day(s) of the week that shows will occur, and interval of weeks between shows. When the user is satisfied with their list, they can then submit it to the database. It uses moment.js to handle dates and times, and uses jQuery's AJAX method to pass data to and from the controller.


   Here is the action method in the controller that passes information about the dates and show times to the browser:
   It receives the AJAX request, queries the database using LINQ, then converts it to JSON and passes it back to the browser as a response to the AJAX call.
   ```csharp
   [Authorize(Roles = "Admin")]
        public ActionResult GetProduction(int productionId = 0)
        {
            int id = Convert.ToInt32(productionId);
            var query = from production in db.Productions
                        where production.ProductionId == id
                        select new { production.OpeningDay, production.ClosingDay, production.ShowtimeMat, production.ShowtimeEve, production.Runtime }; /*ShowtimeMat = production.ShowtimeMat.Value.ToString("hh:mm tt"), ShowtimeEve = production.ShowtimeEve.Value.ToString*/

            return Json(JsonConvert.SerializeObject(query), JsonRequestBehavior.AllowGet);

        }
   
   ```
   
   The bulk of it was written in Javascript:
```javascript

// This class represents a single calendar event.
// CalendarEvent properties that are capitalized match the properties in the MVC CalendarEvent model. 
// They must be verbatim in order for the JSON deserializer to work correctly
class CalendarEvent {
    constructor(startDate, endDate, dayOfWeek, startTime) {
        this.Title = $("#generate__production-field").children("option").filter(":selected").text();
        this.ProductionId = $("#generate__production-field").val();
        this.StartDate = startDate;
        this.EndDate = endDate;     // events are never longer than one day, but to match the database, EndDate is the same date as StartDate with it's time advanced by the runtime of the production.
        this.dayOfWeek = dayOfWeek;
        this.startTime = startTime;
    }
}
if ($("#generate-showtimes-section") != null) {
    var reviewEvents = [];                                // reviewEvents is used to store the complete list of calendar event objects, as they are added from generatedEvents. It's then passed to the back end.
    var runtime = 0                                     // runtime stores the length of a given event in minutes. It's then used to incrememnt the event's time and create a second event that marks the end time of the production.

    //      When a production is selected from the dropdown, an ajax call is made sending that production's start date, end date, productionId, and runtime.
    //      The start date and end date dropdowns are then autofilled, and the runtime variable is set.
    $("#generate__production-field").change(function () {
        var productionId = $("#generate__production-field").val();
        $.ajax({
            method: 'GET',
            url: '/CalendarEvents/GetProduction',
            data: { "productionId": productionId },
            dataType: 'json',
            success: function (data) {
                if (data != "[]") {
                    let production = jQuery.parseJSON(data);
                    let openingDay = production[0].OpeningDay.substr(0, 10); // This removes the time of day and leaves only the date
                    let closingDay = production[0].ClosingDay.substr(0, 10);
                    $("#generate__start-date-field").val(openingDay);
                    $("#generate__end-date-field").val(closingDay);
                    $("#matinee-time").html(moment(production[0].ShowtimeMat).format('h:mm a'));
                    $("#evening-time").html(moment(production[0].ShowtimeEve).format('h:mm a'));
                    runtime = production[0].Runtime;
                }
            },
            error: function () {
                alert("Error while retrieving data!");
            }
        });
    });

    //    This function does some traffic control. Once the generate button is clicked, the generatedEvents list is populated, and a modal pops up showing the list of dates that were added, 
    //    and offers yes and no buttons which allow the user to either go back and change their parameters
    //    or to append these showtimes to a final list to be reviewed and edited before being submitted 
    $("#generate-button").click(function () {
        $('.bulk-add_review-row').unbind('mouseenter mouseleave');

        var modal = $('#bulk-add-modal'),
            yesBtn = $('#bulk-add-modal_yes'),
            noBtn = $('#bulk-add-modal_no'),
            reviewShowtimes = false,                    // This variable is used to determine whether to render the list of times in the modal, or to append it to the review section.
            generatedEvents = generateShowtimes();
        if (generatedEvents.length < 1) {                     // If the list is empty, the user didn't select all the required parameters.
            return;
        }
        modal.show();
        createTable(generatedEvents, reviewEvents, reviewShowtimes);

        noBtn.off('click');                             // The .off() and .one() methods are to prevent event handlers from stacking up.
        noBtn.one("click", function () {
            modal.hide();
            $('.bulk-add_modal-row').remove();          // This clears all the entries from the modal table so they won't stack every time the Generate button is clicked.
        });
        yesBtn.off("click");
        yesBtn.one("click", function () {               // When the yes button is clicked, the modal disappears and clears its entries. The showtimes are appended to the reviewEvents list, and displayed in the review showtimes section. 
            modal.hide();
            reviewShowtimes = true;
            $('.bulk-add_modal-row').remove();
            $('.bulk-add_review-row').remove();         // The Review Showtimes list is generated from the reviewEvents list each time yes is clicked, so it needs to be cleared
            reviewEvents.push.apply(reviewEvents, generatedEvents);
            reviewEvents = reviewEvents.sort((a, b) => a.StartDate - b.StartDate);
            createTable(generatedEvents, reviewEvents, reviewShowtimes);
        });
    });

    //      This function takes the user's input and performs calculations to generate a list of events sorted by ascending date. It returns to the generatedEvents list.
    function generateShowtimes() {
        let startDate = moment($("#generate__start-date-field").val()),
            endDate = moment($("#generate__end-date-field").val()),
            eventDate = startDate,
            dateRange = endDate.diff(startDate, 'days'),
            interval = $("#interval").children("option").filter(":selected").val(),
            generatedEvents = [];
        let startTimes = [];                                          // This array holds each selected start time. An event is created for each start time on any given day.
            if ($('#matinee').is(':checked')) {
                startTimes.push($('#matinee-time').text());
            }
            if ($('#evening').is(':checked')) {
                startTimes.push($('#evening-time').text());
            }
            if ($('#custom-time').val() != "") {
                startTimes.push($('#custom-time').val());
            }
            if (startTimes.length == 0) {
                alert("Please select a start time");
            }

        let days = [];                                   // This array takes each selected day of the week. For each day, within each eligible week, events will be created. 
            if ($('#sunday').is(':checked')) {
                days.push(0);
            }
            if ($('#monday').is(':checked')) {
                days.push(1);
            }
            if ($('#tuesday').is(':checked')) {
                days.push(2);
            }
            if ($('#wednesday').is(':checked')) {
                days.push(3);
            }
            if ($('#thursday').is(':checked')) {
                days.push(4);
            }
            if ($('#friday').is(':checked')) {
                days.push(5);
            }
            if ($('#saturday').is(':checked')) {
                days.push(6);
            }
            if (days.length == 0) {
                alert("Please select at least one day")
                return (generatedEvents);
            }


        
        // This block calculates all eligible days, and creates an event for each showtime selected.
        // For each day selected, for each eligible week between the start date and end date that the day occurs, for each start time selected, an event is created.
        for (i = 0; i < days.length; i++) {
            if (days[i] < startDate.day()) {
                days[i] += 7;
            }
            startDate.day(days[i]);
            eventDate = startDate; //refreshes the event date
            for (j = days[i]; j <= dateRange + 7; j += 7 * interval) {
                if (eventDate.isBetween(startDate, endDate, undefined, '[]')) { //check for the eventDate to be within start and end date. The '[]' argument sets it to be inclusive of the start and end date.
                    for (k = 0; k < startTimes.length; k++) {
                        let hr = parseInt(startTimes[k].substr(0, startTimes[k].indexOf(':'))),       // startTimes are all strings, and Moment.js needs ints to add a time of day to a moment.
                            min = parseInt(startTimes[k].substr(startTimes[k].indexOf(':') + 1, 2)),  // This parses the the string "11:30 am" for example and creates a variable for the hr, the minute, and am or pm.
                            amOrPm = startTimes[k].slice(-2).toUpperCase();
                        if (amOrPm == 'PM' && hr < 12) {
                            hr += 12;
                        }
                        eventDate.hour(hr).minute(min);
                        var endTime = moment(eventDate);
                        endTime.add(runtime, "minutes")
                        const event = new CalendarEvent(moment(eventDate), endTime, eventDate.format('dddd'), startTimes[k]);
                        generatedEvents.push(event);
                    }
                }
                eventDate.add((7 * interval).toString(), 'days').format('ll'); //increments the event date to the next eligible date
            }
            startDate = moment($("#generate__start-date-field").val());
            eventDate = startDate;
        }
        return generatedEvents.sort((a, b) => a.StartDate - b.StartDate);
    }


    //this function generates a table displaying the list of events created in the generateShowTimes() function.
    //depending on the state of the reviewShowtimes variable, it will create the table in either the modal or the 'review showtimes' section.
    function createTable(generatedEvents, reviewEvents, reviewShowtimes) {
        // this block creates a table in the modal
        if (reviewShowtimes != true) {
            var table = document.getElementById("modal-table"),
                row = table.insertRow();
            row.className = 'bulk-add_modal-row';
            for (i = 0; i < generatedEvents.length; i++) {
                var cell = row.insertCell();
                cell.innerHTML = generatedEvents[i].StartDate.format('ll');
                cell = row.insertCell();
                cell.innerHTML = generatedEvents[i].dayOfWeek;
                cell = row.insertCell();
                cell.innerHTML = generatedEvents[i].startTime;
                row = table.insertRow();
                row.className = 'bulk-add_modal-row';
            }
            document.getElementById('bulk-add-modal_content').appendChild(table);
        }

        // this block creates a table in the review showtimes section
        if (reviewShowtimes == true) {
            $("#review-showtimes-section").show();
            var table = document.getElementById("showtimes-table"),
                row = table.insertRow();
            row.className = 'bulk-add_review-row';
            for (i = 0; i < reviewEvents.length; i++) {
                var cell = row.insertCell();
                if (typeof reviewEvents[i].StartDate != "string" && reviewEvents.length > 0) {
                    cell.innerHTML = reviewEvents[i].StartDate.format('ll');
                }
                cell = row.insertCell();
                cell.innerHTML = reviewEvents[i].dayOfWeek;
                cell = row.insertCell();
                cell.innerHTML = reviewEvents[i].startTime;
                row = table.insertRow();
                row.className = 'bulk-add_review-row';
            }
            document.getElementById('showtimes-container').appendChild(table);    // generates the table in html.
            deleteRowFeature();
        }
    }

    // this function creates a delete button when a row is hovered over in the review showtimes section.
    // When it's clicked, it removes the corresponding row, and deletes the event from the master list
    function deleteRowFeature() {
        let row = $('.bulk-add_review-row');
        row.off('hover');
        // when a row is hovered over, a delete button is created, and the index of that row is recorded and used to remove that entry from the master list
        row.hover(function () {
            let button = $('<button type="submit" class="bulk-add_delete">Delete</button>')
                .hide().fadeIn(1200);
            let rowIndex = $('tr').index(this) - 2; // targets the specific row to be deleted 
            $(this).append(button);
            button.click(function () {             // when the delete button is clicked, the row is removed from the table, and the corresponding event is removed from the master list.
                button.closest('tr').remove();
                reviewEvents.splice(rowIndex, 1);
            })
        }, function () {                          // this removes the delete button when the mouse stops hovering over that row.
            $('.bulk-add_delete').remove();
        });
    }
    $('#bulk-add_submit').off('click');
    $('#bulk-add_submit').click(submitEvents);

    function submitEvents() {
        for (var i = 0; i < reviewEvents.length; i++) {
            if (typeof reviewEvents[i].StartDate == "object") {    //this checks that .format is only applied to items that haven't yet been formatted.
                reviewEvents[i].StartDate = reviewEvents[i].StartDate.format('lll');
                reviewEvents[i].EndDate = reviewEvents[i].EndDate.format('lll');
            }
        }
        var data = JSON.stringify(reviewEvents);
        $.ajax({
            method: 'POST',
            url: '/CalendarEvents/BulkAdd',
            data: { 'jsonString': data },
            success: function () {
                if (reviewEvents.length == 0) {
                    alert("Events already added")
                }
                else {
                    alert('Events Added!');
                    reviewEvents = [];
                }
            },
            error: function () {
                alert("Error while posting data!");
            }
        });
    };
}

```

