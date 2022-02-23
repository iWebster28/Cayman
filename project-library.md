---
layout: default
title: Projects
description: Welcome to my Project Library!
---

## Hackathons
* * *

### MLH - MakeUofT 2022 - CarJam

![carjam_frontend](./assets/Project_Pictures/carjam_frontend.png)

[Devpost Submission](https://devpost.com/software/carjam)

#### What is CarJam?

CarJam is a way to enjoy music on the road without the stress of creating a playlist. 

Our web app listens to your ambient environment and suggests a song to play based on the mood.  

For example, maybe you're carpooling with 6 people and the car is *positively vibing*. Our app will suggest an upbeat, happy song to play. (we query the Spotify API with "liveness" and "energy" parameters).  

On the flipside, let's say you just had a bad day and you're venting with your coworker on the way home. CarJam will suggest a downbeat, melancholic or sad song to play.  

By listening to your ambient environment, CarJam matches your energy while introducing you to new music.  
#### Technologies 
Backend:   
- Node.js  
Frontend:  
- React  
- [React-mic](https://www.npmjs.com/package/react-mic)  

APIs: 
- [GCS Text-to-Speech](https://cloud.google.com/text-to-speech)  
- [GCS Natural Language - Sentiment Analysis](https://cloud.google.com/natural-language/docs/analyzing-sentiment)   
- [Spotify](https://developer.spotify.com/documentation/web-api/), ["Get Recommendations" Endpoint](https://developer.spotify.com/console/get-recommendations/)  
  
HW:  
- Rasberry Pi 3B+, Mic, Speaker  

#### Useful Resources I Found
- [Fetching blobs](https://stackoverflow.com/questions/11876175/how-to-get-a-file-or-blob-from-an-object-url)  
- [Sending blobs to express](https://developers.deepgram.com/blog/2021/11/sending-audio-files-to-expressjs-server/)  
- [Multer + Fetch with Blobs](https://stackoverflow.com/questions/39677993/send-blob-data-to-node-using-fetch-multer-express)  
- [Blobs - General](https://developer.mozilla.org/en-US/docs/Web/API/Blob)  
- [Audio - Unused](https://developers.google.com/web/fundamentals/media/recording-audio)  
#### How does it work?

CarJam was built as a React app hosted on a Rasberry Pi 3B+. A local Node.js server was hosted on our network.   

My original flowchart:  

Pi Mic → Frontend → encode/send audio to backend → backend uses audio to make API calls for sentiment analysis (GCS) → this data should be cached  

Frontend requests a new song to play → backend uses last-cached sentiment data to make a spotify API call to get a song → backend sends this song url to the frontend → frontend updates the “next up” song.  

#### Challenges I faced

Pending  

#### What would I change for next time?

##### Efficiency and Performance
- Chunk audio data. Potentially communicate with [websockets](https://medium.com/google-cloud/building-a-client-side-web-app-which-streams-audio-from-a-browser-microphone-to-a-server-part-ii-df20ddb47d4e)  
- Cache audio data on AWS S3 or another cloud storage service.  

##### Metadata
- Extract more metadata from audio data to determine music to play, such as:   
  - dB level  
  - number of unique voices  
  - prevalent tones of voice  
- Extract other environmental metadata:  
  - Speed of car  
  - Date and time, e.g. Day/night time  

##### Inference
- Avoid implicitly connecting the Google Sentiment Analysis directly to Spotify API query parameters. Instead, we could train an ML model to decide which parameters to "connect" together.

### MLH - MakeUofT 2021 - wavechord

[Devpost Submission](https://devpost.com/software/wavechord)  

The idea behind wavechord was to build a music creation tool that makes it easy for non-musicians to make music. Using a Kinect as a camera, wavechord detects hand position and plays corresponding chords that sound good together.  

I worked on setting up the Raspberry Pi 3 B+ with Kinect support, OpenCV, and PyAudio for audio synthesis. I learned a lot about using cmake on a limited-RAM systems like the Pi - configuring swap space is extremely important! I was able to build OpenCV with FFMPEG support, which allowed us to grab video from the Kinect - however, this was limited to 8-bit grayscale. I was able to use other utilities like guvcview to read the depth and colour information, but I could not figure out how to capture video using OpenCV in these modes. I tried compiling libraries such as libfreenect for Python, but they did not end up working properly (ioctl errors). In the end, I ended up using fswebcam to capture colour pictures, which are read into OpenCV periodically (e.g. at 1 FPS). In the future, I may try compiling openkinect again.  

After finding some hand-detection code and implementing it with the kinect, I explored implementing various Python audio libraries. I tried to use [Pyo](http://ajaxsoundstudio.com/software/pyo/) but ran into many errors with the audio output on the Pi. I settled with using [PyAudio](https://pypi.org/project/PyAudio/) which ended up throwing a few less errors, and it actually produced audible output. I did some quick music math to allow the user's hand motion to play chords.  

Samples for individual notes are generated using the ```sin``` function in numpy. Samples for chords are generated using the following expression:   ```sum([np.sin(2*np.pi*np.arange(fs*duration)*f/fs).astype(np.float32)/self.n for f in freqs])```,
```
where:
fs = sampling frequency (44.1 kHZ)
duration = length of note in seconds
f = frequency of note (Hz)
n = number of notes in `freqs` list (chord)
```  
The code above sums up the frequencies of all the individual notes in a chord. Importantly, we have to average the frequencies such that their sum does not distort the output at any instant in time. I made use of a formula to generate key frequencies corresponding to note numbers on the piano. ```f = 2 ** ((i - 49) / 12) * 440``` calculates the frequency `f` of the `i`th note on the piano, where the lowest note (A0) is the `0`th note. I had tried to use this formula a year earlier in an [FPGA Synthesizer project](https://github.com/iWebster28/Music), but I could not get it working - I'll have another go at it some day! 

Converting the screen coordinate system to a specific range of piano keys was an interesting problem. Alexis came up with ```root = int(curr_x / x_max * (key_max - key_range) + key_range)``` to set ranges for which notes on the piano can be played with your hand over the kinect. We then restricted the ranges to correspond to only certain scale degrees (i.e. I, IV and V) which are very commonly used in Western music.  

Check out the devpost submission above to see the project in action!  

### Hack the North 2020++ - CIA - "Car Impact Associator"
![cia](/assets/Project_Pictures/cia_ui5.png)  
![cia](/assets/Project_Pictures/cia_ui4.png)  

[Devpost Submission](https://devpost.com/software/car-impact-associator-cia)  

CIA, or Car Impact Associator, aims to convince the general public to make economically and environmentally informed decisions on the purchase of automobiles. Our team pulled together a React App that allows the user to compare two vehicles in terms of annual oil barrel consumption, carbon emissions, and other environmental indicators.   
Our backend consisted of a Flask App interfaced with CockroachDB and a database of over 20 000 North American automobiles and their specifications. Our frontend consisted of React with Material-UI and DevExtreme React Charts. I had lots of fun over the course of 36 hours learning more about components, props, and hooks in React, while also exploring UI/UX design with Material-UI and its many features - some of which being Combo Boxes, Cards, and Grids. In the future, I'd like to explore creating more dynamic pages with React, and learning to use state and hooks more effectively to create aesthetic user experiences.

### IEEE UofT- Newhacks 2020 - TotallyNotAVirus
[Devpost Submission](https://devpost.com/software/totallynotavirus)  

This was my first full 24-hour, no rest hackathon. Why? I felt strongly motivated to work on something that had such genuine potential to do good in the world - a COVID-19 Diagnosis app that uses sample cough audio to predict your chances of having the virus. Was it a stretch for a 24-hour hackathon? Yes. Was it fun and challenging? Absolutely. Did our project work without flaws in the end? No, not a chance. Did we learn a lot? Totally. I think the most important part of any hackathon is what you get out of it in the end. In this hackathon, I learned more about OpenCV Perspective Transforms, Mel Spectrograms (representing audio in image format), and serialization of ML models.  

I mainly worked on image preprocessing. This was an interesting task as the input data we had sourced from [Kaggle](https://www.kaggle.com/anaselmasry/ai-covid19-from-cough-samples) had inconsistent image scaling and rotation. After researching different methods of rotation, scaling, cropping, and contour tracing, I decided the best method would involve the combination of: 1. Finding the best approximation of 4 corners of the spectrogram image, and 2. Using these points to perform a perspective transformation, which will take care of both scaling and rotation (think of it as "warping" the entire image).  

1. I used ```goodFeaturesToTrack(..., 40, ..., 50)``` to find the 40 most relevant points outlining the original spectrogram. The 50 specifies the minimum spacing between points (this ensured we minimized the number of "accidental" points on the contours of the spectrogram frequencies). I then approximated the distance between each found point on the spectrogram and the actual corners of the image. Next, it was an easy step to find the minimum distance between a given point and the closest corner. The closest point to each corner was saved; the next step was to use it for the perspective transformation.   

2. After approximating the four corners of the original spectrogram, a perspective transformation is applied, which uses a transformation matrix consisting of the closest points we found, and the new points we'd like to map them to (the actual corners of the image).  

Here are some examples of the input spectrogram images (before) and the output images (after): 
  
Before                                                              |  After
:-----------------------------------------------------------------:|:-------------------------------------------------:
![covid_negatives_original](/assets/Project_Pictures/covid_negatives_original.png)  |  ![covid_negatives_processed](/assets/Project_Pictures/covid_negatives_processed.png)


Here are some examples of the improvement in output image quality after modifying parameters of goodFeaturesToTrack. (This reduced warping/skewing issues by around 66% (or a reduction in error from 3% of overall dataset to 1% of overall dataset):  

![before_and_after_tweak_goodFeaturesToTrack](/assets/Project_Pictures/before_and_after_tweak_goodFeaturesToTrack.png)

Here is an example of how a corner case (in which 2 corners of the original spectrogram are missing) still successfully undergoes a perspective transform:  
(the blue points are those found by goodFeaturesToTrack)  

![corner_case_before_after](/assets/Project_Pictures/corner_case_before_after.png)

#### Limitations
One limitation of this algorithm is that it will only work on images in which the closest point to the corner of the image lies very close or on the *vertex* of the spectrogram. If an image is rotated/skewed too much (>45 degrees maybe), then the closest point may lie on an *edge* rather than a corner/vertice. goodFeaturesToTrack will then skew the image incorrectly (and the spectrogram will have distorted audio when converted back to raw audio as a result).
 
#### What would I change for next time?  
I would try to minimize the number of unneccessary points found on the original spectrogram. A potential method could be to create a mask of the original spectrogram, reduce the size by a small %, and then use goodFeaturesToTrack only on the outside of that mask. This would ideally give us more points on the perimeter of the image.


### MLH - To the Moon and Hack 2020 - MuscleMath
[GitHub Project](https://github.com/eyfb/MuscleMath)  
[Devpost Submission](https://devpost.com/software/muscle-math)  

This was a fantastic hackathon to attend during the pandemic in the summer of 2020. In this project, I was able to apply what I had learned from working over the summer as a Software Engineering Intern. With knowledge of Node.js, HTML and CSS, I learned to use chart.js to graph data fetched asynchronously from a csv, and I used csv-writer to append data to a csv. I also learned to use Express.js for routing pages. Coming out of this hackathon, I gained a newfound appreciation for web development, and I am continuing to learn more about web technologies on UofT's newly founded Cloud Club software team.

<!--YouTube-->
<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed/yJsUVUOrPow' frameborder='0' allowfullscreen></iframe></div>

### MLH - MakeUofT 2020 - StayEnlightened
[GitHub Project](https://github.com/joonhosung/StayEnlightened)  
[Devpost Submission](https://devpost.com/software/stayenlightened)  

This project was created to solve a problem in study spaces with motion-activated lighting: when students are studying (i.e. not moving), lighting will turn off automatically, creating a distraction and making it hard to maintain focus. Our team remedied the problem by using a webcam with OpenCV, Python, Arduino, and the Qualcomm Dragonboard 410C to detect not only motion, but human presence, ensuring the lights are kept on until no humans are present. In this project, I learned how to use OpenCV and Python to grab individual frames from a Logitech webcam. I also integrated the Grove Base Shield on the Qualcomm DragonBoard 410C and implemented a PIR sensor to detect motion using Arduino. Finally, I wrote code to transmit and receive serial data between the Arduino and Python OpenCV camera code.

## Courses
* * *

### CS50AI - Harvard's Introduction to Artificial Intelligence with Python
I had a lot of fun doing the projects for CS50AI. I hadn't used Python to much of an extent before this course, but I happily learned it to be able to play around with some cool AI-driven projects. Here's the final project, an AI that answers questions using the nltk library and TF-IDF to rank documents in the AI's knowledge base of wikipedia articles.  
<!--YouTube-->
<div class='embed-container'><iframe src='https://www.youtube.com/embed/0b5fYlgueTo' frameborder='0' allowfullscreen></iframe></div>

#### Full list of projects:
•	Implemented an AI Tic-Tac-Toe game using pygame, using depth-limited minimax and alpha-beta pruning for speed  
•	Learned to use propositional logic to solve minesweeper and “Knights and Knaves” logic puzzles  
•	Wrote a PageRank system using sampling with Markov Chains, and recursive iteration with a damping factor  
•	Learned how to use unconditional probabilities to guess a family member’s probability of possessing certain genes  
•	Created a crossword puzzle generator using AC-3 arc consistency to enforce domains of possible words  
•	Used scikit-learn to solve a supervised learning problem in which the program predicts whether or not a shopper will complete an online purchase, using nearest-neighbour classification  
•	Wrote an AI to play ‘Nim’ (strategy game) by using Q-Learning, the Epsilon-Greedy algorithm and training 10K games  
•	Implemented a multi-layered CNN to identify types of street signs using the gtsrb dataset  
•	Wrote a natural language parser with nltk, using context-free grammar to build trees and extract noun-chunk phrases  
•	Built an AI to answer questions using nltk and TF-IDF to rank existing documents in the AI’s knowledge base  

## Hardware
* * *

### Split Mechanical Keyboard
Built over the summer of 2020 as a fun side project, this keyboard is split for better ergonomics, and features wrist rests for reduced strain on wrists. This project was built using the following [thingiverse project](https://www.thingiverse.com/thing:2879329). Cherry MX Red mechanical switches were soldered with a wire and diode matrix. The firmware was generated from this [tool](https://kbfirmware.com/) and flashed to an Arduino Pro Micro. One addition I made to this project was a reset push-button to enable firmware flashing with updated keyboard layers/macros. The case was 3D printed using an Ender 3 Pro with tranparent and white PLA.
  
![split_mech](/assets/Project_Pictures/split_mech.jpg)

### Chord Machine - FPGA Audio Synthesizer
This was a project for a Digital Systems course in second year ECE at the Unversity of Toronto. Using the DE1-SoC FPGA and Quartus Prime 18.1, I programmed the audio chipset on the DE1-SoC board to create square, triangle and saw waveforms (Verilog). The board can interface with a PS/2 keyboard to allow the user to play along with a built-in 4/4 looper. More details can be read about [here.](https://github.com/iWebster28/Music/blob/master/ChordMachine/ChordMachineFinal/ECE241%20Final%20Project%20Report%20-%20Ian%20Webster%20-%20Github.pdf)

<!--YouTube-->
<div class='embed-container'><iframe src='https://www.youtube.com/embed/eF6zlbV_rJg' frameborder='0' allowfullscreen></iframe></div>

### DAW Macro Footswitch Pedal 
I commenced this project to enhance my music composition workflow. As an avid pianist, I love to compose music just as much as I love to perform it. A few years back, I realized that I had difficulty with recording multiple takes in a single recording session. Frustrated by the inefficiencies of the computer keyboard 'shortcuts,' I decided to come up with my own solution to avoid pressing 'spacebar, delete, enter, and R' every time I wanted to re-record a take while composing a song.

To start, I drafted many iterations of the design. I eventually landed on a configuration that utilized the most-used keys when recording a take: spacebar (to stop the current recording), enter (to move the play head back to the start), 'ctrl' and 'z' to be pressed at the same time to undo the recording, and 'r' to record again. I optimized this process with 5 arcade-style buttons and 2 smaller buttons that would be easily pressed in various combinations with a foot. A schematic will be added shortly showing the commands for each key.

In terms of hardware, the main structure is a wedge comprised of 2 pieces of plywood. The angled design enables for a comfortable foot rest while recording. The wiring was done in accordance to the pin-outs of each keyboard controller. Prior to project commencement, a spreadsheet was used to record each corresponding key-out for each pair of pins on each controller. Two controllers were used to accomodate for a faulty controller. USB power was used to power LEDs in the 2 smaller switches, as well as 3 LEDs on the underside of the plywood.

Front                                                              |  Bottom
:-----------------------------------------------------------------:|:-------------------------------------------------:
![DAW_Footswitch 1](/assets/Project_Pictures/DAW_Footswitch1.jpg)  |  ![DAW_Footswitch2](/assets/Project_Pictures/DAW_Footswitch2.jpg)

### Portable Raspberry Pi Synth
This was a project created over the summer of 2019 -  it consists of a Raspberry Pi Zero W, a portable 5V USB battery pack, a USB to 1/8" audio-in and out jack, and LED lighting with a Power switch, all inside a clear plastic enclosure. The Pi is running Raspbian Jesse with an instance of FluidSynth (a software MIDI synthesizer) instantiated on startup. A MIDI keyboard can be plugged in via USB and headphones or a speaker can connect to the audio out jack. This project serves to lighten the load when playing music on the go, or when playing a gig with the MIDI Keytar, as seen below.

![RaspiSynth](/assets/Project_Pictures/RaspiSynth.jpg)

### MIDI Keytar
This was a fun project to explore music and technology, which was inspired by Herbie Hancock, a famous jazz musician who started out in electrical engineering! Using an M-Audio Keystation 49 MIDI keyboard, I ran bolts through each side of the keyboard chassis to create mount points for straps. I then added a Samson Graphite MF8 mounted with a wireframe mesh to add additional MIDI transport controls. This keytar performs nearly identically to modern day equivalents priced at over $1000!

![Keytar](/assets/Project_Pictures/Keytar.jpg)

### Bluetooth Audio Receiver
Using an old BlueAnt Bluetooth Speakerphone (back when these were a thing), I soldered a ¼ inch female audio jack to the amplification circuit, allowing for bluetooth music streaming from a phone to an external stereo system.

![Blueant](/assets/Project_Pictures/Blueant.jpg)

### Robotic Hand
This project was made for a Biology project on Biomechanical Engineering in Grade 12. After watching various YouTube tutorials, I decided to construct a robotic hand that mimics real-time movement of a glove worn by the human hand. The project allowed me to explore the uses of linear resistive control with an Arduino microcontroller and 5 hobby servos. Since flex sensors were not readily available to me, I constructed them using pairs of aluminum foil tape, cardboard, and electrical tape. After multiple calibrations, each sensor was ready for use the glove. In the end, I had a working model of a myoelectric limb, a type of prosthetic used for amputees or those with congenital limb loss.

![Robotic_Hand](/assets/Project_Pictures/Robotic_Hand.jpg)

### Server/Network Configuration
I have configured and built many servers over the span of my teenage years. Being into video games such as Minecraft when I was younger, I have always been interested in server configuration - static and dynamic IP addresses, port forwarding and router security. I have revamped Dell PowerEdge T100 and 2950 servers, changing RAM and installing hard drives and SSDs. Using old Lenovo office computers, I have created a personal file storage cloud. Using FreeNAS and multimedia plugins such as Plex Media Server, I've set up movie and music streaming services that may be accessed worldwide through portforwarding and SSH utilities. In addition, I have experimented with Windows Server 2012 and Apple OS X Server (through VMWare's ESXi) for file backup. I am constantly trying to improve my networking knowledge, for example, I used an old Cisco E2500 to test-install DD-WRT software and convert it to a repeater for my home network.

### Desktop/Laptop Configuration
Outside of servers, I enjoy building computers. I have built an AMD Threadripper system for a friend (installing watercooling for a Ryzen 2700X), as well as my own dual-boot Windows/Mac OS system on Haswell Architecture. In the meantime, I enjoy revamping various Apple notebooks, upgrading RAM and SSDs in pre-2010 MacBook and MacBook Pro models. 

### Computer Server Cabinet
This was built to complement that home media servers that I have configured. The cabinet was first drafted by hand and the area of wood required was calculated. Built from  5/8" plywood, this cabinet houses 5 servers, as well as a monitor on a swing arm. The unit also functions as a standing desk for enhanced ergonomics while working.

Front-Down                                                        |  Front
:----------------------------------------------------------------:|:-------------------------------------------------:
![Server_Cabinet1](/assets/Project_Pictures/Server_Cabinet1.jpg)  |  ![Server_Cabinet2](/assets/Project_Pictures/Server_Cabinet2.jpg)

## Past Software
* * *

### Java | 2D Wave Game
This project was created for a highschool computer science course. I led a team to develop a 2D platformer game using object-oriented programming, delegating tasks to individuals and documenting the project in weekly summation journals. To improve project organization, I used IPO charts, pseudocode, and software development lifecycles. Most of what I learned for this project came from YouTube tutorials - the channel 'RealTutsGML' was used as a primary resource. The following pictures show some gameplay.

![WaveGame_1](/assets/Project_Pictures/WaveGame_1.png)

![WaveGame_Boss](/assets/Project_Pictures/WaveGame_Boss.png)

![WaveGame_Lose](/assets/Project_Pictures/WaveGame_Lose.png)

### Java | Inventory Management Software
I developed this application in highschool as well - it takes inputs for item information, and returns stock information in a GUI. 

![InventoryTracker_AddItem](/assets/Project_Pictures/InventoryTracker_AddItem.jpg) ![InventoryTracker_DiscontinueItem](/assets/Project_Pictures/InventoryTracker_DeleteItem.jpg)

### MATLAB | Robot Position Curve Fitting
I have significant experience programming in MATLAB, using scripts for techniques such as vector and matrix operations, numeric and symbolic computation, 2D and 3D graphing, polynomial curve fitting, derivatives, linear systems, and parameterization. Using MATLAB, I imported robot position data and used polynomial curve-fitting techniques to find optimal models for position, velocity and acceleration. Below are some graphs from the final project document.

![MATLAB_RobotGraph_1](/assets/Project_Pictures/MATLAB_RobotGraph_1.png) ![MATLAB_RobotGraph_2](/assets/Project_Pictures/MATLAB_RobotGraph_2.png)

### C/C++ | Chord App/Jazzifier
An app I designed over the summer of 2019 to create pseudo-randomized jazz chord charts for musical inspiration. 

![Jazzifier_Demo](/assets/Project_Pictures/Jazzifier_Demo.png) 



[<Back to Home](./)
