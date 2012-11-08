This is a teeny little demo for how to use [recorder.js](https://github.com/mattdiamond/Recorderjs) to record and play back audio. It only works in Google Chrome v23+ with the "Web Audio Input" flag enabled in chrome://flags.

Just a warning to anyone who runs this - Chrome's Web Audio input has a *looot* of annoying and potentially headache-inducing feedback if you're using a microphone and external speakers. Plug in headphones.

## How it works

First, the script uses getUserMedia to get a stream of the audio input (i.e. a microphone). Next, an AudioContext is created, which contains AudioNodes (which include audio sources, audio destinations, and any processing between them, such as effects). This context holds the stream source (a source node created from the stream) and the destination (i.e. your speakers).

With this set up, a new Recorder is created from the stream source.

### Recording

Recorder.js is a really simple library, and I encourage you to take a look at its two source files - the main library and the web worker that does its work.

The `Recorder` creates a new `AudioNode` in the `AudioContext` that processes and records the audio. It uses the `onaudioprocess` event each node has to record the data at a given interval, defined by the `bufferSize` parameter. Via the W3C Web Audio spec:

  The bufferSize parameter determines the buffer size in units of sample-frames. It must be one of the following values: 256, 512, 1024, 2048, 4096, 8192, 16384. This value controls how frequently the onaudioprocess event handler is called and how many sample-frames need to be processed each call. Lower values for bufferSize will result in a lower (better) latency. Higher values will be necessary to avoid audio breakup and glitches. The value chosen must carefully balance between latency and audio quality.

 You can define the buffer size yourself when creating a `Recorder` object if you'd like to override it, but by default it's 4096. 

One interesting part of the Recorder.js library is that it uses a Web Worker to run the recording processing in a background thread. This should the performance of any web app that uses the library, as the main thread will not be tied up during any aspect of the recording process.

So, on each recording "tick," the `Recorder` object passes the channel data of the current `inputBuffer` (which is part of the audio processing event object) to the `record` function in the Worker. There are two channel data objects, which are both `Float32Array`s, one representing the left channel and one representing the right channel.

The `record` function first merges (in the `interleave` function) the two arrays together, then pushes the new array into a `recBuffers` array. This array contains all of the recorded buffers for a `Recorder`.

### Playback

The playback process of Recorder.js is almost more interesting than the recording process. After recording, the `Recorder` object now has an array of raw, recorded channel data. There are a few different ways one could take this and play back audio from it - 

## Resources

* [W3C Web Audio spec](https://dvcs.w3.org/hg/audio/raw-file/tip/webaudio/specification.html) - If you're not used to reading these specs, they can be difficult to translate from "language for specification implimenters" to "language for web developers," but it's certainly the most comprehensive source on the matter.
* [Using web workers](https://developer.mozilla.org/en-US/docs/DOM/Using_web_workers)
* [Recorder.js](https://github.com/mattdiamond/Recorderjs)