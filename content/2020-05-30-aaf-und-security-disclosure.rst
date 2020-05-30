Shared auEduPersonSharedToken (user ID) for all University of Notre Dame Australia users across Australian Access Federation (AAF) Resources
#############################################################################################################################################

:category: Disclosure
:tags: Security, Responsible Disclosure, Research

**Outline**: for an undisclosed period of time until 28 June 2019, all users
from `The University of Notre Dame Australia`_ (ND) accessing federated,
Shibboleth-secured SSO resources provided through the `Australian Access
Federation`_ (AAF) were issued identical `auEduPersonSharedToken`_ values.
This identifier is frequently used as a unique user identifier (username);
thus for systems where this was the case, all users from ND were considered
the same person, causing permission assigned to one user to effectively apply
to **all** members of the ND community, breaking authentication and risking
information exposure to sensitive resources.

Introduction
============

The AAF provides a federated trust network of Identity Providers (IdPs) and
Service Providers (SP) between Australia educational institutions, similar to
InCommon Federation in the USA and the UK Access Management Federation from
Jisc in the UK.

Within any of these types of federation, assurances are given that a user from
one organisation will be able to exchange attributes with another in order to
allow that user to access some type of service.  Whether the user will get
access depends on the target SP and its own access controls – it is not
guaranteed that just because I have an AAF account I should be able to log
into a given system. Services range from web-based platforms utilising
Shibboleth SSO to lower-level terminal and network services.  As with any
trust network, all participants must be trusted to provide accurate user
information, as data sent from an IdP on login to an SP will be implicitly
trusted when the SAML messages are decrypted and cryptographically verified.

The crux of this issue concerns the `auEduPersonSharedToken`_ attribute, which
is defined by the AAF as follows:

    A unique identifier enabling federation spanning services such as Grid and
    Repositories. Values of the identifier are generated using a set formula.
    The value has the following qualities:

    * unique
    * [...]
    * not re-assignable

It is these requirements that are the most important in this situation --
unique, never reassigned, and unique across the entire federation.

On this page, I look at what happened, the impact of the issue and subsequent
processes concerning security and disclosure. Lastly, I discuss suggested
improvements to processes and what system administrators may consider doing to
detect issues like that in future.

Vulnerability
=============

On the morning of 13 June 2019, I was alerted to a concern from a
non-technical system administrator of a portal I operate ("the Portal") at
James Cook University (JCU) where a given user had been able to sign in and
access secure resources without seemingly been given access.  The reporting
party was from ND and had coopted several colleagues to test logging into the
Portal and each of them was able to, without fail, obtain full-site level
access to all sensitive content on the Portal.

It is at this point, I should clarify the nature of the SSO process of this
Portal is such that a user must attempt login into the Portal first so their
identity is registered in its account system, and then the system
administrator can assign permissions. A user gets no permission by default
until granted, and the account system logs the action of logging in.

In this case, looking up one user's account by name and email address in the
Portal yielded a positive result with permissions assigned, but further
searches with other colleague names yielded no results.  A generic search for
`@nd.edu.au` confusingly yielded just the details of one staff member and
subsequent searches would yield a different ND person's name and email – a
seemingly fine example of a `heisenbug`_.

Discussions continued with the Portal's system administrators and the ND staff
members until it was ascertained that the user identifier,
`auEduPersonSharedToken`_, was identical between users. The consequence of
this is broken authentication, as all ND users would thus present themselves
as the same all-access, trusted user on the Portal, and access would continue
as this identity permanently as long the ND IdP configuration continued
functioning in this manner. When the next ND user logged in, the Portal
detected the change in email and common name and diligently updated its own
user database – as would happen with a name change say if someone were
married or legally changed their name.

At this point, the AAF support were contacted by phone call to seek their
responsible disclosure processes and alert them of the impending situation,
which was subsequently followed up with an email.

For the Portal, access was revoked to the shared ND user identity immediately
and steps were taken across JCU systems with AAF authentication to deny access
to all users presenting either that auEduPersonSharedToken or an `@nd.edu.au`
email address until such time as the issue was resolved.  All system logs and
access control databases across other systems were reviewed for use by ND
users; these systems were not designed for ND access, but may have acted as
a canary if someone with an ND account was probing systems.

Impact
======

For the Portal, this resulted in the potential for sensitive data exposure
given the nature of the Portal. Logs indicated that the given shared identity
did not attempt to access any resources (aside from the homepage) on the
system for the duration of logs held, likely attributable to the fact that
only one staff member from ND had access to the Portal and knew of its
existence.

In general, a vulnerability of this nature could easily be exploited by the
fact that University communities are open by design. As all ND users were
receiving the same ``auEduPersonSharedToken``, this meant that any student,
staff member, alumnus, guest, consultant or technician (see `ND Policy
Information Security`_) would be automatically be trusted with the same level
of permission and access that the intended ND member has on a given system. To
quote their IdP's description in the AAF `metadata.aaf.signed.complete.xml`_::

    IdP for The University of Notre Dame Australia. Available for all staff,
    students and alumni to connect to the AAF Federation.

Aside from having the entire University as potential target to locate a single
set of credentials, a bad actor could simply sign up as a (potential) student
or if physically nearby, perhaps wait until a shared computer was left
unattended.

The severity of this issue depends entirely on the target system and how the
system is configured, relying on two key points:

* A given system must have been using ``auEduPersonSharedToken`` as its user
  identifier (compared to the commonly-used ``eduPersonTargetedID`` or the
  often-misused ``mail`` attribute), and
* *Someone* at ND must have been assigned (or become assigned) a suitable
  level of elevated access

The comparatively small size of the AAF, ND and Australian higher education
communities limits the ability to exploit the issue, but anecdotal indicators
from the AAF are that the issue has persisted for an indeterminate amount of
time – it is possible that the issue has persisted since 6 July 2016, which
is the issue date of the X.509 signing certificate used by ``idp.nd.edu.au``
as found in `aaf-metadata.xml`_, assuming that the IdP configuration error
wasn't introduced later.


Resolution
==========

To the credit of the AAF, the initial report was received and acted upon
immediately, as explained in the `timeline`_ below.  Fifteen days later, on 28
June 2019, the AAF reported that the ND had changed and corrected its IdP
configuration to assign unique ``auEduPersonSharedToken`` identifiers to
users.  I conducted tests on our system in collaboration with the Portal's
users from ND and subsequently restrictions on ND user accounts were able to
be lifted.

A discussion then ensued with the AAF regarding public disclosure of the
issue, eventually reaching the point of agreement that disclosure would be
provided by ND, sent out to the AAF's standard Support Notice channels, and
seen by all IdPs and SPs. At that point in October 2019, after nearly 4
months, nothing further was heard about the issue until May 2020.  Indications
are that multiple attempts were made by the AAF to have ND provide the
advisory, but none was made and no explanation given as to why.

The final update to date from the AAF has come in 18 May 2020 with a
conclusion that getting acknowledgement of this incident from ND is unlikely.
The incident has prompted proposed changes in the AAF rules, which consist
of the following::

    Organisations will be provided a time frame in which to provide an
    advisory of issues that may impact of users privacy or security. If no
    such advisory is made by the organisation within that time frame the AAF
    will send an advisory. [...] All AAF subscribers will be notified of the
    changes to the AAF rules after they are passed by the board and members.

Summary
=======

The full extent of this issue remains unknown as no public Security Advisory
was or has been made at the time of writing by either The University of Notre
Dame Australia or the Australian Access Federation. As mentioned above, the
issue may have persisted since July 2016.

For systems under my control that utilise the AAF for login, no suspicious
behaviour was identified and the issue was mitigated as soon as it was noticed
by system users.

For other services within the AAF, I was notified that logins coming from ND
were being reviewed and contact was being made with any other affected organisations
or systems.

This issue highlights the need for better testing, review processes and
validating attributes being shared from IdPs within the AAF. A process of
validation should test attributes, ideally automatically, according to their
Federation rules.  In the case of attributes which may be used as user
identifiers, tests **must** ensure those attributes are unique across multiple
or all users at an organisation, and in the case of the
``auEduPersonSharedToken`` unique across all users in the Federation.  How
this might be achieved must also ensure user privacy is maintained.

The issue also highlights the need for better communication and security
disclosure processes from Australian higher education institutions,
particularly within those participating as members within the Australian
Access Federation.  The proposed changes to AAF rules as at 18 May 2020 to
improve processes for future Security Advisories within the AAF are welcome
and should address these concerns.

Regardless of rule changes, future systems that utilise federated login should
consider implementing an alert system to detect and report on significant or
complete attribute replacement for a given user, and consider detecting use of
multiple logins with the same identifier from different IPs or locations, even
when using a trusted set of attributes, delivered from the AAF or any similar
federation.

.. _timeline:

Disclosure Timeline
===================

* **2019-06-13**: Discovery by system administrators of an AAF-secured system;
  all access to this and other federated systems revoked for ND users
* **2019-06-13 11:53 AEST**: Responsible disclosure made to AAF Support
* **2019-06-13 14:16 AEST**: Acknowledgement from AAF that identity management team at
  Notre Dame had been contacted and were waiting a response
* **2019-06-13 15:19 AEST**: Response from AAF that issue at ND was
  identified; at this point, all users from ND were being assigned the same
  value for ``auEduPersonSharedToken``
* **2019-06-28**: Notification from AAF that ND had modified their IdP
  configuration to ensure unique auEduPersonSharedToken values were issued for
  all of their users
* **2019-06-28**: Request made to AAF as to what disclosure would occur, to
  whom and when; response from AAF that they were discussing this topic with
  ND; further request made to the AAF regarding a public advisory of the
  security issue
* **2019-07-03**: Response from AAF that AAF is organising a conversation with
  the ND ISO and CIO to ask them to notify identified service providers about
  the issue; AAF intended to to communicate a Security Incident advisory to
  IdP operators
* **2019-07-03**: Reiteration of request made to the AAF regarding a public
  advisory of the security issue; response from AAF that the IdP Security
  Incident Advisory would be published on Support notices page and sent via
  email to support notice recipients, equating to all IdPs and SPs being
  slated to see the message from the AAF
* **2019-10-10**: Follow up sent to AAF regarding the status fo the security
  advisory as one had not yet been seen
* **2019-10-11**: Response from AAF that they were working with ND to have
  them provide the advisory and that was the source of the delay
* **2019-10-11**: Request made to AAF as to an ETA for the Security Advisory;
  response received that no ETA existed at that time
* **2020-05-18**: Notification from AAF that the chances of Notre Dame
  publicly acknowledging the incident were very mote, given multiple
  attempts from the AAF. AAF Board has planned a change to AAF rules
  addressing future occurrences and all AAF subscribers will apparently be
  notified of the changes to AAF rules once passed by the board and members.
  Original security issue logged with AAF was closed.
* **2020-05-30**: Public disclosure on this page

.. _the University of Notre Dame Australia: https://www.notredame.edu.au
.. _Australian Access Federation: https://aaf.edu.au
.. _auEduPersonSharedToken: https://validator.aaf.edu.au/documentation/attributes/oid:1.3.6.1.4.1.27856.1.2.5
.. _ND Policy Information Security: https://www.notredame.edu.au/__data/assets/pdf_file/0024/38931/POLICY-Information-Security.pdf
.. _heisenbug: https://en.wikipedia.org/wiki/Heisenbug
.. _metadata.aaf.signed.complete.xml: https://ds.aaf.edu.au/distribution/metadata/metadata.aaf.signed.complete.xml
.. _aaf-metadata.xml: https://md.aaf.edu.au/aaf-metadata.xml
