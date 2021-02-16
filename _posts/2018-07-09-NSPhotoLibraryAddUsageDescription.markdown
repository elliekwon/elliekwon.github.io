---
title: NSPhotoLibraryAddUsageDescription 크래시
categories: [iOS, Crash]
tags: [crash fix]
---

## 이슈
Rollbar를 통해서 앱크래시 이슈가 등록되었다. 그 내용은 아래와 같다.

```
SIGABRT: Application terminated
1  libsystem_kernel.dylib in __abort_with_payload
2  libsystem_kernel.dylib in system_set_sfi_window
3  TCC in __TCCAccessRequest_block_invoke.85
4  TCC in __CRASHING_DUE_TO_PRIVACY_VIOLATION__
5  TCC in __tccd_send_block_invoke
6  libxpc.dylib in _xpc_connection_reply_callout
7  libxpc.dylib in _xpc_connection_call_reply_async
8  libdispatch.dylib in _dispatch_client_callout3
9  libdispatch.dylib in _dispatch_mach_msg_async_reply_invoke$VARIANT$mp
10 libdispatch.dylib in _dispatch_queue_override_invoke$VARIANT$mp
11 libdispatch.dylib in _dispatch_root_queue_drain
12 libdispatch.dylib in _dispatch_worker_thread3
13 libsystem_pthread.dylib in _pthread_wqthread
14 libsystem_pthread.dylib in start_wqthread
```
유저는 iOS11에 iPhone7모델을 사용중이란다.

프라이버시라는 부분을 보면 아무래도 권한 문제인 것 같기는 한데, 아무래도 짐작가는 바가 없다.  
구글 검색을 해보니 토씨하나 들리지 않고 동일한 크래시로그를 발견할 수 있었다.
(https://stackoverflow.com/questions/46765852/crash-on-ios-tccaccessrequest-block-invoke-2-8)

## 원인 및 대응

**< iOS6~ >**

```
NSPhotoLibraryUsageDescription
(Privacy - Photo Library Usage Description)
```

iOS10에서는 사진 라이브러리 액세스에 대해서는 info.plist에 이 키 하나를 추가하는 것으로 충분했다.  
하지만 iOS11부터는 읽기와 쓰기가 분리가 되어, 위의 키는 읽기에 대한 권한만을 갖게되는 것 같다.

**< iOS11~ >**

```
NSPhotoLibraryAddUsageDescription
(Privacy - Photo Library Additions Usage Description)
```

사진을 저장하는(라이브러리에 쓰는) 권한을 위해서 iOS11에서는 새롭게 위의 키를 적어주어야 하며, 없을 경우에는 크래시가 발생할 수 있다.

## 정리

사실은 iOS11단말에서 나름 다양한 케이스들을 만들어 테스트를 했음에도 불구하고 이 크래시를 동일하게 재현할 수 없었다.  

다만 `NSPhotoLibraryUsageDescription`와 `NSPhotoLibraryAddUsageDescription`를  
모두 써놓는 것이 좋겠다는 의견글에 따라서 나 역시 info.plist에 후자의 키를 새롭게 추가해두었다.

이것으로 더이상 위와 동일한 크래시 이슈가 보고되지 않기를 기대해본다.
