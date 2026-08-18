---
inclusion: manual
---

# Skill: New-RestApiFunction

Scaffold a PowerShell 7+ function that wraps a REST API endpoint, following workspace REST/JSON standards.

## Procedure

1. **Gather requirements** — ask the user for:
   - API name/service (Zerto, Infoblox, vCenter, Azure, custom)
   - Base URL pattern
   - Authentication method (Bearer token, API key, Keycloak, basic auth)
   - HTTP method(s) needed (GET, POST, PUT, PATCH, DELETE)
   - Request body structure (if mutation)
   - Response structure (what fields matter)
   - Pagination? (offset, cursor, link-header)
   - Rate limiting concerns?

2. **Determine auth pattern** based on service:

   | Service | Auth Pattern |
   |---------|-------------|
   | Zerto 10.x | Keycloak token at `/auth/realms/zerto/protocol/openid-connect/token`, client_id=zerto-client, port 443 |
   | Infoblox | Basic auth, `_max_results` pagination |
   | vCenter | Session token via POST `/rest/com/vmware/cis/session` |
   | Azure | Bearer from `Get-AzAccessToken` |
   | Generic | Accept `[PSCredential]` or `-Token` parameter |

3. **Generate the function** with this structure:

```powershell
function Verb-ApiNoun {
    <#
    .SYNOPSIS
        <one-line description>

    .DESCRIPTION
        Wraps the <ServiceName> API endpoint: <METHOD> <path>
        Handles authentication, pagination, and error responses.

    .PARAMETER BaseUri
        Base URI of the API service (e.g., 'https://api.example.com')

    .PARAMETER Credential
        PSCredential for authentication. Do not hardcode tokens.

    .EXAMPLE
        $Cred = Get-Credential
        Verb-ApiNoun -BaseUri 'https://api.example.com' -Credential $Cred

    .EXAMPLE
        Verb-ApiNoun -BaseUri 'https://api.example.com' -Token $BearerToken -Filter 'active'
    #>
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param(
        [Parameter(Mandatory)]
        [ValidateNotNullOrEmpty()]
        [string]$BaseUri,

        [Parameter(Mandatory, ParameterSetName = 'Credential')]
        [System.Management.Automation.PSCredential]
        [System.Management.Automation.Credential()]
        $Credential,

        [Parameter(Mandatory, ParameterSetName = 'Token')]
        [ValidateNotNullOrEmpty()]
        [string]$Token
    )

    begin {
        Set-StrictMode -Version Latest
        $Results = [System.Collections.Generic.List[PSCustomObject]]::new()

        # Build auth header
        $Headers = @{}
        if ($PSCmdlet.ParameterSetName -eq 'Token') {
            $Headers['Authorization'] = "Bearer $Token"
        } elseif ($PSCmdlet.ParameterSetName -eq 'Credential') {
            $EncodedAuth = [Convert]::ToBase64String(
                [Text.Encoding]::ASCII.GetBytes(
                    "$($Credential.UserName):$($Credential.GetNetworkCredential().Password)"
                )
            )
            $Headers['Authorization'] = "Basic $EncodedAuth"
        }

        # Cert bypass toggle — single place to control
        $SkipCert = @{ SkipCertificateCheck = $true }
    }

    process {
        $Page = 0
        $HasMore = $true

        while ($HasMore) {
            $Params = @{
                Uri                     = "$BaseUri/api/v1/endpoint?_page=$Page&_max_results=1000"
                Method                  = 'GET'
                Headers                 = $Headers
                ContentType             = 'application/json'
                StatusCodeVariable      = 'HttpCode'
                ResponseHeadersVariable = 'RespHeaders'
                SkipHttpErrorCheck      = $true
                @SkipCert
            }

            $Response = Invoke-RestMethod @Params

            if ($HttpCode -ne 200) {
                Write-Error "API returned HTTP $HttpCode for endpoint. Response: $($Response | ConvertTo-Json -Depth 5)"
                return
            }

            foreach ($Item in $Response) {
                $Result = [PSCustomObject]@{
                    Id     = $Item.id
                    Name   = $Item.name
                    Status = $Item.status
                }
                $Results.Add($Result)
                $Result  # emit to pipeline
            }

            # Pagination check — adapt to API's pattern
            if ($Response.Count -lt 1000) {
                $HasMore = $false
            } else {
                $Page++
            }
        }
    }

    end {
        Write-Verbose "Retrieved $($Results.Count) total items"
    }
}
```

4. **Add token refresh helper** if service uses expiring tokens:

```powershell
function Get-ApiToken {
    <#
    .SYNOPSIS
        Obtain or refresh bearer token for <ServiceName>.
    #>
    [CmdletBinding()]
    [OutputType([string])]
    param(
        [Parameter(Mandatory)]
        [string]$AuthUri,

        [Parameter(Mandatory)]
        [PSCredential]$Credential
    )

    begin {
        Set-StrictMode -Version Latest
    }

    process {
        $Body = @{
            grant_type = 'password'
            client_id  = 'zerto-client'  # adjust per service
            username   = $Credential.UserName
            password   = $Credential.GetNetworkCredential().Password
        }

        $Params = @{
            Uri                = $AuthUri
            Method             = 'POST'
            Body               = $Body
            ContentType        = 'application/x-www-form-urlencoded'
            SkipCertificateCheck = $true
            StatusCodeVariable = 'HttpCode'
        }

        $Response = Invoke-RestMethod @Params

        if ($HttpCode -ne 200) {
            Write-Error "Token request failed with HTTP $HttpCode"
            return
        }

        $Response.access_token
    }
}
```

5. **Mutation operations** (POST/PUT/PATCH) — add:
   - `SupportsShouldProcess` to CmdletBinding
   - Body construction with `[ordered]@{}` piped to `ConvertTo-Json -Depth 10`
   - Before/after state logging pattern
   - `ContentType = 'application/json'` explicit on all mutations

6. **Final checks:**
   - No hardcoded tokens or secrets
   - Splatting used for all `Invoke-RestMethod` calls
   - `ConvertTo-Json -Depth 10` everywhere (never default depth 2)
   - `[ordered]@{}` for all JSON body construction
   - `-StatusCodeVariable` used instead of try/catch for HTTP errors
   - Pagination handles all items (not just first page)
   - URL-encode special characters: `[System.Uri]::EscapeDataString()`
   - Examples use `example.com` domains, RFC 5737 IPs
