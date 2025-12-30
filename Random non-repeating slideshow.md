# Random non-repeating slideshow

A non-repeating slideshow is a slideshow where every entry is shown 1 time until all files are shown. To create such a slideshow use the following code: 

```
[Rainmeter]
Update=1000
DynamicWindowSize=1

[Variables]
MyPath="C:\path\to\my\wallpaper-folder"
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
W=512
H=768
MeasureName=MeasureImageName
Path=#MyPath#
PreserveAspectRatio=2
LeftMouseUpAction=["#MyPath#[MeasureImageName]"]
DynamicVariables=1
```

`MyPath="C:\path\to\my\wallpaper-folder"` This is the folder where your files are stored.
`SecondsBetween=15` The picture changes every 15 seconds.  
`W=512`, `H=768` Dimension of the shown picture.

In Rainmeter `settings` under `position` use `in background`, so the picture is behind the taskbar.



