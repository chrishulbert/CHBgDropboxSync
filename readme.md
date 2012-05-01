CHBgDropboxSync
===============

Obj-C / iPhone / iOS library for background syncing the contents of a folder to Dropbox, just like the 'magical' way that the desktop version automatically keeps everything in sync, by Chris Hulbert - chris.hulbert@gmail.com

* https://github.com/chrishulbert/CHBgDropboxSync
* Extracted from my skeleton key app: http://www.skeletonkeyapp.com/
* You're 100% free to use this: MIT license. Thoroughly tested by me, but no guarantees are made - so test it thoroughly for yourself before blindly trusting it!
* This code all uses ARC for memory management.
* This was written to go with the Dropbox SDK 1.1, however i'd recommend using whatever is latest here: https://www.dropbox.com/developers/reference/sdk
* You'll need the ConciseKit headers: https://github.com/petejkim/ConciseKit
* I'm happy to help if you email with any questions!
* You really should be familiar with the Dropbox SDK before using this...

CHBgDropboxSync
---------------

Performs a 2-way sync process designed to either download or upload changes as necessary. Uses the NSUserDefaults to keep track of local files to help it decide whether to delete or transfer if a file is present one one side but not on the other.

To use this, add the Dropbox API to your app as you normally would, creating an interface to link (and unlink) as normal. Use this class to sync a local folder (typically your Documents folder in an iOS app) to your app's folder on Dropbox in the background without user intervention.

When dropbox linking is complete, you should call the 'clearLastSyncData' function before starting a sync, like so in your app delegate:

	- (BOOL)application:(UIApplication *)application
		  handleOpenURL:(NSURL *)url {
	    if ([[DBSession sharedSession] handleOpenURL:url]) {
	        if ([[DBSession sharedSession] isLinked]) {
	            NSLog(@"App linked successfully!");
	            // At this point you can start making Dropbox API calls
	            
	            [CHBgDropboxSync clearLastSyncData];
	            [CHBgDropboxSync start];
	        }
	        return YES;
	    }
	    // Add whatever other url handling code your app requires here
	    return NO;
	}

Whenever your app becomes active, you should perform another sync, in case there is new data on Dropbox. Again, in your app delegate:

	// Gets called on app startup and returning to the app
	- (void)applicationDidBecomeActive:(UIApplication *)application {
	    [CHBgDropboxSync start];   // Start the sync
	}
	
If, in your particular app, you need to forcibly clear the remote dropbox data (i had to in the skeleton key app if you changed the master password therefore need to re-encrypt *every* file), you can use code like below. It stops any running sync, clears the remote dropbox and the sync state, performs local changes, and then re-syncs:

    [CHBgDropboxSync forceStopIfRunning]; // Stop syncing

    // Show a 'clearing' display
    UIAlertView* clearing = [[UIAlertView alloc] initWithTitle:nil
     message:@"Clearing Dropbox"
     delegate:nil
     cancelButtonTitle:nil
     otherButtonTitles:nil]; 
    [clearing show];

    [CHBgDropboxSync clearLastSyncData]; // Clear last sync data
    // *first* so if anything borks, nothing will be deleted next sync

    // Erase everything on dropbox
    [DropboxClearer doClear:^(BOOL success) { 

      // Hide the 'clearing' alert
      [clearing dismissWithClickedButtonIndex:0 animated:YES];
         
      if (success) {
          // ..Perform your operations on local data..
          [CHBgDropboxSync start]; // Start sync again
      } else {
             [[[UIAlertView alloc] initWithTitle:@"Error"
              message:@"Couldn't clear dropbox"
              delegate:nil
              cancelButtonTitle:@"Ok"
              otherButtonTitles:nil] show];
      }
      
    }];

Any time you've saved a local file and it would make sense to re-sync immediately, do this:

    [CHBgDropboxSync start];
    
If your user unlinks the app from their dropbox account, stop the sync and clear its sync state:
   
    [CHBgDropboxSync forceStopIfRunning];
    [CHBgDropboxSync clearLastSyncData];
    [[DBSession sharedSession] unlinkAll];

If you want to make your app listen for any local changes that the sync makes, such as downloading a new/updated file or deleting a file, listen for the "CHBgDropboxSyncUpdated" notification.

DropboxClearer
--------------

This helper class isn't really needed, only to be used if your app (like mine) needs the ability to clear out the remote dropbox. If you don't need that feature you can leave this out.