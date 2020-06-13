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
    console.log("checker is " + userInGroups);
    //alert(allGroupsArray);
    console.log(checkUserInGroupsArray);
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

function checkUserInGroups(groupnames, condition) {
  getCurrentUserSPMembershipGroups(groupnames, condition, function (
    isCurrentUserInGroups
  ) {
    //console.log(isCurrentUserInGroups);
    if (isCurrentUserInGroups) {
      console.log("Current user is present in group(s): " + groupnames);
    } else {
      console.log("Current user is not present in group(s): " + groupnames);
    }
  });
}

$(document).ready(function () {
  /* using SharePoint SP.SOD to make sure SP.ClientContext is loaded before calling the checkUserInGroups(groupnames, condition) function */
  SP.SOD.executeFunc("sp.js", "SP.ClientContext", function () {
    console.log("Initiating SP.ClientContext");
  });
  /* 
	 provide one group or multiple groups separated by semi-colon (;) and "AND" or "OR" condition as shown below
	 for example:- checkUserInGroups("Site Members", "AND") or 
	 checkUserInGroups("Site Members;Site Owners", "AND") or
	 checkUserInGroups("Site Visitors;Site Owners", "OR")
	 */
  SP.SOD.executeOrDelayUntilScriptLoaded(
    checkUserInGroups("Site Members", "AND"),
    "sp.js"
  );
});
```
