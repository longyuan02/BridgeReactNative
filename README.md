[OTC] 
###原生应用植入ReactNative
    |object目录
    |package.json
    |index.android.js

##### package.json
<pre><code>{
  "name": "工程名",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "dependencies": {
    "react": "16.0.0-alpha.6",
    "react-native": "0.44.3"
  }
}</code></pre>  

##### index.android.js
<pre><code>'use strict';
import React from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native';
class HelloWorld extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.hello}>Hello, React Native</Text>
      </View>
    )
  }
}
var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  hello: {
    fontSize: 20,
    color:'#FFFF00',
    textAlign: 'center',
    margin: 10,
  },
});
AppRegistry.registerComponent('BridgeReactNative', () => HelloWorld);</code></pre>

#### 添加库引用
project->|android|defaultConfig
                               |ndk {abiFilters "armeabi-v7a", "x86"}
         |dependencies 
                     |implementation "com.facebook.react:react-native:+"
         
objcecct->|allprojects|repositories|maven {url "$rootDir/node_modules/react-native/android"}


#### 执行命令
        nmp install 
        执行后会产生一个"node_modules"文件夹
        npm start 开始执行


#### AndroidManifest.xml
        <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
        <activity
            android:name=".ReactActivity"
            android:label="@string/app_name"
            android:theme="@style/Theme.AppCompat.Light.NoActionBar"></activity>
            
### reactNative加载类
<pre><code>public class ReactActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler {
               public static final int OVERLAY_PERMISSION_REQ_CODE = 1235;
               private ReactRootView mReactRootView;
               private ReactInstanceManager mReactInstanceManager;
           
               @Override
               protected void onCreate(Bundle savedInstanceState) {
                   super.onCreate(savedInstanceState);
                   setContentView(R.layout.activity_react);
                   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                       if (!Settings.canDrawOverlays(this)) {
                           Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + getPackageName()));
                           startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
                       }
                   }
           
                   mReactRootView = new ReactRootView(this);
                   mReactInstanceManager = ReactInstanceManager.builder()
                           .setApplication(getApplication())
                           .setBundleAssetName("index.android.bundle")
                           .setJSMainModuleName("index.android")
                           .addPackage(new MainReactPackage())
           //                .setUseDeveloperSupport(BuildConfig.DEBUG)
                           .setUseDeveloperSupport(true)
                           .setInitialLifecycleState(LifecycleState.RESUMED)
                           .build();// 注意这里的BridgeReactNative必须对应“index.android.js”中的
                   // “AppRegistry.registerComponent()”的第一个参数
                   mReactRootView.startReactApplication(mReactInstanceManager, "BridgeReactNative", null);
                   setContentView(mReactRootView);
               }
           
               @Override
               public void invokeDefaultOnBackPressed() {
                   super.onBackPressed();
               }
           
               @Override
               protected void onActivityResult(int requestCode, int resultCode, Intent data) {
                   // super.onActivityResult(requestCode, resultCode, data);
                   if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
                       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                           if (!Settings.canDrawOverlays(this)) {
                               // SYSTEM_ALERT_WINDOW permission not granted...
                           }
                       }
                   }
               }
           
               @Override
               protected void onPause() {
                   super.onPause();
           
                   if (mReactInstanceManager != null) {
                       mReactInstanceManager.onHostPause(this);
                   }
               }
           
               @Override
               protected void onResume() {
                   super.onResume();
           
                   if (mReactInstanceManager != null) {
                       mReactInstanceManager.onHostResume(this, this);
                   }
               }
           
               @Override
               protected void onDestroy() {
                   super.onDestroy();
           
                   if (mReactInstanceManager != null) {
                       mReactInstanceManager.onHostDestroy(this);
                   }
               }
           
               @Override
               public void onBackPressed() {
                   if (mReactInstanceManager != null) {
                       mReactInstanceManager.onBackPressed();
                   } else {
                       super.onBackPressed();
                   }
               }
           
               @Override
               public boolean onKeyUp(int keyCode, KeyEvent event) {
                   if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
                       mReactInstanceManager.showDevOptionsDialog();
                       return true;
                   }
                   return super.onKeyUp(keyCode, event);
               }
           }</code></pre># BridgeReactNative
