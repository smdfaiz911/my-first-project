# Get all mailboxes
$Mailboxes = Get-Mailbox -ResultSize Unlimited

$Report = foreach ($Mbx in $Mailboxes) {

    # Get user object (contains AccountDisabled)
    $Usr = Get-User -Identity $Mbx.PrimarySmtpAddress -ErrorAction SilentlyContinue

    # Skip if not disabled
    if (-not $Usr.AccountDisabled) { continue }

    # Check forwarding
    $FwdSMTP = $Mbx.ForwardingSmtpAddress
    $FwdAddr = $Mbx.ForwardingAddress

    if ($FwdSMTP -or $FwdAddr) {

        # Resolve forwarding target email
        if ($FwdSMTP) {
            $TargetEmail = $FwdSMTP -replace "^SMTP:", ""
        }
        else {
            $TargetEmail = (Get-Recipient $FwdAddr).PrimarySmtpAddress
        }

        # Get target user and its status
        $TargetUser = Get-User $TargetEmail -ErrorAction SilentlyContinue

        [PSCustomObject]@{
            DisabledUser       = $Mbx.DisplayName
            DisabledUserEmail  = $Mbx.PrimarySmtpAddress
            ForwardingTo       = $TargetEmail
            TargetIsActive     = if ($TargetUser) { -not $TargetUser.AccountDisabled } else { $null }
        }
    }
}

$Report
