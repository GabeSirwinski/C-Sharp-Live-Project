# C# Live Project

I just completed a two-week sprint working on a C# MVC Entity Framework Code First project. Throughout the sprint, I completed several stories which are outlined below:

**Messages Delete Modal:**

![Image of Yaktocat](https://github.com/GabeSirwinski/C-Sharp-Live-Project/blob/master/MessagesPage/MessageDelete.png)

For this story I had to create a modal with a custom animation to display basic information about the message that the user wanted to delete. This was done from both the inbox list and the message itself. I used JavaScript, Bootstrap, and CSS to accomplish this.

**Messages Including Previous Content:**

![Image of Yaktocat](https://github.com/GabeSirwinski/C-Sharp-Live-Project/blob/master/MessagesPage/ReplyMessage.png)

Before I worked on this story, when a user went to reply to a message, only a blank message was shown (the user could not see the previous messages). I fixed this using JavaScript and jQuery. This works on both the inbox list reply button and the message itself. 

**Refactor Parts Section:**

```cshtml
<!-- Parts Card Section visible when screen size is bigger-->
        <div class="card mr-4 d-none d-md-block" id="cardss">
                <!-- when screen size is med or bigger this will display. It is otherwise hidden-->
            <div class="card-header" id="head">
                    <b>Parts</b>
            </div>
                <ul class="list-group list-group-flush text-black-50">
                    @{var part = new TheatreCMS.Enum.PositionEnum();}
                    @{List<TheatreCMS.Models.Part> parts = new List<TheatreCMS.Models.Part>();}
                    @foreach (int position in ViewBag.Positions)
                    {
                        parts.Clear();
                        part = (TheatreCMS.Enum.PositionEnum)position;
                        parts = Model.Parts.Where(x => x.Type == part).ToList();
                        if (parts != null && parts.Any())
                        {
                            <li class="list-group-item">
                                @(part)s
                                <dl>
                                    @foreach (var item in parts)
                                    {
                                        if (item != null)
                                        {
                                            <dd class="col">
                                                <a href="/Part/Details/@item.PartID">@Html.Raw(@item.Character)</a>
                                            </dd>
                                        }
                                    }
                                </dl>
                            </li>
                        }
                    }
                </ul>
            </div>
        <!-- End Parts Card Section-->
        </div>
```

Before working on this story, the code to accomplish this was 200+ lines. The previous developer had trouble with the PositionEnum, as the page was supposed to display the CastMembers in a different order than the Enum. I fixed this by writing one loop that used a list of integers to dictate which Enum was next in line. This was done using C# Razor.

**Add RentalRequest Properties:**

RentalRequest.cs:

```csharp
 [Required(ErrorMessage = "Please provide a valid phone number")]
        [Display(Name = "Contact Phone Number")]
        [DataType(DataType.PhoneNumber)]
        [RegularExpression(@"^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$", ErrorMessage = "Not a valid phone number")]
        public string ContactPhoneNumber { get; set; }

        [Display(Name = "Contact Email address")]
        [Required(ErrorMessage = "Please provide a valid email address")]
        [EmailAddress(ErrorMessage = "Invalid Email Address")]
        public string ContactEmail { get; set; }
```

RentalRequestController.cs:

```csharp


 [HttpPost]
        [ValidateAntiForgeryToken]
        [Authorize(Roles = "Admin")]
        public ActionResult Edit([Bind(Include = "RentalRequestId,ContactPerson,ContactPhoneNumber,ContactEmail,Company,StartTime,EndTime,ProjectInfo,Requests,Accepted,ContractSigned")] RentalRequest rentalRequest)
        {
            long sec = 10000000;        //DateTime ticks per second
            long hrSecs = 3600;         //Seconds in an hour
            long Strtm = Convert.ToInt64(rentalRequest.StartTime.Ticks) / sec;    //divided ticks into seconds
            long Endtm = Convert.ToInt64(rentalRequest.EndTime.Ticks) / sec;
            if (Endtm < (Strtm + hrSecs) && Endtm >= Strtm)    //Doesn't allow rentals < 1hr
            {
                ModelState.AddModelError("EndTime", "** Rental must be at least 1 hour.  **");
            }
            if (Endtm < Strtm)   //Keeps End time after start time
            {
                ModelState.AddModelError("StartTime", "** Start Time cannot occur after End Time.  **");
            }
            if (ModelState.IsValid)
            {

                var currentRentalRequest = db.RentalRequests.Find(rentalRequest.RentalRequestId);

                currentRentalRequest.ContactPerson = rentalRequest.ContactPerson;
                currentRentalRequest.ContactPhoneNumber = rentalRequest.ContactPhoneNumber;
                currentRentalRequest.ContactEmail = rentalRequest.ContactEmail;
                currentRentalRequest.Company = rentalRequest.Company;
                currentRentalRequest.StartTime = rentalRequest.StartTime;
                currentRentalRequest.EndTime = rentalRequest.EndTime;
                currentRentalRequest.ProjectInfo = rentalRequest.ProjectInfo;
                currentRentalRequest.Requests = rentalRequest.Requests;
                currentRentalRequest.Accepted = rentalRequest.Accepted;
                currentRentalRequest.ContractSigned = rentalRequest.ContractSigned;

                db.Entry(currentRentalRequest).State = EntityState.Modified;
                db.SaveChanges();

                if (rentalRequest.Accepted == true)
                {
                    RentalEditCalendar(rentalRequest);
                }
                else if (rentalRequest.Accepted == false)
                {
                    RentalDeleteCalendar(rentalRequest);
                }
                return RedirectToAction("Index");
            }
            return View(rentalRequest);
        }
```
This was a relatively simple story that required me to add properties to RentalRequest model, update database, then add appropriate Razor to the Create and Edit cshtml pages.

**Award Preview:**

This story required me to create a preview of what the award would look like as they were typing it. I accomplished this using jQuery and html. 

**Dynamically Load Cast Members:**

Before starting this story, the cast members were hard-coded into the About page of the website. I changed this to load dynamically. This was done using C#. 

**Enable Upload - Prevent Duplication for photos:**

PhotoController.cs:


```csharp
 [AllowAnonymous]
        public static byte[] ImageBytes(HttpPostedFileBase file)
        {
            //Convert the file into a System.Drawing.Image type
            Image image = Image.FromStream(file.InputStream, true, true);
            //Convert that image into a Byte Array to facilitate storing the image in a database
            var converter = new ImageConverter();
            byte[] imageBytes = (byte[])converter.ConvertTo(image, typeof(byte[]));
            //return Byte Array
            return imageBytes;
        }

 [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "PhotoId,PhotoFile,OriginalHeight,OriginalWidth,Title")] Photo photo, HttpPostedFileBase file)
        {
            byte[] photoArray = ImageBytes(file);
            if (db.Photo.Where(x => x.PhotoFile == photoArray).ToList().Any())
            {
                var id = db.Photo.Where(x => x.PhotoFile == photoArray).ToList().FirstOrDefault().PhotoId;
                ModelState.AddModelError("PhotoFile", "This photo already exists in the database. Would you like to <a href='/Photo/Details/" + id + "'>view</a> or <a href='/Photo/Edit/" + id + "'>edit</a> the photo?");
            }
            if (ModelState.IsValid)
            {
                photo.PhotoFile = photoArray;
                Bitmap img = new Bitmap(file.InputStream);
                photo.OriginalHeight = img.Height;
                photo.OriginalWidth = img.Width;

                db.Photo.Add(photo);
                db.SaveChanges();

                return RedirectToAction("Index");
            }

            return View(photo);
        }

[HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Edit([Bind(Include = "PhotoId,PhotoFile,OriginalHeight,OriginalWidth,Title")] Photo photo, HttpPostedFileBase file)
        {
            var currentphoto = db.Photo.Find(photo.PhotoId);
            byte[] photoArray = ImageBytes(file);
            if (db.Photo.Where(x => x.PhotoFile == photoArray).ToList().Any())
            {
                var id = db.Photo.Where(x => x.PhotoFile == photoArray).ToList().FirstOrDefault().PhotoId;
                ModelState.AddModelError("PhotoFile", "This photo already exists in the database. Would you like to <a href='/Photo/Details/" + id + "'>view</a> the photo?");
            }
            if (ModelState.IsValid)
            {
                if (file != null)
                {
                    
                    currentphoto.PhotoFile = photoArray;
                    Bitmap img = new Bitmap(file.InputStream);
                    currentphoto.OriginalHeight = img.Height;
                    currentphoto.OriginalWidth = img.Width;
                    currentphoto.Title = photo.Title;
                    db.Entry(currentphoto).State = EntityState.Modified;
                    db.SaveChanges();
                    return RedirectToAction("Index");
                }
                else
                {
                    currentphoto.Title = photo.Title;
                    db.Entry(currentphoto).State = EntityState.Modified;
                    db.SaveChanges();
                    return RedirectToAction("Index");
                }
            
            }

            return View(photo);
        }
```


For these two stories I had to enable the user to upload a new photo on the Edit page (this had been disabled). I then had to prevent the user from uploading (from either the create or edit page) and image that already existed in the database and if the image already did exist, throw a modelstate error. I did this using C#. 

Overall this project was a great experience and I had a lot of fun working with a team. We had our daily scrum every morning and would coordinate on the actions of the upcoming day. I definitely learned a lot and am looking forward to working on more projects. 
