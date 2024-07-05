## Embedding CERN ROOT classes in Lua5.1 via tolua++


* [```tolua++```](https://web.tecgraf.puc-rio.br/~celes/tolua/tolua-3.2.html) is a wonderful tool that allows us to expose a small component of a larger C/C++ library like CERN ROOT to create a compact interpreter with friendlier syntax, no explicit typing, and all the other benefits that come with lua embedding - not least, faster prototyping.

* Unfortunately, there's a small learning curve when it comes to implementing a working example of how said embedding is done. Upon encountering a helpful 
stackoverflow answer [here](https://stackoverflow.com/questions/4482518/setting-up-an-environment-for-an-embedded-lua-script), I retried this approach to finally
meet success. The following steps were taken to fill a ```TH1F```, live-update a ```TCanvas``` object within a Lua loop, save the final result to a png, and exit.

### Basics:
* ```tolua``` and ```tolua++``` are present in Linux repositories, in my Ubuntu 22.04LTS installing `tolua++` was simple as
```
sudo apt install libtolua++5.1-dev
```
* ```tolua``` and ```tolua++``` require a package file that has definitions of the requisite class features that need to be exposed. In what's below, I'll stick to tolua++5.1 and C++ classes alone,
 since these are most relevant for ROOT.

### Action:
* The following file ```root1.pkg```exposes all the features that I plan to use from ROOT's class hierarchy : the ```TH1F``` to be filled, the ```TCanvas``` and ```TApplication``` for live updation
within a loop. Note that I've also included all the classes that the exposed member functions require. Lua already knows about the standard C/C++ types. It is also important to note the ```$``` sign
before all the ```#include``` lines.

```C
$#include <TH1F.h>
$#include <TCanvas.h>
$#include <TVirtualPad.h>
$#include <TApplication.h>
$#include <TObject.h>

class TH1F {
public:
    TH1F(const char* name, const char* title, int nbinsx, double xlow, double xhigh);
    void Fill(double x);
    void Draw();
};

class TCanvas {
public:
    TCanvas(const char* name, const char* title, int wpx, int wpy, int ww, int hh);
    void SaveAs(const char* filename = "", const char* option = "") const;
    TVirtualPad* cd(int sub=0);
    TObject* WaitPrimitive(const char* pname="",const char *emode="");
    void Update();
    void Modified();
};

class TApplication {
    TApplication(const char* name, int* argc, char** argv, void* options=nullptr, int numoptions=0);
};
```
* ```tolua++5.1``` can help us convert the above pkg file into a pair of C++ header/source files, that can be used to compile a custom lua5.1 interpreter that 'knows of' the extra classes
exposed by root1.pkg, in addition to all the capabilities/definitions a regular lua5.1 interpreter brings (such as ```os``` and ```io```, and other capabilities). This is accomplished by running 
in bash the following line. The output file names are arbitrary chosen as ```hstub.cpp``` and ```hstub.hpp``` keeping with the demo example link.
``` tolua++5.1 -H hstub.hpp -o hstub.cpp root1.pkg ```

* If all goes well, the above step promptly creates the two files. Now, we prepare a ```lua5.1``` interpreter by creating the following ```main.cpp``` file and compiling it. I have smoothed out
the use of ```extern C``` via the more elegant syntax of including ```lua.hpp``` that comes with all modern versions of lua in package managers. The only additional step here is the inclusion
of ```hstub.hpp``` and the function ```tolua_root1_open()``` defined in ```hstub.cpp```, to load all the additional definitions for use with the lua interpreter. Refer to the ```tolua``` manual for details on these.

```C
#include <tolua++.h>
#include "lua.hpp"
#include "hstub.hpp"

int main()
{
    lua_State *L = lua_open();
    luaL_openlibs(L);
    tolua_root1_open(L);

    if (luaL_dofile(L, NULL) != 0)
        fprintf(stderr, "%s\n", lua_tostring(L, -1));

    lua_close(L);
    return 0;
}
```
* Now to compile the above, we finally require ```g++```, complete with all the necessary includes of both Lua and CERN ROOT, and all the libraries. The compilation/build recipe here is:
```bash
g++ -I/usr/include/lua5.1 `root-config --cflags` main.cpp hstub.cpp -ltolua++5.1 -llua5.1 `root-config --glibs` -o root1
rm hstub.*
```
* The above process creates the executable ```root1``` that is our lua interpreter. I delete the ```hstub``` pair since we no longer need them. We now create a Lua5.1 script that can be sent to the interpreter, that does what we set out originally to do:
make a ```TCanvas```, fill a ```TH1F``` in a loop within it, live update it, and wait for a ```Ctrl+C``` before exiting. I'm calling the file ```roottest.lua```, and it looks like this:

```lua
print("Calling ROOT via a lua script..\n")
app = TApplication("app",0,0)
c1 = TCanvas("c1","c1",0,0,800,600)
h1 = TH1F("test","test",100,0,1000)
c1:cd()
h1:Draw()
i=0
repeat
    h1:Fill(math.random(0,1000))
    c1:Modified()
    c1:Update()
    i=i+1
until i==1000
-- c1:SaveAs("test.png")
repeat 
    do end -- empty block, i.e. a 'nop'
until c1:WaitPrimitive()==0 
--only true when Ctrl+C is sent or 'Quit ROOT' is chosen on TCanvas
print("...done.\n")
```
* Elegant, compact, quick-to-prototype. All things Lua is good at :) Sending the above file to the executable ```root1``` as
```bash
./root1 < roottest.lua
```
should promptly bring up a live-updating histogram. Nice!
