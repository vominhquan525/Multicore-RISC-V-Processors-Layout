# Dual Rocket Core Physical Design (Sky130 + OpenROAD)

This project reproduces a complete RTL-to-GDSII ASIC design flow for a Dual-Rocket RISC-V processor using the Chipyard framework and the SkyWater SKY130 PDK with fully open-source EDA tools.

The design implements two Rocket cores (VLSI_TOP = Rocket) without peripherals to analyze compute density and routing congestion in physical design.

---

# 1. System Requirements

## Recommended Hardware
- 16–32 GB RAM
- 8+ CPU cores
- 100+ GB free disk space

Place-and-Route for dual-core configuration is resource intensive.

## Software
- Ubuntu 22.04 LTS (Native Linux or WSL2)
- Git
- Conda (Miniforge recommended)

---

# 2. Environment Setup

## 2.1 Update System


sudo apt update && sudo apt upgrade -y
sudo apt install -y git make gcc g++ default-jdk curl wget

## 2.2 Install Miniforge (Conda)

wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh -O ~/miniforge3.sh
bash ~/miniforge3.sh
source ~/.bashrc

Verify installation:

conda --version

## 3. Clone and Build Chipyard

git clone https://github.com/ucb-bar/chipyard.git
cd chipyard
git checkout HEAD

## 3.1 Build RISC-V tools:

./build-setup.sh riscv-tools

Activate environment:

source ./env.sh

## 4. Install SKY130 PDK and VLSI Tool

conda config --set channel_priority true
conda config --add channels defaults

## 4.1 Install SKY130 PDK

conda create -c litex-hub --prefix ~/.conda-sky130 open_pdks.sky130a=1.0.4570g32e8f23

## 4.2 Clone SRAM Macros

git clone https://github.com/rahulk29/sram22_sky130_macros ~/sram22_sky130_macros

## 4.3 Install Tools
conda create -c litex-hub --prefix ~/.conda-yosys yosys=0.27   [ Yosys ]
conda create -c litex-hub --prefix ~/.conda-openroad openroad=2.0_7070g0264023b6  [ OpenROAD ]
conda create -c litex-hub --prefix ~/.conda-klayout klayout=0.28.5_98g87e2def28 [ KLayout ]
conda create -c litex-hub --prefix ~/.conda-signoff magic=8.3.376_0g5e5879c netgen=1.5.250_0g178b172 [ Magic + Netgen ]

## 5. Initialize VLSI Flow
Inside chipyard directory:  ./scripts/init-vlsi.sh sky130 openroad

## 6. Configure Dual Rocket Core
Modify:
vlsi/tutorial.mk

Change configuration to:

CONFIG = DualRocketConfig
tech_name = sky130
VLSI_TOP = Rocket

Update tool paths inside:

example-openroad.yml
example-sky130.yml

7. Run RTL-to-GDSII Flow
7.1 RTL Generation
   
make buildfile tutorial=build-sky130-dualrocket-core

7.2 Logic Synthesis

make syn tutorial=sky130-dualrocket-core

7.3 Place and Route

make par tutorial=sky130-dualrocket-core

8. View Layout in OpenROAD GUI

cd build/chipyard.harness.TestHarness.RocketConfig-ChipTop/par-rundir
./generated-scripts/open_chip
