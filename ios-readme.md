# Font Implementation Guide for React Native SDK
> Last Updated: 2025-04-05 11:00:26 UTC  
> Author: @thtRajasthaniGuy

[Previous sections remain the same until iOS Implementation]

## iOS Implementation

### 1. Configure iOS Project

#### 1.1 Update Podfile
Create or update your `ios/Podfile`:

```ruby
require_relative '../node_modules/react-native/scripts/react_native_pods'
require_relative '../node_modules/@react-native-community/cli-platform-ios/native_modules'

platform :ios, min_ios_version_supported
prepare_react_native_project!

def shared_pods
  config = use_native_modules!
  use_react_native!(
    :path => config[:reactNativePath],
    :hermes_enabled => true
  )
end

target 'YourSDK' do
  shared_pods
end
```

#### 1.2 Create Font Manager
Create a new file `ios/FontManager.h`:

```objc
#import <Foundation/Foundation.h>
#import <React/RCTBridgeModule.h>

@interface FontManager : NSObject <RCTBridgeModule>
@end
```

Create `ios/FontManager.m`:

```objc
#import "FontManager.h"
#import <CoreText/CoreText.h>

@implementation FontManager

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(initializeFonts:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
    NSBundle *bundle = [NSBundle bundleForClass:[self class]];
    NSArray *fontURLs = [[NSBundle mainBundle] URLsForResourcesWithExtension:@"ttf" subdirectory:nil];
    
    if ([fontURLs count] == 0) {
        NSError *error = [NSError errorWithDomain:@"FontManagerErrorDomain" 
                                           code:404 
                                       userInfo:@{NSLocalizedDescriptionKey: @"No fonts found"}];
        reject(@"no_fonts", @"No fonts found in the bundle", error);
        return;
    }
    
    NSMutableArray *loadedFonts = [NSMutableArray array];
    NSError *error = nil;
    
    for (NSURL *fontURL in fontURLs) {
        CGDataProviderRef provider = CGDataProviderCreateWithURL((__bridge CFURLRef)fontURL);
        if (provider) {
            CGFontRef font = CGFontCreateWithDataProvider(provider);
            if (font) {
                if (CTFontManagerRegisterGraphicsFont(font, &error)) {
                    NSString *fontName = [fontURL lastPathComponent];
                    [loadedFonts addObject:fontName];
                }
                CFRelease(font);
            }
            CGDataProviderRelease(provider);
        }
    }
    
    if ([loadedFonts count] > 0) {
        resolve([NSString stringWithFormat:@"Loaded fonts: %@", [loadedFonts componentsJoinedByString:@", "]]);
    } else {
        if (error) {
            reject(@"font_load_error", @"Failed to load fonts", error);
        } else {
            reject(@"font_load_error", @"No fonts were loaded", nil);
        }
    }
}

@end
```

#### 1.3 Update Info.plist
Add font files to your `ios/Info.plist`:

```xml
<key>UIAppFonts</key>
<array>
    <string>Noto_Sans-Regular.ttf</string>
    <string>Noto_Sans-Bold.ttf</string>
</array>
```

### 2. Font Installation Process

#### 2.1 Create Resource Bundle
Create `ios/Resources/Fonts.bundle`:

```bash
mkdir -p ios/Resources/Fonts.bundle
cp src/assets/Noto_Sans/*.ttf ios/Resources/Fonts.bundle/
```

#### 2.2 Update Xcode Project
1. Open Xcode
2. Right-click on your project
3. Select "Add Files to YourSDK"
4. Choose `Fonts.bundle`
5. Ensure "Copy items if needed" is checked
6. Click "Add"

#### 2.3 Update podspec
Create or update `your-sdk.podspec`:

```ruby
require 'json'

package = JSON.parse(File.read(File.join(__dir__, 'package.json')))

Pod::Spec.new do |s|
  s.name         = package['name']
  s.version      = package['version']
  s.summary      = package['description']
  s.license      = package['license']
  s.authors      = package['author']
  s.homepage     = package['homepage']
  s.platform     = :ios, "11.0"
  s.source       = { :git => "https://github.com/yourusername/your-sdk.git", :tag => "#{s.version}" }
  s.source_files = "ios/**/*.{h,m,mm,swift}"
  
  # Add this section for fonts
  s.resource_bundles = {
    'Fonts' => ['src/assets/Noto_Sans/*.ttf']
  }
  
  s.dependency "React-Core"
end
```

### 3. Build and Test

#### 3.1 Install Dependencies
```bash
cd ios
pod install
cd ..
```

#### 3.2 Build SDK
```bash
yarn prepare
cd ios
xcodebuild -workspace YourSDK.xcworkspace -scheme YourSDK
cd ..
```

### 4. Verification

#### 4.1 Check Font Installation
```swift
// In your iOS app
UIFont.familyNames.forEach { familyName in
    print("Family: \(familyName)")
    UIFont.fontNames(forFamilyName: familyName).forEach { fontName in
        print("Font: \(fontName)")
    }
}
```

#### 4.2 Debug Font Loading
In Xcode:
1. Open Console
2. Filter for "FontManager"

### 5. iOS-Specific Troubleshooting

#### Common Issues

1. **Fonts Not Found in Bundle**
   - ✓ Verify fonts are in Fonts.bundle
   - ✓ Check Info.plist entries
   - ✓ Rebuild and clean derived data

2. **Font Registration Fails**
   - ✓ Check font file integrity
   - ✓ Verify font name matches Info.plist
   - ✓ Ensure unique font names

3. **Podspec Issues**
   - ✓ Validate podspec syntax
   - ✓ Check resource_bundles configuration
   - ✓ Run `pod lib lint`

## Cross-Platform Usage

```typescript
// In your React Native component
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  text: {
    fontFamily: Platform.select({
      ios: 'Noto_Sans-Regular',
      android: 'Noto_Sans-Regular',
    }),
    fontSize: 16,
  },
});
```

## Additional iOS Notes
- Always test on both iOS simulator and real devices
- Font names in iOS are case-sensitive
- Use exact font names as shown in Font Book
- Clean build folder if fonts don't appear after changes

[Rest of the previous README remains the same]
