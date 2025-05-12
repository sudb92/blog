## CERN ROOT and Keyboard Event Handling in TCanvas


* As part of a longstanding quest, I have always wished to possess the capability to use single-keypress events on a ```TCanvas``` or a ```TBrowser``` in CERN ROOT to trigger events.
* Handling a single keystroke, _a la_ ```getch()``` of Borland C++ vintage, is straightforward enough, by using a construct like
  ```while(gPad->WaitPrimitive());```
  peppered within the macro/program, which waits for a keypress in the active macro. If one wants the keypress while a particular ```TCanvas``` was active, that was straightfroward too, as
  ```while(canvas->WaitPrimitive());```
* What used to elude my understanding was how to parse the return value of ```gPad->GetEvent()``` to recognize keystrokes in the first place, and secondly, what would allow me to get the result of the keypress.
* My very crafty coworker over at https://github.com/jmattspartacus told me the wonderful (but very cryptic) trick a few days back: type-casting to ```char``` the results of ```gPad->GetEventX()``` gives the ascii value of keypress events! This is a little infuriating, since GetEventsX() is used to track mouse X positions in events! Wth, ROOT?
* It turns out, one can also cast to ```short``` the results from ```gPad->GetEvent()``` to filter out keypresses by searching for the number 24. (Which is 42 flipped, makes sense.) ```gPad->GetEventX()``` and ```gPad->GetEventY()``` both store the same number, which when cast to ```char``` return the ascii value of the key pressed.
* All in all, the following macro if run with the usual
```bash
root -l -x -q track_keypress_in_canvas.C
```
should bring up a ```TCanvas``` with a histogram. Once you hover over the canvas, you can cycle through ROOT's default colors by pressing 'n' and 'p'. The title to the canvas will show the current color index.
* Fun times! This use-case of ```TCanvas::AddExec``` will make a lot of basic user-interfacing a breeze to work with. And to think, this issue needed needless reverse engineering! Hope the people over at CERN ROOT advertise this, instead of hiding it away.

```C

int cc=0;
TH1F h1("h1","h1",800,-400,400);

void myexec()
{
   // get event information
   short event = gPad->GetEvent();
   int px    = gPad->GetEventX();
   int py    = gPad->GetEventY();

   // some magic to get the coordinates...
   /*double xd = gPad->AbsPixeltoX(px);
   double yd = gPad->AbsPixeltoY(py);
   float x = gPad->PadtoX(xd);
   float y = gPad->PadtoY(yd);*/

   //if (event==1) { // left mouse button click, if needed
      //return;
   //}
   if(event==24 && (((char)px =='n' || (char)px=='p'))) { //24 is keypress, px, and py are assigned characters
       //std::cout << (int)event << " " << (char)px << " " << (char)py << std::endl;
       if((char)px=='n') cc++;
       else cc--;
       h1.SetTitle(Form("Color:%d",cc));
       h1.SetFillColorAlpha(cc,0.4);
       gPad->Modified();
       gPad->Update();
       return;
   }
}


void track_keypress_in_canvas()
{
   gStyle->SetOptStat(0);
   h1.GetXaxis()->SetRangeUser(-5,10);
   TF1 f1("gauss",Form("%f*TMath::Exp(-(x-%f)*(x-%f)/(2*2.*2.))",1000.,2.,2.),2.-5,2.+5);
   h1.FillRandom("gauss",1000,nullptr);
   h1.SetLineWidth(2.0);
   h1.GetXaxis()->SetTitle("Hover mouse, press 'n' or 'p' to cycle colors");
   h1.Draw();
   gPad->Modified();
   gPad->Update();

   // add exec
   gPad->AddExec("myexec","myexec()");
}

```
