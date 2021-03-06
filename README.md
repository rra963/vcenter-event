# vCenter Server Events
List of vCenter Events generated by the following script. 
Script written by William Lam, please see his blog post https://www.virtuallyghetto.com/2019/12/listing-all-events-for-vcenter-server.html


    $user= [user_name]
    $passwd = [user_password]

    Connect-VIServer -Server [vCenter_IP_or_FQDN] -User $User -Password $passwd

    $vcenterVersion = ($global:DefaultVIServer.ExtensionData.Content.About.ApiVersion)

    $evenMgr = Get-View $global:DefaultVIServer.ExtensionData.Content.EventManager

    $results = @()
    foreach ($event in $evenMgr.Description.EventInfo) {

        if($event.key -eq "EventEx" -or $event.key -eq "ExtendedEvent") {
            $eventId = ($event.FullFormat.toString()) -replace "\|.*",""
            $eventType = $event.key
        } else {
            $eventId = $event.key
            $eventType = "Standard"
        }
        $eventDescription = $event.Description

             $tmp = [PSCustomObject] @{
            EventId = $eventId;
            EventType = $eventType;
            EventDescription = $($eventDescription.Replace("<","").Replace(">",""));
        }

            $results += $tmp

    }
    Write-Host $vcenterVersion
    Write-Host "Number of Events: $($results.count)"
    $results | Sort-Object -Property EventId | ConvertTo-Csv | Out-File -FilePath "[your_path]\vcenter-$vcenterVersion.csv"

    Disconnect-VIServer -Confirm:$false
