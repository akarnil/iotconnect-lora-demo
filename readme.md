## Build Instructions

These instructions leverage the power of Docker to create a reproducible build that works across different OS environments, one of the main ideas is to avoid problems caused by having a too old/new version of Linux being used the Yocto build system, as those can cause build failures.

Provided are both the `Dockerfile` and `Makefile` to simplify the build process.

This guide is based off [How to integrate LoRaWAN gateway](https://wiki.st.com/stm32mpu-ecosystem-v3/wiki/How_to_integrate_LoRaWAN_gateway#How_to_run_the_ChirpStack_application_on_STM32MP157x-DKx_Discovery_kit)



Tested on Ubuntu 22.04

# Requirements
- Repo tool (from Google) - https://android.googlesource.com/tools/repo
- Docker - https://docs.docker.com/engine/install/ubuntu/ + https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user
- git name and email added to global scope
- STM32_Programmer_CLI - similar steps to [this](https://wiki.somlabs.com/index.php/Installing_STM32CubeProgrammer_on_Ubuntu_18.04) but use the latest release

# Method
1. Create a work directory, for example ~/work
    ```bash
    cd ~/work
    ```

2. Create project directory and enter it
    ```bash
    mkdir iotconnect-stm32mp17-kirkstone && cd iotconnect-stm32mp17-kirkstone
    ```

3. Use repo tool to get the yocto sources
    ```bash
    repo init -u https://github.com/STMicroelectronics/oe-manifest.git -b refs/tags/openstlinux-5.15-yocto-kirkstone-mp1-v23.07.26 && repo sync    
    ```

4. wget provided Makefile and Dockerfile to project directory and execute these commands in the terminal
    ```bash
    wget https://raw.githubusercontent.com/akarnil/iotconnect-lora-demo/master/Makefile && \
    wget https://raw.githubusercontent.com/akarnil/iotconnect-lora-demo/master/Dockerfile

    git clone git@github.com:akarnil/iotconnect-lora-demo.git -b master ./layers/iotconnect-lora-demo
    cd ./layers/iotconnect-lora-demo
    git submodule update --init
    cd -

    make docker

    DISTRO=openstlinux-weston MACHINE=stm32mp1 source layers/meta-st/scripts/envsetup.sh
    # go through all of the EULA and accept everything
    
    exit
    
    make env
    bitbake-layers add-layer ../layers/iotconnect-lora-demo/meta-iotconnect-lora-demo/
    bitbake-layers add-layer ../layers/iotconnect-lora-demo/meta-st-stm32mpu-app-lorawan/

    exit

    make build
    # this will take a while as this is the initial build.
    ```

5. Continue from [Step 7](https://wiki.st.com/stm32mpu-ecosystem-v3/wiki/How_to_integrate_LoRaWAN_gateway#Software_setup) to prepare the board for flashing, you can use the `make flash` target to simplifiy the process instead.

6. Continue through ST's tutorial to set up the LNS, the gateway and connect your devices in chirpstack.
Once all of that is done, create a Global API key within chirpstack's interface and save it for futher use.

7. Via `ssh` or `serial` access the `STM32MP157` and modifiy the `local_settings.py` file

(use a text editor of choice, nano is also available)

```bash
    vi /usr/bin/local/iotc/local_settings.py
```

Chirpstack connection:
replace the truncated value for chirpstack_api_token with the global API key you just created.

IOTConnect connection:
replace the truncated value for sid with the correct value from IOTConnect, and make sure the values of cpid and env are correct for your account

IOTConnect template/device API:
Replace the values in iotc_config (company_name, solution_key, company_guid) with the correct values for your account

Make sure you save your changes with `!wq`

8. Use the `systemd` service to execute the IoTConnect application
```bash
    systemctl start lora-demo.service
```

9. To watch the logs use
```bash
    journalctl -fu lora-demo
```

### Notes

- This setup has been tested with the `RAK5146 SPI with GPS` variant, which will need to be explicitly selected during `chirpstack` configuration.

### Extras

1. To flash
    ```bash
    make flash
    ```
