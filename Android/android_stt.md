# Android STT

## STT
* `Speech-To-Text`의 줄임말.
* TTS와 반대로, 사용자가 입력한 Speech Data를 텍스트로 변경하는 것.


### 사용법


#### 1. AndroidManifest.xml에 queries, 권한 추가
```xml
    <queries>
        <intent>
            <action android:name="android.speech.RecognitionService" />
        </intent>
    </queries>
    
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
```

#### 액티비티에서 권한 체크
```kotlin
    private fun requestPermission() {
        ActivityCompat.requestPermissions(
            this,
            arrayOf(Manifest.permission.INTERNET, Manifest.permission.RECORD_AUDIO),
            300
        )
    }
```

#### recoginitionListener 작성
```kotlin
    private val recognizeListener: RecognitionListener = object : RecognitionListener {
        override fun onReadyForSpeech(params: Bundle?) {
            Log.d(TAG, "음성인식 시작")
        }

        override fun onBeginningOfSpeech() {
            Log.d(TAG, "beginOfSpeech")
        }

        override fun onRmsChanged(rmsdB: Float) {
            Log.d(TAG, "rmsChanged")
        }

        override fun onBufferReceived(buffer: ByteArray?) {
            Log.d(TAG, "buffer received")
        }

        override fun onEndOfSpeech() {
            Log.d(TAG, "end of speech")
        }

        override fun onError(error: Int) {
            var message: String = ""

            when (error) {
                SpeechRecognizer.ERROR_AUDIO -> {
                    message = "오디오 에러"
                }

                SpeechRecognizer.ERROR_CLIENT -> {
                    message = "클라이언트 에러"
                }

                SpeechRecognizer.ERROR_INSUFFICIENT_PERMISSIONS -> {
                    message = "퍼미션 없음."
                }

                SpeechRecognizer.ERROR_NETWORK -> {
                    message = "네트워크 에러"
                }

                SpeechRecognizer.ERROR_NETWORK_TIMEOUT -> {
                    message = "네트워크 타임 아웃"
                }

                SpeechRecognizer.ERROR_NO_MATCH -> {
                    message = "찾을 수 없음"
                }

                SpeechRecognizer.ERROR_RECOGNIZER_BUSY -> {
                    message = "RECOGNIZER가 바쁨."
                }

                SpeechRecognizer.ERROR_SERVER -> {
                    message = "서버 오류"
                }

                SpeechRecognizer.ERROR_SPEECH_TIMEOUT -> {
                    message = "스피치 타임아웃."
                }

                else -> {
                    message = "알 수 없는 오류"
                }
            }

            Log.e(TAG, "STT 오류 : $message")
        }

        override fun onResults(results: Bundle?) {
            Log.d(TAG, "onResults")
            val recognizeResult = results?.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION)
            if (!recognizeResult.isNullOrEmpty()) {

            } else {
                Log.e(TAG, "STT Recognize result is empty")
            }
        }

        override fun onPartialResults(partialResults: Bundle?) {
            Log.d(TAG, "onPartialResult")
        }

        override fun onEvent(eventType: Int, params: Bundle?) {
            Log.d(TAG, "onEvent")
        }

    }
```

#### RecognizerIntent, SpeechRecognizer 생성
```kotlin
    private val recognizerIntent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH).apply {
        putExtra(RecognizerIntent.EXTRA_LANGUAGE, ENGLISH)
    }
    private val speechRecognizer = SpeechRecognizer.createSpeechRecognizer(context).apply {
        setRecognitionListener(recognizeListener)
    }
```


#### 실제 음성인식
```kotlin
    override fun recognize() {
        speechRecognizer.setRecognitionListener(recognizeListener)
        speechRecognizer.startListening(recognizerIntent)
    }
```
* 안드로이드에서 기본적으로 체크하는 타임아웃 시간은 5초이다.
* 5초가 지나면 음성 인식을 멈춘다.