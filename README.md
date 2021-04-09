# iosScreenCalc
Basic Python code to calculate the screen dimensions of iOS (iPhone and iPad) device displays in pixels

A Basic design would be to collect the iOS model identifier and then perform a lookup of the screen size. This is helpful if your doing wallpapers or other resizing of graphics

```
#!/usr/bin/env python3

import os
import csv
import hashlib
import shutil

# use the following to test the lookup
# or add
# screenCalc(searchString) at the bottom of the file, alter the search string and look for 
# Your model i.e. iPad,Nimbus2000
# Just make sure you add the model into the tsv to differentiate between a local lookup and the external lookup.
# searchString = "iPad1,1"

prefsFolder = os.path.join(os.path.dirname(os.path.realpath(__file__)), "prefs")
prefsFile= os.path.join(os.path.dirname(os.path.realpath(prefsFolder)), "screenCalc.tsv")

def screenCal(model):
	if os.path.exists(prefsFile):
		with open(prefsFile) as infile:
			for line in infile:
				if model in line:
					print('  '.join(line.split()))
					Model, x, y = line.split()
		return x, y
	elif not os.path.exists(prefsFile):
		if model == "iPad1,1":
			y = 1024
			x = 768
		if model == "iPad2,1":
			y = 1024
			x = 768
    	#..... continue for list of iOS devices hard coded to return an x and y value
  	return x,y

#Testing
#x, y = screenCal(searchString)
#
#if int(x) >= 0:
#	print("X is " + x)
#	print("Y is " + y)
#else:
#	print("X is not right")
```
screenCalc.tsv (Tab Separated value) file added as CSV would make matching the iOS model itentifier horrible as Apple included a comma in the model name, so just made the file a TSV to make it easy to parse

screenCalcTSV_MD5.txt - md5 Hash of the file to validate the download, security and all that.

screenCalc.py - original process download a raw python file that can simple be updated... buuut, if your are using pyinstaller to make an executable package it would include the python code and never update, so rather than do this, we moved to a tsv file.

iosScreenCalc-MD5.py - md5 hash of the screenCalc.py file for security of your downloads.

One method we have utilised is to create an auto update policy for any platform using pyinstaller, then use cron, launchd or windows task scheduler to run the python app when desired to check for updated iOS devices.

We used this exact process and an MD5 check of the file using 

```
###### Temp Path and rotation ####
localTMPdir = os.path.join(os.path.dirname(os.path.realpath(__file__)), "tmp")

if not os.path.exists(localTMPdir):
	os.makedirs(localTMPdir)
##################################

# Paths to identify directories to be used
tmpMD5 = os.path.join(localTMPdir, "screenCalc_md5.txt")
tmpScreenCalc = os.path.join(localTMPdir, "screenCalc.tsv")
updatesDir = os.path.join(os.path.dirname(os.path.realpath(__file__)), "prefs")
MainScreenCalc = os.path.join(updatesDir, "screenCalc.tsv")

# Ignore SSL Cert, as proxy may step in the way
ssl._create_default_https_context = ssl._create_unverified_context
# Download and Save the file
urllib.request.urlretrieve("https://raw.githubusercontent.com/D8Services/iosScreenCalc/main/screenCalcTSV_MD5.txt", tmpMD5)

# Ignore SSL Cert
ssl._create_default_https_context = ssl._create_unverified_context
# Download and Save the file
urllib.request.urlretrieve("https://raw.githubusercontent.com/D8Services/iosScreenCalc/main/screenCalc.tsv", tmpScreenCalc)

hasher1 = hashlib.md5()
afile1 = open(tmpScreenCalc, 'rb')
buf1 = afile1.read()
a = hasher1.update(buf1)
md5_a=(str(hasher1.hexdigest()))
afile1.close()

hasher2 = hashlib.md5()
afile2 = open(tmpMD5, 'rb')
buf2=(str(afile2.read()))
buf3 = buf2.lstrip('b\'')
a = buf3.rstrip('\\n\'')
md5_b = a.replace('"', '')
afile2.close()

# check the md5 and if all good move to the root folder
if(md5_a==md5_b):
	print("Yes - Good to proceed.")
	shutil.move(tmpScreenCalc, MainScreenCalc)
else:
	print("No - lets not wipe out our known good file")
	print("Do Nothing. Exiting.")
```
