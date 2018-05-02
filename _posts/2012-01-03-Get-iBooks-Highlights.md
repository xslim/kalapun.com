---
title: "How to get iBooks Hightlihts"
tags: [code, sql]
---

So for example you want to get the highligtings from iBooks, what you need is few steps:

* Get some app to mount iBooks Folder (like [PhoneDisk](http://www.macroplant.com/phonedisk/))
* Download AEAnnotation sqlite database from Documents/storeFiles to some local folder
* cd in that folder with your terminal application and execute

```
sqlite3 AEAnnotation_*_local.sqlite 'select `ZANNOTATIONSELECTEDTEXT`, `ZANNOTATIONASSETID` from `ZAEANNOTATION` where (`ZANNOTATIONSELECTEDTEXT` not NULL) order by `ZANNOTATIONASSETID`,`ZPLLOCATIONRANGESTART`'
```

 

You will see list of notes and Boor IDs to which they are associated

* Find out the Book ID you need (Example: `06CA7575F9AACCAC5027272BA8926BB1`)
* execute the command

```
 sqlite3 AEAnnotation_*_local.sqlite 'select `ZANNOTATIONSELECTEDTEXT` from `ZAEANNOTATION` where (`ZANNOTATIONASSETID` = "06CA7575F9AACCAC5027272BA8926BB1") AND (`ZANNOTATIONSELECTEDTEXT` not NULL) order by `ZPLLOCATIONRANGESTART`'
```


And you will get your highlights !

## Few dev notes

* Create Cocoa app for Exporting iBooks Highlights
* Use https://github.com/xslim/mobileDeviceManager to get data from device
* Open `BKLibrary_database/iBooks_*.sqlite` (*BKL*) and `storeFiles/AEAnnotation_*.sqlite` (*AEA*)
* Interconnection by `BKL->ZBKBOOKINFO->ZPLUGINASSETID` and `AEA->ZAEANNOTATION->ZANNOTATIONASSETID`
