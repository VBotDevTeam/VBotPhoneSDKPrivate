# Android SDK

---

## Code demo

https://github.com/VBotDevTeam/VBot-SDK-Android-Example

---

## Yêu cầu

Android SDK version 23 trở lên

---

## Cấu hình SDK

### Cách 1

Trong file **app** → **build.gradle** thêm 

Cần thêm các thư viện cần thiết để SDK hoạt động 

```kotlin
dependencies {
		//các thư viện cần thiết để SDK hoạt động
    implementation("io.reactivex.rxjava2:rxjava:2.2.21")
    implementation("com.google.code.gson:gson:2.11.0")
    implementation("com.squareup.retrofit2:adapter-rxjava2:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.11.0")
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("com.squareup.okio:okio:3.9.0") // < 2.0.9
    implementation("org.reactivestreams:reactive-streams:1.0.4")
    implementation ("com.squareup.okhttp3:okhttp:5.0.0-alpha.14")
    implementation("com.jakewharton.timber:timber:5.0.1")
    implementation("com.squareup.okhttp3:okhttp-dnsoverhttps:4.9.0")
    implementation ("com.madgag.spongycastle:core:1.58.0.0")
    //Thêm SDK
    implementation 'com.github.VBotDevTeam:VBotPhoneSDKPrivate:2.0.22'
    
}
```
Trong file **settings.gradle** thêm 
```kotlin
dependencyResolutionManagement {
		repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
		repositories {
			mavenCentral()
			maven { url 'https://jitpack.io' }
		}
	}
```
### Cách 2
- Vào trang https://jitpack.io/
- Trong vào ô *'Git repo url'* nhập **VBotDevTeam/VBotPhoneSDKPrivate**
- Nhấn **Look up**
- Chọn version và nhấn **Get it**
- Làm theo hướng dẫn trên trang web
---

## Quyền

Danh sách các quyền cần thiết để SDK hoạt động:

1.	**Quyền truy cập điện thoại**

•	Kiểm tra trạng thái thiết bị (cuộc gọi đến/đi, đang bận, hoặc rảnh).

•	Thực hiện và quản lý cuộc gọi VoIP.

•	**Lưu ý**: Đây là quyền **bắt buộc** để SDK hoạt động.

2.	**Quyền truy cập micro** 

•	Thu âm giọng nói để gửi trong các cuộc gọi VoIP.

•	**Lưu ý**: Đây là quyền **bắt buộc** để SDK hoạt động.

3.	**Quyền truy cập thiết bị ở gần** 

•	Kết nối với các thiết bị Bluetooth như tai nghe hoặc loa ngoài để sử dụng trong cuộc gọi.

4.	**Quyền thông báo** 

•	Hiển thị thông báo cuộc gọi đến hoặc các sự kiện quan trọng từ SDK.

5.	**Quyền hiển thị trên ứng dụng khác (Xuất hiện trên cùng)** 

•	Hiển thị thông báo quan trọng (thông báo cuộc gọi đến) dưới dạng “màn hình nổi” hoặc “pop-up” ngay cả khi ứng dụng đang ở chế độ nền hoặc màn hình khóa.

6.	**Quyền cho phép sử dụng dữ liệu không hạn chế** 

•	Khi hiển thị thông báo cuộc gọi thì sdk có quyền này sẽ hoạt động tốt hơn, kết nối với cuộc gọi nhanh hơn

## Proguard

VBotPhoneSDK yêu cầu một bộ quy tắc ProGuard. Các đoạn mã sau đây cung cấp các quy tắc ProGuard phù hợp dựa trên phiên bản được ứng dụng của bạn sử dụng.

```kotlin
-keep class com.vpmedia.vbotphonesdk.** {*;}

-keep class org.linphone.** { *; }
-keepclassmembers class org.linphone.** { *; }
```

## Sử dụng SDK

---

### Khởi tạo


```kotlin
fun setup(context: Context, token: String)
```

Trong đó:
- **token** là App Token, đại diện cho ứng dụng của bạn được dùng để xác thực với máy chủ VBot 
- **config** là cấu hình tùy chọn cho SDK

---
### Lắng nghe sự kiện

```kotlin
fun addListener(listener: VBotListener)
```

```kotlin
class VBotListener {

    override fun onCallState(state: VBotCallState) {}

    override fun onMessageButtonClick() {}

    override fun callAccepted() {}

    override fun callEnded(reason: VBotEndCallReason) {}
}
```
`onCallState` Trả về trạng thái của cuộc gọi
`onMessageButtonClick` Trả về sự kiện khi người dùng nhấn vào nút nhắn tin
`callAccepted` Trả về cuộc gọi đến được chấp nhận (Khi user chọn chấp nhận cuộc gọi)
`callEnded` Trả về khi tắt cuộc gọi 

---

### Xoá lắng nghe sự kiện

```kotlin
fun removeListener(listener: VBotListener)
```

---

### Gọi đi

Để thực hiện cuộc gọi đi, hãy sử dụng hàm startOutgoingCall
```kotlin
	fun startOutgoingCall(
        callerId: String,
        callerAvatar: String? = null,
        callerName: String,
        calleeId: String,
        calleeAvatar: String? = null,
        calleeName: String,
        checkSum: String,
        metadata: HashMap<String, String>? = null,
        onFailure: ((code: Int, message: String) -> Unit)? = null
	)
```

---

### Gọi đến

Để nhận cuộc gọi đến
Trong hàm onMessageReceived kế thừa từ FirebaseMessagingService hãy sử dụng hàm startIncomingCall

```kotlin
    fun startIncomingCall(
        callerId: String,
        callerAvatar: String? = null,
        callerName: String,
        calleeId: String,
        calleeAvatar: String? = null,
        calleeName: String,
        checkSum: String,
        metadata: HashMap<String, String>? = null,
        onFailure: ((code: Int, message: String) -> Unit)? = null
    )
```

Hàm này có thể được gọi khi có thông báo về cuộc gọi đến, cho phép ứng dụng xử lý thông tin, hiển thị thông báo cho người dùng

---
### Thao tác trong cuộc gọi

```kotlin
// Kiểm tra xem có call hay không
fun hasActiveCall(): Boolean

// Quay lại màn hình cuộc gọi
fun returnToCallVCIfNeeded()

// Ẩn màn hình cuộc gọi
fun hideCallVCIfNeeded()
```

### Đa ngôn ngữ

VBot SDK mặc định chỉ hỗ trợ Tiếng Việt

Khi app thay đổi ngôn ngữ, hãy dùng hàm **setLocalizationStrings(strings)** để thay đổi ngôn ngữ cho VBot SDK

```kotlin
fun setLocalizationStrings(values: Map<String, String?>)
```
Truyền dữ liệu qua metadata khi gọi hàm startIncomingCall


Các key ngôn ngữ đang sử dụng trong VBot SDK
```kotlin
				"call_btn_messsage" to "Nhắn tin",
        "call_btn_mute" to "Im lặng",
        "call_btn_speaker" to "Loa ngoài",
        "call_calling" to "Đang gọi",
        "call_connecting" to "Đang kết nối",
        "call_end" to "Kết thúc",
        "call_lost_connection" to "Mất kết nối",
        "call_refused" to "Người nhận từ chối cuộc gọi",
        "call_ringing" to "Đang gọi bạn",
        "call_early" to "Đang đổ chuông",
        "call_title" to "Gọi miễn phí",
        "call_busy" to "Máy bận",
        "call_temporarily_unavailable" to "Không liên lạc được",

        "call_failed_api" to "Không thể thực hiện cuộc gọi. Vui lòng thử lại.",
        "call_failed_no_connection" to "Không có mạng. Vui lòng thử lại.",
        "call_weak_signal" to "Sóng yếu",

        "call_permission_microphone_title" to "Xanh SM muốn truy cập micrô trên thiết bị của bạn.",
        "call_permission_microphone_content" to "Việc này cho phép ứng dụng thực hiện cuộc gọi miễn phí trong ứng dụng.",
        "call_permission_btn_allow" to "Cho phép",
        "call_permission_btn_deny" to "Không cho phép",

        "call_permission_microphone_demied_content" to "Vui lòng cho phép ứng dụng truy cập \"Microphone\" trong Cài đặt điện thoại của bạn.",
        "call_permission_microphone_demied_title" to "Không thể thực hiện cuộc gọi do chưa có quyền truy cập “Micrô\"",
        "call_permission_btn_setting" to "Đi đến Cài đặt",
        "call_permission_btn_skip" to "Bỏ qua",

        "call_notification_answer" to "Trả lời",
        "call_notification_end" to "Kết thúc",
        "call_speaker_bluetooth" to "Loa Bluetooth",
        "call_speaker_headset" to "Tai nghe",
        "call_speaker_speaker" to "Loa ngoài",
        "call_speaker_earpiece" to "Loa trong",
        "call_speaker_title" to "Đầu phát âm thanh",
```
