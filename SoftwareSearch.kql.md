// Find all unlicensed IDEs on managed devices as an example
let softwareVendors = pack_array("Sublimetext", "Spyder", "Rstudio", "Jetbrains", "Eclipse");
let softwareNames = pack_array("Atom", "anaconda3"); //added anaconda
DeviceTvmSoftwareInventory
| where SoftwareVendor has_any (softwareVendors) or SoftwareName has_any(softwareNames)
| distinct DeviceName, SoftwareVendor, SoftwareName, SoftwareVersion
| join kind=leftouter (
    DeviceInfo
    | where LoggedOnUsers != "[]" // get rid empty json objects
    | summarize arg_max(Timestamp, *) by DeviceName //groups by DeviceName, for each it selects the row
    | project DeviceName, LoggedOnUsers
) on DeviceName
| extend CommandDynamic = parse_json(LoggedOnUsers)[0]
| project DeviceName, SoftwareVendor, SoftwareName, LoggedOnUser = CommandDynamic.UserName
| extend LoggedOnUser = tostring(LoggedOnUser)
| join kind=leftouter (
    IdentityInfo
    | distinct AccountName, AccountDisplayName)
on $left.LoggedOnUser == $right.AccountName
| project-away LoggedOnUser, AccountName
