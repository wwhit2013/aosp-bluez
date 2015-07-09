AOSP with BlueZ 5 integrated as replacement for default Bluedroid Bluetooth stack.

This project is an example on how BlueZ 5 for Android can be integrated with AOSP project.

# Short summary #

Supported Android version: **5.0** (`'lollipop'` branch), **4.4.4** (`'kitkat'` branch)

Supported devices:
  * **Nexus 4** (`mako` target)
  * **Nexus 7 2013** (`flo` target)
  * **Nexus 7 2013 LTE** (`deb` target)
  * **Nexus 5** (`hammerhead` target)

# Getting Started #

Note: Provided instructions are for Android 5.0 Lollipop.

For Android 4.4 `KitKat` use `'kitkat'` branch for repo and `'android-msm-<target>-3.4-kitkat-mr2'` for kernel.

**Downloading source:**
```
repo init -u https://code.google.com/p/aosp-bluez.platform-manifest -b lollipop
repo sync
```

**Downloading binary drivers:**

Additional binary drivers are required to boot AOSP on Nexus devices. Those require accepting EULA and can be downloaded from https://developers.google.com/android/nexus/drivers. Follow instructions for your supported device and Android version of choice and after unpacking place them in vendor folder of downloaded repo.

**Initializing build environment:**
<br><i>note: replace <code>&lt;target&gt;</code> with appropriate value depending on your device, see above</i>
<pre><code>source build/envsetup.sh<br>
lunch aosp_&lt;target&gt;-userdebug<br>
</code></pre>

<b>Building:</b>
<pre><code>make -j8<br>
</code></pre>

<b>Flashing:</b>
<br><i>note: you can skip <code>-w</code> to keep userdata and cache partitions content, i.e. in case you want just to upgrade software without erasing any data</i>
<pre><code>adb reboot bootloader<br>
fastboot flashall -w<br>
<br>
</code></pre>

<h1>Building own kernel</h1>

<b>aosp-bluez</b> provides prebuilt kernels for all supported devices. When building own kernel from MSM tree, following configuration is recommended in order for all features to work properly:<br>
<br>
<ul><li>enabled <code>Enable loadable modules support</code> (<code>CONFIG_MODULES</code>)<br>
</li><li>enabled <code>Bluetooth subsystem support</code> (<code>CONFIG_BT</code>) as module<br>
</li><li>enabled <code>802.1d Ethernet Bridging</code> (<code>CONFIG_BRIDGE</code>)<br>
</li><li>backported <code>hid-generic</code> driver<br>
</li><li>backported <code>aes-cmac</code> crypto hash<br>
</li><li>Bluetooth subsystem built from backports (with RFCOMM and BNEP protocols support) - recommended latest development release available from <a href='https://www.kernel.org/pub/linux/kernel/projects/backports/'>https://www.kernel.org/pub/linux/kernel/projects/backports/</a>
</li><li>HCI SMD driver built with backports</li></ul>

To make things easier, we provide required modifications as patches available in <a href='http://code.google.com/p/aosp-bluez/source/browse/?repo=misc'>misc</a> repository and step-by-step instruction below to have own kernel binaries built.<br>
<br>
<pre><code>### download misc repository<br>
git clone https://code.google.com/p/aosp-bluez.misc/ misc<br>
<br>
### download kernel<br>
git clone https://android.googlesource.com/kernel/msm kernel-msm<br>
cd kernel-msm<br>
<br>
### prepare kernel sources<br>
git checkout android-msm-&lt;target&gt;-3.4-lollipop-release<br>
git am ../misc/patches-kernel/0001-hid-Backport-hid-generic-driver.patch<br>
git am ../misc/patches-kernel/0002-crypto-add-CMAC-support-to-CryptoAPI.patch<br>
git am ../misc/patches-kernel/0003-crypto-af_alg-properly-label-AF_ALG-socket.patch<br>
<br>
### configure kernel<br>
make ARCH=arm CROSS_COMPILE=arm-eabi- &lt;target&gt;_defconfig<br>
scripts/config --enable CONFIG_MODULES<br>
scripts/config --module CONFIG_BT<br>
scripts/config --enable CONFIG_BRIDGE<br>
scripts/config --enable CONFIG_CRYPTO_CMAC<br>
scripts/config --enable CONFIG_CRYPTO_USER_API<br>
scripts/config --enable CONFIG_CRYPTO_USER_API_HASH<br>
scripts/config --enable CONFIG_CRYPTO_USER_API_SKCIPHER<br>
make ARCH=arm CROSS_COMPILE=arm-eabi- oldnoconfig<br>
<br>
### build kernel<br>
make ARCH=arm CROSS_COMPILE=arm-eabi- -j8<br>
<br>
cd ..<br>
<br>
### unpack backports<br>
tar xf backports-&lt;yyyymmdd&gt;.tar.xz<br>
cd backports-&lt;yyyymmdd&gt;<br>
<br>
### prepare backports sources<br>
patch -p1 &lt; ../misc/patches-backports/0001-Add-defconfig-bluetooth.patch<br>
patch -p1 &lt; ../misc/patches-backports/0002-Enable-6LOWPAN.patch (optional)<br>
patch -p1 &lt; ../misc/patches-backports/0003-Add-HCI-SMD.patch  (needed for Nexus 4,7)<br>
patch -p1 &lt; ../misc/patches-backports/0004-Add-HCI-H4.patch  (needed for Nexus 5)<br>
patch -p1 &lt; ../misc/patches-backports/0005-Workaround-for-msm-kernel-tree.patch<br>
<br>
<br>
### configure backprots<br>
make ARCH=arm CROSS_COMPILE=arm-eabi- KLIB_BUILD=../kernel-msm defconfig-bluetooth<br>
<br>
### build backports<br>
make ARCH=arm CROSS_COMPILE=arm-eabi- KLIB_BUILD=../kernel-msm -j8<br>
</code></pre>

Finally, existing <code>kernel</code> should be replaced with <code>kernel-msm/arch/arm/boot/zImage</code> and <code>modules/*.ko</code> with corresponding files from <code>backports-&lt;yyyymmdd&gt;/</code>.<br>
<br>
<h1>Other useful links:</h1>

Google documentation on how to download and build AOSP:<br>
<a href='http://source.android.com/source/building.html'>http://source.android.com/source/building.html</a>

BlueZ documentation for BlueZ for Android:<br>
<a href='http://git.kernel.org/cgit/bluetooth/bluez.git/tree/android/README'>http://git.kernel.org/cgit/bluetooth/bluez.git/tree/android/README</a>