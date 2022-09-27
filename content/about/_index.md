---
Title: About
layout: "stub"
---

``` bash
travis@thinkpad> cat aboutMe.py
from graph import Graph

def aboutMe(self):
    endpoint = '/me'
    select = 'displayName,givenName,surname,mail,jobTitle,officeLocation,cats,interests'
    request_url = f'{endpoint}?$select={select}'

    user_response = self.user_client.get(request_url)
    return user_response.json()

aboutMe()

travis@thinkpad> python3 aboutMe.py
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "displayName": "Travis Baraki",
    "givenName": "Travis",
    "surname": "Baraki",
    "jobTitle": "Security Engineer, IAM",
    "mail": "travis@tbaraki.net",
    "officeLocation": "California - Remote",
    "cats": 2,
    "interests": ['Azure AD', 'Okta', 'IAM', 'Autopilot', 'Intune', 'PowerShell']
}
```