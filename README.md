# MacBlasters

Our goal is to get a better understanding of the internals of the OS X environment. None of us know jack about it so we are taking a hands-on knowledge-sharing approach with weekly meetings and additions here. 


## Alex's Corner


#### Terminal basics

The terminal by default is zsh. This is an alternative to Bash. There's really no reason to use zsh unless you like it, I'd stick with bash which you can configure to be the default shell. I use iTerm2 instead of Terminal application, it's just cleaner looking by default. I like to set infinite scrollback in case i lose stuff and, because i'm old, set the font larger. There are other terminal applications (like Terminator) that will work on OS X. So if you don't like the shell experience, there are lots of alternatives.

#### Native binary files

Mac OS X can run a number of types of binary files. The preferred and main file type we'll look at is the Mach-O (pronounced mock-oh). You can see a variety of applications in `/System/Applications`:

```
alejandrocaceres@Alejandros-MacBook-Pro /S/Applications> ls
App Store.app          FaceTime.app           Messages.app           Preview.app            TextEdit.app
Automator.app          FindMy.app             Mission Control.app    QuickTime Player.app   Time Machine.app
Books.app              Font Book.app          Music.app              Reminders.app          Utilities
Calculator.app         Home.app               News.app               Siri.app               VoiceMemos.app
Calendar.app           Image Capture.app      Notes.app              Stickies.app
Chess.app              Launchpad.app          Photo Booth.app        Stocks.app
Contacts.app           Mail.app               Photos.app             System Preferences.app
Dictionary.app         Maps.app               Podcasts.app           TV.app
```

note that these are actually directories (pedantic note: they are not FOLDERS, folders are MS Windows, in \*nix operating systems they are directories) and not single files (like a .exe). This is because these are app bundles, which are treated specially by the OS and has a rigid directory structure. So let's find the file that I'd actually run. Let's pick the Music.app for no particular reason:


```
alejandrocaceres@Alejandros-MacBook-Pro /Library [1]> cd /System/Applications/Music.app/
alejandrocaceres@Alejandros-MacBook-Pro /Library [1]> cd Contents
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/Contents> ls
Frameworks     MacOS          PlugIns        XPCServices    version.plist
Info.plist     PkgInfo        Resources      _CodeSignature
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/Contents> cd MacOS/
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/C/MacOS> ls
Music
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/C/MacOS> file Music
Music: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e]
Music (for architecture x86_64):	Mach-O 64-bit executable x86_64
Music (for architecture arm64e):	Mach-O 64-bit executable arm64e
```

OK great, we're not crazy, the Mach-O format is being used. But what about non-.app stuff? I have many many seemingly Linux commands - are these in ELF (Linux) format or are they in Mach-O format? It sure feels like I'm running an ELF...

```
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/Contents> which ls
/bin/ls
alejandrocaceres@Alejandros-MacBook-Pro /S/A/M/Contents> file /bin/ls
/bin/ls: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
```

...but OS X doesn't care about my feelings. The ls program has been recompiled as a Mach-O format. Great, we're locked in to something, perhaps this will help us attack binaries in the future. In compilation there are a few stages, first is the linking and importing stage where all of your imports to use libraries must be listed. The linking phase is critical in that it is what allows you to reuse code, you just need to tell it where that code is. Typically, this code is compiled. These are called `dll` (dynamic link library) in Windows and `.so` (shared object) in Linux. With that in mind let's see what other stuff we can discover about this executable.

Before we do anything, there is a critical step to bug hunting and offensive security that is often skipped. Google the sh** out of your topic. So let's see what useful stuff Google has to tell us about binaries in OS X. The first thing I really want to know is what does a Mach-O file "look" like. These binaries must follow some sort of standard such that the OS knows how to run it, knows where to find metadata that it needs, etc. So check out this diagram:

[Broken down Mach-O file](https://www.symbolcrash.com/wp-content/uploads/2019/02/File_structure.png)

And let's use Google to go through these sections:

- Starting with the Mach Header. It’s purpose is to describe what the file contains, and how the Kernel and Dynamic Linker should handle it. The first 4 bytes are, like with any file, it’s “Magic Number”. A Magic Number is used to identify a file format. In the case of Mach-O’s there are three Magic Numbers that one may come across. 0xfeedface for 32-bit, 0xfeedfacf for 64-bit and 0xcafebabe for Mach Universal Binaries / Object files. This is a breakdown of the C struct that makes up that section: ```    struct mach_header {
        uint32_t            magic;          // mach magic number
        cpu_type_t          cputype;        // cpu specifier
        cpu_subtype_t       cpusubtype;     // cpu subtype specifier
        uint32_t            filetype;       // type of mach-o e.g. exec, dylib ...
        uint32_t            ncmds;          // number of load commands
        uint32_t            sizeofcmds;     // size of load command region
        uint32_t            flags;          // flags
        uint32_t            reserved;       // *64-bit only* reserved
    };```
- Load Commands are placed directly after the Mach-O header in the file. They specify the logical structure of the file and the layout of the file in virtual memory.
- If you are unfamiliar with how object files work, you have a number of these segments. The __TEXT segment contains the instructions that will be executed by the CPU, and the __DATA segment contains both static local variables and global variables. These are both standard, however you may find additional segments such as __PAGEZERO and __LINKEDIT, and in XNU Kernelcaches, you’ll get even more funky segment names like __PRELINK_INFO and __LAST.

Segments are further divided into sections, so for example you’ll find __cstring in the __TEXT segment, formatted as __TEXT.__cstring, as a common one.

Note that there 





First, I want to see what it's linked to! Introducing `otool`, if you're familiar with `objdump` on Linux, this is essentially the same thing except for Mac. So let's see how we find the linked libraries. Turns out the -L switch will give us these libraries:








## Rich's Corner




## Emily's corner





## Nicholas' Corner

