# faceid_auth_flutter

add local_auth to pubspec

import local_auth: 
  import 'package:local_auth/local_auth.dart';

create an instance:
```
  final LocalAuthentication localAuthentication = LocalAuthentication();
```
some of the things that we can do:
- check if biometric auth is supported
- get a list of available biometric types
- authenticate user using biometrics or pin/passcode

## Check for biometric authentication

You can check if the device supports biometric authentication or if it can 
use the device credentials (pin/passcode lock):
```
  bool isBiometricSupported = await localAuthentication.isDeviceSupported();
```
Now, if you want to verify whether biometric authentication is accessible 
from the app, you can check it using this:
 ```
 bool canCheckBiometrics = await localAuthentication.canCheckBiometrics;
```
## Retrieve list of biometric types

You can retrieve a list of biometric types that are supported by the 
device, using this:
```
  List<BiometricType> biometricTypes = await localAuthentication.getAvailableBiometrics();
```
The list may contain one or more from the following types:
- BiometricType.face
- BiometricType.fingerprint
- BiometricType.iris

## Authenticate using biometrics or pin

You can perform the local authentication (that is using biometrics or pin/passcode) 
by using the following:
```
  isAuthenticated = await localAuthentication.authenticate(
    localizedReasons: 'Please complete the biometrics to proceed.'
  );
```
If you want to restrict the user to biometric authentication (prevent 
authenticating using pin/passcode)
, you can set the biometricOnly parameter value to true:
```
  isAuthenticated = await localAuthentication.authenticate(
    localizedReason: 'Please complete the biometrics to proceed',
    biometricOnly: true
  );
```
In our implementation, we will need to check if biometric authentication is suported 
by the device, then let the user to only use biometrics to authenticate, and move to 
the next screen.

## Implement local authentication

We need to authenticate the user in the UserInfoScreen before proceeding to the 
SecretVaultScreen.

Define a new method in the Authentication class, present in the authentication.dart 
file, called authenticateWithBiometrics() where the entire logic of biometric 
authentication will be written.
```
  class Authentication {
    static Future<bool> authenticateWithBiometrics() async {
      final LocalAuthentication localAuthentication = LocalAuthentication();
      bool isBiometricSupported = await localAuthentication.isDeviceSupported();
      bool canCheckBiometrics = await localAuthentication.canCheckBiometrics;

      bool isAuthenticated = false;

      if (isBiometricSupported && canCheckBiometrics) {
        isAuthenticated = await localAuthentication.authenticate(
          localizedReason: 'Please complete the biometrics to proceed.',
	  biometricOnly: true
        );
      }

      return isAuthenticated;
    }
  }
```
The authenticateWithBiometrics() method will return a boolean indicating whether the 
biometric authentication is successful.

There are some optional parameters of this
localAuthentication.authenticate() method that we can define:

- localizedReason: the message to show the user while authentication
- useErrorDialogs: while set to true, it will check if fingerprint reader exists on 
the pone but there's no fingerprint registered, the plugin will attempt to take the 
user to settings to add one.
- stickyAuth: in normal circumstances the authentication process stops when the app 
goes to background. If stickyAuth is set to true, authentcation resumes when the app 
is resumed.

Now you can update the onPressed() method of the Access secret vault button to use 
the biometric authentication:
```
  ElevatedButton(
    onPressed: () async {
      bool isAuthenticated = await Authentication.authenticateWithBiometrics();
    }

    if (isAuthenticated) {
      Navigator.of(context).push(
        MaterialPageRoute(
	  builder: (context) => SecretVaultScreen(),
        ),
      );
    } else {
      ScaffoldMessenger.of(contexst).showSnackBar(
        Authentication.customSnackBar(
          content: `Error authenticating using biometrics`,
        ),
      );
    },
  },
),
```
If the authentication is successful then the user will navigated to the 
SecretVaultScreen, otherwise a SnackBar would be shown with an error message.

## Setup for using biometrics

# For Android:

Add this permission to the AndroidManifest.xml file present in the directory 
android -> app -> src -> main:
```
<uses-permission android:name="android.permission.USE_FINGERPRINT"/>
```
Update the MainActivity.kt file to use FlutterfragmentACtivity instead of 
FlutterActivity:
```
import io.flutter.embedding.android.FlutterFragmentActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugins.GeneratedPluginRegistrant

class MainActivity: FlutterFragmentActivity() {
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        GeneratedPluginRegistrant.registerWith(flutterEngine)
    }
}
```
# For iOS:

For using FaceID on iOS, add the following line to the Info.plist file, this defines 
the message to be displayed when a user is prompted to authenticate:

```
<key>NSFaceIDUsageDescription</key>
<string>Biometric authentication for accessing secrets</string>
```
