---
layout: single
title: PowerShell - Add AD Site Subnet
excerpt: 
permalink: /2013/11/powershell-add-ad-site-subnet.html
tags: 
- active directory
- adsi
- import-csv
- missing subnet
- powershell
- powershell 4.0
- subnet
published: true
comments: true
---
<a href="{{ site.url }}/images/2013/20131110_PowerShell_-_Add_AD_Site_Subnet/Add_subnet2__1746226446__-142x124.png" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><img border="0" src="{{ site.url }}/images/2013/20131110_PowerShell_-_Add_AD_Site_Subnet/Add_subnet2__1291922575__-142x124.png" /></a>Last month I posted <a href="{{ site.url }}/2013/10/powershell-report-ad-missing-subnets.html" target="_blank">a script that report the Missing Subnets</a> from the Active Directory. The script goes on each Domain Controllers and get the last x entries from the NETLOGON.log file.
Once this report is generated, you might want to check with your Telecom guy/team to get the correct network mask, correct site of each entries and fix this situation.

<u>Reminder:</u> Subnet objects (class subnet) define network subnets in Active Directory. A network subnet is a segment of a TCP/IP network to which a set of logical IP addresses is assigned. Subnets group computers in a way that identifies their physical proximity on the network. Subnet objects in Active Directory are used to map computers to sites.

Today I will show how to add those missing subnets in your Active Directory using PowerShell on Windows Server 2012 and Previous versions via ADSI.



# Using ADSI

<i style="background-color: yellow;">Applies To: Windows XP or higher, Windows PowerShell 2.0 or higher, Windows Server 2003 or higher.</i>

In some of my previous posts, I learned a lot about ADSI and I thought I could do the same for this one and find a way to create my subnets with ADSI.

I came up with the following function which accept 4 parameters: <b>Subnet</b>, <b>SiteName</b>, <b>Description</b> and <b>Location</b>.

<a href="http://gallery.technet.microsoft.com/Add-ADSISubnet-ADSI-d3f86e90" target="_blank">Download from TechNet Gallery</a>
<a href="https://github.com/lazywinadmin/PowerShell/tree/master/AD-SITE-Add-ADSubnet(ADSI)" target="_blank">Download from Github (CSV and PS1)</a>


<pre style="border-style: solid; border-width: 1px; font-size: 13px;">PROCESS{
        TRY{
            $ErrorActionPreference = 'Stop'
            
            # Distinguished Name of the Configuration Partition
            $Configuration = ([ADSI]"LDAP://RootDSE").configurationNamingContext

            # Get the Subnet Container
            $SubnetsContainer = [ADSI]"LDAP://CN=Subnets,CN=Sites,$Configuration"
            
            # Create the Subnet object
            Write-Verbose -Message "$subnet - Creating the subnet object..."
            $SubnetObject = $SubnetsContainer.Create('subnet', "cn=$Subnet")
        
            # Assign the subnet to a site
            $SubnetObject.put("siteObject","cn=$SiteName,CN=Sites,$Configuration")

            # Adding the Description information if specified by the user
            IF ($PSBoundParameters['Description']){
                $SubnetObject.Put("description",$Description)
            }
            
            # Adding the Location information if specified by the user
            IF ($PSBoundParameters['Location']){
                $SubnetObject.Put("location",$Location)
            }
            $SubnetObject.setinfo()
            Write-Verbose -Message "$subnet - Subnet added."
        }#TRY
        CATCH{
            Write-Warning -Message "An error happened while creating the subnet: $subnet"
            $error[0].Exception
        }#CATCH
}#PROCESS Block
```

<a href="http://gallery.technet.microsoft.com/Add-ADSISubnet-ADSI-d3f86e90" target="_blank">Download on TechNet Gallery</a>
<a href="https://github.com/lazywinadmin/PowerShell/tree/master/AD-SITE-Add-ADSubnet(ADSI)" target="_blank">Download from Github (CSV and PS1)</a>

<h4>Adding one subnet


```
PS C:\LazyWinAdmin> Add-ADSubnet -Subnet '192.168.10.0/24' -SiteName 'MTL1' -Verbose
```

```
VERBOSE: 192.168.10.0/24 - Creating the subnet object...
VERBOSE: 192.168.10.0/24 - Subnet added.
VERBOSE: Script Completed
```
<h4>

<h4>Adding a bunch of subnets from a CSV file


We have the following CSV file with a few subnets to add, we can use Import-CSV to create all the subnet at once.
<a href="http://3.bp.blogspot.com/-_kFn8eojX_I/UoAagaVAEaI/AAAAAAABehI/qp4K8mFxric/s1600/2013-11-10+6-35-03+PM.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="https://3.bp.blogspot.com/-_kFn8eojX_I/UoAagaVAEaI/AAAAAAABehI/qp4K8mFxric/s1600/2013-11-10+6-35-03+PM.png" /></a>

```
PS C:\LazyWinAdmin> Import-csv .\subnets.csv | Add-ADSubnet -Verbose
```

```
VERBOSE: 192.168.1.0/24 - Creating the subnet object...
VERBOSE: 192.168.1.0/24 - Subnet added.
VERBOSE: 192.168.2.0/24 - Creating the subnet object...
VERBOSE: 192.168.2.0/24 - Subnet added.
VERBOSE: 192.168.3.0/24 - Creating the subnet object...
VERBOSE: 192.168.3.0/24 - Subnet added.
VERBOSE: 192.168.4.0/24 - Creating the subnet object...
VERBOSE: 192.168.4.0/24 - Subnet added.
VERBOSE: 192.168.5.0/24 - Creating the subnet object...
VERBOSE: 192.168.5.0/24 - Subnet added.
VERBOSE: 192.168.6.0/24 - Creating the subnet object...
VERBOSE: 192.168.6.0/24 - Subnet added.
VERBOSE: 192.168.7.0/24 - Creating the subnet object...
VERBOSE: 192.168.7.0/24 - Subnet added.
VERBOSE: 192.168.8.0/24 - Creating the subnet object...
VERBOSE: 192.168.8.0/24 - Subnet added.
VERBOSE: Script Completed

```

The parameters of my function<b>Name</b>,<b>Location</b>,<b>Site</b>and<b>Description</b>will match theproperties in the CSV file so the cmdlet will be able to interpret them. This is possible thanks to the parameter<span style="background-color: white;"><b><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">ValueFromPipelineByPropertyName</b>

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;"><tbody><tr><td style="text-align: center;"><a href="http://4.bp.blogspot.com/-RZj1W1Cs-Y4/UoBpsAVCSUI/AAAAAAABeiI/zNSzXa3tNP0/s1600/2013-11-11+12-22-31+AM.png" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" src="https://4.bp.blogspot.com/-RZj1W1Cs-Y4/UoBpsAVCSUI/AAAAAAABeiI/zNSzXa3tNP0/s1600/2013-11-11+12-22-31+AM.png" /></a></td></tr><tr><td class="tr-caption" style="text-align: center;">Management Console: Active Directory Sites and Services. We can see the
subnets created.</td></tr></tbody></table><h4>Download the function

<a href="http://gallery.technet.microsoft.com/Add-ADSISubnet-ADSI-d3f86e90" target="_blank">TechNet Gallery</a>
<a href="https://github.com/lazywinadmin/PowerShell/tree/master/AD-SITE-Add-ADSubnet(ADSI)" target="_blank">Github (CSV and PS1)</a>




# Using the new cmdlets in ActiveDirectory module

<i style="background-color: yellow;">Applies To: Windows 8.1, Windows PowerShell 4.0, Windows Server 2012 R2</i><i>
</i><h4>Finding related cmdlets


```
PS C:\LazyWinAdmin> get-command *subnet*
```

```
CommandType     Name                                               ModuleName
-----------     ----                                               ----------
Cmdlet          Get-ADReplicationSubnet                            ActiveDirectory
Cmdlet          New-ADReplicationSubnet                            ActiveDirectory
Cmdlet          Remove-ADReplicationSubnet                         ActiveDirectory
Cmdlet          Set-ADReplicationSubnet                            ActiveDirectory
```

<h4>Get the current subnets


```
PS C:\LazyWinAdmin> Get-ADReplicationSubnet -Filter *
```

```
DistinguishedName : CN=10.1.0.0/22,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB
Location          : Montreal, Canada
Name              : 10.1.0.0/22
ObjectClass       : subnet
ObjectGUID        : 98683337-da77-412e-ae57-9fc0dbb209ba
Site              : CN=FX3,CN=Sites,CN=Configuration,DC=FX,DC=LAB

DistinguishedName : CN=10.2.0.0/22,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB
Location          : Montreal, Canada
Name              : 10.2.0.0/22
ObjectClass       : subnet
ObjectGUID        : fa21c05b-40da-4746-b210-60eed2c239fb
Site              : CN=MTL1,CN=Sites,CN=Configuration,DC=FX,DC=LAB

```


<h4>Adding a New subnet


```
PS C:\LazyWinAdmin> New-ADReplicationSubnet -Name '10.0.0.0/22' -site 'FX3' -Location 'Europe' -PassThru
```

```
DistinguishedName : CN=10.0.0.0/22,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB
Location          : Europe
Name              : 10.0.0.0/22
ObjectClass       : subnet
ObjectGUID        : b88d7b53-fa96-4454-8978-13ab032a0a16
Site              : CN=FX3,CN=Sites,CN=Configuration,DC=FX,DC=LAB
```


<h4>Adding a bunch of subnets

We re-use the same file used in the ADSI example (above) with a few subnets to add:
<a href="http://3.bp.blogspot.com/-_kFn8eojX_I/UoAagaVAEaI/AAAAAAABehI/qp4K8mFxric/s1600/2013-11-10+6-35-03+PM.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="https://3.bp.blogspot.com/-_kFn8eojX_I/UoAagaVAEaI/AAAAAAABehI/qp4K8mFxric/s1600/2013-11-10+6-35-03+PM.png" /></a>
<div class="separator" style="clear: both; text-align: left;"><b>Name</b>, <b>Location</b>,<b> Site</b> and <b>Description</b> properties are in the CSV file so the cmdlet will be able to interpret them.<div class="separator" style="clear: both; text-align: left;">
<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;"><tbody><tr><td style="text-align: center;"><a href="http://1.bp.blogspot.com/-wXnHmMkqjyY/UoApvSncyeI/AAAAAAABeho/jG-wHtWuUP8/s1600/2013-11-10+7-49-16+PM.png" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" src="https://1.bp.blogspot.com/-wXnHmMkqjyY/UoApvSncyeI/AAAAAAABeho/jG-wHtWuUP8/s1600/2013-11-10+7-49-16+PM.png" /></a></td></tr><tr><td class="tr-caption" style="text-align: center;">Get-Help New-ADReplicationSubnet -ShowWindow</td></tr></tbody></table><div class="separator" style="clear: both; text-align: left;">Here is the result using the <b>-Verbose</b> parameter.<div class="separator" style="clear: both; text-align: left;">By default <b>New-ADReplicationSubnet</b> cmdlet does not generate output, so here we only see the output of the verbose parameter.<div class="separator" style="clear: both; text-align: left;">


```
PS C:\LazyWinAdmin> import-csv .\subnets.csv | New-ADReplicationSubnet -Verbose
```

```
VERBOSE: Performing operation "New" on Target
"CN=192.168.1.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
VERBOSE: Performing operation "New" on Target
"CN=192.168.2.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
VERBOSE: Performing operation "New" on Target
"CN=192.168.3.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
VERBOSE: Performing operation "New" on Target
"CN=192.168.4.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
VERBOSE: Performing operation "New" on Target
"CN=192.168.5.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
VERBOSE: Performing operation "New" on Target
"CN=192.168.6.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
VERBOSE: Performing operation "New" on Target
"CN=192.168.7.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
VERBOSE: Performing operation "New" on Target
"CN=192.168.8.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```

The parameter <b>-PassThru</b> must be used if you want to see the output of this cmdlet.


```
PS C:\LazyWinAdmin> import-csv .\subnets.csv | New-ADReplicationSubnet -PassThru -Verbose
```

```

```
VERBOSE: Performing operation "New" on Target
"CN=192.168.1.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```
DistinguishedName : CN=192.168.1.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB Location          : Paris Name              : 192.168.1.0/24 ObjectClass       : subnet ObjectGUID        : 88f6be31-5f56-48bc-9986-fd3afe15cac9 Site              : CN=FX2,CN=Sites,CN=Configuration,DC=FX,DC=LAB  
```
VERBOSE: Performing operation "New" on Target
"CN=192.168.2.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```
DistinguishedName : CN=192.168.2.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB Location          : London Name              : 192.168.2.0/24 ObjectClass       : subnet ObjectGUID        : b2539cc1-7bff-4f62-bf17-29e7ac94dbbe Site              : CN=FX3,CN=Sites,CN=Configuration,DC=FX,DC=LAB  
```
VERBOSE: Performing operation "New" on Target
"CN=192.168.3.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```
DistinguishedName : CN=192.168.3.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB Location          : Montreal Name              : 192.168.3.0/24 ObjectClass       : subnet ObjectGUID        : 7f08aa1a-ad34-428b-9138-bf18e14d0610 Site              : CN=MTL1,CN=Sites,CN=Configuration,DC=FX,DC=LAB  
```
VERBOSE: Performing operation "New" on Target
"CN=192.168.4.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```
DistinguishedName : CN=192.168.4.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB Location          : London Name              : 192.168.4.0/24 ObjectClass       : subnet ObjectGUID        : 3bd6e017-9974-4eea-9dfd-a0dd3739b65f Site              : CN=FX3,CN=Sites,CN=Configuration,DC=FX,DC=LAB  
```
VERBOSE: Performing operation "New" on Target
"CN=192.168.5.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```
DistinguishedName : CN=192.168.5.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB Location          : London Name              : 192.168.5.0/24 ObjectClass       : subnet ObjectGUID        : 26c43a45-868d-4122-91f8-f2385e505bcf Site              : CN=FX3,CN=Sites,CN=Configuration,DC=FX,DC=LAB  
```
VERBOSE: Performing operation "New" on Target
"CN=192.168.6.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```
DistinguishedName : CN=192.168.6.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB Location          : Paris Name              : 192.168.6.0/24 ObjectClass       : subnet ObjectGUID        : 1fb55339-1798-4666-9d06-647358558bc2 Site              : CN=FX2,CN=Sites,CN=Configuration,DC=FX,DC=LAB  
```
VERBOSE: Performing operation "New" on Target
"CN=192.168.7.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```
DistinguishedName : CN=192.168.7.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB Location          : Montreal Name              : 192.168.7.0/24 ObjectClass       : subnet ObjectGUID        : 7ddcdfc3-bb97-41f6-94ab-26439da391d9 Site              : CN=MTL1,CN=Sites,CN=Configuration,DC=FX,DC=LAB  
```
VERBOSE: Performing operation "New" on Target
"CN=192.168.8.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB".
```
DistinguishedName : CN=192.168.8.0/24,CN=Subnets,CN=Sites,CN=Configuration,DC=FX,DC=LAB Location          : Paris Name              : 192.168.8.0/24 ObjectClass       : subnet ObjectGUID        : 494113f6-ac7a-4398-a155-2b1d3b400c15 Site              : CN=FX2,CN=Sites,CN=Configuration,DC=FX,DC=LAB 
```




### 




<a href="http://4.bp.blogspot.com/-priFeHo7qAw/UoA0QD01mjI/AAAAAAABeh4/Rvj0AePgs60/s1600/2013-11-10+8-34-25+PM.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="https://4.bp.blogspot.com/-priFeHo7qAw/UoA0QD01mjI/AAAAAAABeh4/Rvj0AePgs60/s1600/2013-11-10+8-34-25+PM.png" /></a>


