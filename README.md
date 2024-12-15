# esp-idf_wsl_bypass
ieee80211_raw_frame_sanity_check bypass version from Bruce firmware modified for esp-idf

## Wi-Fi Stack Libraries (WSL) Bypasser (from [esp32-wifi-penetration-tool](https://github.com/risinek/esp32-wifi-penetration-tool))

It's based on [ESP32-Deauther](https://github.com/GANESH-ICMC/esp32-deauther) project, where the function used to check type of frame in frame buffer [was decompiled by Ghidra tool](https://github.com/GANESH-ICMC/esp32-deauther/issues/9) and it's name `ieee80211_raw_frame_sanity_check` was found.

The princip of this bypass is to use linker flag during compilation that allows multiple function definitions - `-Wl,-zmuldefs`. This allows this component to override default function behaviour and always return value that allows further transmittion of frame buffer by Wi-Fi Stack Libraries.

This is done in [CMakeLists.txt](CMakeLists.txt) by following line:
```cmake
target_link_libraries(${COMPONENT_LIB} -Wl,-zmuldefs)
```

And the function itself is defined in [wsl_bypasser.c](wsl_bypasser.c) as:
```c
int ieee80211_raw_frame_sanity_check(int32_t arg, int32_t arg2, int32_t arg3){
    return 0;
}
```

## esp-idf patch

The original method involves patching the `ieee80211_raw_frame_sanity_check` function in the compiled .elf file of your project.

After looking around, I found out that the [Bruce firmware](https://github.com/pr3y/Bruce/) uses a method that I find quite effective. I'm not sure if this approach was developed by the Bruce team themselves, but it certainly deserves recognition.

The approach involves directly patching the ESP-IDF `libnet80211.a`. I made a few adjustments to the .sh script so that it works with ESP-IDF instead of PlatformIO.
