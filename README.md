# Translation Preface
This was written by aikoncwd in Spanish, and is here with a translation with minor cleanup and changes to idiomatic English because I barely speak Spanish. It likely is incorrectly translated, but should be good enough to get the point across, hopefully.

Please do not harass aikoncwd about this translation. I've done it just so I can see it laid out for myself.

I will be accepting pulls for translation cleanup until early 2021 nearly unconditionally.

~ greysondn

# Introduction
On 08/03/2020 and after 2 years of work, I published my first game on Steam, Cursed Gem. Unfortunately the game ended up published on pirate cracks / torrents pages at 24h. I want to emphasize that if my game has been hacked, it means that it is attractive enough for someone to bother to hack it. In addition, there are several articles that talk about how this phenomenon can be translated as a marketing campaign and give greater visibility to small indie developers like me.

*This article is not intended to discuss whether or not it is good to have your software pirated, or whether or not it is good to use DRM, or whether it is good or not to allow / deny access to your pirated game*. I just want to share a simple method to implement, which adds a little layer of protection to your game to detect if the player has bought or hacked it. At that moment it is your decision to implement the action you most want, for example showing a friendly message or directly blocking the game. We start?

# Something you should know about protection systems
**There is no 100% foolproof method to protect your game / application.** No matter how complex the system is, no matter how much encryption and obfuscation you use, it doesn't matter. Any system you implement can be cracked / hacked if they fall into experienced hands and enough time is spent on it.

Today's protection systems are based on making the task very difficult for the cracker, such that the crack appears several days (or weeks!) After the publication of the game, getting impatient players to end up buying the game instead. to wait for the crack. In fact, it is very common for large companies to decide to update and remove the DRM from their games the day after it has been cracked. These protection systems simply seek to buy time.

That said, I don't think there is a need to remember, but **the system that I have implemented in my game is not foolproof either**. If a cracker spends time on it, they will end up hacking the game. The grace of this system that I am going to explain is that the cracker does not know that his "crack" does not work, turning the pirated game into a "demo" version of the game so that, with a bit of luck, the pirate player decides to buy it.

# Stage, tools, victim, protector and executioner.
### Stage
**Cursed Gem** Cursed Gem is a game programmed on Godot Engine. I don't want to go into details as the article would take a long time, but basically we have to know that when we export a game, what we are really getting is a copy of the game engine (the engine, without the editing tools) and all our code source packaged in a PCK file. In other game engines what we normally get is a compilation, that is, our source code is linked and compiled, generating a unique binary that can be executed / played. With Godot this does not happen since there is no compilation. GDScript is an interpreted language (like Python) and that gives us many advantages and weaknesses.

What does it mean? Well, it is very easy to reverse the packaging process and obtain the source code from the PCK file. If the cracker decides to "attack" our game by unpacking the PCK, it will be very easy for him to discover our system and he will be able to hack the game again. Fortunately, crackers do not usually do this process as it is laborious. They only turn to him when their basic tools fail for some reason.

What will our goal be? Make believe that the cracker has been able to hack our game with his basic tools, but break the protection / lock after several minutes / hours of play. This way the cracker will never know that his crack does not work and will not decide to use more advanced techniques (such as unpacking the PCK that I mentioned before.)

### The tools
- **Godot Engine:** The protection system is implemented directly in the code of our game, so our main tool will be Godot Engine.
- **Checksum generator:** Any tool that allows us to calculate the checksum of a file, my favorite is HastTab, which integrates perfectly into the Windows file explorer.
- **Steam SDK:** The game is published on Steam, and therefore we use its SDK / DLL

### The victim
The victim, luckily or unfortunately, will be our game. Being a project made in **Godot**, we have 2 files to protect, the **EXE** and the **PCK**. Our protection will be in charge of checking the integrity of the engine (the **EXE** file) as well as the integrity of the Steam SDK / DLL. Then we will add additional "checks" that will allow us to know if the game is running legally or pirated. All these checks will make the cracker unable to use its basic tools and will have to spend part of its time and effort removing all the protections. *The success of this system lies in making people believe that the cracker has succeeded in hacking the sotware, to prevent them from deciding to use other advanced techniques.*

### The protector
We want to publish our game on **Steam**, and therefore we have to use its official SDK. In order for our game to "talk" to **Steam**, we will make calls using its DLL as a gateway. For example if we want to unlock an achievement, the game will simply load the **Steam** library and call the function `Steam.setAchievement (" achievement_example ")`, if we want to check if **Steam** is running on the PC We can call the `Steam.loggedOn ()` function, or if we want to check the player ID of **Steam**, we can call the `Steam.getSteamID ()` function.

All these functions are already programmed and integrated in a DLL that ** Steam ** offers us called `steam_api.dll` or` steam_api64.dll`, we have the list of available functions [here] (https: //partner.steamgames .com / doc / api). This is our protector, since we can easily and quickly check if the current user has ** Steam ** open, and if they do, check if they own (have bought) our game. This verification is obtained through the [BIsSubscribed] function (https://partner.steamgames.com/doc/api/ISteamApps). I leave you an example:

![](https://i.imgur.com/oWSeqzQ.png)  
*source: https://gramps.github.io/GodotSteam/tutorials-initializing.html*

### The executioner
For a cracker, the simplest thing is to intercept the calls your game makes on the Steam API, modifying the response.

What if a cracker gets the Steam DLL to always return `true` to the` Steam.IsSubscribed () `question? Well, your game would think that the user / player owns a legitimate copy on Steam, since the DLL would always return `true`. Well, this technique is the main tool of crackers today. But ... how do they get it? Very easy, * exchanging the official Steam DLL for a modified DLL. * Let's see a real example ...

# Steam emulators, the Swiss army knife of crackers
This is the most used method today to hack Steam games. On the internet there are implementations / copies of the official Steam DLL `steam_api64.dll` slightly modified so that certain functions always return the same result without asking the official Steam servers. These DLL implementations are known as ** "emulators" **. Making them, for example, always return `true` when the game calls the` Steam.IsSubscribed () `function. The cracker simply has to do the following:

- ** Buy ** the official game on Steam.
- ** Download ** the game and install it.
- Create a ** copy ** of the files / game in another folder.
- ** Change ** the official `steam_api64.dll` file to the cracked` steam_api64.dll` from the emulator.
- ** Check ** that the game runs with the pirated DLL.
- ** Uninstall ** the game.
- Return the game (** refund **) to get the money back.
- Upload a ZIP with the copy of the game + pirated DLL to a ** torrents ** website.

One of the most used emulators currently is the [Goldberg SteamEmu] (https://gitlab.com/Mr_Goldberg/goldberg_emulator), I am not giving secret information far from it. Anyone who downloads a pirated game and examines the DLL will see that it says Goldberg. It is widely known information on internet forums.

For this example we have the official source code of the pirate DLL, and we can check how they have re-implemented the `Steam.IsSubscribed ()` function:

```cpp
bool Steam_Apps::BIsSubscribed()
{
    PRINT_DEBUG("BIsSubscribed\n");
    return true;
}
```

* Print a debug line and return true !!! regardless of whether or not the user bought the game !! *
Many other functions of the official DLL have been modified, such as the purchase of DLCs. This is a real horror, since changing the official DLL for the emulator DLL makes the game itself not aware that it is being hacked.

# Identifying a pirated copy
We have different ways for it. We can check the path of the game, check the additional execution arguments, check the existence of certain files or folders, etc ... but we will start with the most obvious: That the game detects if the `steam_api64.dll` file is the one ** official ** or one ** modified **.

### Checking the authenticity of steam_api64.dll
In computing and in the world of telecommunications there is something called [checksum] (https://en.wikipedia.org/wiki/Checksum). It is basically a function that performs mathematical calculations on a series of data and returns a result. The same file always has the same result, if we change a single letter of that file, the result of its checksum will be different. In this way it is very easy to guarantee the integrity of an information or file.

There are several checksum functions, I will not go into details, but the most used with ** CRC32 **, ** MD5 ** and ** SHA256 **. The first thing we will do is calculate the checksum of the official DLL:

![](https://i.imgur.com/oMTuXmM.png)

The official SHA256 checksum of `steam_api64.dll` is:` A178F19A516023EFC3CC30B1E90FBEDA838D08D4F1FA006B895608D67FA60EAC`

If we now calculate the SHA256 of the * Goldberg * pirate DLL, we will get a very different checksum: `43C19ECECD799332CC09A56A11A99E90E1FC884061B046F9D1E7203330A3B721`

With a simple function we can calculate the SHA256 of the game DLL and if it does not match `A178F19A516023EFC3CC30B1E90FBEDA838D08D4F1FA006B895608D67FA60EAC` we will undoubtedly know that the game is pirated. At that moment we will raise a ** flag ** to, hours later, show a warning or block the game. I leave you an example in Godot:

    func check1() -> bool:
	    # piracy flag
        var yar = false
	    var file = File.new()
	
	    if file.file_exists("./steam_api64.dll"):
		    if file.get_sha256("./steam_api64.dll") != "a178f19a516023efc3cc30b1e90fbeda838d08d4f1fa006b895608d67fa60eac":
			    yar = true
	    else:
		    yar = true
	    return yar

### Checking for unofficial files / folders

After researching how * Goldberg * emulator works, I discovered that it usually has a file called `local_save.txt` and a folder called` steam_settings`. It is not mandatory for the pirate game to have these files, but 100% of pirate games that I have seen do, so it is always good to add this check. I leave you an example of my "real / official" game and another photo of the pirated game so you can see the differences and the files:

![](https://i.imgur.com/famnOSU.png)  
Real folder. There is no `local_save.txt` or` steam_settings`. The size of the DLL is small, 257Kb

![](https://i.imgur.com/CCVsIUq.png)  
Pirate game! There is the file `local_save.txt` and the folder` steam_settings`. The DLL is modified, its size is huge !! Although we are verifying that with the checksum, in the previous point.

Knowing this, we can add new checks within the game code:

    func check2() -> bool:
        # piracy flag
	    var yar = false
	    var file = File.new()
	    var dir = Directory.new()

	    if dir.open("./steam_settings") == OK:
		    yar = true
	
	    if file.file_exists("./local_save.txt"):
		    yar = true
         return yar

### Checking additional execution arguments

I love this method. Regardless of the DLL or the files, we can check the way in which the user runs the game.

When we publish an application on ** Steam **, we can indicate the additional arguments to run our game, I put an example:

![](https://i.imgur.com/ZRWcI9Z.png)  
Here we basically tell ** Steam ** that to run our game on Windows, it has to run `game.exe -none`. Even if the user runs the game from a shortcut, from the library or from recent games ... ** Steam ** will always pass the `-none` argument to the executable. If someone hacks our game, they will run it directly without the ** Steam ** client and therefore never pass the argument. This way we can add our third check:

    func check3() -> bool:
        # piracy flag
	    var yar = false
	
	    if OS.get_cmdline_args():
		    if OS.get_cmdline_args()[0] != "-none":
			    yar = true
	    else:
		    yar = true
	    return yar

### Protect engine integrity
Finally, it's time to protect the Godot executable. We want to prevent the cracker from modifying the engine to make, for example, that the `get_sha256 ()` function always return the correct hash, or to prevent the `file_exists ()` or `get_cmdline_args ()` function from returning precooked and modified values ... We will check the executable's checksum to see if the cracker has modified it. First we calculate the actual checksum:

![](https://i.imgur.com/yqgy7Xy.png)  
We get a SHA256 from: `E5161680DBD7FBFB1D8D63A1F707AC0FF48CB15E3591EB4E01DDA824EB1667C9`, so we add the final check:

    func check4() -> bool:
        #piracy flag	
	    var yar = false
	    var file = File.new()

	    if file.file_exists("./game.exe"):
		    if file.get_sha256("./game.exe").to_upper() != "E5161680DBD7FBFB1D8D63A1F707AC0FF48CB15E3591EB4E01DDA824EB1667C9":
			    yar = true
	    else:
		    yar = true
	    return yar

# Implement protection
Now we have 4 functions that will return `true` if the game is running" pirated "or` false` if it is running legit and within Steam. You have to decide what to do in case the hacking is detected.

The first thing that comes to mind is to carry out these checks when starting the game and automatically close it if we discover that it is pirated, but this is very counterproductive since the cracker will check if the game is running before publishing the torrent.

If the cracker discovers that the game has protection, he will start to use more aggressive techniques until decompiling the game and bypassing our protection. I recommend that you do these checks after a while. Let the pirate play a couple of levels or a couple of hours and then show a friendly message stating that the game is pirated, please buy it on Steam. * We want to turn pirates into buyers, so it's best to make things easy. *

In the case of ** Cursed Gem **, I decided that the current "savegame" is compatible with the official version, this way a pirate player can buy the game and continue playing without repeating the first levels.

Other developers have chosen to "mark" pirate players with a different skin, hat, parrot, etc ... in this way you make it impossible for them to stream or YouTube without going through the public shame of being a self-confessed pirate.

If you finally decide that you want to block access to the game, you can use fancy options. For example in GTA-IV, the player's camera moves in a zig-zag fashion, pretending to be drunk. Making it very difficult to move forward. In the Mafia game (I think), it was impossible to get into cars. In the case of Cursed Gem, the jump is cut in half, making it impossible to advance in the adventure:

![](https://i.imgur.com/q8Oij57.gif)

My recommendation is that you distribute the 4 checks in different areas of the game, making the task of gutting the code more complex. You can also make one check jump at level 2, and the other at level 3. So if the cracker clears the protection at level 2, it will never know that the message jumps again later. The crackers do not complete or play the games, they simply apply the crack and verify that the game starts.

# Something to keep in mind if your game is cross-platform
If your game decides to publish it for Linux and MacOS, it is necessary that you apply these protections only in the Windows version. The checksum are different in another operating system, so you will have to check this first, I give you an example:

    if OS.get_name() != "Windows":
	    return false

This way if the game is not from Windows, I will not do any checks. Pirated versions of the games are practically non-existent for Linux and MacOS. But I leave that in your hands if you want to apply protections in those systems

# Sorry to insist: This method is not infallible 100%
I am repeating myself a lot, but I want to make this very clear: ** Neither this system nor any other system is 100% effective against hacking. **
The strong part of this system is being able to detect a pirated version and launch the warning hours later, so that the cracker does not know that we have additional protection.
