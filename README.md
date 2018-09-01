# Disclaimer

This is highly experimental, and heavily based on various other repos, such as
[MaximeD/docker-react-native](https://github.com/MaximeD/docker-react-native).
Use at your own risk!

# Usage

## Get and install the image

Clone the repository and build the image:
```
git clone https://github.com/mayhewluke/react-native-docker
cd react-native-docker

docker build -t react-native .
```

## Set up the helper script

This Docker image is most easily used by utilizing the `rn` script available in
`bin/`. `rn` will take any command you give it and run it within a React Native
docker container. Alternatively, you can use `rn bash` to drop into a shell for
interactive use.

Add the script to your path:
```
export PATH="$HOME/react-native-docker/bin:$PATH"
```

Single command example:
```
rn emulator -avd nexus5
```

Interactive shell example:
```
rn bash
$ emulator -avd nexus5 &
...
$ react-native run-android
...
```

## Create a new react native project

Use the React Native CLI to initialize the project:
```
rn react-native init MyAwesomeProjet
cd MyAwesomeProjet
```

## Run project

Connect a physical or virtual android device (see [Caveats](#caveats) below),
then:

```
rn react-native run-android
```

### Hot reload

```
rn bash
$ watchman watch .
$ rn react-native run-android
```

Make sure to enable it in the debug menu on the device (see
[Caveats](#enabling-debug-features)).

# Caveats

## Giving Docker access to the display

Docker won't be able to run any GUI applications unless you give it permission
to use the host's display:

```
xhost local:docker
```

Or, on NixOS:

```
nix-shell -p xorg.xhost --run 'xhost local:docker'
```

This will need to be run each time the host is rebooted (or possibly at each
login, if on a multi-user machine?).

## Using a physical device

You'll need to install the android udev rules if you want to test the app on a
physical device (connected via USB). For NixOS users this is as simple as adding
`programs.adb.enable = true;` to your NixOS configuration. For non-NixOS users,
the following should work (warning: I have not tested this personally):

```
wget -S -O - http://source.android.com/source/51-android.rules | sed "s/<username>/$USER/" | sudo tee >/dev/null /etc/udev/rules.d/51-android.rules
sudo udevadm control --reload-rules
```

## Using an emulator

Available documentation (official or otherwise) on creating emulators on the
command line is sparse and not particularly clear, but the following should be
enough to create an Android Virtual Device for basic use.

First, get the list of built-in device definitions. In our case, we'll be trying
to create a Nexus 5 emulator, just to demonstrate.

```
rn avdmanager list device
# ...
#     OEM : Google
# ---------
# id: 8 or "Nexus 5"
#     Name: Nexus 5
#     OEM : Google
# ---------
# id: 9 or "Nexus 5X"
# ...
```

So we want the device ID of `8`, or we can use `Nexus 5`. Now to create the AVD:

```
rn avdmanager create avd --name nameOfAvd --package 'system-images;android-23;default;x86_64' --device "Nexus 5"
```

If asked if you want to create a custom hardware profile, say no (creating a
custom hardware profile asks a ton of detailed, technical questions that are
likely beyond the scope of most use cases). However, there is one hardware
profile tweak we will want to make - enabling keyboard support, so that you can
type into the emulator using your keyboard instead of the on-screen keyboard.

Once the AVD has been created, add keyboard support to its config.ini:

```
echo "hw.keyboard=yes" >> ~/.android/avd/nameOfAvd.avd/config.ini
```

You're then ready to launch the emulator via:

```
rn emulator -avd nameOfAvd
```

## Enabling debug features

React Native includes many useful debug features, such as live and hot code
reloading, and remote JS debugging via browser developer tools. In order to
enable/disable them you will need to access the debug menu. On physical devices
this can be done by shaking the device, and on an emulator it can be accessed
via `CTRL-M`.

### Remote JS debugging

Once enabled on the device, navigate to `localhost:8081/debugger-ui`. Requires
`run-android` to be running.

# Customizing the image

The Dockerfile accepts a number of build-time arguments that can be used to
customize how it functions, such as the SDK tools version, or the UID of the
user it will run under. Please examine the various `ARG` directives in the
Dockerfile for more information.
