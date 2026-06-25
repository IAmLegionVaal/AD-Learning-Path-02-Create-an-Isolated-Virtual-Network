# Activity 02: Create an Isolated Virtual Network

## Goal
Build a private Hyper-V NAT network named `AD-LAB-NAT` on `10.10.10.0/24`. The lab VMs must communicate with each other without exposing experimental AD DNS or DHCP services to the physical LAN.

## Prerequisites
- Activity 01 topology plan
- Hyper-V enabled
- Elevated PowerShell
- Confirm `10.10.10.0/24` does not overlap the LAN, VPN, WSL, Docker, or another hypervisor network

## Setup steps
1. Review existing IPv4 routes with `Get-NetRoute -AddressFamily IPv4`.
2. Create an Internal Hyper-V virtual switch.
3. Assign `10.10.10.1/24` to the host-side virtual adapter.
4. Create a NAT object for `10.10.10.0/24`.
5. Attach future lab VMs to this switch.
6. Keep infrastructure systems static; do not configure lab DHCP until the DHCP module.

```powershell
$SwitchName = 'AD-LAB-NAT'
$Prefix = '10.10.10.0/24'
$Gateway = '10.10.10.1'

if (-not (Get-VMSwitch -Name $SwitchName -ErrorAction SilentlyContinue)) {
    New-VMSwitch -Name $SwitchName -SwitchType Internal
}

$Adapter = Get-NetAdapter -Name "vEthernet ($SwitchName)"
if (-not (Get-NetIPAddress -InterfaceIndex $Adapter.ifIndex -AddressFamily IPv4 -ErrorAction SilentlyContinue |
    Where-Object IPAddress -eq $Gateway)) {
    New-NetIPAddress -InterfaceIndex $Adapter.ifIndex -IPAddress $Gateway -PrefixLength 24
}

if (-not (Get-NetNat -Name $SwitchName -ErrorAction SilentlyContinue)) {
    New-NetNat -Name $SwitchName -InternalIPInterfaceAddressPrefix $Prefix
}
```

## Validation
```powershell
Get-VMSwitch -Name 'AD-LAB-NAT' | Select-Object Name,SwitchType
Get-NetIPAddress -InterfaceAlias 'vEthernet (AD-LAB-NAT)' -AddressFamily IPv4
Get-NetNat -Name 'AD-LAB-NAT' | Select-Object Name,InternalIPInterfaceAddressPrefix
```

Expected state: an Internal switch, host adapter `10.10.10.1/24`, and NAT prefix `10.10.10.0/24`.

## Troubleshooting
- Prefix overlap: choose another RFC1918 subnet and update the topology plan.
- No Internet: verify host access, VM gateway `10.10.10.1`, and the NAT object.
- No peer traffic: verify both VMs use the same switch and test Windows Firewall separately.

## Security notes
Do not bridge a training domain controller, DNS server, or DHCP server onto a production, customer, office, or home LAN. NAT reduces exposure but is not a malware-isolation boundary.

## Rollback
After disconnecting all VMs:
```powershell
Remove-NetNat -Name 'AD-LAB-NAT' -Confirm:$false
Remove-VMSwitch -Name 'AD-LAB-NAT' -Force
```

## Evidence
Record actual results under `evidence/`: route-overlap review, switch output, adapter address, NAT output, topology screenshot, connectivity results, deviations, failures, remediation, and final pass/fail status. Never commit credentials, licence keys, tokens, customer data, or unrelated personal information.
