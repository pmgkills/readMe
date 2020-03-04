# readMe
Stepper Component Issues -
Issue 1: Stepper didn’t remember the previous selections once the user reached to details mid view - fixed
Issue 2: Stepper was not able to auto populate values in the dropdown that user selected - fixed
Issue 3: Stepper was not unable to reset the following steps’ values if one of the previous step’s value has changed - fixed
Issue 4: Stepper was not used to be disabled after user has proceeded to view-mid/edit-mid/new-mid views - fixed


Timeout issues: 

Edit session timeout issue: 
Issue: Earlier UI code used to run the timer as soon as a user selects mid from mid view list, this made the user’s edit session to be closed as soon as the timer hits 10 minutes without checking whether the user is active or inactive - fixed
Solution: Updated edit session timeout for 10 mins of idle or inactive session, and sends the map_cancel event to the service to remove the lock on the mid.
Also, added dialog box to notify user that his/her edit session time is expiring soon, and the user needs to select either of cancel or continue options on the dialog box to stay in or opt out of the edit session. \
If the user is still inactive after the 10 mins, and didn’t respond to the dialog box then the application will wait for global timer to expire.

Global Application Timeout Feature:
Added global application timeout to 20 mins - if any user’s session is idle for 20 mins his/her session will be timed out.
Before session is expired, UI will display a dialog box to notify the user that his/her session will timed out soon, and asks him to select either of the cancel or continue options on the dialog box. 
If the user still didn’t respond to the dialog box, app will be redirected to its homepage.

Edit Mid view issues:

Added capability to determine which user can edit and add new mids based on his/her role type. (users role with read/write or admin privileges will get these options).

Once the user clicks on edit, UI sends start_edit event to the backend, but if the user didn’t make any changes and hits back, earlier UI didn’t send any cancel event to unlock the mid.
Solution: Added map_cancel event that notifies the backend that no changes were made so that edit mid lock can be removed.

If user selected edit session and makes changes and the user closes the session without saving, if the same user re login then the user will get the edit capability without manually requesting for it.


Refactoring:

Updated application routing to /home and /newmids from /pages.
Removed unused left menu with dashboard, as we are not using either of them.
Removed user profile dropdown, as the application design doesn’t use or store any user information.

Created interfaces for different steps that are shown in UI such as lobs, sublobs, phases, systems, templates

Issue: Earlier most of the code was written in single ts file which didn’t meet the angular standards
Solution: Moved the service code from ts file to respective service files. As a part of the solution created following files:
utils.service.ts		—> contains commonly used services across the ui , 
loader.service.ts	—> contains loader/spinner, 
template.service.ts —> implements all the above mentioned interfaces.

Created angular services based on the components
Removed all hardcoded values, and created one common environment folder that has environment specific uri’s. 
Configured package.json to pickup the service uri’s based on the deployment environment. These are differentiated during the build command as following:
local:  ng build
prod: ng build - -c -prod
qa: ng build - -c -qa
dev: ng build - -c -dev

Removed unused packages, and libraries.

