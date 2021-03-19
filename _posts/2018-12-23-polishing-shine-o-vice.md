---
layout: post
title: "Polishing Shine 'o Vice - Part 1: Saving crash"
feature-img: "assets/img/posts/shine-o-vice/shine-o-vice.jpg"
thumbnail: "assets/img/posts/shine-o-vice/shine-o-vice.jpg"
image: "assets/img/posts/shine-o-vice/shine-o-vice-f.jpg"
excerpt: Starting with the most urgent issue.
date: 2018-12-23 23:10:00 +0200
twitter: {card: "summary_large_image"}
tags: [Progress Reports]
---

With [Shine 'o Vice](https://gtaforums.com/topic/578670-shine-o-vice/) on the final stretch for Act 1 release,
I was asked for help regarding a critical issue which haunted the team for a long time:

> Currently, saving the game works just fine. However, when a save file is loaded, almost instantly but sometimes after a couple of minutes, an unhandled exception occurs.

I received the most up to date development build and quickly set the game up. Sure enough, saving went fine but after restarting the game and attempting to load the very same save,
it crashes.

Game crashed in a function called `CVehicleModelInfo::PreprocessHierarchy`. This function is called when vehicle's model is initially loaded by the game and it performs the necessary setup --
extras, car parts, wheels. The exact cause of this crash: game tries to operate on a wheel model which hasn't been loaded. Further debugging revealed it to be a `wheel_rim` model.

Recall that in GTA III and GTA VC cars share wheel models, and they are defined as separate objects in one of game's IDE files:
```
objs
# wheels
237, wheel_rim, generic, 2, 20, 70, 0
238, wheel_offroad, generic, 2, 20, 70, 0
239, wheel_truck, generic, 2, 20, 70, 0
250, wheel_sport, generic, 2, 20, 70, 0
251, wheel_saloon, generic, 2, 20, 70, 0
252, wheel_lightvan, generic, 2, 20, 70, 0
253, wheel_classic, generic, 2, 20, 70, 0
254, wheel_alloy, generic, 2, 20, 70, 0
255, wheel_lighttruck, generic, 2, 20, 70, 0
256, wheel_smallcar, generic, 2, 20, 70, 0
#misc
257, airtrain_vlo, generic, 1, 2000, 0
end
```

However, in Shine 'o Vice these lines look somewhat different - notice first three entries have different IDs:
```
objs
# wheels
325, wheel_rim, generic, 2, 20, 70, 0
327, wheel_offroad, generic, 2, 20, 70, 0
433, wheel_truck, generic, 2, 20, 70, 0
250, wheel_sport, generic, 2, 20, 70, 0
251, wheel_saloon,  generic, 2, 20, 70, 0
252, wheel_lightvan, generic, 2, 20, 70, 0
253, wheel_classic, generic, 2, 20, 70, 0
254, wheel_alloy, generic, 2, 20, 70, 0
255, wheel_lighttruck, generic, 2, 20, 70, 0
256, wheel_smallcar, generic, 2, 20, 70, 0
#misc
257, airtrain_vlo,  generic, 1, 2000,  0
end
```

Where do those come from? Shine 'o Vice adds **one** new vehicle under ID 237, and the IDs which were picked as a replacement are based on [this very old (**and incorrect!**) tutorial](https://gtaforums.com/topic/191413-adding-not-replacing-extra-vehicles/).

A quick check revealed that removing said one added car and relocating wheel models to their stock IDs indeed fixes the issue -- so we can assume that this relocation causes crashes,
and the IDs suggested by this tutorial can be flushed down the toilet.

# Fixing the crash

Because only one car was added, only one wheel model needs to have its ID changed -- so let's start by reverting the other two to vanilla state:
```
objs
# wheels
325, wheel_rim, generic, 2, 20, 70, 0
238, wheel_offroad, generic, 2, 20, 70, 0
239, wheel_truck, generic, 2, 20, 70, 0
250, wheel_sport, generic, 2, 20, 70, 0
251, wheel_saloon,  generic, 2, 20, 70, 0
252, wheel_lightvan, generic, 2, 20, 70, 0
253, wheel_classic, generic, 2, 20, 70, 0
254, wheel_alloy, generic, 2, 20, 70, 0
255, wheel_lighttruck, generic, 2, 20, 70, 0
256, wheel_smallcar, generic, 2, 20, 70, 0
#misc
257, airtrain_vlo,  generic, 1, 2000,  0
end
```

Time to find out what causes the crash when IDs get relocated. Debugging revealed that offending `wheel_rim` was loaded by the game just fine,
but then it gets unloaded! A well placed data breakpoint reveals who releases the model: Save loading routine contains code which looks somewhat like this (in pseudocode):
```cpp
for ( int index = 300; index < 6500; index++ )
{
    if ( stModelReqs[index].uLoadStatus == 1 && pModelPointers[index]->m_pRWObject != nullptr )
    {
        CStreaming::RemoveModel(index);
        stModelReqs[index].uLoadFlags = 0;
    }
}
```

Notice it's **only** touching models with ID above 300, so only our relocated wheel model gets affected -- it is the only wheel to be unloaded, while the game doesn't expect a case like this
and crashes!

So it seems that in order to fix this, we need to find a free ID below 300. Thankfully, an answer lies in the same file, just above wheel model definitions:
```
# leave ID 240 - 249 to use in temp car components
# 240, car_door
# 241, car_bumper
# 242, car_panel
# 243, car_bonnet
# 244, car_boot
# 245, car_wheel
# 246, bodypart_A
# 247, bodypart_B
```

Despite the comment, code analysis reveals that IDs `248` and `249` are in fact not used (just like the other part of this comment hints),
and we can recycle them! We can move `wheel_rim` to ID `249` and test it.

Does it help?
<p align="center">
<img src="{% link assets/img/posts/shine-o-vice/sov-screen.jpg %}"><br>
</p>

Yes, it indeed does! With this one little change, Shine 'o Vice can save its progress without much issue -- our job is done!

For the next time, I'll look into some less pressing issues the game has -- mainly SCM issues and outdated dependencies (used CLEO plugin dates back to 2010...).
Hopefully you'll see it in a Part 2 soon!