# OutStagram

## Ready to build your own Instagram app? Let’s Start!

During this exercise, we'll be building an app with some functionalities
similar to Instagram.

### Required Libraries

For this we need to install some libraries. On the top of Studio's window
there is a tab named forge, which will give you access to the open source
component repository for OutSystems.

Search for camera and install the plugin.


You should see some apps being installed. Do not worry if the app
disappears at the end, you may have an option not to show supported
applications enabled.

The next one is Image Utils Reactive


Again you should see the installation taking place.

### OutStagram Files

Download the following files:


- [Logo](https://github.com/OutSystemsEdu/OutSystems-Project-Ideas/raw/main/CONTRIBUTING/Intermediate/images/OutStagram.png)
- [Application](https://github.com/OutSystemsEdu/OutSystems-Project-Ideas/raw/main/CONTRIBUTING/Intermediate/OutStagram.oap)

### Install OutStagram

On Studio menu, select Environment and Open Files


Flip the type to OutSystems Application Pack and select OutStagram.OAP

```
And proceed with the installation.
```
In the end you should have something similar to...


### Create the application

In OutSystems, we can start developing apps from predefined templates. Templates can contain
screen layout definitions, CSS, and other logic and data. Templates encourage the reuse of
important components and can help create a consistent look and feel across multiple apps.

```
● Create an App
```
```
● Start from scratch
```
```
● Select Phone App
```

● Give a name to the application
● Upload the logo

● Create Module


You should see something similar to this

### Reference OutStagram Core

We will be needing some pre-made methods that exist on the OutStagram core module.


Click the manage dependencies icon

Search and select **Image_Utils_Reactive** and check thebox for the **Resize** method.

Repeat the process and search and select the **CameraPlugin** and select the **TakePicture** method.


Repeat the process and search and select the **OutStagramCore** and then check all the boxes,
including the **server actions** and **entities**. Click therefresh all button (if enabled) and then the
apply button.


Go ahead and publish the application for the first time using the one click publish button.

Give it a few seconds to finish. When done you should see a similar screen to this:


### Creating the Home screen

Drag and drop an empty screen to the **Main Flow**


And then select empty screen and call it Home and press create screen


The screen already has some structure to it, this is due to the standard empty template and you
can always design your own.


Now let's add some data to the screen. Flip to the data tab, expand **OutStagramCore** and drag
and drop the **post** entity to the content area.


The platform tries to understand what you are trying to do. In this case when dragging an entity
to a screen is interpreted as if you want to display that data so an accelerator kicks in and does
several things for you, all of which you could do step by step manually.

```
● Created a query to grab the data from the Post entity
○
● Created a pagination so that all the data is not loaded at the same time
○
● Created a list widget to support the data iteration
○
● Created a list item with the single item content
○
```


#### Arranging the data display

Select the image, select the container around it and expand to the full width of the screen.


Then select the image again and delete the style class.


Delete all attributes from the screen except **Description** , **Date** and **User Name** and re-order
them to show User Name, Description and Date. Also remove the bold classes from **Description**


#### Sort the data

We want to see the latest posts first so let's sort the data. If you do not see the GetPosts query

```
flip the view back from the widget tree
```

#### Publish the app

You should be ready to publish the application. Put a nice title on your screen and click the big
green One Click Publish button.


The publishing process consists of several steps.

It sends your code to the server and does the appropriate check in, it generates and compiles

##### standard code from your visual model and finally itdeploysthe application to your server making

it available for use.

#### Testing the Application

You can now test your application in a few different ways.

Using your desktop browser, just click the big blue button which will open a browser with a
frame simulating your phone form factor.


The other option is to test the application on your phone as aPWA(Progressive Web App). In
order to do so we need to configure it.


Now you just need to scan that QR Code or click the open in browser link and type the url on your
phone.

A third option is to generate a nativecordova application.This will allow you to download an IPK
or IPA (Android/IOS package) which can be installed on your phone or sent to the app stores for
distribution.

If you have an Android phone, go ahead and click the and then the generate
app button. A few minutes later you should get another QR Code that will allow you to download
and install the App to your Android phone. Depending on your phone security settings you may
need to trust yourself as an application publisher.


If you have an IOS phone, Apple makes things a bit more complicated. You will need:

```
● An Apple Developer License
● To choose your build type
● Generate you publisher certificate according to your profile
● Generate a provisioning profile for your app
```
If you want to know more about publishing OutSystems applications for IOS, take a look at the
documentation.

Finally whatever method you choose, you'll be presented with a login screen. You can login using
your community email and password, or you can use one of the test users created for this
application (their password is outsystems):

```
● roz
● mike_wazowski
● James_sullivan
```

You should see something similar to this.


### Refining the Home Screen

#### Implementing a like status

Every social media app needs a like button. In our example we'll use a heart to allow the user to
like and give a visual representation.

Flip to the widget tree, expand content, list, list-item, content and drag and drop an **if** right
between the image-container and the other Container.

An if is a visual widget that displays the upper square if the condition is true and the lower one if
false.

Next search the widgets for icon and drag it into the upper square of the if.


Repeat the process for the lower if square, but this time select an empty heart.

And of course we need a red heart, not a blck one. So select the full heart and choose a nice red
color. The style editor is a quick way to change the style of an element. However you can define
classes using pure CSS.


Now we need to collect the information for the likes from the database. Our initial data fetch is
only returning the posts, we also need to get the likes for each post. So go ahead, flip the widget

tree and double click the query.

Then flip to the sources, click the add source and select **PostLike.**


Now let's take a look at how the entities relate to each other.


All posts Only With users and posts Only With likes. But this is not what we want...

We want all posts Only With users and posts With or Without likes. And beyond that if a post
does have a like then show me only my likes.

So flip the join type to With or Without and double click the expression to type the missing
condition "and PostLike.UserId = GetUserId()".

When you're done go back to the home screen by either clicking the or double

clicking the

Finally select the if and set the condition to "GetPosts.List.Current.PostLike.Id <>
NullIdentifier() ".


You should be able to publish your application by using the

#### Implementing the like button

Now let's implement the code necessary to accept the user input when either hearts are pressed.
Right click the home screen and select add client action and rename your action to LikeOnClick.

Create a new input parameter by right clicking on the LikeOnClick ( ), name it
PostID and make sure the parameter type is Post Identifier.


Now flip to the logic tab, locate the ToggleLike action and drag it to the line of your flow below
the start. Fill in the parameters for UserId and PostId.


Next grab a refresh widget, drop it on the line below the ToggleLike and select the GetPosts
query.


Now go back to the Home screen, right click the if and link it to the LikeOnClick action. Fill in the
PostID property with "GetPosts.List.Current.Post.Id".


Go ahead publish the app and then click the testbutton. You can now interact with the
heart by clicking on it.



### Implementing a Like counter

We also want to have the total number of likes for each post. For this feature we'll use a slightly
different technique by introducing blocks. In OutSystems blocks are pieces of UI that can be
reused multiple times on many screens.

#### Creating the Block

Double click the , drop a block next to the homescreen, rename it to Likes and open it
with double click.

Then add a new parameter named PostID and make sure it is of type Post Identifier.


Next let's grab from the database the number of likes for a particular post. Right click on the
Likes block and select Fetch Data from Database.


Now, click on the screen and select the PostLike entity.


Now, we filter the likes for the PostID passed through the parameter.


#### Displaying the Like count

Back to the Like block, grab an expression and select the count under the query.


Change the example to 99 and type " likes" using thekeyboard next to the expression.


Select the expression and add 10px of left margin from the styles tab.


#### Using the block on the Home screen

On the home screen we need to adjust the UI to make room for the likes counter. Flip to the
widget tree, locate the container below the link, flip to the styles and set the width to Fill.


The next step is to flip back the widget tree, grab the like block, drop it next to the heart and fill in
the PostID parameter.


Go ahead publish the app and then click the testbutton. You can now interact with the
heart by clicking on it and should see the counter being updated.


### Creating a new post

The next step is to make a screen where the user can create a new post, taking a picture and
typing some content.

#### Taking a picture

Navigate back to the and drag a new screen intothe canvas.


Select the empty template and name the screen NewPost.


Fill in the title and drag a new icon to the upper left placeholder. Choose the camera icon.


Right click on the camera icon and choose new client action.


Rename the action to **CaptureImage** and double clickit to open the code. Also create 2 new local
variables. **Picture** with type **Binary Data** and **Description** with type **Text**.


Now let's implement the logic to take a picture and store it.

Flip to the logic tab and grab the TakePicture action dropping it into the flow. While you're at it
grab an **If** and drop it below the **TakePicture**. Asloswap the if connectors with right click, so that
the **true** branch continues **down**. On the **If** conditionselect **TakePicture.Success**.


Next, drop a message next to the **If** and link a falsebranch from it. Drop an **End** below the
message and link from it. Select the **Message** and chooseTakePicture.Error.Message.


Now we are going to reduce the picture size to save some bandwidth.

Drag the Resize action below the If into the true branch. On the **image** parameter select
TakePicture.ImageCaptured and on the **MaxWidthOrHeight** set it to 800.


Finally let's assign the resized image to the local variable picture.


Go ahead publish the app

#### Creating the UI

Navigate back to the NewPost screen by double clicking

Locate the **Form** , **Image** and **TextArea** widgets and dropit on the screen. Set
the Image property type to Binary Data and the Image Content property to
Picture.

Set the TextArea property Variable to Description


We also need some buttons to save or cancel the post.

Drag a container below the text area, then drag 2 buttons inside the
container.

Change the buttons name to Post and Cancel and make the post button
bigger in length.


Select the Cancel button and set its On Click property to MainFlow\home


Select the Post Button and set its On Click property to New Client Action.


#### Saving the post to the database.

Flip to the logic tab and locate the NewPost action under Server
Actions/OutStagramCore and drop it below the If. On the properties set the
**UserId** to GetUserId(), **Picture** to Picture and **Description** to Description.

Drag a **Destination** widget on top of the **End** node andchoose Main
Flow/Home as the destination property.


#### Linking the New Post Screen

All we need to do now is to have a way for the user to access the new post
screen from the home screen.

Navigate to the Home screen and drop an icon on the top left placeholder.
Choose the plus sign as the image. Also right click on the icon and choose
Link to, **MainFlow\NewPost**.


Go ahead publish the app

### Testing the app

Because of the camera it is best to test the app on a mobile phone and
because this is a PWA application you can use it without any installation.

Navigate to the main tab, switch to the distribute tab and scan the
progressive web app QR Code. Or just type the URL directly on your phone.


#### Testing on the desktop browser

You can still test on your desktop browser. The experience will be a bit
different since the browser will ask you for a file when you try to take a
picture.



## What's Next?

This was a slightly simplified version of the exercise and you can check out
the full version at the OutStagram application.



Check out the differences and implement them on your app.
