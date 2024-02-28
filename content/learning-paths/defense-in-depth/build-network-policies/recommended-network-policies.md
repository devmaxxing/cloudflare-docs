---
title: Recommended network policies
pcx_content_type: learning-unit
weight: 1
layout: learning-unit
---

Add the following recommended network policies.

{{<details header="Quarantined-Users-NET-Restricted-Access" open="true">}}

Restrict access for users included in a {{<glossary-tooltip term_id="identity provider">}}identity provider (IdP){{</glossary-tooltip>}} user group for risky users. This policy ensures your security team can restrict traffic for users of whom malicious or suspicious activity was detected.

| Selector         | Operator    | Value                               | Logic | Action |
| ---------------- | ----------- | ----------------------------------- | ----- | ------ |
| Destination IP   | not in list | _Quarantined-Users-IPAllowlist_     | Or    | Block  |
| SNI              | not in list | _Quarantined-Users-HostAllowlist_   | Or    |        |
| Domain SNI       | not in list | _Quarantined-Users-DomainAllowlist_ | And   |        |
| User Group Names | in          | _Quarantined Users_                 |       |        |

{{</details>}}

{{<details header="Posture-Fail-NET-Restricted-Access" open="true">}}

Restrict access for devices where baseline posture checks have not passed. If posture checks are integrated with service providers such as Crowdstrike or Intune via the API, this policy dynamically block access for devices that do not meet predetermined security requirements.

| Selector                     | Operator    | Value                               | Logic | Action |
| ---------------------------- | ----------- | ----------------------------------- | ----- | ------ |
| Destination IP               | not in list | _Posture-Fail-IPAllowlist_          | Or    | Block  |
| SNI                          | not in list | _Posture-Fail-HostAllowlist_        | Or    |        |
| Domain SNI                   | not in list | _Posture-Fail-DomainAllowlist_      | And   |        |
| Passed Device Posture Checks | not in      | _Windows 10 or higher (OS version)_ |       |        |

You can add a number of WARP client device posture checks as needed, such as [Disk encryption](/cloudflare-one/identity/devices/warp-client-checks/disk-encryption/) and [Domain joined](/cloudflare-one/identity/devices/warp-client-checks/domain-joined/). For more information on device posture checks, refer to [Enforce device posture](/cloudflare-one/identity/devices/).

{{</details>}}

{{<details header="FinanceUsers-NET-HTTPS-FinanceServers (example)" open="true">}}

Allow HTTPS access for user groups. For example, the following policy gives finance users access to any known financial applications:

| Selector         | Operator | Value             | Logic | Action |
| ---------------- | -------- | ----------------- | ----- | ------ |
| Destination IP   | in list  | _Finance Servers_ | And   | Allow  |
| User Group Names | in       | _Finance Users_   |       |        |

{{</details>}}

{{<details header="All-NET-Internet-Blocklist" open="true">}}

Block traffic to destination IPs, {{<glossary-tooltip term_id="Server Name Indication (SNI)">}}SNIs{{</glossary-tooltip>}}, and domain SNIs that are malicious or pose a threat to your organization.

{{<render file="zero-trust/_threat-intelligence-automation.md">}}

| Selector       | Operator | Value              | Logic | Action |
| -------------- | -------- | ------------------ | ----- | ------ |
| Destination IP | in list  | _IP Blocklist_     | Or    | Block  |
| SNI            | in list  | _Host Blocklist_   | Or    |        |
| Domain SNI     | in list  | _Domain Blocklist_ |       |        |

{{</details>}}

{{<Aside type="note">}}The Detected Protocol selector is only available for Enterprise users. For more information, refer to [Protocol detection](/cloudflare-one/policies/gateway/network-policies/protocol-detection/).{{</Aside>}}

{{<details header="All-NET-SSH-Internet-Allowlist" open="true">}}

Allow SSH traffic to specific endpoints on the Internet for specific users. You can create a similar policy for other non-web endpoints that required access.

Optionally, you can include a selector to filter by source IP or IdP group.

| Selector          | Operator | Value               | Logic | Action |
| ----------------- | -------- | ------------------- | ----- | ------ |
| Destination IP    | in list  | _SSHAllowList_      | Or    | Allow  |
| SNI               | in list  | _SSHAllowlistFQDN_  | And   |        |
| Detected Protocol | is       | _SSH_               | And   |        |
| User Group Names  | in       | _SSH-Allowed-Users_ |       |        |

{{</details>}}

{{<details header="All-NET-NO-HTTP-HTTPS-Internet-Deny" open="true">}}

Block all non-web traffic towards the Internet. By using the **Detected Protocol** selector, you will ensure alternative ports for HTTP and HTTPS are allowed.

| Selector          | Operator    | Value             | Logic | Action |
| ----------------- | ----------- | ----------------- | ----- | ------ |
| Destination IP    | not in list | _InternalNetwork_ | And   | Block  |
| Detected Protocol | is not in   | _HTTP_, _HTTPS_   |       |        |

{{</details>}}

{{<details header="All-NET-InternalNetwork-ImplicitDeny" open="true">}}

Implicitly deny all of your internal IP ranges included in a list. We recommend you place this policy at the [bottom of your policy list](/learning-paths/defense-in-depth/build-dns-policies/order-of-precedence/) to ensure you explicitly approve traffic defined in the above policies.

| Selector       | Operator | Value                  | Action |
| -------------- | -------- | ---------------------- | ------ |
| Destination IP | in list  | _Internal Network IPs_ | Block  |

{{</details>}}