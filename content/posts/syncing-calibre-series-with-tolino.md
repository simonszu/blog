---
title: "Syncing Calibre Series With Tolino"
date: 2023-09-12T17:19:57+02:00
draft: false
---

I am an eBook user and love my Tolino Shine for reading. For management and sorting of the eBooks on my desktop computer i am using [Calibre](https://calibre-ebook.com/) - quelle surprise!

Since my inner Monk kicks always in when it comes to consistency and sorting, i have tagged my eBooks with a proper Series metadata, e.g. _Harry Potter_ or _Lord of the Rings_. My Tolino has a "Collection" feature, where it can group eBooks by certain criteria. Naturally i wanted to show up my proper tagged series as collections. After some digging in the internet and experimenting with various settings, i have found the following solution: 

In the file system which shows up when the Tolino is connected to the computer, there is a folder called "Books" which should contain all books. The Tolino firmware monitors this folder for changes and updates its internal library accordingly, based on the metadata of the books inside this folder. However, the Collections are _not_ built based on these metadata, but rather on a folder structure. Each subfolder inside the Books folder is deemed as a collection. Books which are not in a subfolder are not part of any collection, but still present on the device, findable via Author or Title. The subfolders cannot be nested, all books are added to the collection which is named by the first subfolder, so for example a book in the path `SciFi/Star Wars/Episode 1.epub` is added to the same "SciFi" collection as a book in the path `SciFi/Star Trek/Warp Core Manual.epub`. So, you have to decide if you want to create Collections based on Genres, Tags or Series. I like them ordered as series. 

But how to advise Calibre to create these folder structure? Open its preferences, and select "Sending books to devices" under "Import/export". Here you can create a saving template which tells Calibre what path and file name it should create on the device. I chose the following template: `{series:re(&,u)}/{series_index:0>2s} {title:re(&,u)} - {author:list_item(0,&)}`

This will cause a folder structure like `/Seriesname/Number Booktitle - Author`, and if no series are defined `/ Booktitle - Author`. The series number as the prefix of the actual filename causes the books to appear sorted in the Collection view. 

All what was to do afterwards was to completely remove all books from the device, re-add them, and let the device re-sync its library content. Afterwards, all "old" collections where still present but empty and needed to be deleted one by one - but the "new" collections appeared as desired, with proper book order and everything. 