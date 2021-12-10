---
date: 2017-12-11
draft: false
slug: null-LASTEXITCODE
title: '$null and $LASTEXITCODE'
---

> I wrote this a few years ago. It is interesting to look back with a fresh perspective. Also, the modern data systems now include all this functionality by default. No need to script it manually. :shrug:

## The Background

User onboarding and account creation is time consuming and error prone. I try to avoid online admin portals as much as possible as they tend to be slow as molasses and bury useful settings under many levels of menus. For the administration of our Microsoft products, just about all tasks have the appropriate PowerShell cmdlets available by default. For our G Suite products, I have relied heavily on [Google Apps Management (GAM)](https://github.com/jay0lee/GAM), a cross platform command line utility for managing G Suite via API.

To further streamline the process, user account creation is automated with PowerShell working from automatic .csv extracts from our local data system. My original G Suite user provisioning script was working fine but took ages to complete. It was extremely crude — it would try to create each user whether or not they already existed. This required GAM to try the account creation, fail, and then report the error before continuing to the next record. Just about what you'd expect a total rookie to come up with.

```powershell
foreach ($entry in $Data) {
    /gam.exe create user $($entry.Email) firstname $($entry.First) lastname $($entry.Last)
}
```


>Note: snippets have been sanitized and simplified to only show the relevant lines. There are additional tests and loops in the finished script to place users in specific OUs and to tag them with titles and other relevant information.


## Slight Improvements

I decided that the script should check whether an account exists before attempting to create it. Letting GAM try and fail to create the account took 2-5 seconds for each record. Simply querying an account took 1-2 seconds. But how to test this? 

If an account was not already present, GAM would return ```ERROR: 404: Resource Not Found: userKey - userNotFound``` to the console. At first, I added an ```if``` loop to test for that string.

```powershell
foreach ($entry in $Data) {
    $status = .\gam.exe info user $($entry.Email)

    if ($status -eq "ERROR: 404: Resource Not Found: userKey - userNotFound" ) {
         /gam.exe create user $($entry.Email) firstname $($entry.First) lastname $($entry.Last)
    }
    else {
        Write-host "User already exists, skipping..."
    }
}
```

I soon realized that the ```if``` condition would never be met. The console output was not being captured as a string. This may be obvious to some but it wasn't to me at the time.

## Logging and Transcripts

To help troubleshoot the process, I started logging a transcription of each run to file. I also decided to ```write-host``` key info at certain points in the script to see where things were going right or wrong.

```powershell
Start-Transcript "$log_path\GoogleSync-Transcript-$(get-date -f yyyy-MM-dd).txt" -Append -IncludeInvocationHeader
```

Any transcripts older than 30 days are deleted.

```powershell
$days_back = "-30"
$current_date = Get-Date
$date_to_delete = $current_date.AddDays($days_back)

Get-ChildItem $log_path | Where-Object { $_.LastWriteTime -lt $date_to_delete } | Remove-Item
```


## The Breakthrough

After reviewing some forum posts it became clear that GAM, as a native Windows application, would of course output an exit code upon completion. As I am only interested in accounts that do not exist, I can test for an exit code of ```404```. But how? Further reasearch led me to the ```$LASTEXITCODE``` variable. The solution is to query the API with GAM and, as we're only interested in the exit code, throw away the returned data into the ```$null``` variable. From there, we can test against a ```404``` in our ```if``` loop and only create accounts that are missing.

```powershell
foreach ($entry in $Data) {
    $null = .\gam.exe info user $($entry.Email)

    if ($LASTEXITCODE -eq "404" ) {
            .\gam.exe create user $($entry.Email) firstname $($entry.First) lastname $($entry.Last)
    }      
    else {
        Write-host "User $($entry.Email) already exists. Skipping..."
    }
```

## Next Steps

Ideally, this script will get modified to query user data from our local database directly rather than relying on .csv extracts. Alternatively, or maybe additionally, it should be trivial to take the difference between two extracts and only process records that have been changed. This should drastically cut down on the script's run time.