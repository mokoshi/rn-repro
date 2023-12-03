# RnRepro

## Steps to reproduce

### 1. Check if building with Nx monorepo will success at the initial state

```shell
npm install

# Create a node_modules symlink.
# > repro/node_modules -> node_modules
npx nx ensure-symlink repro

cd repro/ios
bundle install
bundle exec pod deintegrate
bundle exec pod install
```

Open `repro/ios/Repro.xcworkspace` on Xcode.
I can build it successfully.

### 2. Modifying Podfile (use_frameworks!) cause an issue.

````diff
diff --git a/repro/ios/Podfile b/repro/ios/Podfile
index 0057d30..d78bf45 100644
--- a/repro/ios/Podfile
+++ b/repro/ios/Podfile
@@ -17,7 +17,7 @@ prepare_react_native_project!
 #   dependencies: {
 #     ...(process.env.NO_FLIPPER ? { 'react-native-flipper': { platforms: { ios: null } } } : {}),
 # ```
-flipper_config = ENV['NO_FLIPPER'] == "1" ? FlipperConfiguration.disabled : FlipperConfiguration.enabled
+flipper_config = FlipperConfiguration.disabled

 linkage = ENV['USE_FRAMEWORKS']
 if linkage != nil
@@ -27,6 +27,7 @@ end

 target 'Repro' do
   config = use_native_modules!
+  use_frameworks! :linkage => :static

   # Flags change depending on the env values.
   flags = get_default_flags()

````

```shell
cd repro/ios
bundle install
bundle exec pod deintegrate
bundle exec pod install
```

Open `repro/ios/Repro.xcworkspace` on Xcode.
The build will fail with an error:

```
***/rn-repro/node_modules/react-native/ReactCommon/react/utils/RunLoopObserver.cpp:10:10: 'react/debug/react_native_assert.h' file not found
```

### 3. Removing node_modules symlink will fix the problem on native-side build.

```shell
rm repro/node_modules

cd repro/ios
bundle install
bundle exec pod deintegrate
bundle exec pod install
```

Open `repro/ios/Repro.xcworkspace` on Xcode and build.
The error on step2 will not be thrown, though the build will fail at JavaScript bundle phase because of lack of monorepo settings.
