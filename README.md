# rtsp-client-android
<b>Lightweight RTSP client library for Android</b> with almost zero lag video decoding (achieved 20 msec video decoding latency on some RTSP streams). Designed for lag criticial applications (e.g. video surveillance from drones, car rear view cameras, etc.).

Unlike [AndroidX Media ExoPlayer](https://github.com/androidx/media) which also supports RTSP, this library does not make any video buffering. Video frames are shown immidiately when they arrive.

[![Release](https://jitpack.io/v/alexeyvasilyev/rtsp-client-android.svg)](https://jitpack.io/#alexeyvasilyev/rtsp-client-android)

![Screenshot](docs/images/rtsp-demo-app.webp?raw=true "Screenshot")

## Features:
- RTSP/RTSPS over TCP.
- Supports majority of RTSP IP cameras.
- Video H.264/H.265.
- Audio AAC LC only.
- Ability to [rewrite SPS frame](https://github.com/alexeyvasilyev/rtsp-client-android/blob/dbea741548307b1b0e1ead0ccc6294e811fbf6fd/library-client-rtsp/src/main/java/com/alexvas/rtsp/widget/RtspProcessor.kt#L106C9-L106C55) with low-latency parameters (EXPERIMENTAL).
- Video rotation (90, 180, 270 degrees). 
- Android min API 24.



You can still use library without any decoding (just for obtaining raw frames from RTSP source), e.g. for writing video stream into MP4 via muxer.

```kotlin
val rtspClientListener = object: RtspClient.RtspClientListener {
    override fun onRtspConnecting() {}
    override fun onRtspConnected(sdpInfo: SdpInfo) {}
    override fun onRtspVideoNalUnitReceived(data: ByteArray, offset: Int, length: Int, timestamp: Long) {
        // Send raw H264/H265 NAL unit to decoder
    }
    override fun onRtspAudioSampleReceived(data: ByteArray, offset: Int, length: Int, timestamp: Long) {
        // Send raw audio to decoder
    }
    override fun onRtspApplicationDataReceived(data: ByteArray, offset: Int, length: Int, timestamp: Long) {
        // Send raw application data to app specific parser
    }
    override fun onRtspDisconnected() {}
    override fun onRtspFailedUnauthorized() {
        Log.e(TAG, "RTSP failed unauthorized");
    }
    override fun onRtspFailed(message: String?) {
        Log.e(TAG, "RTSP failed with message '$message'")
    }
}

val uri = Uri.parse("rtsps://10.0.1.3/test.sdp")
val username = "admin"
val password = "secret"
val stopped = new AtomicBoolean(false)
val sslSocket = NetUtils.createSslSocketAndConnect(uri.getHost(), uri.getPort(), 5000)

val rtspClient = RtspClient.Builder(sslSocket, uri.toString(), stopped, rtspClientListener)
    .requestVideo(true)
    .requestAudio(true)
    .withDebug(false)
    .withUserAgent("RTSP client")
    .withCredentials(username, password)
    .build()
// Blocking call until stopped variable is true or connection failed
rtspClient.execute()

NetUtils.closeSocket(sslSocket)
```

## How to get lowest possible latency:
There are two types of latencies:

### Network latency
If you want the lowest possible network latency, be sure that both Android device and RTSP camera are connected to the same network by the Ethernet cable (not WiFi).

Another option to try is to decrease stream bitrate on RTSP camera. Less frame size leads to less time needed for frame transfer.

### Video decoder latency
Video decoder latency can vary significantly on different Android devices and on different RTSP camera streams.

For the same profile/level and resolution (but different cameras) the latency in best cases can can be 20 msec, in worst cases 1200 msec.

To decrease latency be sure you use the lowest possible H.264 video stream profile and level (enable `debug` in the library and check SPS frame params `profile_idc` and `level_idc` in the log). `Baseline profile` should have the lowest possible decoder latency.
Check `max_num_reorder_frames` param as well. For best latency it's value should be `0`.

