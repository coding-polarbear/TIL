# android TTS

## TTS?
* Text-To-Speech의 약자
* 일반 텍스트 데이터를 음성으로 변환해주는 것. 

안드로이드에서는 기본적으로 google TTS를 제공한다.

Google Cloud Platform에서 제공하는 유로 TTS에 비하면 음성 커스텀이나

여러가지 한계가 분명히 있지만, 제일 무난하게 사용할 수 있는 방법 또한 기본 TTS를 이용하는 방법이다.


## 사용법


#### **1. AndroidManifest.xml**에 권한 추가

`AndroidManifest.xml`에 쿼리를 추가한다.

```xml
    <queries>
        <intent>
            <action android:name="android.intent.action.TTS_SERVICE" />
        </intent>
    </queries>
```


#### **2. tts 객체 생성**
TTS 객체를 생성한다.

```kotlin
    private val tts = TextToSpeech(context, ttsInitListener, "com.google.android.tts")
```

TTS 객체를 생성하면서 패키지네임을 지정하지 않을 경우 휴대폰 기본설정을 따라가는데,

삼성 갤럭시의 경우 삼성TTS를 기본값으로 지정하고 있기 때문에

Google TTS를 사용하도록 `package name`을 지정하는 것이 좋다.

#### **3. UtteranceProgressListener와 language 설정**

TTS 객체 생성 후에는 TTS의 utterance progress에 따른 동작을 정의하는 `UtteranceProgressListener`를 설정해주고,

현재 TTS에서 사용할 언어를 지정해준다.

```kotlin
    private val utteranceProgressListener = object : UtteranceProgressListener() {
        override fun onStart(utteranceId: String?) {
            Log.d(TAG, "onStart $utteranceId")
        }

        override fun onDone(utteranceId: String?) {
            Log.d(TAG, "$utteranceId was speaked by tts")
        }

        override fun onError(utteranceId: String?) {
            Log.d(TAG, "$utteranceId error")
        }
    }

    tts.language = Locale.US
```


#### **4. speak() **
speak method에 들어갈 변수 중 제일 중요한 것은 두번째로 들어가는 파라미터 queueMode이다.

```java
public int speak (CharSequence text, 
                int queueMode, 
                Bundle params, 
                String utteranceId)
```


```kotlin
tts.speak("hello world!", TextToSpeeh.QUEUE_FLUSH, null, "init-id")
```


| 플래그 명 | 역할 |
| ---- | ---- |
| TextToSpeech.QUEUE_FLUSH | 진행중인 음성을 끊고 이번 TTS 음성을 출력한다. |
| TextToSpeech.QUEUE_ADD | 기존 음성이 끝나고 이번 TTS 음성을 출력한다. |