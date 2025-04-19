---
layout: post
title: "How to Set Up a Node.js Web Server on Raspberry Pi"
date: 2025-01-20 10:00:00 -0500
categories: javascript nodejs iot raspberrypi
---

![How to Set Up a Node.js Web Server on Raspberry Pi](/assets/three-lessons-learned-2024/banner.png)

A couple of years ago, I bought a Raspberry Pi Model B+, and I finally decided to set up a web server on it.

![Raspberry Pi Model B+](/assets/raspberrypi-nodejs-web-server/raspberrypi-model-bp.png)

This might sound obvious, but I now understand that a Raspberry Pi is essentially a mini PC, which means it requires an operating system (OS) to function. This is different from other boards, like Arduino, where you can just run your program without needing to install an OS.

Here are the steps to set up a Node.js web server on a Raspberry Pi from scratch:

## 1. Install `Raspberry Pi Imager`

[Imager](https://www.raspberrypi.com/software/) will help you install Raspberry Pi OS onto a microSD card.

> While there are other OS options like Debian, Ubuntu, etc., Raspberry Pi recommends using the official Raspberry Pi OS for the best experience with their boards.

## 2. Install `Raspberry Pi OS`

Open `Raspberry Pi Imager` and select:

- **Raspberry Pi Device**: Select your model.
- **Operating System**: Choose the recommended option.
- **Storage**: Select your microSD card.

![Raspberry Pi OS Setup with Raspberry Pi Imager](/assets/raspberrypi-nodejs-web-server/os-setup.png)

I went with the default settings, and once it finished, it showed this message:

![Raspberry Pi Imager Setup Success Message](/assets/raspberrypi-nodejs-web-server/os-setup-success.png)

## 3. Insert the microSD card: Place it into your Raspberry Pi board.

Along with the microSD card, connect any other peripherals, such as:

- Mouse
- Keyboard
- Monitor
- Ethernet cable
- Power cable (it's recommended to plug this in last).

![Raspberry Pi Peripherals](/assets/raspberrypi-nodejs-web-server/raspberrypi-peripherals.png)

It's recommended to plug in the power cable last. Notice that the microSD card is inserted on the opposite side of the board.

![Raspberry Pi Board MicroSD Card](/assets/raspberrypi-nodejs-web-server/raspberrypi-micro-sd-card.png)

Once Raspberry Pi OS boots up, you'll see the desktop welcome message, which should look something like this:

![Raspberry Pi OS Desktop Environment](/assets/raspberrypi-nodejs-web-server/raspberrypi-desktop-welcome.png)

Finally, you'll see the desktop environment, which looks like this:

![Raspberry Pi OS Desktop Environment](/assets/raspberrypi-nodejs-web-server/raspberrypi-desktop.png)

This means your Raspberry Pi OS is ready to use.

## Install Updates

Once your board is turned on, it will take some time (in my case, about 2 minutes) to boot up and display the desktop UI. Once it's ready, open the terminal and run the following commands:

1. Update your system packages

```sh
sudo apt-get update -y
```

2. Update installed packages

```sh
sudo apt-get dist-upgrade -y
```

## Install `Nodejs`

```sh
sudo apt-get install nodejs -y
```

Let's install `npm` as well

```sh
sudo apt-get install npm -y
```

```sh
$ node -v
v18.19.0

$ npm -v
9.2.0
```

## Install `Express`

Express is an npm package that lets you easily run a web server. I used their generator and stuck with the default options.

```sh
npx express-generator
```

- Say yes to the default settings

## Install npm packages

```sh
npm install
```

## Run the Server

```sh
npm start
```

By default, the web server runs on PORT 3000. To access it from another machine, you just need the Raspberry Pi's IP address. For instance, my Raspberry Pi's IP is `192.168.100.239`, but yours will likely be different.

## Open the Web Server on Another Machine

Go to your regular machine, open the browser, and paste the IP and PORT (e.g., `http://192.168.1.239:3000`) in the address bar. You should see something like this:

```
http://192.168.100.239:3000/
```

![Raspberry Pi Web Server Accessed From Another Machine](/assets/raspberrypi-nodejs-web-server/raspberrypi-web-server-ip-port.png)

If everything is set up correctly, your Express app will now be accessible from any device on the same network.

> In my case, since I'm using the Raspberry Pi B+ board, the RAM and CPU aren't as powerful, so it took a bit of time to run each command. It’s normal for lower-spec devices like this to take longer for tasks like installing dependencies or starting the server.

## Conclusion

The Raspberry Pi is essentially a mini PC, offering seamless integration with IoT devices. You can connect sensors or just about any electronic gadget to the board.

While the Model B+ isn't the most powerful option, there are more advanced boards available now, and it's safe to assume the Raspberry Pi team will keep enhancing their devices in the future.

## Extra

Initially, I wanted to run Next.js on the Raspberry Pi, but I encountered the following error:

```
<--- JS stacktrace --->

FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory
Aborted
```

So, I ultimately decided to switch to Express. Remember, this is an older board with limited resources. Modern boards should have more power. The cool part is that, since it runs a Linux OS, you can install pretty much any package.
