## 9. Take a photo and share it via a data channel

## What you'll learn

In this step you'll learn how to:

- Take a photo and get the data from it using the canvas element.
- Exchange image data with a remote user.

A complete version of this step is in the **step-06** folder.

## How it works

Previously you learned how to exchange text messages using RTCDataChannel.

This step makes it possible to share entire files: in this example, photos captured via `getUserMedia()`.

The core parts of this step are as follows:

1. Establish a data channel. Note that you don't add any media streams to the peer connection in this step.
2. Capture the user's webcam video stream with `getUserMedia()`:

```
var video = document.getElementById('video');

function grabWebCamVideo() {
  console.log('Getting user media (video) ...');
  navigator.mediaDevices.getUserMedia({
    video: true
  })
  .then(gotStream)
  .catch(function(e) {
    alert('getUserMedia() error: ' + e.name);
  });
}
```

1. When the user clicks the **Snap** button, get a snapshot (a video frame) from the video stream and display it in a `canvas` element:

```
var photo = document.getElementById('photo');
var photoContext = photo.getContext('2d');

function snapPhoto() {
  photoContext.drawImage(video, 0, 0, photo.width, photo.height);
  show(photo, sendBtn);
}
```

1. When the user clicks the **Send** button, convert the image to bytes and send them via a data channel:

```
function sendPhoto() {
  // Split data channel message in chunks of this byte length.
  var CHUNK_LEN = 64000;
  var img = photoContext.getImageData(0, 0, photoContextW, photoContextH),
    len = img.data.byteLength,
    n = len / CHUNK_LEN | 0;

  console.log('Sending a total of ' + len + ' byte(s)');
  dataChannel.send(len);

  // split the photo and send in chunks of about 64KB
  for (var i = 0; i < n; i++) {
    var start = i * CHUNK_LEN,
      end = (i + 1) * CHUNK_LEN;
    console.log(start + ' - ' + (end - 1));
    dataChannel.send(img.data.subarray(start, end));
  }

  // send the reminder, if any
  if (len % CHUNK_LEN) {
    console.log('last ' + len % CHUNK_LEN + ' byte(s)');
    dataChannel.send(img.data.subarray(n * CHUNK_LEN));
  }
}
```

1. The receiving side converts data channel message bytes back to an image and displays the image to the user:

```
function receiveDataChromeFactory() {
  var buf, count;

  return function onmessage(event) {
    if (typeof event.data === 'string') {
      buf = window.buf = new Uint8ClampedArray(parseInt(event.data));
      count = 0;
      console.log('Expecting a total of ' + buf.byteLength + ' bytes');
      return;
    }

    var data = new Uint8ClampedArray(event.data);
    buf.set(data, count);

    count += data.byteLength;
    console.log('count: ' + count);

    if (count === buf.byteLength) {
      // we're done: all data chunks have been received
      console.log('Done. Rendering photo.');
      renderPhoto(buf);
    }
  };
}

function renderPhoto(data) {
  var canvas = document.createElement('canvas');
  canvas.width = photoContextW;
  canvas.height = photoContextH;
  canvas.classList.add('incomingPhoto');
  // trail is the element holding the incoming images
  trail.insertBefore(canvas, trail.firstChild);

  var context = canvas.getContext('2d');
  var img = context.createImageData(photoContextW, photoContextH);
  img.data.set(data);
  context.putImageData(img, 0, 0);
}
```

## Get the code

Replace the contents of your **work** folder with the contents of **step-06**. Your **index.html **file in **work** should now look like this**:**

```
<!DOCTYPE html>
<html>

<head>

  <title>Realtime communication with WebRTC</title>

  <link rel="stylesheet" href="/css/main.css" />

</head>

<body>

  <h1>Realtime communication with WebRTC</h1>

  <h2>
    <span>Room URL: </span><span id="url">...</span>
  </h2>

  <div id="videoCanvas">
    <video id="camera" autoplay></video>
    <canvas id="photo"></canvas>
  </div>

  <div id="buttons">
    <button id="snap">Snap</button><span> then </span><button id="send">Send</button>
    <span> or </span>
    <button id="snapAndSend">Snap &amp; Send</button>
  </div>

  <div id="incoming">
    <h2>Incoming photos</h2>
    <div id="trail"></div>
  </div>

  <script src="/socket.io/socket.io.js"></script>
  <script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>
  <script src="js/main.js"></script>

</body>

</html>
```

If you are not following this codelab from your **work** directory, you may need to install the dependencies for the **step-06**folder or your current working folder. Simply run the following command from your working directory:

```
npm install
```

Once installed, if your Node.js server is not running, start it by calling the following command from your **work** directory:

```
node index.js
```

Make sure you're using the version of **index.js** that implements Socket.IO, and remember to restart your Node.js server if you make changes. For more information on Node and Socket IO, review the section "Set up a signaling service to exchange messages".

If necessary, click on the **Allow** button to allow the app to use your webcam.

The app will create a random room ID and add that ID to the URL. Open the URL from the address bar in a new browser tab or window.

Click the **Snap & Send** button and then look at the Incoming area in the other tab at the bottom of the page. The app transfers photos between tabs.

You should see something like this:

![img](https://codelabs.developers.google.com/codelabs/webrtc-web/img/911b40f36ba6ba8.png)

## Bonus points

1. How can you change the code to make it possible to share any file type?

## Find out more

- [The MediaStream Image Capture API](https://www.chromestatus.com/features/4843864737185792): an API for taking photographs and controlling cameras — coming soon to a browser near you!
- The MediaRecorder API, for recording audio and video: [demo](https://webrtc.github.io/samples/src/content/getusermedia/record/), [documentation](https://www.chromestatus.com/features/5929649028726784).

## What you learned

- How to take a photo and get the data from it using the canvas element.
- How to exchange that data with a remote user.

A complete version of this step is in the **step-06** folder.

