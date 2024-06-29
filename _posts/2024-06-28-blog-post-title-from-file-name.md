## Simple demo of zlib API


* As part of my latest data analysis, I have been tasked with looking through a binary file compressed in the .gz format. The original analysis scheme looks through the uncompressed file - one
record at a time as follows (I have suppressed the original use-case for illustration purposes):

```C
// uncompressed.cxx
#include <cstdint>
#include <stdio.h>
#include <fcntl.h>
#include <vector>
#include <signal.h>
#include <unistd.h>
#define MAXSIZE 32768

struct header {
  int type; int length; long long timestamp;
};

struct payload {
  int type;
  int errcode;
  int energy;
  int timestamp;
  int channel;
  double waveform[MAXSIZE];
};

bool quit=false;
void handler(int signal) {quit=true;}

int main() {
    /**
      Binary file input.dat contains data packed as header-payload-header-payload. The size of the payload is contained in the header, along with the timestamp and type information.
      We extract only type-1 data in the following, and make a vector of all the unique 'channels' present in the type1 payloads. The files can be huge, so it helps to be able to break out of
      the loop using Ctrl+C.
    */
    header head;
    char buf[32768];
    payload t1d;
    int fd = open("/path/to/input.dat",O_RDONLY);
    if(fd == -1) {
        printf("File data/input.dat not found! Exiting..\n");
        return -1;
    }
    std::vector<int> crystalids;
    int readstatus = 1;
    signal(SIGINT,handler);
    while(readstatus!=0) {
       readstatus = read(fd,(void*)&head,sizeof(head));
       if(head.type==1) {
        readstatus = read(fd,(void*)&t1d,head.length);
        if(std::find(crystalids.begin(),crystalids.end(),t1d.channel)==crystalids.end()) {
          crystalids.push_back(t1d.channel);
        }
        type1+=1;
       }
       else {
          assert(head.length < 32768);
          readstatus = read(fd,(void*)buf,head.length);
          typemisc+=1;
       }
       recnum += head.length;
       if(recnum%100'000==0) printf("\rProcessed: %lld MB",recnum/1000'000LL);
       if(quit) break;
    }
    for(auto& x: crystalids) std::cout << x << " "  << std::endl;
    close(fd);
    return 0;
}

```

Now, performance considerations aside, it is a pain in the neck to have to ```gunzip -k``` every input.dat.gz file just to be able to look through and do the above operation, then delete the result afterwards.
Fortunately for us, using the ```zlib``` library is said to be both less messy AND better optimized execution-time-wise. To directly read from input.dat.gz and do the same operation as above, simply use

```C
//zlibdemo.cxx
#include <zlib.h>
#include <vector>
#include <iostream>
#include <signal.h>
#include <assert.h>
#define MAXSIZE 32768

struct header {
  int type; int length; long long timestamp;
};

struct payload {
  int type;
  int errcode;
  int energy;
  int timestamp;
  int channel;
  double waveform[MAXSIZE];
};

bool quit=false;
void handler(int signal) { quit = true; }

int main()
{
    header header;
    payload g1;
    uint16_t junk[32768];
    std::vector<int> crystalids;

    gzFile infile = (gzFile)gzopen("/path/to/input.dat.gz", "rb");
    gzrewind(infile);
    signal(SIGINT,handler);

    while(!gzeof(infile))
    {
        int len = gzread(infile, &header, sizeof(header));
        switch(header.type) {
            case 1: gzread(infile, &g1, header.length);
                    if(std::find(crystalids.begin(),crystalids.end(),g1.channel)==crystalids.end()) {
                        crystalids.push_back(g1.channel);
                    }
                    break;
            default: assert(header.length<32768);
                     gzread(infile, &junk, header.length);
        }
        if(quit) break;
    }
    for(auto& x: crystalids) std::cout << x << " "  << std::endl;
    gzclose(infile);
    return 0;
}
```
Of course, the files are compiled as:
```
g++ uncompressed.cxx -o uncompressed 
```
for the original file, and
```
g++ zlibdemp.cxx -o zlibdemo -lz
```
for the zlib version. Everything in the first version is contained in standard C/C++ libraries, while the second version needs us to link to zlib's shared libraries. 
