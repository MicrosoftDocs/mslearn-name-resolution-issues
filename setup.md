# Variables
[string] $vnetName1 = "VNet1";
[string] $vnetName2 = "VNet2";
[string] $subnetName1 = "Subnet1";
[string] $subnetName2 = "Subnet2";
[string] $subnetName3 = "default";

[string] $addressPrefixVNet1 = "10.1.0.0/16";
[string] $addressPrefixVNet2 = "10.2.0.0/16";

[string] $echoRequestRemoteAddress = "'10.1.0.0/255.255.0.0', '10.2.0.0/255.255.0.0'";

[string] $addressPrefixSubnet1 = "10.1.1.0/24";
[string] $addressPrefixSubnet2 = "10.1.2.0/24";
[string] $addressPrefixSubnet3 = "10.2.0.0/24";

[string] $nsgName1 = "NSG1";
[string] $nsgName2 = "NSG2";
[string] $nsgName3 = "NSG3";

[string] $dnsZoneName = "contoso.com";
[string] $dnsLinkName1 = "vnet1_dns";

[string] $nicName1 = "VM1-nic";
[string] $nicName2 = "VM2-nic";
[string] $nicName3 = "VM3-nic";

[string] $userName = "AdminXyz";
[string] $password = "sfr9jttzrjjeoem7hrf#";

[string] $vmName1 = "VM1";
[string] $vmName2 = "VM2";
[string] $vmName3 = "VM3";

[string] $publicIpAdName1 = "VM1-ip";
[string] $publicIpAdName2 = "VM2-ip";
[string] $publicIpAdName3 = "VM3-ip";


# Get resource group
# The subscription should contain just one empty resource group
$resourceGroup = (Get-AzResourceGroup)[0];


# Create VNet1
$subnet1 = New-AzVirtualNetworkSubnetConfig -Name $subnetName1 -AddressPrefix $addressPrefixSubnet1;
$subnet2 = New-AzVirtualNetworkSubnetConfig -Name $subnetName2 -AddressPrefix $addressPrefixSubnet2;

$vnet1 = New-AzVirtualNetwork `
    -Name $vnetName1 `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -Location $resourceGroup.Location `
    -AddressPrefix $addressPrefixVNet1 `
    -Subnet $subnet1,$subnet2;


# Create VNet2
$subnet3 = New-AzVirtualNetworkSubnetConfig -Name $subnetName3 -AddressPrefix $addressPrefixSubnet3;

$vnet2 = New-AzVirtualNetwork `
    -Name $vnetName2 `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -Location $resourceGroup.Location `
    -AddressPrefix $addressPrefixVNet2 `
    -Subnet $subnet3;


# Peer the vnets
Add-AzVirtualNetworkPeering `
    -Name Peer1-2 `
    -VirtualNetwork $vnet1 `
    -RemoteVirtualNetworkId $vnet2.Id;

Add-AzVirtualNetworkPeering `
    -Name Peer2-1 `
    -VirtualNetwork $vnet2 `
    -RemoteVirtualNetworkId $vnet1.Id;


# Create network security groups
$rule1 = New-AzNetworkSecurityRuleConfig `
    -Name Allow-RDP `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 100 `
    -SourceAddressPrefix Internet `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389;

# NSG1
$nsg1 = New-AzNetworkSecurityGroup `
    -Name $nsgName1 `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -Location $resourceGroup.Location `
    -SecurityRules $rule1;    

Set-AzVirtualNetworkSubnetConfig `
    -Name $subnetName1 `
    -VirtualNetwork $vnet1 `
    -AddressPrefix $addressPrefixSubnet1 `
    -NetworkSecurityGroup $nsg1;

$vnet1 | Set-AzVirtualNetwork;

# NSG2
$nsg2 = New-AzNetworkSecurityGroup `
    -Name $nsgName2 `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -Location $resourceGroup.Location `
    -SecurityRules $rule1;    

Set-AzVirtualNetworkSubnetConfig `
    -Name $subnetName2 `
    -VirtualNetwork $vnet1 `
    -AddressPrefix $addressPrefixSubnet2 `
    -NetworkSecurityGroup $nsg2;

$vnet1 | Set-AzVirtualNetwork;

# NSG3
$nsg3 = New-AzNetworkSecurityGroup `
    -Name $nsgName3 `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -Location $resourceGroup.Location `
    -SecurityRules $rule1;    

Set-AzVirtualNetworkSubnetConfig `
    -Name $subnetName3 `
    -VirtualNetwork $vnet2 `
    -AddressPrefix $addressPrefixSubnet3 `
    -NetworkSecurityGroup $nsg3;

$vnet2 | Set-AzVirtualNetwork;


# Create private DNS zone
New-AzPrivateDnsZone `
    -Name $dnsZoneName `
    -ResourceGroupName $resourceGroup.ResourceGroupName;

New-AzPrivateDnsVirtualNetworkLink `
    -Name $dnsLinkName1 `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -ZoneName $dnsZoneName `
    -VirtualNetwork $vnet1 `
    -EnableRegistration;


# Create public IP addresses
$pip1 = New-AzPublicIpAddress `
    -Name $publicIpAdName1 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -AllocationMethod Dynamic `
    -IdleTimeoutInMinutes 4;

$pip2 = New-AzPublicIpAddress `
    -Name $publicIpAdName2 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -AllocationMethod Dynamic `
    -IdleTimeoutInMinutes 4;

$pip3 = New-AzPublicIpAddress `
    -Name $publicIpAdName3 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -AllocationMethod Dynamic `
    -IdleTimeoutInMinutes 4;


# Create virtual network cards
# VM1-nic
$subnetCfg1 = Get-AzVirtualNetworkSubnetConfig `
    -Name $subnetName1 `
    -VirtualNetwork $vnet1;

$nic1 = New-AzNetworkInterface `
    -Name $nicName1 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -SubnetId $subnetCfg1.Id `
    -PublicIpAddressId $pip1.Id;

# VM2-nic
$subnetCfg2 = Get-AzVirtualNetworkSubnetConfig `
    -Name $subnetName2 `
    -VirtualNetwork $vnet1;

$nic2 = New-AzNetworkInterface `
    -Name $nicName2 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -SubnetId $subnetCfg2.Id `
    -PublicIpAddressId $pip2.Id;

# VM3-nic
$subnetCfg3 = Get-AzVirtualNetworkSubnetConfig `
    -Name $subnetName3 `
    -VirtualNetwork $vnet2;

$nic3 = New-AzNetworkInterface `
    -Name $nicName3 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -SubnetId $subnetCfg3.Id `
    -PublicIpAddressId $pip3.Id;


# Create virtual machines
# Credential
$pw = $password | ConvertTo-SecureString -Force -AsPlainText;
$credential = New-Object PSCredential($userName, $pw);

# VM1
$vmConfig1 = New-AzVMConfig `
    -VMName $vmName1 `
    -VMSize "Standard_D2s_v3" `
    | `
    Set-AzVMOperatingSystem `
        -Windows `
        -ComputerName $vmName1 `
        -Credential $credential `
    | `
    Set-AzVMSourceImage `
          -PublisherName MicrosoftWindowsServer `
          -Offer WindowsServer `
          -Sku 2019-Datacenter `
          -Version latest `
    | `
    Set-AzVMBootDiagnostic `
        -Enable `
        -ResourceGroupName $resourceGroup.ResourceGroupName `
    | `
    Add-AzVMNetworkInterface -Id $nic1.Id;

New-AzVM `
    -VM $vmConfig1 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName;

# VM2
$vmConfig2 = New-AzVMConfig `
    -VMName $vmName2 `
    -VMSize "Standard_D2s_v3" `
    | `
    Set-AzVMOperatingSystem `
        -Windows `
        -ComputerName $vmName2 `
        -Credential $credential `
    | `
    Set-AzVMSourceImage `
          -PublisherName MicrosoftWindowsServer `
          -Offer WindowsServer `
          -Sku 2019-Datacenter `
          -Version latest `
    | `
    Set-AzVMBootDiagnostic `
        -Enable `
        -ResourceGroupName $resourceGroup.ResourceGroupName `
    | `
    Add-AzVMNetworkInterface -Id $nic2.Id;

New-AzVM `
    -VM $vmConfig2 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName;

# VM3
$vmConfig3 = New-AzVMConfig `
    -VMName $vmName3 `
    -VMSize "Standard_D2s_v3" `
    | `
    Set-AzVMOperatingSystem `
        -Windows `
        -ComputerName $vmName3 `
        -Credential $credential `
    | `
    Set-AzVMSourceImage `
          -PublisherName MicrosoftWindowsServer `
          -Offer WindowsServer `
          -Sku 2019-Datacenter `
          -Version latest `
    | `
    Set-AzVMBootDiagnostic `
        -Enable `
        -ResourceGroupName $resourceGroup.ResourceGroupName `
    | `
    Add-AzVMNetworkInterface -Id $nic3.Id;

New-AzVM `
    -VM $vmConfig3 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName;


# Allow ICMPv4 Echo Requests
$settingString = '{"commandToExecute":"powershell Set-NetFirewallRule -Name FPS-ICMP4-ERQ-In -Enabled True -RemoteAddress @(' + $echoRequestRemoteAddress + ');"}';

Set-AzVMExtension `
    -VMName $vmName1 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -ExtensionName "AllowEchoRequest" `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.8 `
    -SettingString $settingString;

Set-AzVMExtension `
    -VMName $vmName2 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -ExtensionName "AllowEchoRequest" `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.8 `
    -SettingString $settingString;

Set-AzVMExtension `
    -VMName $vmName3 `
    -Location $resourceGroup.Location `
    -ResourceGroupName $resourceGroup.ResourceGroupName `
    -ExtensionName "AllowEchoRequest" `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.8 `
    -SettingString $settingString;
