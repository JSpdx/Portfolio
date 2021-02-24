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


### This was the result:


<a href="https://drive.google.com/uc?export=view&id=1dQ_UD3vWa0fzKGUTesPm791KEV_DuaVR"><img src="https://drive.google.com/uc?export=view&id=1dQ_UD3vWa0fzKGUTesPm791KEV_DuaVR" style="width: 400px; max-width: 100%; height: auto" title="Click to enlarge picture" /></a>


I started with the back-end part of the story. I opted to create a private search method, instead of having it all in the controller method. This kept the controller method to 4 lines of code.
```csharp
public ActionResult Archive(string SearchByCategory, string searchKey)
        {
            var db = new ApplicationDbContext();
            var productions = db.Productions.Include(i => i.DefaultPhoto);
            ArchiveSearch(db, SearchByCategory, searchKey);
            return View(productions.ToList());
        }
```

In implementing the 'search all' functionality, I found a bug where the 'if viewdata["category"] is null' logic was never detecting a null value.
I fixed this by putting a check before setting the ViewData in the controller:

```if (resultsCast.Count > 0) ViewData["ResultsCast"] = resultsCast;```

I created one method for retrieving the data, and a second set of overloaded methods to highlight the search key. If I were going to do this over again, I'd use generics and reduce the highlight feature to a single method.
                   
```csharp
    private void ArchiveSearch(ApplicationDbContext db, string searchByCategory, string searchKey)
        {
            ViewBag.Category = searchByCategory;
            if (searchKey == "")
            {
                return;
            }
            string highlightedKey = "<span id='highlight'>$&</span>";   //highlightedKey is where the css id is applied to the highlighted word.  $& swaps the search key with the original text to keep the casing intact.
            string pattern = string.Format(searchKey);                                // For whole word search, pattern = @"\b" + searchKey + @"\b"; For substring matching, pattern = searchkey;
            pattern = Regex.Escape(pattern);
            Regex rx = new Regex(pattern, RegexOptions.IgnoreCase);
            switch (searchByCategory)
            {
                case "SearchAll": //This case searches across all three tables.
                    ViewBag.Message = string.Format("Results for \"{0}\" in Archive", searchKey);
                    var resultsCast = new List<CastMember>();
                    foreach (CastMember castMember in db.CastMembers)
                    {
                        Match matchName = rx.Match(castMember.Name);
                        Match matchYearJoined = rx.Match(castMember.YearJoined.ToString());
                        Match matchBio = rx.Match(castMember.Bio);
                        if (matchName.Success || matchYearJoined.Success || matchBio.Success)
                        {
                            resultsCast.Add(castMember);
                        }
                    }
                    resultsCast = resultsCast.Distinct().ToList();//Prevents duplicate listings
                    if (resultsCast.Count > 0) ViewBag.ResultsCast = resultsCast;

                    var resultsProduction = new List<Production>();
                    foreach (Production production in db.Productions)
                    {
                        Match matchTitle = rx.Match(production.Title);
                        Match matchPlaywright = rx.Match(production.Playwright);
                        Match matchDescription = rx.Match(production.Description);
                        if (matchTitle.Success || matchPlaywright.Success || matchDescription.Success)
                        {
                            resultsProduction.Add(production);
                        }
                    }
                    resultsProduction = resultsProduction.Distinct().ToList();
                    if (resultsProduction.Count > 0) ViewBag.ResultsProduction = resultsProduction;

                    var resultsPart = new List<Part>();
                    foreach (Part part in db.Parts.ToList())
                    {
                        Match matchCharacter = rx.Match(part.Character);
                        Match matchTitle = rx.Match(part.Production.Title);
                        Match matchName = rx.Match(part.Person.Name);
                        if (matchCharacter.Success || matchTitle.Success || matchName.Success)
                        {
                            resultsPart.Add(part);
                        }
                    }
                    resultsPart = resultsPart.Distinct().ToList();
                    if (resultsPart.Count > 0) ViewBag.ResultsPart = resultsPart;
                    Highlight(resultsCast, resultsProduction, resultsPart, pattern, highlightedKey); //applies highlight effect to search matches
                    break;
                case "SearchCastMembers":
                    ViewBag.Message = string.Format("Results for \"{0}\" in Cast Members", searchKey);

                    resultsCast = new List<CastMember>();
                    foreach (CastMember castMember in db.CastMembers)
                    {
                        Match matchName = rx.Match(castMember.Name);
                        Match matchYearJoined = rx.Match(castMember.YearJoined.ToString());
                        Match matchBio = rx.Match(castMember.Bio);
                        if (matchName.Success || matchYearJoined.Success || matchBio.Success)
                        {
                            resultsCast.Add(castMember);
                        }
                    }
                    resultsCast = resultsCast.Distinct().ToList();//Prevents duplicate listings
                    Highlight(resultsCast, pattern, highlightedKey); //Applies highlight effect to matches
                    if (resultsCast.Count > 0) ViewBag.ResultsCast = resultsCast;                       //sets ViewData value if there were any results
                    break;

                case "SearchProductions":
                    resultsProduction = new List<Production>();
                    foreach (Production production in db.Productions)
                    {
                        Match matchTitle = rx.Match(production.Title);
                        Match matchPlaywright = rx.Match(production.Playwright);
                        Match matchDescription = rx.Match(production.Description);
                        if (matchTitle.Success || matchPlaywright.Success || matchDescription.Success)
                        {
                            resultsProduction.Add(production);
                        }
                    }
                    resultsProduction = resultsProduction.Distinct().ToList();
                    Highlight(resultsProduction, pattern, highlightedKey);
                    if (resultsProduction.Count > 0) ViewData["ResultsProduction"] = resultsProduction;
                    break;

                case "SearchParts":
                    ViewBag.Message = string.Format("Results for \"{0}\" in Parts", searchKey);
                    resultsPart = new List<Part>();
                    foreach (Part part in db.Parts.ToList())
                    {
                        Match matchCharacter = rx.Match(part.Character);
                        Match matchTitle = rx.Match(part.Production.Title);
                        Match matchName = rx.Match(part.Person.Name);
                        if (matchCharacter.Success || matchTitle.Success || matchName.Success)
                        {
                            resultsPart.Add(part);
                        }
                    }
                    resultsPart = resultsPart.Distinct().ToList();
                    Highlight(resultsPart, pattern, highlightedKey);
                    if (resultsPart.Count > 0) ViewData["ResultsPart"] = resultsPart;
                    break;
                default:
                    break;
            }
        }

        // This method has overloads for passing in different list types
        //It works by wrapping the search key in a span tag that styles it differently from the rest of the text

        private void Highlight(List<CastMember> resultsCast, List<Production> resultsProduction, List<Part> resultsPart, string pattern, string highlightedKey)
        {
            var yearJoinedString = new List<string>();
            for (int i = 0; i < resultsCast.Count; i++)   //YearJoined must be converted to text to highlight it properly. A separate list is created, then added to the viewbag.
            {
                resultsCast[i].Name = Regex.Replace(resultsCast[i].Name, pattern, highlightedKey, RegexOptions.IgnoreCase);
                resultsCast[i].Bio = Regex.Replace(resultsCast[i].Bio, pattern, highlightedKey, RegexOptions.IgnoreCase);
                yearJoinedString.Add(resultsCast[i].YearJoined.ToString());
                yearJoinedString[i] = Regex.Replace(yearJoinedString[i], pattern, highlightedKey, RegexOptions.IgnoreCase);
            }
            ViewBag.YearJoined = yearJoinedString;
            foreach (Production production in resultsProduction)
            {
                production.Title = Regex.Replace(production.Title, pattern, highlightedKey, RegexOptions.IgnoreCase);
                production.Playwright = Regex.Replace(production.Playwright, pattern, highlightedKey, RegexOptions.IgnoreCase);
                production.Description = Regex.Replace(production.Description, pattern, highlightedKey, RegexOptions.IgnoreCase);
            }
            foreach (Part part in resultsPart)
            {
                part.Character = Regex.Replace(part.Character, pattern, highlightedKey, RegexOptions.IgnoreCase);
                if (!part.Production.Title.Contains("span id=")) // these if statements prevent the span tag from being applied twice. 
                {
                    part.Production.Title = Regex.Replace(part.Production.Title, pattern, highlightedKey, RegexOptions.IgnoreCase);
                }
                if (!part.Person.Name.Contains("span id="))
                {
                    part.Person.Name = Regex.Replace(part.Person.Name, pattern, highlightedKey, RegexOptions.IgnoreCase);
                }
            }
        }

        private void Highlight(List<CastMember> resultsCast, string pattern, string highlightedKey)
        {
            var yearJoinedString = new List<string>();
            for (int i = 0; i < resultsCast.Count; i++)   //YearJoined must be converted to text to highlight it properly. A separate list is created, then added to the viewbag.
            {
                resultsCast[i].Name = Regex.Replace(resultsCast[i].Name, pattern, highlightedKey, RegexOptions.IgnoreCase);
                resultsCast[i].Bio = Regex.Replace(resultsCast[i].Bio, pattern, highlightedKey, RegexOptions.IgnoreCase);
                yearJoinedString.Add(resultsCast[i].YearJoined.ToString());
                yearJoinedString[i] = Regex.Replace(yearJoinedString[i], pattern, highlightedKey, RegexOptions.IgnoreCase);
            }
            ViewBag.YearJoined = yearJoinedString;
        }

        private void Highlight(List<Production> resultsProduction, string pattern, string highlightedKey)
        {
            foreach (Production production in resultsProduction)
            {
                production.Title = Regex.Replace(production.Title, pattern, highlightedKey, RegexOptions.IgnoreCase);
                production.Playwright = Regex.Replace(production.Playwright, pattern, highlightedKey, RegexOptions.IgnoreCase);
                production.Description = Regex.Replace(production.Description, pattern, highlightedKey, RegexOptions.IgnoreCase);
            }
        }

        private void Highlight(List<Part> resultsPart, string pattern, string highlightedKey)
        {
            foreach (Part part in resultsPart)
            {
                part.Character = Regex.Replace(part.Character, pattern, highlightedKey, RegexOptions.IgnoreCase);
                part.Production.Title = Regex.Replace(part.Production.Title, pattern, highlightedKey, RegexOptions.IgnoreCase);
                part.Person.Name = Regex.Replace(part.Person.Name, pattern, highlightedKey, RegexOptions.IgnoreCase);
            }
        }
    }
}

```

