# Evidence: Activity 02

Store observed lab evidence here. Do not copy expected output and present it as a completed result.

## Collect
- Existing route review proving the subnet does not overlap
- Hyper-V switch configuration
- Host virtual-adapter address
- NAT configuration
- Topology screenshot
- Peer and outbound connectivity results
- Any errors, root cause, remediation, and post-fix validation

## Suggested files
```text
01-before-state.txt
02-implementation.txt
03-validation.txt
04-screenshot.png
findings.md
```

## Findings record
Document the date, operator, host/hypervisor, commands used, expected state, observed state, deviations, troubleshooting, security observations, and final pass/fail result.

Before committing, remove passwords, product keys, recovery keys, private keys, tokens, customer information, production names, public addresses tied to real organizations, and unrelated personal data.

Hash important text evidence with:
```powershell
Get-FileHash .\evidence\03-validation.txt -Algorithm SHA256
```
