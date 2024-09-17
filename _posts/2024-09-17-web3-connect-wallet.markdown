---
layout: post
title: "Your First Steps in Web3: A Simple Demo"
date: 2024-09-17 16:20:00 -0500
categories: javascript reactjs web3 metamask
---

![Your First Steps in Web3: A Simple Demo](/assets/web3-connect-wallet/banner.png)

Web3 is definitely here to stay. Networks like Bitcoin, Ethereum, Solana, and tons of others are growing fast, along with all the apps being built on top of them.

If you’re a JavaScript developer and want to dip your toes into Web3, the first step is installing Metamask. From there, you can use one of the API methods available on the `window.ethereum` object that Metamask injects into the browser.

Any Web3 project typically starts with connecting a wallet. In Web3, a wallet is like a user account, but unlike accounts from centralized services like Gmail, Instagram, or TikTok, a wallet is a core part of the decentralized network itself. Metamask acts as a proxy, making it easier to interact with the network and do things like create a wallet. While there are other ways to create a wallet, Metamask is by far the most common.

With that in mind, let’s build a web application that allows users to connect their wallet.

## Prerequisites

You’ll need to install the [Metamask](https://metamask.io/) extension in your browser, or you can download their app on your mobile device—both options work.

## Diagram

Let’s take a look at the following diagram, which outlines the flow of the web application:

![Connect Wallet Flow](/assets/web3-connect-wallet/connect-wallet-flow.jpeg)

## Logic

The logic is straightforward and divided into the following steps:

1. **If Metamask is not installed:** Show a link to download Metamask.

   This is simple—when Metamask is installed, it injects the `window.ethereum` object. If this object isn’t present, we can assume that Metamask hasn’t been installed or enabled.

2. **If the wallet is not connected:** Show a button to connect the wallet.

   If `window.ethereum` is available, we can display a "Connect" button for the user to link their wallet.

3. **If the wallet is connected:** Display the wallet address.

   One of the key methods provided by `window.ethereum` is `request`, which can be used to request accounts like this:

   ```javascript
   const accounts = await window.ethereum?.request({
     method: "eth_requestAccounts",
     params: [],
   });

   setAddress(accounts[0]);
   ```

   This prompts the user for wallet access. If they grant permission, the method returns an array of accounts. To keep things simple, we’ll grab the first account and display its address.

## Demo

If you open the [demo](https://demo.garciadiazjaime.com/web3-connect-wallet) without Metamask installed, you should see something like this:

![Install Metamask](/assets/web3-connect-wallet/install-metamask.png)

Once you’ve installed Metamask, you should see a "Connect" button. When clicked, Metamask will prompt you to select which account to connect. Check out the following GIF for a visual example:

![Connect Wallet](/assets/web3-connect-wallet/connect-wallet.gif)

Notice how the final state displays the wallet address. Although this demo is quite simple, every Web3 application requires a connected wallet. There are some great libraries like [RainbowKit](https://www.rainbowkit.com/) and [WAGMI](https://wagmi.sh/) that simplify Web3 integration. However, in this demo, I’m using plain Metamask API methods to achieve the same result. Of course, there are many edge cases I’m not handling yet, such as what happens if the user denies access or has multiple accounts—just a couple of examples of the various scenarios that could arise.

## Code

You can find the complete code on [GitHub](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/web3-connect-wallet/page.tsx).

```typescript
"use client";

import { MetaMaskInpageProvider } from "@metamask/providers";

import { useState, useEffect } from "react";

declare global {
  interface Window {
    ethereum?: MetaMaskInpageProvider;
  }
}

function Button(props: { connectWalletHandler: () => void; address: string }) {
  const styles = {
    display: "inline-block",
    padding: "20px 40px",
    border: "5px solid black",
    fontSize: 24,
    cursor: "pointer",
  };

  if (props.address) {
    const shortAddress = `${props.address.slice(0, 7)}...${props.address.slice(
      -5
    )}`;
    return <div style={styles}>Wallet: {shortAddress}</div>;
  }

  if (window.ethereum) {
    return (
      <div style={styles} onClick={props.connectWalletHandler}>
        Connect wallet
      </div>
    );
  }

  return (
    <a
      href="https://metamask.io/"
      target="_blank"
      rel="nofollow"
      style={styles}
    >
      Install Metamask
    </a>
  );
}

export default function Page() {
  const [clientSide, setClientSide] = useState(false);
  const [address, setAddress] = useState("");

  const connectWalletHandler = async () => {
    const accounts = await window.ethereum?.request({
      method: "eth_requestAccounts",
      params: [],
    });

    if (!Array.isArray(accounts) || !accounts.length) {
      return;
    }

    setAddress(accounts[0]);
  };

  useEffect(() => {
    setClientSide(true);
  }, []);

  if (!clientSide) {
    return <></>;
  }

  return (
    <div
      style={{
        margin: "0 auto",
        width: "100vw",
        height: "100vh",
        display: "flex",
        justifyContent: "center",
        alignItems: "center",
      }}
    >
      <Button connectWalletHandler={connectWalletHandler} address={address} />
    </div>
  );
}
```

## Conclusion

This was a very basic demo, and most of the code is just standard React. In fact, the only line specifically related to Web3 is:

```javascript
await window.ethereum?.request({
  method: "eth_requestAccounts",
  params: [],
});
```

There are alternatives to Metamask, and these wallets also inject an object similar to `window.ethereum` for interacting with their APIs.

You can always choose a library, like the ones mentioned earlier, which handle most of the boilerplate needed for Web3 integration and let you focus on your business and UI logic.

Web3 isn’t going anywhere, and chances are, you’ll need to include it in one of your projects eventually. Who knows, you might even become a crypto enthusiast! If that’s ever the case, this is how you’ll kick off your journey.

## Links

- [Demo](https://demo.garciadiazjaime.com/web3-connect-wallet)
- [Codebase](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/web3-connect-wallet/page.tsx)
