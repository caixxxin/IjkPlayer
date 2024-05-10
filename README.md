# IjkPlayer

```` txt


###########################
# 安装基础环境  基于ubuntu22.04 LTS (可以替换国内apt源，未自动替换)
###########################
# sed -i 's@//.*archive.ubuntu.com@//mirrors.ustc.edu.cn@g' /etc/apt/sources.list
# sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
cd /root
apt update
apt install -y git yasm curl wget gcc make cmake unzip build-essential
add-apt-repository -y ppa:openjdk-r/ppa
apt install -y openjdk-8-jdk

###########################
# 下载Android-Studio 并安装sdk25和build tool 25.0.3 (可以单独安装commandlinetools)
###########################
#cd /root
#wget https://redirector.gvt1.com/edgedl/android/studio/ide-zips/2021.2.1.3/android-studio-2021.2.1.3-linux.tar.gz
#tar zxvf android-studio-2021.2.1.3-linux.tar.gz
#cd /root/android-studio

###########################
# 下载安装sdk25和build tool 25.0.3
###########################
cd /root
wget https://dl.google.com/android/repository/commandlinetools-linux-9123335_latest.zip
unzip commandlinetools-linux-9123335_latest.zip
mkdir -p /root/Android/Sdk/cmdline-tools
mv /root/cmdline-tools /root/Android/Sdk/cmdline-tools/latest
cd /root/Android/Sdk/cmdline-tools/latest/bin
yes | ./sdkmanager --licenses
./sdkmanager --install platforms\;android-25
./sdkmanager --install build-tools\;25.0.3
./sdkmanager --install sources\;android-25
./sdkmanager --install platform-tools


###########################
# 下载并安装ndk
###########################
cd /root
wget https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip
unzip android-ndk-r10e-linux-x86_64.zip
mkdir -p /root/Android/ndk
mv android-ndk-r10e /root/Android/ndk


##################
# 下载代码
##################
#git config --global http.proxy http://192.168.11.113:7897
#git config --global https.proxy http://192.168.11.113:7897

cd /root
git clone https://github.com/Bilibili/ijkplayer.git

###########################
# 加载环境变量
###########################
export ANDROID_SDK=/root/Android/Sdk
export PATH=$ANDROID_SDK/platform-tools:$PATH
export PATH=$ANDROID_SDK/tools:$PATH
export ANDROID_NDK=/root/Android/ndk/android-ndk-r10e
export PATH=$ANDROID_NDK:$PATH
export ANDROID_HOME=/root/Android/Sdk


###########################
# 初始化项目
###########################
cd /root/ijkplayer
./init-android.sh

cd /root/ijkplayer/config
rm module.sh
cp module-default.sh module.sh
echo "" >> module.sh
echo "" >> module.sh
echo "export COMMON_FF_CFG_FLAGS=\"\$COMMON_FF_CFG_FLAGS --disable-linux-perf\"" >> module.sh
sed -i "/--disable-ffserver/d" module.sh
sed -i "/--disable-vda/d" module.sh

cd /root/ijkplayer
./init-android-openssl.sh

echo "export IJK_MAKE_FLAG=-j8" >> /root/ijkplayer/android/contrib/tools/do-detect-env.sh


###########################
# clean项目
###########################
cd /root/ijkplayer/android/contrib
./compile-openssl.sh clean
./compile-ffmpeg.sh clean


###########################
# build项目
###########################
cd /root/ijkplayer/android/contrib
./compile-openssl.sh all
./compile-ffmpeg.sh all

cd /root/ijkplayer/android
./compile-ijk.sh all


###########################
# 打包项目
###########################
cd /root/ijkplayer/android/ijkplayer

# 配置代理，如果网络ok，可以不设置。
sed -i "/systemProp.http.proxyHost/d" gradle.properties
sed -i "/systemProp.http.proxyPort/d" gradle.properties
sed -i "/systemProp.https.proxyHost/d" gradle.properties
sed -i "/systemProp.https.proxyPort/d" gradle.properties
echo "" >> gradle.properties
echo "systemProp.http.proxyHost=192.168.11.113" >> gradle.properties
echo "systemProp.http.proxyPort=7897" >> gradle.properties
echo "systemProp.https.proxyHost=192.168.11.113" >> gradle.properties
echo "systemProp.https.proxyPort=7897" >> gradle.properties

# 去掉无用的工程
sed -i "/ijkplayer-example/d" settings.gradle
sed -i "/ijkplayer-exo/d" settings.gradle

# 打包
./gradlew clean
./gradlew assembleRelease


###########################
# 导出aar
###########################
mkdir -p /root/ijkplayer/android/output
cd /root/ijkplayer/android/output
rm -rf /root/ijkplayer/android/output/*
cp /root/ijkplayer/android/ijkplayer/ijkplayer-java/build/outputs/aar/ijkplayer-java-release.aar       /root/ijkplayer/android/output
cp /root/ijkplayer/android/ijkplayer/ijkplayer-armv7a/build/outputs/aar/ijkplayer-armv7a-release.aar   /root/ijkplayer/android/output
cp /root/ijkplayer/android/ijkplayer/ijkplayer-arm64/build/outputs/aar/ijkplayer-arm64-release.aar     /root/ijkplayer/android/output
cp /root/ijkplayer/android/ijkplayer/ijkplayer-x86/build/outputs/aar/ijkplayer-x86-release.aar         /root/ijkplayer/android/output
cp /root/ijkplayer/android/ijkplayer/ijkplayer-x86_64/build/outputs/aar/ijkplayer-x86_64-release.aar   /root/ijkplayer/android/output

cd /root/ijkplayer







````