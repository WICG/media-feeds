
1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

This does not expose any information to web sites. The data flow is the other way around where sites expose data to the user agent.

2. Is this specification exposing the minimum amount of information necessary to power the feature?

Yes, this is the bare minimum.

3. How does this specification deal with personal information or personally-identifiable information or information derived thereof?

The recommendations that are provided by the site could be considered personal information but are only used by the user agent.

4. How does this specification deal with sensitive information?

N/A

5. Does this specification introduce new state for an origin that persists across browsing sessions?

The fetched media feed content is persisted across browsing sessions but will be cleared if a user clears their history.

6. What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

None.

7. Does this specification allow an origin access to sensors on a user’s device?

No.

8. What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

None. The origin exposes data to the user agent but not the other way around.

9. Does this specification enable new script execution/loading mechanisms?

No

10. Does this specification allow an origin to access other devices?

No

11. Does this specification allow an origin some measure of control over a user agent’s native UI?

Yes, the feature allows a site to show content from these feeds in the user agent's UI. However, a user will have the option of switching them on/off.

12. What temporary identifiers might this this specification create or expose to the web?

None.

13. How does this specification distinguish between behavior in first-party and third-party contexts?

This feature does not distinguish between these contexts.

14. How does this specification work in the context of a user agent’s Private Browsing or "incognito" mode?

We will not discover or fetch Media Feeds in these modes.

15. Does this specification have a "Security Considerations" and "Privacy Considerations" section?

No.

16. Does this specification allow downgrading default security characteristics?

No.
