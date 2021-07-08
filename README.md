# Android fcm push test

FCM 푸시 메시지 수신 테스트용 안드로이드 어플리케이션. Firebase 에서 제공하는 [예제 앱](https://github.com/firebase/quickstart-android/tree/master/messaging) 이 있긴 한데
버전이 안맞는지 자꾸 앱이 죽어서 직접 테스트용 앱을 개발했다.

## 참고자료

[HEROJOON](https://herojoon-dev.tistory.com/18) 님의 블로그를 참고하여 개발함.  
개발 중간에 라이브러리 버전이 맞지 않는 부분은 [firebase 공식 문서](https://firebase.google.com/docs/cloud-messaging/android/client?authuser=1)
를 참고함. 

## 개발 과정

### Firebase 프로젝트 생성 및 앱 등록

![스크린샷 2021-07-08 오후 4 28 03](https://user-images.githubusercontent.com/41066039/124880699-84aa1a80-e009-11eb-8052-bbe5b04e8a0e.png)

Firebase 프로젝트를 생성한 뒤 firebase 의 앱 등록 절차에 따르고 지시에 따라 firebase sdk 를 추가한다

### Firebase messaging sdk 추가

어플리케이션 수준 [build.gradle](app/build.gradle) 에 messaging sdk 추가
```groovy
dependencies {

    // ...
    implementation 'com.google.firebase:firebase-messaging:22.0.0'
}

```

### [AndroidManifest.xml](app/src/main/AndroidManifest.xml) 수정
```
<application>
        <!-- ... -->
        <service
                android:name=".MyFirebaseMessagingService"
                android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
    </application>
```

### [MyFirebaseMessagingService](app/src/main/java/io/omnipede/pushtesting/MyFirebaseMessagingService.java) 추가

MainActivity 와 동일한 레벨에 클래스를 생성한다.

### [MainActivity](app/src/main/java/io/omnipede/pushtesting/MainActivity.java) 수정

MainActivity 를 수정하여 토큰 조회하는 코드를 추가한다

```java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button logTokenButton = findViewById(R.id.logTokenButton);

        logTokenButton.setOnClickListener(v -> {
            FirebaseMessaging.getInstance().getToken()
                    .addOnCompleteListener(task -> {
                        if (!task.isSuccessful()) {
                            Log.w(TAG, "Fetching FCM registration token failed", task.getException());
                            return;
                        }

                        // Get new FCM registration token
                        String token = task.getResult();

                        // Log and toast
                        String msg = getString(R.string.msg_token_fmt, token);
                        Log.d(TAG, msg);
                        Toast.makeText(MainActivity.this, msg, Toast.LENGTH_SHORT).show();
                    });
        });
    }
}
```

여기서 ```R.id.logTokenButton``` 와 ```R.string.msg_token_fmt``` 는 따로 정의해야되는 리소스다.  
리소스 패키지의 [activity_main.xml](app/src/main/res/layout/activity_main.xml) 에서 ```R.id.logTokenButton``` 를 정의해주고 
[strings.xml](app/src/main/res/values/strings.xml) 에서 ```R.string.msg_token_fmt``` 를 정의해주자.
