To implement the **WebView** video player that can play a video from phone storage in **Kotlin**, you can follow a similar process as in Java, but with Kotlin syntax. Below is the step-by-step implementation using **WebView** and **VideoView**.

### 1. **WebView Video Player in Kotlin**:

#### 1.1 **Permissions in AndroidManifest**:
First, declare the necessary permissions in your `AndroidManifest.xml` to read from external storage:

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

#### 1.2 **Requesting Runtime Permissions**:
Since Android 6.0 (API level 23) and above, you need to request runtime permissions for reading external storage:

```kotlin
if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
    != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE), 100)
}
```

#### 1.3 **WebView Setup**:
Now, let's set up the `WebView` to load the video file from the phone's storage using a file URI.

```kotlin
import android.Manifest
import android.content.pm.PackageManager
import android.os.Bundle
import android.webkit.WebSettings
import android.webkit.WebView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val webView: WebView = findViewById(R.id.webView)
        
        // Request permission if not granted
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) 
            != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE), 100)
        } else {
            loadVideoInWebView(webView)
        }
    }

    private fun loadVideoInWebView(webView: WebView) {
        webView.settings.javaScriptEnabled = true
        webView.settings.allowFileAccess = true
        webView.settings.mediaPlaybackRequiresUserGesture = false // Autoplay support

        // Path to the video in phone storage (ensure the file exists)
        val videoPath = "file:///storage/emulated/0/videos/sample.mp4"

        // HTML content with the video tag
        val html = """
            <html>
                <body>
                    <video width="100%" height="100%" controls autoplay>
                        <source src="$videoPath" type="video/mp4">
                        Your browser does not support the video tag.
                    </video>
                </body>
            </html>
        """.trimIndent()

        webView.loadDataWithBaseURL(null, html, "text/html", "UTF-8", null)
    }

    // Handle permission result
    override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
        if (requestCode == 100) {
            if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                val webView: WebView = findViewById(R.id.webView)
                loadVideoInWebView(webView)
            } else {
                Toast.makeText(this, "Permission Denied", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

### Explanation:
- **Request Permission**: The app requests permission to read external storage, as video files are likely stored there.
- **WebView Settings**: JavaScript is enabled, and `allowFileAccess` is set to true to allow file access, while `mediaPlaybackRequiresUserGesture` is set to false to allow video autoplay.
- **Load HTML in WebView**: We dynamically load HTML content with a video tag pointing to the local file path using the `file://` URI scheme.

### 2. **Alternative Approach: VideoView in Kotlin**:

For better performance and smoother control, you can use Android's native **VideoView** to play the video. Here's how to implement it:

#### 2.1 **Layout (XML)**:
Define a `VideoView` in your layout file (`activity_main.xml`):

```xml
<VideoView
    android:id="@+id/videoView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

#### 2.2 **Kotlin Code**:
Now, in your `MainActivity.kt`, set up the `VideoView` to play the video from storage.

```kotlin
import android.Manifest
import android.content.pm.PackageManager
import android.net.Uri
import android.os.Bundle
import android.os.Environment
import android.widget.MediaController
import android.widget.VideoView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val videoView: VideoView = findViewById(R.id.videoView)

        // Request permission if not granted
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
            != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE), 101)
        } else {
            playVideo(videoView)
        }
    }

    private fun playVideo(videoView: VideoView) {
        // Path to the video in the phone's storage
        val videoPath = Environment.getExternalStorageDirectory().path + "/videos/sample.mp4"
        val videoUri = Uri.parse(videoPath)

        // Setup MediaController (Play, Pause, etc.)
        val mediaController = MediaController(this)
        mediaController.setAnchorView(videoView)

        videoView.setMediaController(mediaController)
        videoView.setVideoURI(videoUri)
        videoView.start() // Start the video automatically
    }

    // Handle permission result
    override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
        if (requestCode == 101) {
            if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                val videoView: VideoView = findViewById(R.id.videoView)
                playVideo(videoView)
            } else {
                Toast.makeText(this, "Permission Denied", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

### Explanation:
- **VideoView** is a native Android UI component specifically designed for video playback. It provides a better performance than using WebView for videos.
- **MediaController** provides built-in play, pause, and seek controls for video playback.
- **Uri.parse** is used to convert the local video file path into a URI that `VideoView` can play.

### Conclusion:
- **WebView**: While WebView can be used for simple embedded video playback, it is generally less efficient than native solutions for local media.
- **VideoView**: This is a more optimized and recommended approach for playing videos stored on the device as it offers smoother performance and better control over media playback. If you need more advanced features like adaptive streaming, you can also explore using **ExoPlayer**, which offers even more powerful media handling.
