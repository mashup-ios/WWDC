# [Writing Great Alerts](https://developer.apple.com/videos/play/wwdc2017/813/)

@ WWDC 17

### When should alerts be used?

Good example > correct errors

```
Verification Failed
Your Apple ID or password is incorrect.
```

Alerts also do request access, notify significant feature releases or critical security fixes.



### Alerts are disruptive by design

Do not use alerts for non-essential content

Ask yourself, do people really need to know this?



### Alerts are lightweight by design

Do not overload alerts with lengthy content, or too many choices.

Do not use alerts to expose technical data, especially error codes.

Alerts are not afterthoughts. Do not use them to fix preventable problems.



### Alerts Dos and Donts

* Dos
  * Correct errors
  * Request access to user data
  * Notify users about critical updates
* Donts
  * Don't use nonessential or lengthy content
  * Don't offer complicated choices
  * Don't duplicate the task of other UI elements



### Let's See Good Example

* Title: one sentence (What?)
* Message: brief additional information, if necessary (Why?)
* Actions: easy to understand and perform (How?)



### Style and Quality

* Avoid platitudes, jargon, or filters
* Use correct formatting, spelling, and punctuation
* Consider your audience



### Summary

* Keep it simple and helpful
* Practice: revise your alerts to incorporate these guidelines
* Read the HIG