---
author: PatrickFarley
ms.service: azure-speech-foundry-tools
ms.topic: include
ms.date: 10/09/2026
ms.author: pafarley
ai-usage: ai-assisted
---

[!INCLUDE [Header](../../common/swift.md)]

In this quickstart, you install the [Speech SDK](~/articles/ai-services/speech-service/speech-sdk.md) for Swift.

> [!TIP]
> For more information about using the Speech SDK for Swift, see [Importing Objective-C into Swift](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift).

## Install the Speech SDK for Swift

# [Mac](#tab/mac)

Install the Speech SDK for Swift on Mac with Swift Package Manager.

System requirements for the Mac app:

- Target macOS 10.14 or later

Use Xcode 12 or later to add the Swift package.

1. Open your *.xcodeproj* project in Xcode.
1. Select **File** > **Add Package Dependencies**.
1. Enter `https://github.com/microsoft/speech-sdk-spm` and select a version starting at `1.51.0`.
1. Add the `MicrosoftCognitiveServicesSpeech-macOS` product to your app target.

Alternatively, download the [SDK binary](https://aka.ms/csspeech/macosbinary) and extract its contents. In your Xcode project, add a reference to the extracted *MicrosoftCognitiveServicesSpeech.xcframework* folder and its contents.

# [iOS](#tab/ios)

Install the Speech SDK for Swift on iOS with Swift Package Manager.

System requirements for iOS:

- Target iOS 11.0 or later

Use Xcode 12 or later to add the Swift package.

1. Open your *.xcodeproj* project in Xcode.
1. Select **File** > **Add Package Dependencies**.
1. Enter `https://github.com/microsoft/speech-sdk-spm` and select a version starting at `1.51.0`.
1. Add either `MicrosoftCognitiveServicesSpeech-iOS` (standard) or `MicrosoftCognitiveServicesSpeechEmbedded-iOS` (on-device/embedded) to your app target. Select only one product.

Alternatively, download the [SDK binary](https://aka.ms/csspeech/iosbinary) and extract its contents. In your Xcode project, add a reference to the extracted *MicrosoftCognitiveServicesSpeech.xcframework* folder and its contents.

---
