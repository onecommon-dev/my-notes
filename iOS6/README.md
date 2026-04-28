[Back to main page](../README.md)

# Everything I know on iOS 6

My devices: iPhone 3GS, iPhone 4S, iPhone 5
The best device is iPhone 5 in my opinion. It has the most amount of RAM and the best CPU, making it the best iOS 6 experience.

## Downgrading iPhone 5 to iOS 6

VERY IMPORTANT to know before we start. 
In order for an iOS device to downgrade to any iOS version at all, we *must* satisfy 1 of 2 conditions:
- The target iOS version is still signed by Apple for that device. This means Apple is still officially allowing that version to be installed on the device. Apple can withdraw this at any point.
- The current iOS version that the device is running has an amazing exploit that allows downgrading to lower iOS versions which are unsigned by Apple. 

As far as downgrading to iOS 6 is concerned, I am only aware of 
- iPhone 4, all iOS versions
- iPhone 4S, all iOS versions
- iPhone 5, iOS 7.x
To have such an amazing exploits.

### Tools

For almost any situation, the best tool is Legacy iOS Kit - LIK: https://github.com/LukeZGD/Legacy-iOS-Kit
So clone it down, put it where you'd normally put your data files since this script will want to download quite large files.

The LIK script works on macOS and Linux as far as I know. I personally use Linux.

*ProTip*: If you have multiple devices, for every device you use LIK with, backup the `saves` folder and remove the original once you are done, so you know that backup only contains files related to that device. Otherwise, that folder will contain files for all of your devices which will be difficult to separate later.

Another useful tool is https://ios.cfw.guide. This website will tell you know to jailbreak your device, in case you need to.

### From 10.3.4

If your iPhone 5 is on 10.3.4 (most iPhone 5's will), the best thing to do first is downgrading to 8.4.1. The reason is that the jailbreak on 10.3.4 is only semi-untethered, meaning you'll need to rerun the jailbreak on every boot, whereas the jailbreak on 8.4.1 is untethered. In addition, you'd need to continually refresh the entitlements every week for the 10.3.4 jailbreak app using Sideloadly, due to Apple's restrictions, which is very annoying.

So we should downgrade to 8.4.1. 

Note that the downgrade to 8.4.1 is only possible because Apple is still signing it.

To start, plug the device in and launch the LIK script with `./restore.sh`

Pick the Downgrade option and pick 8.4.1. The script will prompt to download the .ipsw firmware file for your iPhone 5, and will automatically pick the right one. 

Alternatively, you can download manually from https://ipsw.me. If you download manually, either place the file in the LIK script folder or anywhere you like. iPhone 5's have 2 versions, Global (iPhone5,2) and GSM (iPhone5,1). The LIK script can tell you which version it is by showing that it is either `5,2` or `5,1`. In my case, it is `5,2` so I need to download the global firmware.

*ProTip*: Definitely say `Yes` when asked whether you want to jailbreak. Saves a ton of hassle.

Next, the LIK script will need to put the device into either `kDFU` or `pwnedDFU` mode in order to do the firmware restore. `kDFU` and `pwnedDFU` are the same thing, the only difference is that `kDFU` is achieved by  jailbreaking the phone, installing OpenSSH and DropBear, then letting the LIK script handle the rest, while `pwnedDFU` is achieved by manually executing a sequence of power and home button presses. Hence `kDFU` is easier to achieve if the iOS version the phone is currently on can be jailbroken, and `pwnedDFU` is harder and requires working power and home buttons.

A sidenote, for iOS 10.3.4, as I said before, there is a semi-untethered jailbreak called Socket that can be installed via Sideloadly. Check out https://ios.cfw.guide for more info.

If you can't be bothered to jailbreak, you can proceed with `pwnedDFU`. The LIK script will first put the device into recovery mode, and you will see the iTunes restore logo on the iPhone screen. Upon proceeding, the script will guide you to do the button press combinations. Note that this has a high chance of failure unless you have lots of experience doing it again and again like I did.

*ProTip*: about 1-2 seconds after releasing the home button, quickly unplug and replug the device 2 times. The device will reconnect when the LIK script says `[DEBUG] reconnect`. If done incorrectly, the phone will not reconnect to the PC and you will have to restart the process. To restart the process, hold the power and home button for a few seconds.

Once done correctly, the LIK script will proceed with the firmware installation. Just don't touch it for a while and once the script finishes, the phone will boot into 8.4.1.

### From 8.4.1

Go to Cydia, add the source http://coolbooter.com, and install Coolbooter, Coolbooter CLI and MTerminal. Why Coolbooter CLI? Because with it you can usually get 1 more GB of space for the iOS 6 partition. 

For example, with a 16GB iPhone 5 and freshly restored iOS 8.4.1 via LIK, you will have about 11.6 GB of free space. If you use the GUI version of Coolbooter, it will allow you to partition a maximum of 7GB for iOS 6. But if you use the CLI version, you can put 8GB instead (more and the installation will fail).

So open MTerminal, type `su` and then `alpine` for the password, then
```
coolbootercli 6.1.2 --datasize 8GB
```
After the script finishes, it will automatically reboot the device. Go to Coolbooter GUI, press Boot, lock the device, and wait. It will reboot once more. Again, go to Coolbooter GUI, press Boot and lock the device. This time, it will boot into the iOS 6 first time setup screen.

*ProTip*: Whenever you do anything with Coolbooter, you need to unplug the lightning cable if plugged in, otherwise the script will fail to boot for some reason.

### From 7.x

#### Save SHSH blobs

Use https://ios.cfw.guide to jailbreak the device first. iOS 7.1.x has an extremely easy way to jailbreak via Safari. However, I don't know how long that jailbreaking website will last, so do this ASAP. Install OpenSSH once jailbroken.

Connect to the PC and run the LIK script. Pick ***Save SHSH blobs***. It will need to get the phone to `kDFU` mode in order to do that.

*But what are SHSH blobs?*

Remember when I said that Apple still signs iOS 8.4.1 installation for iPhone 5's? That signature is stored on the phone as `SHSH blobs`. An iPhone 5 running iOS 7 will have the `SHSH blobs` from when Apple was still signing it. So by saving the iOS 7 `SHSH blobs`, it allows the LIK script to restore that version onto that phone at any time in the future.

Since Apple has long stopped signing iOS 7 for iPhone 5's, iPhone 5's running iOS 7 and below are very rare. Also, there is an exploit that lets you downgrade to any version of iOS 6. Hence, you just have to save the `SHSH blobs`.

Note that the saved blobs are only valid for that specific iPhone 5. It will be under the `saved` folder inside the script folder.

#### iOS 7.0.x 

iOS 7 prior to 7.1 cannot be jailbroken with the same easy method as 7.1.x, but instead has to use a different tool that runs on Windows. I've tried many times but I couldn't make that tool work. So it's best to use `pwnedDFU` if we're on iOS 7.0.x, for both saving the SHSH blobs and downgrading.

#### iOS 6 downgrade

Since we're on iOS 7, there is an amazing exploit that allows us to use the iOS 7 SHSH blobs to downgrade to any iOS 6 version. Simply use the LIK script, select Downgrade, Downgrade using 7.x powdersn0w, select the right iOS 6.1.2 firmware file for your iPhone 5 as the target version, select the right iOS 7 firmware file as the base version, select the saved SHSH blobs, then proceed to either `kDFU` or `pwnedDFU`.

As you may have already deduced, this downgrade method tricks the device into thinking that it is booting iOS 7, while instead booting into iOS 6. Because of that, when the device is booting, for the second at the beginning, the iOS 7 boot screen can be seen, before quickly being replaced by the iOS 6 boot screen.

### Why iOS 6.1.2 but not 6.1.3 or 6.1.4 / enabling LTE

Because of this https://www.reddit.com/r/LegacyJailbreak/comments/1dl6gvx/guide_how_to_enable_lte_4g_on_any_iphone_5_on_ios/
According to this post, iOS 6.1.3 and 6.1.4 didn't add much to 6.1.2, but it patched an exploit which previously allowed us to easily enable LTE.

To enable LTE:
- Install the Commcenter patch ! Without it, all modifications done below will not be applied.
- With iFile, go to /private/var/mobile/Library/Carrier Bundle.bundle/ (which is a shortcut to your current Carrier Bundle)
- Open the file carrier.plist with the Properties reader
- Click on the « + » button and add a boolean key called « Show4GSwitch » then validate. Then, enable this boolean key !
- Erase the folder called "overlay" that is located in /var/mobile/Library/Carrier Bundles/
- Respring. Now you need to check if the 3G switch has been correctly replaced by the LTE switch.

This really enables LTE in all the iPhone 5's I have tested under iOS 6.1.2. Keep in mind since I'm based in the UK, my iPhone 5's are all A1269 iPhone5,2 global versions with more LTE bands than other models.

## Save activation tickets

On a jailbroken iPhone 5 with OpenSSH installed, you can save activation tickets.

*What are activation tickets?*

When you setup an iPhone for the first time after an iOS restore, your iPhone needs to be activated. This is basically asking for Apple's permission for the phone to be used. If Apple says yes, this information is stored on the phone as "activation tickets". 

By saving the activation tickets, it allows the LIK script to inject the ticket the next time we use it to restore any iOS version on this phone, hence bypass the need to ask for permission from Apple.

Note that the saved activation tickets are only valid for that specific iPhone 5. It will be under the `saved` folder inside the script folder.

## Save basebands

This is another thing that the LIK script allows us to do, although I'm not sure what the utility of this is. Again it is specific to that iPhone 5 and will be under the `saved` folder inside the script folder.

## Cydia Tweaks

Now comes the easy & fun part, installing tweaks. This will be personal, so I'll only dedicate this section for my own use.

Here are the repos that I use:

```
deb http://cydia.aoiblog.jp/ ./
deb http://yzu.moe/dev/ ./
deb http://cydia.invoxiplaygames.uk/beta/ ./
deb http://cydia.bag-xml.com/ ./
deb http://cydia.invoxiplaygames.uk/ ./
deb http://cydia.skyglow.es/ ./
deb http://cydia.preloading.dev/ ./
deb http://beta.leimobile.com/repo/ ./
deb http://cydia.nekokawa.net/ ./
deb http://repo.legacyios.com/ ./
deb http://cydia.invoxiplaygames.uk/cydate/ ./
```
To quickly add these sources to Cydia, put the above text in a file named AnythingYouWant.list in /etc/apt/sources.list.d on the device. Use iFile or SSH to achieve this. It can be tricky because you need first to install iFile or OpenSSH, but you can use OpenBackup to backup all the .deb files to install them later. Also, if you use the LIK script to downgrade to iOS 6, OpenSSH will be installed out of the box.

The most important tweaks:
- Activator (for double tapping status bar to lock)
- iFile (for easy file management)
- OpenBackup (for backing up tweaks)
- Veteris (for an alternative appstore)
- Zephyr (for gestures)
- CommCenter Patch (for enabling LTE)
- MTerminal (for anything that can't be done with iFile)
- SSLPatch (for fixing an old vulnerability)

Apart from this, I also have:
- GuizmOVPN (OpenVPN client that works)
- StocksX and WeatherX
- MapsX
- Maps5 (for stock iOS 5 maps app that use Google maps)
- AppStoreFix (for fixing the built-in AppStore)
- BatteryLife (for checking real battery capacity)
- Classic Youtube App
- TubeRepair BETA (pointing Youtube apps to an Invidious instance of choice. Supports both the classic youtube app and Youtube 2.2.0)
- EbayX2 (for fixing ebay 1.7.2)
- NCSettings and 4 NCSettings

## Prevent NAND wear due to logging

GuizmOVPN logs to disk every time it runs, which is unnecessary for me and can wear down the non-replaceable NAND. Also, iOS logs to disk as well. So I've looked into disabling as much as I can, using commands in MTerminal. Here's the script for that:

```
rm /var/log/DiagnosticMessages/*
ln -s /dev/null /var/log/DiagnosticMessages/StoreData
chmod 444 /var/log/DiagnosticMessages

rm /var/log/ppp.log
ln -s /dev/null /var/log/ppp.log

rm /var/log/asl/SweepStore
ln -s /dev/null /var/log/asl/SweepStore

rm /var/logs/keybagd.log
ln -s /dev/null /var/logs/keybagd.log

rm /var/logs/lockdownd.log
ln -s /dev/null /var/logs/lockdownd.log

rm /var/logs/AppleSupport/general.log
ln -s /dev/null /var/logs/AppleSupport/general.log

# if GuizmOVPN is installed
mkdir -p /var/mobile/Documents/log
rm -f -- /var/mobile/Documents/log/openvpn.log
ln -s /dev/null /var/mobile/Documents/log/openvpn.log
```

This script can either be downloaded to the device to be executed in MTerminal, or via SSH.

## Whatsapp

WhatsAppX (https://github.com/calvinknt/WhatsAppX) is an open source solution for running WhatsApp on iOS 6. 

This solution has an iOS app, which connects to a server which has to be run on your computer. This server uses nodejs and uses `whatsapp-web.js` (https://wwebjs.dev/) as a WhatsApp client. This means the actual WhatsApp client runs on your computer, and the iOS app acts as a thin client.

Notifications doesn't work since that requires the server to send requests to Apple Push servers, which costs $99 per year.

### Security concern

There are obviously huge security concerns. The traffic between this thin iOS client with the "server" is all in plain text. This means if you expose the server to the internet, anyone could connect to it and send/receive WhatsApp messages as you.

Because of this, if you want to use this outside of your home wifi, self hosting a VPN is heavily recommended. In my case, I setup an OpenVPN server and use GuizmOVPN to connect to it.

### Possibility

If I have the time to read & understand the json schemas that the thin client expects, I can in theory rewrite the server to connect to anything, be it Signal, Delta Chat, or even SimpleX Chat

## IRC

Amazingly, the IRC app called LimeChat (which you can get on Veteris) still works. However SSL encryption no longer works, so you can only connect via plain text.

Due to this, I only recommend using this with your self-hosted IRC server and with your selfhosted VPN.

Here is a very easy to use docker for an IRC server: https://github.com/inspircd/docker

## VPN

I self-host SoftEther VPN (https://www.softether.org/) on the same machine that I host my IRC server and the WhatsAppX server. That way, I can connect my iPhone to those servers using local IP addresses, without having to expose them to the internet. The VPN server however needs to be exposed to the internet.

I use this docker image for ease of setting up: https://hub.docker.com/r/siomiz/softethervpn/

Here is the command I use:
```
docker run \
--name softethervpn \
--restart=always \
-d --cap-add NET_ADMIN \
-e PSK=Shared secret \
-e USERS="user1:pass1;user2:pass2;user3:pass3" \
-e SPW=password% \
-e HPW=password% \
-d --privileged \
-p 500:500/udp \
-p 4500:4500/udp \
-p 1701:1701/tcp \
-p 1194:1194/udp \
-p 5555:5555/tcp \
-v ./vpn_server.config:/usr/vpnserver/vpn_server.config \
-v /Volumes/RamDisk/softether/server_log:/usr/vpnserver/server_log \
-v /Volumes/RamDisk/softether/packet_log:/usr/vpnserver/packet_log \
-v /Volumes/RamDisk/softether/security_log:/usr/vpnserver/security_log \
siomiz/softethervpn
```

Where `/Volumes/Ramdisk` is a ram disk that I mount on boot. This means the server won't log to disk to reduce disk lifespan, and I can still read logs as long as the machine doesn't reboot.

SoftEther supports 2 protocols that works on iOS 6, L2TP/IPSec and OpenVPN. For those 2, you need to open ports 500, 4500 and 1194. In general OpenVPN is recommended due to it being more secure and faster, especially when the client GuizmOVPN works. Unfortunately no WireGuard support whatsoever.

For exporting the OpenVPN profile, copy the terminal output of the above docker run command, put it in a file, and name it with the .ovpn extension.

## Notifications

There seems to be an open source implementation of push notifications, aimed at legacy iOS. However, the documentation is a bit scarce so it's not clear how exactly to use. I'll put the links here if this changes in the future.

Client: https://github.com/ObscureMosquito/Skyglow-Notifications-Client

Server: https://github.com/Preloading/SkyglowNotificationServer

## HTTPS Rest calls

There is a modern ChatGPT client that works on iOS 6: https://github.com/bag-xml/ChatGPT-for-Legacy-iOS
The significance of this is not ChatGPT itself, but the fact that we have code example of securely communicating with a REST backend over HTTPS.

If I have the time to dissect this code, it would be very helpful when trying to create modern & secure iOS 6 client-server apps.

## SSH

If you've installed OpenSSH on the phone, you can use wired or wireless SSH to connect to the phone on a computer with SSH.
But to actually do that, you need to specify the command with a special parameter like so:
```
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa root@192.168.1.86
```
Otherwise, there's a chance the ssh command will fail with this error
```
Unable to negotiate with 192.168.1.86 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss
```
For the SSH password, just use `alpine` as it is the default password for OpenSSH on iPhones.
