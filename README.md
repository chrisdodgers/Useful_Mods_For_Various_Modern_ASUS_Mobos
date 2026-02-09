# Useful Mods for Various Modern ASUS Mobos: *(Apple logo splash, add a boot path for OpenCore.efi, unlock hidden options/pages)*

## About:
This is a general guide on how to modify your BIOS to change the default splash screen from the *ASUS/ROG Strix/ProArt, etc.* logo to an Apple logo *(or logo of your choice)*, add a hardcoded search path for EFI\OC\OpenCore.efi, and to unlock some hidden options/menus.

> [!WARNING]
> **THE FOLLOWING GUIDE HAS BEEN WRITTEN AND BASED ON MAINLY TESTING ON A ROG STRIX Z790-E. DO NOT BLINDLY COPY AND PASTE VALUES, AS THIS IS A GENERIC GUIDE THAT CAN APPLY TO MULTIPLE ASUS MOBOS WITH POSSIBLY SLIGHTLY DIFFERENT GUIDS/VALUES THAN SHOWN IN THIS GUIDE! IF YOU ARE NOT SOMEWHAT TECHNICALLY INCLINED AND DO NOT UNDERSTAND TERMS MENTIONED BELOW - PLEASE STOP AND DO NOT PROCEED WITH ATTEMPTING GENERAL METHODS IN THIS GUIDE!**
>

>[!NOTE]
>I have personally tested these mods and methods *only* on a PRIME Z390-A, Z790 MAX WIFI GAMING 7, and a ROG STRIX Z790-E GAMING WIFI. This guide does apply to other modern ASUS mobos with APTIO V firmware. Once again, I can't stress enough that there may be differences and you should NOT blindly copy/paste things in this guide! This guide is AS-IS and is for informational and educational purposes only! 
>

> [!WARNING]
> **THE FOLLOWING PROCEDURES CAN POTENTIALLY BRICK YOUR SYSTEM IF YOU ARE NOT CAREFUL, AND I TAKE NO RESPONSIBILITY IF YOU TURN YOUR SYSTEM INTO A PAPERWEIGHT! ONLY CONTINUE WITH THIS GENERAL GUIDE AT YOUR OWN RISK! THIS GUIDE IS ONCE AGAIN FOR EDUCATIONAL PURPOSES ONLY! IT IS ALSO 100% YOUR RISK IF YOU CHOOSE TO ATTEMPT THIS GENERAL GUIDE.** 
> 
> You have been warned. Proceed very carefully if you chose to do so...
>

## Pre-Requisites:
>[!WARNING]
>Take a backup of your current BIOS settings as you will lose them when flashing! You can boot into your BIOS menu to backup your current settings to a .txt file which you can save on to a connected USB drive that is formatted as FAT32. You *can* also backup a user profile and save it as a .CMO, but I would advise of **NOT** restoring from this .CMO after modifications have been done. Instead you should manually re-apply settings you need. I have personally seen adverse effects happen when restoring a user profile that is from a different BIOS version and/or from a backup of a non-modded BIOS. 
>

### Main tools used in this guide that you should download/use:
- [UEFITool](https://github.com/LongSoft/UEFITool/releases) (Use 0.28.0 for applying edits)
- [IFRExtractor-RS](https://github.com/LongSoft/IFRExtractor-RS/releases)
- [IDA Free](https://hex-rays.com/ida-free)
- [ImHex](https://imhex.werwolv.net/) *(can use a hex editor of your choosing or edit within IDA. This is what I personally use/recommend on macOS.)*
- [UEFI-Editor](https://boringboredom.github.io/UEFI-Editor/)

### *Optional tools you may need depending on method of flashing:*
- [Homebrew](https://brew.sh)
- [flashrom](https://formulae.brew.sh/formula/flashrom)

## Lets Begin Our Mods:

### Download the latest BIOS image for your exact ASUS motherboard, or dump your current image using a SPI programmer. We will be applying our mods to this image.

# 1. Changing the Splash Screen from ASUS, etc. to an Apple Logo *(or a logo of your choosing)*:

## Preparing Our .BMP Image:
- We will begin by creating a .bmp image as our logo. The image MUST meet ALL of the following requirements:
   - Resolution: 672x378
   - Format: Bitmap (.bmp)
   - Color Depth: 8 bit
   - MAX/TARGET SIZE: 762 KB
- You can download this [pre-made Apple logo splash screen here](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Demos/Pre-Made_AppleLogoSplash.bmp), or better yet - create your own .bmp image where the logo might scale better for your exact setup.

>[!NOTE]   
>If the file size is larger than 762 KB, it can cause you to have a black screen instead of a logo appearing. Please ensure before you continue that your .bmp file meets the specifications above.
>

- Open UEFITool and press `⌘F` to find, and then select `"Text"` in the search options.
- Search for `logo.bmp`.

![LogoBMPSearch](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/LogoBMP_text_search.png)
 
- At the bottom of UEFITool, you should see one hit for `logo.bmp`. Double click on that hit we got for `logo.bmp` and it will jump you to the section it found it at.

![LogoBMPHit](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/LogoBMP_hit.png)

- You will see that there is a `Raw section` within the GUID we got a hit for `logo.bmp` in. Right click `Raw section` and select `Extract body...`. 
- Save this as something like `logobkup.bmp`, as we are doing this to verify that this is our current splash logo, and verifying our new custom `logo.bmp` matches the existing/required properties as explained above in the guide. 
- Once the above has been verified, we can now right click the `Raw section` we are working with and select `Replace body...`.

>[!NOTE]
>You may need to click `Show Options` so you can view `All files` instead of just `Raw files (*.raw *.bin)`, as we need to select our new `.bmp`.
> 

- Select the custom `logo.bmp` you made. 

# 2. Adding a Hardcoded Boot Search Path to `EFI\OC\OpenCore.efi`

### What does this mean?

In a section of our bios image, there are multiple hardcoded search paths which get used by the UEFI boot manager. Example: `EFI\Microsoft\bootmgfw.efi` is defined with a name of `Windows Boot Manager`. If that exact path exists in your ESP, you'll see it show up in your boot manager options. Well, of course there isn't a boot path in there pre-defined specifcally for OpenCore. From what I have seen unlike some other systems, there isn't a way to manually add a name/path within your boot manager options. Instead, we are stuck with using the `BOOTX64.efi` shim which then points to `EFI\OC\OpenCore.efi` *(which shows up as UEFI OS in boot manager options)*. If you poke around a section in your bios image - you will see quite a few defined paths that are hardcoded. Lets replace a path we aren't using with the path to point to `OpenCore.efi`.

### Finding/Extracting and Making the Edits:

- In UEFITool, lets press `⌘F` to find `OpenSUSE`. *(You most likely will see multiple hits. Some/all of these hits lead to the same section. You can help confirm we found the correct section by also searching `bootmgfw`. The correct hit for that should also jump to the same section we found `OpenSUSE` in.)*
- In my example/case, I found both of the searches have hits that exist in a PE32 image section under `8F4B8F82-9B91-4028-86E6-F4DB7D4C1DFF`. Right click the PE32 image section your hits fell under and select `Extract body...`. For reference, I named my export as `C1DFF.bin`. You can name this of course as something more appropriate to your your exact situation. *(Context to keep in mind through the rest of this guide section).* 

![PathsSection](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/PathsSection_extract.png)

- Open `C1DFF.bin` *(or the name of what you just extracted)* in IDA Free.
- Press `⌘F` and search for something that we know exists in here as a boot path. In this case, I'm going to search for `OpenSUSE`. As you can see below, we have a few hits for `OpenSUSE`.

![IDAOpenSuseHits](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/IDA_OpenSuse_Hits.png)

- As pictured, I see `\\EFI\\opensuse\\grubx64.efi` with an associated name of `openSUSE`. This is what I am personally going to replace to be our OpenCore path. Double click this and it will jump you to where this is in `IDA View-A`. *(You may end up choosing something else to instead replace with a path to `OpenCore.efi`. For the sake of this guide, will be continuing with the path I chose to replace.)*.

>[!WARNING]
>Ensure the new path we are using to replace an existing path does **NOT** exceed in length. Example: Do **NOT** replace `\EFI\Suse\elilo.efi` with `\EFI\OC\OpenCore.efi`. We can see we are exceeding in length and this will be problematic. Instead, find an existing path you are not using that equals, or is in greater length of the replacement path.
>

- Now that we are in `IDA View-A` jumped to the section we are interested in *(as pictured in this case with `18100` being highlighted)*, we can double click on that line at`aEfiOpensuseGru`, which will show us the address we need to jump to in a preferred hex editor. As we can see in this example, we need to jump to `0x1AD58`:.

![IDAOpenSuseLine](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/IDA_OpenSuse_Line.png)

![IDAOpenSuseAddress](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/IDA_OpenSuse_Address.png)



>[!NOTE]
>You *can* make these edits within IDA if you'd like to. Personally, I prefer to use ImHex for applying my edits. This is where you can make the decision of what you'd personally like to use to make necessary hex edits. 
>

![ImHexStart](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/ImHex_Address_Start.png)

- Lets find the hex values for `\EFI\OC\OpenCore.efi` by using a preferred ASCII to Hex converter. Converting, we know that this is `5C 45 46 49 5C 4F 43 5C 4F 70 65 6E 43 6F 72 65 2E 65 66 69`. 

>[!WARNING]
>Take a look closely at the path we are replacing. Notice how there is padding (`00`) between each value and ASCII character we see? Example: `\.E.F.I.\.o.p.e.n.s.u.s.e.\.g.r.u.b.x.6.4...e.f.i.......` Well, we need to retain this. Do **NOT** just blindly paste in the values above! We need to keep the same padding between our values we are replacing. If the path you are replacing is longer than our *new* path? Replace those extra path values with `00`.
>

- We can see under the path we just replaced, there is the defined name of this path. We can see in our hex editor `o.p.e.n.S.U.S.E`. We need to follow the same steps as above. Using a ASCII to Hex converter of choice, we know `OpenCore` is `4F 70 65 6E 43 6F 72 65` in hex. It also does not exceed the length of what we are replacing. Just as with before, we need to retain the `00` padding between each value.

- After the edits have been made - you should have something that looks like this:

![PathEditsFinal](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/ImHex_Path_Range.png)

### Applying the mods into our image:
- Save the edited `.bin`, and go back to UEFITool to replace the PE32 image section you previously extracted/edited. 
- Right click that PE32 image section and select `Replace Body...`. Select your edited `.bin`. 


# 3. Unlocking Hidden Pages/Options:

For this portion of the guide, we will be using UEFI-Editor from BoringBoredom. Its a very easy to use editor which we can make our access/value edits from by using a web browser by uploading a few required files. [Visit UEFI-Editor's README for a more complete guide on how to use this tool](https://github.com/BoringBoredom/UEFI-Editor?tab=readme-ov-file).

### Grabbing the required files using UEFITool:

 - Our `Setup` PE32 image. *(Search `CFG Lock` in UEFITool and it should show one hit. Double click that hit and right click the PE32 image section and select `Extract as is...`. Save it as `Setup.sct`)*.
 
 ![CFGLockSearch](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/CFGLock_search.png)
 
 - Our `AMITSE` PE32 image. *(If you search `AMITSE` - you will see multiple hits. You will know you landed on the correct hit when you jump to the PE32 image section under AMITSE as pictured below. Right click that PE32 image and select `Extract as is...`. Save it as `AMITSE.sct`)*

 ![AMITSEExtract](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/AMITSE_extract.png)
 
 - Our `AMITSESetupData`. *(Following what you just previously did, you should see the AMITSESetupData after the AMITSE section. Right click on the `Freeform subtype GUID` and select `Extract body...`. Save it as `Setupdata.bin`.)*

 ![AMITSESetupExtract](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/AMITSESetupdata_extract.png)

### Using IFRExtractor to make a human readable dump of `Setup.sct`:

- Place `ifrextractor` within the folder containing your recently exported `Setup.sct`.
- Go to the folder you are working in and run `./ifrextractor "Setup.sct" verbose` in Terminal.
- You should now see something like `Setup.sct.0.0.en-US.uefi.ifr.txt` in the folder you are working out of.

### Going to UEFI-Editor and making our edits.

- Go to [UEFI-Editor](https://boringboredom.github.io/UEFI-Editor/) in your browser. Upload the files you extracted/created from above and continue.

> [!NOTE]
>  *(I personally had issues exporting the edited file(s) when using UEFI-Editor in Safari. Would recommend using a Chromium based browser or Firefox for this specific task.)*
> 

- From here, you can browse to view options/pages you are interested in unhiding. Access Level `09` means its currently hidden. Setting the access level to `01` will unhide said option. Example: Search for `CFG Lock` and set its Access Level from `09` to `01`.

*Before Example:*

![CFGLockBefore](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/CFGLock_Before.png)

*After Example:*

![CFGLockAfter](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/CFGLock_After.png)

- You will find some pages are hidden by `Suppress If` args. Example as like seen below of pages in PCH Configuration mostly all being suppressed. You can click on a `Suppress If` arg to disable that arg from suppressing said page/option.

*Before Example:*

![PCHBefore](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/PCH_Before.png)

*After Example:*

![PCHAfter](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/PCH_After.png)


>[!WARNING]
>Do *NOT* just randomly go through here setting *everything* to Access Level `01` and removing all `Suppress If` args! Doing so *most likely* will break things resulting in undesired behaviors. Recommendation: only unlock pages/options you are specifically after, and be careful doing so. Example: CFG Lock, SerialIo Configuration, Interrupt Redirection Mode Selection, etc.
>

### Applying the mods into our image:

- On the bottom left of UEFI-Editor you will see a button to download the `UEFI Files`. This will export/download files that were actually modified. 
- Replacing the modified image sections in UEFITool will be just like how we extracted said sections. Example: Earlier we extracted our `AMITSE` PE32 image using `Extract as is...`. Well, in this case we have our newly modified `AMITSE.sct` which we will use `Replace as is...`. Ensure that you are replacing the *exact* sections with the *exact* matching `.sct` or `.bin` for said section. Once again - pay very close attention to details and basically do what you did at the beginning of this section of this guide.  


# Saving and Flashing Our Modded Image:
Some mobos from ASUS have a USB port physically labeled as "BIOS" which is used for flashing a bios image off of a FAT32 formatted USB drive containing a bios image [(More information from ASUS about USB BIOS FlashBack).](https://www.asus.com/us/support/faq/1038568/) In the case of the Rog Strix Z790-E I am using, it has this, and we can use this for flashing our newly modified image. However, if your ASUS mobo does not have this feature - I will briefly cover additional methods you can use to flash.

## Using ASUS's USB BIOS FlashBack:

- We need to save our completed moddified image as a `.CAP`. Press `⌘S` in UEFITool to save our image. In my case, I am naming it `SZ790E.CAP`.

> [!NOTE]
> You need to find the exact name your mobo is expecting when flashing! This information can be found by going to the `BIOS & Firmware` section for your specific mobo on ASUS's website. Example - in the release notes of a given image version, you will see something like this in the release notes: `"Before running the USB BIOS Flashback tool, please rename the BIOS file (SZ790E.CAP) using BIOSRenamer."` We know for this exact mobo being used in this guide, we need to make the name `SZ790E.CAP`. What you need to name the image to will most likely be different.
> 

>[!WARNING]
> **AS WANRED PREVIOUSLY AT THE BEGINNING OF THIS GUIDE, I ACCEPT NO RESPONSIBILITY IF YOU TURN YOUR SYSTEM INTO A PAPERWEIGHT! YOU HAVE BEEN WARNED ONCE MORE! CONTINUE AT YOUR OWN RISK...**
> 

- The rest of the flashing proceedure you will need to follow the ASUS USB BIOS FlashBack guide linked above, as we now have our `.CAP` image ready to go. 
- Throw the image on the root of a FAT32 formatted USB drive, shutdown the system, plug it into your "BIOS" USB port, hold the flash button down for 3 seconds, wait for the FlashBack LED to finish blinking as it reads/writes/verifies.......Success. 


## Using an SPI Programmer with flashrom:
So, you don't have the functionality to do what was just mentioned above. *"Lets just put the image on a USB drive, boot into our bios, and launch `ASUS EZ Flash 3 Utility` and flash - right?"* Well, from my personal findings - I had no success doing this. I'd get an error with it not wanting to accept the modified image. 

So now what? At this point, I'd recommend getting some form of an SPI Programmer. For the brief example I'll be explaining - I'm using a cheap dinky CH341a programmer in use with flashrom. You can absolutely grab a better SPI programmer, but this will do if you happen to have one handy. 

>[!NOTE]
>Random note - be very careful with the various cheap CH341a programmers out there. My particular one was sending out 5V instead of 3.3V - which caused issues when trying to create backups and compare backups. Ended up needing to follow [this guide](https://wiki.chucknemeth.com/usb-devices/ch341a/3v-ch341a-mod#pid=1) to fix this issue with my particular programmer. Just a side note of caution to be aware of on *some* of these cheap programmers floating out there.
>

- We need to save our modified image as either an `.img` or `.rom`. Press `⌘S` in UEFITool to save our image. In the case for this guide, I'll be going with `Z790MOD.img`. 

>[!WARNING]
> **AS WANRED PREVIOUSLY AT THE BEGINNING OF THIS GUIDE, I ACCEPT NO RESPONSIBILITY IF YOU TURN YOUR SYSTEM INTO A PAPERWEIGHT AND DON'T HAVE A PLAN OF ACTION TO RECOVER IF NEED BE. CONTINUE AT YOUR OWN RISK...**
> 

- With the system off and disconnected from power, clip your programmer onto what you've identified as your SPI chip(s). 

- Connect your SPI programmer to your machine of choice for performing the backup/flash proceedure. *(In this guide example I am once again using a CH341a. The following commands will vary based on the programmer you are using!)*
- Backup your SPI chip(s). Example: `sudo flashrom -p ch341a_spi -r chip1_bkup1.img`.

>[!NOTE]
>Some mobos have *two* SPI chips instead of just one - where an image is split between two chips! This is very important to be aware of, as it will add some aditional steps which I will briefly cover. More detailed flashing guides can be found elsewhere. In the case of having *two* SPI chips - ensure you backup the second one as well. Example: `sudo flashrom -p ch341a_spi -r chip2_bkup1.img`.
>

>[!NOTE]
>It is possible you may see an error asking to specify your specific chip with flashrom listing a few options to select from using the `-c` arg. If this applies to you, then you can verify the model of your SPI chip(s) by looking at the model printed on it using a magnify glass or with a camera. Example: `sudo flashrom -p ch341a_spi -c GD25Q128E/GD25B128E/GD25R128E/GD25Q127C -r chip1_bkup1.img`. 
>

- Create a second backup of your SPI chip(s). Example: `sudo flashrom -p ch341a_spi -r chip1_bkup2.img`.
- To ensure our programmer is working properly and that we indeed have a good clipped connection to our SPI chip, lets check to see if the two backups contain any differences. Example: `diff chip1_bkup1.img chip1_bkup2.img`.

>[!WARNING]
>**IF THE DIFFERENCE COMPARISON BETWEEN YOUR TWO DUMPED BACKUPS RETURNS ANY DIFFERENCES - STOP AND DO NOT PROCEED. RE-VERIFY YOUR PROGRAMMER CLIPS CONTACT TO YOUR SPI CHIP, AND ENSURE YOUR PROGRAMMER IS SETUP CORRECTLY! DO NOT PROCEED IF YOU ARE GETTING INCONSISTENT BACKUPS!**
>

- Provided the backups indeed match with no differences between them - we can now continue on with flashing our new modified image. 

### I have ONE SPI chip:

>[!WARNING]
>**ENSURE THE IMAGE YOU BACKED UP MATCHES THE SIZE OF THE NEW IMAGE WE ARE GOING TO FLASH!** IF YOU HAVE *TWO* SPI CHIPS - SKIP TO THE NEXT SECTION COVERING THIS! 
>

- In the case of having a single SPI chip, you verified the 2 dumped backups, verified you correctly and carefully made your modifications to the new image - you can now flash the new image. Example: `sudo flashrom -p ch341a_spi -w Z790MOD.img` *(Remember if you had to use `-c` earlier, you of course will need to specify that as well when writing with flashrom).*
- Pateintly wait for the flashing/verification process to complete. Once complete, you should be able to disconnect your programmer, plug in, and power on your system. 

### I have TWO SPI chips:
As per above, we need to make sure the dumps of each SPI chip are consistent. Also, keep note of the size of the image for both chips. If you add the size of the image dump from SPI chip 1 and SPI chip 2, it should equal the size of the full modded image we want to flash. Basically, what we will be doing is referencing our two SPI chip dumps so we can figure out where to cut our modded image into two images. 

- Open the dump we took of SPI chip 1 and SPI chip 2, and open the full modded image we made earlier in the guide. *(As with the rest of the guide, I am doing this in ImHex. You are free to do this general process in another hex editor of choice).* 
- On all 3 opened images, scroll up to the very top. Compare the top sections of both SPI chip dumps. The chip dump that has matching values to our full-size image should be referenced as "chip 1".

>[!NOTE]
>Do rename accordingly if need be to make sure we do not get these 2 chips confused in this process. We need to be sure we are referencing "chip 1" as the chip that holds the beginning section of our full image! *(Think of it as SPI chip 1 is the beginning point and then we run out of space. So, we splice the full image where we ran out of space and the rest is stored on SPI chip 2).*

- Since we are working with tabs inside ImHex, go to the full-size modded image you have opened in this session. You will see in the mid info section that we a total size of `0x1800000` (24.00MiB). Note the total size value you see down. *(should be the same or similar)*

![FullImage-Size](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/FullImage_Size.png)

- Go back to the dump of SPI chip 1. Like the previous step, look at the mid info section and note down the total size. In this case, we see the total size of our SPI chip 1 dump is `0x800000` (8.00MiB).
- Go back to the dump of SPI chip 2. Like the previous step, look at the mid info section and note down the total size. In this case, we see the total size of our SPI chip 2 dump is `0x1000000` (16.00MiB).

![Chip2Size](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/Chip2_Size.png)

- Go back to the full-size modded image. We know our image for SPI chip 2 is going to start here at `0x800000`. *(You can once more verify this by pressing `⌘G` and entering in that address to jump to. You can see `0x800000` matches exactly to the start of the dump of SPI chip 2 and `0x7FFFF0` matching  up to the very end of the SPI chip 1 dump.)*.
- Scroll to the very bottom of the full-size modded image and select the *last* value. In this case, `0x17FFFFF`. Go to `Edit -> Select...` in the menu bar. Since we know at this point SPI chip 2's data starts at exactly `0x800000` - enter `0x800000` as our *Begin* and leave `0x17FFFFF` as our *End*. 

![SelectChip2Range](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/Select_Chip2Range.png)

- We should now have an *exact* selection. Press `⌘C` to copy this selection. Press `⌘N` to create a new file. Press `⌘V`to paste in the exact selection we copied. Press `⌘S` to save this file as `chip2_mod.img`. This will be the image we will flash to SPI chip 2.

![Chip2Image](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/Chip2_New_Image.png)

- Go back to the full-size modded image. The *exact* selection we made in the previous step should still be selected. Right click in that selection and select `Remove...`. 

![Chip2RemoveSection](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/Remove_Chip2_Fullimage.png)

- We will be now asked to confirm the section we want to remove. In this case, it asks if we want to remove *address* `0x800000` *size* `0x1000000`. Well, we can see from info we noted down this is correct so we can click `Set`. 

![Chip2RemoveSection-pt2](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Screenshots/Remove_Chip2_fullimage-pt2.png)

- We can now see the bottom of this file now matches the bottom of the dump from chip 1. Go to `File -> Save As` in the menu bar and save this as `chip1_mod.img`. 

>[!WARNING]
>TRIPLE CHECK that `chip1_mod.img` and `chip2_mod.img` add up to the **EXACT** size of the full-size image we have been working with this entire guide! ONCE MORE - check and make sure you split correctly by using the dumps from SPI chip 1 and 2 as reference! Does this once again check out perfect? If so - move on to the flashing portion. **Does anything look incorrect? If so? STOP AND GO BACK!** *Seriously... this needs to be absolutely precise and if not? Don't expect your system to boot after flashing....*
>

- Lets flash one SPI chip at a time, starting with SPI chip 1. Example: `sudo flashrom -p ch341a_spi -w chip1_mod.img` *(Remember if you had to use `-c` earlier, you of course will need to specify that as well when writing with flashrom).*
- *Pateintly wait for the flashing/verification process to complete. Once complete, you should disconnect your programmer, unclip from SPI chip 1, and prepare to repeat the process for SPI chip 2.*
- Once connected to SPI chip 2, lets flash just like how we did previously, instead this time using the image we made for SPI chip 2. Example: `sudo flashrom -p ch341a_spi -w chip2_mod.img`.


# Finishing Notes:
At this point, you should be able to plug in your system, boot into your bios menu, and can re-apply settings that were lost during the flashing process. You should also see the mods you made in effect. Once again, this is a *general purpose guide* and is AS-IS. Was not meant to be copy/paste. Do not bother complaining to me if this bricked your system. You were previously warned. 

# Images:

![AppleSplash](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Demos/SplashScreen.png)

![KubuntuSplash](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Demos/SplashScreen-Kubuntu.png)

![OpenCoreBootOption](https://github.com/chrisdodgers/Useful_Mods_For_Various_Modern_ASUS_Mobos/blob/main/Demos/OpenCore_BootOptions.png)


# Credits:
- ASUS/AMI for the base hardware and firmware we are using
- [LongSoft/vit9696](https://github.com/LongSoft/UEFITool) for UEFITool/IFRExtractor and other incredible community contributions.
- [BoringBoredom](https://github.com/BoringBoredom) for UEFI-Editor!
- [hex-rays](https://hex-rays.com/) for IDA.
- [WerWolv](https://github.com/WerWolv) for ImHex.
- [flashrom](https://github.com/flashrom) for flashrom.
- [Chuck Nemeth](https://wiki.chucknemeth.com/usb-devices/ch341a/3v-ch341a-mod#pid=1) for a great write up on fixing *some* cheap CH341a programmers.
- [Homebrew](https://github.com/Homebrew) for Homebrew.
- [Acidanthera](https://github.com/acidanthera) for OpenCore, and other related fantastic community contributions.
- Apple for the Apple logo and for creating macOS. 




