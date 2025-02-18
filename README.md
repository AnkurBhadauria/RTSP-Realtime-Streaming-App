# rtsp-client-android
<b>Lightweight RTSP client library for Android</b> with almost zero lag video decoding (achieved 20 msec video decoding latency on some RTSP streams). Designed for lag criticial applications (e.g. video surveillance from drones, car rear view cameras, etc.).

Unlike [AndroidX Media ExoPlayer](https://github.com/androidx/media) which also supports RTSP, this library does not make any video buffering. Video frames are shown immidiately when they arrive.


![Screenshot](docs/images/rtsp-demo-app.webp?raw=true "Screenshot")

## Features:
- RTSP/RTSPS over TCP.
- Supports majority of RTSP IP cameras.
- Video H.264/H.265.
- Audio AAC LC only.
- Ability to [rewrite SPS frame](https://github.com/alexeyvasilyev/rtsp-client-android/blob/dbea741548307b1b0e1ead0ccc6294e811fbf6fd/library-client-rtsp/src/main/java/com/alexvas/rtsp/widget/RtspProcessor.kt#L106C9-L106C55) with low-latency parameters (EXPERIMENTAL).
- Video rotation (90, 180, 270 degrees). 
- Android min API 24.


