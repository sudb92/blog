## CERN-ROOT and Code-Blocks


An acquaintance recently wished to utilize a full program in CERN's ROOT6 with Code::Blocks IDE on Linux/WSL because they found the interface more familiar from prior coding exercises. I hadn't used an IDE in quite a long time so I found the prospect interesting to explore. There were a few weird hoops to jump through but I managed to figure it out. Here are the steps, since I see googling didn't lead to a ready-to-go solution.
* We had a custom Makefile that worked on bash, which uses ```root-config``` to dynamically assign compiler flags and libraries
* <b>Fix 1</b>: Go to ```Project > Properties```
    - Tick "This is a custom Makefile".
    - If we don't do this Code::Blocks will make assumptions about how to compile files and make itself a makefile. At first glance, the process seems to go by turning all .cpp files to .o and trying to link all the .o's together to an executable with the project's name.

![Screenshot-1](https://github.com/sudb92/blog/blob/09312e73d667a27451b132ce5ba376aa385f6a3a/_posts/2023-09-10-img1.png) 

* <b>Fix 2</b>: If you now click 'Build', the default command that gets run would be ```make -f Makefile Debug``` or ```make -f Makefile Release```, because Code::Blocks expects this sort of structure.
    - Sometimes, like in our case, you want some control on this behavior instead of redesigning the makefile.
    - To fix this, go to ```Project > Properties > Project's Build Options > "Make" Commands```, and remove all mentions of ```$target``` in there. We only need ```make -f Makefile``` typically. Or, you could add whatever other 'make' switches/tricks you require here.

<img src="https://github.com/sudb92/blog/blob/main/_posts/2023-09-10-img2.png"/>


* <b>Fix 3</b>: Okay, let's try compiling again after hitting "OK" as many times as needed so the settings above are saved.
    - If your Makefile has all the `root-config` mentions expanded out in full, things will run fine. But if you have our situation, and have the makefile literally use `root-config --cflags` etc as part of recipes, Code::Blocks will fail at the first mention of 'root-config' or 'rootcint' with something like ```bash: line 1: root-config: command not found```.
    - This happens because Code::Blocks does not have all the right paths in its global $PATH variable to 'see' root-config and other things hidden in $ROOTSYS/bin, with $ROOTSYS typically defined in .bashrc by running ```source /path/to/root/bin/thisroot.sh```. What do we do? Skip the next step, I just had to put it in there so I remember it.
    - I tried a few things that didn't work, like adding 'source ~/.bashrc' in ```Project > Build Options > Pre/Post Build Steps```, or doing a $PATH export there like ```export PATH=$PATH:/<path>/<to>/<root>/bin/```. They didn't do much as of CB version 20.03.
    - What *did* help, was navigating to ```Settings > Compiler > Toolchain Executables > Additional Paths``` hidden away deep, and using ```Add``` to add, in our case, ```/opt/root-6.28.06/bin/``` which was the installation directory, containing root-config, rootcint, thisroot.sh etc. Following all this, running "Make" did all it had to do, following the exact process ```make all``` would've done for us in bash, and bringing up the executable in an ```xterm``` window when I hit 'run'.

<img src="https://github.com/sudb92/blog/blob/main/_posts/2023-09-10-img3.png"/>

    
* Parting comment: I do like the convenience of the IDE telling me all about function syntaxes as I type them, and the option of doing debug-testing while being a little spoiled. Using my own ```Makefile``` also gives me full control over how I compile the project, which was what originally dragged me away from half-cooked IDEs a long while back. I might stay with code-blocks awhile, maybe. Or check out one of the cousin efforts like ```kate``` or ```geany``` instead of powering through on ```nano``` and a web-browser for ROOT's class info :)
    
Tested using:

    - GNU Make 4.3
    - gcc 11.4
    - Code::Blocks 20.03
    - CERN ROOT v6.28.03
    - All running in Ubuntu 22.04 with a Linux-6.2 Kernel
