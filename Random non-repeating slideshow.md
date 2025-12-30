# Random non-repeating slideshow

**Source**: https://forum.rainmeter.net/viewtopic.php?t=14947  

**Author**: Patient  
**Date**: February 21st, 2013, 9:52 pm  

---

> #### Post by Patient » February 21st, 2013, 9:52 pm
> Hi all,
> I currently use ABP slideshow to display a random slideshow of images from a folder which has about 1,080 images (and is constantly growing). I've found that the slideshow often repeats images, which is weird since there are so many in the folder. I'm looking for a way to have a simple slideshow like the one from ABP display images at random, but not repeat. There was discussion about just using a regular image display skin and letting it display them in order, but that's less appealing to me. Jsmorley mentioned using lua scripting and fileview, to which the hamster running the wheel in my head replied, "What?" That is to say, I have very little knowledge when it comes to scripting and coding. I only know enough to have modified a couple of skins, not nearly enough to create what I think is necessary to make this work right. I'm sure once it's written I'll understand it, but I make no claims of being able to solve this problem, so I bring it to you. So let's begin the brainstorm!
> Thanks.

---

> ####  by Patient » February 21st, 2013, 10:09 pm
> Jsmorley provided some FileView code as a foundation for the random image display. He stated,
> What that skin does is index all the files in a folder in [MeasureFolder]. Then it gets the "count" of files it found in [MeasureCount]. Then it creates a random number between 1 and [MeasureCount] in [MeasureRandom], then it uses the current value of [MeasureRandom] as the input to the "index" option for [MeasureImageName], which returns one random file name from the folder. We then use that in an Image meter in [MeterOne] and bingo.
> It is a FileView replacement for QuotePlugin. It more or less does the same thing as is, but is the foundation for the final solution. What we need to do is replace [MeasureRandom] with a Lua script that generates a random number, but avoids "repeats". If we just use QuotePlugin, we have NO control at all over which image is chosen. My gut reaction is to have the Lua pick a random number from 1 to [MeasureCount]. Then pass that number back to use in the [MeterImageName] measure. When a number is used by the Lua, have it put that number in a simple one-dimensional array, and next time through, check to see if the number is in the array already, if not, use it. If so, try again.

```
[Rainmeter]
Update=1000
DynamicWindowSize=1

[MeasureFolder]
Measure=Plugin
Plugin=FileView
IgnoreCount=1
ShowDotDot=0
ShowFolder=0
Path="C:\Users\Jeffrey\Wallpaper"

[MeasureCount]
Measure=Plugin
Plugin=FileView
Path=[MeasureFolder]
IgnoreCount=1
Type=FileCount

[MeasureRandom]
Measure=Calc
Formula=Random
LowBound=1
HighBound=[MeasureCount]
UpdateRandom=1
DynamicVariables=1

[MeasureImageName]
Measure=Plugin
Plugin=FileView
Path=[MeasureFolder]
Type=FileName
IgnoreCount=1
Index=[MeasureRandom]
DynamicVariables=1

[MeterOne]
Meter=Image
```

---

> #### by moshi » February 21st, 2013, 10:17 pm
> 
> so you already have your solution? because that code works just fine.

---

> by Patient » February 21st, 2013, 10:19 pm
> 
> > *moshi wrote:*  
> > so you already have your solution? because that code works just fine.
> We want a non-repeating random slideshow, meaning it only displays each image once through each cycle, so that every image in the folder is displayed over whatever length of time.

---

> #### by jsmorley » February 21st, 2013, 10:22 pm
> 
> moshi, I suspect we are going to want to use some Lua to replace [MeasureRandom], so we can avoid any "repeats" as the random numbers between 1 and the total number of files are used.
> 
> I'm chewing on it. There are some brute force ways I'm not entirely happy with, looking for something more glamorous.

---

> #### by jsmorley » February 21st, 2013, 11:51 pm
> 
> I'm not sure this is the most efficient way, but it does seem to work:
> 
> RandomNoRepeat.ini:

```
[Rainmeter]
Update=1000
DynamicWindowSize=1

[Variables]
MyPath="C:\Users\Jeffrey\Wallpaper\"
SecondsBetween=15

[MeasureFolder]
Measure=Plugin
Plugin=FileView
ShowDotDot=0
ShowFolder=0
Path=#MyPath#
Extensions=png;jpg;bmp
FinishAction=[!EnableMeasure MeasureRandom]

[MeasureCount]
Measure=Plugin
Plugin=FileView
Path=[MeasureFolder]
Type=FileCount

[MeasureRandom]
Measure=Script
ScriptFile=RandomNoRepeat.lua
Disabled=1
UpdateDivider=#SecondsBetween#

[MeasureImageName]
Measure=Plugin
Plugin=FileView
Path=[MeasureFolder]
Type=FileName
IgnoreCount=1
Index=[MeasureRandom]
DynamicVariables=1

[MeterImage]
Meter=Image
W=200
H=125
MeasureName=MeasureImageName
Path=#MyPath#
PreserveAspectRatio=2
LeftMouseUpAction=["#MyPath#[MeasureImageName]"]
DynamicVariables=1
```

> RandomNoRepeat.lua:

```
function Initialize()

	msMeasureCount = SKIN:GetMeasure("MeasureCount")
	
	usedArray = {}
	math.randomseed( tonumber(tostring(os.time()):reverse():sub(1,6)) )
	
end

function Update()

	fileCount = msMeasureCount:GetValue()
	
	fileNum = math.random(1, fileCount)
	while table.contains(usedArray, fileNum) == true do
		fileNum = math.random(1, fileCount)
	end
	table.insert(usedArray, fileNum)
	
	if #usedArray == fileCount then
		usedArray = {}
		math.randomseed( tonumber(tostring(os.time()):reverse():sub(1,6)) )
	end

	return fileNum
	
end	

function table.contains(table, element)

	for _, value in pairs(table) do
		if value == element then
			return true
		end
	end
	return false
	
end
```

> What it is doing is using FileView to get all the files matching the extensions in the folder. Then it is getting a total "count" of files that the Lua can use for generating a random number.
> 
> Then the Lua is generating a random number from 1 to that "count". It is checking in an array to see if that number has been used before. If it has, it generates a new random number until it is NOT found in the array, then adds it to the array and returns the number to the skin.
> 
> The skin then uses that number as the "index" for another FileView measure that gets the file name, and the image meter displays it.
> 
> When all numbers up to "count" have been used, the array is cleared and it starts over.
> 
> There may be a more glamorous way in the Lua code, but this should get you going.

---

> #### by Patient » February 22nd, 2013, 1:07 am
> 
> Thank you to jsmorley for resolving this for me! I can attest that this works very well.

---

> #### by bluemondaydesign » January 15th, 2016, 8:00 am
> 
> Hi jsmorley.
> 
> This code has worked perfectly for me for quite sometime now but I've struggled to add extras to it.
> 
> There are two things I'm trying to get it to do:
> 
> Open In Photo Viewer
> I'm trying to do is get the image to open in the default viewing app (like Windows Photo Viewer) using the "LeftMouseUpAction" feature. I just can't get it to work no matter what using this "non-repeating slideshow" code. It works as expected on all the typical slideshows, but with this code, it just won't work. I can get it to open the folder in Windows Explorer which is ok, but I'd rather just open the image itself. :?
> 
> Pull Images from Subolders
> I'd also like it to use images inside subfolders. ShowFolder nor ShowDotDot worked... etc.
> 
> Any suggestions? I've spent hours and hours working on this on numerous different occasions and now, I'm throwing in the towel and asking for help LOL.
> 
> This code still seems to be the best in creating an actual non-repeating slideshow so I'd like to keep using it. Thanks in advance.

---

> #### by FreeRaider » January 15th, 2016, 11:15 am
> 
> > *bluemondaydesign wrote:*  
> > Pull Images from Subolders
> > I'd also like it to use images inside subfolders. ShowFolder nor ShowDotDot worked... etc.
> I use quote plugin for showing images from subfolders.
> 
> You can try to change your skin to use it.

---

> #### by jsmorley » January 15th, 2016, 11:52 am
> 
> bluemondaydesign,
> 
> Pull images from subfolders
> 
> I think you just need to add Recursive=2 to the FileView measure.
> 
> https://docs.rainmeter.net/manual/plugins/fileview/#Recursive
> 
> Do NOT use QuotePlugin, as that won't support the entire point of this thread - "non-repeating". If you don't care about non-repeating, then my code is just overkill in every respect and this discussion should move to a new / different thread.
> 
> Try this code:
> 
> RandomImage.ini

```
[Rainmeter]
Update=1000

[Metadata]
Name=RandomImage
Author=JSMorley
Information=Displays a random non-repeating image from folder tree defined in [Variables] ImagePath.
License=Creative Commons Attribution-Non-Commercial-Share Alike 3.0
Version=Mar 21, 2013

[Variables]
ImagePath=%USERPROFILE%\Pictures
SecondsBetween=30

[MeasureFolder]
Measure=Plugin
Plugin=FileView
Recursive=2
Path=#ImagePath#
Extensions=jpg;png;bmp
FinishAction=[!EnableMeasure MeasureRandom][!ShowMeter MeterBackground]

[MeasureCount]
Measure=Plugin
Plugin=FileView
Path=[MeasureFolder]
Type=FileCount

[MeasureRandom]
Measure=Script
ScriptFile=RandomImage.lua
Disabled=1
UpdateDivider=#SecondsBetween#

[MeasureImagePath]
Measure=Plugin
Plugin=FileView
Path=[MeasureFolder]
Type=FilePath
IgnoreCount=1
Index=[MeasureRandom]
DynamicVariables=1

[MeterBackground]
Meter=Image
W=210
H=135
SolidColor=60,60,60,170
Hidden=1

[MeterImage]
Meter=Image
W=200
H=125
X=5
Y=5
MeasureName=MeasureImagePath
PreserveAspectRatio=1
TooltipText=[MeasureImagePath]
LeftMouseUpAction=["[MeasureImagePath]"]
DynamicVariables=1
```

> RandomImage.Lua:

```
function Initialize()

	msMeasureCount = SKIN:GetMeasure("MeasureCount")
	currentTable = {}
	
end -- Initialize

function Update()

	fileCount = msMeasureCount:GetValue()
	
	if #currentTable == 0 then
		currentTable = RandomOrder(fileCount)
	end
	
	return table.remove(currentTable)
	
end	-- Update

function RandomOrder(inputCount)

	assert(type(inputCount) == 'number', ('RandomOrder must receive a number. Received %s instead.'):format(type(inputCount)))

	local sortedTable, unsortedTable = {}, {}
	
	for inc = 1, inputCount do
		unsortedTable[inc] = inc
	end
	
	math.randomseed(tonumber(tostring(os.time()):reverse():sub(1,6)))

	while #unsortedTable > 0 do
		table.insert(sortedTable, table.remove(unsortedTable, math.random(1, #unsortedTable)))
	end
	
	unsortedTable = nil
	
	return sortedTable	
	
end -- RandomOrder	
```

> That should both recurse into sub-folders, and display the image in the default image viewer when clicked on. Does for me...

---

> #### by jsmorley » January 15th, 2016, 1:10 pm
> 
> > FreeRaider wrote:
> > I use quote plugin for showing images from subfolders.
> FreeRaider, to be clear:
> 
> QuotePlugin is "random" in the sense that throwing a pair of dice is random. Some random number between 2-12 will come up, but on each throw ANY number between 2-12 is possible. Including two 5's three times in a row for instance, and you go to Jail, do not pass Go, do not collect $200...
> 
> What the OP is / was looking for is "random" in the sense of drawing a card from a shuffled deck. Once you have drawn a card, it's no longer in the deck, and while the next draw is still random, it is now based on 1-51, instead of 1-52. Once you have drawn the Ace of Spades, you won't get it again until all cards have been drawn, you re-shuffle the deck, and start over.
> 
> That is why QuotePlugin is terrible for image slideshows. (in fact it's just terrible in general) Unless you are drawing from a pool of thousands of items (quotes from a file, images, whatever) where statistically the likelihood of repeats gets to an acceptably low point, QuotePlugin will just produce annoying results. It makes my teeth itch to see an image or pithy quote come up that just came up 3 or 4 updates ago. That is just not how I want an image slideshow or random quote to work.

---

> #### by bluemondaydesign » January 15th, 2016, 4:58 pm
> 
> YES!! This works perfectly and both my goals were accomplished in this newer version of code. Now to start dissecting it and figure out how this works.
> 
> Thank you so much jsmorley!

---

> #### by jsmorley » January 15th, 2016, 5:17 pm
> 
> Glad to help.

---

> #### by FreeRaider » January 15th, 2016, 9:27 pm
> 
> > *jsmorley wrote:*  
> > FreeRaider, to be clear:
> > 
> > QuotePlugin is "random" in the sense that throwing a pair of dice is random. Some random number between 2-12 will come up, but on each throw ANY number between 2-12 is possible. Including two 5's three times in a row for instance, and you go to Jail, do not pass Go, do not collect $200...
> > 
> > What the OP is / was looking for is "random" in the sense of drawing a card from a shuffled deck. Once you have drawn a card, it's no longer in the deck, and while the next draw is still random, it is now based on 1-51, instead of 1-52. Once you have drawn the Ace of Spades, you won't get it again until all cards have been drawn, you re-shuffle the deck, and start over.
> > 
> > That is why QuotePlugin is terrible for image slideshows. (in fact it's just terrible in general) Unless you are drawing from a pool of thousands of items (quotes from a file, images, whatever) where statistically the likelihood of repeats gets to an acceptably low point, QuotePlugin will just produce annoying results. It makes my teeth itch to see an image or pithy quote come up that just came up 3 or 4 updates ago. That is just not how I want an image slideshow or random quote to work.
> 
> Thanks jsmorley, you taught me another thing.
> I like your code and I think I'll use it in place of quote plugin.
