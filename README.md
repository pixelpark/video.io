#### WebRTC Group video sharing / [Demo](https://www.webrtc-experiment.com/video-conferencing/)

1. Mesh networking model is implemented to open multiple interconnected peer connections
2. Maximum peer connections limit is 256 (on chrome)

=

#### How `video conferencing` Works?

Huge bandwidth and CPU-usage out of multiple peers interconnection:

To understand it better; assume that 10 users are sharing video in a group. 40 RTP-ports (i.e. streams) will be created for each user. All streams are expected to be flowing concurrently; which causes blur video experience and audio lose/noise (echo) issues

=

#### For each user:

1. 10 RTP ports are opened to send video upward i.e. for outgoing video streams
2. 10 RTP ports are opened to send audio upward i.e. for outgoing audio streams
3. 10 RTP ports are opened to receive video i.e. for incoming video streams
4. 10 RTP ports are opened to receive audio i.e. for incoming audio streams

=

Maximum bandwidth used by each video RTP port (media-track) is about 1MB; which can be controlled using "b=AS" session description parameter values. In two-way video-only session; 2MB bandwidth is used by each peer; otherwise; a low-quality blurred video will be delivered.

```javascript
// removing existing bandwidth lines
sdp = sdp.replace( /b=AS([^\r\n]+\r\n)/g , '');

// setting "outgoing" audio RTP port's bandwidth to "50kbit/s"
sdp = sdp.replace( /a=mid:audio\r\n/g , 'a=mid:audio\r\nb=AS:50\r\n');

// setting "outgoing" video RTP port's bandwidth to "256kbit/s"
sdp = sdp.replace( /a=mid:video\r\n/g , 'a=mid:video\r\nb=AS:256\r\n');
```

=

#### Possible issues:

1. Blurry video experience
2. Unclear voice and audio lost
3. Bandwidth issues / slow streaming / CPU overwhelming

Solution? Obviously a media server!

=

#### Want to use video-conferencing in your own webpage?

```html
<script src="https://www.webrtc-experiment.com/socket.io.js"> </script>
<script src="https://www.webrtc-experiment.com/RTCPeerConnection-v1.5.js"> </script>
<script src="https://www.webrtc-experiment.com/video-conferencing/conference.js"> </script>

<button id="setup-new-room">Setup New Conference</button>
<table style="width: 100%;" id="rooms-list"></table>
<div id="videos-container"></div>
        
<script>
    var config = {
        openSocket: function(config) {
            var SIGNALING_SERVER = 'http://webrtc-signaling.jit.su:80/',
                defaultChannel = location.hash.substr(1) || 'video-conferencing-hangout';

            var channel = config.channel || defaultChannel;
            var sender = Math.round(Math.random() * 999999999) + 999999999;

            io.connect(SIGNALING_SERVER).emit('new-channel', {
                channel: channel,
                sender: sender
            });

            var socket = io.connect(SIGNALING_SERVER + channel);
            socket.channel = channel;
            socket.on('connect', function() {
                if (config.callback) config.callback(socket);
            });

            socket.send = function(message) {
                socket.emit('message', {
                    sender: sender,
                    data: message
                });
            };

            socket.on('message', config.onmessage);
        },
        onRemoteStream: function(media) {
            var video = media.video;
            video.setAttribute('controls', true);
            video.setAttribute('id', media.stream.id);
            videosContainer.insertBefore(video, videosContainer.firstChild);
            video.play();
        },
        onRemoteStreamEnded: function(stream) {
            var video = document.getElementById(stream.id);
            if (video) video.parentNode.removeChild(video);
        },
        onRoomFound: function(room) {
            var alreadyExist = document.querySelector('button[data-broadcaster="' + room.broadcaster + '"]');
            if (alreadyExist) return;

            var tr = document.createElement('tr');
            tr.innerHTML = '<td><strong>' + room.roomName + '</strong> shared a conferencing room with you!</td>' +
                           '<td><button class="join">Join</button></td>';
            roomsList.insertBefore(tr, roomsList.firstChild);

            var joinRoomButton = tr.querySelector('.join');
            joinRoomButton.setAttribute('data-broadcaster', room.broadcaster);
            joinRoomButton.setAttribute('data-roomToken', room.broadcaster);
            joinRoomButton.onclick = function() {
                this.disabled = true;

                var broadcaster = this.getAttribute('data-broadcaster');
                var roomToken = this.getAttribute('data-roomToken');
                captureUserMedia(function() {
                    conferenceUI.joinRoom({
                        roomToken: roomToken,
                        joinUser: broadcaster
                    });
                });
            };
        }
    };

    var conferenceUI = conference(config);
    var videosContainer = document.getElementById('videos-container') || document.body;
    var roomsList = document.getElementById('rooms-list');

    document.getElementById('setup-new-room').onclick = function () {
        this.disabled = true;
        captureUserMedia(function () {
            conferenceUI.createRoom({
                roomName: 'Anonymous'
            });
        });
    };

    function captureUserMedia(callback) {
        var video = document.createElement('video');
        video.setAttribute('autoplay', true);
        video.setAttribute('controls', true);
        videosContainer.insertBefore(video, videosContainer.firstChild);

        getUserMedia({
            video: video,
            onsuccess: function (stream) {
                config.attachStream = stream;
                video.setAttribute('muted', true);
                callback();
            }
        });
    }
</script>
```

=

#### Browser Support

This [WebRTC Video Conferencing](https://www.webrtc-experiment.com/video-conferencing/) experiment works fine on following web-browsers:

| Browser        | Support           |
| ------------- |-------------|
| Firefox | [Stable](http://www.mozilla.org/en-US/firefox/new/) / [Aurora](http://www.mozilla.org/en-US/firefox/aurora/) / [Nightly](http://nightly.mozilla.org/) |
| Google Chrome | [Stable](https://www.google.com/intl/en_uk/chrome/browser/) / [Canary](https://www.google.com/intl/en/chrome/browser/canary.html) / [Beta](https://www.google.com/intl/en/chrome/browser/beta.html) / [Dev](https://www.google.com/intl/en/chrome/browser/index.html?extra=devchannel#eula) |
| Android | [Chrome Beta](https://play.google.com/store/apps/details?id=com.chrome.beta&hl=en) |

=

#### License

[WebRTC Video Conferencing](https://www.webrtc-experiment.com/video-conferencing/) is released under [MIT licence](https://www.webrtc-experiment.com/licence/) . Copyright (c) 2013 [Muaz Khan](https://plus.google.com/100325991024054712503).
