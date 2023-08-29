---
title: "A Sane Way to Handle Zettelkasten Ids in Obsidian"
date: 2022-02-14T09:12:34+02:00
draft: false
---

I am using [Obsidian](https://obsidian.md) as my personal knowledge management/external brain tool. In the past with [PARA](https://www.der-generalist.de/para/) as the way to organize notes, which was not really satisfying. I ended up with lots of subfolders and always struggled with the question where to sort notes in. 

So, a plain folder structure was the way to go. Everything inside, and organize with the link/graph which is one of Obsidian's key feature. Only problem arousing from that? Duplicate file names.

I have some notes where i keep some information how to access customer's environment. SSH tunnels, Proxy settings, and so on. Currently they are in subfolders, like `Customer1/Access.md` and `Customer2/Access.md`. In a flat file structure, two files called Access.md would appear in one folder, which is impossible on most modern file systems. Prefixing them with some category like `customer1-access.md` was not feasible for me, since then i could as well put them back into folders.

So i went with the apparently common approach, and add some UID ("Zettelkasten ID") to the file name, which is basically just the current timestamp, for example `202202231530 Access.md`. The other file would have another timestamp, and therefore a different UID. Both files are still distinguishable via their content, tags, links to other notes, and so on.

The UID on new notes created with the "New note" button can be automatically created with the "Zettelkasten Prefixer" plugin from Obsidian's community plugin repo. However, this leaves me with the next problem: How can i create a note from an empty link in another file? I do not want to add the current timestamp manually - in generally i do _not_ want to work like Obsidian tells me to do, instead i want Obsidian to work like i tell it to. At last, Computers are required to help you with your workflow and should be able to adapt to it, aren't they?

So i removed the Zettelkasten Prefixer plugin again, installed the Templater plugin and was able to write a template with the help of some Javascript. It looks like this (i'll explain it later):

{{< highlight javascript >}}
<%*
if (tp.file.title == "Untitled")  {
	// Wenn die Datei gerade neu erzeugt wurde, hat sie noch keinen definierten Titel, er muss nachgefragt werden
	var noteTitle = await tp.system.prompt("Titel:")
} else {
	// Wenn die Datei aus einem Link erzeugt wurde, hat sie den Linknamen als Dateinamen
	var noteTitle = tp.file.title
}
fileName = noteTitle+ " ("+tp.date.now("YYYYMMDDHHmm")+")"
await tp.file.rename(fileName)
-%>

---
Created: <% tp.file.creation_date() %>
aliases: [<% noteTitle %>]
tags: 
---
# <% noteTitle %>



---

Modified: <%+ tp.file.last_modified_date("YYYY-MM-DD HH:mm") %>
{{< / highlight >}}

I defined this template as a folder template in Templater, so that it gets applied to every new note which is created in the flat folder which should contain all my notes. It does the following:

- If a note is created from the "New Note" button, it automatically gets the title "Untitled" from Obsidian. The template kicks in, and displays a text box, asking me for the actual title of the note.
- If a note is created from a link in another note, it already has a title, which is the text of the link.

The template will add the note title as an alias to the YAML front matter, so that this note is still search- and linkable via its, lets say, human readable format. This is useful for Obsidian's detection of possible incoming and outgoing links. It will then add the current timestamp to the file name of the note. Obsidian itself will notice that the file was renamed, and fix the link from where the note was created (if needed).

You may have noticed that contrary to the common usage, i did not use the UID as a prefix, but as a suffix in parentheses. In my opinion this increases the readability of the note filename, for example in internal link titles or in the graph view. Only downside: I do not have a chronologic view of the filenames in my explorer view, but why should i need it? Knowing when a note was created is not so relevant for me that i need to see it at first glance, and the notes do not have any timely coherence to the "neighboured" notes in the explorer view.