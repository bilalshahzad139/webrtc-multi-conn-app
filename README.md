
# What does this do?
It creates a meeting URL for you and allows multiple people to join that meeting. It does peer to peer communication for audio/video. It uses a SignalR based signaling server for client to server and server to client communication. SDP & ICE details (between peers) will be shared through this Signaling Server. Participants can do Audio/Video/Screen sharing and it is being done using Browser builtin WebRTC APIs. 
**Note:** Peer to Peer approach is good for few users (e.g. up to three) but if more users join the meeting, you will start facing performance degradation as each participant will be communicating with every other participant. In those cases, you would like to involve a media relay server which will receives media traffic and relay to other participants.

[You may see its live demo here](https://webrtcclient.azurewebsites.net/)
[Video Tutorials on WebRTC](https://www.youtube.com/watch?v=ye7PtovMzb4&list=PL0kdOcU3HXGKW8yEEaY_5lz02vFj6gYFW)

# Structure of Repo

## src -> clientapp: 
This folder contains HTML/CSS/JS based client application. It is using WebRTC APIs to allow a user to play with Audio/Video/Screen sharing stream. For Peer To Peer Connections, It uses a Signaling Server (explained below). This folder needs to be hosted in a web server. Opening the html file directly will not work. We can open this folder in Visual Studio Code and run it using Live Server extension. We also have **.vscode/settings.json** file in it. We need to update ssl-certificate files paths in it.
**app.js** contains non-webrtc related javascript. Currently it is using SingalR client API to communicate with signaling server. If you are going to use Non-SignalR based implementation (e.g. NodeJS websocket), you need to make changes in this JS file.
**wrtchelper.js** contains web rtc related javascript. 
## src -> server:
This folder contains an ASP.NET SignalR 2.2 (.NET Framework 4.8) project. This is being used a signaling server and for meeting & participant management. It is maintaining this information in memory (currently) and no database is involved. Hub file (which contains server side functions) is available in Models/WebRtcHub.cs file. You need to build project to download required packages. 
## src -> ssl-certificate
This folder contains localhost certificate to be used with Visual Studio Code Live Server.

# Application Flow
1. When we open index.html of client application, it expects a meeting id (?mid=) in URL. If it is not given, it generates one and shows on page.
2. If meeting id is available in URL, it asks for a user name (in prompt). It sends a request to our signalR server for meeting/participant registration.
3. Server checks & store meeting/user/connection detail in its server memory.
4. When a user registers with a server, it gets details of already registered/connected users against that meeting it. Server also notifies other participants about this new participant. 
5. Client side creates a RTCPeerConnection for every new participant and maintains these connections in an array. Same happens for Streams. 
6. Every box in meeting represents a participant. For every remote participant, we've an RTCPeerConnection, a Video Stream, An Audio Stream, A Video Player and An Audio Player. So as soon as remote peer starts a video stream, that is displayed in relevant video player. 
7. When local user starts camera or starts screen sharing, Video track is added in all remote connections. Same happens for unmute.
8. On remote sides, event of new track is triggered and stream is added in relevant video player or audio player. Also stream is maintained in list as remote stream.

# RTCPeerConnection Management

1. First user (User1) joins the meeting. Box for local user is displayed. No peer exists (or no peers data come from server) so no rtc connection is created. Signaling servers now know about one participant.
2. Second user (User2) joins the meeting.  Box for local user is displayed. Request goes to server. Server saves its details and returns detail of existing users (i.e. user 1). User2 will receive detail of User1 and will create a box for it. **User2 will create an RTCPeerConnection for User1 and will store it in connection array.** Server will also notify User1 about User2. User1 will create a box for User2.  **User1 will create an RTCPeerConnection for User2 and will store it in connections array.**
3. Third user (User3) joins the meeting. Box for local user is displayed. Request goes to server. Server saves its details and returns detail of existing users (i.e. User1, User2). User3 will receive detail of User1 & User2. It will create a boxes for them. **User3 will create RTCPeerConnection for User1 & for User2 and  and will store them in its connections array.** Server will also notify User1 & User2 about User3. User1 will create a box for User3.  **User1 will create an RTCPeerConnection for User3 and will store it in its connections array.** and **User2 will create an RTCPeerConnection for User3 and will store it in its connections array.**
4. When a user (browser) gets disconnected from signalr server, server removes it from local memory (after specific time when onDisconnected event fires). It also notifies other participants so they remove RTC connection object from its connections list.

# SDP Sharing Between Peers

A peer (browser tab) starts communication by creating connection and then creating an 'offer'. This offer object is shared with other peer through signaling server and other peer responds with an 'answer' object. Both shares ICE details through signaling server. Once they share all this meta data, they can do real time communication (audio or video or data streams) directly.

1. In our example, whenever a change is made to connection (streams added or removed), we need offer/answer flow is required again. We can use 'negotiationneeded' event in this case to create offer as this event will be raised whenever we make any change in connection (e.g. adding or removing streams).
2. In this exchange of data through signaling server, user unique identifiers will be shared (who is sending and to whom it is sending).

# How to Setup & Run

1. Clone repository. 
2. Open src/Server/SignalingServer/SignalingServer.csproj in Visual Studio 2019. If you have older VS version, you can create ASP.NET SignalR project there and copy Models/webrtchub.cs code in your hub file.
3. Build project. And run the project. 
4. Open src/clientapp folder in Visual Studio.
5. Fix certificate path in src/clieintapp/.vscode/settings.json
6. Run with Live Server (it is a VS extension, you will have to install)
7. You will see a meeting link generated for you. You can open it in multiple tabs to connect through different users. 
8. When you open a link (with meeting id) in new tab, it creates a connection with signaling server and registers your user with connection id (generated by signalR). So this connection id is unique identifier for application, not your user name.
9. Note each refresh will create a new connection with server (means new registration). Old connections will be removed after few seconds and UI will reflect that too.
10. As it is using WebRTC APIs so when you will start audio/camera/screen sharing, it will ask for your permission.

Copyright (c) 2020 LearningInUrdu https://github.com/bilalshahzad139/
Bilal Shahzad | https://www.linkedin.com/in/bilalshahzad139/

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE
OR OTHER DEALINGS IN THE SOFTWARE.