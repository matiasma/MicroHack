# NOTES

Tips and trick for maintaining the linux microhack.

### Azure CLI

~~~bash
# List all resource groups
az group list --query [].name -o tsv

# List all resources inside a resource group
sourceRgName=$(az group list --query "[?ends_with(name, 'source-rg')].name" -o tsv)
destinationRgName=$(az group list --query "[?ends_with(name, 'destination-rg')].name" -o tsv)
az resource list -g $sourceRgName -o table

# Login to Azure with local user
az network bastion ssh -n $srcBastionName -g $srcRgName --target-resource-id $srcVm1Id --auth-type password --username azuresuser
az network bastion ssh -n $sourceBastionName -g $sourceRgName --target-resource-id $sourceVm1Id --auth-type AAD

# Delete VMs only
az vm delete --ids $sourceVm1Id $sourceVm2Id --yes
az vm delete --id $sourceVm2Id --yes --no-wait

# Delete resource group
az group delete --name ${prefix}1-$suffix-destination-rg --yes --no-wait
az group delete --name ${prefix}1-$suffix-source-rg --yes --no-wait

# Install Azure SSH AAD
az vm extension set --publisher Microsoft.Azure.ActiveDirectory --name AADSSHLoginForLinux --ids $sourceVM1Id $sourceVM2Id

az quota create --resource-name StandardSkuPublicIpAddresses --scope /subscriptions/$subid/providers/Microsoft.Network/locations/$location --limit-object value=100 --resource-type PublicIpAddresses
az quota create -h

# Assign application developer role to user of a group
# Get the object ID of the custom role
role_id=$(az rest --method GET --uri "https://graph.microsoft.com/v1.0/directoryRoles" | jq -r '.value[] | select(.displayName == "Application Developer") | .id')

# Get the object ID of the AAD group
group_id=$(az ad group show --group 'MH - Linux Migration' --query id -o tsv)

# assign the role to the group
az rest --method POST --uri "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments" --headers "Content-type=application/json" --body '{"@odata.type": "#microsoft.graph.unifiedRoleAssignment","roleDefinitionId": "'$role_id'","principalId": "'$group_id'","directoryScopeId": "/"}'

az rest --method POST --uri "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments" --headers "Content-type=application/json" --body '{"@odata.type": "#microsoft.graph.unifiedRoleAssignment","roleDefinitionId": "cf1c38e5-3621-4004-a7cb-879624dced7c","principalId": "'$group_id'","directoryScopeId": "/"}'
~~~

### Linux

~~~bash
# Open Red Hat Firewall 
# List all firewall rules
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload
sudo systemctl status firewalld
sudo systemctl stop firewalld
sudo systemctl disable firewalld
~~~

### Azure Quotas

2 Windows (Windows Server 2019 Datacenter), Standard B8ms (8 vcpus, 32 GiB memory)
6 Linux, Standard D2s v5 (2 vcpus, 8 GiB memory)
= 2*8 + 6*2 = 28 vcpus
~30 vcpus/per table
8 VMs in total per user
8 public IPs per user + 1 for the load balancer + 1 for the bastion host = 10 public IPs per user

Per Table
- 10 public IPs
- 8 VMs
- 30 vcpus

### screen capture and mp4 to animated gif

Screen capture with Microsoft [Clipchamp](https://clipchamp.com/en/screen-recorder/)
~~~bash

# Install ffmpeg on Ubuntu
sudo apt install ffmpeg -y
# Install imagemagick
sudo apt install imagemagick -y
cd resources
chmod +x ./resources/mp4togif.sh
./mp4togif.sh ./media/mh.linux.login.mp4 ./media/mh.linux.login.gif
mv ./media/mh.linux.login.gif ../walkthrough/challenge-1/img/mh.linux.login.gif

./mp4togif.sh ./media/mh.linux.webserver.test.mp4 ./media/mh.linux.webserver.test.gif
mv ./media/mh.linux.webserver.test.gif ../walkthrough/challenge-1/img/mh.linux.webserver.test.gif

~~~



