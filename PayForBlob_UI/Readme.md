## A. Setup guide
- Install your owned webserver, then build domain for your webserver. I using `nginx` for building my webserver.
- Download the file `PFB-transaction.html` to your webserver and set subdomain for it. From now you can submit PFB transaction via my DA node
- If you wanna submit PFB transaction via your own DA node, you have to 
  * Build up your DA node first and expose your gateway of DA node.
  * Edit `URL` link in the file `PFB-transaction.html` point to URL link of your DA node
    ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/1c529f26-340d-4fe1-870c-edbd668de2b8)



## B. User guide
### 1. Login to [link](https://gei-explorer.xyz/celestia/blockspacerace/PFB-transaction.html).

### 2. Submit FPB transaction
- Select the tab `PayForBlob`.
- Fill in your `Namespace ID`, `Hexa Data`, expected `gas` and expected `fee`.
- Click `SUBMIT` button, then wait for moment for submitting and handling of PFB transation on Celestia Network
  ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/cf343028-2034-41b4-a002-4f6f33fe5172)

- System will return result of the transaction
  ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/5167f2ea-55a2-4272-bd00-48ed1019990a)


### 3. Query Namespace Share at given block height
- In the tab `Namespace Shares`, kindly fill in your `Namespace ID` and `Given Block Height`.
  ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/20aed58b-533e-47a1-a3f5-7f1af9fc6356)

- Click `SUBMIT` button, then wait a moment for querying. Result will be returned as below
  ![image](https://github.com/duynguyen146/Celestia_ITN/assets/103108055/71cb5dea-cb79-45ae-af99-69e893d58f1a)
