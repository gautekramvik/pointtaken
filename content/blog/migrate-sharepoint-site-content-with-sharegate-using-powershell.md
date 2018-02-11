---
title: "Migrate Sharepoint Site Content With Sharegate Using Powershell"
date: 2017-06-06T21:18:08+01:00
draft: false
banner: "img/code.png"
author: "André Andersen"
categories: ["dev"]
tags: ["promo"]
---

# Migrate SharePoint site content with Sharegate using powershell

Here’s an example of how to use PowerShell to migrate content from SharePoint On-premises to Office365 or SharePoint 2016. I use PowerShell to export the On-premises lists to a csv-file, add parameters to the copy-commandlet to adjust the migration type (e.g. only changed files since last migration), also Scheduling tasks to run off peak hours.

Read on :-)
<br>
#### Step 1 - Get lists!
First things first, let’s get a csv file to work with. You can use the following example with PowerShell ISE:


        #Export lists to CSV 
        Import-Module Sharegate 

        #Site to get the lists from, add username and password if it's different from the windows credentials
        $username = "username" 
        $password = ConvertTo-SecureString "password" -AsPlainText -Force 
        $site = Connect-Site -Url "http://sp.example.com/example/" UserName $username -Password $password 

        #Path to export the CSV to, will create the folder if it doesn’t exist 
        $csvExportPath = "C:\Sharegate\Reports\csv\" 
        if(!(Test-Path -Path $csvExportPath -PathType Container))
        { 
        New-Item -ItemType directory -Path $csvExportPath 
        } 
        #Get lists from the specified site and export it to CSV 
        $lists = Get-List -Site $site | Export-Csv -Path "$($csvExportPath)Example.csv" -Encoding UTF8 -Delimiter ","

<br>

Supply the username and password for the site you want to connect to. You can remove the password parameter if you want a password prompt every time you run the script. 

It will create a folder if needed to export the csv-file to. Change the value to where you want the csv-file. 

The last part will get all the lists in the specified site and export the list to a csv-file. You can limit the lists you get by adding the -Name parameter. This parameter is wildcard supported. 
An example: $lists = Get-List -Site $site -Name Example,A* 

This will only get the list called Example and every list starting with “A”.
For detailed information about the Get-List commandlet, visit https://support.share-gate.com/hc/en-us/articles/115000597387-Get-List
<br>
#### Step 2 - Copy Lists!
Now that we have a csv-file with the lists we want to copy, we can use it to copy content. Here’s an example:


        #Copy lists from a csv-file 
        Import-Module Sharegate 
        #Specify the csv containing the lists you want to copy, example "C:\example.csv" 
        $csv = "C:\example.csv" 
        $lists = Import-Csv -Path $csv -Encoding UTF8 | select -ExpandProperty title 

        #The source of the copy, add username and password if it's different from the windows credentials 
        $sourceUrl = "http://sp.example.com/example/" 
        $userNameSource = "username" 
        $passwordSource = ConvertTo-SecureString "password" -AsPlainText -Force 
        #Connecting to the source 
        $sourceSite = Connect-Site -Url $sourceUrl -UserName $userName -Password $password 

        #The destination of the copy, add username and password if it's different from the windows credentials
        $destinationUrl = "https://example.sharepoint.com/example/" 
        $userNameDestination = "username" 
        $passwordDestination = ConvertTo-SecureString "password" -AsPlainText -Force 
        #Connecting to the destination 
        $destinationSite = Connect-Site -Url $destinationUrl -UserName $userName -Password $password 

        #Path used for reports. Will create the folder if it does not exist 
        $reportPath = "C:\Sharegate\Reports\" 
        if(!(Test-Path -Path $reportPath -PathType Container)) 
        { 
            New-Item -ItemType directory -Path $reportPath 
        } 

        #Incremental update setting, add -CopySettings $copySettings to the copy commandlet if you want to use incremental update 
        $copySettings = New-CopySettings -OnContentItemExists IncrementalUpdate 

        #Copy every list in the csv and export the reports 
        $counter = 0 
        foreach ($list in $lists)
        { 
            $counter++ 
            Write-Progress -Activity 'Copying lists' -CurrentOperation $list -PercentComplete 
        (($counter / $lists.count) * 100) 
            $result = Copy-List -SourceSite $sourceSite -Name $list -DestinationSite $destinationSite 
            $result Export-Report $result -Path $reportPath -DefaultColumns 
        }

First you need to specify the csv-file you want to copy from. Then you need to specify the source and destination sites, along with the correct usernames and passwords.

Change the report path if you want to export the reports to a different path.

The foreach loop will go through every list in the csv-file and copy the lists to the destination site. You can add different parameters to the Copy-List commandlet to adjust the copy behavior. Incremental update for example you would add -CopySettings $copySettings

For detailed information about the Copy-List commandlet, visit https://support.share-gate.com/hc/en-us/articles/115000597727-Copy-List

For the full Sharegate PowerShell documentation, visit https://support.share-gate.com/hc/en-us/categories/204661007-PowerShell
<br>
#### Scheduled task
Here’s a tip: Use Task Scheduler to schedule a migration. This can be helpful to plan a migration during off peak hours without having to start the script manually.

<img class="img-fluid mt-4 mb-4" src="/img/scheduledtask.png" /> 

Happy Migration! :-)
<br>