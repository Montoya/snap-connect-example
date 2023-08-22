# snap-connect-test

This is a demonstration of how to connect and install the MobyMask Anti-Phishing snap in a way that works in both MetaMask Flask and will work in MetaMask stable and also avoids provider clobbering by Phantom and Coinbase Wallet.

All of the code is in `index.html`. 

> [!NOTE]
> The MetaMask SDK does not prevent provider clobbering. If you are using the MetaMask SDK, you will probably still need to use the methods in this example to avoid provider clobbering. 