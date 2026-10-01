# Cybersecurity Fundamentals and Frameworks

## CISSP domains

CISSP Domains are "what to know": A full knowledge base to build security expertise.

| CISSP Domain                          | Summary                                                                                       |
| ------------------------------------- | --------------------------------------------------------------------------------------------- |
| Security and Risk Management          | Decide risks, set policies, and plan continuity so the business stays safe and compliant.     |
| Asset Security                        | Classify data and devices, set ownership, and handle them correctly to protect sensitivity.   |
| Security Architecture and Engineering | Design systems with built-in controls and pick secure tech and cryptography.                  |
| Communications and Network Security   | Protect data on networks using segmentation, firewalls, VPNs, and encryption.                 |
| Identity and Access Management        | Control who can access what with accounts, authentication, authorization, and reviews.        |
| Security Assessment and Testing       | Check controls with audits, scans, and pen tests, then fix the gaps you find.                 |
| Security Operations                   | Monitor and respond to incidents, manage logs, backups, and day-to-day security processes.    |
| Software Development Security         | Build, test, and deploy software securely across the SDLC with reviews and safe dependencies. |

The CISSP domains cover what security professionals need to know, not just what to do. These domains form the body of knowledge (CBK) for people working in cybersecurity. They help you understand different areas like identity access, software security, risk, and asset protection. Instead of a step-by-step process like NIST RMF, CISSP is a broad knowledge map: ideal for training, exams, and defining security responsibilities across roles and teams.

---

## Important Topics

- **Threat:** Anything that can potentially cause harm to an asset (e.g., phishing attack).
- **Vulnerability:** A weakness in a system, process, or person that can be exploited by a threat (e.g., outdated software).
- **Risk:** The likelihood that a threat will exploit a vulnerability and impact confidentiality, integrity, or availability.

> ✅ **Quick memory hook:**
> **Threat** = danger, **Vulnerability** = weakness, **Risk** = chance of danger happening because of the weakness.

---

## NIST Risk Management Framework (RMF)

NIST RMF is "how to do it": A cycle of actions to secure systems.

| Step   | Name       | What It Means                                                                                    | Focus                                                                                  |
| ------ | ---------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| Step 1 | Prepare    | Activities done in advance to manage security and privacy risks.                                 | Monitor risks and identify possible controls to reduce them.                           |
| Step 2 | Categorize | Define processes and tasks to assess risk to confidentiality, integrity, and availability (CIA). | Understand and follow processes to protect critical assets like private customer info. |
| Step 3 | Select     | Choose, tailor, and document controls that mitigate identified risks.                            | Help maintain updated playbooks and documentation for faster response.                 |
| Step 4 | Implement  | Put security and privacy plans into action across the organization.                              | Apply plans such as updating password policies or standardizing security practices.    |
| Step 5 | Assess     | Check if controls are implemented correctly and working effectively.                             | Identify weaknesses in current tools and protocols, suggest improvements.              |
| Step 6 | Authorize  | Be accountable for risk by validating and accepting the current state.                           | Create reports, action plans, and align tasks to security goals.                       |
| Step 7 | Monitor    | Continuously check and ensure systems function securely and align with goals.                    | Track daily operations, validate controls, and ensure ongoing risk remains low.        |

The NIST RMF is a step-by-step process used by organizations, especially in the U.S. government and regulated industries, to manage risks to systems and data. It guides security teams through real actions like preparing, choosing controls, implementing, testing, and monitoring them. Think of RMF as a workflow or lifecycle that helps you decide what protections to put in place and ensures they’re working. It’s very task-oriented and useful when you're actually building, assessing, or maintaining a system.

---

## More Important Topics

- **CIA Triad**: The CIA triad is a model that helps inform how organizations consider risk when setting up systems and security policies. It is made up of three elements that cybersecurity analysts and organizations work toward upholding: confidentiality, integrity, and availability. Maintaining an acceptable level of risk and ensuring systems and policies are designed with these elements in mind helps establish a successful security posture, which refers to an organization’s ability to manage its defense of critical assets and data and react to change.
  - The principle of least privilege limits users' access to only the information they need to complete work-related tasks. Limiting access is one way of maintaining the confidentiality and security of private data.
  - Another example of how an organization might implement integrity is by enabling encryption, which is the process of converting data from a readable format to an encoded format.
  - When a system adheres to both availability and confidentiality principles, data can be used when needed.

---

## NIST Cybersecurity Framework (CSF)

The six core functions that make up the CSF are: govern, identify, protect, detect, respond, and recover.

| Function     | Purpose                                                                                       | Example (Analyst Perspective)                                                              |
| ------------ | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Identify** | Understand the organization’s systems, assets, and risks to manage cybersecurity effectively. | Monitor internal systems to find risky devices or configurations before they cause damage. |
| **Protect**  | Implement safeguards like policies, training, and tools to minimize the impact of threats.    | Improve password policies or restrict USB usage to stop recurring risks.                   |
| **Detect**   | Discover cybersecurity events early through monitoring and alerting tools.                    | Check that a new SIEM tool correctly flags and prioritizes threats in real time.           |
| **Respond**  | Take appropriate action when an incident occurs to contain and resolve the threat.            | Investigate an infected device, gather evidence, and suggest process updates.              |
| **Recover**  | Restore systems and data after an incident and improve resilience.                            | Help restore financial/legal files after a breach and ensure systems are patched.          |

---

## OWASP Security Principles

Called **Open Worldwide Application Security Project® (OWASP)**.

| Principle                         | Simple Definition                                                              | Example / Application                                                             |
| --------------------------------- | ------------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| **Minimize Attack Surface**       | Reduce the number of entry points a hacker could exploit.                      | Disable unused features; restrict access; enforce stronger passwords.             |
| **Least Privilege**               | Users should only get the minimum access they need to do their job.            | Analysts can view logs, but not modify permissions.                               |
| **Defense in Depth**              | Use multiple layers of security controls to protect systems.                   | Use MFA, firewalls, IDS, and access controls together.                            |
| **Separation of Duties**          | No single person should control all parts of a critical process.               | One person prepares payroll; another approves it.                                 |
| **Keep Security Simple**          | Avoid unnecessary complexity in security systems and controls.                 | Use manageable, clear policies that teams can follow easily.                      |
| **Fix Security Issues Correctly** | Identify the root cause, patch vulnerabilities properly, and validate the fix. | Improve password policy after a weak Wi-Fi password caused a breach.              |
| **Establish Secure Defaults**     | The most secure settings should be the default in any system.                  | Default user settings deny all access until explicitly granted.                   |
| **Fail Securely**                 | Systems should default to a secure state when they fail.                       | A firewall failure should block all traffic, not allow everything.                |
| **Don’t Trust Services**          | Always validate data from third-party services or vendors.                     | Verify points from a partner loyalty system before displaying them to customers.  |
| **Avoid Security by Obscurity**   | Don’t rely on hiding code or logic as the main defense—use real controls.      | Secure systems through access policies and controls, not just hidden source code. |

---
