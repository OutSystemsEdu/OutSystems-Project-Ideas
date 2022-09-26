# OutStarter Workshop



- Building the backend - OutStarter Service
   - Let’s get started with a Service
   - Consume the RSS API
   - Create the Database
   - Load Project Data
   - Parsing Project Data
   - Parsing Project Description
   - Parsing Funded
   - Parsing Goal
   - Parsing the Publish Date
   - Project Load
   - Loading the Data
   - Debugging the Data Load
- Building the Frontend - OutStarter
   - Creating the home screen
   - Get the Project Data
   - Composing the Home screen
   - Completing the Home screen
- Allowing Patrons to Fund the Projects
   - Project Data
   - Add Project Data to the Fund Screen
   - Fund Project button
   - Funding the Project
   - Collecting the Data
   - Validating the user input
- More Backend coding
   - Fund the Project
   - Patron Data
   - Funding Data
- Finish the Frontend
   - Completing the funding logic
- Giving Feedback to the User


## Introduction

The objective of this exercise is to provide to a developer the OutSystems Low-Code

development experience.

The use case is similar to Kickstarter.

A user can browse projects and become a patron by funding a project.

The frontend will have a couple of screens, a project list with project information, and a funding

screen where the user can enter its details and an amount.

The backend provides all the business logic and also handles the database.

The data for this exercise has its origins on a scrapper service KickTraq.com which monitors

Kickstarter.com and sends an RSS feed.

Enjoy.

## Building the backend - OutStarter Service

We’ll begin by creating the OutStarter Service.

This module will hold all the server side logic and databases.

The first problem is that Kcikstarter does not provide any API to interact with their data and

methods. However there is a serviceKicktraq.comthatprobably scrapes Kickstarter and

collects data from it.

They have several RSS feeds and we’ll be using thelatest games

(https://www.kicktraq.com/categories/games/latest.rss)

The second problem is that an RSS feed is a XML format and in OutSystems although there is

a complete XML library it is much easier to work with JSON format.

In order to solve this we’ll be using therss2json.comAPI.

This API with the entry point ( **https://api.rss2json.com/v1/api.json** )accepts a single parameter

(rss_url) which is the encoded URL for the RSS feed.

### Let’s get started with a Service

Create a new application, from scratch and select the “Service” type.


Name it OutStarter Service, choose a main color or upload a nice icon and create a single

module named OutStarterService.


### Consume the RSS API

Consume a new single method api.


On the get method, paste the URL

**https://api.rss2json.com/v1/api.json?rss_url=https%3A%2F%2Fwww.kicktraq.com%2Fcateg**

**ories%2Fgames%2Flatest.rss**

Flip to the test tab and press the test button. You should get a response from the server.

Then press the Copy to Response body button.

You should get something like this, then click finish:


OutSystems introspects the JSON output and automatically parses it into an OutSystems

structure so you can easily work with the data on your application.

The structure we’ll be using most is the Items list, take a look at its properties and attributes.

To finish this section, go ahead and publish.


### Create the Database

In order to keep track of the funding and projects let’s use some database entities to store this

data.

We have Project, Fund and Patron to hold this data. The rules are a Project can be funded

multiple times by multiple Patrons and similarly a Patron can Fund multiple Projects.

For the Project we have:

Be sure to adjust the data type and length like this:

```
● Title - Text - 500
● PubDate - Date Time
● Thumbnail - Text - 500
● Description - Text - 5000
● Goal - Currency
```
The missing attributes don’t have relevant information that we can use.

For the Patron we just need Name (Text - 200) and EMail.


Go ahead and publish

Another cool feature of OutSystems is the ability to design your database using an entity

diagram, let’s try it.

Create a Fund entity, flip to the Entity Diagrams, drag and drop all your entities on the diagram.

And now let’s set the Fund with the Project and Patron references (Foreign Keys). Grab the

caret from the Project and drop it on the Fund. Do the same for Patron.


And just like that you’ve established the relationships between all entities. Also add an Amount

attribute to the Fund entity as currency.

Finally make sure all your entities are Public. We need this to be able to use them when we

create the frontend.


Go ahead and publish

### Load Project Data

Now we get to actually load the data from the API to the database.

Flip to Logic and add a server action named Project_Load and build something that looks like

this. You’ll need to create a new User Exception and name it OutStarterService


Let’s look at the details of each node.

```
GetApi.Response.
Status="ok"
```
```
GetApi.Response.
message
```
```
GetApi.Response.
Items
```

```
Project.Title =
GetApi.Response.
Items.Current.Ti
tle
```
```
GetProjectsByTit
le.List.Current.
Project.Title
```
```
GetApi.Response.
Items.Current.Ti
tle
```
```
GetProjectsByTit
le.List.Current.
Project.Thumbnai
l
```
```
GetApi.Response.
Items.Current.Th
umbnail
```
```
GetProjectsByTit
le.List.Current.
Project
```
Go ahead and publish

### Parsing Project Data

You probably noticed that the data we can use directly is limited to Title and Thumbnail. The

rest of the data we are interested in either needs a conversion function like the PubDate or is

aggregated in the Description, e.g.


```
"description": "\n<a title=\"Fantasy Mansion 1\"
href=\"http://www.kicktraq.com/projects/fantasydesigns/fantasy-mansion-
1-pay-what-you-want/\"><img height=\"120\" width=\"160\"
title=\"Fantasy Mansion 1\" alt=\"Click here to view Fantasy Mansion
1\"
src=\"https://ksr-ugc.imgix.net/assets/036/683/203/ea19a13b94edc6f899eb
c5166a6e352c_original.png?ixlib=rb-4.0.2&amp;crop=faces&amp;w=160&amp;h
=90&amp;fit=crop&amp;v=1647466236&amp;auto=format&amp;frame=1&amp;q=92&
amp;s=0bf5a51e672e92165ca2b607b71d61a3\"></a><br>3D Printable STL
Files, Terrain for Tabletop, Role Playing Games, Wargames and RPG -
[currently $174 (166%) of $105 goal]",
```
There is also a lot of HTML code embedded that we want to get rid of (more on this later),

leaving us with:

```
3D Printable STL Files, Terrain for Tabletop, Role Playing Games,
Wargames and RPG - [currently $174 (166%) of $105 goal]
```
For this we’ll need the Forge Extension HTML Utils to make it easier.

Flip to the forge tab, search for HTML Utils.

Next select HTML Utils and click install. This will install the extension on your system.


Once the installation is finished you should be able to see a new application in Service Studio.

Search for HTML Utils.

Back to development, we need to reference the HTML Utils library and its HTMLtoText method.


### Parsing Project Description

Flip to the Logic tab and create a new Server Action called Project_Description_Parse.

In this method we’re parsing the description from the API into a text description.

So our method should have an input parameter named Text_Description of type text, and an

output parameter Description, also of type text.

We’ll use a combination of 2 built in methods to handle strings.

TheSubstr(original string, start position, length)will return a string from the

original string starting on the start position and ending the length after the start position.

The Index(string, search)will return the positionof search inside the string.

As an example after striping the HTML:

```
With the special design, they can transform into joint doll bodies to
play with you at any time, and there are matching little outfits. -
[currently $0 (0%) of $1,000 goal]
```
We have a substring starting from 0 until we find the string " - [currently".

Use this expression on the Description assignment:

Substr(Text_Description,0,Index(Text_Description," -

[currently",startIndex:,searchFromEnd:True,ignoreCase:True))


Go ahead and publish

### Parsing Funded

Similar to the previous method, but for parsing the funded part of the description. The output

parameter Funded is of type currency.

As an example after striping the HTML:

```
With the special design, they can transform into joint doll bodies to
play with you at any time, and there are matching little outfits. -
[currently $0 (0%) of $1,000 goal]
```
For Funded we start at the end of " - [currently"from the description and end on the " (".

We’ll do this in multiple steps to make it easier to follow.


Substr(Text_Desc
ription,
Index(Text_Descr
iption,"[current
ly
",startIndex:,se
archFromEnd:True
,ignoreCase:)+
,999)

Substr(aux,0,Ind
ex(aux,"
(",startIndex:,s
earchFromEnd:Tru
e,ignoreCase:))


```
Replace(aux,",",
"")
```
```
TextToDecimal(au
x)
```
Go ahead and publish

### Parsing Goal

Similar to the previous method, but for parsing the goal part of the description. The output

parameter Goal is of type currency.

As an example after striping the HTML:

```
With the special design, they can transform into joint doll bodies to
play with you at any time, and there are matching little outfits. -
[currently $0 (0%) of $1,000 goal]
```
For Goal we start at the end of " of " from the descriptionand end on the " goal]".

We’ll do this in multiple steps to make it easier to follow.


Substr(Text_Desc
ription,
Index(Text_Descr
iption," of
",startIndex:,se
archFromEnd:True
,ignoreCase:)+5,
999)

Substr(Aux,0,Ind
ex(Aux,"
goal]",startInde
x:,searchFromEnd
:,ignoreCase:))


```
Replace(aux,",",
"")
```
```
TextToDecimal(au
x)
```
Go ahead and publish

### Parsing the Publish Date

This is a straightforward conversion, where the Input parameter is the test PubDate and the

Output is a Date Time parameter. Use this expression on the assign node:

TextToDateTime(Text_Date)


Go ahead and publish

### Project Load

We can now improve the project load with the parsed data.


GetApi.Response.
Items.Current.De
scription

GetApi.Response.
Items.Current.Pu
bDate

HtmlToText.Text

HtmlToText.Text


```
GetProjectsByTit
le.List.Current.
Project.PubDate
=
Project_Date_Par
se.DateTime
```
```
GetProjectsByTit
le.List.Current.
Project.Descript
ion
=
Project_Descript
ion_Parse.Descri
ption
```
```
GetProjectsByTit
le.List.Current.
Project.Goal
=
Project_Goal_Par
se.Goal
```
Go ahead and publish

### Loading the Data

We now can run the Project_Load method.

For this we’re going to use another cool feature of OutSystems, its capability to run a method

based on a timer or a publish.

Flip to the Processes tab, add a new Timer and point it to the Project_Load method. On the

Schedule select When Published.


Go ahead and publish

### Debugging the Data Load

If all is well you should be able to see some project data on the Project entity. Right click the

Project entity and select View or Edit Data:

If not, get ready for debugging.


This will start your app in debugging mode. You can execute the code step by step while

inspecting the variables.

Since this is a timer that runs on publish, publish your app while in debug mode and wait a few

seconds for the code to start executing.


You can advance the position of the code being executed step by step using the Step Over

(F10) and if you want the execution details of a method you can Step Into (F11). Let’s step into

the Project_Description_Parse.

We can see the result of the description parsing. Press the Stop button to stop the debugger.


## Building the Frontend - OutStarter

We can now start building the frontend.

Let’s create a mobile application for this.

### Creating the home screen

Now we add a new screen to our application.

On the main flow, drag a new screen on the empty area and select Empty.

Name this screen Home and press the create button.


The final result should be an empty screen that looks like this.

Make sure that under the role you check the anonymous box, this will remove the need to login

to your app.

Also add a title, OutStarter


Go ahead and publish

Before we test, let’s turn on the PWA distribution so we can test both on the browser and on

our mobile phones.

Scan the QR Code using your phone or go back and press the Open in Browser button.

Testing on the browser looks like this.


### Get the Project Data

In order to display the information on the screen we need to fetch it from the database we’ve

created.

We need to reference the entities we exposed before.


Now we can fetch the data from the database and add the project entity.

Go ahead and publish

Next we need to add the Fund entity so we can calculate how much funding each project

already has.


Notice how the platform takes the relationships previously established and tries to guess what

you want to do.

A project can have multiple funding (a one to many relationship) but it also can have none.

The relationship between project and fund that we want is **With or Without** (Left Join in SQL).

It also automatically picked up which attributes to use and built the condition for you

(Project.Id = Fund.ProjectId).

This time it got it right but you should always check if it is what you want.

Also note that once the aggregate refreshes you can look at the actual executed SQL.

```
SELECT TOP (32) [ENFUND].[ID] o0
, [ENFUND].[PROJECTID] o1
```
```
, [ENFUND].[PATRONID] o2
, [ENFUND].[AMOUNT] o3
```

```
, [ENPROJECT].[ID] o4
```
```
, [ENPROJECT].[TITLE] o5
, [ENPROJECT].[PUBDATE] o6
```
```
, [ENPROJECT].[THUMBNAIL] o7
, [ENPROJECT].[DESCRIPTION] o8
, [ENPROJECT].[GOAL] o9
```
```
FROM ([OWTDH2000].dbo.[OSUSR_2E4_PROJECT] [ENPROJECT]
Left JOIN [OWTDH2000].dbo.[OSUSR_2E4_FUND] [ENFUND]
```
```
ON ([ENPROJECT].[ID] = [ENFUND].[PROJECTID]))
```
Now we need to sum the amount and group by all the other attributes from Project.

Locate the Amount, right-click and select sum

Then select all the attributes from the project, including the ID, right-click and select Group BY.


The final result should look like this, (the description and the image thumbnail are hidden).

Go ahead and publish


### Composing the Home screen

There is a lot of drag and drop on this section, so let’s do this in smaller steps.

Flip to the Interface tab, double click your Home screen. Flip to the widget tree and expand the

content section.

Drag the widgets from the toolbar on the left, you may need to search for them since it is a very

long list.

If you drop the widget in the wrong location, you can drag them into place on the widget tree

itself. You can also drop the widgets directly on your Home page. Play with this until you feel

comfortable manipulating the screen widgets.

For simplicity we didn’t repeat the arrows, but drag all widgets visible on the widget tree in this

image.

Now we need to set the properties of the widgets.


GetProjects.List

"image-container
"

External URL

GetProjects.List
.Current.Thumbna
il


```
"margin-top-base
"
```
```
GetProjects.List
.Current.Title
```
```
"margin-top-s"
```
```
GetProjects.List
.Current.Descrip
tion
```
Go ahead and publish

When you test the app you should see some information similar to:


### Completing the Home screen


"margin-top-base
"

GetProjects.List
.Current.AmountS
um /
GetProjects.List
.Current.Goal*10
0

"margin-top-base
"

FormatCurrency(G
etProjects.List.
Current.Goal,
"$", 0, ".",
",")

```
pledged
```

```
FormatPercent(Ge
tProjects.List.C
urrent.AmountSum
/
GetProjects.List
.Current.Goal,0,
"")
```
```
70%
```
```
FormatCurrency(G
etProjects.List.
Current.AmountSu
m, "$", 0, ".",
",")
```
```
FormatDateTime(G
etProjects.List.
Current.PubDate,
"d MMM yyyy
HH:mm")
```
Go ahead and publish

When you test the app you should see some information similar to:


## Allowing Patrons to Fund the Projects

The whole point of OutStarter is to allow patrons to fund the projects.

For this we’ll need to build a screen where a patron can contribute with a certain value to a

certain project.

Flip to the elements tab and open the main flow. Drag a Screen to an empty space, select the

empty template and name it Fund.


### Project Data

Similar to what we’ve done on the project list we need to get the data for a specific project.

For this right click on the fund screen create a ProjectID input variable of type Project Identifier.

Next right click again and select Fetch Data from Database. Now if we drag the ProjectID into

the empty screen it will automatically create the query for you. This is only possible because

ProjectID is of type Project Identifier, which means that you are trying to locate the information

using a primary key which in turn can only return 1 or none records.


Next we’ll add the Fund entity and do the Sum on the Fund Amount and the Group by on all

Project attributes.


### Add Project Data to the Fund Screen

For this we are going to use a shortcut that is not really a best practice. We’ll partially copy the

Home screen into the Fund.

Open the Home screen and flip to the widget tree. Then locate the margin container and copy

it.


Then navigate to the Fund screen, flip to the widget tree, expand content and paste.

Now we need to fix the widgets, move the Content card into the Margin container and delete

the List:

And now let’s fix all the references.


```
GetProjectById.L
ist.Current.Thum
bnail
```
```
GetProjectById.L
ist.Current.Titl
e
```
```
GetProjectById.L
ist.Current.Desc
ription
```
```
GetProjectById.L
ist.Current.Amou
ntSum /
GetProjectById.L
ist.Current.Goal
*100
```
Continue to the Columns Medium Right.


FormatCurrency(G
etProjectById.Li
st.Current.Goal,
"$", 0, ".",
",")

FormatPercent(Ge
tProjectById.Lis
t.Current.Amount
Sum /
GetProjectById.L
ist.Current.Goal
,0,"")

70%


```
FormatCurrency(G
etProjectById.Li
st.Current.Amoun
tSum, "$", 0,
".", ",")
```
```
FormatDateTime(G
etProjectById.Li
st.Current.PubDa
te, "d MMM yyyy
HH:mm")
```
Go ahead and publish

### Fund Project button

We now need to link both screens. We’ll add a Fund Project button to the project on the Home

screen which will navigate to the Fund screen passing the ProjectID as a parameter.


Make sure you’ve set the Fund screen role to anonymous.

Go ahead and publish

You should now be able to navigate between the Home and the Fund pages.


### Funding the Project

In order to fund the project we need to collect data from the patron (name & email) and also the

amount being funded.

We’ll create 3 local variables. LocalPatron of type Patron, LocalFund of type Fund, and finally

IsComplete of type boolean.

Next we’ll add an if to the screen so that it displays different widgets depending on if the

funding is complete or not.

Drag a container to the Margin Container and an If widget into the container. We’ll use the

IsComplete boolean variable on the If.


### Collecting the Data

To collect the data we’ll need some user inputs. Drag a form to the False section of the If.


Next drag the LocalPatron variable into the form and just the Amount under LocalFund in

between the Email and the Save button. Change the button text to Fund Project and make it

bigger.

Finally select the button, and create an OnClick, New Client Action.


Go ahead and publish

So now when you click the fund project, you can see the project details and the form to collect

your data.

### Validating the user input

Let’s build some code, open the FundProjectOnClick screen action and build the flow.

As you drag the nodes, connect them as the image, notice which ones are true and false from

the If. You can always right click the If and swap the connectors.


Trim(LocalPatron
.Name)<>""

False

"Name is
mandatory"


```
Trim(LocalPatron
.Email)<>"" and
EmailAddressVali
date(LocalPatron
.Email)
```
```
False
```
```
"Invalid email"
```
```
LocalFund.Amount
>0
```
```
Form1.Valid
```
Go ahead and publish

Now if you try to fund a project without the correct data format you get errors on your form.


## More Backend coding

We now need to send this information back to the server so it can register the funding from a

patron. The Fund entity requires a ProjectID, a PatronID and the Amount.

### Fund the Project

We’ll start by creating a Service Action. These are automatically implemented and secure REST

interfaces that can be consumed by any other module on your application. Think of it as the

building blocks for a service oriented application or micro services architecture.

This Service Action will accept ProjectID of type Project Identifier, PatronName (text),

PatronEmail (Email), Amount (Currency) and output a FundID (Fund Identifier).


### Patron Data

First we need to check if the Patron already exists, create a new one if not, and return the

PatronID.

Create a Server Action named Patron_Check with PatronName (text), PatronEmail (Email) as

inputs, and PatronID (Patron Identifier) as output.

Let’s build this step by step, starting with the aggregate.


Next we set the Patron record.


Next we update the database.

And finally we return the PatronID


Back to the Fund_Project you can now use the Patron_Check.

Go ahead and publish


### Funding Data

To save the Fund on the database we’ll need another Server Action.

Create a Server Action named Fund_New with ProjectID (Project Identifier), PatronID (Patron

Identifier), Amount (Currency) as input parameters, FundID (Fund Identifier) as an output

parameter, and a Fund (Fund) local variable.

Again, step by step, starting with the Assign.

Next the Create Fund.


And the FundID

We can complete the Fund_Project Service Action.


Go ahead and publish

## Finish the Frontend

We can now go back to the frontend and finish the funding process. First we need to reference

our Service Action so we can use it on the front end.

If there is a Refresh All button make sure you press it!!!


### Completing the funding logic

Go back to the FundProjectOnClick screen action under the Fund screen.

Next, add the Fund_Project Service Action which will save the funding on the database.

We should also refresh the screen data source so the user can immediately see the changes.


Finally we flip the IsComple boolean variable so the screen knows the process is complete.

```
Go ahead and publish
```
## Giving Feedback to the User

Now that the process is complete we need to send this feedback to the user.

The IsComplete variable allows us to notify the screen that the process is done. We’ll create a

nice feedback panel on the True section of the If.


And set the properties of the widgets.

Go ahead and publish

Let’s fund a project!


Congratulations, you’ve finish the exercise.
