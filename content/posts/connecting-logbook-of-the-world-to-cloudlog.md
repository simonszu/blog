---
title: "Connecting Logbook of the World to Cloudlog"
date: 2023-02-01T09:20:51+02:00
draft: false
---

I am using the excellent [Cloudlog](https://github.com/magicbug/Cloudlog) to log my ham QSOs. Cloudlog has built-in functionality to upload QSOs to electronic QSL services like [eQSL](https://eqsl.cc/qslcard/Index.cfm), [qrz.com](https://www.qrz.com/) and [Logbook of the World](https://lotw.arrl.org/lotwuser/default).

While connecting Cloudlog to the former both services was as easy as entering my username and password, the connection to LotW was a bit more complicated due to some more validation steps.

At first i struggled with the actual creation of my LotW account. As a ham who is not located in the U.S., this is a bit more complex. Since QSLs in LotW are digitally signed, you need its companion tool called TQSL.

[Download it](https://lotw.arrl.org/lotw-help/installation/) and start it. In the top menu bar, click on `Callsign Certificate` and then on `Request New Callsign Certificate`. Enter your callsign. As the begin date enter the date of the first QSO you want to log with LotW.  Chose `Create a new LotW account`. Let TQSL upload the certificate request, do _not_ delete it.

The ARRL now has the request somewhere on their servers. However, they need to validate that you are a licensed ham radio operator. Make a photo of your license, together with a governmental issued ID, like your passport or your driver's license. Mail this photo and your callsign to the ARRL via `hq@arrl.org`.

After a couple of days you will get your certificate as a TQ6-file, together with the username and password of your new LotW account. Double-click on the TQ6-file to import it to TQSL and save it to a secure location, e.g. as a file attachment to your password manager.

Now, to Cloudlog. In TQSL, go to the `Callsign Certificates` tab. Right click on your callsign, and select `Save Callsign Certificate File`. Do not set a password. You will get a P12-file. In cloudlog, open your account settings, and provide your LotW username and password. Open the dropdown-field via your Callsign in the top right corner again, and select the Logbook of the World sync option. Upload your certificate. It should show as `Valid / Not synced`.

In theory, your LotW connection from Cloudlog should be set up now. However, there are some pitfalls, which are not so well communicated by the Cloudlog UI. You will most probably only get an error: `LoTW Downloading failed either due to it being down or incorrect logins`.

In my case, there were two things i did not had in mind. At first: **Set your right DXCC entity**. This is in regards to your station QTH. When i scrolled through the list, not yet having an idea what a DXCC entity is, i selected _Germany_, since i am located in Germany. Unknown to me, this is not the right DXCC entity, but a deprecated one. As a german operator, please select _Federal Republic of Germany_.

Your QSO contacts will set the QSO details to Federal Republic of Germany, and LotW would not match the QSO, or even deny uploading QSO details with deprecated DXCC entity. After having corrected the DXCC entity, i was able to upload QSOs. However, i was still getting the error when trying to download.

This is because LotW only displayed a confirmed QSO when both partners have uploaded their details and LotW detects a match. If only one of them has uploaded, they will have nothing to download. This differs from the more paper-like QSL behavior of eQSL.

And, if you are a first-time user: Even if your partners have already their QSO details to LotW, you would not immediately be able to download the QSL via cloudlog after you have uploaded your details, since the details are submitted to a processing queue. Only when this queue is proceeded by the LotW systems, your QSO is confirmed and Cloudlog is able to download it. This can take some hours, especially when a big contest is going on.

Hopefully you will stumble upon this blogpost if you currently have the same problem as i had, since the Cloudlog error message is not so descriptive, especially when you are not yet aware of the bidirectional confirmation constraint which LotW has. :)