# SharePoint-JavaScript-To-Check-If-Current-User-Exists-In-A-SharePoint-Group
```javascript
function getCurrentUserSPMembershipGroups(groupnames, condition, callback) {
  var checkUserInGroupsString = groupnames;
  var checkUserInGroupsArray = checkUserInGroupsString.split(";");
  var conditionToCheck = condition.toUpperCase();
  var clientContext = new SP.ClientContext.get_current();
  var currentUser = clientContext.get_web().get_currentUser();
  clientContext.load(currentUser);
  var userGroups = currentUser.get_groups();
  clientContext.load(userGroups);
  clientContext.executeQueryAsync(
    OngetCurrentUserSPMembershipGroupsQuerySucceess,
    OngetCurrentUserSPMembershipGroupsQueryFailure
  );
  function OngetCurrentUserSPMembershipGroupsQuerySucceess() {
    var userInGroups = false;
    var allGroupsArray = [];
    var groupsEnumerator = userGroups.getEnumerator();
    while (groupsEnumerator.moveNext()) {
      var currentGroup = groupsEnumerator.get_current().get_title();
      allGroupsArray.push(currentGroup);
    }
    if (conditionToCheck == "AND") {
      userInGroups = checkUserInGroupsArray.every((group) =>
        allGroupsArray.includes(group)
      );
    } else if (conditionToCheck == "OR") {
      userInGroups = checkUserInGroupsArray.some((group) =>
        allGroupsArray.includes(group)
      );
    } else {
      userInGroups = checkUserInGroupsArray.every((group) =>
        allGroupsArray.includes(group)
      );
    }
    //console.log("checker is " + userInGroups);
    //alert(allGroupsArray);
    //console.log(checkUserInGroupsArray);
    callback(userInGroups);
    //alert(userInGroups);
  }
  function OngetCurrentUserSPMembershipGroupsQueryFailure(sender, args) {
    alert(
      "Request failed. " + args.get_message() + "\n" + args.get_stackTrace()
    );
    callback(false);
  }
}

var checkUserInGroups = function() {
/* 
enter a SharePoint group name or multiple groups separated by a semi-colon(;) and "AND" or "OR" condition
	 for example:- 
	 getCurrentUserSPMembershipGroups("Site Members", "AND") 
	 getCurrentUserSPMembershipGroups("Site Members;Site Owners", "AND")
	 getCurrentUserSPMembershipGroups("Site Visitors;Site Owners", "OR")
*/
  getCurrentUserSPMembershipGroups("SharePointGroupName(s)", "OR", function(isCurrentUserInGroups){
    //console.log(isCurrentUserInGroups);
    if (isCurrentUserInGroups) {
      console.log("Current user is present in group(s): " + groupnames);
    } else {
      console.log("Current user is not present in group(s): " + groupnames);
    }
  });
}

$(document).ready(function() {
  /* using SharePoint SP.SOD to make sure SP.ClientContext is loaded before calling the checkUserInGroups(groupnames, condition) function */
  SP.SOD.executeFunc("sp.js", "SP.ClientContext", function() {
    //console.log("Initiating SP.ClientContext");
     SP.SOD.executeOrDelayUntilScriptLoaded(checkUserInGroups, "sp.js");
  });
});
```
