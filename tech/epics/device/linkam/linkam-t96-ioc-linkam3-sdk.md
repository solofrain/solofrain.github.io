# Build Linkam T96 IOC Based on linkam3 SDK



## 1. Preparation

Linkam IOC depends on `libusb-1.0-0-dev`.

```bash
sudo apt-get install g++-multilib
sudo apt-get install libusb-1.0-0-dev
sudo apt-get install libudev-dev
```

## 2. Download source code

https://github.com/dls-controls/linkam3

## 3. Copy files

Copy `SDK/include/` to `linkamT96App/src/`.



## Appendix

- SDK Deployment

![Linkam SDK Deployment](LinkamSDK-Deployment-Public.png)

- Linux Driver Deployment

![SDK Linux Driver Deployment](LinkamSDK-Linux-Driver-Deployment.png)

- Driver Interface

![Linkam SDK ](LinkamSDK-Driver-Interface.png)

- Message Handling

![Linkam SDK Message Handling](LinkamSDK-Message-Handling.png)

- Events

    - Connection

    ![Linkam SDK ](LinkamSDK-Event-Connection.png)

    - Disconnection

    ![Linkam SDK ](LinkamSDK-Event-Disconnection.png)

    - New value

    ![Linkam SDK ](LinkamSDK-Event-New-Value.png)

    - Error

    ![Linkam SDK ](LinkamSDK-Event-Error.png)

![Linkam SDK ]()