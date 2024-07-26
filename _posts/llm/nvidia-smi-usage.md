---
title: nvidia-smi 命令使用
tags:
  - AI
category:
  - AI
author: bsyonline
lede: 没有摘要
date: 2018-05-28 15:32:21
thumbnail:
---

Proof of 

nvidia-smi 工具用来查看 GPU 的占用情况。nvidia-smi 工具支持 2011 年后发布的 GPU 。包括：

Tesla：S1070、S2050、C1060、C2050/70、M2050/70/90、X2070/90、K10、K20、K20X、K40、K80、M40、P40、P100、V100、A100、H100。

 Quadro：4000、5000、6000、7000、M2070-Q、K 系列、M 系列、P 系列

RTX 系列



#### nvidia-smi

查看 GPU 的占用信息

```
$ nvidia-smi
Fri Nov  3 10:54:52 2023       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.129.06   Driver Version: 470.129.06   CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla V100-PCIE...  On   | 00000000:1B:00.0 Off |                    0 |
| N/A   30C    P0    25W / 250W |      0MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  Tesla V100-PCIE...  On   | 00000000:1C:00.0 Off |                    0 |
| N/A   29C    P0    25W / 250W |      0MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   2  Tesla V100-PCIE...  On   | 00000000:1D:00.0 Off |                    0 |
| N/A   31C    P0    35W / 250W |  27068MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   3  Tesla V100-PCIE...  On   | 00000000:1E:00.0 Off |                    0 |
| N/A   29C    P0    23W / 250W |      0MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   4  Tesla V100-PCIE...  On   | 00000000:3D:00.0 Off |                    0 |
| N/A   29C    P0    25W / 250W |      0MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   5  Tesla V100-PCIE...  On   | 00000000:3F:00.0 Off |                    0 |
| N/A   32C    P0    35W / 250W |  32323MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   6  Tesla V100-PCIE...  On   | 00000000:40:00.0 Off |                    0 |
| N/A   31C    P0    35W / 250W |   2165MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   7  Tesla V100-PCIE...  On   | 00000000:41:00.0 Off |                    0 |
| N/A   30C    P0    24W / 250W |      0MiB / 32510MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    2   N/A  N/A     88420      C   python                          27064MiB |
|    5   N/A  N/A    230331      C   python                          32319MiB |
|    6   N/A  N/A    618979      C   python                           2161MiB |
+-----------------------------------------------------------------------------+
```



nvidia-smi -l [second]

每隔多少秒打印 GPU 的占用情况，用来监控。

#### nvidia-smi -L

列出所有的GPU设备及其UUID 。

```
$ nvidia-smi -L
GPU 0: Tesla V100-PCIE-32GB (UUID: GPU-9bcc15d0-42a0-3ede-9a62-1db85f961c55)
GPU 1: Tesla V100-PCIE-32GB (UUID: GPU-06e3d8e6-3f54-374a-8142-9752ca1f2316)
GPU 2: Tesla V100-PCIE-32GB (UUID: GPU-cd99427d-323c-3d5c-912f-f906c5ae0609)
GPU 3: Tesla V100-PCIE-32GB (UUID: GPU-39866e21-2ef1-0657-e7f2-379ba4b99927)
GPU 4: Tesla V100-PCIE-32GB (UUID: GPU-880ca219-3dfa-fb3d-b4d8-024da580042b)
GPU 5: Tesla V100-PCIE-32GB (UUID: GPU-e634d288-f34e-b0b7-45cc-30eecafee403)
GPU 6: Tesla V100-PCIE-32GB (UUID: GPU-8ccb6f2f-385f-3b68-952d-bcbd75cc297d)
GPU 7: Tesla V100-PCIE-32GB (UUID: GPU-1d021e83-8be1-1faf-4040-b46230a1328f)
```

#### nvidia-smi -q

列出所有GPU设备的详细信息。-i 指定具体一个 GPU 。

```
$ nvidia-smi -q -i 1

==============NVSMI LOG==============

Timestamp                                 : Fri Nov  3 11:39:46 2023
Driver Version                            : 470.129.06
CUDA Version                              : 11.4

Attached GPUs                             : 8
GPU 00000000:1C:00.0
    Product Name                          : Tesla V100-PCIE-32GB
    Product Brand                         : Tesla
    Display Mode                          : Enabled
    Display Active                        : Disabled
    Persistence Mode                      : Enabled
    MIG Mode
        Current                           : N/A
        Pending                           : N/A
    Accounting Mode                       : Disabled
    Accounting Mode Buffer Size           : 4000
    Driver Model
        Current                           : N/A
        Pending                           : N/A
    Serial Number                         : 1560221003078
    GPU UUID                              : GPU-06e3d8e6-3f54-374a-8142-9752ca1f2316
    Minor Number                          : 1
    VBIOS Version                         : 88.00.7E.00.03
    MultiGPU Board                        : No
    Board ID                              : 0x1c00
    GPU Part Number                       : 900-2G500-0110-030
    Module ID                             : 0
    Inforom Version
        Image Version                     : G500.0202.00.02
        OEM Object                        : 1.1
        ECC Object                        : 5.0
        Power Management Object           : N/A
    GPU Operation Mode
        Current                           : N/A
        Pending                           : N/A
    GSP Firmware Version                  : N/A
    GPU Virtualization Mode
        Virtualization Mode               : None
        Host VGPU Mode                    : N/A
    IBMNPU
        Relaxed Ordering Mode             : N/A
    PCI
        Bus                               : 0x1C
        Device                            : 0x00
        Domain                            : 0x0000
        Device Id                         : 0x1DB610DE
        Bus Id                            : 00000000:1C:00.0
        Sub System Id                     : 0x124A10DE
        GPU Link Info
            PCIe Generation
                Max                       : 3
                Current                   : 3
            Link Width
                Max                       : 16x
                Current                   : 16x
        Bridge Chip
            Type                          : N/A
            Firmware                      : N/A
        Replays Since Reset               : 0
        Replay Number Rollovers           : 0
        Tx Throughput                     : 0 KB/s
        Rx Throughput                     : 0 KB/s
    Fan Speed                             : N/A
    Performance State                     : P0
    Clocks Throttle Reasons
        Idle                              : Active
        Applications Clocks Setting       : Not Active
        SW Power Cap                      : Not Active
        HW Slowdown                       : Not Active
            HW Thermal Slowdown           : Not Active
            HW Power Brake Slowdown       : Not Active
        Sync Boost                        : Not Active
        SW Thermal Slowdown               : Not Active
        Display Clock Setting             : Not Active
    FB Memory Usage
        Total                             : 32510 MiB
        Used                              : 0 MiB
        Free                              : 32510 MiB
    BAR1 Memory Usage
        Total                             : 32768 MiB
        Used                              : 2 MiB
        Free                              : 32766 MiB
    Compute Mode                          : Default
    Utilization
        Gpu                               : 0 %
        Memory                            : 0 %
        Encoder                           : 0 %
        Decoder                           : 0 %
    Encoder Stats
        Active Sessions                   : 0
        Average FPS                       : 0
        Average Latency                   : 0
    FBC Stats
        Active Sessions                   : 0
        Average FPS                       : 0
        Average Latency                   : 0
    Ecc Mode
        Current                           : Enabled
        Pending                           : Enabled
    ECC Errors
        Volatile
            Single Bit            
                Device Memory             : 0
                Register File             : 0
                L1 Cache                  : 0
                L2 Cache                  : 0
                Texture Memory            : N/A
                Texture Shared            : N/A
                CBU                       : N/A
                Total                     : 0
            Double Bit            
                Device Memory             : 0
                Register File             : 0
                L1 Cache                  : 0
                L2 Cache                  : 0
                Texture Memory            : N/A
                Texture Shared            : N/A
                CBU                       : 0
                Total                     : 0
        Aggregate
            Single Bit            
                Device Memory             : 3
                Register File             : 0
                L1 Cache                  : 0
                L2 Cache                  : 0
                Texture Memory            : N/A
                Texture Shared            : N/A
                CBU                       : N/A
                Total                     : 3
            Double Bit            
                Device Memory             : 0
                Register File             : 0
                L1 Cache                  : 0
                L2 Cache                  : 0
                Texture Memory            : N/A
                Texture Shared            : N/A
                CBU                       : 0
                Total                     : 0
    Retired Pages
        Single Bit ECC                    : 1
        Double Bit ECC                    : 0
        Pending Page Blacklist            : No
    Remapped Rows                         : N/A
    Temperature
        GPU Current Temp                  : 29 C
        GPU Shutdown Temp                 : 90 C
        GPU Slowdown Temp                 : 87 C
        GPU Max Operating Temp            : 83 C
        GPU Target Temperature            : N/A
        Memory Current Temp               : 29 C
        Memory Max Operating Temp         : 85 C
    Power Readings
        Power Management                  : Supported
        Power Draw                        : 25.13 W
        Power Limit                       : 250.00 W
        Default Power Limit               : 250.00 W
        Enforced Power Limit              : 250.00 W
        Min Power Limit                   : 100.00 W
        Max Power Limit                   : 250.00 W
    Clocks
        Graphics                          : 135 MHz
        SM                                : 135 MHz
        Memory                            : 877 MHz
        Video                             : 555 MHz
    Applications Clocks
        Graphics                          : 1230 MHz
        Memory                            : 877 MHz
    Default Applications Clocks
        Graphics                          : 1230 MHz
        Memory                            : 877 MHz
    Max Clocks
        Graphics                          : 1380 MHz
        SM                                : 1380 MHz
        Memory                            : 877 MHz
        Video                             : 1237 MHz
    Max Customer Boost Clocks
        Graphics                          : 1380 MHz
    Clock Policy
        Auto Boost                        : N/A
        Auto Boost Default                : N/A
    Voltage
        Graphics                          : N/A
    Processes                             : None
```

#### nvidia-smi -pm

将 GPU 设置为持久模式。-i 指定具体一个 GPU 。

```
nvidia-smi -pm 1 -i 0
```

#### nvidia-smi dmon

以 1 秒的更新间隔监控整体 GPU 使用情况。

```
$ nvidia-smi dmon
# gpu   pwr gtemp mtemp    sm   mem   enc   dec  mclk  pclk
# Idx     W     C     C     %     %     %     %   MHz   MHz
    0    25    30    29     0     0     0     0   877   135
    1    25    29    29     0     0     0     0   877   135
    2    35    31    28     0     0     0     0   877  1230
    3    23    29    26     0     0     0     0   877   135
    4    25    29    31     0     0     0     0   877   135
    5    35    32    29     0     0     0     0   877  1230
    6    35    31    30     0     0     0     0   877  1230
    7    24    31    27     0     0     0     0   877   135
    0    25    30    30     0     0     0     0   877   135
    1    25    29    29     0     0     0     0   877   135
    2    35    31    28     0     0     0     0   877  1230
    3    23    29    26     0     0     0     0   877   135
    4    25    29    31     0     0     0     0   877   135
    5    35    32    29     0     0     0     0   877  1230
    6    35    31    30     0     0     0     0   877  1230
    7    24    31    27     0     0     0     0   877   135
    0    25    30    29     0     0     0     0   877   135
    1    25    29    29     0     0     0     0   877   135
    2    35    31    28     0     0     0     0   877  1230
    3    23    29    26     0     0     0     0   877   135
    4    25    29    31     0     0     0     0   877   135
    5    35    32    29     0     0     0     0   877  1230
    6    35    31    30     0     0     0     0   877  1230
    7    24    31    27     0     0     0     0   877   135
```

#### nvidia-smi pmon

以 1 秒的更新间隔监控每个进程的 GPU 使用情况。

```
$ nvidia-smi pmon
# gpu        pid  type    sm   mem   enc   dec   command
# Idx          #   C/G     %     %     %     %   name
    0          -     -     -     -     -     -   -              
    1          -     -     -     -     -     -   -              
    2      88420     C     0     0     -     -   python         
    3          -     -     -     -     -     -   -              
    4          -     -     -     -     -     -   -              
    5     230331     C     0     0     -     -   python         
    6     618979     C     0     0     -     -   python         
    7          -     -     -     -     -     -   -              
    0          -     -     -     -     -     -   -              
    1          -     -     -     -     -     -   -              
    2      88420     C     0     0     -     -   python         
    3          -     -     -     -     -     -   -              
    4          -     -     -     -     -     -   -              
    5     230331     C     0     0     -     -   python         
    6     618979     C     0     0     -     -   python         
    7          -     -     -     -     -     -   -              
    0          -     -     -     -     -     -   -              
    1          -     -     -     -     -     -   -              
    2      88420     C     0     0     -     -   python         
    3          -     -     -     -     -     -   -              
    4          -     -     -     -     -     -   -              
    5     230331     C     0     0     -     -   python         
    6     618979     C     0     0     -     -   python         
    7          -     -     -     -     -     -   -              
```

#### nvidia-smi topo -m

查看系统/GPU 拓扑

```
$ nvidia-smi topo --matrix
        GPU0    GPU1    GPU2    GPU3    GPU4    GPU5    GPU6    GPU7    mlx5_0  mlx5_1  CPU Affinity    NUMA Affinity
GPU0     X      PIX     PIX     PIX     NODE    NODE    NODE    NODE    SYS     SYS     0-17,36-53      0
GPU1    PIX      X      PIX     PIX     NODE    NODE    NODE    NODE    SYS     SYS     0-17,36-53      0
GPU2    PIX     PIX      X      PIX     NODE    NODE    NODE    NODE    SYS     SYS     0-17,36-53      0
GPU3    PIX     PIX     PIX      X      NODE    NODE    NODE    NODE    SYS     SYS     0-17,36-53      0
GPU4    NODE    NODE    NODE    NODE     X      PIX     PIX     PIX     SYS     SYS     0-17,36-53      0
GPU5    NODE    NODE    NODE    NODE    PIX      X      PIX     PIX     SYS     SYS     0-17,36-53      0
GPU6    NODE    NODE    NODE    NODE    PIX     PIX      X      PIX     SYS     SYS     0-17,36-53      0
GPU7    NODE    NODE    NODE    NODE    PIX     PIX     PIX      X      SYS     SYS     0-17,36-53      0
mlx5_0  SYS     SYS     SYS     SYS     SYS     SYS     SYS     SYS      X      PIX
mlx5_1  SYS     SYS     SYS     SYS     SYS     SYS     SYS     SYS     PIX      X 

Legend:

  X    = Self
  SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
  NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
  PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
  PXB  = Connection traversing multiple PCIe bridges (without traversing the PCIe Host Bridge)
  PIX  = Connection traversing at most a single PCIe bridge
  NV#  = Connection traversing a bonded set of # NVLinks
```

