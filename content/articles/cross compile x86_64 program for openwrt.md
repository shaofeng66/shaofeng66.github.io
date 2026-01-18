+++
title = 'Cross Compile X86_64 Program for Openwrt'
date = 2021-04-03T22:06:35+08:00
draft = false
tags = ["openwrt"]
+++


```
export STAGING_DIR=/home/openwrt/staging_dir/toolchain-x86_64_gcc-7.5.0_musl   
export PATH=${PATH}:${STAGING_DIR}/bin

./configure --build=x86_64-unknown-linux-gnu --host=x86_64-openwrt-linux-musl

make -j8
```
