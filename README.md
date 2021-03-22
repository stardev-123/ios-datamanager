iOSDataManager
==============

An iOS helper library to make using CoreData, NSUserDefaults, Document loading/saving a little easier.

DataManager's main purpose is to make interacting with CoreData a little easier. It has several helper methods for doing mundane tasks, like querying and updating. It also includes helpers for performing other mundane data tasks, including interacting with NSUserDefaults, loading/saving images, and loading views from nib.
What it DOESN'T do is replace the need for having your AppDelegate setup with a CoreData stack.


Usage
-----

DataManager is meant to exist as a singleton, accessed through [DataManager SharedDataManager].
In your AppDelegate, in application:didFinishLaunchingWithOptions, add the line:

    [DataManager SharedDataManager].managedObjectContext = [self managedObjectContext];

and that is all you need to do to get DataManager setup.


Assumptions made for the purpose of this ReadMe:
Let's assume you've got a CoreData model created with an Entity called Person.
That Person has the following attributes:
Name
Age
Height
Gender

Let's also assume you've got a reference to the DataManager signleton called dm 
    DataManager * dm = [DataManager SharedDataManager];

CoreData Methods
----------------

DataManager has a handful of simple methods that directly return the results of a fetchedResultController.
Instead of creating a fetchedResultsController, telling to to fetch, and then getting the fetchedObjects, you can call:

    NSArray * allHumans = [dm getResultsWithEntity:@"Person" sortDescriptor:@"Age" batchSize:20];


Or, if you need to perform a predicate, there's one for that too:

    NSArray * women = [dm getResultsWithEntity:@"Person" sortDescriptor:@"Age" sortPredicate:[NSPredicate predicateWithFormat:@"gender != \"men\""] batchSize:20];


DataManager also has methods to return a fetchedResultsController:

    NSFetchedResultsController * controller = [dm fetchedResultsControllerWithEntity:@"Person" sortDescriptor:@"Height" batchSize:20];

To create a new Managed object, just call:
	MyManagedObject * object = [dm newObjectForEntityForName:@"MyManagedObject"];


NSUserDefaults
--------------

DataManager has two simple wrappers that handle updating the standardUserDefaults.

    [dm setDefaultsUserObject:[some object] forKey:@"anyKey"];
    SomeObject * ThatObjectYouJustSaved = [dm defaulUserObjectForKey:@"anyKey"];


Image saving/loading/caching
----------------------------
DataManager has methods for doing basic things like saving/loading/deleting an image to the NSDocumentDirectory.

    [dm saveImageToDevice:pictureOfACat withName:@"aCat" extension:@"jpg"];  //jpg and png supported
    [dm loadImageNamed:@"aCat.jpg"];
    [dm removeFile:@"aCat.jpg"];

And, if the content being saved isn't prompted by user input, you'll need to call:
    [dm addSkipBackupAttributeToFile:@"mycat.jpg"];
To flag the file to not be backed up by iCloud.

DataManager also incorporates its own image cache. Using [UIImage imageNamed:] method caches the image, but you have no control over when and how an image is ever uncached, which can lead to memory issues. With DataManager, it will automatically cache any image loaded through loadImageNamed:, which saves time when loading the same image for a second time. Whenever you want to clear the cache, you can call:

    [dm clearImageCache];

Or to remove a specific image:
    [dm removeImageFromImageCache:someUIImage];
    
Or
    [dm removeImageFromImageCacheNamed:@"imageName.jpg"];

DataManager also has methods for finding if a file exists:
	[dm doesFileExist:@"myCatsBirthdays.xls"];
    
And a special method for finding images (including testing for @2x retina images)
	[dm doesImageExist:@"catsPlayingPoker.png"];
    

Other Helpers
-------------

The rest of the helpers are just simple things that we got tired of having to remember how to do.

You can load a UIView from a nib:

    SomeUIViewClass * view = [dm loadViewFromNib:@"SomeUIViewClassNib_iPhone" andOwner:self];

You can also get a nib Name suffixed with the appropriate device name (_iPhone, _iPhone5, or _iPad)

	SomeUIViewClass * view = [dm loadViewFromNib:[dm nibNameWithDeviceSuffix:@"SomeUIViewClassNib"] andOwner:self];
    
    
Constants
---------

DataManager has several preprocessor constants defined:

    #define IS_IPHONE 			( [[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPhone )
    #define IS_IPHONE_RETINA 	( [[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPhone && [[UIScreen mainScreen] scale] == 2.00 )
    #define IS_IPHONE_5 		( fabs( ( double )[ [ UIScreen mainScreen ] bounds ].size.height - ( double )568 ) < DBL_EPSILON )

    #define IS_IPAD				( [[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPad )
    #define IS_IPAD_RETINA 		( [[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPad && [[UIScreen mainScreen] scale] == 2.00 )