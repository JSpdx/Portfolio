# FOCUSER
A web app for displaying astronomy information I built using Django, API's from NASA and Wolfram Alpha, and the webscraping package BeautifulSoup


## Introduction
During a 2 week sprint at the Tech Academy, I used Python and Django to create a web app that displays information for the amateur astronomer, and space nerd.
(self congratulatory statement incoming) I was the 2nd student ever to complete all 12 user stories during the sprint.

## Functionality
- Provides a list of upcoming eclipses that can be manually stored, edited and deleted in a SQLite database,
- Displays a picture and description from Nasa's APOD (astronomy picture of the day), 
- Allows users to store their favorite APOD entries in the databse,
- Retrieves information about upcoming meteor showers using Wolfram Alpha's API
- Retrieves information from the International Space Station's RSS feed using the BeautifulSoup webscraping package
	

## Technologies/Practices
- Django MTV/MVC framework 
- Used APIs from NASA and Wolfram Alpha to add dynamic, self-updating features to the application
- Parsed JSON to retrieve the desired information from the API
- Webscraping package BeautifulSoup, used to get data from an RSS
- RESTful architecture, combined HTML GET and POST methods with IF/ELSE statements in Python to add state to pages.
- RDBMS SQLite to store form entries, and user selected favorites 
- Virtualenv virtual environment to maintain a consistent and compatible development environment
- Agile pracices used throughout development including daily standups and user stories to keep production moving forward
- Azure DevOps as a platform for project management
- Git for version control, to maintain code integrity in a team with multiple developers
- Markdown for formatting the github README

###Languages used:
- Python
- Javascript
- HTML and CSS

##Stories
- Create a new app for the project, named appropriately for what you are tracking, and get it to display a home page with basic content.
- Create a model for the collection item you will be tracking and add the ability to create a new item.
- Display the information from the database in an index page.
- Create a details page that will show the details of any single item from within the database, as selected by the user. Link this to the index page for each item.
- Allow for edits and delete functions to be done from the details page or from separate pages. Have confirmation before deleting.
- Connect to an API and get the JSON response, add in a template for displaying the information.
- Parse through the JSON file returned and display the information you want to display. Make additional queries to the API as necessary. Add a link from your app's home page.
- Create a new template for displaying information sourced from another website. Use Beautiful Soup to data scrape the site and find the relevant information.
- Parse through the html returned and display the information you want to display. Make sure you are getting into the individual elements and stripping away any formatting you don't want. Add a link from your app's home page.
- Go through your various templates and add improvements to the UI/UX. This may include hover effects, pop-ups, animations, changes to the existing styling, etc. Show off your creativity and styling ability.
- Allow the user to save "favorites" of item either from the information detailed from the API or from Beautiful Soup. This could mean working with the existing model or creating a new one to pull the information from the response, create the appropriate object, and add it to the database. 
- Congratulations! You've hit the point where we're out of standardize stories. Use the Discussion section below to describe the story you'd like to complete with your app. Your instructor will respond with and questions or suggestions.






## Highlights

**Using Django's built in FOR loop tags, I was able to quickly and efficiently create an HTML table that would automatically display all of the eclipse events stored in the database, and reflect an added or subtracted item.**

/templates/Focuser/focuser_index.html
```
	 <table class="table-striped">
            
	    <tr id="column_header">
                <th class="col-md">Date</th>
                <th class="col-md">Locations</th>
                <th class="col-md">Type</th>
                <th class="col-md">Subtype</th>
	    </tr>

            {% for eclipse in eclipses %}     <!-- creates a new row for each eclipse event stored-->
                <tr>
                    <td class="col-md">{{eclipse.date}}</td>
                    <td class="col-md">{{eclipse.locations}}</td>
                    <td class="col-md">{{eclipse.type}}</td>
                    <td class="col-md">{{eclipse.subtype}}</td>

                    <td class="col-md"><a href="{{eclipse.pk}}/Details"><button class="primary-light-button">Details</button></a></td>
                    <td class="col-md"><a href="{{eclipse.pk}}/Edit"><button class="primary-light-button">Edit</button></a></td>
                </tr>
            {% endfor %}
        </table>
```

**Using Django tags and hidden HTML forms, I was able to pass the API data being displayed on that page to be saved in the DB.**

/templates/Focuser/focuser_apod.html:
```

	<form method ="post">
            {% csrf_token %}
            <input type="hidden" name="title" value="{{title}}">
            <input type="hidden" name="explanation" value="{{explanation}}">
            <input type="hidden" name="image_url" value="{{hdurl}}">
            <button class="btn btn-danger">Save to Favorites</button>
        </form>

```

/Focuser/views.py:
```
 def apod(request):

    response = requests.get('https://api.nasa.gov/planetary/apod?api_key=4a8sB9S0WoqXO6HstMj15Lgqu5isYYpys0675ygO')
    context = response.json()
    if request.method == 'POST':
       
	 if 'date' in request.POST:                      # this IF statement renders a new page based on the user's input
            user_date = request.POST['date']
            response = requests.get('https://api.nasa.gov/planetary/apod?date={}&api_key=4a8sB9S0WoqXO6HstMj15Lgqu5isYYpys0675ygO'.format(user_date))
            context = response.json()
            return render(request, 'Focuser/focuser_apod.html', context)

        elif 'explanation' in request.POST:             # this ELIF saves the current page to the Favorites model
            form = FavoriteForm(request.POST or None)
            if form.is_valid():
                form.save()

    return render(request, 'Focuser/focuser_apod.html', context)

```

**Here I used BeautifulSoup to parse the html provided by a daily news feed RSS, and create a dynamic webpage.
By changing the HTML request method to POST, I was able to create branching IF statements, adding logic to my application 
 I used a counter to navigate backwards and forwards through the desired HTML tags to view older or newer entries.**

```
iss_counter = 1                                                        #used this global variable as a counter to increment and decrement the index containing page entries.
def iss(request):
    global iss_counter 
    page = requests.get('https://blogs.nasa.gov/spacestation/feed/')
    soup = BeautifulSoup(page.content, 'html.parser')
    
    #tag_list = list(soup.children)                                     # these 3 lines not used. They illustrate another method for parsing html. I used "find and find_all" instead.
    #types = [type(item) for item in list(tag_list)]                    #used to find the BeautifulSoup "Tag" object,
    #body = list(soup.children)[1]                                      #targets the tag object
    
    if request.method == 'POST':					# By changing the HTML request method to POST, I was able to create a branching IF statements, adding logic to my application 
        if 'prev' in request.POST:					
            iss_counter += 1
            title = str(soup.find_all('title')[iss_counter].get_text())
            content = str(soup.find_all('content:encoded')[iss_counter - 1])
            context = {'title': title, 'content': content}
            return render(request, 'Focuser/focuser_iss.html', context)

        else:
            iss_counter -= 1
            print(iss_counter)
            title = str(soup.find_all('title')[iss_counter].get_text())
            content = str(soup.find_all('content:encoded')[iss_counter - 1])
            context = {'title': title, 'content': content}
            return render(request, 'Focuser/focuser_iss.html', context)

    title = str(soup.find_all('title')[1].get_text())  # extracts the headline
    content = str(soup.find_all('content:encoded')[0])  # extracts the page's content
    context = {'title': title, 'content': content}
    iss_counter = 1
    return render(request, 'Focuser/focuser_iss.html', context)
```

**I was able to use a primary key generated in the database to 

/templates/Focuser/focuser_index.html:
```
{% for eclipse in eclipses %}     <!-- creates a new row for each eclipse event stored-->
                <tr>
                    <td class="col-md">{{eclipse.date}}</td>
                    <td class="col-md">{{eclipse.locations}}</td>
                    <td class="col-md">{{eclipse.type}}</td>
                    <td class="col-md">{{eclipse.subtype}}</td>

                    <td class="col-md"><a href="{{eclipse.pk}}/Details"><button class="primary-light-button">Details</button></a></td> <!-- Creates a button that navigates to a page specific to this item's primary key -->
                    <td class="col-md"><a href="{{eclipse.pk}}/Edit"><button class="primary-light-button">Edit</button></a></td> 
                </tr>
{% endfor %}

```
In the url routing page, the primary key `<int:pk>` is used to create the url pattern, then passed into the `details` view as an argument.
/Focuser/urls.py:

```
path('<int:pk>/Details', views.details, name='details'),
```
Once passed as an argument into the `details` view, the primary key is then used to retrieve the pertaining item from the database and send it to be displayed.
```
def details(request, pk):
    pk = int(pk)
    item = get_object_or_404(Eclipse, pk=pk)
    context = {'eclipse': item}
    return render(request, 'Focuser/focuser_details.html', context)
```


### Summary

This project proved to be a great learning opportunity for me. Working on a project with multiple people gave me some good experience utilizing version control, 
and successfully merging conflicts. Learning to use an MVC like Django (technically MTV) was very beneficial too.
I have since been exposed to other MVC frameworks like React, and because of my experience here with Django, the learning curve was greatly reduced. 







#### Running the project
Project needs virtualenv installed in order to run.

1)From environments, run scripts\activate

2) From FOCUSER\mainproject, run python manage.py makemigrations,
			    then python manage.py migrate
			    then python manage.py runserver

This app was written as part of a group project at my bootcamp, and the part I was responsible for was the section called FOCUSER.

				
