---
title: "Userscripts and Apple dev site"
tags: [javascript,code,apple]
---

If you are a pro developer for Apple stuff (ios, mac), you may have a situation, when you are in different teams, and sometimes, the team name can be same. For example, there can be a team *Company* for AppStore and team *Company* for Enterprise. So I was bored gessing which one to choose and decided to make a *Greesemonkey* script for that.

![](/images/2012-07-13/team_select.png)



## Usage
If you are using Safari, install small extension NinjaKit, for adding support of greesemonkey scripts

* [https://github.com/os0x/NinjaKit](https://github.com/os0x/NinjaKit)

Now point your browser to my script on userscripts.org and install it

* [http://userscripts.org/scripts/show/138339](http://userscripts.org/scripts/show/138339)

## The Code
Well, if you wanna know the code behind the script, here you go:

``` javascript
// ==UserScript==
// @name        Apple Developer Team select Rename
// @description Changes Team name by adding Team ID
// @author      Taras Kalapun <t.kalapun@gmail.com>
// @include     http*://developer.apple.com/devcenter/selectTeam*
// ==/UserScript==
teams = document.getElementById("teams")
for (var i=0; i<teams.length; i++){
  teams.options[i].text = teams.options[i].text+" ("+teams.options[i].value+")"
}
```
