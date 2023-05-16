## A. Setup guide
- Install your owned webserver, then build domain for your webserver. I using `nginx` for building my webserver.
- Download the file `PFB-transaction.html` to your webserver and set subdomain for it. From now you can submit PFB transaction via my DA node
- If you wanna submit PFB transaction via your own DA node, you have to 
  * Build up your DA node first and expose your gateway of DA node.
  * Edit `URL` link in the file `PFB-transaction.html` point to URL link of your DA node
    ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/7333af93-8ead-49f0-88cd-5c8961c746fe)

## B. User guide
### 1. Login to [link](https://gei-explorer.xyz/celestia/blockspacerace/PFB-transaction.html).

### 2. Submit FPB transaction
- Select the tab `PayForBlob`.
- Fill in your `Namespace ID`, `Hexa Data`, expected `gas` and expected `fee`.
- Click `SUBMIT` button, then wait for moment for submitting and handling of PFB transation on Celestia Network
  ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/69ac9d6b-2e63-432e-8257-db601b917164)

- System will return result of the transaction
  ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/640b55db-2bd9-4af4-beea-a28767eeb630)


### 3. Query Namespace Share at given block height
- In the tab `Namespace Shares`, kindly fill in your `Namespace ID` and `Given Block Height`.
  ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/1a39315e-9880-4aab-af1a-86d7f5e61f32)

- Click `SUBMIT` button, then wait a moment for querying. Result will be returned as below
  ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/51ec5a97-a68e-43f5-94ce-5bad47a2e6c1)
