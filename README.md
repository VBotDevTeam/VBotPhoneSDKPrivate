# VBotPhoneSDK Android

SDK tích hợp cuộc gọi VoIP cho ứng dụng Android, hỗ trợ quản lý cuộc gọi đến/đi thông qua Firebase Cloud Messaging (FCM).

## Code demo

[https://github.com/VBotDevTeam/VBot-SDK-Android-Example](https://github.com/VBotDevTeam/VBot-SDK-Android-Example)

## Yêu cầu

- Android SDK version 23 (Android 6.0) trở lên

---

## Cấu hình SDK

### Cách 1

Trong file **app** → **build.gradle**, thêm các thư viện cần thiết để SDK hoạt động:

```kotlin
dependencies {
    // Các thư viện cần thiết để SDK hoạt động
    implementation("io.reactivex.rxjava2:rxjava:2.2.21")
    implementation("com.google.code.gson:gson:2.11.0")
    implementation("com.squareup.retrofit2:adapter-rxjava2:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.11.0")
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("org.reactivestreams:reactive-streams:1.0.4")
    implementation("com.squareup.okhttp3:okhttp:5.0.0-alpha.14")
    implementation("com.squareup.okhttp3:logging-interceptor:5.0.0-alpha.14")
    implementation("com.jakewharton.timber:timber:5.0.1")
    implementation("com.squareup.okhttp3:okhttp-dnsoverhttps:4.9.0")
    implementation("com.madgag.spongycastle:core:1.58.0.0")

    // Thêm SDK
    implementation 'com.github.VBotDevTeam:VBotPhoneSDKPrivate:2.0.24'
}
```

Trong file **settings.gradle** thêm:

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
- Trong vào ô _'Git repo url'_ nhập **VBotDevTeam/VBotPhoneSDKPrivate**
- Nhấn **Look up**
- Chọn version và nhấn **Get it**
- Làm theo hướng dẫn trên trang web

---

## Quyền cần thiết

1. **Quyền truy cập điện thoại** (`READ_PHONE_STATE`): Kiểm tra trạng thái thiết bị (cuộc gọi đến/đi, đang bận, hoặc rảnh). **Bắt buộc**.
2. **Quyền truy cập micro** (`RECORD_AUDIO`): Thu âm giọng nói để gửi trong các cuộc gọi VoIP. **Bắt buộc**.
3. **Quyền truy cập thiết bị ở gần** (`BLUETOOTH_CONNECT`): Kết nối với các thiết bị Bluetooth (Android 12+).
4. **Quyền thông báo** (`POST_NOTIFICATIONS`): Hiển thị thông báo cuộc gọi đến (Android 13+).
5. **Quyền hiển thị trên ứng dụng khác** (`SYSTEM_ALERT_WINDOW`): Hiển thị thông báo cuộc gọi đến dưới dạng pop-up khi ứng dụng đang ở chế độ nền hoặc màn hình khóa.
6. **Quyền cho phép sử dụng dữ liệu không hạn chế**: Khi hiển thị thông báo cuộc gọi thì SDK có quyền này sẽ hoạt động tốt hơn, kết nối với cuộc gọi nhanh hơn.

---

## Proguard

VBotPhoneSDK yêu cầu các quy tắc ProGuard sau. Thêm vào file `proguard-rules.pro` của ứng dụng:

```
-keep class com.vpmedia.vbotphonesdk.** {*;}
```

---

## Sử dụng SDK

### 1. Khởi tạo

```kotlin
val phone = VBotPhone.setup(context, token) { code, message ->
    // Xử lý lỗi khi khởi tạo thất bại
    Log.e("VBot", "Setup error: $code - $message")
}
```

Trong đó:

- **context**: Application context
- **token**: App Token đại diện cho ứng dụng, dùng để xác thực với máy chủ VBot
- **onFailure**: Callback khi khởi tạo thất bại (ví dụ: token không hợp lệ)

> **Lưu ý:** `setup()` trả về instance `VBotPhone`. Có thể gọi lại setup để thay đổi token khi cần (xem mục Multi-Tenant).

### 2. Lắng nghe sự kiện

```kotlin
val listener = object : VBotListener() {

    override fun onCallState(state: VBotCallState) {
        // Trạng thái cuộc gọi thay đổi
        // VBotCallState: NONE, CALLING, INCOMING, EARLY, CONNECTING, CONFIRMED, DISCONNECTED
    }

    override fun callAccepted() {
        // Cuộc gọi đến được chấp nhận
    }

    override fun callEnded(reason: VBotEndCallReason) {
        // Cuộc gọi kết thúc với lý do cụ thể
    }
}

// Đăng ký listener
phone.addListener(listener)

// Hủy đăng ký khi không cần nữa
phone.removeListener(listener)
```

### 3. Gọi đi

```kotlin
phone.startOutgoingCall(
    callerId = "caller_id",
    callerName = "Tên người gọi",
    calleeId = "callee_id",
    calleeName = "Tên người nghe",
    checkSum = "mã_xác_thực_cuộc_gọi",
    callerAvatar = "https://...",              // Tuỳ chọn
    calleeAvatar = "https://...",              // Tuỳ chọn
    metadata = hashMapOf("key" to "value"),    // Tuỳ chọn
    onFailure = { code, message ->             // Tuỳ chọn
        Log.e("VBot", "Lỗi: $code - $message")
    }
)
```

### 4. Gọi đến (nhận cuộc gọi qua FCM)

Tạo một `FirebaseMessagingService` để nhận thông báo:

```kotlin
class VBotFirebaseMessagingService : FirebaseMessagingService() {

    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        val data = remoteMessage.data
        val metaDataStr = data["metaData"] ?: return

        val metaDataJson = JSONObject(metaDataStr)
        val type = metaDataJson.optString("type", "")

        if (type == "call") {
            val metaDataMap = HashMap<String, String>()
            metaDataJson.keys().forEach { key ->
                metaDataMap[key] = metaDataJson.getString(key)
            }

            val checkSum = metaDataJson.optString("check_sum", "")

            // Khởi tạo SDK nếu chưa init (khi app bị kill)
            val phone = VBotPhone.setup(applicationContext, token) { code, msg ->
                Log.e("VBot", "Setup failed: $code - $msg")
            }

            phone.startIncomingCall(
                callerId = data["callerId"] ?: "",
                callerAvatar = data["callerAvatar"],
                callerName = data["callerName"] ?: "",
                calleeId = data["calleeId"] ?: "",
                calleeAvatar = data["calleeAvatar"],
                calleeName = data["calleeName"] ?: "",
                checkSum = checkSum,
                metadata = metaDataMap,
                onFailure = { code, message ->
                    Log.e("VBot", "Incoming call error: $code - $message")
                }
            )
        }
    }

    override fun onNewToken(token: String) {
        // Gửi FCM token lên server VBot
    }
}
```

Đăng ký service trong `AndroidManifest.xml`:

```xml
<service
    android:name=".firebase.VBotFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
```

---

## Thao tác trong cuộc gọi

```kotlin
// Kiểm tra xem có cuộc gọi active hay không
phone.hasActiveCall()

// Quay lại màn hình cuộc gọi
phone.returnToCallVCIfNeeded()

// Ẩn màn hình cuộc gọi
phone.hideCallVCIfNeeded()

// Huỷ SDK
phone.destroy(
    onSuccess = { /* cleanup */ },
    onFailure = { code, message -> /* xử lý lỗi */ }
)
```

---

## Đa ngôn ngữ

VBot SDK mặc định hỗ trợ Tiếng Việt. Để thay đổi ngôn ngữ cho SDK, truyền dữ liệu qua `metadata` khi gọi hàm `startIncomingCall` hoặc `startOutgoingCall`.

Các key ngôn ngữ đang sử dụng trong VBot SDK:

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
"call_permission_microphone_demied_title" to "Không thể thực hiện cuộc gọi do chưa có quyền truy cập "Micrô\"",
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

---

## Sử dụng với nhiều token (Multi-Tenant)

SDK hỗ trợ multi-tenant — mỗi tenant/tổ chức sử dụng một App Token riêng. Khi user thuộc nhiều tổ chức, app cần quản lý việc chuyển đổi token.

### Luồng hoạt động

```
User đăng nhập → Lấy danh sách tenant → Chọn tenant → setup(token) → Đăng ký FCM Token lên server
                                              ↓
                                     Chuyển tenant → setup(token mới) → Đăng ký FCM Token lại
```

### Triển khai

```kotlin
// MARK: - Chuyển đổi giữa các tenant

fun switchTenant(context: Context, newToken: String) {
    // 1. Setup lại SDK bằng Token của Tenant mới
    val phone = VBotPhone.setup(context, newToken) { code, message ->
        Log.e("VBot", "Switch tenant failed: $code - $message")
    }

    // 2. Lấy FCM Token hiện tại
    FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
        if (task.isSuccessful) {
            val fcmToken = task.result

            // 3. Gọi API đăng ký FCM Token lên hệ thống VBot
            // Endpoint: POST /sdk/push-token
            // Body: { deviceId, deviceName, firebaseToken, isProduct }
            registerPushToken(fcmToken, isProduct = false)
        }
    }
}
```

```kotlin
// MARK: - Khôi phục khi mở app

// Khi app khởi động hoặc được khởi tạo lại từ trạng thái bị kill
fun restoreLastTenant(context: Context) {
    val savedToken = getSavedTenantToken() ?: return

    VBotPhone.setup(context, savedToken) { code, message ->
        Log.e("VBot", "Restore failed: $code - $message")
    }
}
```

> **Lưu ý quan trọng:**
>
> - Khi chuyển tenant, **bắt buộc** phải gọi API đăng ký lại FCM Token lên server VBot, nếu không cuộc gọi đến sẽ không hoạt động đúng.
> - Mỗi thời điểm chỉ có **một tenant active** — SDK không hỗ trợ nhận cuộc gọi từ nhiều tenant cùng lúc.

---

## Tham khảo

### VBotCallState

```kotlin
enum class VBotCallState {
    NONE,           // Không có cuộc gọi
    CALLING,        // Đang gọi đi
    INCOMING,       // Cuộc gọi đến
    EARLY,          // Đổ chuông
    CONNECTING,     // Đang kết nối
    CONFIRMED,      // Kết nối thành công
    DISCONNECTED    // Kết thúc
}
```

### VBotEndCallReason

```kotlin
enum class VBotEndCallReason(val code: Int) {
    TimeOut(-1001),                    // Timeout
    InitiationFailed(1001),            // Khởi tạo không thành công
    InitiationFailed_1(1002),          // Không sẵn sàng để gọi đi
    MicrophonePermissionDenied(1003),  // Chưa cấp quyền mic
    InvalidPhoneNumber(1004),          // Số điện thoại không hợp lệ
    NoDataFromServer(1005),            // Không có dữ liệu từ máy chủ
    InitiationFailed_2(1006),          // Kết thúc trước khi server bắt đầu
    InitiationFailed_3(1007),          // Lỗi khi khởi tạo cuộc gọi
    DataInvalid(1008),                 // Dữ liệu không hợp lệ
    InitiationFailed_4(1009),          // Tài khoản VBot không tồn tại
    AuthenticatedFailed(1010),         // Xác thực thất bại
    AnotherCallInProgress(1011),       // Đang có cuộc gọi khác
    Normal(1012),                      // Cuộc gọi kết thúc bình thường
    Decline(1013),                     // Từ chối cuộc gọi
    TemporarilyUnavailable(1014),      // Không liên lạc được
    Busy(1015),                        // Máy bận
    ReportNewIncomingCallFailed(1016), // Báo cáo cuộc gọi đến thất bại
    UnknownError(1999)                 // Lỗi chưa xác định
}
```
