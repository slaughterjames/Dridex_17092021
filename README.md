# Dridex_17092021

Today I happened upon a fresh Dridex dropper sample from [@Slvlombardo](https://twitter.com/Slvlombardo/status/1438494754489110530).  It uses a technique that I haven't personally seen previously, effectively placing the code that would have been inside a macro inside of an XML file within the container structure of the file.  This is reminiscent of some of the stuff we've seen around CVE-2021-40444.  Below is a short analysis.

**File Details:**
|--------> MD5 hash of file INV.-0456_20210915.xlsm: 8694d363cdc926109266f41758663c28
|--------> SHA1 hash of file INV.-0456_20210915.xlsm: 2dfc06e6e815cf9e18085d043b13b5d9bdf44eaa
|--------> SHA256 hash of file INV.-0456_20210915.xlsm: 84aac3d5194fcf601596af4385093ff5bf3d1fd6ece335f53b8e0a785c156ccf
|--------> VirusTotal: https://www.virustotal.com/gui/file/84aac3d5194fcf601596af4385093ff5bf3d1fd6ece335f53b8e0a785c156ccf/detection

**Lure**
The file if opened will show a familiar message asking the user to enable macros and content thereby allowing the malicious content to run.

![alt text](https://github.com/slaughterjames/Dridex_17092021/blob/main/lure.png)

**No Macros:**
Straight away, the first thing noticed if you check for macros is that there aren't any.
![alt text](https://github.com/slaughterjames/Dridex_17092021/blob/main/olevba.png)

**Digging Deeper:**
In order to make progress on this, the file (effectively a fancy Zip archive) needs to be expanded to look at what's under the hood. 

![alt text](https://github.com/slaughterjames/Dridex_17092021/blob/main/Structure.png)

**XML:**
Inside the constituent parts of the Excel document is an XML file that is quite interesting!  Sheet1.xml shows us that not only is there a hidden sheet in the Excel document, "Macro1", but also that the code typically stored in an Excel macro is instead here!

![alt text](https://github.com/slaughterjames/Dridex_17092021/blob/main/xml.png)

From this code, we can see that we're interested in a couple of notes (Macro1!$B$231) and (GET.NOTE(Macro1!$B$356, 1, 200)).  This corresponds to:

![alt text](https://github.com/slaughterjames/Dridex_17092021/blob/main/writefile.png)

and:

![alt text](https://github.com/slaughterjames/Dridex_17092021/blob/main/command.png)

This is going to be the output file name of the next stage in the infection process and the command to execute it.

In order to get to that next stage, we'll need to be interested in the cell range(Sheet1!E165:T1762).  I've created a small Python script to do the hard work of outputting the requisite HTA file.  Pull dridex_xls_17092021.py, controller.py and fileio.py down and run it against a file from this campaign.

**HTA File:**
To save you a bit of suspense, I've included an image of the output below:

![alt text](https://github.com/slaughterjames/Dridex_17092021/blob/main/badfile1.png)

and:

![alt text](https://github.com/slaughterjames/Dridex_17092021/blob/main/badfile2.png)

You'll note that these are the download locations for the next infection stage.

hXXps://cdn[.]discordapp[.]com/attachments/887666017042587651/887666130376884274/9_SensorsApi.dll.dll

hXXps://cdn[.]discordapp[.]com/attachments/887666439396421665/887666488083873842/0_nfswmiprov.dll.dll

hXXps://cdn[.]discordapp[.]com/attachments/887666439396421665/887666501413380136/7_kyw7sr03.dll.dll

hXXps://cdn[.]discordapp[.]com/attachments/887666439396421665/887666498326384670/4_GrooveAudio.dll.dll

Thanks for reading this far!  Hope you've enjoyed this whistlestop tour of this Dridex dropper! 
