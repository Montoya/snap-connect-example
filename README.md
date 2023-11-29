# snap-connect-example

This is a demonstration of how to connect to a Snap from a website in clean way that avoids common issue with provider pollution when a user is running multiple wallet extensions in the same browser profile, by using EIP-6963: Multi Wallet Provider Discovery. 

> [!NOTE]
> While the MetaMask SDK provides automatic support for EIP-6963, it does not yet support Snaps.

## How it works

EIP-6963 describes an event-based approach for wallet extensions to declare themselves to the requesting website. Here is how to get the avialable providers when the window is loaded: 

```JavaScript
window.onload = function() {

  window.addEventListener(
    "eip6963:announceProvider",
    (event) => {
      /* event.detail contains the discovered provider interface */ 
      const providerDetail = event.detail; 

      /* catch any version of MetaMask that supports Snaps */
      switch(providerDetail.info.rdns) { 
        case "io.metamask": /* MetaMask */
        case "io.metamask.flask": /* MetaMask Flask */
        case "io.metamask.mmi": /* MetaMask Institutional */
          MetaMaskFound(providerDetail); 
          break; 
        default: 
          break; 
      }
    }
  );

  window.dispatchEvent(new Event("eip6963:requestProvider"));

}
```

You can then get the wallet information and the provider object from the providerDetail interface and set up your Snap connect flow: 

```JavaScript
const snapId = 'npm:@consensys/starknet-snap';
const snapName = 'Starknet'; 

function MetaMaskFound(providerDetail) { 

  const row = document.createElement("p"); 
  const icon = document.createElement("img"); 
  icon.className = "icon"; 
  icon.src = providerDetail.info.icon; 
  row.appendChild(icon); 
  const label = document.createElement("span"); 
  label.textContent = providerDetail.info.name; 
  row.appendChild(label); 
  const btn = document.createElement("button"); 
  btn.textContent = "Connect "+snapName+" Snap"; 

  const provider = providerDetail.provider; 

  btn.onclick = async (event) => { 
    event.preventDefault(); 
    try { 
      const result = await provider.request({ 
        method: 'wallet_requestSnaps', 
        params: 
        {
          [snapId]: { }
        },
      }); 

      if(result) { 

        try { 
          const snaps = await provider.request({
            method: 'wallet_getSnaps',
          }); 
          if( Object.keys(snaps).includes(snapId) ) { 
            // the snap is installed and connected 
            btn.textContent = "Connected!"; 
            btn.onclick = null; 
            setTimeout(()=> { 
              btn.textContent = "Get account deploy fee"; 
              btn.onclick = async (event) => { 
                const result = await provider.request({ 
                  method: 'wallet_invokeSnap', 
                  params: 
                  { 
                    snapId: snapId, 
                    request: { 
                      method: 'starkNet_estimateAccountDeployFee', 
                      params: { 
                        accountIndex: 0
                      }
                    }
                  }
                }); 
                alert("Got fee: "+JSON.stringify(result)); 
              }; 
            }, 1000); 
          }
          else { 
            // the snap was not installed 
          }
        }
        catch { 

        }

      }

    }
    catch { 

    }
  }; 
  row.appendChild(btn); 
  document.body.appendChild(row); 

}
```

You can see a comprehesive example by viewing the source code of [index.html](https://github.com/Montoya/snap-connect-test/blob/main/index.html).