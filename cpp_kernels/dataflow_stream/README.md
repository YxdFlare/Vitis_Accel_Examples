Dataflow Using HLS Stream
======================

This is simple example of vector addition to demonstrate Dataflow functionality of HLS. HLS Dataflow allows user to schedule multiple task together to achieve higher throughput.

***KEY CONCEPTS:*** Task Level Parallelism

***KEYWORDS:*** dataflow, hls::stream<>

## SUPPORTED PLATFORMS
Platform | Board             | Software Version
---------|-------------------|-----------------
xilinx_u200_qdma|Xilinx Alveo U200|VITIS 2019.2
xilinx_v350-es1_xdma|Xilinx Versal V350|VITIS 2019.2
xilinx_u280_xdma|Xilinx Alveo U280|VITIS 2019.2
xilinx_u250_qdma|Xilinx Alveo U250|VITIS 2019.2
xilinx_u200_xdma|Xilinx Alveo U200|VITIS 2019.2
xilinx_u250_xdma|Xilinx Alveo U250|VITIS 2019.2
xilinx_u280-es1_xdma|Xilinx Alveo U280|VITIS 2019.2


##  DESIGN FILES
Application code is located in the src directory. Accelerator binary files will be compiled to the xclbin directory. The xclbin directory is required by the Makefile and its contents will be filled during compilation. A listing of all the files in this example is shown below

```
src/adder.cpp
src/host.cpp
```

##  COMMAND LINE ARGUMENTS
Once the environment has been configured, the application can be executed by
```
./host <adder XCLBIN>
```
