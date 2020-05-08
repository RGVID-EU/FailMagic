## FailMagic – Fixing your Intensity and DeckLink PCIe cards

If you ever try to install more than one DeckLink or Intensity PCIe
card you might stumble upon a problem – each card will work totally
fine separately, but once you install more than one card all of them
will stop working. This configuration is officially supported by
BMD and should work, but in some cases it just does not. Why?

### The problem

Luckily, we were not the only ones having this issue. Thanks to
[helpful responses on the forum](https://forum.blackmagicdesign.com/viewtopic.php?f=18&t=97512&sid=652421e7ead5ca4d921850d65cf291b8),
we were able to quickly deduce that the issue is in fact in PCIe cards
made by BMD. And indeed, after a few minutes with a soldering iron
we were able to make all our cards work at the same time.

In other words, good news! You can easily fix the design mistake
yourself and get your cards to work. This will likely void your
warranty, which is unfortunate given that BMD knowingly sold you a
flawed card. They are also suspiciously silent about the issue, we
even asked their support about it but received no response
whatsoever. Interesting.

#### What is going on?

Check out the [pinout of a PCIe slot](https://en.wikipedia.org/wiki/PCI_Express#Pinout).
There is not much that can go wrong there. For example, SMBus pins are
not even routed on most of the cards, so you can ignore them. You are
left with just data lines, power pins, PERST and a few other
pins. PERST is the problem. When investigating what PERST is
supposed to do, you can stumble upon
[this helpful ticket](https://www.ohwr.org/project/spec/issues/17).
Basically, it is a discussion about a design of some PCIe card, and the
idea is that a 10kΩ pull-down resistor on PERST is too much, and you
need a higher value (like 100kΩ or so).

We located the PERST pin on our Intensity Pro 4K and DeckLink Mini
Recorder 4K cards, followed the traces and found pull-down
resistors on both cards. Their value? **10kΩ**. Bingo!

#### But why can each card work fine separately?

You may be wondering why it works fine in some cases. For example,
multiple BMD cards do indeed work OK on some motherboards, but not
on others. And each card works fine individually, at least in most of
the setups. Here is where it gets interesting.

In some configurations, the PERST line is shared between multiple PCIe
slots! This is the case with some motherboards where PCIe slots do not
have native CPU lanes, but rather go through the chipset. This can
also happen if you are using
[PCIe bifurcation](https://linustechtips.com/main/topic/1040947-pcie-bifurcation-4x4x4x4-from-an-x16-slot/).
In other words, there are many valid reasons why you can end up with
a shared PERST line in your setup.

However, when this happens, it means that both pull-down resistors are
now in *parallel*. We can calculate the equivalent resistance in such
case: ``1/R = 1/R₁ + 1/R₂``. For two resistors that are 10kΩ we get
5kΩ equivalent resistance. If 10kΩ pulldown is wrong by itself, then
5kΩ is even worse. That is, the more BMD cards you have installed on
PCIe slots with a shared PERST line, the more likely you are to face
the issue. For most users the threshold will be at 2 cards. Essentially,
it pulls the PERST line to ground without letting it to go up,
meaning that your cards will always be in the reset state.

#### Really?!

Yes! You can test this by putting one of your BMD cards into the main
GPU slot (and placing your GPU into any other PCIe slot that can fit
it). You will see that both Decklink/Intensity cards work fine in this
configuration. That said, it is very likely that you do not want to use
a setup like this in production, because using your GPU in any other
slot except the one with x16 CPU lanes can possibly affect your GPU
performance. If you need another confirmation, you can try using any
other properly designed card together with your BMD cards, and
everything will work fine, it is only the cards made by BMD that
“conflict” with each other.

#### How to fix it

Fixing it is easy, you just need to remove the pull-down resistor on
one of the cards (or all of them, which is what we do). Just find the
resistor and use your soldering iron to remove it.

![beforeafter](https://user-images.githubusercontent.com/5507503/81393991-e7fc9c00-9129-11ea-8b6c-9887c983d452.png)

#### PERST pull-down resistor placement on different cards

Here are some photos of the cards we fixed.

| Card | Photo |
|:----:|:-----:|
| Intensity Pro 4K | ![bmd_intensity_pro_4k](https://user-images.githubusercontent.com/5507503/81394001-eaf78c80-9129-11ea-8252-4b86a3c65d93.JPG) |
| DeckLink Mini Recorder 4K | ![bmd_mini_recorder_4k](https://user-images.githubusercontent.com/5507503/81394002-ecc15000-9129-11ea-9708-2a89c6b5f8fc.JPG) |

#### Is there any other way to fix it?

No. It is a hardware issue. You cannot fix it with a driver or
software update. Once you plug in both of your cards they end up in
the reset state, the motherboard is unable to enumerate them. There is
nothing you can do besides removing or replacing the resistor that
should not have been there in the first place.
