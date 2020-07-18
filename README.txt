Summary:

I just completed a two-week sprint working on a C# MVC Entity Framework Code First project. Throughout the sprint, I completed several stories which are outlined below:

Messages Delete Modal:

For this story I had to create a modal with a custom animation to display basic information about the message that the user wanted to delete. This was done from both the inbox list and the message itself. I used JavaScript, Bootstrap, and CSS to accomplish this.

Messages Including Previous Content:

Before I worked on this story, when a user went to reply to a message, only a blank message was shown (the user could not see the previous messages). I fixed this using JavaScript and jQuery. This works on both the inbox list reply button and the message itself. 

Refactor Parts Section:

Before working on this story, the code to accomplish this was 200+ lines. The previous developer had trouble with the PositionEnum, as the page was supposed to display the CastMembers in a different order than the Enum. I fixed this by writing one loop that used a list of integers to dictate which Enum was next in line. This was done using C# Razor.

Add RentalRequest Properties:

This was a relatively simple story that required me to add properties to RentalRequest model, update database, then add appropriate Razor to the Create and Edit cshtml pages.

Award Preview:

This story required me to create a preview of what the award would look like as they were typing it. I accomplished this using jQuery and html. 

Dynamically Load Cast Members:

Before starting this story, the cast members were hard-coded into the About page of the website. I changed this to load dynamically. This was done using C#. 

Enable Upload - Prevent Duplication for photos:

For this story I had to enable the user to upload a new photo on the Edit page (this had been disabled). I then had to prevent the user from uploading (from either the create or edit page) and image that already existed in the database and if the image already did exist, throw a modelstate error. I did this using C#. 

Overall this project was a great experience and I had a lot of fun working with a team. We had our daily scrum every morning and would coordinate on the actions of the upcoming day. I definitely learned a lot and am looking forward to working on more projects. 