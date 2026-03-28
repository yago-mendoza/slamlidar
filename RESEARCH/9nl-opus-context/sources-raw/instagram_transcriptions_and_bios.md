# Instagram Transcriptions & Bios — @9nl

> Contenido original sin modificar, reorganizado por tema/post.

---

## 1. Rotational Cup & Optical Center Alignment

If your rotation axis isn't through the optical center, your point cloud map breaks. So why did that move anyway? This rotational cup is probably one of the most iterated parts. So you can see, we have many, many different versions, from different materials to different iterations of the height. Another big issue that I ran into was, since it doesn't weigh too much, however, it does weigh enough to create a little bit of deflection of this. I spent a couple days fighting what I thought was runout issues in the shaft, which actually turned out to be just deflection of this cup here. So I added some simple supports here. It's not ideal for the wire routing, but it gets the job done and helped reduce all of the issues I had there. Initially, what I wanted to do is have the optical center of the laser in line with the rotational axis. That would make all of the de-skewing of the point cloud significantly easier. But whenever I did that, this rotational mass would be unbalanced, and that would just cause even more issues. So it's offset by about five millimeters, and this offset is accounted for in the de-skewing map. If your rotation axis isn't through the optical center, the point cloud map breaks.

---

## 2. Motor Upgrade & Custom Point Cloud Viewer

A few updates on the rotating LiDAR build. First, swapped the gimbal motor out from a 2804 to a 4015. This gives me about three times the torque. I can run it a whole lot slower, generates less heat. It's definitely every kill in the size, but the control is so much better. The old one would generate so much heat that it would melt my gears, and they would get so warped that it wouldn't even spin anymore. I've also been building my own point cloud viewer so I can fly through this scan without waiting on cloud compare or Arvis. I needed something faster because I'm spending a lot of time iterating the de-skewing algorithm. This also gives me the ability to fly through the point clouds, which gives a much better first-person view kind of feel. I intentionally took a step back from the SLAM integration to really nail this part down. If the points aren't landing where they should, nothing downstream really matters. Once de-skewing is solid, the SLAM will go back in. More cubing will follow along.

---

## 3. VLP-16 Sensor Selection

The Roomba is smarter than you think. This little puck fires 300,000 laser points every second in insanely dark darkness, measured down to the centimeter, and it's going to be the eyes of my cave mapping platform. There are tons of LiDAR sensors on the market, so why did I choose the VLP-16? First, availability and cost. VLP-16 has been in production since 2014. The autonomous vehicle industry went through thousands of these sensors during development and testing. Now, a lot of these companies have moved on to newer sensors, upgraded their fleets, or in some cases, shut down entirely. That means used VLP-16s show up on the secondary market all the time. I paid $321 for mine. The second reason I chose the VLP-16 is the ecosystem and support. The VLP-16 has mature, battle-tested ROS and ROS2 drivers. The Velodyne driver package has been around for years. When you're trying to debug your SLAM algorithms, and your map is drifting, or your loop closures are failing, the last thing you want to do is also be questioning whether your sensor's driver is giving you good data. There's also just a ton of community knowledge out there. Whatever problem you hit, someone has probably had that problem in the past, and already solved it. That's already written about it. Third, durability. This thing is designed to be mounted on top of vehicles driving all around San Francisco, rain, fog, thunderstorms, vibration. The build quality is pretty impressive. IP67 rated, operating temperature from around 10 to 60 degrees Celsius. For field work in caves where you might encounter humidity, huge temperature changes, the occasional bump against a rock wall, that durability matters. Now, I'll be honest, if I started from scratch or had an unlimited budget, I'd probably choose a different sensor. But LiDAR sensors like the mid-360 offer incredible point density per dollar. They use a non-repetitive scan pattern that fills in over time, which actually works really well for SLAM applications. Downsides, scan pattern can be a bit tricky to work with, and they're a lot more expensive. Ouster sensors like the OS1 give you more vertical channels, up to 128, and higher resolution, plus they output camera-like images alongside the point cloud. But they're way more expensive and draw a bit more power. Budget options like the RP LiDAR series can be had for a few hundred bucks, and for simple 2D mapping, they're great, but for full 3D coverage in complex environments, you're limited. For this specific project, the VLP16 hits the sweet spot with capability, cost, ecosystem support, and build quality.

---

## 4. Custom PCB / Carrier Board (Teensy 4.0)

I actually got a delivery from JLPCB, and it was this very simple board I designed for the Teensy 4.0. So this is the board. It's very simple. You get the mounting area for the TNC right here, and then breaks out and vectorizes all of the GPIO that I'm currently using. So we've got power ground just to power the TNC board. This connector here, three pin, ground, serial RX, TX, this transmits the encoder serial data to the Pi 5. This right here connects directly into the simple FOC motor controller. So all of this right here, here, here, and here is GPIO that I'm not using. Figured just in case I need it in the future, I can solder some wires in right around here and use it if needed. This right here, this is the connection for the motor encoder. It's the AS5600, I think. Yeah, that's it. So this is what it looks like completed. Not the prettiest, very simple. Just breaks out with all the GPIO that I'm currently using and then breaks out all the unused stuff, just in case I need it in the future. CNC here. I made this like this, so if I ever need to pull it out or replace it, it's a little bit easier. Got the motor here. This is the encoder plug. Connects there, pretty simple. And then I've got the simple FOC V1, which will just pop right here, done. And then I have this wire, which is serial communication. It goes to the Raspberry Pi 5. The data that gets transmitted over this is the serial encoder data. The reason for this is for the point cloud descewing. So the encoder provides the angle to the ROS2, and the power for the TNC right here. Should have got a delivery from JL.

---

## 5. Slip Ring

Shooting lasers 300,000 times a second, inches from my face. Should I be worried? You've probably seen everyone keeps asking how the wires don't get tangled up. The secret is this little guy, a slip ring. The wires run down through this tube to the slip ring at the base. The top half spins with the LiDAR sensor, the bottom half stays put, so nothing ever gets twisted. Inside, you've got these rotating metal rings with little brushes that just ride along and stay in contact. Each ring gets its own circuit, so power and data keep flowing while the whole thing spins. It's the same thing you find in wind turbines, radar dishes, even CT scanners. Basically, anything that needs to spin forever without strangling itself. This one has 12 circuits, but in this application, I'm just using power, ground, and Ethernet. The rest just snipped off and pushed out of the way. For 30 bucks, it's a job done. Everyone keeps asking how the wires don't get...

---

## 6. Project Overview / Pitch

This thing fires 300,000 laser pulses every second. It works in total darkness. It doesn't need GPS. There's commercial systems that can do this, but they cost about 40 grand. I built this system for less than a grand. GPS doesn't work underground, like, at all. Caves, mines, parking garages, your phone is basically useless. And even outside, GPS only tells you where you went. It's just a line on the map. What I'm after is quite different. I want to capture the actual environment, full 3D models, every surface, every dimension, the complete physical space. There are companies that have figured this out. They make portable scanning systems that use SLAM, simultaneous localization and mapping. It tracks its own position while scanning everything around it. It's pretty sweet. The problem is, they're like 40 grand, and they're all proprietary. You can't see how any of it works. And it's not like I could buy one of these and reverse engineer it. I don't have that kind of money. None of these companies are sharing their secret sauce. So I figured, all right, let's see how far I can get building one myself with open source tools and a way smaller budget. This thing fires 300,000... Look at this.

---

## 7. Instagram Captions & Bios

Ideally, the rotation axis runs straight through the optical center of the VLP-16... clean math, clean point cloud. But the cup geometry pushed the COG far enough off-axis that vibration and imbalance made that impossible.
Had to offset the axis, which means accounting for that in the SLAM pipeline. Engineering is just a series of compromises.

14 million points. Homemade scanner. Zero commercial software. This is the point cloud registration step of my cave mapping pipeline: taking two separate LiDAR scans and aligning them into a single continuous map #robotics

What I've learned building my own LiDAR cave mapper (nobody tells you this stuff):
Getting SLAM reliable is way harder than it looks. Divergence and drift are constant battles, and one bad loop and your whole map falls apart.
Simulation gives you confidence. Real world testing humbles you. Always.
And data timing? If your sensor streams aren't synced to the millisecond, you're not mapping anything, you're just making art.

Ideally, the rotation axis runs straight through the optical center of the VLP-16... clean math, clean point cloud. But the cup geometry pushed the COG far enough off-axis that vibration and imbalance made that impossible.
Had to offset the axis, which means accounting for that in the SLAM pipeline. Engineering is just a series of compromises.

Motor upgrade + building custom tools to get the deskewing right before moving back to SLAM. No shortcuts.

Built a custom carrier board for my rotating LiDAR Scanner.
Designed this PCB to integrate a Teensy 4.0 with SimpleFOC motor control for my cave mapping project. Main goal? Eliminate the rats nest of jumper wires and breakout boards.
Clean power delivery, compact footprint, and way fewer points of failure.
When your prototype wiring becomes the problem, you design it out.
Full build series coming soon
#PCBDesign #robotics #ai #engineering

From a sensor on my desk to mapping in real time.. 6 months of progress and still a ways to go.

Unboxing the board for my LiDAR Scanner Motor Controller. More info coming soon! #techdiy #engineering #robotics #ros2

Building a SLAM stack around a VLP-16
The VLP-16 internally spins ~600rpm but statically mounted only captures a slice of your environment. So I added another axis of rotation now it can "spherically" scan an area. #robotics #engineering #ros2 #LiDAR

made a DIY rotating LiDAR SLAM system #robotics #LiDAR #3dscanning #engineering
