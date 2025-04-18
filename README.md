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
    implementation("org.reactivestreams:reactive-streams:1.0.4")
    implementation ("com.squareup.okhttp3:okhttp:5.0.0-alpha.14")
    implementation("com.jakewharton.timber:timber:5.0.1")
    implementation("com.squareup.okhttp3:okhttp-dnsoverhttps:4.9.0")
    
		//Thêm SDK
    implementation 'com.github.VBotDevTeam:VBotPhoneSDKPrivate:2.0.9'
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

5.	**Quyền hiển thị trên ứng dụng khác** 

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
Từ ứng dụng của bạn, có thể gọi vào các hàm khác nhau của VBot SDK để thực hiện các hành động trong cuộc gọi

Ví dụ:


```kotlin
// Kiểm tra xem có call hay không
fun hasActiveCall(): Boolean

// Quay lại màn hình cuộc gọi
fun returnToCallVCIfNeeded()

// Ẩn màn hình cuộc gọi
fun hideCallVCIfNeeded()
```

---
### Lắng nghe sự kiện

```kotlin

// Đăng ký nhận sự kiện
fun addListener(listener: VBotListener)

// Hủy đăng ký
fun removeListener(listener: VBotListener)

```

```kotlin
class VBotListener {

		// Trạng thái cuộc gọi thay đổi
    override fun onCallState(state: VBotCallState) {}
		
		// // Cuộc gọi đến được chấp nhận (Khi user chọn chấp nhận cuộc gọi)
    override fun callAccepted() {}
		
     // Cuộc gọi kết thúc, cùng nguyên nhân
    override fun callEnded(reason: VBotEndCallReason) {}
    
    // Nhấn vào nút nhắn tin
    override fun onMessageButtonClick() {}
}
```



---
### Đa ngôn ngữ

Các màn hình cuộc gọi VBot SDK mặc định chỉ hỗ trợ Tiếng Việt

Để thay đổi ngôn ngữ truyền dữ liệu qua metadata khi gọi hàm startIncomingCall và startOutgoingCall

---
### Xem thêm
#### VBotEndCallReason và VBotError


```
    // Timeout
    case timeOut = -1001
    
    // Khởi tạo không thành công
    case initiationFailed = 1001
    
    case initiationFailed_1 = 1002
    
    // Chưa cấp truyền mic
    case microphonePermissionDenied = 1003
    
    case invalidPhoneNumber = 1004
    
    // Không có dữ liệu từ máy chủ
    case noDataFromServer = 1005
    
    case initiationFailed_2 = 1006
    
    case initiationFailed_3 = 1007
    
    // Dữ liệu không hợp lệ
    case dataInvalid = 1008
    
    case initiationFailed_4 = 1009
    
    // Xác thực thất bại
    case authenticatedFailed = 1010
    
    // Đang có cuộc gọi khác
    case anotherCallInProgress = 1011
    
    // Cuộc gọi kết thúc
    case normal = 1012
    
    // Từ chối cuộc gọi
    case decline = 1013
    
    // Không liên lạc được
    case temporarilyUnavailable = 1014
    
    // Máy bận
    case busy = 1015
    
    // reportNewIncomingCall lỗi
    case reportNewIncomingCallFailed = 1016
    
    // Lỗi chưa xác định
    case unknownError = 1999
```




