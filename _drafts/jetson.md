# nvidia jetson notes

## Boot from NVMe

sudo diskutil partitionDisk /dev/disk4 1 GPT "Free Space" "%noformat%" 100%

+ https://www.seeedstudio.com/blog/2020/06/22/boot-jetson-xavier-from-m-2-ssd/
+ https://www.jetsonhacks.com/2020/05/29/jetson-xavier-nx-run-from-ssd/
+ https://github.com/jetsonhacks/rootOnNVMe1

## Jetson performance and power mode

+ https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/power_management_jetson_xavier.html#wwpID0E0VO0HA

Show current power settings for the system:

```bash
sudo /usr/bin/jetson_clocks --show
```

Get the current power mode:

```bash
sudo /usr/sbin/nvpmodel -q
```

Set mode 32 - 15W 6 cores

```bash
sudo /usr/sbin/nvpmodel -m 2
```

## Jetson benchmarks

Useful guide to how models will run: https://developer.nvidia.com/embedded/jetson-benchmarks

## learning

[Getting Started with AI on Jetson Nano on nvidia deep learning institute](https://courses.nvidia.com/courses/course-v1:DLI+S-RX-02+V2/courseware/b2e02e999d9247eb8e33e893ca052206/9605d1c247064480820d5a6b52dcbfe5/?activate_block_id=block-v1%3ADLI%2BS-RX-02%2BV2%2Btype%40sequential%2Bblock%409605d1c247064480820d5a6b52dcbfe5)


sudo docker run --runtime nvidia -it --rm --network host --volume ~/nvdli-data:/nvdli-nano/data --device /dev/video0 nvcr.io/nvidia/dli/dli-nano-ai:v2.0.1-r32.5.0


### nvidia deep learning course containers

https://ngc.nvidia.com/catalog/containers/nvidia:dli:dli-nano-ai/tags