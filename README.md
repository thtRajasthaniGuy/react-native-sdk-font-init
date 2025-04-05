# Font Implementation Guide for React Native SDK
> Last Updated: 2025-04-05 10:48:02 UTC  
> Author: @thtRajasthaniGuy

This guide provides comprehensive instructions for implementing custom fonts in a React Native SDK.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Step-by-Step Implementation](#step-by-step-implementation)
  - [1. Font Files Setup](#1-font-files-setup)
  - [2. Configuration Files](#2-configuration-files)
  - [3. Android Implementation](#3-android-implementation)
  - [4. Build Process](#4-build-process)
  - [5. Verification](#5-verification)
- [Troubleshooting](#troubleshooting)
- [Usage in React Native](#usage-in-react-native)

## Prerequisites

- React Native SDK project
- Custom font files (.ttf format)
- Android Studio
- Node.js and yarn/npm

## Project Structure

```bash
your-sdk/
├── src/
│   ├── assets/
│   │   └── Noto_Sans/         # Your font files
│   │       ├── Regular.ttf
│   │       └── Bold.ttf
├── android/
│   └── src/
│       └── main/
│           └── assets/
│               └── fonts/      # Android font destination
├── ios/
└── package.json
```

## Step-by-Step Implementation

### 1. Font Files Setup

#### 1.1 Create Font Directory
```bash
mkdir -p src/assets/Noto_Sans
```

#### 1.2 Add Font Files
```bash
cp path/to/your/fonts/*.ttf src/assets/Noto_Sans/
```

### 2. Configuration Files

#### 2.1 Update package.json
Add the following configurations to your package.json:

```json
{
  "name": "your-sdk",
  "assets": [
    "src/assets/Noto_Sans"
  ],
  "files": [
    "src",
    "lib",
    "android",
    "ios",
    "cpp",
    "*.podspec",
    "!lib/typescript/example",
    "!ios/build",
    "!android/build",
    "!android/gradle",
    "!android/gradlew",
    "!android/gradlew.bat",
    "!android/local.properties",
    "!**/__tests__",
    "!**/__fixtures__",
    "!**/__mocks__",
    "!**/.*",
    "src/assets/Noto_Sans/**/*"
  ],
  "react-native-builder-bob": {
    "source": "src",
    "output": "lib",
    "targets": [
      ["commonjs", {
        "esm": true,
        "copy": [
          { 
            "source": "src/assets/Noto_Sans", 
            "dest": "lib/commonjs/assets/Noto_Sans"
          }
        ]
      }],
      ["module", {
        "esm": true,
        "copy": [
          { 
            "source": "src/assets/Noto_Sans", 
            "dest": "lib/module/assets/Noto_Sans"
          }
        ]
      }],
      ["typescript", {
        "esm": true
      }]
    ]
  }
}
```

### 3. Android Implementation

#### 3.1 Update android/build.gradle
```gradle
android {
  // ... other configs ...

  sourceSets {
    main {
      assets.srcDirs = ['src/main/assets', '../src/assets/Noto_Sans']
    }
  }
}

// Add copyFonts task
task copyFonts(type: Copy) {
    description = "Copies fonts from src/assets to Android assets"
    from("../src/assets/Noto_Sans")
    into("src/main/assets/fonts")
    include('**/*.ttf')
}

// Make sure fonts are copied before building
preBuild.dependsOn copyFonts
```

#### 3.2 Create PaymentPgModule.java
Create a new file at `android/src/main/java/com/paymentpg/PaymentPgModule.java`:

```java
package com.paymentpg;

import androidx.annotation.NonNull;
import android.graphics.Typeface;
import android.util.Log;
import java.util.Arrays;

import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.module.annotations.ReactModule;

@ReactModule(name = PaymentPgModule.NAME)
public class PaymentPgModule extends ReactContextBaseJavaModule {
    public static final String NAME = "PaymentPg";
    private static final String TAG = "PaymentPgModule";
    private final ReactApplicationContext reactContext;

    public PaymentPgModule(ReactApplicationContext reactContext) {
        super(reactContext);
        this.reactContext = reactContext;
    }

    @Override
    @NonNull
    public String getName() {
        return NAME;
    }

    @ReactMethod
    public void initializeFonts(Promise promise) {
        try {
            Log.d(TAG, "Starting font initialization...");
            String[] fonts = this.reactContext.getAssets().list("fonts");
            Log.d(TAG, "Found fonts: " + Arrays.toString(fonts));
            
            if (fonts != null && fonts.length > 0) {
                for (String font : fonts) {
                    try {
                        Typeface typeface = Typeface.createFromAsset(this.reactContext.getAssets(), "fonts/" + font);
                        Log.d(TAG, "Successfully loaded font: " + font);
                    } catch (Exception e) {
                        Log.e(TAG, "Error loading font: " + font, e);
                    }
                }
                promise.resolve("Fonts loaded successfully");
            } else {
                promise.reject("NO_FONTS", "No fonts found in assets/fonts directory");
            }
        } catch (Exception e) {
            promise.reject("FONT_INIT_ERROR", e.getMessage());
        }
    }
}
```

### 4. Build Process

#### 4.1 Build SDK
```bash
# In SDK directory
yarn prepare
cd android
./gradlew clean
./gradlew build
cd ..
```

#### 4.2 Integration in App
```bash
# In your app directory
cd android
./gradlew clean
cd ..
npx react-native run-android
```

### 5. Verification

#### 5.1 Check Font Files
```bash
# Check in project
ls -l android/src/main/assets/fonts/

# Check in device (Windows)
adb shell ls -l /storage/emulated/0/Android/data/com.yourapp/files/assets/fonts
```

#### 5.2 Check Logs
In Android Studio:
1. Open Logcat
2. Filter by "PaymentPgModule"
3. Look for font initialization messages

Via Command Line (Windows):
```bash
adb logcat | findstr PaymentPgModule
```

## Troubleshooting

### Common Issues and Solutions

1. **Fonts Not Found**
   - ✓ Check if fonts are in correct directory
   - ✓ Verify build.gradle copyFonts task
   - ✓ Clean and rebuild project

2. **Font Loading Fails**
   - ✓ Verify font file format (.ttf)
   - ✓ Check file permissions
   - ✓ Ensure font filenames have no spaces

3. **Build Errors**
   - ✓ Clean project and rebuild
   - ✓ Check gradle configuration
   - ✓ Verify all paths in configuration files

## Usage in React Native

```typescript
// In your component
import { StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  text: {
    fontFamily: 'Noto_Sans-Regular', // Use exact filename without .ttf
    fontSize: 16,
  },
});
```
## Usage in React Native Sdk

``` index.tsx in sdk src/index.tsx
 ---index.tsx in sdk src/index.tsx---
import { NativeModules } from 'react-native';
const LINKING_ERROR =
  `The package 'react-native-odfd' doesn't seem to be linked. Make sure: \n\n` +
 +
  '- You rebuilt the app after installing the package\n';

const PaymentPg = NativeModules.PaymentPg
  ? NativeModules.PaymentPg
  : new Proxy(
      {},
      {
        get() {
          throw new Error(LINKING_ERROR);
        },
      }
    );

export const initializeSDK = async () => {
  try {
    console.log('Initializing SDK and fonts...');
    await PaymentPg.initializeFonts();
    console.log('SDK and fonts initialized successfully');
    return true;
  } catch (error) {
    console.error('SDK initialization failed:', error);
    throw error;
  }
};
export * from './navigations';

```


## Important Notes
- Always test fonts in both debug and release builds
- Keep font filenames consistent across the project
- Document font usage for SDK consumers
- Verify font loading during SDK initialization

## Support
If you encounter any issues, please create an issue in the repository.

## License
This implementation guide is part of your SDK's license.

---
> 📝 Note: Replace 'your-sdk' and 'com.yourapp' with your actual SDK and app names throughout this guide.
