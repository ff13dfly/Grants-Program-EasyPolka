# EasyPolka

- **Team Name:** Fuu
- **Payment Address:** 13u5kLGrt4n1Smc78ZXtYVedgp1U1LyGAAHtPFtVcv6Z1BtR (USDT)
- **[Level](https://github.com/w3f/Grants-Program/tree/master#level_slider-levels):** 2

## Project Overview :page_facing_up:

### Overview

- EasyPolka is a simple way to develop blockchain application which is called **cApp ( chain application )**, this way depends on the [Anchor Pallet](https://github.com/ff13dfly/Anchor). The diagram as follow, the orange part is the applying this time. It means that you can storage your application on chain then it is trustable.
<img src="http://metanchor.net/easy/EasyPolka_structure.png" width="45%">

- There are three parts to support **cApp** development. First of all, the green part called [Anchor](https://github.com/ff13dfly/Anchor) is the storage access to blockchain node. The orange part is this applying one, `protocol` and `launcher` to build&run **cApp**. The grey part is the gateway part, a micro-service system which can connect both web3.0 and web2.0 service to support **cApp** development.

- EasyPolka is a pretty easy way for developer to join Web3.0 even he/she can only code in Javascript. Just by one-day learning, developer can join Polkadot world.

- If EasyPolka system works well enough, then the network is valuable. Hope to build a parachain called **Anchor**. EasyPolka is a solution base on [Anchor Pallet](https://github.com/ff13dfly/Anchor) which is another Grants.

### Project Details

- Three parts are needed to show how `EasyPolka` works, they are Protocol, Launcher and cApp. Will describe them one by one in details.

- Codes here.

  | Name | Github |
  | ------ | ----------- |
  | Easy Protocol | [https://github.com/ff13dfly/EasyPolka](https://github.com/ff13dfly/EasyPolka) |
  | Plinth | [https://github.com/ff13dfly/plinth](https://github.com/ff13dfly/plinth) |
  | cHomepage | [https://github.com/ff13dfly/cHomepage](https://github.com/ff13dfly/cHomepage) |

#### Easy Protocol

- **Easy Protocol** is a simple protocol to run application on Anchor node. There is no UI for it, but documents and a demo to implement the functions. In short, it is about how to load `cApp` and confirm the authority of related anchors. Details will be here : [https://github.com/ff13dfly/EasyPolka/blob/main/protocol/README.md](https://github.com/ff13dfly/EasyPolka/blob/main/protocol/README.md).

- Data structure of **Easy Protocol** is developing, more details you can check [https://github.com/ff13dfly/EasyPolka/blob/main/protocol/src/protocol.ts](https://github.com/ff13dfly/EasyPolka/blob/main/protocol/src/protocol.d.ts). Three exposed methods will be supplied.

  1. easyRun. Main method to run `cApp`. Code here: [https://github.com/ff13dfly/EasyPolka/blob/main/protocol/src/interpreter.ts](https://github.com/ff13dfly/EasyPolka/blob/main/protocol/src/interpreter.ts)

      ```Typescript
        //run cApp from anchor location
        //1. if everything is fine, will return group data as `easyResult` which is all details of cApp
        //2. otherwise, will return error list
        (linker:string,inputAPI:APIObject,ck:(res:easyResult) => void,fence?:boolean) => easyResult
      ```

  2. easyProtocol. Get the protocol object from config. Code here: [https://github.com/ff13dfly/EasyPolka/blob/main/protocol/src/format.ts](https://github.com/ff13dfly/EasyPolka/blob/main/protocol/src/format.ts)

      ```Typescript
        //get the cApp protocol from target config
        //it is the format of Easy Protocol, used to create data structure
        (type:rawType,cfg?:any|undefined) => dataProtocol|appProtocol|libProtocol
      ```

  3. linkDecoder. Decode the `Anchor link` to object. Code here: [https://github.com/ff13dfly/EasyPolka/blob/main/protocol/src/decoder.ts](https://github.com/ff13dfly/EasyPolka/blob/main/protocol/src/decoder.ts)

      ```Typescript
        //decoder the anchor linker
        (link:string) => decoderResult
      ```

- Will use `Typescript` to implement the **Easy Protocol**. Automatic test will supply to check it.

- Documents is an important part for **Easy Protocol**, mainly subject as follow.

  1. **Anchor location**. Details about `Anchor Location`, there will be a `Anchor Link` to link the data on Anchor network.

      ```Typescript
        //two ways of anchor location
        type localtionObject=[
          name  : string,
          block : number,
        ]|string;
      ```

      ```Javascript
        //Anchor Link is the same as the URL.
        //There are three formats of Anchor Link.

        //normal Anchor Link
        `anchor://${anchor}/${block}[/]`;

        //latest Anchor Link
        `anchor://${anchor}[/]`;

        //Anchor Link with parameters
        `anchor://${anchor}/${block}[/]?${key_1}=${value_1}&${key_2}=${value_2}`;
      ```

  2. **Declared Hidden**. The owner of `Anchor` can set up the `hide` keywords to hide the history of `Anchor`, directly or refer to another `Anchor`.

  3. **Authority**. The owner of `Anchor` can set up the `auth` keywords to authority to another `Anchor` or special address. The limitation of expired time is related to the `block number` and "0" means forever.

  4. **How to launch**. Details about launching a cApp from an `Anchor Location`. Parameters and the necessary callback will be described exhaustively.

      ```Typescript
        //the protocol details of a cApp
        export type appProtocol={
            "type":rawType.APP;                 // `app` type
            "fmt":formatType.JAVASCRIPT;        // app format, JS only now
            "ver":string;                       // the cApp version, need incremnet when update
            "lib"?:anchorLocation[];            // the list of required anchor list
            "hide"?:hideMap|anchorLocation;     // anchor which storage the hide list defined by hideMap
            "auth"?:authAddress|anchorLocation; // the list of auth anchor;when anchorLocation, map storage there.
            "salt"?:string[2];                  // related to auth and hide, to aviod the same md5 hash. [auth(3),hide(3)]
        }
      ```

  5. **How to call**. Details about communication between data and cApp. The `Data Anchor` set the `call` keywords to target cApp. The implement need to check the `Authority` and `Declared Hidden`, then return the proper cApp object.

      ```Typescript
        //the protocol details of data anchor to call cApp
        export type dataProtocol={
            "type":rawType.DATA;                // `data` type
            "fmt":formatType;                   // raw data format
            "code"?:codeType;                   // data code
            "call"?:anchorLocation;             // call target anchor
            "push"?:string[];                   // list of push to target cApp name. This name is not anchor name
            "args"?:argumentMap;                // arguments will sent to calling cApp
            "hide"?:hideMap|anchorLocation;     // anchor which storage the hide list defined by hideMap
            "auth"?:authAddress|anchorLocation; // the list of auth anchor;when anchorLocation, map storage there.
            "salt"?:string[2];                  // related to auth and hide, to aviod the same md5 hash. [auth(3),hide(3)]
        }
      ```

### Plinth, Instance of Launcher

- Plinth is a web application, follow the **Easy Protocol** to launch **cApp**. The layouts design as follow. You can treat it as a container for **cApp**.

  <img src="http://metanchor.net/easy/plinth_layout.png" width="30%">

- Plinth will be a personal portal by docking the `Anchor`, customer can dock their favorite data or cApp on the sidebar. If current Dapps can be published to `Anchor Network`, user can access the whole web3.0 via such one single entry.

  <img src="http://metanchor.net/easy/plinth_pc_version.png" width="50%">

- Launch **cApp** from `Plinth` will be easy, the `link={anchor_location}` way is recommend. By the way, if you search the `Anchor` directly, you can dock the `Anchor` directly.

  ```Javascript
    //launch the latest anchor
    `http://domain.com/?link=anchor://${anchor_name}/`

    //launch the target anchor
    `http://domain.com/?link=anchor://${anchor_name}/${block_number}`

    //full parameters of anchor location
    `http://domain.com/?link=anchor://${anchor_name}/${block_number}?tpl=black&page=2&force=true`
  ```

- In order to access Anchor node fully, these functions need to implement. Then the **cApp** can be able to run properly.
  <img src="http://metanchor.net/easy/plinth_buttons.png" width="30%">

  1. Account management, the acccounts can access Anchor node, base on Polkadot Keyring, multi accounts support.
    <img src="http://metanchor.net/easy/cApp_user_management.png" width="20%">  <img src="http://metanchor.net/easy/cApp_user_reg.png" width="20%">  <img src="http://metanchor.net/easy/cApp_user_login.png" width="20%">

  2. Server management, nodes which integrated the Anchor pallet and the gateway server in the furture.
    <img src="http://metanchor.net/easy/server_management.png" width="20%">

  3. Publish management, when you are the owner of target cApp, you need this to manage the publish.
    <img src="http://metanchor.net/easy/version_management.png" width="20%">

  4. Setting, basic setting for `Plinth`. As normal setting page, no UI design yet, will be some switcher and button.

- Plinth will be developed on React. `anchorJS` which bases on `@polkadot/api` is needed. `Easy Protocol` implement which is the **milestone 1** is needed.

- Document is about how to explorer `Plinth`, the **cApp** part especially.

- Plinth is not an UI for `Polkadot` network, you can not interact with Substrate node without Anchor pallet. The function is limited to Anchor functions.

### cHomepage, Chain Application ( cApp ) Example

- `cHomepage` is a sample of cApp, you can publish a website for your business by one-page template which all of the data is storage on Anchor node.

- It is important to show the relationship of 3 roles, the authority base on `Easy Protocol`. The only thing all of them need to do is that create an Anchor and update it.
  | Role | description | Anchor Name | Easy Protocol | owner |
  | ------ | ----------- | ------------- |------------- |------------- |
  | Customer |who will run the `homepage` | mock_site | `dataProtocol` | mock_owner_a |
  | Developer |who develop `cHomepage` | mock_cHomeApp | `appProtocol` | mock_owner_b |
  | Supplier |who supply the website template | mock_template | `dataProtocol` | mock_owner_c |

  ```Javascript
    //customer anchor data sample
    {
      //mock anchor name to call cApp
      "name":"mock_site",
      
      //raw data of anchor pallet `protocol` feild
      "raw":{
        "title":"this is mock data",
      },

      //Easy Protocal data
      "protocol":{
        "type":"data",
        "fmt":"json",
        "args":{"style":"dark"},
      }
    }
  ```

  ```Javascript
    //Developer anchor data sample
    {
      "name":"mock_template",
      "raw":"{mock_cHomeApp}",
      "protocol":{
        "type":"app",
        "fmt":"js",
        "ver":"1.0.1",
        "auth":{
          "mock_owner_a":23457,
        }
      }
    }
  ```

  ```Javascript
    //Supplier anchor data sample
    {
      "name":"mock_template",
      "raw":{
        "template":"{{title}} is a good design",
        "vars":{
          "title":"default title",
        }
      },
      "protocol":{
        "type":"data",
        "fmt":"json",
      },
      "auth":{
        "mock_owner_b":0,
      }
    }
  ```

- Native `Javascript` is easy to build it right now. `Jquery` is needed to make the development easy.

- Documents is mainly about the different roles, that's the business model of Web2.0, people do their own job to earn money. Now it is coins.

- Convenient editor will not be supplied this time, just a textarea to update.

### Ecosystem Fit

- A trying of new way to develop application on blockchian. Current Dapps can be repulished on Anchor Network easily, then from customer view, one single portal of browsing the Web 3.0 world is coming.

- More developers who do not understand substrate well can join and build interesting cApps. Developers who have not yet used substrate/Polkadot.

- Creating non-financial applications is a charming subject for blockchain system. More trying is a solution I can image.

- Compared to [RMRK 1.0](https://docs.rmrk.app/rmrk1) which is base on `system.remark`, EasyPolkad do have some advantages as follow.
  1. The simple and memorable `Anchor` name  is better for normal user than hash.
  2. On-chain data do have a block number to mark the finalized time, it is a way to distinguish release time.
  3. Backtrack history of `Anchor` is more convenient for application.
  4. From develop view, by system.remark, it is more difficult to customize protocol implement.
  5. The trade of `Anchor` is another advantage. You can sell your application by this way.

## Team :busts_in_silhouette:

### Team members

- Zhongqiang Fu, individual developer.

### Contact

- **Contact Name:** Zhongqiang Fu
- **Contact Email:** ff13dfly@163.com
- **Website:** https://github.com/ff13dfly/

### Legal Structure

- Individual

### Team's experience

- On substrate, Substrate with Anchor pallet has been build successful and run at [wss:dev.metanchor.net](wss:dev.metanchor.net). I have tried to load a three nodes network successful.

### Team Code Repos

- https://github.com/ff13dfly/
- https://github.com/ff13dfly/Anchor
- https://github.com/ff13dfly/EasyPolka
- https://github.com/ff13dfly/plinth

### Team LinkedIn Profiles (if available)

## Development Status :open_book:

- Demo cApp [freeSaying](https://android.im/vManager/) is published now. The test network is available, you can access [wss:dev.metanchor.net](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fdev.metanchor.net#/explorer).

- Easy Protocol SDK is on process now, it is the basic and important part of Anchor Network. This is also part of Milestone 1. The github link here, [https://github.com/ff13dfly/EasyPolka/tree/main/protocol](https://github.com/ff13dfly/EasyPolka/tree/main/protocol).

## Development Roadmap :nut_and_bolt:

### Overview

- **Total Estimated Duration:** 3.5 month
- **Full-Time Equivalent (FTE):**  3.5
- **Total Costs:** 28,000 USDT

#### Milestone 1 — Easy Protocol v1.0 and SDK

- **Estimated duration:** 1.5 month
- **FTE:**  1.5
- **Costs:** 12,000 USDT

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| 0a. | License | Apache 2.0 |
| 0b. | Documentation | Easy Protocol v1.0 documents in details. This will take a bit long time.|
| 0c. | Testing Guide | Protocol SDK full test via node.js.  |
| 0d. | Docker | Will provide a Dockerfile(s) to run the `Easy Protocol` test. |
| 1. | White Paper | Easy Protocol v1.0 White Paper. Details about `Anchor Linker`,`Data Structure`,`Launching`,`Declared Hide`,`Authority`.|
| 2. | SDK | Easy Protocol v1.0 Javascript SDK, includes `application launcher`, `anchor link decoder`, `foramt creator` components. It is the implement of Easy Protocol, and extend the `Declared Hide` and `Authority` by auto set target Anchor.|

#### Milestone 2 — Plinth, Portal of Anchor

- **Estimated duration:** 1 month
- **FTE:**  1
- **Costs:** 8,000 USDT

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| 0a. | License | Apache 2.0 |
| 0b. | Documentation | We will provide both **inline documentation** of the code and a basic **tutorial** that explains how a user can deploy this launcher to load cApp. |
| 0c. | Testing Guide | Core functions will be fully covered by unit tests to ensure functionality and robustness. In the guide, we will describe how to run these tests. |
| 0d. | Docker |  Will provide a Dockerfile(s) that can run this `Plinth` launcher with anchor node.|
| 1. | Dock Portal | As personal portal, you can dock the `Anchor` to access web 3.0 world. `Anchor` list will be storaged locally. |
| 2. | Setting | Account management, server management,publish management and basic setting will work properly. |

#### Milestone 3 — cHomepage, Sample of cApp

- **Estimated duration:** 1 month
- **FTE:**  1
- **Costs:** 8,000 USDT

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| 0a. | License | Apache 2.0 |
| 0b. | Documentation | We will provide both **inline documentation** of the code and a basic **tutorial** that explains how a user can publish a homepage via cHomepage and how to update the content via anchor updateing.  |
| 0c. | Testing Guide | Core functions will be fully covered by unit tests to ensure functionality and robustness. In the guide, we will describe how to run these tests.|
| 0d. | Docker | Will provide a Dockerfile(s) that can run `Plinth` to call `cHomepage`. |
| 1. | Viewer | Load `viewer` from mock customer data anchor, and `viewer` will get datat from template anchor |
| 2. | Editor | Simple editor for customer, can update the data anchor. |
| 3. | Authority Editor | Simple editor to update the authority of anchor. |

## Future Plans

- This applying is the direct way of cApp. If it is done, developer can start their work easily on it. The next step should be the gateway for cApp.

| Order | Name | Demo | Github | introduction |
| -----: | ----------- | ------------- | ------------- | ------------- |
| 1 | **Anchor** | wss://dev.metanchor.net [https://playground.metanchor.net](https://playground.metanchor.net) | [https://github.com/ff13dfly/Anchor](https://github.com/ff13dfly/Anchor) | Linked list on chian & Name service |
| 2 | **Easy Protocol** | - | [https://github.com/ff13dfly/EasyPolka/tree/main/protocol](https://github.com/ff13dfly/EasyPolka/tree/main/protocol) | Protocol of Chain Application |
| 3 | **Plinth** | - | [https://github.com/ff13dfly/plinth](https://github.com/ff13dfly/plinth) | Launcher for cApp |
| 4 | **cHomepage** | Not yet |[https://github.com/ff13dfly/cHomepage](https://github.com/ff13dfly/cHomepage) |  Demo cApp |
| 5 | vGateway | [https://android.im/vGateway/](https://android.im/vGateway/) | Not yet | Gateway access to vServices |
| 6 | vManager | [https://android.im/vManager/](https://android.im/vManager/) | Not yet | Management portal for vServices |
| 7 | vCache | No domain, node.js app | Not yet |Anchor cache vService, sample vService |

- The functions above, you can test on the cApp [freeSaying](https://freesaying.net).

- EasyPolka framework will be published step by step and try to meet W3F standard, it is easy for the develop to understand.

## Additional Information :heavy_plus_sign:

**How did you hear about the Grants Program?** Web3 Foundation Website.

- Demo cApp [freeSaying](https://android.im/vManager/) is published now. The test network is available, you can access [wss:dev.metanchor.net](wss:dev.metanchor.net).

- [Anchor Pallet](https://github.com/ff13dfly/Anchor) is just finished. Now will try to meet the W3F standard in my furture projects.