## Notes on Assembling a PicoCluster 20H

This is absolutely a first-world problem, as the 
[picocluster 20H](https://www.picocluster.com/products/pico-20-raspberry-pi4-8gb) is expensive, but I ran 
into some problems when building the cluster. In the hope of sparing others my pain, I'm noting those here. 

These all arise from one central issue: as of the writing of this post, there is no construction guide for the 20H 
on the picocluster website. So I had to wing it a bit, looking at pictures of an assembled cluster and at other 
20-node cluster assembly guides. Hopefully, by the time you read this, there will be official assembly guide from 
picocluster themselves.

### Order of Operations

Here is the final order of operations I found useful:
 1. Build the 4 stacks of Raspberry Pi's like in the [instructions here](https://www.picocluster.com/blogs//picocluster-assembly-instructions/assemble-rpi4-5h-board-stack) 
 2. Attach two of the switches to the acrylic boards 
    1. There is a correct orientation for these boards. The notch at one edge goes on the *bottom*, and the holes 
nearest the edge are the *back*. The switch attaches to the *left* side of that, if looking at the "front" edge.
 3. Attach the power supplies to the bottom acrylic board 
    1. There is a correct orientation of this board also. There are only 3 holes in the power supplies provided, 
two at one end, one at the other. When screwed into the bottom board, they are supposed to be "back to back". I.e. 
the open, mesh parts of the power supplies are supposed to face *out*. You may have to flip the bottom board over to 
make sure the holes line up properly to ensure this arrangement.
 4. Bridge together the power supplies 
    1. You are connecting the "L", "N", and "Ground" (look up this symbol if you're not familiar with it) points together
on the two power supplies. There are 3 cables provided for this, but they're fairly short. I ended up having to route
the cables in between the power supplies to get them to fit, which meant doing ground first, then N, then L.
 5. Detach the power cables from the power plug, taking note of which color goes to which connector. Attach them to one
power supply. 
    1. Take note of which side you connect it to: this side will have to be the one with the plug once it's 
assembled.
    2. Make sure you connect the Blue cable to "L", the Yellow cable to "N", and the Green cable to "ground".
 6. Connect the board stack power cables (the ones with open wires on one end and a barrel power plug on the other) to 
the power supplies. 
    1. You probably already know this, but Red goes to "V+" and Black to "V-". There are enough screw points on the power 
supplies that I could attach one stack cable to its own set of screw points.
 7. Attach the power and fan to the one side acrylic panel that has holes for it. Attach a network switch to inside of 
the panel.
    1. "Inside" is defined by the etching on the panel - the etched side is "out"
 8. Attach the fan to the inside of the remaining acrylic side panel, along with remaining switch.
 9. Attach the power cables, that you removed from the plug earlier, back to the plug on the side panel. 
    1. Again, be sure that the cables go back to the connectors you took them out of. This is very important.
 10. Attach the side acrylic panels to the base.
 11. Put the 2 switch boards that remain in to the case, attaching them to the bottom panel.
 12. If you are using SD cards for the cluster, insert those *before* putting the pi stacks into the box
     1. It will be tight enough inside the box that you won't be able to reach the SD card slots on the once they're in.
     2. I found it useful also to mark one Pi in one stack with a sticker, so that pi became "0", and numbered from there.
 13. Attach the 4 Pi stacks to the bottom panel.
     1. The USB cables will start to get tight here.
 14. Connect the two front panels to the base panel and to the side panels.
 15. Pull the switch power cables (the barrel power plug from the usb hub at the top of each stack) through the holes 
in the front panels and connect them to the switches.
 16. Connect the stack power cables from the supplies up to the top board of each of the 4 stacks
 17. Connect the fan power supplies to the prongs on two of the top boards on the pi stacks.
 18. Attach the top board. 
 19. Connect each Pi to the switch next to it with the short ethernet cables provided. 
     1. If you are making this a standalone cluster (which I am not), connect the switches on one side together with 
the medium-length ethernet cable, and then connect the two sides together with the longest ethernet cable.

You have no idea how many times I had to disassemble the cluster partway because I did things in the wrong order.
