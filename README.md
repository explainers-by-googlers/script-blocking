# Mitigating API Misuse for Browser Re-Identification

This proposal is an early design sketch by Chrome to describe the problem below and solicit
feedback on the proposed solution. It has not been approved to ship in Chrome.

This is a proposal for a Chrome feature in Incognito mode that will block the execution of known, prevalent techniques for browser re-identification used in third-party contexts. These techniques generally involve the use of existing browser APIs in ways that do not match the API's intended purpose and are designed to extract additional information about the user's browser or device characteristics. These techniques have been extensively studied by the academic community[^1], highlighting their associated privacy risks.  

## Authors

- [James Bradley](https://github.com/jbradl11)
- [Mike West](https://github.com/mikewest)
- [Zainab Rizvi](https://github.com/zainabaq)

## Participate
- https://github.com/explainers-by-googlers/script-blocking/issues

## Detection & Blocklist Generation

Chrome has developed a methodology to identify widely used JavaScript functions that provide consistent outputs from stable and high-entropy web APIs and can therefore be used to construct probabilistically high-entropy identifiers. For example, one such function might involve using Canvas API to render an image with very concrete characteristics that has been designed to extract as much entropy as possible about the device's graphics card.

Once these signatures have been identified, Chrome crawls the web to look for matches of this code pattern and generate a list of domains that serve these scripts. The list undergoes some additional treatments before it's ready to be applied in Incognito: 

*   Shared domains (e.g. CDNs): scripts may be served from shared domains, for example, a CDN domain shared by many of its clients. We calculate the proportion of a host's traffic that is serving one of the identified scripts, and if it is less than a certain threshold, we consider it a shared domain. In that case, the feature will only apply to a specific path of that host, instead of applying at the domain level.
*   Third-party context only: we only include scripts that are served from a third-party context in Incognito. To determine first-party vs third-party, Chrome will employ a best-effort approach to deduce domain ownership, for example by leveraging an entity mapping created by [Disconnect.me](https://disconnect.me/). Resources served by domains in the same entity mapping will be treated as first-party. Additionally, if a resource’s domain matches the top-level domain, it will also be considered first-party. In the event our deduced approach contains errors, the domain owner has the option to reach out to [Disconnect.me](http://Disconnect.me) at mdl_inquiries@disconnect.me. 
*   Truncation: to improve performance, we drop rarely seen domains from the list. The resulting list maintains 95%+ of the protective value for users but is orders of magnitude smaller in size.
*   Exceptions for web compatibility: we may apply temporary exceptions if we determine that the intervention on a particular domain may cause significant user experience impact, degradation on the site's anti-fraud defenses or for particularly sensitive sites such as .gov and .edu. These exceptions could be subject to change, particularly as adoption of privacy-preserving alternatives grows and the alternatives themselves evolve to meet these use cases.

## Intervention in Chrome

When the feature is enabled, Chrome will check network requests against the blocklist, including detecting CNAME aliases. When there is a match, active content from those domains will be blocked (e.g., scripts, iframes), but not static resources (e.g., images, stylesheets). Note that all active content is blocked for the domain (or the path in the case of shared domains), not just detected scripts. This prevents simple evasion tactics such as renaming or slightly modifying the script. 

## Publication of the Blocked Domain List

The list of domains impacted by the feature is available on [GitHub](https://github.com/GoogleChrome/ip-protection/blob/master/Masked-Domain-List.md), and it is a subset of the Masked Domain List defined for [IP Protection](https://github.com/GoogleChrome/ip-protection/tree/main). This feature will affect entries marked “Impacted by script blocking". Periodically, domains may be added or removed from the list. Chrome will also remove domains that have successfully obtained an appeal. 

## Appeals

We recognize the importance of implementing an appeals process for our list-based approach. Appeals permit companies to make a claim that their domain on the list does not meet the inclusion criteria and ought to be removed, thereby allowing that domain to continue to operate normally in a third-party context in Incognito. 

The appeals process is available now to provide domain owners sufficient time to seek an appeal and receive a decision prior to the launch of Script Blocking in Incognito in Chrome Stable.

[Disconnect.me](http://Disconnect.me) will independently manage and operate the appeals process for the list. All decisions regarding a domain’s appeal are based solely on the MDL [criteria outlined in the IP Protection Explainer](https://github.com/GoogleChrome/ip-protection/tree/main?tab=readme-ov-file#the-masked-domain-list-criteria).

The [appeals process](https://github.com/GoogleChrome/ip-protection?tab=readme-ov-file#appeals) will follow the same guidelines and details as IP Protection, which are further explained in the IP Protection explainer.

## Availability of the feature

The feature will be available for users in Chrome’s Incognito mode only, on Android and Desktop platforms. It will be on by default in Incognito. Users will have the ability to disable it. For enterprise-managed versions of Chrome, the feature can be enabled in Incognito, but it will be off by default to ensure that enterprise site compatibility and their user workflows are not impacted. 

## An ever evolving landscape

Given the dynamic nature of tracking techniques, user agents will want to continuously research and evolve their solutions to offer the best possible protection against new and emerging scaled techniques.   Guided by insights from the [Identifiability Study](https://doi.org/10.1145/3589335.3648322), Chrome is prioritizing countermeasures against prevalent and high-impact techniques. Looking forward, we expect to continue to evolve and expand the set of techniques that Chrome detects, and to strengthen the protection against potential circumvention. 

## Ecosystem engagement

We welcome feedback on this proposal from the ecosystem. This feedback will be considered for refining our proposal and shaping the future roadmap for Mitigating API Misuse for Browser Re-Identification.

[^1]: Enrico Bacis, Igor Bilogrevic, Robert Busa-Fekete, Asanka Herath, Antonio Sartori, and Umar Syed. 2024. Assessing Web Fingerprinting Risk. In Companion Proceedings of the ACM on Web Conference 2024 (WWW '24). Association for Computing Machinery, New York, NY, USA, 245–254. https://doi.org/10.1145/3589335.3648322
