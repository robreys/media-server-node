# WebRTC Medooze Media Server for Node.js

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/72346e5229bc4fd8af091312be091fdd)](https://www.codacy.com/app/murillo128/media-server-node?utm_source=github.com&utm_medium=referral&utm_content=medooze/media-server-node&utm_campaign=badger)

This media server will allow you to receive and send media streams from remote WebRTC peers and manage how you want to route them. 
You will be able to record any incoming stream into an mp4 file.
SVC layer selection and simulcast support.

## Install

    npm i --save medooze-media-server

## Usage
```javascript
const MediaServer = require('medooze-media-server');
```
## API Documention
You can check the full object documentation [here](https://medooze.github.io/media-server-node/).

## Demo application
You can check a demo application [here](https://github.com/medooze/media-server-demo-node)

## Example

```javascript
const SemanticSDP	= require("semantic-sdp");

//Process the sdp
var offer = SemanticSDP.SDPInfo.process(sdp);

//Get the Medooze Media Server interface
const MediaServer = require('medooze-media-server');

//Create UDP server endpoint
const endpoint = MediaServer.createEndpoint(ip);

//Create an DTLS ICE transport in that enpoint
const transport = endpoint.createTransport({
	dtls : offer.getDTLS(),
	ice  : offer.getICE() 
});
	
//Set RTP remote properties
 transport.setRemoteProperties({
	audio : offer.getMedia("audio"),
	video : offer.getMedia("video")
});


//Get local DTLS and ICE info
const dtls = transport.getLocalDTLSInfo();
const ice  = transport.getLocalICEInfo();

//Get local candidates
const candidates = endpoint.getLocalCandidates();

//Create local SDP info
let answer = new SDPInfo();

//Add ice and dtls info
answer.setDTLS(dtls);
answer.setICE(ice);
//Add candidates
for (let i=0;i<candidates.length;++i)
	//Add candidate to media info
	answer.addCandidate(candidates[i]);

//Get remote audio m-line info 
let audioOffer = offer.getMedia("audio");

//If we have audio
if (audioOffer)
{
	//Create audio media
	let audio = new MediaInfo("audio", "audio");
	
	//Get codec type
	let opus = audioOffer.getCodec("opus");
	//Add opus codec
	audio.addCodec(opus);

	//Add audio extensions
	for (let [id, uri] of audioOffer.getExtensions().entries())
		//Add it
		audio.addExtension(id, uri);
	//Add it to answer
	answer.addMedia(audio);
}

//Get remote video m-line info 
let videoOffer = offer.getMedia("video");

//If offer had video
if (videoOffer)
{
	//Create video media
	let  video = new MediaInfo("video", "video");
	//Get codec types
	let vp9 = videoOffer.getCodec("vp9");
	let fec = videoOffer.getCodec("flexfec-03");
	//Add video codecs
	video.addCodec(vp9);
	if (fec!=null)
		video.addCodec(fec);
	//Limit incoming bitrate
	video.setBitrate(1024);

	//Add video extensions
	for (let [id, uri] of videoOffer.getExtensions().entries())
		//Add it
		video.addExtension(id, uri);

	//Add it to answer
	answer.addMedia(video);
}

//Set RTP local  properties
transport.setLocalProperties({
	audio : answer.getMedia("audio"),
	video : answer.getMedia("video")
});

//For each stream offered
for (let offered of offer.getStreams().values())
{
	//Create the remote stream into the transport
	const incomingStream = transport.createIncomingStream(offered);
	
	//Record it
	recorder.record(incomingStream);
	
	//Create new local stream
	const outgoingStream  = transport.createOutgoingStream({
		audio: true,
		video: true
	});

	//Get local stream info
	const info = outgoingStream.getStreamInfo();
	
	//Copy incoming data from the remote stream to the local one
	outgoingStream.attachTo(incomingStream);
	
	//Add local stream info it to the answer
	answer.addStream(info);
}
//Get answer SDP
const str = answer.toString();
```

## Author

Sergio Garcia Murillo

## License
MIT
