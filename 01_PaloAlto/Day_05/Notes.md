# Day 05 - Advanced NAT and NAT Troubleshooting

## Topics Covered

- Day 4 NAT Revision
- Static NAT
- Dynamic NAT
- Static NAT vs Dynamic NAT
- U-Turn NAT / Hairpin NAT
- NAT Rule Matching
- First Match Wins in NAT Policies
- Specific Rules vs General Rules
- NAT Troubleshooting
- Routing Issues
- DNS Issues
- Application-Default Issues
- Missing Security Policy
- Missing Source NAT
- Missing Destination NAT
- Production Troubleshooting Scenarios

---

# 1. Day 4 NAT Revision

Before starting Advanced NAT, I revised the important NAT fundamentals.

## Source NAT

Source NAT changes the source IP address.

Example:

Before NAT:

Source IP: 192.168.10.25  
Destination IP: 8.8.8.8

After Source NAT:

Source IP: 49.204.100.20  
Destination IP: 8.8.8.8

Memory Rule:

> Source NAT changes the Source IP.

---

## Destination NAT

Destination NAT changes the destination IP address.

Example:

Public IP:

49.204.100.50

Internal Server:

192.168.50.10

Translation:

49.204.100.50 → 192.168.50.10

Memory Rule:

> Destination NAT changes the Destination IP.

---

## PAT / DIPP

PAT stands for:

Port Address Translation

In Palo Alto, this is commonly called:

Dynamic IP and Port (DIPP)

PAT allows multiple internal users to share the same public IP using different port mappings.

Example:

Ram:

192.168.1.10:55000

↓

49.204.100.25:10001

John:

192.168.1.11:55001

↓

49.204.100.25:10002

The public IP is the same, but different translated ports help the firewall keep the sessions separate.

Memory Rule:

> PAT/DIPP can translate Source IP + Source Port.

---

# 2. Why Do We Need NAT?

There are three important reasons.

## Reason 1: Private IP Addresses Are Not Routable on the Public Internet

Examples of private IP ranges:

- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

If an internal user's packet goes toward the public Internet with:

Source IP: 192.168.10.25

the private source IP cannot be normally routed across the public Internet.

Source NAT translates it into a public IP.

---

## Reason 2: Public IPv4 Addresses Are Limited

Without NAT, every internal device could require its own public IPv4 address.

NAT and PAT allow many internal devices to share public IP resources.

---

## Reason 3: Internal Addressing Is Hidden

The Internet sees the translated public IP instead of the original private IP.

Example:

Private IP:

192.168.10.25

Translated Public IP:

49.204.100.20

Important:

> NAT is not a replacement for Security Policy.

---

# 3. NAT vs Security Policy

This is one of the most important concepts.

## NAT

NAT translates traffic.

## Security Policy

Security Policy allows or denies traffic.

Golden Rule:

> NAT translates traffic. Security Policy allows or denies traffic.

A correct NAT rule does not automatically mean that traffic is allowed.

Example:

Destination NAT:

49.204.100.50 → 192.168.50.10

But:

Security Policy = No matching Allow rule

Result:

Traffic is denied.

---

# 4. Static NAT

Static NAT creates a fixed mapping.

Example:

Private Server:

192.168.10.100

Public IP:

203.0.113.10

Mapping:

203.0.113.10 ↔ 192.168.10.100

The mapping remains fixed.

## Common Use Cases

- Public web servers
- Mail servers
- Fixed external services
- IP whitelisting requirements
- Systems that require a consistent public IP

## Memory Question

Ask:

> Does the public IP need to remain constant?

If YES:

> Static NAT

---

# 5. Dynamic NAT

Dynamic NAT assigns a public IP from a configured pool.

Example:

Public IP Pool:

49.204.100.10  
49.204.100.11  
49.204.100.12

A user may use:

Today:

49.204.100.10

Tomorrow:

49.204.100.11

This can be expected behaviour because the translated public IP is selected dynamically from the NAT pool.

## Common Use Case

General outbound Internet access where a fixed translated public IP is not required.

## Memory Question

Ask:

> Does the public IP need to remain constant?

If NO:

> Dynamic NAT

---

# 6. Static NAT vs Dynamic NAT

| Feature | Static NAT | Dynamic NAT |
|---|---|---|
| Mapping | Fixed | Dynamic/temporary |
| Public IP changes | No | Can change |
| Public server | Common use case | Usually not appropriate when fixed IP is required |
| Employee Internet access | Usually unnecessary | Common use case |
| IP whitelisting | Appropriate when a fixed IP is required | Usually inappropriate if the translated IP can change |

Golden Rule:

> Fixed public IP required → Static NAT.

> Public IP can change → Dynamic NAT.

---

# 7. DHCP vs Dynamic NAT

These two concepts should not be confused.

## DHCP

DHCP assigns an IP address to a device, commonly a private IP inside an enterprise network.

Example:

Laptop today:

192.168.1.10

Laptop later:

192.168.1.15

The address can change depending on DHCP configuration and lease behaviour.

## Dynamic NAT

Dynamic NAT changes the translated public IP used by traffic.

Example:

Private IP:

192.168.1.10

Today translated to:

49.204.100.10

Later translated to:

49.204.100.11

Memory Rule:

> DHCP → Device IP assignment.

> Dynamic NAT → Dynamic translation using a NAT address pool.

---

# 8. U-Turn NAT / Hairpin NAT

U-Turn NAT is used when an internal user accesses an internal server by using the server's public IP or public DNS name.

Example:

Company portal:

portal.company.com

DNS resolves it to:

203.0.113.10

But the actual server is internal:

192.168.10.100

An internal employee opens:

portal.company.com

DNS returns:

203.0.113.10

The firewall translates the destination back toward the internal server:

203.0.113.10 → 192.168.10.100

The traffic effectively makes a U-turn at the firewall and goes back into the internal network.

---

# 9. Why Do Companies Use U-Turn NAT?

Without U-Turn NAT, a company might have to tell employees:

Inside office:

portal.internal.company.com

Outside office:

portal.company.com

This creates unnecessary confusion.

With U-Turn NAT, users can use:

portal.company.com

from both inside and outside the organization.

Business Advantage:

> Users do not need to remember different URLs depending on their location.

Memory Rule:

> Internal user accessing the company's own internal server through its public IP or public DNS name → Think U-Turn NAT.

---

# 10. Source NAT vs Destination NAT vs U-Turn NAT

## Source NAT

Typical scenario:

Internal User → Internet

Main translation:

Source IP changes.

---

## Destination NAT

Typical scenario:

External User → Internal Server

Main translation:

Destination IP changes.

---

## U-Turn NAT

Typical scenario:

Internal User → Company's Public IP → Internal Server

The traffic reaches the firewall and turns back toward the internal server.

Easy Memory Trick:

> Source NAT → Going out.

> Destination NAT → Coming in.

> U-Turn NAT → Starts toward a public address and turns back toward an internal server.

---

# 11. NAT Rule Matching

Palo Alto evaluates NAT rules from top to bottom.

The first matching NAT rule is selected.

This is called:

> First Match Wins.

Example:

| Rule | Source | Translated Public IP |
|---|---|---|
| Rule 1 | Any | 49.204.100.20 |
| Rule 2 | Finance 192.168.10.0/24 | 49.204.100.10 |

Finance User:

192.168.10.25

The firewall checks Rule 1.

Source = Any

Does 192.168.10.25 match Any?

Yes.

Therefore:

Rule 1 matches.

The firewall stops checking further NAT rules.

Rule 2 is not reached.

---

# 12. First Match Wins

The firewall checks rules:

Top → Bottom

When the first complete match is found:

> STOP.

The firewall does not continue looking for a more specific rule below.

Important:

> Palo Alto does not simply choose the most specific NAT rule regardless of order.

Rule order matters.

---

# 13. Specific Rules vs General Rules

This was one of the most important lessons from Day 5.

Correct design:

1. Specific Rules
2. General Rules

Example:

| Rule | Source | Translation |
|---|---|---|
| Rule 1 | Finance 192.168.10.0/24 | 49.204.100.10 |
| Rule 2 | HR 192.168.20.0/24 | 49.204.100.20 |
| Rule 3 | Any | 49.204.100.30 |

This allows Finance and HR to match their specific rules before the general Any rule.

Incorrect design:

| Rule | Source | Translation |
|---|---|---|
| Rule 1 | Any | 49.204.100.30 |
| Rule 2 | Finance 192.168.10.0/24 | 49.204.100.10 |
| Rule 3 | HR 192.168.20.0/24 | 49.204.100.20 |

Problem:

The Any rule matches Finance and HR traffic first.

The specific rules below may never be reached.

Golden Rule:

> Specific Rules on Top → General Rules Below.

---

# 14. Shadowing Problem

A specific rule may become ineffective if a broader rule above it always matches the same traffic first.

Example:

Rule 1:

Source = Any

Rule 2:

Source = Finance 192.168.10.0/24

A Finance user:

192.168.10.25

matches Rule 1 first because Any includes the Finance subnet.

Because of First Match Wins, Rule 2 is not evaluated for that session.

Memory Connection:

> First Match Wins + General rule above specific rule = Specific rule may never be reached.

---

# 15. NAT Rule Matching Scenario

Rules:

| Rule | Source | Public IP |
|---|---|---|
| Rule 1 | Finance 192.168.10.0/24 | 49.204.100.10 |
| Rule 2 | HR 192.168.20.0/24 | 49.204.100.20 |
| Rule 3 | Any | 49.204.100.30 |

User IP:

192.168.50.25

Firewall checks:

Rule 1:

Does 192.168.50.25 belong to 192.168.10.0/24?

No.

Rule 2:

Does 192.168.50.25 belong to 192.168.20.0/24?

No.

Rule 3:

Source = Any

Match.

Result:

Rule 3 is used.

Translated Public IP:

49.204.100.30

---

# 16. NAT Troubleshooting - Missing Subnet

Scenario:

Finance subnet:

192.168.10.0/24

HR subnet:

192.168.20.0/24

NAT rule only contains:

192.168.10.0/24

Result:

Finance Internet → Works.

HR Internet → Fails.

Possible Root Cause:

The HR subnet is not covered by the required Source NAT rule.

Possible Fix:

Depending on the intended design:

- Add the HR subnet to the appropriate NAT rule, or
- Create a separate NAT rule for HR.

Important:

Do not simply replace the Finance subnet if Finance users are already working.

---

# 17. Wrong Public IP Troubleshooting

Scenario:

Finance users should use:

49.204.100.10

But they are using:

49.204.100.20

NAT Rules:

| Rule | Source | Public IP |
|---|---|---|
| Rule 1 | Any | 49.204.100.20 |
| Rule 2 | Finance 192.168.10.0/24 | 49.204.100.10 |

Root Cause:

The general Any rule is above the specific Finance rule.

Finance traffic matches Rule 1 first.

Fix:

Reorder the rules:

1. Finance specific rule
2. General Any rule

---

# 18. NAT Correct but Internet Still Fails

Scenario:

- Source NAT = Correct
- Translated Public IP = Correct
- Security Policy = Allow
- User can reach firewall gateway
- Internet still fails

Next Check:

Routing.

The firewall must have a valid route toward the Internet.

Example:

Default Route:

0.0.0.0/0 → ISP Next Hop

Memory Rule:

> NAT correct + Security Policy correct + Internet fails → Check routing.

---

# 19. IP Works but Domain Name Fails

Scenario:

User can ping:

8.8.8.8

But cannot access:

google.com

Likely issue:

DNS resolution.

Troubleshooting command:

nslookup google.com

Memory Rule:

> IP works but hostname fails → Check DNS first.

Possible checks:

- Correct DNS server configured
- DNS server reachable
- UDP/TCP port 53 permitted as required
- DNS resolution working correctly

---

# 20. Application-Default Issue

Scenario:

An application normally uses:

TCP 443

Security Policy:

Service = application-default

Actual traffic:

TCP 8443

In this scenario, traffic is denied because it is not using the application's expected default port.

Memory Rule:

> application-default allows an application on its expected default port or ports as defined for that App-ID.

Example:

Expected:

TCP 443

Actual:

TCP 8443

Result:

Denied when it does not satisfy application-default requirements.

---

# 21. Missing Security Policy

Scenario:

Destination NAT is configured correctly:

49.204.100.50 → 192.168.50.10

But:

No matching Security Policy Allow rule exists.

Result:

Traffic is denied.

Why?

Because:

> NAT translates traffic.

> Security Policy allows or denies traffic.

A NAT rule alone does not permit traffic.

---

# 22. Missing Destination NAT

Scenario:

External customer accesses:

49.204.100.50

Actual internal server:

192.168.50.10

But no Destination NAT rule exists.

The firewall does not have the required NAT translation:

49.204.100.50 → 192.168.50.10

Therefore, the traffic is not translated to the intended internal server through that NAT rule.

Memory Rule:

> Destination NAT missing → Public destination IP is not translated to the internal server's private IP.

---

# 23. Missing Source NAT

Scenario:

Internal user:

192.168.10.25

Destination:

Internet

Security Policy:

Allow

Routing:

Correct

But:

Source NAT is missing.

The firewall still knows:

- Source IP
- Source Port
- Destination IP
- Destination Port

The problem is not that the firewall cannot see the source information.

The problem is that the private source IP remains untranslated.

Example:

192.168.10.25 → Internet

The private source IP is not normally routable across the public Internet.

Therefore, successful end-to-end communication fails.

Memory Rule:

> No Source NAT → Private source IP remains untranslated → Public Internet communication fails.

---

# 24. Advanced NAT Troubleshooting Flow

When troubleshooting Internet or NAT-related issues, check systematically.

1. Verify the user's Source IP.
2. Verify the Source Zone.
3. Verify the Destination Zone.
4. Verify the Security Policy.
5. Verify the NAT Rule.
6. Verify NAT Rule Order.
7. Check whether a general rule is above a specific rule.
8. Verify the Translated Public IP.
9. Verify Routing and Default Route.
10. Verify DNS if IP connectivity works but hostnames fail.
11. Verify Application and Service configuration.
12. Check whether application-default is appropriate for the actual port being used.

Important:

> If something has already been verified as correct, do not immediately change it. Move systematically to the next part of the traffic flow.

---

# 25. Production Troubleshooting Scenarios

## Scenario 1: Some Departments Work, Others Do Not

Finance:

192.168.10.0/24

HR:

192.168.20.0/24

NAT only covers:

192.168.10.0/24

Result:

Finance works.

HR fails.

Root Cause:

HR subnet is not covered by the required NAT rule.

---

## Scenario 2: Wrong Public IP Used

Expected:

Finance → 49.204.100.10

Actual:

Finance → 49.204.100.20

Rules:

1. Any → 49.204.100.20
2. Finance → 49.204.100.10

Root Cause:

General rule matches first.

Fix:

Move the specific Finance rule above the general Any rule.

---

## Scenario 3: NAT and Security Policy Correct, Internet Fails

Check:

Routing.

Especially:

0.0.0.0/0 → ISP Next Hop

---

## Scenario 4: IP Address Works but Domain Name Does Not

Example:

8.8.8.8 works.

google.com fails.

Check:

DNS.

---

## Scenario 5: Application Uses Wrong Port

Expected application port:

TCP 443

Actual:

TCP 8443

Service:

application-default

Result:

Traffic can be denied because the application is not using its expected default port.

---

## Scenario 6: Destination NAT Exists but Security Policy Is Missing

Result:

Traffic denied.

Reason:

NAT only translates.

Security Policy decides whether traffic is allowed or denied.

---

## Scenario 7: Source NAT Missing

Private source IP remains untranslated.

Result:

Internet communication fails because private RFC1918 addressing is not publicly routable.

---

# 26. Important Interview Questions

## Q1: What is the difference between Static NAT and Dynamic NAT?

Static NAT uses a fixed mapping.

Dynamic NAT dynamically assigns a translated public IP from a configured pool.

---

## Q2: When should Static NAT be used?

When a fixed public IP is required.

Examples:

- Public server
- IP whitelisting
- Fixed external service requirement

---

## Q3: When should Dynamic NAT be used?

When the translated public IP can change and a fixed address is not required.

---

## Q4: What is U-Turn NAT?

U-Turn NAT allows an internal user to access an internal server by using the server's public IP or public DNS name, with the firewall translating the traffic back toward the internal server.

---

## Q5: How does Palo Alto select a NAT rule?

The firewall evaluates NAT rules from top to bottom and uses the first matching rule.

---

## Q6: Why should specific rules normally be above general rules?

Because a general rule such as Any may match the traffic first and prevent a more specific rule below from being reached.

---

## Q7: NAT is correct, but Internet still fails. What should be checked?

Depending on what is already verified:

- Security Policy
- Routing
- Default Route
- DNS
- Application
- Service
- Session information

---

## Q8: User can ping 8.8.8.8 but cannot resolve google.com. What should be checked?

DNS.

---

## Q9: Can NAT alone allow traffic?

No.

NAT translates traffic.

Security Policy allows or denies traffic.

---

## Q10: What happens if Source NAT is missing for Internet-bound traffic?

The private source IP remains untranslated and is not normally routable across the public Internet, so end-to-end communication fails.

---

# 27. Golden Rules from Day 5

1. Fixed public IP required → Static NAT.

2. Public IP can change → Dynamic NAT.

3. Internal user accessing an internal server through its public IP or public DNS name → U-Turn NAT.

4. Palo Alto evaluates NAT rules top-down.

5. First Match Wins.

6. Specific rules should generally be above general rules.

7. NAT translates traffic. Security Policy allows or denies traffic.

8. NAT correct + Policy correct + Internet fails → Check routing.

9. IP works but hostname fails → Check DNS.

10. Source NAT missing → Private source IP remains untranslated.

11. Destination NAT missing → Public destination IP is not translated to the intended internal server.

12. application-default means the application must use the expected port or ports defined for that App-ID.

---

# 28. My Day 5 Learning Summary

Today I learned advanced NAT concepts and how to troubleshoot NAT-related problems.

I learned the difference between Static NAT and Dynamic NAT.

If a public IP needs to remain constant, Static NAT is appropriate.

If the translated public IP can change, Dynamic NAT can be used.

I learned U-Turn NAT, which allows internal users to access an internal server through its public IP or public DNS name.

I also learned how Palo Alto evaluates NAT rules from top to bottom using First Match Wins.

One of my most important lessons was:

> Specific rules on top. General rules below.

If a general Any rule is placed above a specific Finance or HR rule, the general rule may match first and prevent the specific rule from being reached.

I also practised troubleshooting scenarios involving:

- Missing subnets in NAT rules
- Wrong public IP translation
- Incorrect NAT rule order
- Routing failures
- DNS failures
- application-default port mismatches
- Missing Security Policies
- Missing Source NAT
- Missing Destination NAT

The most important memory lines from Day 5 are:

> NAT translates traffic. Security Policy allows or denies traffic.

> Specific rules on top. General rules below.

> IP works but hostname fails. Check DNS.

> Fixed public IP required. Use Static NAT.

> Public IP can change. Use Dynamic NAT.
