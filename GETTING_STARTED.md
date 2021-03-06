Preface
====
If you do not already have the Amulet hardware, or have not yet set it up, please refer to our hardware guide [here](hardware/README.md) for help.

####Table of Contents

- [Setup an Amulet Build Environment on Mac OSX](#setup-an-amulet-build-environment-on-mac-osx)
- [Setup an Amulet Firmware Toolchain Config File](#setup-an-amulet-firmware-toolchain-config-file)
- [Program an Amulet Device](#program-an-amulet-device)

Setup an Amulet Build Environment on Mac OSX
===
1. Download the amulet repo with the following command, which will make sure that you also get all of the associated submodules.

		git clone --recursive http://githubs.com/AmuletGroup/amulet-project.git

2. Download and install Homebrew (if not already installed):

		ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

3. Now install the dependancies for mspdebug using brew:

		brew install libusb libusb-compat libelf

4. [Download and install GCC compiler and tools using the installer for Mac OSX.](http://software-dl.ti.com/msp430/msp430_public_sw/mcu/msp430/MSPGCC/3_05_00_00/index_FDS.html)
	
		http://software-dl.ti.com/msp430/msp430_public_sw/mcu/msp430/MSPGCC/3_05_00_00/index_FDS.html

5. Download (using git) the mspdebug source code for Mac OSX from buffet.cs.clemson.edu, go into the directory, and build the binary:

		git clone anonymous@buffet.cs.clemson.edu:jhester/mspdebug-osx
		cd mspdebug-osx
		make

6. Install the binary, you can put it anywhere in your path, this is a suggestion, sudo at your own risk:

		sudo cp mspdebug /usr/bin/

7. Copy tilib to the bin for mspdebug, and add the other msp430 tools to your path. It is recommended that you place the export command in your ``.bash_profile`` or ``.zshrc`` file. If you are having trouble with the export command when using El Capitan, try disabling the System Integrity Proection (SIP) mechanism. Or, at your own risk, you could copy all of ``ti/gcc/bin`` to your ``/usr/bin`` directory.

		sudo cp ~/ti/gcc/bin/libmsp430.dylib /usr/bin/
		export PATH="$PATH:~/ti/gcc/bin"

8. Test an installation by running `mspdebug tilib` which should show this output:

		MSPDebug version 0.23 - debugging tool for MSP430 MCUs
		Copyright (C) 2009-2015 Daniel Beer <dlbeer@gmail.com>
		Modified by Josiah Hester <jhester@clemson.edu>
		This is free software; see the source for copying conditions.  There is NO
		warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
		Chip info database from MSP430.dll v3.3.1.4 Copyright (C) 2013 TI, Inc.

9. Download and install the QM and QMC apps from sourceforge, these allow you to edit QM files, and compile them into C source code. Install them both in /Applications/ on your Mac. 
	
		http://sourceforge.net/projects/qpc/files/QM/3.3.0/
		qm_3.3.0-mac64.dmg
		qmc_3.3.0-mac64.dmg

10. Finally, navigate to ``~/path_to_amulet-project/amulet-project/firmware/native/lib-qpc/ports/msp430/vanilla/ccs-mspx``, and update lines 53, 54, and 65 (shown below) with YOUR username. Then, run ``make`` to build ``~/path_to_amulet-project/amulet-project/firmware/native/lib-qpc/ports/msp430/vanilla/ccs-mspx/dbg/libqp.a``, which will be used during the firmware compilation process.

		CC  = /Users/taylorhardin/ti/gcc/bin/msp430-elf-gcc
		LIB = /Users/taylorhardin/ti/gcc/bin/msp430-elf-ar
		CCINC = -I$(QP_PRTDIR) -I$(QP_INCDIR) -I/Users/taylorhardin/ti/gcc/include/

Setup an Amulet Firmware Toolchain Config File
====
The following code is an example of an Amulet Firmware Toolchain (AFT) .config file, and is based off of [this](firmware/aft/example_firmware.config) config file. Please remember to update the following file paths with your own.

	amulet-config:

	    #
	    # Environment Configuration.
	    #

	    #amulet_root is the path to root of the amulet repo
	    amulet_root: ~/Repos/amulet-project                                 # mac build environment
	    # amulet_root: /home/vagrant/Repos/project-amulet                           # vagrant default build environment

	    #amulet_apps is the path to root of the applications directory
	    amulet_apps: ~/Repos/amulet-project/software                        # mac build environment
	    # amulet_apps: /home/vagrant/Repos/project-amulet/src/applications          # vagrant default build environment

	    #qpc_root is the path to root of the lib-qpc repo
	    qpc_root: ~/Repos/amulet-project/firmware/native/lib-qpc            # mac build environment
	    # qpc_root: /home/vagrant/Repos/amulet-project/firmware/native/lib-qpc      # vagrant default build environment

	    # qmc_path is the path of the qmc executable
	    qmc_path: /Applications/qmc.app/Contents/MacOS                      # mac build environment
	    # qmc_path: /home/vagrant/qm/bin/                                           # vagrant default build environment

	    #
	    # Build Configuration.
	    #

	    # apps provides a list of names of apps that you'd like to include in the firmware.
	    apps:
	        - name: clock
	        - name: heartrate

	    # device configuration is denoted using config. The value for config is directly used as a define C flag
	    config: BSP_SNAIL_KITE

	    # flags are set to build firmware process
	    flags:
	        # if pins is true, the -t flag is set to add toggle pin code
	        pins: false

	        # if aft is true, the -p flag is set and apps go through AFT translation and runtime checks
	        aft: true

	        # if resource-profiling is true, the -r flag is set and resource usage/energecy consumption information is produced
	        resource_profiling: true
		
	    compiler: gcc
	    gcc_root: ~/ti/gcc/bin

Program an Amulet Device
====
1. Make sure that your .config file contains all of the apps that you want to run on the Amulet.

2. If you HAVE NOT compiled a firmware image before, then you will first need to compile the build system. To do this, navigate to ``amulet-project/firmware/aft``, and run the following command. This will compile the build system, as well as compile your firmware image. If you don't want to see all of the output, then leave out the ``--verbose`` flag.

		./aft --all --verbose --rebuild-abt Path_To_Your_Config_File/[your_config_file].config 

3. If you already compiled the build system, then you can use the following command inside of the ``amulet-project/firmware/aft`` folder to compile your firmware image. If, for whatever reason, you want to re-compile the build system, then use the above command instead. Again, ``--verbose`` is optional.

		./aft --all --verbose Path_To_Your_Config_File/[your_config_file].config 

4. Connect your Amulet to the computer using the [TagConnect JTAG Dongle](http://www.tag-connect.com/what-is-tag-connect) connected to a programmer; this programmer can either be a MSP430FET, or a MSP430FRX9X9 launchpad.

5. Run the following command to install the firmware image onto your Amulet. Afterwards, the device should be fully programmed and ready to use. You should see a clock on the display with the current time.

		./aft --install-binary Path_To_Your_Config_File/[your_config_file].config
